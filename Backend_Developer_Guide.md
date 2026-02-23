# Hướng Dẫn Phát Triển Backend (Backend Developer Guide) - Internship OneConnect (IOC)

Tài liệu này hướng dẫn cách sử dụng, cấu trúc code và cách phát triển các tính năng mới cho hệ thống Backend của dự án Internship OneConnect (IOC).

## 1. Kiến Trúc Hệ Thống (Architecture)

Hệ thống được xây dựng theo kiến trúc **Clean Architecture** kết hợp với các pattern hiện đại như **CQRS** (Command Query Responsibility Segregation).

### Các Lớp Trong Hệ Thống:

- **IOCv2.Domain**: Chứa các thực thể (Entities), Enums, và các quy tắc nghiệp vụ cốt lõi. Không phụ thuộc vào bất kỳ thư viện ngoài nào ngoại trừ các thư viện hệ thống.
- **IOCv2.Application**: Chứa logic nghiệp vụ (Services, MediatR Handlers), DTOs, Mappings, và Interfaces cho các service bên ngoài. Đây là lớp điều phối chính.
- **IOCv2.Infrastructure**: Chứa các triển khai chi tiết cho việc lưu trữ (Persistence - EF Core), Security (JWT), User Identity, Redis Cache, v.v.
- **IOCv2.API**: Chứa các Controllers, Middlewares, Configurations để giao tiếp với bên ngoài.

---

## 2. Công Nghệ Sử Dụng (Tech Stack)

- **Language**: C# 13 / .NET 9
- **Database**: PostgreSQL (Entity Framework Core 9)
- **Mapping**: AutoMapper
- **Messaging**: MediatR (CQRS Pattern)
- **Validation**: FluentValidation
- **Caching**: Redis (IDistributedCache)
- **Logging**: Microsoft.Extensions.Logging (Default)
- **Documentation**: Swagger/OpenAPI

---

## 3. Cấu Trúc Folder & Quy Tắc Đặt Tên

### Folder Structure

- `IOCv2.Domain/Entities/`: Tên file PascalCase, số ít (ví dụ: `Student.cs`, `University.cs`).
- `IOCv2.Application/Features/[FeatureName]/Commands/`: Chứa các yêu cầu thay đổi dữ liệu (Create, Update, Delete).
- `IOCv2.Application/Features/[FeatureName]/Queries/`: Chứa các yêu cầu đọc dữ liệu (Get, Search).
- `IOCv2.Infrastructure/Persistence/Configurations/`: Cấu hình Fluent API cho EF Core.

### Coding Rules

- Sử dụng **File-scoped namespaces** để giảm indentation.
- Luôn sử dụng `async/await` cho các thao tác IO (DB, Network).
- Tuân thủ quy tắc đặt tên: Class/Method/Property: PascalCase, Parameter/Variable: camelCase.
- Sử dụng `var` khi kiểu dữ liệu đã rõ ràng bên phải.

---

## 4. Cách Thêm Tính Năng Mới (Step-by-Step)

Giả sử bạn muốn thêm tính năng "Lấy danh sách sinh viên" (GetStudents):

### Bước 1: Tạo Response DTO

Tạo file `GetStudentsResponse.cs` trong `IOCv2.Application/Features/Students/Queries/GetStudents/`:
Sử dụng `IMapFrom<Student>` để tự động mapping.

```csharp
public class GetStudentsResponse : IMapFrom<Student>
{
    public Guid Id { get; set; }
    public string FullName { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public string StudentCode { get; set; } = string.Empty;
    // ... các trường khác
}
```

### Bước 2: Tạo Query & Handler

Tạo file `GetStudentsQuery.cs` cùng thư mục:

```csharp
// Query record định nghĩa tham số đầu vào
public record GetStudentsQuery(PaginationParams Pagination) : IRequest<Result<PagedResult<GetStudentsResponse>>>;

// Handler xử lý logic
public class GetStudentsQueryHandler : IRequestHandler<GetStudentsQuery, Result<PagedResult<GetStudentsResponse>>>
{
    private readonly IUnitOfWork _unitOfWork;
    private readonly IMapper _mapper;

    public GetStudentsQueryHandler(IUnitOfWork unitOfWork, IMapper mapper)
    {
        _unitOfWork = unitOfWork;
        _mapper = mapper;
    }

    public async Task<Result<PagedResult<GetStudentsResponse>>> Handle(GetStudentsQuery request, CancellationToken cancellationToken)
    {
        // 1. Khởi tạo query từ Repository
        var query = _unitOfWork.Repository<Student>().Query();

        // 2. Áp dụng search, filter, sort (xem phần 8)
        // query = query.ApplyGlobalSearch(request.Pagination.Search, searchableFields);

        // 3. Projection và Phân trang
        var result = await query
            .ProjectTo<GetStudentsResponse>(_mapper.ConfigurationProvider)
            .ToPagedResultAsync(request.Pagination);

        // 4. Trả về kết quả
        return Result<PagedResult<GetStudentsResponse>>.Success(result);
    }
}
```

