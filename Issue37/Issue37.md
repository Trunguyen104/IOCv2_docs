## 1. Tổng quan & Bối cảnh (Overview & Context)

## Vấn đề (Problem):

Trong một dự án thực tập, ngoài Mentor và các thành viên trong nhóm, sinh viên thường phải tương tác với nhiều bên liên quan khác (Stakeholders) như: Khách hàng (Client), Người dùng cuối (End-user), Chuyên gia tư vấn, hoặc các phòng ban hỗ trợ khác. Việc không lưu trữ thông tin của họ một cách hệ thống dẫn đến khó khăn trong việc liên lạc, báo cáo, hoặc xin ý kiến phản hồi khi cần thiết.

## Giá trị Nghiệp vụ (Business Value):

- **Centralized Contact List:** Tạo một danh bạ tập trung cho dự án, giúp bất kỳ thành viên nào cũng có thể tra cứu thông tin liên hệ.

- **Phân loại rõ ràng:** Giúp phân biệt được đâu là Stakeholder thực tế (Real) và đâu là nhân vật Ảo (Persona - thường dùng trong thiết kế sản phẩm/UX), tránh nhầm lẫn trong quá trình phát triển.

- **Dễ dàng tương tác:** Lưu trữ đầy đủ vai trò và thông tin liên lạc (Email, SĐT) giúp việc kết nối nhanh chóng hơn.

## Đối tượng (Actor):

- **Primary Actor:** Sinh viên (Student) — người trực tiếp quản lý và cập nhật danh sách này.

- **Secondary Actors:** Mentor — xem để biết sinh viên đang tương tác với ai.

---

## 2. Luồng Người dùng (User Flow)

## 2.1. Luồng Xem Danh sách & Tìm kiếm

1. Student truy cập menu **"Bên liên quan"** trong Workspace. Hệ thống gọi API `GET /api/projects/{projectId:guid}/stakeholders`.

2. Hệ thống hiển thị danh sách các Stakeholders có phân trang (Pagination) ở dạng bảng hoặc lưới (Grid), hỗ trợ sắp xếp theo Tên, Email hoặc Ngày tạo.

3. Student nhập từ khóa vào thanh tìm kiếm. Hệ thống gọi API kèm tham số `SearchTerm`.

4. Danh sách tự động lọc Server-side theo Tên, Vai trò hoặc Email và hiển thị kết quả tương ứng.

## 2.2. Luồng Thêm mới Stakeholder (Create)

1. Student nhấn nút **"Thêm mới"**.

2. Hệ thống hiển thị Modal/Form "Thêm Bên liên quan".

3. Student nhập các thông tin:

   - **Phân loại (Type):** Mặc định là "Thực tế" (Real), hoặc chọn "Ảo" (Persona).
   - **Tên (Name):** Bắt buộc, tối đa 200 ký tự.
   - **Email:** Bắt buộc, đúng định dạng email, tối đa 150 ký tự và không được trùng với Stakeholder khác trong cùng một dự án.
   - **Vai trò (Role):** Tùy chọn, tối đa 100 ký tự (Ví dụ: Product Owner, Sponsor, Tester...).
   - **Mô tả (Description):** Tùy chọn, tối đa 500 ký tự.
   - **Số điện thoại (PhoneNumber):** Tùy chọn, hợp lệ định dạng số, tối đa 15 ký tự.

4. Nhấn nút **"Thêm mới"** (Save): Hệ thống gọi API `POST /api/stakeholders`. Thành công hoặc đóng form nếu Hủy.

## 2.3. Luồng Chỉnh sửa & Xóa (Update & Delete)

1. **Sửa:** Student click vào nút "Sửa" (icon bút chì) trên dòng thông tin → Gọi API `GET /api/stakeholders/{id:guid}` để lấy thông tin chi tiết (hoặc dùng thông tin lấy từ bảng) → Form hiện ra với dữ liệu cũ → Chỉnh sửa → Lưu qua API `PUT /api/stakeholders/{id:guid}`.

2. **Xóa:** Student click nút "Xóa" (icon thùng rác) → Hệ thống hiện popup xác nhận "Bạn có chắc chắn muốn xóa?" → Xác nhận → Gọi API `DELETE /api/stakeholders/{id:guid}` (Thực hiện hành động Soft Delete).

---

## 3. Tiêu chí Chấp nhận (Acceptance Criteria)

## AC-STK-01 — Hiển thị Danh sách Stakeholders

