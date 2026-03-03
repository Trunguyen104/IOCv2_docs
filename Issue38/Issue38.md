# 1. Tổng quan & Bối cảnh (Overview & Context)

## Vấn đề (Problem):

Trong quá trình làm việc với các bên liên quan (khách hàng, người dùng, tư vấn...), thường xuyên phát sinh các vấn đề, phản hồi, hoặc yêu cầu thay đổi (Issues/Concerns). Nếu các vấn đề này chỉ được trao đổi qua tin nhắn/email mà không được ghi nhận lại hệ thống, nhóm dự án dễ bị bỏ sót, quên xử lý hoặc không theo dõi được trạng thái giải quyết, dẫn đến mâu thuẫn hoặc chậm tiến độ.

## Giá trị Nghiệp vụ (Business Value):

- **Theo dõi chặt chẽ:** Đảm bảo mọi phản hồi/vấn đề từ Stakeholder đều được ghi nhận (Logged) và có trạng thái xử lý rõ ràng.

- **Minh bạch trách nhiệm:** Biết rõ vấn đề nào đến từ Stakeholder nào để dễ dàng liên hệ làm rõ.

- **Cải thiện quan hệ:** Việc phản hồi và giải quyết vấn đề kịp thời giúp nâng cao sự hài lòng của Stakeholder và Mentor.

## Đối tượng (Actor):

- **Primary Actor:** Sinh viên (Student) — người ghi nhận và cập nhật trạng thái vấn đề.

- **Secondary Actors:** Mentor — xem để nắm bắt các rủi ro hoặc khó khăn nhóm đang gặp phải với bên ngoài.

---

# 2. Luồng Người dùng (User Flow)

## 2.1. Luồng Xem Danh sách Vấn đề

1. Student truy cập vào module **"Vấn đề Stakeholder"**. Hệ thống gọi API `GET /api/stakeholder-issues`.

2. Hệ thống hiển thị bảng danh sách các vấn đề đã ghi nhận có dạng phân trang (Pagination).

3. Bảng gồm các cột: **Tiêu đề**, **Bên liên quan (StakeholderName)**, **Trạng thái** (Status), **Ngày tạo (CreatedAt)** và **Ngày giải quyết (ResolvedAt)**. Dữ liệu có thể được sắp xếp (Sort).

## 2.2. Luồng Tạo mới Vấn đề (Create)

1. Student nhấn nút **"Ghi nhận vấn đề"** (hoặc "Thêm mới").

2. Hệ thống hiển thị Form tạo mới. Student nhập thông tin:

   - **Bên liên quan (StakeholderId):** Chọn từ danh sách (Dropdown/Combobox kết nối API `GET /api/projects/{projectId}/stakeholders`).
   - **Tiêu đề (Title):** Yêu cầu thông tin, độ dài tối đa 200 ký tự.
   - **Mô tả (Description):** Nội dung tóm tắt vấn đề phát sinh, tối đa 2000 ký tự.

3. Nhấn **"Lưu"**, hệ thống gọi API `POST /api/stakeholder-issues`. Bản ghi mới sẽ mang trạng thái mặc định "Open".

## 2.3. Luồng Cập nhật & Xử lý (Update Status)

1. Cập nhật tiến độ không yêu cầu sửa toàn bộ thông tin. Student có thể đổi trạng thái nhanh.

2. Chọn cập nhật trạng thái mới (ví dụ như "InProgress", "Resolved", hoặc "Closed"). Hệ thống gọi API `PATCH /api/stakeholder-issues/{id}/status`.

3. Khi trạng thái chuyển qua "Resolved" hoặc "Closed", hệ thống Backend tự động ghi nhận thời gian `ResolvedAt`. Nếu đổi ngược về "Open" hoặc "InProgress", `ResolvedAt` sẽ được xóa trăng (Null).

## 2.4. Luồng Tìm kiếm, Lọc & Xóa

1. **Tìm kiếm (Search):** Nhập từ khóa, API lọc Server-side trên cả 3 trường `Title`, `Description` và `Stakeholder.Name`.
2. **Lọc (Filter):** Dữ liệu có thể lọc thêm theo cụ thể `ProjectId` dự án, `StakeholderId` riêng biệt, hoặc `Status` thông qua query string.
3. **Xóa:** Student nhấn nút Xóa, hệ thống gọi API `DELETE /api/stakeholder-issues/{id}` (Lưu ý: Thực hiện **Hard Delete** xóa vĩnh viễn khỏi Database).