### Bước 3: Tạo Controller

Controllers trong project kế thừa trực tiếp từ `ControllerBase` hoặc `BaseController` (nếu có) và inject `IMediator`.

```csharp
[ApiController]
[Route("api/[controller]")]
public class StudentsController : ControllerBase
{
    private readonly IMediator _mediator;
    public StudentsController(IMediator mediator) => _mediator = mediator;

    // Helper method để chuẩn hóa response (thường được đặt trong BaseController)
    private IActionResult HandleResult<T>(Result<T> result)
    {
        if (result.IsSuccess)
        {
            if (result.HasWarning) return Ok(new { data = result.Data, warning = result.Warning });
            return Ok(result.Data);
        }

        return result.ErrorType switch
        {
            ResultErrorType.NotFound => NotFound(new { message = result.Error }),
            ResultErrorType.Unauthorized => Unauthorized(new { message = result.Error }),
            ResultErrorType.Conflict => Conflict(new { message = result.Error }),
            _ => BadRequest(new { message = result.Error })
        };
    }

    [HttpGet]
    public async Task<IActionResult> GetStudents([FromQuery] PaginationParams pagination)
    {
        var result = await _mediator.Send(new GetStudentsQuery(pagination));
        return HandleResult(result);
    }
}
```

---

## 5. Các Patterns Quan Trọng

### Result Pattern

Sử dụng `Result<T>` để trả về kết quả thành công hoặc lỗi từ tầng Application.

- **Thành công**: `Result<T>.Success(data)` hoặc `Result<T>.SuccessWithWarning(data, "Lưu ý...")`
- **Thất bại**: `Result<T>.Failure("Thông báo lỗi", ResultErrorType.BadRequest)` hoặc các method shortcut như `Result<T>.NotFound("Không tìm thấy")`.

### Validation Behavior

Mọi Command được gửi qua MediatR sẽ tự động được kiểm tra bởi các lớp kế thừa `AbstractValidator<T>`. Nếu có lỗi, hệ thống sẽ throw `ValidationException` và trả về mã lỗi 400 (Bad Request).

### AutoMapper (IMapFrom)

Interface `IMapFrom<T>` giúp tự động cấu hình Mapping. Mặc định nó sẽ thực hiện `CreateMap<T, GetType>().ReverseMap()`.

```csharp
// Trong DTO
public class StudentDto : IMapFrom<Student> { }

// Nếu cần custom mapping:
public void Mapping(Profile profile)
{
    profile.CreateMap<Student, StudentDto>()
        .ForMember(d => d.UniversityName, opt => opt.MapFrom(s => s.University.Name));
}
```

### Unit of Work & Generic Repository

Dùng để quản lý dữ liệu và transaction.

- `_unitOfWork.Repository<T>().Query()`: Lấy IQueryable để đọc dữ liệu.
- `_unitOfWork.Repository<T>().AddAsync(entity)`: Thêm mới.
- `_unitOfWork.SaveChangeAsync()`: Thực thi lưu xuống DB.

---

## 6. EF Core Migrations

Chạy lệnh Migration tại thư mục gốc `Internship-OneConnect_IOC_v2.0_Backend` (nơi chứa file solution `.sln`):

1. **Thêm Migration:** 
   ```bash
   dotnet ef migrations add [MigrationName] --project IOCv2.Infrastructure --startup-project IOCv2.API
   ```
2. **Cập nhật Database:** 
   ```bash
   dotnet ef database update --project IOCv2.Infrastructure --startup-project IOCv2.API
   ```
3. **Xóa Migration cuối (khi chưa update DB):** 
   ```bash
   dotnet ef migrations remove --project IOCv2.Infrastructure --startup-project IOCv2.API
   ```
   
**Lưu ý**: Trong môi trường `Development`, ứng dụng sẽ tự động chạy migration khi khởi động (được cấu hình trong `Program.cs`).

---

## 7. Hướng Dẫn Chạy Project

### Yêu cầu:
- Docker Desktop (để chạy PostgreSQL và Redis)
- .NET 9 SDK

### Các bước:

1. **Cấu hình Environment**:
   - Kiểm tra file `appsettings.json` hoặc biến môi trường trong `docker-compose.yml`.
   - Connection String mặc định kết nối tới `iocv2_db` (PostgreSQL) và `iocv2_redis` (Redis).

