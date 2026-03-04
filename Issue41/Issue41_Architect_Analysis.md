# Báo Cáo Phân Tích Kiến Trúc - Issue 41: Evaluation (Student View)

**Chế độ hoạt động:** IOCv2 INTERNAL ARCHITECT MODE

## BƯỚC 1 — Phân tích Feature
- **Module:** Evaluation (Đánh giá).
- **Loại tác vụ (Command/Query):** Hoàn toàn là **Query** (Đọc dữ liệu do Sinh viên chỉ xem).
- **Phạm vi ảnh hưởng:** Giới hạn trong context đọc dữ liệu bảng `evaluations`, truy vấn chéo sang `internship_groups`. Không thay đổi state của hệ thống, không làm side-effect sang các module khác.
- **Yêu cầu kỹ thuật đặc thù:**
  - **Transaction:** Không yêu cầu transaction vì luồng thuần Read.
  - **Cache:** Có thể sử dụng caching (MemoryCache/Redis) cho danh sách `Evaluation Cycles` của kỳ, do dữ liệu này ít thay đổi. Chi tiết điểm (MyScore) chưa cần cache để tránh stale data hoặc phức tạp invalidation.
  - **Domain Event & External Integration:** Không yêu cầu.
- **Data Flow (Lớp truy vấn):**
  - Controller (API) $\to$ Query (Request DTO) $\to$ Handler (Application Logic & Security Check) $\to$ Repository (chứa `.AsNoTracking()`) $\to$ DTO/ViewModel (Đã lọc dữ liệu theo quyền) $\to$ Response.

## BƯỚC 2 — Đối chiếu kiến trúc (Architecture Validation)
- **Clean Architecture & Dependency:** 
  - Không truy vấn trực tiếp Entity Database từ Controller. Phải đóng gói kết quả vào DTO chuyên biệt trong lớp Application ở Handler trả ra. 
  - Đặc biệt đảm bảo không trả Data Entity chưa được Mapping thẳng xuống HTTP Response làm rò rỉ dữ liệu bảng.
- **Separation of Concern (Module boundary):** 
  - Logic **"che lấp điểm số của sinh viên khác"** (đổi thành `null` hoặc chuỗi asterisks) là **Business Rule** / **Authorization Logic**. Logic này **CHẮC CHẮN PHẢI** nằm trong tầng Application (Handler) hoặc Domain. TUYỆT ĐỐI không đặt logic ẩn dữ liệu tại Controller hoặc đẩy xuống Frontend (đã thiết kế trong Issue).
- **Nguy cơ sai layer:** 
  - Có nguy cơ Developer viết logic ẩn điểm tại API Controller, vi phạm **FFA-CTL** (Thin Controller).
  - Nguy cơ quên lọc Ownership ở tầng DB, thay vào đó là fetch toàn bộ rồi mới filter `if(studentId == ...)` ở RAM làm tốn bộ nhớ.

## BƯỚC 3 — Đề xuất kiến trúc triển khai
### 1. Cấu trúc Thư mục và File
Sử dụng kiến trúc CQRS với bộ thư mục trong tầng Application (`src/Core/Application/Evaluations/Queries/`):
- `GetStudentEvaluationCycles/`
  - `GetStudentEvaluationCyclesQuery.cs`
  - `GetStudentEvaluationCyclesHandler.cs` (Trả về các đợt đánh giá chung)
- `GetTeamEvaluationsOverview/`
  - `GetTeamEvaluationsOverviewQuery.cs`
  - `GetTeamEvaluationsOverviewHandler.cs` (Xử lý mapping và che điểm người khác)
- `GetMyEvaluationDetails/`
  - `GetMyEvaluationDetailsQuery.cs`
  - `GetMyEvaluationDetailsHandler.cs` (Chi tiết phiếu điểm, trả 404/403 nếu ko authz)

### 2. Transaction Boundary & Logging
- **Transaction:** Bỏ qua vì là Read-Only.
- **Logging (FFA-LOG):** Ghi log ở mức `Warning` khi bắt được hành vi Access Denied (cố gắng mở API xem chi tiết phiếu điếm nhưng Token.UserId không khớp chủ sở hữu), bao gồm các thông tin: UserId, RequestedEvaluationId.

