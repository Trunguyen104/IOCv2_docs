# 1. Tổng quan & Bối cảnh (Overview & Context)

## 1.1 Vấn đề (Problem)
Hiện tại, hệ thống IOC cần quản lý các kỳ thực tập (Term) một cách tập trung. Nếu thiếu đi cốt lõi này, toàn bộ dữ liệu (sinh viên, doanh nghiệp, kết quả đánh giá) sẽ bị rải rác, không có ranh giới chu kỳ và không thể thống kê chính xác theo từng giai đoạn học tập.

## 1.2 Giá trị Nghiệp vụ (Business Value)
- **Tạo nền móng dữ liệu**: Kỳ thực tập (Term) là Container chứa toàn bộ hoạt động thực tập. Khi vòng đời kỳ thực tập rõ ràng (Draft $\to$ Open $\to$ Closed), các luồng nghiệp vụ khác tự động vận hành trơn tru.
- **Ràng buộc thời gian hợp lệ**: Kiểm soát chặt chẽ `start_date` và `end_date` ngăn chặn việc gán sinh viên hoặc điểm số vào sai thời điểm.

## 1.3 Đối tượng (Actor)
- **Primary Actor**: Uni Admin - Người quản lý và tạo lập các kỳ thực tập của trường học mình.
- **Secondary Actor**: Không có.

---

# 2. Luồng Người dùng (User Flow)

## 2.1 Luồng Quản lý danh sách Kỳ thực tập
1. `Uni Admin` đăng nhập tài khoản.
2. Tại menu hệ thống, truy cập "Quản lý Kỳ thực tập".
3. Giao diện gọi API hiển thị bảng dữ liệu chứa danh sách các Kỳ, kèm các bộ lọc (Filter) cơ bản và nút Add New.

## 2.2 Luồng Khởi tạo Kỳ thực tập (Create)
1. Từ trang danh sách, click "Thêm kỳ thực tập".
2. Điền Form nhập liệu: Tên kỳ (`name`), Ngày bắt đầu (`start_date`), Ngày kết thúc (`end_date`), và Trạng thái khởi tạo (VD: Draft hoặc Open).
3. Validate dữ liệu: `end_date` phải lớn hơn `start_date`.
4. Gọi API Create, hệ thống thêm mới dữ liệu và hiển thị Toast "Tạo mới thành công".

## 2.3 Luồng Cập nhật & Thay đổi Trạng thái (Update/Patch)
1. `Uni Admin` chọn 1 Kỳ thực tập đang ở trạng thái `Draft` hoặc `Open`.
2. Có thể đổi trạng thái (Từ `Draft` $\to$ `Open`, hoặc `Open` $\to$ `Closed`). Khi chuyển sang `Closed` (Đóng kỳ), sinh viên sẽ không thể Enroll được nữa.
3. Edit trực tiếp Tên và Ngày tháng (chỉ khi kỳ chưa đóng).

## 2.4 Luồng Xóa Kỳ thực tập (Delete)
1. Uni Admin chọn "Xóa" một kỳ mới tạo nhầm.
2. Hệ thống cảnh báo.
3. Nếu tiếp tục, xóa và tải lại danh sách. (Lưu ý: Không cho phép xóa nếu đã có dữ liệu ràng buộc bên trong).

---

# 3. Tiêu chí Chấp nhận (Acceptance Criteria)

## AC-01: Hiển thị giao diện danh sách
- **Given**: Quyền `UniAdmin` đã login.
- **When**: Mở trang "Quản lý Kỳ thực tập".
- **Then**:
  - Load danh sách có phân trang (Pagination).
  - Bảng hiển thị: Tên kỳ, Ngày bắt đầu, Ngày kết thúc, Trạng thái (Nháp/Mở/Đóng).
  - Có các Action: Create, Edit, State Toggle (Open/Close), Delete.

## AC-02: Thêm mới Kỳ thực tập
- **Given**: Pop-up Create đang mở.
- **When**: Nhập đầy đủ & hợp lệ các thông tin và ấn Save.
- **Then**:
  - Giao diện render thẻ mới. Bắn thông báo xanh "Thành công".
  - Backend lưu chính xác UUID của User (Trường) vào cột `university_id`.

## AC-03: Ràng buộc Validation đầu vào
- **Given**: Input đang sửa hoặc tạo.
- **When**: Nhập `end_date` nhỏ hơn `start_date`, hoặc Tên bị bỏ trống.
- **Then**:
  - API cản bước, báo lỗi `400 Bad Request` kèm `{"endDate": "Ngày kết thúc phải lớn hơn ngày bắt đầu"}`.

## AC-04: Luồng khóa/Mở (Close/Open)
- **Given**: Một Term đang `Open`.
- **When**: Chuyển trạng thái sang `Closed`.
- **Then**:
  - Giao diện update trạng thái. Bắn API Update trạng thái.
  - Term từ trạng thái này sẽ bị khóa các liên kết (Tự backend chặn ở các module khác).

## AC-05: Chống thao tác quyền chéo (Security)
- **Given**: Admin của Trường A.
- **When**: Bằng cách nào đó dùng Postman truyền UUID của Term Trường B vào API Sửa/Xóa.
- **Then**:
  - Backend đối chiếu Claim, nhả `403 Forbidden` do vi phạm "Data Ownership" (Mất quyền).

---

# 4. Đặc tả Kỹ thuật (Technical Specifications)

Dựa theo chuẩn **FFA Framework (IOCv2 Agent Skills)** và ERD **DB.md**:

## 4.1 Danh sách API Endpoints

| API Endpoint | Method | Path | Request Body / Query | Response Structure (JSON) | Status Codes & Messages |
| --- | --- | --- | --- | --- | --- |
| Lấy danh sách Term | **GET** | `/api/terms` | Query: `searchTerm`, `status` (int), `pageIndex`, `pageSize` | `{ data: [ { termId, name, startDate, endDate, status } ], totalCount }` | **200 OK**<br>**403 Forbidden** |
| Xem chi tiết Term | **GET** | `/api/terms/{termId}` | Path: `termId` (uuid) | `{ termId, name, startDate, endDate, status }` | **200 OK**<br>**404 Not Found**<br>**403 Forbidden** |
| Thêm mới Term | **POST** | `/api/terms` | `{ name, startDate, endDate, status }` | `{ termId }` | **201 Created**<br>**400 Bad Request**<br>**403 Forbidden** |
| Cập nhật Term | **PUT** | `/api/terms/{termId}` | `{ name, startDate, endDate }` | `204 No Content` | **204**<br>**400**<br>**404 Not Found** |
| Đổi trạng thái | **PATCH** | `/api/terms/{termId}/status` | `{ status: smallint }` | `204 No Content` | **204**<br>**400**<br>**404 Not Found** |
| Xóa cứng Term | **DELETE** | `/api/terms/{termId}` | Chạy thao tác DELETE Entity | `204 No Content` | **204**<br>**400 / 409 Conflict** (Có data)<br>**403 Forbidden** |

## 4.2 [⚠️ DB Mismatch] Cảnh báo Lệch chuẩn Mô hình ERD

Dựa vào cấu trúc `DB.md`, có **3 điểm sai lệch** khổng lồ cần DBA và Backend giải quyết nếu bám theo yêu cầu Issue gốc (hiện đã được tôi chỉnh lý cho chuẩn với ERD DB):
1. **Trạng thái Trôi nổi (Status Not Mapped)**: Issue gốc yêu cầu trạng thái "Upcoming, Active, Ended, Closed". Nhưng bảng `terms` trong `DB.md` chỉ có ghi chú `status smallint // 0=Draft,1=Open,2=Closed`. Hệ thống sẽ bám theo DB ERD (Draft/Open/Closed) thay vì chia vặt như requirement cũ.
2. **Thiếu cơ chế Soft-delete (IsDeleted)**: Yêu cầu cũ bắt buộc "Hành động API chỉ thực hiện soft delete: isDeleted=true, update deletedAt, deletedBy". Tuy nhiên bảng `terms` KHÔNG HỀ CÓ bất kỳ cột nào thuộc loại này (Chỉ có created_at, updated_at). Thay vì phá vỡ nguyên lý DB, tôi đã thiết kế thành hành động **Xóa cứng (Physical Delete)** nếu Term chưa có liên kết nào, và trả `400 / 409 Conflict` nếu đang có Entity reference đến nó (FK ràng buộc).
3. **Thiếu Optimistic Locking (Version)**: Yêu cầu cũ bắt kiểm tra đồng thời (Concurrency Control) thông qua `Version`. Bảng `terms` không hề có cột RowVersion. Tôi đã gạt bỏ qua ràng buộc Version này cho đồng nhất với năng lực DB.

## 4.3 [FFA-ACV] Quy tắc Validation Data

- **Validation Rules**:
  - `name`: Tên bắt buộc (`Required`), `MaxLength(100)` bám vào schema DB.
  - `start_date`, `end_date`: Cần định dạng chuẩn. Backend bắt buộc kiểm tra `end_date` > `start_date`. Nếu sai trả mã lỗi **400 Bad Request** kèm thông điệp.
- Kiểm tra tính duy nhất (Unique Title) dù DB không cấm trùng lặp để thoải mái đặt tên, nhưng có thể bổ sung check trùng nếu Domain cần. (Option).

## 4.4 [FFA-SEC] Authentication & Authorization

- **Ràng buộc Phân Quyền (RBAC)**: Gắn Attribute `[Authorize(Roles = "UniAdmin")]` vào tất cả các Endpoint nhóm `/api/terms` để Role Student hay Mentor không thể xem, sửa hoặc xóa.
- **Ownership Identity (Chống IDOR ngầm)**:
  - Khai thác trích xuất JWT Token. Handler trong kiến trúc CQRS phải bóc ra `Token.UniversityId` (Từ bảng `university_users`).
  - Toàn bộ thao tác GET List / Chi tiết / Sửa / Xóa bắt buộc móc theo điều kiện Filter ngầm: `Where(x => x.term_id == request.TermId && x.university_id == currentUser.UniversityId)`. Ném thẳng `403 Forbidden Access` nếu UUID không khớp (đừng văng 404). Việc User A xóa term trường B là bất khả thi.

## 4.5 [FFA-PERF] Cấu hình Rate Limit

- **Mutate APIs Rate Limit (POST, PUT, PATCH, DELETE)**: Tấn công Spam dữ liệu Master vô cùng nhạy cảm. Config mức `20 requests / 1 phút / IP` để bảo vệ năng lực Database Write.
- **Read APIs Rate Limit (GET)**: Giao diện danh sách (GET) rate limit nhỉnh hơn: `60 requests/phút`. 
- Endpoint lấy danh sách List GET, đi kèm Pagination `.Skip().Take()` BẮT BUỘC gắn thêm `.AsNoTracking()` trong Entity Framework Core bởi vì List danh sách chỉ nhằm show Grid Data (Read only), không tracking States.