2. **Khởi chạy Infrastructure (DB & Redis)**:
   Mở terminal tại thư mục gốc và chạy:
   ```bash
   docker-compose up -d db redis
   ```
   (Lệnh này sẽ khởi động container `iocv2_db` và `iocv2_redis`)

3. **Chạy Backend API**:
   ```bash
   dotnet run --project IOCv2.API
   ```
   Hoặc mở Solution bằng Visual Studio / Rider và nhấn F5.

4. **Truy cập Swagger**:
   Mở trình duyệt và truy cập: `[http://localhost:5000/swagger](http://localhost:5133/swagger)` (hoặc port 8080 tùy cấu hình).
### Run full project với Docker: docker compose up -d --build

---

## 8. Hệ Thống Search, Filter & Sort Đa Năng

Sử dụng các Extension Method trong `IOCv2.Application/Extensions/Query/` để xử lý tìm kiếm, lọc và phân trang.

```csharp
// 1. Search (Tìm kiếm theo nhiều trường)
var searchableFields = new List<Expression<Func<Student, string?>>> {
    u => u.FullName, u => u.Email, u => u.StudentCode
};
query = query.ApplyGlobalSearch(request.Pagination.Search, searchableFields);

// 2. Filter (Lọc theo điều kiện chính xác)
var filterMapping = new Dictionary<string, Expression<Func<Student, object?>>> {
    { "status", u => u.Status },
    { "universityId", u => u.UniversityId }
};
query = query.ApplyFilters(request.Pagination.Filters, filterMapping);

// 3. Sort (Sắp xếp)
var sortMapping = new Dictionary<string, Expression<Func<Student, object?>>> {
    { "fullname", u => u.FullName },
    { "createdAt", u => u.CreatedAt }
};
query = query.ApplySorting(request.Pagination.OrderBy, sortMapping, u => u.Id);
```

---

## 9. Caching với Redis

Project sử dụng `IDistributedCache` để tương tác với Redis. 
- Container name: `iocv2_redis`
- Port: `6379`
- Connection String cấu hình trong `appsettings.json`.

---

## 10. Quản lý Message & Đa ngôn ngữ (Localization)

Resources nằm tại `IOCv2.Application/Resources/`:

- `ErrorMessages.resx`: Chứa các key báo lỗi.
- `Messages.resx`: Chứa các key thông báo thành công.

Cách sử dụng: Inject `IStringLocalizer<ErrorMessages>` hoặc `IStringLocalizer<Messages>` để lấy chuỗi thông báo theo ngôn ngữ hiện tại (dựa vào header `Accept-Language` của request).

---

## 11. Quy Trình Phát Triển Feature (Development Workflow)

Khi nhận một tính năng mới (User Story / Task), thành viên cần tuân thủ quy trình sau **từ đầu đến cuối**:

### 11.1. Phân Tích Yêu Cầu

1. Đọc kỹ User Story / Task trên Jira (hoặc công cụ quản lý tương ứng).
2. Xác định rõ:
   - Đây là **Query** (đọc dữ liệu) hay **Command** (thay đổi dữ liệu)?
   - Entity nào liên quan? Có cần tạo Entity mới không?
   - Có cần thêm Migration không?
   - API endpoint cần trả về response format nào?

### 11.2. Thứ Tự Viết Code (Bottom-Up)

Luôn viết code theo thứ tự **từ trong ra ngoài** (từ Domain → Application → Infrastructure → API):

```
📦 Bước 1: Domain Layer (nếu cần)
│   ├── Thêm Entity mới hoặc sửa Entity hiện tại
│   └── Thêm Enum nếu cần
│
📦 Bước 2: Application Layer
│   ├── 2a. Tạo Response DTO (XxxResponse.cs)
│   ├── 2b. Tạo Command/Query (XxxCommand.cs / XxxQuery.cs)
│   ├── 2c. Tạo Handler (XxxHandler.cs)
│   ├── 2d. Tạo Validator (XxxValidator.cs)
│   └── 2e. Thêm MessageKeys nếu cần (Constants/MessageKeys.cs)
│
📦 Bước 3: Infrastructure Layer (nếu cần)
│   ├── Thêm Configuration cho Entity mới (Persistence/Configurations/)
│   ├── Cập nhật DbContext (nếu thêm DbSet mới)
│   └── Tạo Migration
│
📦 Bước 4: API Layer
│   ├── Thêm endpoint vào Controller hiện tại hoặc tạo Controller mới
│   └── Bổ sung XML Documentation cho Swagger
```

### 11.3. Ví Dụ: Tạo Feature "CreateAdminUser" (Command)

Theo cấu trúc folder hiện tại, mỗi Command/Query được đặt trong folder riêng gồm **4 file**:

