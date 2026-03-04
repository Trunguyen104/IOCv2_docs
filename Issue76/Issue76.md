# 1. Tổng quan & Bối cảnh (Overview & Context)

## 1.1 Vấn đề (Problem)
Sinh viên trong quá trình học tập tại trường có thể tham gia nhiều kỳ thực tập khác nhau (hoặc bị rớt, rút khỏi kỳ và tham gia lại). Hiện tại, hệ thống thiếu một giao diện tập trung để sinh viên có thể tự theo dõi lịch sử tham gia các kỳ thực tập của chính mình, đặc biệt là theo dõi trạng thái xem bản thân đã được Nhà trường sắp xếp (Placed) vào Doanh nghiệp nào hay chưa. Việc không có thông tin minh bạch khiến sinh viên hoang mang và phải liên hệ trực tiếp với nhà trường để xác nhận.

## 1.2 Giá trị Nghiệp vụ (Business Value)
- **Tăng tính minh bạch và chủ động**: Sinh viên được tự quyền tra cứu lịch sử và trạng thái thực tập một cách rõ ràng.
- **Giảm tải cho Uni Admin**: Giảm số lượng câu hỏi/email từ sinh viên hỏi về việc "Em đã được phân công thực tập chưa?". 
- **Lưu trữ lịch sử học tập**: Cho phép sinh viên tra cứu lại thông tin các kỳ thực tập cũ (`Withdrawn` hoặc `Completed`).

## 1.3 Đối tượng (Actor)
- **Primary Actor**: Sinh viên (Student) - Người trực tiếp xem lịch sử và trạng thái Placement của mình.
- **Secondary Actor**: Không có. Dữ liệu mang tính cá nhân hóa 100%.

---

# 2. Luồng Người dùng (User Flow)

## 2.1 Luồng Khám Phá Từ Dashboard
1. Sinh viên (Student) đăng nhập thành công vào hệ thống.
2. Trên màn hình Dashboard, hiển thị một Widget nhỏ tóm tắt về Kỳ thực tập hiện tại (Đã được cover chi tiết ở Issue67).
3. Sinh viên nhấn nút "Xem tất cả" hoặc click menu "Kỳ thực tập của tôi" ở Sidebar để truy cập Danh sách.

## 2.2 Luồng Xem Danh Sách Kỳ Thực Tập (My Terms)
1. Giao diện trang "Kỳ thực tập của tôi" gọi API lấy toàn bộ lịch sử các Term mà sinh viên này từng ghi danh (`student_terms`).
2. Màn hình liệt kê các Card/Row với thông tin: Tên kỳ (`terms.name`), Ngày ghi danh, Trạng thái Ghi danh (Active/Withdrawn), và Trạng thái Placement (Placed/Unplaced kèm Tên Enterprise nếu có).
3. Kỳ thực tập nào đang có trạng thái `terms.status = 1` (Open/Active) được ghim huy hiệu "Hiện tại" (Badge).

## 2.3 Luồng Xem Chi Tiết
1. Sinh viên click vào một Term cụ thể trong danh sách.
2. Hệ thống gọi API Detail. Màn hình trượt ra (hoặc chuyển trang) hiển thị thông tin cụ thể của Kỳ đó:
   - Nếu `Unplaced`: Cảnh báo màu đỏ/xám "Chưa được xếp doanh nghiệp" và khuyến nghị sinh viên nộp CV.
   - Nếu `Placed`: Badge xanh lá "Đã được xếp doanh nghiệp", kèm tên Enterprise.

---

# 3. Tiêu chí Chấp nhận (Acceptance Criteria)

## AC-01: Hiển thị danh sách tất cả các kỳ (Internship History)
- **Given**: Student truy cập trang "Kỳ thực tập của tôi".
- **When**: Dữ liệu tải thành công.
- **Then**:
  - Giao diện liệt kê tất cả các kỳ Student đã từng tham gia (bao gồm cả các kỳ đã cũ hoặc bị rớt).
  - Sắp xếp mặc định theo `Ngày enroll mới nhất` (Date Descending).
  - Thông tin hiển thị cho 1 Item: Tên kỳ, Ngày enroll, Trạng thái Enrollment, Trạng thái Placement, Tên Enterprise (nếu Placed).
  - Có Badge "Hiện tại" cho kỳ thực tập mang trạng thái Open.