- **Given:** Student truy cập trang Bên liên quan.
- **When:** Trang tải xong.
- **Then:**
  - Hiển thị danh sách đầy đủ các Stakeholders đã tạo theo từng trang (Pagination).
  - Hiển thị các cột thông tin chính: Tên, Phân loại (Type - Label màu khác nhau cho Thực tế/Ảo), Vai trò, Email, SĐT.
  - Có nút "Thêm mới" nổi bật.
  - Cho phép sắp xếp dữ liệu (Sort) theo Name, Email, CreatedAt.

## AC-STK-02 — Thêm mới Stakeholder thành công

- **Given:** Student mở form Thêm mới.
- **When:** Nhập đầy đủ các trường hợp lệ và nhấn "Thêm mới".
- **Then:**
  - Bản ghi mới được lưu vào hệ thống.
  - Modal đóng lại.
  - Danh sách tự động tải lại (hoặc cập nhật) hiển thị Stakeholder vừa thêm.
  - Thông báo "Thêm mới thành công".

## AC-STK-03 — Validate Form Thêm mới/Sửa (Validation Integrity)

- **Given:** Student đang nhập liệu.
- **When:**
  - Bỏ trống Name hoặc Email.
  - Name > 200 ký tự, Email > 150 ký tự, PhoneNumber > 15 ký tự, Role > 100 ký tự, Description > 500 ký tự.
  - Nhập sai định dạng Email hoặc Regex PhoneNumber không hợp lệ.
  - Email nhập vào đã được sử dụng cho một Stakeholder khác trong cùng Project.
- **Then:**
  - Hệ thống (FluentValidation Backend) báo lỗi hoặc ném ra exception HTTP 400 Bad Request / 409 Conflict.
  - Nút "Thêm mới/Lưu" bị disable hoặc Frontend hiển thị lỗi validation ngay tại trường tương ứng.

## AC-STK-04 — Chức năng Hủy bỏ

- **Given:** Student đang mở form Thêm/Sửa và đã nhập một số dữ liệu.
- **When:** Nhấn nút "Hủy".
- **Then:**
  - Modal đóng lại ngay lập tức.
  - Dữ liệu đang nhập dở không được lưu.
  - Quay về màn hình danh sách ban đầu.

## AC-STK-05 — Tìm kiếm Stakeholder

- **Given:** Danh sách có nhiều Stakeholders.
- **When:** Student nhập từ khóa vào ô tìm kiếm.
- **Then:**
  - Hệ thống thực hiện gọi API Server-side kết hợp truyền tham số `SearchTerm`.
  - Lọc danh sách Stakeholder theo trường "Tên" (Name), "Vai trò" (Role) hoặc "Email". Việc so khớp mang tính tương đối và không phân biệt chữ hoa chữ thường.
  - Lưới hiển thị kết quả khớp ngay lập tức.

## AC-STK-06 — Xóa Stakeholder

- **Given:** Student chọn xóa một Stakeholder.
- **When:** Xác nhận trên popup cảnh báo.
- **Then:**
  - Backend thực hiện gọi API trả về HTTP 200 OK tiến hành Soft Delete bản ghi.
  - Danh sách cập nhật lại loại bỏ Stakeholder đã xóa.

---

## 4. Đặc tả kỹ thuật (Technical Notes)

- **Mapping Enum Type:** Trường "Phân loại" (Type) trong database được lưu bằng `StakeholderType` Enum (`0: Real`, `1: Persona`). Khi trả về Response DTO hoặc nhận vào qua API, dữ liệu thể hiện dưới dạng Chuỗi String (`"Real"`, `"Persona"`).
- **Pagination & Search:** Danh sách Stakeholders trên API `GET /api/projects/{projectId}/stakeholders` sử dụng model `PaginatedResult` cung cấp phân trang `PageNumber`, `PageSize`, kết hợp tìm kiếm `SearchTerm` và cho phép Sort qua `SortColumn` (`name`, `email`, `createdat`).
- **Data Validation & Integrity:** Hệ thống xài FluentValidation trên Command hạn chế độ dài thuộc tính (Name max 200, Role max 100, Description max 500, ..). Bắt buộc kiểm tra trùng Email tại level Database (Email không được trùng lặp cho cùng một `ProjectId`, so sánh case-insensitive).
- **Security & Authorization:** Đầu cuối REST Endpoint sử dụng `[Authorize]`.
  - **[⚠️ Architecture Violation]:** Tại tầng Application Services (Commands/Queries Handler) đang thiếu logic phân quyền kiểm tra Project Membership. Nếu user có token Authorization hợp lệ, họ dùng Client Call tùy ý vẫn có thể CRUD thao tác đến Stakeholder của `projectId` bất kì chỉ cần dự án đó có thực. Hệ thống cần được bảo mật và thêm một lớp Permission Middleware hoặc Security Check trong Handler cho Stakeholder Management.