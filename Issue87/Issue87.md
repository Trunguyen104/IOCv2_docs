# Issue 87: Quản lý Hồ sơ Sinh viên (Student Profile)

## 1. Tổng quan & Bối cảnh (Overview & Context)

### 1.1 Vấn đề (Problem)
Hồ sơ sinh viên (Profile) là dữ liệu cốt lõi để doanh nghiệp đánh giá năng lực và xét tuyển thực tập. Hiện tại, sinh viên cần một giao diện tập trung để tự quản lý thông tin cá nhân, kỹ năng, và các liên kết sản phẩm nhằm đảm bảo hồ sơ luôn ở trạng thái tốt nhất trước khi gửi tới nhà tuyển dụng.

### 1.2 Giá trị Nghiệp vụ (Business Value)
- **Tăng cơ hội thực tập**: Hồ sơ chuyên nghiệp, đầy đủ thông tin giúp tăng khả năng "match" giữa sinh viên và dự án.
- **Dữ liệu chính xác**: Đảm bảo thông tin liên lạc (Email, Số điện thoại) luôn được cập nhật để giảng viên và doanh nghiệp có thể liên hệ.
- **Quyền riêng tư**: Sinh viên có quyền kiểm soát nội dung hiển thị trong hồ sơ cá nhân của mình.

### 1.3 Đối tượng (Actor)
- **Primary Actor**: Sinh viên (Student) — Người xem và trực tiếp chỉnh sửa hồ sơ.
- **Secondary Actors**: Doanh nghiệp (HR/Mentor), Quản trị viên trường — Chỉ có quyền xem thông tin.

---

## 2. Luồng Người dùng (User Flow)

### 2.1 Luồng Xem Hồ sơ (View Profile)
1. Sinh viên đăng nhập vào hệ thống và truy cập menu **"Hồ sơ của tôi"**.
2. Hệ thống gọi API lấy thông tin định danh và hồ sơ cá nhân dựa trên `UserId` từ token.
3. Hệ thống trả về thông tin cơ bản: Định danh (MSSV, Họ tên, Lớp), Thông tin liên hệ (Email, Số điện thoại), Kỹ năng, Link CV/Portfolio.

### 2.2 Luồng Chỉnh sửa & Cập nhật (Edit Profile)
1. Tại màn hình Profile, sinh viên nhấn nút **"Chỉnh sửa"**.
2. Sinh viên thay đổi thông tin cá nhân: Email, Số điện thoại, Kỹ năng, Link Portfolio.
3. Sinh viên nhấn **"Lưu thay đổi"**. Hệ thống validate dữ liệu và cập nhật dữ liệu.

### 2.3 Luồng Xóa nội dung tùy chọn (Partial Delete)
1. Sinh viên có thể chọn xóa từng phần nội dung không bắt buộc như: Link Portfolio, hoặc ảnh đại diện.
2. Hệ thống thực hiện cập nhật giá trị tương ứng về `null`.
*Lưu ý: Các trường thông tin định danh do trường cung cấp (MSSV, Tên, Lớp) sẽ không được phép xóa sửa tại đây*.

---

## 3. Tiêu chí Chấp nhận (Acceptance Criteria)

### AC-STU-01: Hiển thị thông tin Profile chính xác
- **Given**: Sinh viên đã đăng nhập và truy cập trang Profile.
- **When**: Gửi yêu cầu lấy thông tin hồ sơ.
- **Then**: Hệ thống trả về đầy đủ thông tin từ DB: `user_code`, `full_name`, `email`, `phone_number`, `major`. Kèm theo dữ liệu mở rộng như Kỹ năng, Portfolio.

### AC-STU-02: Chỉnh sửa thông tin thành công
- **Given**: Sinh viên đang ở chế độ chỉnh sửa.
- **When**: Nhập thông tin liên hệ (`email`/`phone_number`) mới hợp lệ và gửi yêu cầu.
- **Then**: Hệ thống cập nhật thành công, trả về HTTP 200/201 và cập nhật trên giao diện ngay lập tức.

