# Hướng Dẫn Phát Triển Backend (Backend Developer Guide) - Internship OneConnect (IOC)

Tài liệu này hướng dẫn cách sử dụng, cấu trúc code và cách phát triển các tính năng mới cho hệ thống Backend của dự án Internship OneConnect (IOC).

---

## Mục Lục

1. [Kiến Trúc Hệ Thống](#1-kiến-trúc-hệ-thống-architecture)
2. [Công Nghệ Sử Dụng](#2-công-nghệ-sử-dụng-tech-stack)
3. [Cấu Trúc Project & Folder](#3-cấu-trúc-project--folder)
4. [Quy Trình Phát Triển Feature](#4-quy-trình-phát-triển-feature-development-workflow)
5. [Hướng Dẫn Thêm Tính Năng Mới](#5-hướng-dẫn-thêm-tính-năng-mới-step-by-step)
6. [Các Patterns Quan Trọng](#6-các-patterns-quan-trọng)
7. [C# Code Style & Conventions](#7-c-code-style--conventions)
8. [Quy Tắc Đặt Tên File & Folder](#8-quy-tắc-đặt-tên-file--folder)
9. [EF Core Migrations](#9-ef-core-migrations)
10. [Hướng Dẫn Chạy Project](#10-hướng-dẫn-chạy-project)
11. [Hệ Thống Search, Filter & Sort](#11-hệ-thống-search-filter--sort-đa-năng)
12. [Caching với Redis](#12-caching-với-redis)
13. [Quản lý Message & Đa ngôn ngữ](#13-quản-lý-message--đa-ngôn-ngữ-localization)
14. [Git Workflow & Branching Strategy](#14-git-workflow--branching-strategy)
15. [Code Review Checklist](#15-code-review-checklist)
16. [Common Mistakes & Anti-Patterns](#16-common-mistakes--anti-patterns)
17. [Cấu Trúc Mẫu (Templates)](#17-cấu-trúc-mẫu-cho-từng-loại-file)

---

## 1. Kiến Trúc Hệ Thống (Architecture)

Hệ thống được xây dựng theo kiến trúc **Clean Architecture** kết hợp với các pattern hiện đại như **CQRS** (Command Query Responsibility Segregation).

### Các Lớp Trong Hệ Thống:

| Layer | Mô tả | Phụ thuộc |
|-------|--------|-----------|
| **IOCv2.Domain** | Entities, Enums, quy tắc nghiệp vụ cốt lõi | Không phụ thuộc gì |
| **IOCv2.Application** | Logic nghiệp vụ (MediatR Handlers), DTOs, Mappings, Interfaces | Chỉ phụ thuộc Domain |
| **IOCv2.Infrastructure** | Triển khai chi tiết: EF Core, JWT, Redis Cache, Email... | Phụ thuộc Application & Domain |
| **IOCv2.API** | Controllers, Middlewares, Configurations — giao tiếp bên ngoài | Phụ thuộc tất cả layers |

> ⚠️ **Nguyên tắc quan trọng**: Application layer **KHÔNG ĐƯỢC** import trực tiếp từ Infrastructure layer. Mọi phụ thuộc phải thông qua Interface (Dependency Inversion Principle).

---

## 2. Công Nghệ Sử Dụng (Tech Stack)

| Công nghệ | Mục đích | Version |
|-----------|---------|---------|
| **C# / .NET** | Language & Framework | C# 13 / .NET 9 |
| **PostgreSQL** | Database | EF Core 9 |
| **AutoMapper** | Object-to-Object Mapping | — |
| **MediatR** | CQRS Pattern (Command/Query) | — |
| **FluentValidation** | Input Validation | — |
| **Redis** | Distributed Cache & Rate Limiting | IDistributedCache |
| **Serilog** | Structured Logging | — |
| **Swagger/OpenAPI** | API Documentation | — |
| **Docker** | Containerization | docker-compose |

---

## 3. Cấu Trúc Project & Folder

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
│   ├── BackgroundJobs/                    ← Hosted services (EmailHostedService)
│   ├── Persistence/
│   │   ├── AppDbContext.cs
│   │   ├── Configurations/               ← EF Core Fluent API configs
│   │   ├── DbInitializer.cs              ← Seed data
│   │   ├── Repositories/                 ← GenericRepository
│   │   └── UnitOfWork.cs
│   ├── Security/                          ← JWT, Password hashing
│   ├── Services/                          ← Email, Cache, RateLimiting, Logging...
│   ├── Migrations/
│   └── DependencyInjection.cs
│
├── IOCv2.API/                             ← Entry point
│   ├── Attributes/                        ← Custom attributes
│   ├── Configurations/                    ← Extension method configs (modular)
│   │   ├── ControllerConfig.cs
│   │   ├── CorsConfig.cs
│   │   ├── DatabaseConfig.cs
│   │   ├── EnvironmentConfig.cs
│   │   ├── JwtConfig.cs
│   │   ├── LocalizationConfig.cs
│   │   ├── RedisConfig.cs
│   │   └── SwaggerConfig.cs
│   ├── Controllers/
│   │   ├── ApiControllerBase.cs           ← Base controller với HandleResult<T>
│   │   ├── Admin/AdminUsersController.cs
│   │   └── Auth/AuthController.cs
│   ├── Middlewares/                       ← Exception, RateLimiting...
│   └── Program.cs                         ← Minimal hosting
│
├── docker-compose.yml
├── IOCv2.sln
└── README.md
```

---

## 4. Quy Trình Phát Triển Feature (Development Workflow)

Khi nhận một tính năng mới (User Story / Task), thành viên cần tuân thủ quy trình sau **từ đầu đến cuối**:

### 4.1. Phân Tích Yêu Cầu

1. Đọc kỹ User Story / Task trên Jira (hoặc công cụ quản lý tương ứng).
2. Xác định rõ:
   - Đây là **Query** (đọc dữ liệu) hay **Command** (thay đổi dữ liệu)?
   - Entity nào liên quan? Có cần tạo Entity mới không?
   - Có cần thêm Migration không?
   - API endpoint cần trả về response format nào?

### 4.2. Thứ Tự Viết Code (Bottom-Up)

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

### 4.3. Cấu Trúc Feature Folder

Mỗi Command/Query được đặt trong folder riêng gồm **4 file**:

```
Features/
└── [ModuleName]/              ← Admin, Authentication, Students, Universities...
    └── [EntityName]/          ← Users, Courses, Internships...
        ├── Commands/
        │   ├── Create[Entity]/
        │   │   ├── Create[Entity]Command.cs      ← Tham số đầu vào
        │   │   ├── Create[Entity]Handler.cs      ← Logic nghiệp vụ
        │   │   ├── Create[Entity]Response.cs     ← DTO trả về
        │   │   └── Create[Entity]Validator.cs    ← FluentValidation
        │   ├── Update[Entity]/
        │   ├── Delete[Entity]/
        │   └── [Action][Entity]/                 ← ToggleUserStatus, ResetUserPassword...
        └── Queries/
            ├── Get[Entity]s/                     ← Danh sách (số nhiều)
            │   ├── Get[Entity]sQuery.cs
            │   ├── Get[Entity]sHandler.cs
            │   ├── Get[Entity]sResponse.cs
            │   └── Get[Entity]sValidator.cs
            └── Get[Entity]ById/                  ← Chi tiết (số ít + ById)
```

### 4.4. Kiểm Tra Trước Khi Commit

- [ ] Build project thành công (`dotnet build`)
- [ ] Chạy thử API endpoint trên Swagger
- [ ] Kiểm tra response format đúng chuẩn
- [ ] Validation hoạt động đúng (test với dữ liệu sai)
- [ ] Đã thêm XML documentation cho Controller action

---

## 5. Hướng Dẫn Thêm Tính Năng Mới (Step-by-Step)

Giả sử bạn muốn thêm tính năng "Lấy danh sách sinh viên" (GetStudents):

### Bước 1: Tạo Response DTO

Tạo file `GetStudentsResponse.cs` trong `IOCv2.Application/Features/Students/Queries/GetStudents/`:

```csharp
using IOCv2.Application.Extensions.Mappings;
using IOCv2.Domain.Entities;

namespace IOCv2.Application.Features.Students.Queries.GetStudents;

public class GetStudentsResponse : IMapFrom<Student>
{
    public Guid Id { get; set; }
    public string FullName { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public string StudentCode { get; set; } = string.Empty;

    // Custom mapping (nếu cần)
    public void Mapping(MappingProfile profile)
    {
        profile.CreateMap<Student, GetStudentsResponse>()
            .ForMember(dest => dest.FullName, opt => opt.MapFrom(src => src.User.FullName));
    }
}
```

### Bước 2: Tạo Query & Handler

Tạo file `GetStudentsQuery.cs` cùng thư mục:

```csharp
using IOCv2.Application.Common.Models;
using MediatR;

namespace IOCv2.Application.Features.Students.Queries.GetStudents;

// Query record định nghĩa tham số đầu vào
public record GetStudentsQuery : IRequest<Result<PaginatedResult<GetStudentsResponse>>>
{
    public string? SearchTerm { get; init; }
    public string? Status { get; init; }
    public int PageNumber { get; init; } = 1;
    public int PageSize { get; init; } = 10;
    public string? SortColumn { get; init; }
    public string? SortOrder { get; init; }
}
```

Tạo file `GetStudentsHandler.cs`:

```csharp
using AutoMapper;
using AutoMapper.QueryableExtensions;
using IOCv2.Application.Common.Models;
using IOCv2.Application.Interfaces;
using IOCv2.Domain.Entities;
using MediatR;
using Microsoft.EntityFrameworkCore;

namespace IOCv2.Application.Features.Students.Queries.GetStudents;

public class GetStudentsHandler : IRequestHandler<GetStudentsQuery, Result<PaginatedResult<GetStudentsResponse>>>
{
    private readonly IUnitOfWork _unitOfWork;
    private readonly IMapper _mapper;

    public GetStudentsHandler(IUnitOfWork unitOfWork, IMapper mapper)
    {
        _unitOfWork = unitOfWork;
        _mapper = mapper;
    }

    public async Task<Result<PaginatedResult<GetStudentsResponse>>> Handle(
        GetStudentsQuery request, CancellationToken cancellationToken)
    {
        // 1. Khởi tạo query từ Repository
        var query = _unitOfWork.Repository<Student>().Query()
            .AsNoTracking();

        // 2. Áp dụng search, filter
        if (!string.IsNullOrWhiteSpace(request.SearchTerm))
        {
            var term = request.SearchTerm.Trim().ToLower();
            query = query.Where(s => s.User.FullName.ToLower().Contains(term));
        }

        // 3. Sorting
        query = (request.SortColumn?.ToLower(), request.SortOrder?.ToLower()) switch
        {
            ("name", "desc") => query.OrderByDescending(s => s.User.FullName),
            ("name", _)      => query.OrderBy(s => s.User.FullName),
            _                => query.OrderByDescending(s => s.User.CreatedAt)
        };

        // 4. Count + Pagination + Projection
        var totalCount = await query.CountAsync(cancellationToken);
        var items = await query
            .Skip((request.PageNumber - 1) * request.PageSize)
            .Take(request.PageSize)
            .ProjectTo<GetStudentsResponse>(_mapper.ConfigurationProvider)
            .ToListAsync(cancellationToken);

        var result = PaginatedResult<GetStudentsResponse>.Create(
            items, totalCount, request.PageNumber, request.PageSize);
        return Result<PaginatedResult<GetStudentsResponse>>.Success(result);
    }
}
```

### Bước 3: Tạo Validator

Tạo file `GetStudentsValidator.cs`:

```csharp
using FluentValidation;

namespace IOCv2.Application.Features.Students.Queries.GetStudents;

internal class GetStudentsValidator : AbstractValidator<GetStudentsQuery>
{
    public GetStudentsValidator()
    {
        RuleFor(x => x.PageNumber)
            .GreaterThanOrEqualTo(1);

        RuleFor(x => x.PageSize)
            .GreaterThanOrEqualTo(1)
            .LessThanOrEqualTo(100);
    }
}
```

### Bước 4: Tạo Controller

Controllers trong project kế thừa từ `ApiControllerBase` và inject `IMediator`.

```csharp
using IOCv2.Application.Features.Students.Queries.GetStudents;
using MediatR;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace IOCv2.API.Controllers;

/// <summary>
/// Quản lý sinh viên.
/// </summary>
[Tags("Students")]
[Authorize]
public class StudentsController : ApiControllerBase
{
    private readonly IMediator _mediator;
    public StudentsController(IMediator mediator) => _mediator = mediator;

    /// <summary>
    /// Get paginated list of students with optional filters and sorting.
    /// </summary>
    [HttpGet]
    [Route("students")]
    [ProducesResponseType(typeof(Result<PaginatedResult<GetStudentsResponse>>), StatusCodes.Status200OK)]
    public async Task<IActionResult> GetStudents([FromQuery] GetStudentsQuery query)
    {
        var result = await _mediator.Send(query);
        return HandleResult(result);
    }
}
```

> **Lưu ý quan trọng**:
> - Controller luôn kế thừa `ApiControllerBase` (không phải `ControllerBase`).
> - Dùng `HandleResult<T>()` để chuẩn hóa response.
> - Luôn thêm `[Tags("...")]` cho nhóm Swagger.
> - Luôn thêm `/// <summary>` XML documentation.

---

## 6. Các Patterns Quan Trọng

### 6.1. Result Pattern

Sử dụng `Result<T>` để trả về kết quả thành công hoặc lỗi từ tầng Application.

```csharp
// Thành công
Result<T>.Success(data)
Result<T>.SuccessWithWarning(data, "Cảnh báo...")

// Thất bại  
Result<T>.Failure("Thông báo lỗi", ResultErrorType.BadRequest)
Result<T>.NotFound("Không tìm thấy")
Result<T>.Failure(_messageService.GetMessage(MessageKeys.Users.NotFound), ResultErrorType.NotFound)
```

### 6.2. ApiControllerBase & HandleResult

`ApiControllerBase` đã tích hợp sẵn `HandleResult<T>()` để chuẩn hóa response:

```csharp
[ApiController]
[Route("api/[controller]")]
public abstract class ApiControllerBase : ControllerBase
{
    protected IActionResult HandleResult<T>(Result<T> result)
    {
        if (result == null) return BadRequest();

        if (result.IsSuccess)
        {
            if (result.Data == null) return NoContent();
            if (result.HasWarning)
                return Ok(new { data = result.Data, warning = result.Warning });
            return Ok(new { data = result.Data });
        }

        var statusCode = result.ErrorType switch
        {
            ResultErrorType.NotFound    => 404,
            ResultErrorType.Unauthorized => 401,
            ResultErrorType.Forbidden   => 403,
            ResultErrorType.Conflict    => 409,
            _ => 400
        };

        var response = new ErrorResponse(statusCode, result.Error ?? "An error occurred");
        return StatusCode(statusCode, response);
    }
}
```

### 6.3. Validation Behavior (MediatR Pipeline)

Mọi Command/Query gửi qua MediatR sẽ tự động được kiểm tra bởi `ValidationBehavior<TRequest, TResponse>`. Nếu có lỗi, hệ thống throw `ValidationException` → `ExceptionMiddleware` trả về 400 (Bad Request).

```
Request → ValidationBehavior → Handler → Response
              ↓ (nếu lỗi)
         ValidationException → ExceptionMiddleware → 400 Bad Request
```

### 6.4. AutoMapper (IMapFrom)

Interface `IMapFrom<T>` giúp tự động cấu hình Mapping:

```csharp
// Mapping tự động (tên property khớp)
public class StudentDto : IMapFrom<Student> { }

// Custom mapping (tên property khác nhau)
public class StudentDto : IMapFrom<Student>
{
    public string UniversityName { get; set; } = string.Empty;

    public void Mapping(MappingProfile profile)
    {
        profile.CreateMap<Student, StudentDto>()
            .ForMember(d => d.UniversityName, opt => opt.MapFrom(s => s.University.Name));
    }
}
```

### 6.5. Unit of Work & Generic Repository

| Method | Mô tả |
|--------|--------|
| `_unitOfWork.Repository<T>().Query()` | Lấy `IQueryable<T>` để đọc dữ liệu |
| `_unitOfWork.Repository<T>().AddAsync(entity, ct)` | Thêm mới entity |
| `_unitOfWork.Repository<T>().UpdateAsync(entity, ct)` | Cập nhật entity |
| `_unitOfWork.Repository<T>().DeleteAsync(entity, ct)` | Xóa entity |
| `_unitOfWork.Repository<T>().ExistsAsync(predicate, ct)` | Kiểm tra tồn tại |
| `_unitOfWork.Repository<T>().CountAsync(predicate, ct)` | Đếm số lượng |
| `_unitOfWork.SaveChangeAsync(ct)` | Lưu thay đổi xuống DB |
| `_unitOfWork.BeginTransactionAsync(ct)` | Bắt đầu transaction |
| `_unitOfWork.CommitTransactionAsync(ct)` | Commit transaction |
| `_unitOfWork.RollbackTransactionAsync(ct)` | Rollback transaction |

### 6.6. MessageKeys & IMessageService

Sử dụng hằng số `MessageKeys` + `IMessageService` thay vì hardcode string:

```csharp
// ❌ KHÔNG hardcode
return Result<T>.Failure("User not found", ResultErrorType.NotFound);

// ✅ Dùng MessageKeys
return Result<T>.Failure(
    _messageService.GetMessage(MessageKeys.Users.NotFound),
    ResultErrorType.NotFound
);
```

Các nhóm MessageKeys hiện có: `Common`, `Password`, `ResetPassword`, `Auth`, `Users`, `University`, `Enterprise`, `Profile`.

---

## 7. C# Code Style & Conventions

### 7.1. Quy Tắc Đặt Tên

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
| **Enum member** | PascalCase | `UserRole.SuperAdmin`, `UserStatus.Active` |
| **Enum with backing type** | Khai báo rõ kiểu `: short` | `public enum UserRole : short` |
| **Enum at API Boundary** | **Bắt buộc dùng `string`** | `public string Role { get; init; }` |

### 7.1.1. Quy tắc xử lý Enum (Mandatory)

Để đảm bảo khả năng mở rộng và tính tương thích cao với Frontend/Third-party:
1. **Request/Response DTO**: KHÔNG ĐƯỢC để Enum dưới dạng số (Integer). Mọi property Enum phải được khai báo là `string`.
2. **Validation**: Sử dụng `IsEnumName<TEnum>(bool caseSensitive)` của FluentValidation để kiểm tra tính hợp lệ của string input.
3. **Database**: EF Core vẫn lưu trữ Enum dưới dạng `short`/`int` để tối ưu hiệu năng. Việc chuyển đổi (Mapping) sẽ diễn ra ở tầng Application.
4. **Mapping**: 
   - Từ Request String -> Domain Enum: Dùng `Enum.Parse<TEnum>()`.
   - Từ Domain Enum -> Response String: Dùng `.ToString()`.

### 7.2. Namespace & File Structure

| Quy tắc | Ví dụ đúng | Ví dụ sai |
|---------|-----------|----------|
| Namespace phải khớp với đường dẫn folder | `IOCv2.Application.Features.Admin.Users.Commands.CreateAdminUser` | `IOCv2.Application.Admin` |
| **Ưu tiên file-scoped namespace** cho file mới | `namespace IOCv2.API.Configurations;` | — |
| Block-scoped namespace vẫn được chấp nhận | `namespace IOCv2.Application.Features... { }` | — |

### 7.3. Record vs Class

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

### 7.4. Null Safety & Default Values

```csharp
// ✅ Dùng null! cho required string properties (Entity & Command)
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

### 7.5. Async Pattern

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

### 7.6. Expression Body & Compact Syntax

```csharp
// ✅ Dùng expression body cho constructor đơn giản
public StudentsController(IMediator mediator) => _mediator = mediator;

// ✅ Dùng expression body cho computed property
public bool HasPreviousPage => PageNumber > 1;
public bool HasWarning => !string.IsNullOrEmpty(Warning);

// ✅ Dùng target-typed new
public static Result<T> Success(T data) => new(true, data, null, ResultErrorType.None);

// ✅ Dùng var khi kiểu dữ liệu đã rõ ràng bên phải
var user = _mapper.Map<User>(request);
var query = _unitOfWork.Repository<User>().Query();
```

### 7.7. Pattern Matching (Switch Expression)

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

### 7.8. Comment Style

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

## 8. Quy Tắc Đặt Tên File & Folder

### 8.1. Naming Convention Theo Layer

| Layer | Loại File | Naming Convention | Ví dụ |
|-------|----------|-------------------|-------|
| Domain | Entity | PascalCase, số ít | `User.cs`, `Student.cs` |
| Domain | Base Entity | PascalCase | `BaseEntity.cs` |
| Domain | Enum | PascalCase, mô tả rõ | `UserRole.cs`, `UserStatus.cs` |
| Application | Command | `[Action][Entity]Command.cs` | `CreateAdminUserCommand.cs` |
| Application | Query | `[Action][Entity]Query.cs` | `GetAdminUsersQuery.cs` |
| Application | Handler | `[Action][Entity]Handler.cs` | `CreateAdminUserHandler.cs` |
| Application | Validator | `[Action][Entity]Validator.cs` | `CreateAdminUserValidator.cs` |
| Application | Response DTO | `[Action][Entity]Response.cs` | `CreateAdminUserResponse.cs` |
| Application | Interface | `I[ServiceName].cs` | `IUnitOfWork.cs`, `ICacheService.cs` |
| Infrastructure | EF Config | `[Entity]Configuration.cs` | `UserConfiguration.cs` |
| Infrastructure | Service | `[ServiceName].cs` | `PasswordService.cs`, `JwtTokenService.cs` |
| API | Controller | `[Module]Controller.cs` | `AdminUsersController.cs` |
| API | Config | `[Feature]Config.cs` | `SwaggerConfig.cs`, `CorsConfig.cs` |
| API | Middleware | `[Feature]Middleware.cs` | `ExceptionMiddleware.cs` |

### 8.2. Quy Tắc Quan Trọng

- **Entity** dùng tên **số ít**: `User`, `Student`, `University` (KHÔNG phải `Users`, `Students`).
- **Query lấy danh sách** dùng tên **số nhiều**: `GetAdminUsers`, `GetStudents`.
- **Query lấy chi tiết** dùng **ById**: `GetAdminUserById`, `GetStudentById`.
- **Table name** trong EF Config dùng **snake_case, số nhiều**: `"users"`, `"students"`.
- **Audit column** mapping rõ ràng: `builder.Property(e => e.CreatedAt).HasColumnName("created_at")`.

---

## 9. EF Core Migrations

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

**Lưu ý**: Trong môi trường `Development`, ứng dụng sẽ tự động chạy migration khi khởi động (được cấu hình trong `Program.cs` → `DatabaseConfig.ApplyMigrations`).

---

## 10. Hướng Dẫn Chạy Project

### 10.1. Cấu trúc thư mục chuẩn
Để dự án chạy mượt mà giữa Backend và Frontend, cấu trúc thư mục nên được tổ chức như sau:
```text
Project_Root/
├── Internship-OneConnect_IOC_v2.0_Backend/   (Project Backend)
├── internship-oneconnect_ioc_v2.0_frontend/  (Project Frontend)
├── .env                                      (File cấu hình chung)
└── docker-compose.yml                        (File điều phối chung)
```

### 10.2. Cấu hình Environment (.env)
Bố trí file `.env` nằm tại **thư mục gốc (Project_Root)**. Đây là nơi chứa các biến môi trường dùng chung cho cả Docker và khi chạy Local (FE/BE).

### 10.3. Cách chạy dự án

#### Cách 1: Chạy toàn bộ bằng Docker (Project Setup nhanh)
Sử dụng Docker Compose để khởi chạy tất cả dịch vụ (Database, Redis, Backend, Frontend):
```bash
docker compose up -d --build
```
- API Swagger: `http://localhost:8080/swagger` (Port trong Docker)
- Frontend: `http://localhost:3000`

#### Cách 2: Chạy Backend trên Docker + Frontend Local (Phát triển Frontend)
Nếu bạn muốn dev Frontend và sử dụng Hot Reload (`npm run dev`), hãy làm theo các bước sau:

1. Mở file `docker-compose.yml` tại Project_Root.
2. **Comment (đóng)** toàn bộ block dịch vụ `frontend` (từ dòng `frontend:` đến hết cấu hình của nó).
3. Khởi chạy Backend và các hạ tầng (DB, Redis):
   ```bash
   docker compose up -d --build
   ```
4. Mở terminal mới, vào thư mục Frontend và chạy lệnh:
   ```bash
   cd internship-oneconnect_ioc_v2.0_frontend
   npm install
   npm run dev
   ```

---

---

## 11. Hệ Thống Search, Filter & Sort Đa Năng

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

## 12. Caching với Redis

Project sử dụng `ICacheService` (wrapper trên `IConnectionMultiplexer`) để tương tác với Redis.

- **Container name**: `iocv2_redis`
- **Port**: `6379`
- **Connection String**: cấu hình trong `appsettings.json`

```csharp
// Xóa cache theo pattern
await _cacheService.RemoveByPatternAsync("user:list", cancellationToken);
```

---

## 13. Quản lý Message & Đa ngôn ngữ (Localization)

Resources nằm tại `IOCv2.Application/Resources/`:

- `ErrorMessages.resx`: Chứa các key báo lỗi.
- `Messages.resx`: Chứa các key thông báo thành công.

Cách sử dụng: Inject `IStringLocalizer<ErrorMessages>` hoặc `IStringLocalizer<Messages>` để lấy chuỗi thông báo theo ngôn ngữ hiện tại (dựa vào header `Accept-Language` của request).

Khi cần thêm message mới:
1. Thêm constant vào `IOCv2.Application/Constants/MessageKeys.cs`
2. Thêm giá trị vào file `.resx` tương ứng
3. Sử dụng qua `_messageService.GetMessage(MessageKeys.Xxx.Yyy)`

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
3. **Viết code** theo thứ tự (Section 4.2).
4. **Commit thường xuyên** theo convention (Section 14.2).
5. **Push và tạo Pull Request** vào `develop`:
   ```bash
   git push origin feat/[feature-name]
   ```
6. **Review** bởi ít nhất 1 thành viên khác.
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
- [ ] Tên class/method/property đúng convention (Section 7.1)
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
- [ ] Route sử dụng format: `[Route("users")]`
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

### ❌ Sai: Inject DbContext trực tiếp vào Application Layer

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

### ❌ Sai: Hardcode error message

```csharp
return Result<T>.Failure("User not found", ResultErrorType.NotFound);
```

### ✅ Đúng: Dùng MessageKeys + IMessageService

```csharp
return Result<T>.Failure(
    _messageService.GetMessage(MessageKeys.Users.NotFound),
    ResultErrorType.NotFound
);
```

### ❌ Sai: Load toàn bộ entity rồi mới map

```csharp
// ❌ Load tất cả columns, tất cả rows → chậm
var users = await query.ToListAsync(ct);
var result = _mapper.Map<List<UserDto>>(users);
```

### ✅ Đúng: Dùng ProjectTo (query chỉ SELECT columns cần thiết)

```csharp
// ✅ EF Core chỉ SELECT đúng columns cần cho DTO
var result = await query
    .ProjectTo<UserDto>(_mapper.ConfigurationProvider)
    .ToListAsync(ct);
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
    public string PropertyName { get; init; } = null!;
    public string Status { get; init; } = null!; // Enum dạng string
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
    public string Status { get; set; } = string.Empty; // Enum dạng string

    // Custom mapping
    public void Mapping(MappingProfile profile)
    {
        profile.CreateMap<[DomainEntity], [Action][Entity]Response>()
            .ForMember(dest => dest.Name, opt => opt.MapFrom(src => src.FullName))
            .ForMember(dest => dest.Status, opt => opt.MapFrom(src => src.Status.ToString())); // Chuyển Enum sang String
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

        // Enum conversion
        builder.Property(e => e.Status)
            .HasConversion<short>()
            .HasDefaultValue(Status.Active);

        // Indexes
        builder.HasIndex(e => e.Code).IsUnique();

        // Audit columns
        builder.Property(e => e.DeletedAt).HasColumnName("deleted_at");
        builder.Property(e => e.CreatedAt).HasColumnName("created_at");
        builder.Property(e => e.UpdatedAt).HasColumnName("updated_at");
        builder.Property(e => e.CreatedBy).HasColumnName("created_by");
        builder.Property(e => e.UpdatedBy).HasColumnName("updated_by");

        // Relationships
        // builder.HasOne(e => e.RelatedEntity)
        //     .WithOne(r => r.Entity)
        //     .HasForeignKey<RelatedEntity>(r => r.EntityId);
    }
}
```

### 17.6. Controller (Template)

```csharp
using MediatR;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace IOCv2.API.Controllers.[Module];

/// <summary>
/// [Mô tả module].
/// </summary>
[Tags("[Module] - [SubModule]")]
[Authorize]
public class [Entity]Controller : ApiControllerBase
{
    private readonly IMediator _mediator;

    public [Entity]Controller(IMediator mediator) => _mediator = mediator;

    /// <summary>
    /// [Mô tả endpoint].
    /// </summary>
    [HttpGet]
    [Route("[entities]")]
    [ProducesResponseType(typeof(Result<PaginatedResult<Response>>), StatusCodes.Status200OK)]
    public async Task<IActionResult> Get[Entities]([FromQuery] Get[Entities]Query query)
    {
        var result = await _mediator.Send(query);
        return HandleResult(result);
    }
}
```

---

_Tài liệu này được cập nhật cho dự án Internship OneConnect (IOC) v2.0._
_Cập nhật lần cuối: 04/03/2026._
