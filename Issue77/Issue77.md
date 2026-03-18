# 1. Tổng quan & Bối cảnh (Overview & Context)

## 1.1 Vấn đề (Problem)
Sau khi Uni Admin thao tác phân công (Place) sinh viên vào thực tập tại doanh nghiệp, Enterprise HR cần một không gian quản lý để tiếp nhận, đánh giá và phân bổ nguồn lực (Mentor, Dự án) cho các sinh viên này. Nếu không có danh sách quản lý trạng thái, doanh nghiệp sẽ khó nắm bắt được số lượng sinh viên mình cần tiếp nhận, dẫn đến việc chậm trễ trong khâu phản hồi (Accept/Reject) và phân bổ công việc.

## 1.2 Giá trị Nghiệp vụ (Business Value)
- **Tối ưu quy trình tiếp nhận**: Giúp HR dễ dàng duyệt hoặc từ chối sinh viên do nhà trường gửi tới.
- **Phân bổ nguồn lực hiệu quả**: Quản lý việc gán Mentor và Vị trí/Dự án cụ thể cho từng cá nhân sinh viên.
- **Minh bạch thông tin**: Luồng thông báo (Notification) tự động giúp Uni Admin và Sinh viên nắm ngay kết quả sau khi doanh nghiệp ra quyết định.

## 1.3 Đối tượng (Actor)
- **Primary Actor**: 
  - **Enterprise HR / Admin**: Duyệt, từ chối, và chỉ định Mentor/Dự án.
  - **Mentor**: Xem danh sách các sinh viên đã được assign cho riêng mình.
- **Secondary Actor**: Sinh viên, Uni Admin (Nhận notification).

---

# 2. Luồng Người dùng (User Flow)

## 2.1 Luồng Quản lý và Tiếp nhận
1. **HR / Enterprise Admin** đăng nhập và truy cập trang "Danh sách sinh viên thực tập".
2. Hệ thống gọi API liệt kê tất cả sinh viên được Placed vào doanh nghiệp (thông qua các `internship_applications` hoặc mảng `internship` tương ứng của công ty), kèm Status (Pending, Accepted, Rejected).
3. Tại dòng của sinh viên đang chờ (`Pending`), HR chọn hành động:
   - **Accept**: Chuyển trạng thái sinh viên sang `Accepted`, hệ thống bắn Notify cho Student/Uni Admin.
   - **Reject**: Popup yêu cầu HR ghi rõ "Lý do từ chối". Chuyển trạng thái sinh viên sang `Rejected` và bắn Notify kèm lý do cho Uni Admin.

## 2.2 Luồng Phân bổ (Assign) Dành cho HR
1. Với các sinh viên đã được nhận (`Accepted`), HR bấm nút "Assign".
2. Popup chỉ định mở ra, yêu cầu HR chọn một `Mentor` (từ danh sách nhân viên công ty) và điền tên Vị trí/Dự án.
3. Submit thành công, dữ liệu Mentor và Project được gắn vào Sinh viên. Hệ thống thông báo cho Mentor và Student.

## 2.3 Luồng Xem Danh sách Dành cho Mentor
1. **Mentor** đăng nhập và cũng vào trang "Danh sách sinh viên thực tập".
2. API nhận diện Role Mentor qua Token, tự động Filter và chỉ trả về danh sách sinh viên mà Mentor này được gán quản lý (Ẩn toàn bộ sinh viên khác của công ty).
3. Mentor chỉ có quyền `Read-only` (Xem) danh sách và thông tin chi tiết, không được Accept/Reject/Assign.

---

# 3. Tiêu chí Chấp nhận (Acceptance Criteria)

## AC-01: Hiển thị danh sách sinh viên
- **Given**: HR/Admin/Mentor đã login và truy cập "Danh sách sinh viên thực tập".
- **When**: Dữ liệu tải lên màn hình.
- **Then**:
  - Giao diện dạng Table (Grid) liệt kê tất cả sinh viên được gán vào Enterprise trong kỳ hiện tại.
  - Các cột hiển thị: Họ tên, MSSV, Email, SĐT, Trường, Chuyên ngành, Trạng thái tiếp nhận, Mentor, Vị trí/Dự án.
  - Nếu không có data, in ra Empty State: *"Chưa có sinh viên nào được gán vào công ty"*.