---

## 3. Tiêu chí Chấp nhận (Acceptance Criteria)

## AC-ISS-01 — Hiển thị Bảng Vấn đề

- **Given:** Student truy cập trang quản lý vấn đề Stakeholder.
- **When:** Trang tải xong.
- **Then:**
  - Hệ thống gọi API `GET /api/stakeholder-issues` với `PaginationParams`.
  - Hiển thị đầy đủ thông tin phân trang. Dữ liệu bao gồm Tiêu đề, Tên Stakeholder, Status (Enum String) và thời gian tương ứng.
  - Cho phép sắp xếp (Sort) theo title, status, createdat, hoặc stakeholdername.

## AC-ISS-02 — Tạo mới Vấn đề thành công (Validation Integrity)

- **Given:** Student mở form thêm mới.
- **When:**

  - Chọn đúng một Stakeholder từ danh sách (Dropdown phải load được list Stakeholder từ module trước).

  - Nhập Tiêu đề và Mô tả hợp lệ.

  - Nhấn Lưu.
  
  [Missing] Cần có Validation cho trường StakeholderId, Title, Description. 

- **Then:**
  - Nếu tất cả hợp lệ, bản ghi mới được lưu. Trạng thái mang định là "Open".
  [Validation] Controller (FluentValidation) trả về HTTP 400 Bad Request ngay lập tức. Nhưng trong code hiện tại không có Validation.

## AC-ISS-03 — Cập nhật Vấn đề (Đổi trạng thái)

- **Given:** Một vấn đề đang ở trạng thái Open.
- **When:** Student đổi trạng thái (chẳng hạn từ "Open" sang "Resolved") qua `PATCH`.
- **Then:**
  - Trạng thái thay đổi thành công.
  - Nếu Status là `Resolved` hoặc `Closed`, trường `ResolvedAt` cập nhật biến thời gian chuẩn `UtcNow`. 
  - Nếu Status là `Open` hoặc `InProgress`, trường `ResolvedAt` mang giá trị NULL.

## AC-ISS-04 — Xóa Vấn đề (Xóa mềm) [Missing]

- **Given:** Student chọn xóa một vấn đề.
- **When:** Xác nhận xóa.
- **Then:**
  [Missing] API DELETE trả về thành công 200 OK. Hệ thống thực hiện Soft Delete trong database.
  - Danh sách bản ghi được tải lại.

## AC-ISS-05 — Chức năng Tìm kiếm & Lọc Server-Side

- **Given:** Có nhiều vấn đề từ nhiều Stakeholder khác nhau.
- **When:** Student sử dụng các filter/search field.
- **Then:**

  - Danh sách chỉ hiển thị các vấn đề Open của Mr. John.

  - Bộ lọc hoạt động kết hợp (AND logic).

---

## 4. Đặc tả kỹ thuật (Technical Notes)

- **Status Enum:** Định nghĩa tập Status cố định trong Domain Enum (`0: Open`, `1: InProgress`, `2: Resolved`, `3: Closed`). Mặc dù vậy, API nhận và trả thông tin qua DTO ở dạng Chuỗi (String format như "Open", "InProgress").
- **Endpoint Structure:** Thiết kế RESTful chia tách rõ ràng như `POST` để tạo mới, và `PATCH` trên url `/status` để quản lý trạng thái cập nhật (Tránh `PUT` vì thiết kế hiện tại không support Full Update Issue). Việc Xóa gọi bằng `DELETE` thực hiện Hard Delete trực tiếp.
- **[⚠️ Architecture Violation]:** System đang dùng `[Authorize]` trên Controller nhưng tại những Handlers cấp dưới (như quản lý Xóa / Đổi trạng thái UpdateStatus), code *chưa* kiểm tra xem người dùng thực hiện (Current Student) có thật sự thuộc dự án Project chứa Issue này để được quyền chỉnh sửa hay không. Mọi user đủ Authorization Token đều có thể thao tác nếu có được param `Id`. Cần bổ sung phân quyền trong backend!