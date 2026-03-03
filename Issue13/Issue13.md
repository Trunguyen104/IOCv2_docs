# Xác thực & Phân quyền (Authentication & Authorization)

## 1. Tổng quan & Bối cảnh (Overview & Context)

### Vấn đề (Problem):

Hệ thống IOC có 3 đối tượng người dùng chính (Nhà trường, Doanh nghiệp, Sinh viên) và các cấp quản trị (MASTER, VẬN HÀNH). Cần một cơ chế xác thực tập trung để:

- Đảm bảo tính bảo mật cho dữ liệu nhạy cảm (CV, bảng điểm, hợp đồng).

- Kiểm soát quyền truy cập: Sinh viên không thể vào Portal của Nhà trường và ngược lại.

- Duy trì phiên làm việc ổn định và an toàn.

### Giá trị Nghiệp vụ (Business Value):

1. **Bảo mật**: Chỉ người dùng hợp lệ mới có thể truy cập tài nguyên.

2. **Cá nhân hóa**: Điều hướng người dùng đến đúng Portal tương ứng ngay sau khi đăng nhập.

3. **Tuân thủ**: Lưu vết (Audit log) các lượt đăng nhập để phòng ngừa rủi ro bảo mật.

### Đối tượng (Actor):

- **Toàn bộ người dùng**: Sinh viên, Doanh nghiệp (Mentor/HR), Nhà trường (School Admin), Quản trị hệ thống.

### User Story Statement

**Là người dùng hệ thống**, tôi muốn **đăng nhập vào hệ thống bằng Email/Mật khẩu** để **truy cập vào các tính năng và dữ liệu phù hợp với vai trò của mình**.

---

## 2. Luồng Người dùng (User Flow)

### 2.1. Luồng Đăng nhập (Login Flow)

1. Người dùng truy cập trang Login.

2. Nhập **Email**, **Mật khẩu** và tùy chọn **Remember Me** (Ghi nhớ đăng nhập).

3. Hệ thống kiểm tra:

   - Hệ thống đánh giá Rate limit (Chặn theo IP tối đa 5 lần mỗi 10 phút. Chặn theo Email tối đa 5 lần nhập sai mật khẩu mỗi 15 phút, khóa 15 phút).
   - Nếu thông tin sai: Hệ thống ghi nhận số lần sai và hiển thị lỗi.
   - Nếu tài khoản bị vô hiệu hóa (INACTIVE): Hiển thị thông báo "Tài khoản hiện đang bị khóa".

4. Nếu thông tin đúng:

   - Hệ thống tạo mã **JWT Token** (Access Token & Refresh Token).
     - *Lưu ý: Nếu người dùng chọn "Remember Me", thời gian hết hạn của Refresh Token sẽ được kéo dài (30 ngày), ngược lại sử dụng cấu hình mặc định (7 ngày).*

   - Lưu thẳng Access Token và Refresh Token vào hệ thống **Cookies (HttpOnly, Secure, SameSite)** ở phía Client.

   - Trả thông tin (Email, Role, AccessToken, RefreshToken...) về cho Client xử lý tiếp.

5. Điều hướng (Routing) dựa trên Role trả về:

   - `Role: STUDENT` → Chuyển hướng đến `/student-portal`

   - `Role: SCHOOL_ADMIN` → Chuyển hướng đến `/university-portal`

   - `Role: ENTERPRISE_ADMIN` → Chuyển hướng đến `/enterprise-portal`

   - `Role: MASTER` → Chuyển hướng đến `/admin-dashboard`

### 2.2. Luồng Kiểm soát truy cập (Authorization Flow)

1. Khi người dùng truy cập một đường dẫn (URL) cụ thể hoặc gọi dữ liệu từ API (`[Authorize]`):

   - Middleware/Guard kiểm tra tính hợp lệ của JWT Token (Access Token).

   - Kiểm tra quyền của Role hiện tại có khớp với yêu cầu của trang hay không.

2. Nếu không có Token: Chuyển hướng về trang `/login` (API trả về 401 Unauthorized).

3. Nếu có Token nhưng sai Role: Hiển thị trang **403 Forbidden** (Không có quyền truy cập).

### 2.3. Luồng Đăng xuất (Logout Flow)

1. Người dùng nhấn "Đăng xuất".

2. Chuyển thao tác xuống server (kèm theo token). Hệ thống thu hồi (Revoke) Refresh Token trong DB bằng lệnh `RevokeTokenCommand` và server yêu cầu xóa ngay `accessToken` & `refreshToken` khỏi Cookies của Client.

3. Chuyển hướng người dùng về trang Login.

---

## 3. Tiêu chí Chấp nhận (Acceptance Criteria)

**AC-AUTH-01 — Đăng nhập thành công và điều hướng đúng role**

