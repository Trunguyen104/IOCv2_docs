# 1. Tổng quan & Bối cảnh (Overview & Context)

## 1.1 Vấn đề (Problem)
Khi công ty (Enterprise) tiếp nhận sinh viên thực tập từ nhiều trường đại học khác nhau, họ cần theo dõi sát sao tiến độ và các mốc thời gian quan trọng của từng kỳ thực tập. Tuy nhiên, hiện tại doanh nghiệp gặp khó khăn do:
- Không có một bảng điều khiển (Dashboard) tập trung để xem "Kỳ thực tập đang diễn ra".
- Bỏ lỡ các mốc deadline quan trọng như: Hạn chót nộp phiếu đánh giá sinh viên (evaluation submission) hoặc chốt điểm (grading).
- Dữ liệu bị quá tải hoặc nhầm lẫn giữa các trường nếu không có sự phân tách rõ ràng.

## 1.2 Pain Points hiện tại
- Phải hỏi trực tiếp sinh viên hoặc trường để biết ngày kết thúc (`EndDate`) kỳ thực tập.
- Thường xuyên quên submit phiếu đánh giá đánh giá giữa kỳ/cuối kỳ vì không có cảnh báo deadline.
- Tra cứu thông tin rườm rà, UX không tối ưu cho người dùng doanh nghiệp (HR/Mentor).

## 1.3 Giá trị Nghiệp vụ (Business Value)
- **Tăng tính chủ động cho Doanh nghiệp**: Enterprise dễ dàng theo dõi thời gian thực tập của sinh viên.
- **Tuân thủ Deadline**: Đảm bảo các mốc đánh giá và báo cáo điểm không bị miss sót, giúp tiến độ của Uni Admin suôn sẻ.
- **Cải thiện Trải nghiệm (UX)**: Giao diện trực quan với Timeline và Countdown, cùng bộ lọc thông minh khi công ty có sinh viên từ nhiều trường khác nhau.

## 1.4 Đối tượng (Actor)
- **Primary Actor**:
  - **Enterprise HR**: Xem tổng quan tất cả các kỳ đang Active của toàn công ty.
  - **Mentor**: Xem các kỳ Active liên quan trực tiếp đến sinh viên do mình hướng dẫn.
- **Secondary Actor**: Không có.

---

# 2. Luồng Người dùng (User Flow)

## 2.1 Luồng Hiển Thị Dashboard Cơ Bản
1. User (Enterprise HR hoặc Mentor) đăng nhập thành công.
2. Hệ thống chuyển hướng vào trang chủ (Dashboard) hoặc User click vào menu "Kỳ thực tập".
3. Giao diện gọi API lấy danh sách các Term đang `ACTIVE` liên quan đến doanh nghiệp đó.
4. Màn hình render các Card hiển thị thông tin Term: Tên kỳ, Tên trường, Timeline (Start -> End) và Bộ đếm ngược số ngày.

## 2.2 Luồng Nhắc Nhở Deadline Đánh Giá
1. Bên trong từng Card Term hiển thị, hệ thống liệt kê mục "Deadlines".
2. Nếu hạn chót (Evaluation/Grading) còn khoảng `<= 7 ngày`, thời gian đếm ngược chuyển sang viền màu Cảnh báo (Đỏ/Cam).
3. Người dùng dễ dàng nắm bắt sự gấp rút và ấn vào Action đi tới màn hình "Đánh giá sinh viên".

## 2.3 Luồng Tương Tác Khi Không Có Term Active
1. Nếu kỳ thực tập mới kết thúc (`ENDED`/`CLOSED`) hoặc công ty chưa tiếp nhận sinh viên nào.
2. Dashboard tự động render giao diện rỗng (Empty State) với text: *"Hiện chưa có kỳ thực tập nào đang diễn ra"*.

---

# 3. Tiêu chí Chấp nhận (Acceptance Criteria)

## AC-01: Truy cập thông tin "Kỳ thực tập đang diễn ra"
- **Given**: Enterprise HR / Mentor đăng nhập thành công.
- **When**: Truy cập Dashboard hoặc Menu "Kỳ thực tập".
- **Then**: 
  - Hệ thống hiển thị các Term đang mang trạng thái `ACTIVE` của các trường mà doanh nghiệp đang có sinh viên thực tập.
  - Giao diện dạng Card hoặc List chia rõ Tên trường đại học và Tên kỳ thực tập.

## AC-02: Hiển thị Timeline & Bộ đếm ngược
- **Given**: Dữ liệu tải về thành công.
- **When**: Render UI của 1 Term đang Active.
- **Then**: 
  - Hiển thị ngày: Mốc Start và Mốc End.
  - Có thanh Timeline (Progress bar) chỉ định vị trí thời gian của mốc "Hôm nay".
  - Chạy text Countdown: *"Còn [X] ngày đến khi kết thúc kỳ thực tập"*.

## AC-03: Hiển thị Deadlines đánh giá/chấm điểm
- **Given**: Uni Admin có thiết lập cấu hình Timeline mốc deadline đánh giá của Term.
- **When**: Enterprise xem mục "Deadlines" trên Card.
- **Then**:
  - Hiển thị hạn nộp đánh giá giữa kỳ/cuối kỳ (Evaluation submission deadline).
  - Có hệ thống Countdown riêng (ví dụ: *"Hạn Đánh giá cuối kỳ: Còn 5 ngày"*).
  - Đổi màu cảnh báo Đỏ/Vàng nếu deadline `<=` 7 ngày.
  - Nếu Uni Admin chưa set deadline: Render dòng chữ *"Chưa có thông tin deadline cho kỳ thực tập này"*.