### 3. Validation & Exception Strategy (FFA-ERR)
- **Validation (FluentValidation):** 
  - Đảm bảo `CycleId` không rỗng và đúng định dạng Guid (nếu dùng Guid).
- **Exceptions:**
  - Ném `ForbiddenException` khi phát hiện IDOR (cố tình xem detail phiếu bạn khác). Middleware sẽ bắt và mapping thành `HTTP 403 Forbidden`.
  - Ném `NotFoundException` nếu cycle không áp dụng cho cấu hình internship_term của sinh viên (mapping `HTTP 404`).

### 4. Test Strategy (FFA-TST)
**Cần viết các Unit Test thiết yếu cho Handler:**
- **[IDOR Test]:** Gợi request xem `Team Evaluations Overview` -> Verify bản ghi của sinh viên `User A` sẽ hiển thị điểm thật, còn bản ghi của bạn `User B` trong list phải trả về `TotalScore = null`.
- **[Privacy Detail Test]:** Gọi request xem chi tiết (`GetMyEvaluationDetails`) bằng tài khoản A nhưng truyền Cycle/Student mapping của tài khoản B -> Trả về `ForbiddenException` / hoặc `NotFound`.
- **[Status Rule Test]:** Trạng thái phiếu `< 3` (Pending/Draft) thì trả về điểm là null dù cho đó là điểm của chính mình. 

## BƯỚC 4 — Đánh giá rủi ro
- 🔴 **Critical Risk (Security - IDOR):** Bảo vệ API `GetMyEvaluationDetails` rất quan trọng. Mặc dù Issue nhắc đến lấy theo `cycleId` và token, nhưng Backend cần phải chắc chắn query EF là `.Where(x => x.cycle_id == req.CycleId && x.student_id == token.StudentId)`.
- 🟠 **Architectural Risk (Leaking Rules):** Business rule giấu điểm không thể nằm ở UI hay Controller. Nằm trong Handler là chỗ hợp lý nhất. Nếu dùng AutoMapper, có thể config Custom Resolver định nghĩa dựa vào chuỗi Dependency Injection (để mượn `ICurrentUserService` check chủ sở hữu lúc ánh xạ).
- 🟡 **Maintainability Risk:** Cấu trúc dữ liệu `Team Evaluations` yêu cầu liệt kê tất cả sinh viên nhóm. EF Core có thể phát sinh Select N+1 khi duyệt lấy danh sách tên người dùng (`Users` table). Bắt buộc `Include()` hoặc viết `Select` projection thẳng vào DTO mới.
- 🔵 **Performance Risk:** Màn hình Xem đánh giá có lượng fetch dữ liệu truy sâu các cấp `Evaluation` $\to$ `Evaluation Detail` $\to$ `Criteria`. Rất dễ gây phình RAM hoặc overhead SQL nếu không `.AsNoTracking()` và `.AsSplitQuery()`. Đã ghi chú trong Issue, chỉ cần bám sát.

## BƯỚC 5 — Kết luận
- **Đánh giá thiết kế Issue:** Issue 41 được vạch ra chi tiết, bao hàm đủ bảo mật Ownership, che mắt điểm số và lường trước các vấn đề EF. Hoàn toàn phù hợp để duyệt giao cho team DEV implement.
- **Đề xuất nâng cấp (Refinement):**
  - Mặc dù Document đề nghị `.Include(e => e.Details).ThenInclude(d => d.Criteria).AsSplitQuery()`, architect khuyên dùng **DTO Projection** (`.Select(e => new EvaluationDetailDto { ... })`) để EF Core build lệnh truy vấn SQL siêu tối ưu, lấy chính xác field cần (tên tiêu chí, điểm) thay vì kéo full Entity object về RAM rồi mới Map bằng tay/AutoMapper. Điều này giảm GC Pressure cực mạnh.
  - Về luồng ẩn điểm sinh viên bè bạn, Architect khuyển khích làm ở **Application Handler** vòng `foreach` sau khi lấy DTO về, thay vì phức tạp hóa ở SQL query (Dễ sinh case trả về sai logic).

**Trạng thái:** SẴN SÀNG IMPLEMENTATION (Với lưu ý đặc biệt về DTO Projection và IDOR Defense).
