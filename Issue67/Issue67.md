# 1. Tổng quan & Bối cảnh (Overview & Context)

## 1.1 Vấn đề (Problem)
Khi sinh viên đăng nhập vào hệ thống IOC, họ cần ngay lập tức nắm bắt được thông tin về kỳ thực tập hiện tại hoặc sắp tới của mình trực tiếp trên Dashboard (Trang chủ). Việc thiếu một Widget/Section thông tin tóm tắt khiến sinh viên phải tốn nhiều thao tác click chuột để xem trạng thái placement và thời gian còn lại của kỳ thực tập, dẫn đến trải nghiệm người dùng (UX) kém và dễ bỏ lỡ các thông tin quan trọng như deadline.

## 1.2 Pain Points hiện tại
- Thiếu màn hình tổng quan ngay trên Dashboard để bao quát thông tin các kỳ thực tập mà Sinh viên đang tham gia.
- Khó khăn trong việc theo dõi liên tục tiến độ thời gian (Countdown countdown days) của kỳ thực tập.
- Sinh viên không có call-to-action (lời kêu gọi hành động) rõ ràng nếu đang ở trạng thái Unplaced.

## 1.3 Giá trị Nghiệp vụ (Business Value)
- **Tăng trải nghiệm UX**: Sinh viên nắm bắt ngay lập tức trạng thái thực tập (Placed/Unplaced) và thông tin cơ bản của Term ở ngay trang đầu tiên.
- **Nhắc nhở tự động**: Hiển thị bộ đếm ngược thời gian giúp sinh viên nhận thức được các mốc thời gian quan trọng (Deadline apply CV, báo cáo).
- **Điều hướng thông minh**: Điều hướng những sinh viên chưa có công ty (Unplaced) thẳng tới kho tin tuyển dụng (Job Postings).

## 1.4 Đối tượng (Actor)
- **Primary Actor**: Sinh viên (Student) - Xem thông tin cá nhân hóa trên trang Dashboard.
- **Secondary Actor**: Không

---

# 2. Luồng Người dùng (User Flow)

1. Sinh viên đăng nhập vào hệ thống thành công với role `Student`.
2. Hệ thống tự động chuyển hướng Sinh viên đến trang chủ `Dashboard`.
3. Khi Dashboard render, FE gọi API fetch thông tin "Kỳ thực tập hiện tại (My Current Term)".
4. Dựa vào response từ hệ thống:
   - **TH1: Có kỳ thực tập (Active/Upcoming)**: Hiển thị Card tóm tắt chứa Tên kỳ, Thời gian, Countdown timer và Trạng thái Placed/Unplaced.
   - **TH2: Không có kỳ thực tập**: Hiển thị Empty State thông báo *"Bạn hiện không có kỳ thực tập nào đang diễn ra"*.
5. Sinh viên có thể click "Xem chi tiết" trên Card để đi sâu vào trang danh sách các kỳ. Nếu sinh viên "Unplaced", hiển thị thêm nút "Tìm công ty thực tập ngay".

---

# 3. Tiêu chí Chấp nhận (Acceptance Criteria)

## AC-01: Hiển thị thông tin kỳ thực tập hiện tại (ACTIVE)
- **Given**: Student đăng nhập thành công và đã Enroll vào 1 kỳ (Status ACTIVE).
- **When**: Truy cập trang Dashboard/Home.
- **Then**: 
  - Hiển thị Widget với: Tên kỳ, Ngày BĐ - Ngày KT, Trạng thái (ACTIVE), Trạng thái Placement (Placed/Unplaced).
  - Tên doanh nghiệp (Nếu đã Placed).

## AC-02: Hiển thị thông tin kỳ thực tập sắp bắt đầu (UPCOMING)
- **Given**: Student Enroll vào 1 kỳ (Status UPCOMING).
- **When**: Truy cập trang Dashboard/Home.
- **Then**: 
  - Hiển thị Trạng thái: *"Sắp bắt đầu"*.
  - Trạng thái Placement: Luôn là Unplaced hiển thị mờ.

## AC-03: Bộ đếm ngược (Countdown timer)
- **Given**: Data Term đã được tải về.
- **When**: Render trên Frontend.
- **Then**:
  - Nếu `ACTIVE`: Tính số ngày từ *Ngày hiện tại -> Ngày kết thúc*. Format: *"Còn [X] ngày đến khi kết thúc"*.
  - Nếu `UPCOMING`: Tính số ngày từ *Ngày hiện tại -> Ngày bắt đầu*. Format *"Còn [X] ngày đến khi bắt đầu"*.
  - UI đổi màu Vàng/Đỏ nếu số ngày `<=` 7 ngày.

