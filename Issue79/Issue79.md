# 1. Tổng quan & Bối cảnh (Overview & Context)

## 1.1 Vấn đề (Problem)
Hệ thống IOC có mục tiêu kết nối sinh viên với các doanh nghiệp (Enterprise). Tuy nhiên, hiện tại nền tảng chưa có module để Ban quản trị (Sup Admin) quản lý tập trung hệ sinh thái doanh nghiệp này. Việc thiếu một "Admin Control Panel" dẫn đến: 
- Việc onboard (đăng ký mới) doanh nghiệp bị rải rác, không được xác thực, không có đầu mối cấp tài khoản chuẩn.
- Gặp khó khăn trong việc rà soát, theo dõi xem doanh nghiệp nào đang hoạt động hợp tác (`status = 1`) hay đã tạm ngừng (`status = 2`).
- Rủi ro gán nhầm sinh viên vào các công ty không còn khả năng tiếp nhận do dữ liệu công ty là dữ liệu "chết", không được quản lý states.

## 1.2 Giá trị Nghiệp vụ (Business Value)
- **Chuẩn hóa quy trình Onboarding**: Cung cấp công cụ để Sup Admin duyệt và tự tay cấu hình mở kết nối cho một doanh nghiệp mới tham gia nền tảng (Kéo theo việc sinh tài khoản Enterprise HR đầu tiên).
- **Trạm điều khiển tập trung (Single Source of Truth)**: Sup Admin và Uni Admin dễ dàng tra cứu, lọc thông tin và chẩn đoán sự cố cho bất cứ công ty nào.
- **Bảo mật và Kiểm soát Cửa ngõ**: Nút chặn (Toggle Status) giúp Sup Admin vô hiệu hóa ngay lập tức các doanh nghiệp vi phạm hoặc dừng hợp tác. Giúp chặn đứng mọi hoạt động API của doanh nghiệp đó.

## 1.3 Đối tượng (Actor)
- **Primary Actor**: `SupAdmin` - Tốc quyền cao nhất, quản lý toàn bộ danh mục Doanh nghiệp.
- **Secondary Actor**: 
  - `UniAdmin`: Có thể dùng chung các hàm GET Lists để tra cứu thông tin (Read-only) khi cần đưa doanh nghiệp vào danh sách phân sinh viên của trường.

---

# 2. Luồng Người dùng (User Flow)

## 2.1 Luồng Xem Danh sách & Tra cứu
1. `SupAdmin` đăng nhập, truy cập menu **"Quản lý Doanh nghiệp"**.
2. Giao diện tải API lấy toàn bộ thông tin các công ty trên nền tảng (Lấy từ bảng `enterprises`).
3. Màn hình tự động hiển thị Table/DataGrid. Có các thanh tìm kiếm (Theo Tên, Mã số thuế) và Bộ lọc (Theo Ngành nghề, Trạng thái Hoạt động).

## 2.2 Luồng Tạo tài khoản Doanh nghiệp mới (Onboard) 
1. `SupAdmin` click nút **"Thêm mới Doanh nghiệp"**.
2. Điền thông tin profile công ty: `Tên`, `Mã số thuế`, `Lĩnh vực`.
3. Điền thông tin của người đại diện (Enterprise HR) để hệ thống cấp tài khoản: `Email`, `Họ Tên`, `SĐT`.
4. Submit Form. Hệ thống sẽ:
   - Insert vào bảng `enterprises`.
   - Insert vào bảng `users` với Role HR, và map vào `enterprise_users`.
   - Bắn email thông báo mật khẩu khởi tạo về cho `Email` người đại diện.
5. Giao diện bắn Toast "Tạo mới thành công" và reload danh sách.

## 2.3 Luồng Khóa / Kích hoạt (Update Status)
1. Tại danh sách, `SupAdmin` có thể bấm nút **"Ngừng hoạt động"** hoặc **"Khóa"** một doanh nghiệp đang Active.
2. Hệ thống bật Popup "Bạn có chắc chắn muốn ngưng hợp tác với công ty [XYZ]? Mọi tài khoản HR của công ty này sẽ bị cấm đăng nhập".
3. Xác nhận. Hệ thống API chạy lệnh Patch cập nhật `enterprises.status` thành Inactive/Suspended.

---

# 3. Tiêu chí Chấp nhận (Acceptance Criteria)

## AC-01: Hiển thị Grid danh sách
- **Given**: `SupAdmin` đăng nhập thành công.
- **When**: Vào menu "Quản lý Doanh nghiệp".
- **Then**:
  - Giao diện Table liệt kê: Tên Doanh nghiệp, Mã số thuế, Ngành nghề, Website, Ngày tham gia, và Cột Trạng thái (Active/Inactive/Suspended).
  - Có cơ chế Phân trang (Pagination) 10-20 dòng/trang.

## AC-02: Tạo mới (Onboard) Doanh nghiệp & HR
- **Given**: `SupAdmin` đang mở pop-up Thêm mới.
- **When**: Điền đủ các thông tin hợp lệ và nhấn "Tạo".
- **Then**:
  - Hệ thống tạo thành công Doanh nghiệp và 1 Account đại diện (Admin/HR của cty).
  - Tự động sinh password ngẫu nhiên và gửi 1 thư thông báo qua Email của HR.
  - Hiển thị Toast mầu xanh báo thành công.

## AC-03: Ràng buộc tính hợp lệ Mật độ (Validation)
- **Given**: Khung nhập Thêm mới / Chỉnh sửa.
- **When**: Nhập `TaxCode` đã tồn tại, hoặc `Email` người đại diện đã được đăng ký.
- **Then**:
  - Hệ thống validation từ cản lại, không gọi API, hoặc API trả lỗi **400 Bad Request**.
  - Hiển thị Text cảnh báo: *"Mã số thuế này đã được đăng ký"* hoặc *"Email này đã tồn tại trên hệ thống"*.

