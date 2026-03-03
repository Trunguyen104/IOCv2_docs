## 1. Tổng quan & Bối cảnh (Overview & Context)

## Vấn đề (Problem):

Trong quá trình thực tập, sinh viên cần nắm rõ các thông tin hành chính của nhóm mình (như thuộc kỳ thực tập nào, mentor là ai, thời hạn dự án...) và thông tin liên lạc của các thành viên khác trong nhóm. Nếu thiếu màn hình này, sinh viên sẽ phải hỏi thủ công hoặc tra cứu rời rạc, gây mất thời gian kết nối.

## Giá trị Nghiệp vụ (Business Value):

- **Minh bạch thông tin:** Cung cấp "Single Source of Truth" về thông tin dự án (thời gian, người hướng dẫn, phạm vi).

- **Kết nối đội nhóm:** Giúp sinh viên dễ dàng tìm thấy thông tin liên lạc (email) của đồng đội để phối hợp làm việc.

- **Xác nhận bối cảnh:** Giúp sinh viên xác nhận mình đã được add vào đúng nhóm, đúng kỳ và đúng doanh nghiệp hay chưa.

## Đối tượng (Actor):

- **Primary Actor:** Sinh viên (Student) — xem để biết thông tin nhóm mình.

- **Secondary Actors:** Mentor, HR — CRUD danh sách thành viên nhóm.

---

## 2. Luồng Người dùng (User Flow)

## 2.1. Luồng Xem Thông tin chung (Project Metadata)

1. Student truy cập vào menu **"Thông tin chung"** (hoặc "Nhóm dự án") trong Workspace.

2. Hệ thống hiển thị phần **"Thông tin dự án"** bao gồm các trường:

   - ID Dự án, Tên nhóm.

   - Kỳ thực tập (Internship Phase).

   - Doanh nghiệp (Enterprise), Trường (University).

   - Mentor hướng dẫn.

   - Thời gian: Ngày bắt đầu, Ngày kết thúc.

   - Mô tả dự án.

   - Thông tin Audit: Ngày tạo, Ngày cập nhật.

## 2.2. Luồng Xem & Tìm kiếm Thành viên (Members List)

1. Bên dưới phần thông tin chung là phần **"Danh sách thành viên"**.

2. Hệ thống hiển thị bảng danh sách sinh viên trong nhóm với các cột: STT, Avatar, Họ tên, Email, Ngày sinh, Giới tính, Vai trò (Leader/Member - nếu có).

3. Student nhập từ khóa vào ô **Tìm kiếm thành viên** (tìm theo Tên hoặc Email).

4. Bảng danh sách tự động lọc hiển thị các thành viên khớp với từ khóa.

---

## 3. Tiêu chí Chấp nhận (Acceptance Criteria)

## AC-TEAM-01 — Hiển thị Thông tin chung Dự án

- **Given:** Student truy cập trang Thông tin nhóm.

- **When:** Trang tải xong.

- **Then:**

  - Hiển thị đầy đủ và chính xác các trường thông tin Read-only: ID, Tên nhóm, Kỳ thực tập, Doanh nghiệp, Trường, Mentor, Ngày bắt đầu/kết thúc, Mô tả.

  - Định dạng ngày tháng chuẩn (dd/MM/yyyy).

  - Tên Doanh nghiệp/Trường/Mentor có thể là link (click vào để xem profile chi tiết nếu hệ thống hỗ trợ).

## AC-TEAM-02 — Hiển thị Danh sách Thành viên

- **Given:** Nhóm đã có thành viên.

- **When:** Student xem phần danh sách thành viên.

- **Then:**

  - Hiển thị bảng danh sách sinh viên.

  - Các cột bắt buộc: STT, Avatar (nếu không có avatar thì hiển thị placeholder hoặc chữ cái đầu), Họ tên, Email, Ngày sinh, Giới tính.

  - Danh sách sắp xếp theo tên (A-Z) hoặc vai trò (Leader trước).

## AC-TEAM-03 — Tìm kiếm Thành viên

- **Given:** Nhóm có nhiều thành viên.

- **When:** Student nhập từ khóa vào ô tìm kiếm thành viên.

- **Then:**

  - Lọc danh sách theo Tên hoặc Email ngay lập tức (client-side filter là đủ vì số lượng thành viên nhóm thường nhỏ &lt; 20).

  - Nếu không tìm thấy, hiển thị thông báo "Không tìm thấy thành viên phù hợp".

## AC-TEAM-04 — Quyền hạn (View-only)

- **Given:** Student truy cập trang này.

- **When:** Thử chỉnh sửa thông tin nhóm hoặc xóa thành viên.

- **Then:**

  - Student **KHÔNG** nhìn thấy nút Sửa/Xóa.

  - Chỉ Mentor hoặc Quản lý (Admin) mới có quyền chỉnh sửa thông tin nhóm hoặc thêm/bớt thành viên.

## AC-TEAM-05 — UI/UX Responsive

- **Given:** Student truy cập trên thiết bị di động.

- **When:** Xem danh sách thành viên.

- **Then:**

  - Bảng danh sách hiển thị dạng Card hoặc có thanh cuộn ngang để không bị vỡ giao diện.

  - Các thông tin quan trọng (Tên, Email) vẫn dễ đọc.

---

## 4. Đặc tả kỹ thuật (Technical Notes)

- **Data Source:** API lấy thông tin chi tiết Project (`GET /api/projects/{id}`) cần trả về kèm object `members` (danh sách sinh viên) và object `mentor`, `enterprise`, `university` đã được populate (join bảng) để hiển thị tên thay vì chỉ hiển thị ID.

- **Search:** Vì số lượng thành viên trong một nhóm thực tập thường ít (5-10 người), nên thực hiện chức năng **Search/Filter tại Client (Frontend)** để phản hồi nhanh nhất, không cần gọi API search riêng.

- **Avatar:** Sử dụng component Avatar (như của AntD/MUI) để xử lý hiển thị ảnh fallback khi user chưa upload avatar.