```
IOCv2.Application/
└── Features/
    └── Admin/
        └── Users/
            └── Commands/
                └── CreateAdminUser/
                    ├── CreateAdminUserCommand.cs     ← Định nghĩa tham số đầu vào
                    ├── CreateAdminUserHandler.cs     ← Logic nghiệp vụ
                    ├── CreateAdminUserResponse.cs    ← DTO trả về
                    └── CreateAdminUserValidator.cs   ← Validation FluentValidation
```

### 11.4. Ví Dụ: Tạo Feature "GetAdminUsers" (Query)

```
IOCv2.Application/
└── Features/
    └── Admin/
        └── Users/
            └── Queries/
                └── GetAdminUsers/
                    ├── GetAdminUsersQuery.cs          ← Tham số tìm kiếm/filter/sort
                    ├── GetAdminUsersHandler.cs        ← Logic truy vấn
                    ├── GetAdminUsersResponse.cs       ← DTO kết quả
                    └── GetAdminUsersValidator.cs      ← Validation parameters
```

### 11.5. Kiểm Tra Trước Khi Commit

- [ ] Build project thành công (`dotnet build`)
- [ ] Chạy thử API endpoint trên Swagger
- [ ] Kiểm tra response format đúng chuẩn
- [ ] Validation hoạt động đúng (test với dữ liệu sai)
- [ ] Đã thêm XML documentation cho Controller action

---

## 12. C# Code Style & Conventions

### 12.1. Namespace & File Structure

| Quy tắc | Ví dụ đúng | Ví dụ sai |
|---------|-----------|----------|
| Namespace phải khớp với đường dẫn folder | `IOCv2.Application.Features.Admin.Users.Commands.CreateAdminUser` | `IOCv2.Application.Admin` |
| **Ưu tiên file-scoped namespace** cho file mới trong API layer | `namespace IOCv2.API.Configurations;` | — |
| Block-scoped namespace vẫn được chấp nhận | `namespace IOCv2.Application.Features... { }` | — |

### 12.2. Quy Tắc Đặt Tên

| Thành phần | Convention | Ví dụ |
|-----------|-----------|-------|
| **Class / Record / Enum** | PascalCase | `CreateAdminUserCommand`, `UserRole` |
| **Interface** | `I` + PascalCase | `IUnitOfWork`, `IGenericRepository<T>` |
| **Method** | PascalCase | `Handle()`, `SaveChangeAsync()` |
| **Property** | PascalCase | `FullName`, `CreatedAt` |
| **Private field** | `_` + camelCase | `_unitOfWork`, `_mapper`, `_logger` |
| **Parameter / Local variable** | camelCase | `cancellationToken`, `parsedRole`, `query` |
| **Constant** | PascalCase (trong nested static class) | `MessageKeys.Users.NotFound` |
| **Enum member** | PascalCase | `UserRole.SuperAdmin`, `UserStatus.Active` |
| **Enum with backing type** | Khai báo rõ kiểu `: short` | `public enum UserRole : short` |

### 12.3. Record vs Class

```csharp
// ✅ Dùng record cho Command/Query (immutable input)
public record CreateAdminUserCommand : IRequest<Result<CreateAdminUserResponse>>
{
    public string FullName { get; init; } = null!;
    public string Email { get; init; } = null!;
}

// ✅ Dùng record cho Query với default values
public record GetAdminUsersQuery : IRequest<Result<PaginatedResult<GetAdminUsersResponse>>>
{
    public int PageNumber { get; init; } = 1;
    public int PageSize { get; init; } = 10;
}

// ✅ Dùng class cho Response DTO (cần AutoMapper)
public class CreateAdminUserResponse : IMapFrom<User>
{
    public Guid UserId { get; set; }
    public string Email { get; set; } = null!;
}

// ✅ Dùng class cho Entity
public class User : BaseEntity
{
    public Guid UserId { get; set; }
    public string Email { get; set; } = null!;
}
```

### 12.4. Null Safety & Default Values

```csharp
// ✅ Dùng null! cho required string properties
public string FullName { get; set; } = null!;
public string Email { get; init; } = null!;

// ✅ Dùng string.Empty cho optional string properties trong Response DTO
public string FullName { get; set; } = string.Empty;

// ✅ Dùng ? cho truly nullable properties
public string? PhoneNumber { get; set; }
public Guid? UnitId { get; init; }
public DateOnly? DateOfBirth { get; set; }

// ✅ Collection properties luôn khởi tạo rỗng
public virtual ICollection<RefreshToken> RefreshTokens { get; set; } = new List<RefreshToken>();
```

### 12.5. Async Pattern