## AC-02: Phản hồi khi Sinh viên chưa từng tham gia kỳ nào
- **Given**: Sinh viên chưa bao giờ được đưa vào hệ thống OJT của trường nào.
- **When**: Truy cập "Kỳ thực tập của tôi".
- **Then**: 
  - Giao diện hiển thị Empty State dạng đồ họa thân thiện: *"Bạn chưa tham gia kỳ thực tập nào"*.

## AC-03: Xem chi tiết 1 kỳ thực tập (View Details)
- **Given**: Student đang ở trang "Kỳ thực tập của tôi".
- **When**: Click xem chi tiết 1 kỳ cụ thể.
- **Then**:
  - Thông tin bung ra chứa đủ: Tên kỳ, Ngày enroll, Khối Trạng thái Ghi danh (Active / Withdrawn) và Khối Trạng thái Placement.
  - Xử lý **Unplaced**: Badge "Chưa được xếp doanh nghiệp" bật lên, trường Enterprise bị vô hiệu hóa hoặc ghi "Chưa có".
  - Xử lý **Placed**: Badge "Đã xếp doanh nghiệp" mầu xanh, hiển thị nổi bật Tên Enterprise đã nhận.

## AC-04: Đẩy Noti (Notification) khi trạng thái thay đổi
- **Given**: Trạng thái của sinh viên đang nằm nguyên.
- **When**: 
  - (A) Uni Admin xếp sinh viên vào 1 Enterprise mới (Chuyển Unplaced $\to$ Placed).
  - (B) Uni Admin Rút sinh viên khỏi đợt thực tập (Đổi status Enrollment thành Withdrawn).
- **Then**:
  - Hệ thống tự động PUSH Notification hiển thị trong app (Bell Icon) gửi trực tiếp tời Student thông báo sự thay đổi đó. Sinh viên ấn vào Noti để xem chi tiết.
  - Đồng bộ đổi trạng thái Placement trên Danh sách nếu sinh viên Load lại trang.

## AC-05: Bảo mật dữ liệu Cá nhân hóa tuyệt đối
- **Given**: Mọi người dùng trên hệ thống.
- **When**: Cố tình chọc vào URI xem thông tin các kỳ thực tập này.
- **Then**:
  - Chỉ đúng User mang Role `Student` và là chủ nhân của `student_terms` đó mới được truy cập dữ liệu của mình. 
  - Sinh viên khác, Enterprise hay UniAdmin gọi sẽ nhận HTTP Error `403 Forbidden` do vi phạm Data Ownership (trừ luồng Dashboard riêng của UniAdmin).

---

# 4. Đặc tả Kỹ thuật (Technical Specifications)

Dựa theo chuẩn **FFA Framework (IOCv2 Agent Skills)** và đối chiếu **DB.md**:

## 4.1. Thiết kế API Endpoints (Bắt buộc)

| API Endpoint | Method | Path | Request Variables | Response Structure (JSON) | Status Codes & Messages |
| --- | --- | --- | --- | --- | --- |
| Lấy lịch sử các kỳ của SV | **GET** | `/api/students/me/terms` | *Header*: Token (Bearer) | `{ data: [ { termId, termName, termStatus, enrolledAt, enrollmentStatus, placementStatus, enterpriseName } ] }` | **200 OK**: Thành công.<br>**401 Unauthorized**: Token lỗi.<br>**403 Forbidden**: Role sai.<br>**429 Too Many Requests**. |
| Xem chi tiết 1 kỳ của SV | **GET** | `/api/students/me/terms/{termId}` | `termId` (uuid) từ URL Path | `{ termId, termName, termStatus, enrolledAt, enrollmentStatus, placementStatus, enterpriseId, enterpriseName }` | **200 OK**: Thành công.<br>**403 Forbidden**: Trái quyền lấy data người khác.<br>**404 Not Found**: Lỗi ID.<br>**429 Blocked**. |

