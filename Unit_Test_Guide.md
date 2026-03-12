# Hướng dẫn viết Unit Test cho IOCv2

Tài liệu này hướng dẫn cách viết unit test cho IOCv2.

## Mục lục

- [Các nguyên tắc khi viết unit test](#các-nguyên-tắc-khi-viết-unit-test)
- [Cấu trúc thư mục](#cấu-trúc-thư-mục)
- [Các bước viết unit test](#các-bước-viết-unit-test)
- [Tạo test class](#tạo-test-class)
- [Setup các mock](#setup-các-mock)
- [Các mẫu thường dùng](#các-mẫu-thường-dùng)
- [Cách chạy unit test](#cách-chạy-unit-test)

## Các nguyên tắc khi viết unit test

1. Mỗi feature có 1 test class riêng
2. Mỗi test class có 1 test method riêng cho mỗi case
3. Mỗi test method có 1 test case riêng cho mỗi trường hợp
4. Mỗi test case có 1 test method riêng cho mỗi trường hợp

## Cấu trúc thư mục

```
IOCv2.Tests/
├── Features/
│   ├── FeatureName/
│   │   ├── CreateFeatureNameHandlerTests.cs
│   │   └── ...
│   └── ...
└── ...
```

## Các bước viết unit test

1. Tạo test class
2. Mock các dependencies
3. Setup các mock
4. Act
5. Assert

## Tool sử dụng

- Moq: Thư viện mock
    - Setup: Dùng để mock các dependencies
    - Returns: Dùng để mock các hàm có tham số là T
    - It.IsAny<T>(): Dùng để mock các hàm có tham số là T
- FluentAssertions: Thư viện assertion
    - Should(): Bắt đầu assertion
    - Be(): So sánh giá trị
    - BeTrue(): Kiểm tra giá trị là true
    - BeFalse(): Kiểm tra giá trị là false
    - BeNull(): Kiểm tra giá trị là null
    - BeNotNull(): Kiểm tra giá trị là not null
    - BeEmpty(): Kiểm tra giá trị là empty
    - BeNotEmpty(): Kiểm tra giá trị là not empty
    - BeEqualTo(): Kiểm tra giá trị là equal
    - BeNotEqualTo(): Kiểm tra giá trị là not equal
    - BeGreaterThan(): Kiểm tra giá trị là greater than
    - BeLessThan(): Kiểm tra giá trị là less than
    - BeGreaterThanOrEqualTo(): Kiểm tra giá trị là greater than or equal to
    - BeLessThanOrEqualTo(): Kiểm tra giá trị là less than or equal to
    - BeOfType(): Kiểm tra giá trị là type
    - BeNotOfType(): Kiểm tra giá trị là not type
    - BeIn(): Kiểm tra giá trị là in
    - BeNotIn(): Kiểm tra giá trị là not in
    - BeInAscendingOrder(): Kiểm tra giá trị là in ascending order
    - BeInDescendingOrder(): Kiểm tra giá trị là in descending order
    - BeInOrder(): Kiểm tra giá trị là in order
    - BeInRandomOrder(): Kiểm tra giá trị là in random order
    - BeInRandomOrder(): Kiểm tra giá trị là in random order
- Xunit: Framework test
Note: assertion là kiểm tra kết quả

## Tạo test class

### Khai báo các dependencies cần thiết.
Ví dụ:
```csharp
private readonly Mock<IUnitOfWork> _mockUnitOfWork;
private readonly Mock<IMessageService> _mockMessageService;
private readonly Mock<IMapper> _mockMapper;
private readonly Mock<ILogger<CreateInternshipGroupHandler>> _mockLogger;
private readonly CreateInternshipGroupHandler _handler;
```

### Khởi tạo các mock và handler
Ví dụ:
```csharp
public CreateInternshipGroupHandlerTests()
{
    _mockUnitOfWork = new Mock<IUnitOfWork>();
    _mockMessageService = new Mock<IMessageService>();
    _mockMapper = new Mock<IMapper>();
    _mockLogger = new Mock<ILogger<CreateInternshipGroupHandler>>();

    // Inject dependencies
    _handler = new CreateInternshipGroupHandler(
        _mockUnitOfWork.Object,
        _mockMessageService.Object,
        _mockMapper.Object,
        _mockLogger.Object);
}
```

### Setup các mock

Sử dụng hàm Setup để mock các dependencies.
Syntax: 
```csharp
Setup(x => x.Method()).Returns(value);
```

Espression<Func<T, bool>>: Dùng để mock các hàm có tham số là Expression<Func<T, bool>>.

It.IsAny<T>(): Dùng để mock các hàm có tham số là T.

## Các mẫu thường dùng

### Mock ExistsAsync
Hàm này dùng để kiểm tra xem một entity có tồn tại trong database hay không.
```csharp
_mockUnitOfWork.Setup(x => x.Repository<Term>().ExistsAsync(It.IsAny<Expression<Func<Term, bool>>>(), It.IsAny<CancellationToken>()))
    .ReturnsAsync(true);
```

### Mock FindAsync
Hàm này dùng để tìm kiếm một entity trong database.
```csharp
_mockUnitOfWork.Setup(x => x.Repository<Student>().FindAsync(It.IsAny<Expression<Func<Student, bool>>>(), It.IsAny<CancellationToken>()))
    .ReturnsAsync(new List<Student> { new Student { StudentId = studentId, UserId = Guid.NewGuid() } });
```

### Mock AddAsync
Hàm này dùng để thêm một entity vào database.
```csharp
_mockUnitOfWork.Setup(x => x.Repository<InternshipGroup>().AddAsync(It.IsAny<InternshipGroup>(), It.IsAny<CancellationToken>()))
    .ReturnsAsync((InternshipGroup g, CancellationToken c) => g);
```

### Mock AddRangeAsync
Hàm này dùng để thêm nhiều entity vào database.
```csharp
_mockUnitOfWork.Setup(x => x.Repository<InternshipGroup>().AddRangeAsync(It.IsAny<IEnumerable<InternshipGroup>>(), It.IsAny<CancellationToken>()))
    .Returns(Task.CompletedTask);
```

### Mock GetByIdAsync
Hàm này dùng để lấy một entity theo id.
```csharp
_mockUnitOfWork.Setup(x => x.Repository<Term>().GetByIdAsync(It.IsAny<Guid>(), It.IsAny<CancellationToken>()))
    .ReturnsAsync(new Term { Id = termId, Name = "Test Term" });
```

### Mock UpdateAsync
Hàm này dùng để cập nhật một entity trong database.
```csharp
_mockUnitOfWork.Setup(x => x.Repository<Term>().UpdateAsync(It.IsAny<Term>(), It.IsAny<CancellationToken>()))
    .Returns(Task.CompletedTask);
```

### Mock DeleteAsync
Hàm này dùng để xóa một entity trong database.
```csharp
_mockUnitOfWork.Setup(x => x.Repository<Term>().DeleteAsync(It.IsAny<Term>(), It.IsAny<CancellationToken>()))
    .Returns(Task.CompletedTask);
```

### Mock SaveChangeAsync
Hàm này dùng để lưu thay đổi vào database.
```csharp
_mockUnitOfWork.Setup(x => x.SaveChangeAsync(It.IsAny<CancellationToken>()))
    .ReturnsAsync(1);
```

### Mock Mapper
Hàm này dùng để map một entity sang một DTO.
```csharp
_mockMapper.Setup(x => x.Map<CreateInternshipGroupResponse>(It.IsAny<InternshipGroup>()))
    .Returns(new CreateInternshipGroupResponse { GroupName = "Test Group" });
```

### Mock Logger
Hàm này dùng để mock logger.
```csharp
_mockLogger.Setup(x => x.LogInformation(It.IsAny<string>()));
```

## Act

Gọi hàm cần test.
```csharp
var result = await _handler.Handle(command, CancellationToken.None);
```

## Assert

Kiểm tra kết quả.
```csharp
result.IsSuccess.Should().BeTrue();
```

## Các trường hợp cần test

1. Valid request
2. Invalid request
3. Exception

## Các lỗi thường gặp

1. Mock không đúng
    - Expression không đúng: Dùng It.IsAny<T>() để mock các hàm có tham số là Expression<Func<T, bool>>.
    - Value không đúng: Dùng Returns() để mock các hàm có tham số là T.
    - CancellationToken không đúng: Dùng CancellationToken.None để mock các hàm có tham số là CancellationToken.
2. Mapper không đúng
    - Map không đúng: Dùng It.IsAny<T>() để mock các hàm có tham số là T.
    - DTO không đúng: Dùng Returns() để mock các hàm có tham số là T.
3. Logger không đúng
    - Log không đúng: Dùng It.IsAny<string>() để mock các hàm có tham số là string.
    - Message không đúng: Dùng Returns() để mock các hàm có tham số là string.
4. Transaction không đúng: Dùng BeginTransactionAsync() để bắt đầu transaction, CommitTransactionAsync() để commit transaction, RollbackTransactionAsync() để rollback transaction.

## Cách chạy unit test

Lệnh dùng để chạy unit test:

```bash
dotnet test IOCv2.Tests/IOCv2.Tests.csproj
```

Lệnh dùng để chạy unit test với filter:

```bash
dotnet test IOCv2.Tests/IOCv2.Tests.csproj --filter "Name=TestName"
```