```csharp
// ✅ Tất cả method DB/IO phải là async, và luôn truyền CancellationToken
public async Task<Result<Response>> Handle(Command request, CancellationToken cancellationToken)
{
    var exists = await _unitOfWork.Repository<User>()
        .ExistsAsync(u => u.Email == request.Email, cancellationToken);

    await _unitOfWork.Repository<User>().AddAsync(user, cancellationToken);
    await _unitOfWork.SaveChangeAsync(cancellationToken);
}

// ❌ KHÔNG ĐƯỢC quên CancellationToken
await _unitOfWork.SaveChangeAsync(); // Thiếu cancellationToken!
```

### 12.6. Expression Body & Compact Syntax

```csharp
// ✅ Dùng expression body cho constructor đơn giản
public StudentsController(IMediator mediator) => _mediator = mediator;

// ✅ Dùng expression body cho computed property
public bool HasPreviousPage => PageNumber > 1;
public bool HasWarning => !string.IsNullOrEmpty(Warning);

// ✅ Dùng target-typed new
public static Result<T> Success(T data) => new(true, data, null, ResultErrorType.None);
```

### 12.7. Pattern Matching (Switch Expression)

```csharp
// ✅ Dùng switch expression cho mapping logic
var statusCode = result.ErrorType switch
{
    ResultErrorType.NotFound => 404,
    ResultErrorType.Unauthorized => 401,
    ResultErrorType.Forbidden => 403,
    ResultErrorType.Conflict => 409,
    _ => 400
};

// ✅ Dùng tuple pattern cho sorting
query = (request.SortColumn?.ToLower(), request.SortOrder?.ToLower()) switch
{
    ("fullname", "desc")  => query.OrderByDescending(u => u.FullName),
    ("fullname", _)       => query.OrderBy(u => u.FullName),
    ("createdat", "desc") => query.OrderByDescending(u => u.CreatedAt),
    _                     => query.OrderByDescending(u => u.CreatedAt)
};
```

### 12.8. Comment Style

```csharp
// ✅ Comment tiếng Việt HOẶC tiếng Anh (nhất quán trong 1 file)
// Dùng inline comment ngắn gọn giải thích WHY, không giải thích WHAT
public DateTime? DeletedAt { get; set; } // database xử lý datetime nhanh hơn boolean

// ✅ XML documentation cho Controller actions (bắt buộc cho Swagger)
/// <summary>
/// Get paginated list of admin accounts with optional filters and sorting.
/// </summary>
[HttpGet]
public async Task<IActionResult> GetAdminUsers([FromQuery] GetAdminUsersQuery query)

// ✅ Section comment trong Handler để chia logic
// 1. Validate auditor
// 2. Parse Role
// 3. Check email conflict
// 4. Create user
```

---

## 13. Quy Tắc Cấu Trúc File & Folder

### 13.1. Cấu Trúc Tổng Quan

```
Internship-OneConnect_IOC_v2.0_Backend/
├── IOCv2.Domain/                          ← Không phụ thuộc gì
│   ├── Entities/                          ← BaseEntity, User, Student...
│   └── Enums/                             ← UserRole, UserStatus...
│
├── IOCv2.Application/                     ← Chỉ phụ thuộc Domain
│   ├── Common/
│   │   ├── Behaviors/                     ← MediatR Pipeline (ValidationBehavior)
│   │   ├── Exceptions/                    ← BusinessException, NotFoundException
│   │   └── Models/                        ← Result<T>, PaginatedResult<T>, ErrorResponse
│   ├── Constants/                         ← MessageKeys
│   ├── Extensions/
│   │   ├── Mappings/                      ← MappingProfile, IMapFrom<T>
│   │   ├── Pagination/                    ← Extension methods phân trang
│   │   └── Query/                         ← ApplyGlobalSearch, ApplyFilters...
│   ├── Features/                          ← ⭐ Nơi viết Feature chính
│   │   ├── Admin/Users/Commands/...
│   │   ├── Admin/Users/Queries/...
│   │   ├── Authentication/Commands/...
│   │   └── Users/...
│   ├── Interfaces/                        ← Interface cho Infrastructure
│   ├── Resources/                         ← Localization (.resx files)
│   ├── Services/                          ← Application services
│   ├── Validators/                        ← Shared validators
│   └── DependencyInjection.cs
│
├── IOCv2.Infrastructure/                  ← Triển khai interfaces
│   ├── BackgroundJobs/                    ← Hosted services
│   ├── Persistence/
│   │   ├── AppDbContext.cs
│   │   ├── Configurations/               ← EF Core Fluent API configs
│   │   ├── DbInitializer.cs              ← Seed data
│   │   ├── Repositories/                 ← GenericRepository
│   │   └── UnitOfWork.cs
│   ├── Security/                          ← JWT, Password hashing
│   ├── Services/                          ← Email, Cache, RateLimiting...
│   ├── Migrations/
│   └── DependencyInjection.cs
│
├── IOCv2.API/                             ← Entry point
│   ├── Attributes/                        ← Custom attributes
│   ├── Configurations/                    ← Extension method configs (modular)
│   ├── Controllers/
│   │   ├── ApiControllerBase.cs           ← Base controller với HandleResult<T>
│   │   ├── Admin/AdminUsersController.cs
│   │   └── Auth/AuthController.cs
│   ├── Middlewares/                        ← Exception, RateLimiting...
│   └── Program.cs                         ← Minimal hosting
│
├── docker-compose.yml
├── IOCv2.sln
└── README.md
```

