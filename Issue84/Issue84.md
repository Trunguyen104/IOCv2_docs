# 1. Tổng quan & Bối cảnh (Overview & Context)

## 1.1 Vấn đề (Problem)
Thông tin doanh nghiệp (địa chỉ, website, mô tả, logo) là yếu tố quyết định để thu hút sinh viên ứng tuyển. Hiện tại, Enterprise HR chưa có giao diện tập trung để tự quản lý thông tin này, dẫn đến việc dữ liệu có thể bị lỗi thời hoặc phải phụ thuộc vào Uni Admin để cập nhật thủ công, gây tốn kém thời gian và giảm tính chủ động.

## 1.2 Giá trị Nghiệp vụ (Business Value)
- **Tính chủ động**: Cho phép Doanh nghiệp tự cập nhật thông tin thương hiệu, tuyển dụng theo thời gian thực.
- **Tăng tỷ lệ Match**: Hồ sơ đầy đủ, chuyên nghiệp giúp sinh viên có cái nhìn tin cậy, từ đó tăng số lượng và chất lượng hồ sơ ứng tuyển.
- **Dữ liệu chính xác**: Đảm bảo các thông tin liên hệ và địa chỉ luôn được cập nhật đúng để hỗ trợ quá trình quản lý sinh viên tại hiện trường.

## 1.3 Đối tượng (Actor)
- **Primary Actor**: Enterprise HR / Enterprise Admin - Xem và chỉnh sửa thông tin công ty của chính mình.
- **Secondary Actor**: Students / Uni Admin - Xem thông tin công ty (Read-only) để phục vụ việc chọn hướng thực tập hoặc quản lý.

---

# 2. Luồng Người dùng (User Flow)

## 2.1 Luồng Xem Hồ sơ Công ty (My Profile)
1. User (HR / Enterprise Admin) đăng nhập vào hệ thống và truy cập menu "Hồ sơ Công ty".
2. Hệ thống gọi API lấy thông tin dựa trên `EnterpriseId` gắn liền với tài khoản (được bóc tách từ Token).
3. Hiển thị thông tin tổng quan: Tên công ty, Mã số thuế, Logo, Hình nền, Website, Địa chỉ, Mô tả công ty và Lĩnh vực hoạt động.

## 2.2 Luồng Cập nhật Hồ sơ (Update Profile)
1. Tại trang hồ sơ, HR/Admin nhấn nút "Chỉnh sửa".
2. Hệ thống hiển thị Form cho phép HR thay đổi các trường thông tin hợp lệ. Các thông tin nhạy cảm của hệ thống như Mã số thuế (Tax Code) sẽ bị Disable (Read-only) để đảm bảo pháp lý.
3. HR hoàn thiện thông tin, nhấn "Lưu thay đổi".
4. Quá trình Submit thực hiện validate, ghi đè dữ liệu lên Database và trả kết quả cho HR (Hiển thị mầu xanh: Cập nhật thành công).

---

# 3. Tiêu chí Chấp nhận (Acceptance Criteria)

## AC-01: Hiển thị đầy đủ thông tin hồ sơ Doanh nghiệp
- **Given**: Enterprise HR / Admin đã đăng nhập thành công.
- **When**: Kích hoạt trang "Hồ sơ Công ty".
- **Then**:
  - API trả về đúng dữ liệu của Doanh nghiệp mình mà không tiết lộ mã UUID của ID lên URL trực diện (Bảo vệ thông tin).
  - Khung Logo và Background hiển thị dưới dạng hình ảnh, fall-back logo rỗng nếu chưa cập nhật.
  - Các thông tin khác hiển thị đúng cấu trúc (Hỗ trợ text paragraph cho Description).

## AC-02: Flow Cập nhật thông tin thành công
- **Given**: HR đang mở Form chỉnh sửa hồ sơ.
- **When**: Nhập Data Title, Website, Address an toàn và ấn "Lưu".
- **Then**:
  - Hệ thống BE ghi dữ liệu chuẩn mới đè lên và trả Status thành công.
  - Hiển thị Toast-message mầu xanh: *"Cập nhật hồ sơ thành công"*.
  - Render lại trang không cần tải lại toàn bộ browser, refresh đúng frame dữ liệu.

## AC-03: Xác thực dữ liệu đầu vào (Form Validation)
- **Given**: HR nhập dữ liệu sai.
- **When**: Gửi Request cập nhật cho Form.
- **Then**:
  - Nếu `website` truyền vào nhưng không hỗ trợ prefix chuẩn `http/https`: Trả thông báo lỗi cụ thể.
  - Nếu `name` (Tên công ty) bị xóa rỗng: Trả lỗi.
  - Hệ thống API phản xạ lỗi Validation thông qua mã **400 Bad Request** trả ngược dạng mảng `{"website": ["Không đúng định dạng URL"]}`.

