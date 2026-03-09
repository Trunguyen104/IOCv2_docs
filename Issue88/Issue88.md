# 1. Tổng quan & Bối cảnh (Overview & Context)

## 1.1 Vấn đề (Problem)
Trong quá trình thực tập, có thể phát sinh các trường hợp sinh viên vi phạm nội quy doanh nghiệp (đi trễ, vắng mặt, thái độ không chuyên nghiệp, v.v.). Hiện tại, hệ thống chưa cung cấp công cụ để Doanh nghiệp (Enterprise Admin/HR/Mentor) ghi nhận, theo dõi và xử lý các vi phạm này, gây khó khăn trong việc đánh giá ý thức kỷ luật của sinh viên và thiếu căn cứ để phản hồi về cho Nhà trường.

## 1.2 Giá trị Nghiệp vụ (Business Value)
- **Minh bạch hóa & Số hóa kỷ luật:** Giúp Doanh nghiệp dễ dàng ghi nhận, quản lý và lưu trữ hồ sơ vi phạm của từng sinh viên thực tập.
- **Tương tác đa chiều:** Hỗ trợ tính năng bình luận (comment) cho phép các bên (HR, Mentor, Sinh viên) trao đổi trực tiếp trên từng báo cáo vi phạm, làm rõ thông tin.
- **Theo dõi tiến độ xử lý:** Hệ thống hóa các trạng thái xử lý vi phạm (Chờ xử lý, Đang xử lý, Đã xử lý, Không vi phạm) đảm bảo mọi sự cố đều được giải quyết trọn vẹn.
- **Báo cáo chuẩn xác:** Giúp xuất danh sách vi phạm làm minh chứng cho quá trình đánh giá (Evaluation) và gửi báo cáo về nhà trường khi cần thiết.

## 1.3 Đối tượng (Actor)
- **Primary Actor:** `EnterpriseAdmin` / `EnterpriseHR` / `Mentor` - Quản lý, tạo và chỉnh sửa báo cáo; Export dữ liệu.
- **Secondary Actor:** 
  - `Student`: Xem vi phạm của cá nhân, phản hồi (comment).
  - `UniAdmin`: Không có quyền truy cập ở module này (bị cấm 403 Forbidden theo đặc tả).

---

# 2. Luồng Người dùng (User Flow)

## 2.1 Luồng Quản lý Danh sách Báo cáo
1. HR/Mentor đăng nhập, vào "Quản lý Báo cáo Vi phạm". 
2. Hệ thống hiển thị Data Grid chứa danh sách các báo cáo. Mentor chỉ thấy báo cáo nhóm mình, HR/Admin thấy toàn bộ công ty.
3. Người dùng tìm kiếm, lọc (theo Trạng thái, Loại vi phạm, Thời gian) và sử dụng chức năng "Export" để xuất Excel/PDF.

## 2.2 Luồng Tạo & Xử lý Vi phạm
1. Chọn "Tạo báo cáo vi phạm". Hệ thống hiển thị Form.
2. Mentor chọn Sinh viên trong nhóm quản lý của mình. Nhập Loại vi phạm, Mức độ, Ngày xảy ra (không vượt quá hiện tại), Mô tả và tải File đính kèm.
3. Gửi thành công, Report mang trạng thái "Chờ xử lý". Sinh viên nhận được Notification.
4. Quá trình giải quyết: HR/Mentor thay đổi mức độ hoặc chuyển trạng thái sang "Đang xử lý / Đã xử lý / Không vi phạm".

## 2.3 Luồng Phản hồi (Bình luận)
1. Trong chi tiết Báo cáo, có một khoang thảo luận.
2. Sinh viên hoặc Mentor có thể nhập Comment để thanh minh, làm rõ sự việc.
3. Người tạo có thể chỉnh sửa/xóa comment của mình (Admin xóa được tất cả). Mọi thao tác đều được UI/UX ghi nhận và thể hiện huy hiệu *(đã chỉnh sửa)*.

---

# 3. Tiêu chí Chấp nhận (Acceptance Criteria)