### 13.2. Quy Tắc Đặt Tên File

| Layer | Loại File | Naming Convention | Ví dụ |
|-------|----------|-------------------|-------|
| Domain | Entity | PascalCase, số ít | `User.cs`, `Student.cs` |
| Domain | Enum | PascalCase, mô tả rõ | `UserRole.cs`, `UserStatus.cs` |
| Application | Command | `[Action][Entity]Command.cs` | `CreateAdminUserCommand.cs` |
| Application | Query | `[Action][Entity]Query.cs` | `GetAdminUsersQuery.cs` |
| Application | Handler | `[Action][Entity]Handler.cs` | `CreateAdminUserHandler.cs` |
| Application | Validator | `[Action][Entity]Validator.cs` | `CreateAdminUserValidator.cs` |
| Application | Response DTO | `[Action][Entity]Response.cs` | `CreateAdminUserResponse.cs` |
| Application | Interface | `I[ServiceName].cs` | `IUnitOfWork.cs`, `ICacheService.cs` |
| Infrastructure | Configuration | `[Entity]Configuration.cs` | `UserConfiguration.cs` |
| Infrastructure | Service impl | `[ServiceName].cs` | `PasswordService.cs`, `JwtTokenService.cs` |
| API | Controller | `[Module]Controller.cs` | `AdminUsersController.cs` |
| API | Configuration | `[Feature]Config.cs` | `SwaggerConfig.cs`, `CorsConfig.cs` |
| API | Middleware | `[Feature]Middleware.cs` | `ExceptionMiddleware.cs` |

### 13.3. Quy Tắc Tạo Feature Folder

```
Features/
└── [ModuleName]/              ← Admin, Authentication, Students, Universities...
    └── [EntityName]/          ← Users, Courses, Internships...
        ├── Commands/
        │   ├── Create[Entity]/
        │   │   ├── Create[Entity]Command.cs
        │   │   ├── Create[Entity]Handler.cs
        │   │   ├── Create[Entity]Response.cs
        │   │   └── Create[Entity]Validator.cs
        │   ├── Update[Entity]/
        │   ├── Delete[Entity]/
        │   └── [Action][Entity]/      ← ToggleUserStatus, ResetUserPassword...
        └── Queries/
            ├── Get[Entity]s/          ← Danh sách (số nhiều)
            │   ├── Get[Entity]sQuery.cs
            │   ├── Get[Entity]sHandler.cs
            │   ├── Get[Entity]sResponse.cs
            │   └── Get[Entity]sValidator.cs
            └── Get[Entity]ById/       ← Chi tiết (số ít + ById)
```

---

## 14. Git Workflow & Branching Strategy

### 14.1. Branch Naming Convention

```
main                           ← Production-ready code
├── develop                    ← Integration branch
│   ├── feat/[feature-name]    ← Feature mới (VD: feat/admin-users)
│   ├── fix/[bug-name]         ← Sửa bug (VD: fix/login-token-expired)
│   ├── refactor/[scope]       ← Refactor code (VD: refactor/logging)
│   └── docs/[scope]           ← Cập nhật tài liệu (VD: docs/api-guide)
```

### 14.2. Commit Message Convention

Sử dụng format: `<type>(<scope>): <mô tả ngắn>`

```bash
# Types phổ biến:
feat(admin-users): add create admin user endpoint
fix(auth): resolve token refresh race condition
refactor(middleware): move logging to infrastructure layer
docs(api): update swagger documentation
chore(docker): update docker-compose ports
style(controllers): apply consistent route formatting
```

### 14.3. Quy Trình Làm Việc

1. **Pull code mới nhất** từ `develop`:
   ```bash
   git checkout develop
   git pull origin develop
   ```
2. **Tạo branch mới** từ `develop`:
   ```bash
   git checkout -b feat/[feature-name]
   ```
3. **Viết code** theo thứ tự (Section 11.2).
4. **Commit thường xuyên** theo convention (Section 14.2).
5. **Push và tạo Pull Request** vào `develop`:
   ```bash
   git push origin feat/[feature-name]
   ```
