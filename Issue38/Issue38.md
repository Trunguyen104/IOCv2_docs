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

1. Student truy cập vào tab (hoặc module) **"Vấn đề Stakeholder"** (nằm trong mục Quản lý Bên liên quan hoặc tách riêng).

2. Hệ thống hiển thị bảng danh sách các vấn đề đã ghi nhận.

3. Bảng gồm các cột: **Tiêu đề**, **Stakeholder** (Tên người liên quan), **Trạng thái** (Status), **Ngày tạo**.

## 2.2. Luồng Tạo mới Vấn đề (Create)

1. Student nhấn nút **"Ghi nhận vấn đề"** (hoặc "Thêm mới").

2. Hệ thống hiển thị Form tạo mới.

3. Student nhập thông tin:

   - **Bên liên quan (Stakeholder):** Chọn từ danh sách Stakeholder đã có (Dropdown/Combobox).

   - **Tiêu đề:** Tóm tắt ngắn gọn vấn đề.

   - **Mô tả:** Chi tiết nội dung phản hồi hoặc vấn đề gặp phải.

   - **Trạng thái (Status):** Mặc định là "Mới" (Open/New).

4. Nhấn **"Lưu"** để tạo bản ghi.

## 2.3. Luồng Cập nhật & Xử lý (Update Status)

1. Student click vào một dòng vấn đề để xem chi tiết hoặc sửa.

2. Cập nhật trạng thái (ví dụ: từ "Mới" sang "Đang xử lý" hoặc "Đã giải quyết").

3. Lưu lại thay đổi.

## 2.4. Luồng Tìm kiếm & Lọc

1. **Tìm kiếm:** Nhập từ khóa vào ô tìm kiếm (tìm theo Tiêu đề).

2. **Lọc (Filter):**

   - Chọn lọc theo **Status** (ví dụ: chỉ xem các vấn đề "Đang xử lý").

   - Chọn lọc theo **Stakeholder** (ví dụ: xem tất cả vấn đề của "Khách hàng A").

3. Danh sách tự động cập nhật kết quả.

---

## 3. Tiêu chí Chấp nhận (Acceptance Criteria)

## AC-ISS-01 — Hiển thị Bảng Vấn đề

- **Given:** Student truy cập trang quản lý vấn đề Stakeholder.

- **When:** Trang tải xong.

- **Then:**

  - Hiển thị đúng các cột: Tiêu đề, Stakeholder, Status (dạng Badge màu), Ngày tạo (format dd/MM/yyyy).

  - Nếu danh sách trống, hiển thị thông báo "Chưa có vấn đề nào được ghi nhận".

## AC-ISS-02 — Tạo mới Vấn đề thành công

- **Given:** Student mở form thêm mới.

- **When:**

  - Chọn đúng một Stakeholder từ danh sách (Dropdown phải load được list Stakeholder từ module trước).

  - Nhập Tiêu đề và Mô tả hợp lệ.

  - Nhấn Lưu.

- **Then:**

  - Vấn đề mới được tạo với Status mặc định (Open).

  - Hiển thị thông báo thành công.

  - Dòng mới xuất hiện trên đầu danh sách.

## AC-ISS-03 — Cập nhật Vấn đề (Sửa/Đổi trạng thái)

- **Given:** Một vấn đề đang ở trạng thái Open.

- **When:** Student sửa trạng thái thành "Resolved" (Đã giải quyết) và cập nhật mô tả giải pháp.

- **Then:**

  - Trạng thái thay đổi thành công.

  - Màu sắc Badge trạng thái thay đổi tương ứng (ví dụ: Open màu Xanh dương → Resolved màu Xanh lá).

## AC-ISS-04 — Xóa Vấn đề

- **Given:** Student chọn xóa một vấn đề.

- **When:** Xác nhận xóa.

- **Then:**

  - Bản ghi bị xóa khỏi hệ thống.

  - Danh sách được reload.

## AC-ISS-05 — Chức năng Tìm kiếm & Lọc

- **Given:** Có nhiều vấn đề từ nhiều Stakeholder khác nhau.

- **When:**

  - Student chọn Filter Stakeholder = "Mr. John".

  - Và Filter Status = "Open".

- **Then:**

  - Danh sách chỉ hiển thị các vấn đề Open của Mr. John.

  - Bộ lọc hoạt động kết hợp (AND logic).

---

## 4. Đặc tả kỹ thuật (Technical Notes)

- **Relationship:** Bảng `Stakeholder_Issues` phải có khóa ngoại (Foreign Key) trỏ đến bảng `Stakeholders`.

- **Dropdown Data:** Khi load form tạo mới, Frontend cần gọi API lấy danh sách Stakeholder (`GET /api/stakeholders`) để đổ vào Dropdown chọn. Nếu Stakeholder chưa có, có thể gợi ý UX cho phép "Thêm nhanh Stakeholder" ngay tại đó (Optional).

- **Status Enum:** Nên định nghĩa tập Status cố định (Open, In Progress, Resolved, Closed) để dễ quản lý và filter.