## AC-01: Xem Danh sách & Phân quyền Dữ liệu
- **Given:** User truy cập danh sách Báo cáo.
- **When:** Hệ thống truy vấn dữ liệu.
- **Then:**
  - Admin/HR xem tất cả sinh viên toàn công ty.
  - Mentor **chỉ** xem được các report của Sinh viên thuộc internship group do Mentor đó phụ trách.
  - Hỗ trợ sắp xếp (Sort) cột và phân trang 10/20/50.

## AC-02: Tạo Báo cáo Vi phạm & Validation
- **Given:** User bấm "Tạo báo cáo" và nhập liệu.
- **When:** Click Submit.
- **Then:**
  - **Ngày xảy ra:** Phải $\le$ Ngày hiện tại, và $\ge$ Ngày bắt đầu thực tập của sinh viên.
  - Nếu loại vi phạm là "Khác" (Others), bắt buộc có Mô tả.
  - Mentor cố nhập báo cáo cho Sinh viên ngoài nhóm $\to$ Bắn lỗi 403 Forbidden.
  - Thành công: Gửi Toast "Tạo báo cáo thành công", bắn Notification tới Sinh viên.

## AC-03: Chỉnh sửa & Xóa (Ownership Policy)
- **Given:** Xem chi tiết một Báo cáo đã tạo.
- **When:** Thực hiện Update / Delete.
- **Then:**
  - HR/Mentor chỉ được sửa/xóa Báo cáo do **chính mình tạo**. Admin có thể sửa/xóa tất cả.
  - Nếu báo cáo đã có Comment sinh viên, Xóa phải hiển thị Cảnh báo mạnh về việc mất dữ liệu liên đới.
  - HR/Mentor không thể Reopen báo cáo đã "Đã xử lý" (Chỉ Admin mới có quyền Reopen kèm lý do).

## AC-04: Thảo luận / Comment
- **Given:** Khu vực Comment trong Detail view.
- **When:** Ghi và Gửi phản hồi.
- **Then:**
  - Nội dung rỗng không cho gửi. Nút gửi Disabled.
  - Tối đa 1000 ký tự. 
  - Lưu thành công $\to$ Bắn Notification cho các bên liên quan.
  - Sửa comment không đổi nội dung $\to$ Disabled. Chỉnh sửa xong hiện label *(đã chỉnh sửa)*.

---

# 4. Đặc tả Kỹ thuật (Technical Specifications)

Dựa theo chuẩn **FFA Framework (IOCv2 Agent Skills)** và đối chiếu **DB.md**:

## 4.1. Thiết kế API Endpoints

| API Endpoint | Method | Path | Request Body / Query | Response Structure (JSON) | Status Codes & Messages |
| --- | --- | --- | --- | --- | --- |
| Lấy danh sách Report | **GET** | `/api/violation-reports` | `?page, size, status, type, severity, sortBy` | `{ data: [{ reportId, studentName... }], total }` | **200 OK** |
| Xem chi tiết Report | **GET** | `/api/violation-reports/{id}` | Path: `id` (uuid) | `{ reportId, type, description, status, comments: [...] }` | **200 OK** / **404 Not Found**<br>**403 Forbidden** |
| Tạo Report vi phạm | **POST** | `/api/violation-reports` | `{ studentId, type, severity, incidentDate, description, attachments }` | `{ reportId, message: "Created" }` | **201 Created**<br>**400 Bad Request** / **403** |
| Cập nhật Report | **PUT** | `/api/violation-reports/{id}` | `{ type, severity, status, description, incidentDate, attachments }` | `204 No Content` | **204** / **400** / **404**<br>**403 Forbidden** |
| Xóa Report | **DELETE** | `/api/violation-reports/{id}` | *Empty* | `204 No Content` | **204** / **403 Forbidden** |
| Tạo Comment | **POST** | `/api/violation-reports/{id}/comments` | `{ content }` | `{ commentId, createdAt }` | **201 Created**<br>**400 Bad Request** |
| Sửa / Xóa Comment | **PUT / DEL** | `/api/violation-reports/{id}/comments/{cid}` | `{ content }` (Cho PUT) | `204 No Content` | **204** / **403 Forbidden** |

## 4.2. [⚠️ DB Mismatch] Cảnh báo Lệch chuẩn Mô hình ERD