6. **Review** bởi ít nhất 1 thành viên.
7. **Merge** sau khi approved.

---

## 15. Code Review Checklist

Khi review PR của đồng đội, kiểm tra theo danh sách sau:

### 15.1. Architecture & Structure
- [ ] Feature được đặt đúng module/folder (`Features/[Module]/[Entity]/Commands|Queries/`)
- [ ] Đủ 4 file cho mỗi Command/Query: Command/Query, Handler, Response, Validator
- [ ] Không import trực tiếp Infrastructure từ Application layer (vi phạm Clean Architecture)
- [ ] Controller kế thừa `ApiControllerBase` và dùng `HandleResult<T>()`

### 15.2. Code Quality
- [ ] Tên class/method/property đúng convention (Section 12.2)
- [ ] Private fields có prefix `_`
- [ ] Dùng `record` cho Command/Query, `class` cho Response và Entity
- [ ] Tất cả async method truyền `CancellationToken`
- [ ] Không có `magic string` — dùng `MessageKeys` constants
- [ ] Error messages dùng `IMessageService` thay vì hardcode string

### 15.3. Validation
- [ ] Mỗi Command/Query đều có Validator tương ứng
- [ ] Validator access modifier là `internal` class
- [ ] Validation rules bao gồm: NotEmpty, MaxLength, format check...
- [ ] Enum parsing dùng `Enum.TryParse<T>(value, true, out _)`

### 15.4. Database & Performance
- [ ] Dùng `.AsNoTracking()` cho Query (read-only)
- [ ] Dùng `ProjectTo<T>()` thay vì load entity rồi map
- [ ] Transaction có `try/catch` với `RollbackTransactionAsync`
- [ ] Pagination dùng `Skip/Take` (không load toàn bộ)
- [ ] Index đã được cấu hình cho các trường thường filter/sort

### 15.5. Controller & API
- [ ] XML documentation `/// <summary>` cho mỗi action
- [ ] `[ProducesResponseType]` khai báo cho success và error
- [ ] Route sử dụng dấu `/` đầu: `[Route("users")]`
- [ ] Controller có `[Tags("...")]` cho nhóm Swagger
- [ ] Controller có `[Authorize]` nếu cần xác thực

---

## 16. Common Mistakes & Anti-Patterns

### ❌ Sai: Viết logic nghiệp vụ trong Controller

```csharp
// ❌ KHÔNG LÀM THẾ NÀY
[HttpPost]
public async Task<IActionResult> CreateUser([FromBody] CreateUserDto dto)
{
    var user = new User { Email = dto.Email };  // Logic trong Controller!
    _dbContext.Users.Add(user);
    await _dbContext.SaveChangesAsync();
    return Ok(user);
}
```

### ✅ Đúng: Controller chỉ điều phối qua MediatR

```csharp
// ✅ LÀM THẾ NÀY
[HttpPost]
[Route("users")]
public async Task<IActionResult> CreateAdminUser([FromBody] CreateAdminUserCommand command)
{
    var result = await _mediator.Send(command);
    return HandleResult(result);
}
```

### ❌ Sai: Hardcode error message

```csharp
// ❌ KHÔNG hardcode string
return Result<T>.Failure("User not found", ResultErrorType.NotFound);
```

### ✅ Đúng: Dùng MessageKeys + IMessageService

```csharp
// ✅ Dùng localized message
return Result<T>.Failure(
    _messageService.GetMessage(MessageKeys.Users.NotFound),
    ResultErrorType.NotFound
);
```

### ❌ Sai: Inject DbContext trực tiếp

```csharp
// ❌ Vi phạm Clean Architecture
public class MyHandler
{
    private readonly AppDbContext _context;    // ← Phụ thuộc trực tiếp Infrastructure
}
```

### ✅ Đúng: Dùng IUnitOfWork + IGenericRepository

```csharp
// ✅ Dependency Inversion qua interface
public class MyHandler
{
    private readonly IUnitOfWork _unitOfWork;  // ← Interface từ Application layer
}
```

### ❌ Sai: Quên Transaction khi thao tác nhiều bảng

```csharp
// ❌ Thiếu transaction — nếu step 2 fail, step 1 vẫn lưu
await _unitOfWork.Repository<User>().AddAsync(user);
await _unitOfWork.SaveChangeAsync(ct);
await _unitOfWork.Repository<Student>().AddAsync(student);  // Nếu fail ở đây?
await _unitOfWork.SaveChangeAsync(ct);
```

### ✅ Đúng: Wrap trong Transaction