- **Given**: Người dùng đã có tài khoản ACTIVE trong hệ thống.

- **When**: Nhập đúng Email, Mật khẩu (có thể kèm Remember Me) và nhấn "Đăng nhập".

- **Then**:

  - API set Cookie chứa thông tin Access Token và Refresh Token với cờ an toàn HttpOnly.
  - Trả về response chứa: `AccessToken`, `RefreshToken`, `Email`, `Role`, `ExpiresIn`...

  - Điều hướng đúng về Dashboard tương ứng (Sinh viên vào trang Student, Nhà trường vào trang University...).

  - Hiển thị Toast thông báo "Đăng nhập thành công".

**AC-AUTH-02 — Xử lý đăng nhập thất bại và Rate Limit**

- **Given**: Người dùng nhập sai Email/Mật khẩu hoặc tài khoản đang trạng thái INACTIVE.

- **When**: Nhấn "Đăng nhập".

- **Then**:

  - Không chuyển hướng trang.

  - Nếu sai Email/Mật khẩu: Hiển thị lỗi xác thực. Sau 5 lần nhập sai trong vòng 15 phút, tài khoản đó sẽ tạm thời bị vô hiệu hóa tính năng đăng nhập trong 15 phút tiếp theo.

  - Hiển thị thông báo lỗi tương ứng (ví dụ: "Tài khoản của bạn đã bị khóa, vui lòng liên hệ Admin").

**AC-AUTH-03 — Bảo mật JWT Token**

- **Given**: Người dùng đã đăng nhập thành công.

- **When**: Thực hiện các yêu cầu API (fetch dữ liệu).

- **Then**:

  - Request tự động đính kèm Token dựa vào Cookies, hoặc được client gửi kèm Header `Authorization: Bearer <token>`.

  - Nếu Token hết hạn (Expired), hệ thống có thể gọi `/api/v1/auth/tokens/refresh` để dùng Refresh Token (từ Cookies) đổi Access Token mới tự động không gián đoạn (Silent Refresh).

**AC-AUTH-04 — Phân quyền truy cập trang (Role-based Routing)**

- **Given**: Một Sinh viên đang đăng nhập.

- **When**: Cố tình nhập URL của trang Quản trị nhà trường (ví dụ: `/university-portal/settings`).

- **Then**: Hệ thống ngăn chặn truy cập và hiển thị HTTP lỗi "403 - Bạn không có quyền truy cập".

**AC-AUTH-06 — Quên mật khẩu**

- **Given**: Người dùng quên mật khẩu.

- **When**: Nhấn "Quên mật khẩu" và nhập Email.

- **Then**: 
  - API kiểm tra Rate Limit (Tối đa 3 requests/10 phút theo IP).
  - API **luôn** trả về thành công với thông báo mặc định (ví dụ: "Nếu email tồn tại, hệ thống đã gửi link reset") bất kể email có trong CSDL hay không để bảo mật chống Email Enumeration (Dò tìm danh sách Email).
  - Nếu email tồn tại, hệ thống sinh ra Reset Token băm (SHA256) lưu DB và gửi link (thời hạn 15 phút).

---

## 4. Đặc tả kỹ thuật (Technical Notes)

- **Công nghệ**: JWT (JSON Web Token), thao tác qua HttpOnly/Secure Cookies.

- **Password Hashing**: Mật khẩu mã hóa trước khi lưu Database. Quá trình tạo token Request Reset Password cũng sử dụng SHA256 mã hóa Token DB.

- **Token Expiration**:

  - Access Token: Quản lý theo cấu hình (mặc định server config, thường là vài phút đến 1 giờ).

  - Refresh Token: Phụ thuộc vào `RememberMe` (Có chọn Remember Me -> 30 Ngày. Không -> Mặc định theo config: 7 Ngày).

- **Validation**:

  - Email phải đúng định dạng regex.
  [Missing] - [⚠️ Architecture Violation] **Login Component**: Validation của Request Login (Tệp `LoginValidator.cs`) hiện tại **KHÔNG** kiểm tra Regex Email, và thiết lập minimum Password chỉ 3 ký tự (không tuân thủ độ phức tạp như Docs).

  - Mật khẩu tối thiểu 8 ký tự, bao gồm chữ hoa, chữ thường và ký tự đặc biệt.

- **Cơ chế Rate Limit (Giới hạn requests)**:
  - Controller (IP Focus): Ngăn chặn Flood attack tại mọi endpoint Authentication (IP-RateLimit 5req/10m tại Login, 3req/10m tại Forgot Password).
  - Handler (Email Focus): Ngăn chặn thuật toán vét cạn Password (Brute-force) với rule block 15 phút nếu nhập sai cho 1 email 5 lần trong 15 phút.