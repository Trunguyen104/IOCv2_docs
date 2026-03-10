# Báo Cáo Phân Tích Kiến Trúc - Issue 67: Current Term Widget (Student Dashboard)

**Chế độ hoạt động:** IOCv2 INTERNAL ARCHITECT MODE

## BƯỚC 1 — Phân tích Feature
- **Module:** Dashboard / Student Term Information.
- **Loại tác vụ (Command/Query):** Hoàn toàn là **Query** (Truy vấn tổng hợp để hiển thị UI Widget).
- **Phạm vi ảnh hưởng:** Giới hạn trong context đọc dữ liệu. Giao diện này xuất hiện ngay trang chủ, nên tỷ lệ (hit rate) gọi hàm cực cao, ảnh hưởng trực tiếp đến UX và tổng hiệu suất server nếu Query chậm.
- **Yêu cầu kỹ thuật đặc thù:**
  - **Transaction:** Không yêu cầu (luồng Read-Only).
  - **Cache:** Rất đáng cân nhắc dùng Redis/MemoryCache (TTL khoảng vài phút hoặc cho đến khi có event thay đổi thông tin placement) do dữ liệu này gần như tĩnh xuyên suốt ngày.
  - **Domain Event & External Integration:** Không yêu cầu.
- **Data Flow:** Controller (Get API) $\to$ Query DTO $\to$ Handler (Get Student ID từ Token) $\to$ Repository (Join `Enrollments`, `Terms`, `Enterprises` kèm `.AsNoTracking()`) $\to$ Tính toán `RemainingDays` $\to$ Response DTO $\to$ 200/204.

## BƯỚC 2 — Đối chiếu kiến trúc (Architecture Validation)
- **Clean Architecture & Dependency:**
  - Đảm bảo logic tính toán (tính số ngày đếm ngược `remainingDays`) phải nằm tại tầng Application (Handler) hoặc Domain, KHÔNG để logic này rò rỉ ra Controller.
  - Document chỉ rõ không bắt Front-end gọi nhiều API để ráp lại $\to$ Backend phải cung cấp 1 DTO gọn gàng chứa tất cả thông tin Dashboard cần.
- **Separation of Concern:**
  - Logic xác định "Đâu là Term hiện tại (ACTIVE/UPCOMING)" có thể coi là một Domain Rule, hoặc đặt cụ thể trong Handler thông qua `OrderByDescending` ưu tiên status và thời gian.
  - Tránh ném lỗi `404 Not Found` nếu sinh viên này thuần túy chưa enroll. Đây là trường hợp Business hợp lệ (Empty State). Việc trả về `204 No Content` hoặc `200 Null` đúng chuẩn HTTP Semantic hơn.

## BƯỚC 3 — Đề xuất kiến trúc triển khai
### 1. Cấu trúc Thư mục và File
Triển khai CQRS, đặt trong `src/Core/Application/Students/Queries/GetDashboardCurrentTerm/`:
- `GetDashboardCurrentTermQuery.cs` (Không cần property nào vì lấy ID từ Context)
- `GetDashboardCurrentTermHandler.cs` (Handler thực hiện nối bảng, logic tính toán)
- `CurrentTermDto.cs` (Class chứa kết quả: TermName, Status, Placed, EnterpriseName, RemainingDays)

### 2. Transaction Boundary & Logging
- **Transaction:** Bỏ qua vì là Read-Only.
- **Logging (FFA-LOG):** Log lại warning nếu phát hiện bất thường như: Sinh viên tồn tại cùng lúc 2 Term `ACTIVE` (Data Inconsistency). Logging lỗi Access Denied nếu cố tình gọi bằng Role không thuộc Student.

### 3. Validation & Exception Strategy (FFA-ERR)
- **Exceptions:**
  - `ForbiddenException` lọt thành `HTTP 403 Forbidden` (Chặn quyền Role khác).
  - Không quăng `NotFoundException` khi query ra null. Handler sẽ return `null`, Controller hứng và trả ra `HTTP 204 No Content` để báo Frontend không có dữ liệu render card.

### 4. Test Strategy (FFA-TST)
**Cần viết Unit Test cho Handler (`GetDashboardCurrentTermHandler`):**
- **[Empty State Test]:** Setup Mock DB không có Enrollment nào $\to$ Verify Handler trả về null, không ném exception.
- **[Logic Countdown Test]:** Setup DB có Term ACTIVE kết thúc vào 7 ngày sau $\to$ Verify DTO trả về `RemainingDays = 7`.
- **[Logic Prioritization Test]:** Setup DB vừa có kỳ UPCOMING, vừa có kỳ ACTIVE $\to$ Verify Handler ưu tiên query trúng kỳ ACTIVE.

## BƯỚC 4 — Đánh giá rủi ro
- 🔴 **Critical Risk (Bảo Mật - Tự suy từ Token):** API thiết kế là `/api/students/me/current-term` để chống IDOR ngầm định. Handler BẮT BUỘC phải dùng `_currentUserService.UserId` để truy vấn nội tại. TUYỆT ĐỐI KHÔNG nhận path param `?studentId=...`.
- 🟠 **Architectural Risk (Server-side Timing):** Rủi ro sai lệch thời gian đếm ngược giữa Client và Server. Việc Document yêu cầu BE chủ động tính `RemainingDays` để gửi cho FE là quyết định kiến trúc chính xác. Tuy nhiên, lập trình viên cần cẩn thận mốc timezone (UTC vs GMT+7) khi tính hiệu số `Ngày kết thúc - Ngày hiện tại`.
- 🟡 **Maintainability Risk:** Tương đối thấp, luồng đơn giản.
- 🔵 **Performance Risk (Chậm do Join quá nhiều bảng ở Dashboard):** Do đây là trang chủ, mọi lần login/refresh đều nã API này. Nếu query không tốt (quên `.AsNoTracking()`, thiếu Index các cột FK, Select N+1 khi load Enterprise) sẽ kéo gục Database. Bắt buộc dùng DTO Projection (`.Select(x => new DTO {...})`). Thiết lập Rate Limiting là cần thiết.

## BƯỚC 5 — Kết luận
- **Trạng thái:** Document mô tả sắc nét luồng UX/UI Frontend và rủi ro truy vấn API. Đã sẵn sàng để phát triển (Ready for Implementation).
- **Đề xuất nâng cấp (Refinement):**
  - Củng cố nhắc nhở DEV: Tại lệnh `.Select()` DTO, logic ngày tháng có thể viết gọn bằng hiệu hai biến `DateTime`: `(term.EndDate.Date - DateTime.UtcNow.Date).Days`.
  - Nên bắt đầu áp dụng Memory Cache `[OutputCache(Duration = 60)]` nếu Framework cho phép trên Endpoint này, hoặc cache kết quả DTO trong Redis bằng Key `Dashboard_Student_{studentId}`, tự động xoá Cache khi Sinh viên Enroll kỳ mới hoặc được gán cty (Placed). Tiết kiệm tối đa chi phí Query join nhiều bảng liên tục.

**KHUYẾN NGHỊ:** ĐÃ SẴN SÀNG IMPLEMENTATION. Giữ vững Thin Controller và dùng Server-side datetime tính toán Countdown.