### AC-STU-03: Ràng buộc Validation dữ liệu
- **Given**: Sinh viên nhập Email sai định dạng hoặc Số điện thoại chứa ký tự chữ.
- **When**: Gửi yêu cầu cập nhật hồ sơ.
- **Then**: Hệ thống từ chối và trả về lỗi **400 Bad Request** kèm message chi tiết.

### AC-STU-04: Bảo mật quyền sở hữu (Ownership)
- **Given**: Sinh viên A cố gắng gửi request kèm ID của sinh viên B để chỉnh sửa.
- **When**: Server nhận yêu cầu chỉnh sửa thông tin.
- **Then**: Hệ thống chỉ sử dụng ID trực tiếp lấy từ JWT Token (không tin tưởng body request) để truy xuất dữ liệu. Nếu phát hiện sai phạm, trả về **403 Forbidden** với cấu trúc: "Bạn không có quyền thực hiện hành động này".

---

## 4. Đặc tả Kỹ thuật (Technical Specifications)

### 4.1 Danh sách API Endpoints

| Ý định | HTTP Method | URL Path | Request Body / Query | Response Body | Status Codes |
|----------|-------------|----------|----------------------|---------------|--------------|
| Xem thông tin hồ sơ | `GET` | `/api/students/profile` | (None) | `{ "userCode": "...", "fullName": "...", "email": "...", "phoneNumber": "...", "major": "...", "skills": [...], "portfolioUrl": "..." }` | 200 OK<br>401 Unauthorized<br>404 Not Found<br>429 Too Many Requests |
| Cập nhật hồ sơ | `PUT` | `/api/students/profile` | `{ "email": "...", "phoneNumber": "...", "skills": [...], "portfolioUrl": "..." }` | `{ "message": "Cập nhật hồ sơ thành công" }` | 200 OK<br>400 Bad Request<br>401 Unauthorized<br>403 Forbidden<br>429 Too Many Requests |
| Xóa ảnh/portfolio | `PATCH` | `/api/students/profile/portfolio` | `{ "removePortfolio": true, "removeAvatar": true }` | `{ "message": "Xóa thành công" }` | 200 OK<br>401 Unauthorized<br>429 Too Many Requests |

### 4.2 Sự Lệch Chuẩn Cơ sở Dữ liệu (DB Alignment)

- **[⚠️ DB Mismatch] Thiếu trường Kỹ năng (`Skills`)**: Trong file `DB.md`, các bảng `students` và `users` đều không chứa thông tin liên quan đến kỹ năng. Cần bổ sung bảng `student_skills` hoặc thêm một cột JSONB `skills` vào bảng `students`.
- **[⚠️ DB Mismatch] Thiếu Link CV/Portfolio (`PortfolioUrl`)**: Tính năng yêu cầu cấu hình Link Portfolio nhưng `DB.md` hiện tại chỉ hỗ trợ `avatar_url` trong bảng `users`. Cần thêm cột `portfolio_url` (`varchar(255)`) vào bảng `students`.

### 4.3 Validation Rules & Constraints

- **`email`**: Regex chuẩn định dạng phổ quát, bắt buộc. Tối đa 150 ký tự (theo `users.email`).
- **`phoneNumber`**: Regex bắt buộc chỉ chứa ký tự số, tối đa 15 ký tự (theo `users.phone_number`).
- **`skills`**: Max length danh sách hoặc chuỗi không quá 500 ký tự.
- **`portfolioUrl`**: Định dạng URL chuẩn, tối đa 255 ký tự.

### 4.4 [FFA-SEC] Authentication & Authorization

- **Authentication**: Xác thực bằng **JWT**. Bắt buộc Header `Authorization: Bearer <token>`.
- **Authorization**: Role hợp lệ: `STUDENT`.
- **[⚠️ Architecture Violation] Zero-Trust ID**: Application Service/Handler KHÔNG được nhận `id` hoặc `student_id` từ input payload để tránh các tấn công IDOR xuyên quyền. Phải lấy `UserId` ngầm trong Context thông qua `ICurrentUserService`.

### 4.5 [FFA-PERF] Cấu hình Rate Limit

- Nhóm Query (`GET /api/students/profile`): 60 requests / 1 phút / IP.
- Nhóm Command (`PUT / PATCH` Profile): 10 requests / 1 phút / IP  (Nhằm ngăn chặn bot spam data).