## AC-04: Xử lý nhiều kỳ Active (Nhiều trường kết nối)
- **Given**: Enterprise đang có SV thực tập từ 3 trường đại học khác nhau.
- **When**: Danh sách Term Active có nhiều Card khác nhau.
- **Then**: 
  - Hệ thống cung cấp Dropdown Bộ Lọc (Filter) theo "Trường (University)".
  - Việc lọc phải phản hồi tức thời.

## AC-05: Xử lý Empty State
- **Given**: Hiện hành không có SV đang thực tập (hoặc mọi kỳ đã chuyển `CLOSED`/`ENDED`).
- **When**: Render logic danh sách.
- **Then**: 
  - Hiển thị Empty State rõ ràng: *"Hiện chưa có kỳ thực tập nào đang diễn ra"*. 
  - Tuyệt đối không render thông tin trắng dở dang hoặc lỗi UI.

## AC-06: Bảo mật Data Access Control (RBAC)
- **Given**: User thuộc Enterprise A (ID = 100).
- **When**: API Fetch dữ liệu hoặc cố gắng giả mạo URL để lấy thông tin Term của Enterprise khác.
- **Then**: API chặn Request, trả về mã HTTP `403 Forbidden` do thiếu Permission & Ownership.

---

# 4. Đặc tả Kỹ thuật (Technical Specifications)

Dựa theo chuẩn **FFA Framework (IOCv2 Agent Skills)** và đối chiếu **DB.md**:

## 4.1. Thiết kế API Endpoints (Bắt buộc)

| API Endpoint | Method | Path | Request Variables | Response Structure (JSON) | Status Codes & Messages |
| --- | --- | --- | --- | --- | --- |
| Lấy danh sách Term Active của Enterprise | **GET** | `/api/enterprises/me/active-terms` | `universityId` (uuid, optional, map với `universities.university_id`) | `{ data: [ { termId, universityId, universityName, termName, startDate, endDate, termRemainingDays, deadlines: [ { cycleId, cycleName, dueDate, remainingDays } ] } ], totalCount }` | **200 OK**: Thành công.<br>**401 Unauthorized**: Lỗi xác thực.<br>**403 Forbidden**: Bạn không có quyền thực hiện hành động này.<br>**429 Too Many Requests**: Vi phạm Rate Limit. |

## 4.2. [DB Alignment] Đối chiếu Database
- **Mapping Dữ liệu**:
  - `termId` $\to$ `terms.term_id` (uuid)
  - `termName` $\to$ `terms.name` (varchar 100)
  - `startDate`, `endDate` $\to$ `terms.start_date`, `terms.end_date` (date)
  - `universityId` $\to$ `universities.university_id` (uuid)
  - `universityName` $\to$ `universities.name` (varchar 255)
  - `deadlines.cycleId` $\to$ `evaluation_cycles.cycle_id` (uuid)
  - `deadlines.cycleName` $\to$ `evaluation_cycles.name` (varchar 255)
  - `deadlines.dueDate` $\to$ `evaluation_cycles.end_date` (timestamptz)
- **Logic Quan hệ (Relationship)**: 
  - Hoàn toàn khả thi, không có `[⚠️ DB Mismatch]`. 
  - Câu query Join từ bảng `internship_groups` -> `terms` -> `universities`. Đồng thời Left Join với `evaluation_cycles` nơi `term_id = terms.term_id`.
  - Condition: `terms.status = 1` (1 = Open/Active).

## 4.3. [FFA-ACV] & Validation Rules
- **Request Query Parameter Validation**:
  - `universityId`: Không được bắt buộc (Optional), nhưng nếu truyền vào phải là định dạng Guid chuẩn (`^[0-9a-fA-F]{8}-([0-9a-fA-F]{4}-){3}[0-9a-fA-F]{12}$`). Ném `400 Bad Request` nếu truyền sai format.
- Truy vấn trả về mảng rỗng `{"data": [], "totalCount": 0}` nếu không có dữ liệu để Frontend tự hiển thị Empty State, thay vì trả `404 Not Found`.

## 4.4. [FFA-SEC] Authentication & Authorization
- **Ràng buộc Quyền hạn (RBAC)**:
  - Chỉ Roles `EnterpriseHR` / `EnterpriseAdmin` và `Mentor` mới được quyền truy cập (Dùng Attribute `[Authorize(...)]`).
- **Phân luồng logic Ownership**:
  - `Enterprise HR`: Lấy tất cả Group nơi `internship_groups.enterprise_id == Token.EnterpriseId`.
  - `Mentor`: Chỉ lấy các Group nơi `internship_groups.mentor_id == Token.UserId` (Id của `enterprise_users`). Không cho Mentor xem các Term chỉ liên kết với Mentor khác.
  - Cấm sử dụng Parameter truyền `enterprise_id` từ máy khách lên để query tránh Insecure Direct Object Reference (IDOR).

## 4.5. [FFA-PERF] Cấu hình Rate Limit & Database
- **Rate Limit (IP-based)**:
  - URL Query Dashboard tải danh sách Term là nơi User lưu tới lấy lui cực kỳ thường xuyên mỗi khi vào web.
  - Rate Limit cấu hình mức: `60 requests / 1 phút / IP`. Quá giới hạn chốt Block `429 Too Many Requests`.
- **Database Optimizations (EF Core)**:
  - Tất cả tác vụ List DTO phải được gắn hàm `.AsNoTracking()` giúp bộ nhớ Entity Framework Core không tracking Object trạng thái.
  - Logic tính ngày `termRemainingDays` và `deadlines.remainingDays` BẮT BUỘC được Execute ngầm toán học DateDiff(Now, *Date*) bằng C# tại Server (theo Timezone GMT+7 chuẩn của hệ thống), thay vì xuất Date trơn gửi xuôi cho Frontend tự tính toán. Hệ số Client PC Time sai lẹnh vô cùng lớn.