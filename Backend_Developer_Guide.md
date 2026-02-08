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
   Mở trình duyệt và truy cập: `http://localhost:5000/swagger` (hoặc port 8080 tùy cấu hình).
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

_Tài liệu này được cập nhật cho dự án Internship OneConnect (IOC) v2.0._