## AC-04: Khóa / Vô hiệu hóa Doanh nghiệp (Toggle Status)
- **Given**: Công ty A đang có trạng thái `Active`.
- **When**: `SupAdmin` bấm Suspend/Deactivate.
- **Then**:
  - Trạng thái dòng trên Grid đổi sang mầu xám/đỏ (Tương ứng Inactive/Suspended).
  - API trả về thành công 204.

## AC-05: Bảo mật Quyền Mức Hệ Thống
- **Given**: User là `Student` hoặc `EnterpriseHR`.
- **When**: Phóng POST Request thẳng lên API tạo mới Doanh nghiệp hoặc đổi Status.
- **Then**: API chặn đứng bằng mã **403 Forbidden**.

---

# 4. Đặc tả Kỹ thuật (Technical Specifications)

Dựa theo chuẩn **FFA Framework (IOCv2 Agent Skills)** và đối chiếu thiết kế **DB.md**:

## 4.1. Thiết kế API Endpoints (Bắt buộc)

| API Endpoint | Method | Path | Request Variables | Response Structure (JSON) | Status Codes & Messages |
| --- | --- | --- | --- | --- | --- |
| Lấy danh sách Doanh nghiệp | **GET** | `/api/admin/enterprises` | `searchTerm`, `status` (smallint, opt), `pageIndex`, `pageSize` | `{ data: [ { enterpriseId, name, taxCode, industry, status, createdAt } ], totalCount }` | **200 OK**<br>**403 Forbidden** |
| Xem chi tiết Doanh nghiệp | **GET** | `/api/admin/enterprises/{enterpriseId}` | Path: `enterpriseId` (uuid) | `{ enterpriseId, name, taxCode, industry, address, website, description, logoUrl, status, isVerified, adminUsers: [ { userId, email, fullName, position } ] }` | **200 OK**<br>**404 Not Found**<br>**403 Forbidden** |
| Tạo mới Doanh nghiệp (Kèm lập Account HR đại diện) | **POST** | `/api/admin/enterprises` | `{ name, taxCode, industry, address, website, hrFullName, hrEmail, hrPhone }` | `{ enterpriseId, message: "Created and email sent" }` | **201 Created**<br>**400 Bad Request**<br>**403 Forbidden** |
| Cập nhật Trạng thái (Suspend/Activate) | **PATCH** | `/api/admin/enterprises/{enterpriseId}/status` | `{ status: int }` (1=Active, 2=Suspended) | `204 No Content` | **204 No Content**<br>**404 Not Found**<br>**400 Bad Request** |

## 4.2. [DB Alignment] Đối chiếu Database
- **Mapping Dữ liệu Creation (API POST)**: 
  - Hệ thống sẽ cần mở Transaction:
    - Insert bảng `enterprises` $\to$ Cột `name` (varchar 255), `tax_code` (varchar 50), `industry` (varchar 150), `address`, `website`. Lấy ra `enterprise_id`.
    - Insert bảng `users` $\to$ Hash password ngẫu nhiên. Insert `full_name`, `email`, `phone_number`. Lấy ra `user_id`.
    - Insert bảng `enterprise_users` $\to$ Map `enterprise_id` và `user_id`, `position = 'Admin/HR'`.
  - Không tồn tại `[⚠️ DB Mismatch]`. Thiết kế DB hoàn toàn support luồng Master-Detail Identity này.

## 4.3. [FFA-ACV] & [FFA-TXG] Validation Rules
- **Regex & Length Guard (Bắt buộc Config FluentValidation)**:
  - `name`: Tuyệt đối NotEmpty, MaxLength = 255.
  - `taxCode` (Mã số thuế): Chỉ chứa chữ và số, không chứa ký tự đặc biệt, MaxLength = 50. Cần Rule Builder kết nối DB check Unique `tax_code`. Nếu trùng, báo lỗi `400 Bad Request` dạng `"taxCode": "Mã số thuế đã tồn tại"`.
  - `hrEmail`: Chạy validator `.EmailAddress()`. Check Unique qua bảng `users.email`.
- **Database Transaction Guard (FFA-TXG)**:
  - Hành động API POST bắt buộc phải bọc bằng `IDbContextTransaction`. Nếu rớt ở việc gửi Email hoặc lỗi Insert `users`, hệ thống phải Rollback lại lệnh Insert `enterprises`, tránh việc sinh ra công ty Rác không có người quản lý truy cập cục bộ.

## 4.4. [FFA-SEC] Authentication & Authorization
- **Ràng buộc Quyền hạn (RBAC)**:
  - Nhóm Route `/api/admin/enterprises` gắn Tag System Administrator: `[Authorize(Roles = "SupAdmin")]`. (Có thể cấp mở quyền GET list cho Uni Admin tùy Policy dự án).
- Nếu User không đủ Claim Policy, lập tức nhả `403 Forbidden`. Không trả `401` nếu đã Verify Bearer Token thành công.

## 4.5. [FFA-PERF] Cấu hình Rate Limit 
- **Read APIs Rate Limit (GET)**: Trả list tại Grid là thao tác nghiệp vụ trung bình khá. Giới hạn IP Tracking `60 requests / 1 phút / IP`.
- **Write/Mutate APIs Rate Limit (POST/PATCH)**: Việc Setup cty mới là công việc cực chậm. Cấm ddos qua spam Post: `10 requests / 1 phút / IP`.
- Phân trang List bằng `.Skip().Take()` đính kém `.AsNoTracking()` trong EF Core context. Không List AsEnumerable toàn bộ database bảng công ty.