## AC-02: Flow Phê duyệt (Accept) Sinh viên
- **Given**: Sinh viên ở trạng thái **Pending**.
- **When**: HR click nút "Accept" và Confirm xác nhận.
- **Then**:
  - Trạng thái DB nhảy sang **Accepted**.
  - Gửi Notification báo về cho Uni Admin và Student.
  - UI quăng Toast màu xanh: *"Đã tiếp nhận sinh viên thành công"*.

## AC-03: Flow Từ chối (Reject) Sinh viên
- **Given**: Sinh viên ở trạng thái **Pending**.
- **When**: HR click nút "Reject".
- **Then**:
  - Bung Dialog bắt buộc HR nhập **Lý do từ chối** (Bắt buộc).
  - Xác nhận xong, trạng thái sang **Rejected**.
  - Gửi Notification kèm *"Lý do"* nảy về UniAdmin & Student.

## AC-04: Flow Phân bổ (Assign) Mentor và Dự Án
- **Given**: Sinh viên đã mang trạng thái **Accepted**.
- **When**: HR bấm "Assign" và điền thông tin (Dropdown Mentor, Tên Dự án) rồi Lưu.
- **Then**:
  - Dữ liệu Mentor và Dự án được cập nhật vào dòng của Sinh viên.
  - Notification đẩy tới Mentor (để biết có lính mới) và Student (để biết sư phụ mình là ai).
  - *Lưu ý: Nút Assign này chỉ HR/Admin nhìn thấy, Mentor không có quyền bấm.*

## AC-05: Tìm kiếm, Bộ lọc & Sắp xếp (Search, Filter, Sort)
- **Given**: Table đang hiển thị dữ liệu lớn.
- **When**: Người dùng thao tác nghiệp vụ tìm kiếm nâng cao.
- **Then**:
  - **Tìm kiếm**: Debounce 300ms trên Text (Họ tên, MSSV, Email).
  - **Lọc (Filter)**: Multi-select hỗ trợ lọc theo Trạng thái (Pending/Accepted/Rejected), Theo Trường/Ngành, và Theo trạng thái "Đã có/Chưa có Mentor".
  - **Sắp xếp (Sort)**: Hỗ trợ Sort theo Tên (A-Z) và Ngày Placed (Mới/Cũ). Mặc định là Mới nhất lên đầu.
  - **Phân trang (Pagination)**: 10, 20 hoặc 50 row/trang.

## AC-06: Phân Quyền & Giới hạn dữ liệu (Row Level Security)
- **Given**: Người dùng thao tác gọi API.
- **When**: Quyền là Admin/HR hoặc Mentor.
- **Then**:
  - `EnterpriseAdmin` / `HR`: Xem được bảng danh sách full của toàn ty, được thao tác Approve/Reject/Assign.
  - `Mentor`: Chỉ xem được đúng các sinh viên mà hệ thống đã assign ID của Mentor đó. Các quyền Edit bị khóa (UI tắt nút).
  - `Student` / `UniAdmin`: Cấm gọi API GET danh sách của công ty (Trả HTTP `403 Forbidden`).

---

# 4. Đặc tả Kỹ thuật (Technical Specifications)

Dựa theo chuẩn **FFA Framework (IOCv2 Agent Skills)** và ERD **DB.md**:

## 4.1 Danh sách API Endpoints

| API Endpoint | Method | Path | Request Body / Query | Response Structure (JSON) | Status Codes & Messages |
| --- | --- | --- | --- | --- | --- |
| Lấy danh sách Sinh viên của Doanh nghiệp | **GET** | `/api/enterprises/me/applications` | Query: `pageIndex`, `pageSize`, `searchTerm`, `status`, `mentorAssigned` (bool) | `{ data: [ { applicationId, studentId, fullName, studentCode, universityName, major, status, reason, mentorName, projectName, createdAt } ], totalCount }` | **200 OK**<br>**401 Unauthorized**<br>**403 Forbidden** |
| Xem chi tiết một Application | **GET** | `/api/enterprises/me/applications/{applicationId}` | Path: `applicationId` (uuid) | `{ applicationId, student: {...}, status, reason, logs: [...] }` | **200 OK**<br>**404 Not Found**<br>**403 Forbidden** |
| Phê duyệt sinh viên (Accept) | **PATCH** | `/api/enterprises/me/applications/{applicationId}/accept` | Rỗng | Chuyển status = 1 (Approved) | **204 No Content**<br>**400 Bad Request** |
| Từ chối sinh viên (Reject) | **PATCH** | `/api/enterprises/me/applications/{applicationId}/reject` | Body: `{ reason: string }` | Chuyển status = 2, lưu reason | **204 No Content**<br>**400 Bad Request** |
| Phân bổ Mentor và Project | **PATCH** | `/api/enterprises/me/applications/{applicationId}/assign` | Body: `{ mentorId: uuid, projectName: string }` | Map project & mentor vào internship_group | **204 No Content**<br>**400 Bad Request** |