Đối chiếu `DB.md`, hệ thống **HOÀN TOÀN KHÔNG CÓ** các table nào lưu thông tin về Quản lý Vi phạm (Violation Reports) và Phản hồi/Bình luận (Comments). Để triển khai tính năng theo Issue 88, DBA cần thực hiện Migration tạo mới các table sau:

1. **Table `violation_reports`**:
   - `report_id` [pk, uuid]
   - `student_id` [ref: > `students.student_id`]
   - `created_by` [ref: > `users.user_id`] (Lưu HR/Mentor/Admin tạo)
   - `internship_id` [ref: > `internship_groups.internship_id`] (Để dễ dàng map với Mentor và validate query phân quyền).
   - `violation_type` [smallint] (1=Đi trễ, 2=Vắng mặt, ...)
   - `severity` [smallint] (1=Nhẹ, 2=Trung bình, 3=Nghiêm trọng)
   - `incident_date` [date]
   - `description` [text]
   - `attachments` [jsonb] hoặc `[text]` (Lưu mạng file Url)
   - `status` [smallint] (1=Chờ xử lý, 2=Đang xử lý, 3=Đã xử lý, 4=Không vi phạm)
   - `timestamps`

2. **Table `report_comments`**:
   - `comment_id` [pk, uuid]
   - `report_id` [ref: > `violation_reports.report_id`]
   - `user_id` [ref: > `users.user_id`]
   - `content` [text]
   - `is_edited` [boolean] (Cờ để UI render chữ "đã chỉnh sửa")
   - `timestamps`

> **Kiến trúc dữ liệu:** Đây là Missing Tables cốt lõi. Cần bổ sung Schema ERD ngay lập tức. Dữ liệu Lookup như Loại vi phạm (`violation_type`) nên được hard-code ENUM tại tầng Application (C# Enum) theo đúng tinh thần MVP trong Issue, chưa cần thiết kế bảng Category riêng.

## 4.3. [FFA-ACV] Quy tắc Validation & Business Guard
- **Time Window Validation:** Ngày xảy ra (`incidentDate`) $\le$ `DateTime.UtcNow`. Cần lấy Metadata từ bảng `internship_groups` để check `incidentDate` $\ge$ `start_date` của Group đó. Nếu vi phạm, trả **400 Bad Request**.
- **Enum Guard:** Check cứng `violation_type`, `severity` nằm trong Range định nghĩa sẵn (Enum isDefined). Bắn lỗi 400 nếu truyền Type rác. 
- **Content MaxLength:** Bảng Comment có `content` maxlength = 1000 char.

## 4.4. [FFA-SEC] Authentication & Authorization
- Đây là trung tâm của việc check **Tầng Dữ liệu Data Ownership (Multi-tenancy)**:
  - Nếu Role là `EnterpriseMentor`: Cần bọc Query Command thêm `WHERE report.internship_id IN (SELECT internship_id FROM internship_groups WHERE mentor_id = :MyUserId)`.
  - Khâu Update/Delete Comment và Update Report: Bắt đầu hàm handler luôn phải gọi xác thực quyền tạo hóa: Dù người đó là Mentor hợp lệ, nhưng báo cáo này do HR khác tạo $\to$ Mentor cấm sửa/xóa (trả 403 Forbidden). Chỉ Tác giả (`created_by == me`) hoặc `EnterpriseAdmin` mới có quyền Update/Delete.
- API Route này cấm hoàn toàn Student gọi lên các method POST/PUT của `/violation-reports`, Student chỉ được gọi Route `GET /me/violation-reports` (Sẽ làm ở Issue riêng cho Student) hoặc POST `/comments`.

## 4.5. [FFA-PERF] Cấu hình Rate Limit 
- Thao tác POST tạo Report: **10 request / 1 phút / IP**.
- Thao tác Comments (Tránh Spam tin nhắn): **5 comments / 30 giây / UserID**. Quá số lượng chặn ngay tại Middleware 429 Too Many Requests.
- Export Data: Cực kỳ nặng I/O và Memory. Thiết lập **2 requests / 1 phút / IP**. Bắt buộc dùng background job nếu query report số lượng lớn, nhưng ở quy mô MVP, có thể Stream trả file trực tiếp với AsNoTracking().