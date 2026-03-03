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

1. Student truy cập menu **"Bên liên quan"** trong Workspace.

2. Hệ thống hiển thị danh sách các Stakeholders dạng bảng hoặc lưới (Grid).

3. Student nhập tên Stakeholder vào thanh tìm kiếm.

4. Danh sách tự động lọc và hiển thị kết quả tương ứng.

## 2.2. Luồng Thêm mới Stakeholder (Create)

1. Student nhấn nút **"Thêm mới"**.

2. Hệ thống hiển thị Modal/Form "Thêm Bên liên quan".

3. Student nhập các thông tin:

   - **Phân loại (Bắt buộc):** Chọn "Thực tế" (Real) hoặc "Ảo" (Persona).

   - **Tên (Bắt buộc):** Nhập tên người hoặc tổ chức.

   - **Vai trò:** (Ví dụ: Product Owner, Sponsor, Tester...).

   - **Mô tả:** Ghi chú thêm về trách nhiệm/ảnh hưởng.

   - **Email & SĐT:** Thông tin liên hệ.

4. Nhấn nút **"Thêm mới"** (Save) để lưu hoặc **"Hủy"** để đóng form.

## 2.3. Luồng Chỉnh sửa & Xóa (Update & Delete)

1. **Sửa:** Student click vào nút "Sửa" (icon bút chì) trên dòng thông tin → Form hiện ra với dữ liệu cũ → Chỉnh sửa → Lưu.

2. **Xóa:** Student click nút "Xóa" (icon thùng rác) → Hệ thống hiện popup xác nhận "Bạn có chắc chắn muốn xóa?" → Xác nhận → Bản ghi bị xóa khỏi danh sách.

---

## 3. Tiêu chí Chấp nhận (Acceptance Criteria)

## AC-STK-01 — Hiển thị Danh sách Stakeholders

- **Given:** Student truy cập trang Bên liên quan.

- **When:** Trang tải xong.

- **Then:**

  - Hiển thị danh sách đầy đủ các Stakeholders đã tạo.

  - Hiển thị các cột thông tin chính: Tên, Phân loại (Label màu khác nhau cho Thực tế/Ảo), Vai trò, Email, SĐT.

  - Có nút "Thêm mới" nổi bật.

## AC-STK-02 — Thêm mới Stakeholder thành công

- **Given:** Student mở form Thêm mới.

- **When:** Nhập đầy đủ các trường bắt buộc (Tên) và nhấn "Thêm mới".

- **Then:**

  - Bản ghi mới được lưu vào hệ thống.

  - Modal đóng lại.

  - Danh sách tự động cập nhật hiển thị Stakeholder vừa thêm.

  - Thông báo "Thêm mới thành công".

## AC-STK-03 — Validate Form Thêm mới/Sửa

- **Given:** Student đang nhập liệu.

- **When:**

  - Bỏ trống Tên.

  - Nhập sai định dạng Email.

- **Then:**

  - Hệ thống hiển thị lỗi validation ngay tại trường tương ứng.

  - Nút "Thêm mới/Lưu" bị disable hoặc không cho submit cho đến khi dữ liệu hợp lệ.

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

  - Lọc danh sách theo trường "Tên" (có thể mở rộng tìm theo Vai trò/Email).

  - Hiển thị kết quả khớp ngay lập tức (Client-side search hoặc Debounce API search).

## AC-STK-06 — Xóa Stakeholder

- **Given:** Student chọn xóa một Stakeholder.

- **When:** Xác nhận trên popup cảnh báo.

- **Then:**

  - Bản ghi bị xóa vĩnh viễn.

  - Danh sách cập nhật lại.

---

## 4. Đặc tả kỹ thuật (Technical Notes)

- **Enum Field:** Trường "Phân loại" trong database nên lưu dưới dạng Enum hoặc Integer (`0: Real`, `1: Persona`) hoặc String code (`REAL`, `VIRTUAL`) để dễ dàng filter và style màu sắc (ví dụ: Real màu xanh, Persona màu tím).

- **Security:** Chỉ thành viên trong nhóm dự án (Project Member) mới có quyền CRUD Stakeholder của dự án đó. Cần check permission ở Backend.