## 4.2 [⚠️ DB Mismatch] Cảnh báo Kiến trúc Dữ liệu

Dự vào cấu trúc `DB.md`, có **2 điểm sai lệch** khổng lồ cần DBA và Backend giải quyết trước khi Code:

1. **Thiếu cột lưu Lý do từ chối**:
   - Issue yêu cầu (AC-03) bắt buộc HR nhập "Lý do từ chối", và Notification đẩy về cho Uni Admins. Nhưng bảng `internship_applications` KHÔNG CÓ cột `reason` hay `reject_reason`.
   - *Hướng khắc phục*: Thêm cột `reject_reason varchar(500) null` vào `internship_applications`. Hoặc Backend phải tự lưu vào cột `reason` của bảng `audit_logs` sau đó query siêu lằng nhằng.

2. **Nghịch lý Gán Mentor (Null constraint)**:
   - Theo Flow, Uni Admin gán sinh viên vào Công ty (Trạng thái Pending) $\to$ HR Approve $\to$ HR gán Mentor.
   - Quá trình này được map vào `internship_groups` và `internship_applications`. Tuy nhiên, trong DB, `internship_groups.mentor_id` được set **[not null]**. Điều này có nghĩa là KHÔNG THỂ tạo Group thực tập cho công ty nếu HR chưa chọn Mentor! Vậy lúc Uni Admin gán SV vào cty ở trạng thái Pending, record đó tham chiếu `internship_id` kiểu gì?
   - *Hướng khắc phục*: DBA cần đổi `mentor_id` thành Nullable (Có thể Null), để UniAdmin tạo ra 1 Group rỗng chờ HR assign Mentor sau (Hoặc đổi cấu trúc liên kết `internship_applications`).

## 4.3 [FFA-ACV] Quy tắc Validation

- **Từ chối (Reject)**:
  - Nếu Body gọi API Reject mà biến `reason` rỗng (Trắng hoặc Null) $\to$ validation trả ngay **400 Bad Request** ngưng Pipeline: `{"reason": "Lý do từ chối là bắt buộc."}`. Text MaxLength: 500 ký tự.
- **Phân bổ (Assign)**:
  - `mentorId` check theo Regex GUID chuẩn, và phải query dưới DB chắc chắn `mentorId` này thuộc về bảng `enterprise_users` của CÙNG công ty hiện tại. Nếu truyền bừa Guid, bắn `400 Bad Request`: `"mentorId": "Mentor không tồn tại hoặc không thuộc công ty."`.
  - `projectName` (MaxLength: 255): Nếu truyền lên rỗng, bắn `400`.

## 4.4 [FFA-SEC] Authentication & Authorization

- **RBAC Limit**:
  - Nhóm thao tác **PATCH** (Accept/Reject/Assign) bắt buộc đánh tag `[Authorize(Roles = "EnterpriseAdmin, EnterpriseHR")]`. Cấm Mentor nhảy vào sửa record qua Postman.
  - Phân luồng **GET**:
    - Trích xuất `Token.EnterpriseId`.
    - Lấy tất cả Application nơi `ig.enterprise_id == query_enterprise_id`.
    - Nếu JWT báo Role là `Mentor`: Backend TỰ ĐỘNG append đuôi query `.Where(x => x.MentorId == Token.EnterpriseUserId)` để ngắt dứt điểm việc Mentor xem hồ sơ của nhóm mentor khác.

## 4.5 [FFA-PERF] Cấu hình Rate Limit & Pagination

- **Danh sách (List API)**: Trang Search này có mức rải Request trung bình, cấu hình IP Rate Limit là **60 requests / 1 phút / IP**.
- **Change APIs (Accept/Reject/Assign)**: Chỉ khoảng **20 requests / 1 phút / IP**. Ngăn bot auto click nát DB.
- **Pagination**: Bắt buộc tuân thủ `.Skip().Take()`. TUYỆT ĐỐI append `.AsNoTracking()` trong EF Core khi thực hiện join qua vô số bảng (Application $\leftarrow$ Group $\leftarrow$ Students $\leftarrow$ Users) để in ra danh sách, nếu thiếu NoTracking, server sẽ bị Memory Leak do phình Context Object.