```csharp
// ✅ Atomic operation
await _unitOfWork.BeginTransactionAsync(cancellationToken);
try
{
    await _unitOfWork.Repository<User>().AddAsync(user, cancellationToken);
    await _unitOfWork.SaveChangeAsync(cancellationToken);

    await _unitOfWork.Repository<Student>().AddAsync(student, cancellationToken);
    await _unitOfWork.SaveChangeAsync(cancellationToken);

    await _unitOfWork.CommitTransactionAsync(cancellationToken);
}
catch
{
    await _unitOfWork.RollbackTransactionAsync(cancellationToken);
    throw;
}
```

---

## 17. Cấu Trúc Mẫu Cho Từng Loại File

### 17.1. Command (Template)

```csharp
using IOCv2.Application.Common.Models;
using MediatR;

namespace IOCv2.Application.Features.[Module].[Entity].Commands.[Action][Entity];

public record [Action][Entity]Command : IRequest<Result<[Action][Entity]Response>>
{
    // Properties dùng { get; init; } cho immutability
    public string PropertyName { get; init; } = null!;
    public Guid? OptionalId { get; init; }
}
```

### 17.2. Handler (Template)

```csharp
using AutoMapper;
using IOCv2.Application.Common.Models;
using IOCv2.Application.Constants;
using IOCv2.Application.Interfaces;
using IOCv2.Domain.Entities;
using MediatR;
using Microsoft.Extensions.Logging;

namespace IOCv2.Application.Features.[Module].[Entity].Commands.[Action][Entity];

public class [Action][Entity]Handler : IRequestHandler<[Action][Entity]Command, Result<[Action][Entity]Response>>
{
    private readonly IUnitOfWork _unitOfWork;
    private readonly IMapper _mapper;
    private readonly ILogger<[Action][Entity]Handler> _logger;
    private readonly IMessageService _messageService;

    public [Action][Entity]Handler(
        IUnitOfWork unitOfWork,
        IMapper mapper,
        ILogger<[Action][Entity]Handler> logger,
        IMessageService messageService)
    {
        _unitOfWork = unitOfWork;
        _mapper = mapper;
        _logger = logger;
        _messageService = messageService;
    }

    public async Task<Result<[Action][Entity]Response>> Handle(
        [Action][Entity]Command request,
        CancellationToken cancellationToken)
    {
        // 1. Validate business rules
        // 2. Execute business logic
        // 3. Persist data
        // 4. Return result
        throw new NotImplementedException();
    }
}
```

### 17.3. Validator (Template)

```csharp
using FluentValidation;

namespace IOCv2.Application.Features.[Module].[Entity].Commands.[Action][Entity];

internal class [Action][Entity]Validator : AbstractValidator<[Action][Entity]Command>
{
    public [Action][Entity]Validator()
    {
        RuleFor(x => x.PropertyName)
            .NotEmpty()
            .MaximumLength(150);
    }
}
```

### 17.4. Response DTO (Template)

```csharp
using IOCv2.Application.Extensions.Mappings;
using IOCv2.Domain.Entities;

namespace IOCv2.Application.Features.[Module].[Entity].Commands.[Action][Entity];

public class [Action][Entity]Response : IMapFrom<[DomainEntity]>
{
    public Guid Id { get; set; }
    public string Name { get; set; } = string.Empty;

    // Custom mapping (nếu cần)
    public void Mapping(MappingProfile profile)
    {
        profile.CreateMap<[DomainEntity], [Action][Entity]Response>()
            .ForMember(dest => dest.Name, opt => opt.MapFrom(src => src.FullName));
    }
}
```

### 17.5. EF Core Configuration (Template)

```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;
using IOCv2.Domain.Entities;

namespace IOCv2.Infrastructure.Persistence.Configurations;

public class [Entity]Configuration : IEntityTypeConfiguration<[Entity]>
{
    public void Configure(EntityTypeBuilder<[Entity]> builder)
    {
        // Table name (snake_case, số nhiều)
        builder.ToTable("[entities]");

        // Primary key
        builder.HasKey(e => e.[Entity]Id);

        // Properties
        builder.Property(e => e.Name).IsRequired().HasMaxLength(100);

        // Indexes
        builder.HasIndex(e => e.Code).IsUnique();

        // Audit columns
        builder.Property(e => e.DeletedAt).HasColumnName("deleted_at");
        builder.Property(e => e.CreatedAt).HasColumnName("created_at");
        builder.Property(e => e.UpdatedAt).HasColumnName("updated_at");
        builder.Property(e => e.CreatedBy).HasColumnName("created_by");
        builder.Property(e => e.UpdatedBy).HasColumnName("updated_by");

        // Relationships
        // builder.HasOne(e => e.RelatedEntity)...
    }
}
```

---

_Tài liệu này được cập nhật cho dự án Internship OneConnect (IOC) v2.0._
_Cập nhật lần cuối: 24/02/2026._