## 4.2. [DB Alignment] Đối chiếu Database
- **Mapping Dữ liệu**:
  - `termId` $\to$ `student_terms.term_id` (uuid)
  - `termName`, `termStatus` $\to$ `terms.name`, `terms.status`
  - `enrolledAt` $\to$ `student_terms.created_at` (timestamptz)
  - `enrollmentStatus` $\to$ `student_terms.status` (smallint)
  - `placementStatus`: Logic tính bằng cách Left Join bảng `internship_groups` qua bảng trung gian `internship_students` (có chứa `student_id`). Nếu ra NULL Enterprise -> Unplaced. Nếu có -> Placed.
  - `enterpriseName`, `enterpriseId` $\to$ `enterprises.name` và `enterprises.enterprise_id`.
- **Logic Quan hệ (Relationship)**: 
  - Toàn bộ flow đều khả thi, không có `[⚠️ DB Mismatch]`. 
  - Mối nối truy vấn: Bắt đầu từ Query `student_terms` `WHERE student_id = @Me`. Inner join sang `terms`. Left Join với `internship_students` & `internship_groups` (Tìm condition `internship_groups.term_id == terms.term_id` & `internship_students.student_id == student_terms.student_id`) và trích sang `enterprises`.

## 4.3. [FFA-ACV] Valiation Rules
- **Quy tắc Request Parameter**:
  - `termId` (UUID) trên URL Path của API Detail giới hạn buộc Regex chuẩn Guid (`^[0-9a-fA-F]{8}-([0-9a-fA-F]{4}-){3}[0-9a-fA-F]{12}$`). Chặn từ Middleware trả về `400 Bad Request` dạng `"termId": "Không đúng định dạng UUID"`.
- Response của API GET List bắt buộc trả Array rỗng `{"data": []}` khi Sinh viên không tham gia kỳ nào (để Frontend xử lý render AC-02 dễ dàng). Cấm văng Error 404 cho thao tác List bình thường.

## 4.4. [FFA-SEC] Authentication & Authorization
- **Ràng buộc Quyền hạn (RBAC)**:
  - Chỉ Roles `Student` được đi vào URL dạng `/api/students/me/...` nàỵ (Bảo vệ bởi Attribute `[Authorize(Roles = "Student")]`).
- **Phân luồng logic Ownership (Chống IDOR ngầm)**:
  - Tất cả các tác vụ Data Access trong Handler lấy ID mục tiêu duy nhất từ Claim Token (`User.UserId` trỏ xuống SQL tra ra `student_id` tương ứng tại bảng `students`).
  - Ở API Detail `/{termId}`, kiểm tra kỹ xem record `student_terms` có thực sự khớp `student_id == Me` không. Ngăn triệt sinh viên đổi UUID của Term khác mà trường không phân.

## 4.5. [FFA-PERF] Cấu hình Rate Limit & Database
- **Rate Limit (IP-based)**:
  - Phân vùng cấu hình: Cho phép Request GET tối đa **60 requests / 1 phút / IP**. Ngăn bot cào trộm API Profile. 
- **Database Optimizations (EF Core)**:
  - Logic tính toán Left Join sang cụm Liên kết `internship_groups` và `enterprises` để truy tìm `Placement Status` có thể tốn Query Cost. Bắt buộc dùng `.AsNoTracking()` trong Entity Framework nhằm giảm cấp phát Memory vô ích. Query Select trực diện các Field thay vì lấy toàn bộ Object Domain sang RAM.
  - Sắp xếp (Ordering): Order trực tiếp `ORDER BY student_terms.created_at DESC` trên Database Engine (SQL Server), TUYỆT ĐỐI không tải List về LINQ rồi mới xài `.OrderByDescending()` để tránh Overload Memory lúc số record lớn.