## AC-04: Xử lý State Không có Kỳ thực tập
- **Given**: Student không Enroll vào kỳ nào, hoặc tất cả các kỳ đều đã `ENDED`/`CLOSED`.
- **When**: Truy cập Dashboard.
- **Then**:
  - Không render Widget đếm ngược.
  - Render Card rỗng kèm text: *"Bạn hiện không có kỳ thực tập nào đang diễn ra."*

## AC-05: Call to Action cho Unplaced Student
- **Given**: Sinh viên đang có kỳ thực tập `ACTIVE`.
- **When**: Placement đang tính là `Unplaced`.
- **Then**:
  - Hiển thị Text màu cảnh báo (Đỏ/Cam): *"Chưa có công ty"*. Hỗ trợ Subtext: *"Hãy apply vào các vị trí tuyển dụng"*.
  - Nút bấm action mở trang *"Danh sách Job Postings"*.

## AC-06: Bảo mật Data Access Control (RBAC)
- **Given**: Các Role khác `Student` (Ví dụ UniAdmin / Enterprise HR).
- **When**: Cố tình chọc API Get Current Term của Student.
- **Then**: Trả về `403 Forbidden`.

---

# 4. Đặc tả Kỹ thuật (Technical Specifications)

Dựa theo chuẩn **FFA Framework (IOCv2 Agent Skills)**:

## 4.1. Thiết kế API Endpoints (Bắt buộc)

| API Endpoint | Method | Path | Request Variables | Response Structure (Thành công 200/201) | Mã lỗi dự kiến |
| --- | --- | --- | --- | --- | --- |
| Lấy thông tin Kỳ thực tập hiện tại của Student trên Dashboard | **GET** | `/api/students/me/current-term` | *Header*: Token (Bearer) | `{ termId, termName, startDate, endDate, status, placementStatus, enterpriseName, remainingDays }` | **204 No Content** (Không có kỳ thích hợp)<br>**403 Forbidden** |

*[⚠️ Architecture Violation]: Tránh thiết kế API Frontend phải query nhiều bảng. BE nên xây dựng một Query Handler (Dapper hoặc View) Map thẳng ra DTO để Frontend không cần ghép logic.*

## 4.2. [FFA-ACV] & [FFA-ERR] Validation & Error Handling
- **Logic Xác định "Current Term"**: 
  - Trong DB, một sinh viên có thể tham gia nhiều `Enrollment` thuộc nhiều `Term` ở các lịch sử quá khứ.
  - Tuy nhiên, API Getter này CHỈ Query ra 1 bản ghi thỏa mãn điệu kiện: Sinh viên đó Enroll vào Term mang Status = `ACTIVE` hoặc `UPCOMING`. Ưu tiên lấy `ACTIVE` nếu trùng lặp (Mặc dù Logical không cho phép trùng 2 term Active cùng lúc).
  - Nếu SQL trả về NULL -> Handler không được văng Exception 404. Hãy trả về Response HTTP `204 No Content` hoặc HTTP `200` mang JSON `null` để Front-end dễ dàng map ra Empty State. Tránh lạm dụng HTTP `404 Not Found` cho Business Logic trống.

## 4.3. [FFA-SEC] Authentication & Authorization
- **Ràng buộc Quyền hạn**:
  - Đặt Controller Tag `[Authorize(Roles = "Student")]`.
  - Phân tích User Claim Token trích xuất Identity `UserId` (tương ứng với StudentId). 
  - Handler fetch bằng JWT Context chứ KHÔNG nhận tham số ID sinh viên từ Parameter. Cấm lộ lọt ID lên URl như kiểu `/api/students/{id}/current-term` để chống Insecure Direct Object Reference (IDOR).

## 4.4. Cấu hình Rate Limit & Database Perf [FFA-PERF]
- **Read APIs Rate Limit**:
  - Do API nằm ở Homepage (Dashboard), tỷ lệ truy cập Refresh rất cao. Cần Set Rate Limit: `60 requests / 1 phút / IP`.
- **Database Optimizations (No Tracking)**:
  - Do API là Query dạng Read-Only, tuyệt đối gắn `.AsNoTracking()` trong Entity Framework Core khi join bảng `Enrollment`, `Term`, `Enterprise` để tiết kiệm RAM. Lấy chính xác những field DTO cần.
- **Tính toán Countdown `remainingDays`**:
  - Bắt buộc tính Total Days bằng Datetime logic từ phía Server (BE), sau đó map vào DTO Response. Cấm không cho BE đổ Local Time trực tiếp bắt Front-end phải tính (Vì tránh sai số múi giờ của Client PC của Sinh Viên). Server căn cứ theo GMT+7 của Server để báo hiệu `RemainingDays`.