## AC-04: Bảo mật & Chống IDOR (Security Rules)
- **Given**: Một HR của Công ty A am hiểu kĩ thuật.
- **When**: Tìm cách chặn bắt Postman, cố tình đổi Parameter hoặc Update dữ liệu chéo của Công ty B.
- **Then**:
  - Hệ thống BE hoàn toàn KHÔNG tin tưởng data truyền kiểu `{"enterpriseId": "..."}`.
  - BE tự quét Context Token bắt buộc khớp `EnterpriseId` nội vùng. Trả về mã lỗi **403 Forbidden** nếu cố tình gửi request trái thẩm quyền.

---

# 4. Đặc tả Kỹ thuật (Technical Specifications)

Dựa theo chuẩn **FFA Framework (IOCv2 Agent Skills)** và đối chiếu **DB.md**:

## 4.1. Thiết kế API Endpoints (Bắt buộc)

| API Endpoint | Method | Path | Request Body | Response Structure (JSON) | Status Codes & Messages |
| --- | --- | --- | --- | --- | --- |
| Xem hồ sơ công ty | **GET** | `/api/enterprises/me` | *Token Header* | `{ enterpriseId, taxCode, name, industry, description, address, website, logoUrl, backgroundUrl, isVerified }` | **200 OK**<br>**404 Not Found**<br>**403 Forbidden** |
| Phê duyệt cập nhật hồ sơ | **PUT** | `/api/enterprises/me` | `{ name, industry, description, address, website, logoUrl, backgroundUrl }` | `204 No Content` hoặc `{ message: "Cập nhật thành công" }` | **200 OK** / **204**<br>**400 Validation Bad Request**<br>**403 Forbidden** |

## 4.2. [⚠️ DB Mismatch] Cảnh báo Lệch chuẩn Mô hình ERD

So sánh Issue và ERD Table `enterprises`, phát sinh 2 lệch chuẩn cần thiết kế / DBA khắc phục cực kì quan trọng:
**Thiếu thông tin liên hệ của Công ty (Phone, Email tổng đài)**:
   - File yêu cầu cũ ghi *"Các thông tin liên hệ (Email, Phone) được format chuẩn"*. Tuy nhiên, bảng `enterprises` hoàn toàn không có cột lưu `phone` hay `email` (chỉ bảng User là người dùng mới có cất giữ cá nhân).
   - **Xử lý**: Lược bỏ Email và Phone ở Profile công ty trừ khi DBA thiết kế thêm bằng alter table. (Trong tài liệu này đã chỉnh lại khớp hoàn toàn theo ERD - không cho phép sửa Email/Phone).

## 4.3. [FFA-ACV] Quy tắc Validation & Business Guard
- **TaxCode Lock (Mã số thuế read-only)**:
  - Request `PUT` không phơi API (không cho nhận) field `taxCode`. Nếu nhận JSON trôi nổi chứa `taxCode`, Validator tự drop (Ignored) để không vô tình sửa MST của Cty do sai lầm. Tính chất Pháp lý phải Fix cứng.
- **Validation Max Length & Types**:
  - `name`: Cần NotEmpty & MaxLength = 255.
  - `industry`: MaxLength = 150.
  - `description`: Không giới hạn Text trong Postgres, nhưng có thể Set FluentRule giới hạn = 3000 chars.
  - `website`: Dùng Regex `^(http|https)://` kết hợp độ dài dưới 255 char.
  - `address`: MaxLength = 500.

## 4.4. [FFA-SEC] Authentication & Authorization
- **Ràng buộc Quyền hạn (RBAC)**:
  - Dùng thẻ `[Authorize(Roles = "EnterpriseAdmin, EnterpriseHR")]` để chặn đứng các tác vụ ngầm từ Sinh viên gọi URI `/api/enterprises/me`.
- **Ownership Identity (Chống IDOR)**:
  - Tại tầng Command Handler, TẤT CẢ phải khai thác cái lõi của Claim Token. Đọc và lấy ra `User.EnterpriseId` làm Key Where Update SQL duy nhất. Không được truyền biến Id giả vào DTO từ Body.

## 4.5. [FFA-PERF] Cấu hình Rate Limit 
- **Read APIs Rate Limit (GET)**: URL `/me` lấy Profile lúc HR load trang quản trị (Load Page/Refresh Data). Mức cấu hình là `60 requests / 1 phút / IP`.
- **Write/Mutate APIs Rate Limit (PUT)**: Nghiệp vụ thay đổi nội dung profile rất hiếm khi diễn ra, cấu hình Block Limit vô cùng gắt: **5 requests / 1 phút / IP** để chống Script tự động spam ddos Update Query ngốn I/O Write của Database Server.