# 1. Tổng quan & Bối cảnh (Overview & Context)

## 1.1 Vấn đề (Problem)
Hiện tại, sinh viên (Student) đang thiếu một quy trình chuẩn hóa để tìm kiếm, ứng tuyển và quản lý các cơ hội thực tập. Tương tự, hệ thống thiếu luồng trạng thái trung gian (như Interviewing, Offered) và các chức năng tải lên tài liệu cá nhân (CV, Cover Letter), khiến quá trình tuyển dụng giữa Doanh nghiệp và Sinh viên trở nên thiếu minh bạch và thủ công.

## 1.2 Giá trị Nghiệp vụ (Business Value)
- **Minh bạch và Xuyên suốt:** Cung cấp trải nghiệm ứng tuyển liền mạch cho sinh viên từ lúc xem tin tuyển dụng (Job Posting), tải CV, đến việc chốt Offer.
- **Tăng tỷ lệ chuyển đổi:** Tính năng Bookmark và quản lý Portfolio giúp Sinh viên dễ dàng theo dõi cơ hội.
- **Chuẩn hóa luồng trao đổi:** Các trạng thái `Applied` $\to$ `Interviewing` $\to$ `Offered` $\to$ `Hired` / `Rejected` / `Withdrawn` giúp hệ thống bắt trọn toàn bộ vòng đời tuyển dụng.

## 1.3 Đối tượng (Actor)
- **Primary Actor:** `Student` - Tìm kiếm, lưu thông tin, ứng tuyển, quản lý lịch phỏng vấn, nhận và xử lý offer.
- **Secondary Actor:** 
  - `EnterpriseHR` - Gửi lịch phỏng vấn, gửi Offer, hoặc Reject.
  - `UniAdmin` - Xem (Read-only) và override nếu có sự vụ khẩn cấp (Can thiệp luồng).

---

# 2. Luồng Người dùng (User Flow)

## 2.1 Luồng Ứng tuyển & Lưu tin
1. Sinh viên vào trang **Explore Jobs**, xem các Job Postings đang Open.
2. Sinh viên có thể **Bookmark** tin để xem lại sau.
3. Sinh viên nhấp **Apply**, một Form hiện ra bắt buộc đính kèm CV và tùy chọn Cover Letter.
4. Nộp đơn thành công, hệ thống chuyển sang trạng thái **Applied/In Review**. Application xuất hiện ở trang **Manage Applications**.

## 2.2 Luồng Phỏng vấn & Offer
1. Enterprise HR xem đơn và chọn phỏng vấn, đẩy trạng thái lên **Interviewing**, hệ thống bắn thông báo yêu cầu xác nhận.
2. Sinh viên thấy tác vụ ở Application: Chọn **Confirm Time** (Xác nhận) hoặc **Reschedule** (Xin đổi lịch kèm lý do), hoặc **Withdraw** (Rút đơn kèm lý do).
3. Sau phỏng vấn, HR quyết định cấp offer, chuyển trạng thái qua **Offered**. Sinh viên có thể **Accept** (Thành Hired) hoặc **Decline** (Thành Rejected).

## 2.3 Quản lý Hồ sơ Sinh viên
1. Sinh viên vào phần **Manage CV & Portfolio** trên hồ sơ cá nhân.
2. Cập nhật CV, Link Portfolio. Có công tắc **Toggle Privacy** để thu hồi quyền xem CV từ hệ thống doanh nghiệp nếu cần.

---

# 3. Tiêu chí Chấp nhận (Acceptance Criteria)

## AC-01: Xem danh sách Job Posting
- **Given:** Sinh viên đăng nhập, kỳ thực tập đang Active.
- **When:** Sinh viên truy cập trang "Explore Jobs".
- **Then:**
  - Hiển thị danh sách Job Posting trạng thái Open.
  - Nếu sinh viên đã có nơi thực tập (Hired), ẩn danh sách và hiển thị *"Bạn đã có nơi thực tập"*.
  - Cho phép click nút Bookmark để lưu tin.

## AC-02: Apply CV — Form & Validation
- **Given:** Sinh viên gọi Form Apply.
- **When:** Bấm Submit.
- **Then:**
  - Nếu thiếu CV, bắn lỗi *"Vui lòng upload CV"*.
  - File nếu sai quy cách (chỉ hỗ trợ PDF, DOC), bắn lỗi **400 Bad Request**.
  - Nếu hợp lệ, hệ thống tạo Application, chuyển sang **Applied/In Review**.

## AC-03: Trạng thái Tương tác Application
- **Given:** Application ở trang "Manage Applications".
- **When:** Sinh viên tương tác với Application.
- **Then:** Action hiển thị phụ thuộc vào Status:
  - **Applied:** `Withdraw`
  - **Interviewing:** `Confirm Time`, `Reschedule`, `Withdraw`.
  - **Offered:** `View Offer`, `Accept`, `Decline`.
  - **Hired/Rejected:** `Delete Record` (Soft delete giao diện).

## AC-04: Reschedule / Withdraw (Giai đoạn Interviewing)
- **Given:** Application ở trạng thái Interviewing.
- **When:** Sinh viên bấm Withdraw hoặc Reschedule.
- **Then:** Hệ thống bắt buộc nhập Lý do (Reason). Lưu lý do và bắn Notification cho HR/UniAdmin.

## AC-05: Nhận và Xác nhận Offer
- **Given:** Đơn ở trạng thái Offered.
- **When:** Sinh viên bấm "Accept".
- **Then:**
  - Chuyển application qua **Hired**.
  - Không cho phép apply thêm các Job khác.
  - Tự động thay đổi `internship_status` tại bảng `students`.

---

# 4. Đặc tả Kỹ thuật (Technical Specifications)

Dựa theo chuẩn **FFA Framework (IOCv2 Agent Skills)** và đối chiếu **DB.md**:

## 4.1. Thiết kế API Endpoints

| API Endpoint | Method | Path | Request Body / Query | Response Structure (JSON) | Status Codes & Messages |
| --- | --- | --- | --- | --- | --- |
| Lấy danh sách Job | **GET** | `/api/jobs` | `?termId={id}&limit=10&page=1` | `{ data: [{ jobId, title, enterpriseName... }], total }` | **200 OK** |
| Bookmark Job | **POST** | `/api/jobs/{jobId}/bookmark` | *Empty* | `204 No Content` | **204** / **404 Not Found**<br>**401 Unauthorized** |
| Lấy My Applications | **GET** | `/api/applications/me` | *Token Header* | `[{ applicationId, jobId, status, appliedAt }]` | **200 OK** |
| Nộp đơn (Apply) | **POST** | `/api/applications` | `{ jobId, cvUrl, coverLetterUrl, note }` | `{ applicationId, message: "Applied" }` | **201 Created**<br>**400 Bad Request**<br>**403 Forbidden** |
| Cập nhật trạng thái (Interact) | **PATCH** | `/api/applications/{id}/status` | `{ action: "Accept/Decline/Withdraw", reason }` | `204 No Content` | **204** / **400** / **404**<br>**422 Unprocessable** |
| Bật/Tắt Privacy | **PATCH** | `/api/students/me/privacy` | `{ isPublic: boolean }` | `204 No Content` | **204** / **403 Forbidden** |

## 4.2. [⚠️ DB Mismatch] Cảnh báo Lệch chuẩn Mô hình ERD

Dựa theo `DB.md` hiện tại, hệ thống **CHƯA SẴN SÀNG** đáp ứng các tính năng trong Issue 90. Các bảng và cột đang bị thiếu hụt nghiêm trọng:
1. **Thiếu Job Postings:** Không có bảng nào định nghĩa "tin tuyển dụng" (Job Posting / JD). Bảng `internship_groups` hay `projects` không đáp ứng được khái niệm mô tả công việc (Job Title, Requirements).
2. **Thiếu thông tin Bookmark:** Không có bảng `student_bookmarks` để ánh xạ quan hệ N-N giữa `students` và `Job Posting`.
3. **Bảng `internship_applications` chưa đủ thông tin:**
   - Cột `status` chỉ có `0=Pending, 1=Approved, 2=Rejected, 3=Withdrawn`. Hoàn toàn thiếu các trạng thái `Interviewing`, `Offered`, `Hired`.
   - Cột lưu `cv_url`, `cover_letter_url` cho việc nộp đơn: **Không tồn tại**.
   - Cột lưu `reason` (lý do từ chối/rút đơn/đổi lịch): **Không tồn tại**.
4. **Thiếu quản lý Lịch Phỏng vấn (Interview Schedules):** Không có bảng lưu thời gian hẹn phỏng vấn, trạng thái lịch hẹn.
5. **Thuộc tính Profile Sinh viên:** Bảng `students` **thiếu cột** `cv_url`, `portfolio_url` và `is_profile_public` (Privacy Toggle). Cột `internship_status` có trạng thái: `0=NoInternship, 1=Applied, 2=Onboarded, 3=Completed`, thiếu trạng thái trung gian `Offered/Interviewing`.

> **Khuyến nghị Đặc tả Kiến trúc:** Database Admin cần thiết kế bổ sung bảng `job_postings`, `student_bookmarks`, cấu trúc lại trạng thái Enum của bảng `internship_applications` (Thêm cột URL, note, reshedule logic) trước khi Backend triển khai API.

## 4.3. [FFA-ACV] Quy tắc Validation & Business Guard
- **File Upload:** Backend phải cấu hình Antivirus scan hoặc MIME type check (chỉ cho phép `application/pdf`, `application/msword`). Giới hạn MaxSize = 5MB. Trả **400 Bad Request** nếu sai.
- **Workflow State Guard:** Hành động `PATCH /status` bắt buộc chạy qua State Machine. Không cho phép nhảy trạng thái (ví dụ: đang `Applied` không thể gọi Accept action). Nếu vi phạm, trả **422 Unprocessable Entity**.
- **Reason Requirement:** Với action `Withdraw` từ `Interviewing`, `reason` là `[Required]`. Cần dùng FluentValidation `RuleFor(x => x.Reason).NotEmpty().When(x => ...)` để bắt logic.

## 4.4. [FFA-SEC] Authentication & Authorization
- Toàn bộ hành vi Apply/Withdraw/Accept/Decline của Application bắt buộc dùng ID User từ Token để bảo vệ Dữ liệu (**Chống IDOR**): Phải query `WHERE student_id = :Me`.
- Các Controller GET Job/ POST Application gắn `[Authorize(Roles = "Student")]`.

## 4.5. [FFA-PERF] Cấu hình Rate Limit
- **Apply CV (POST `/api/applications`)**: Cực kỳ nghiêm ngặt, chống Spam spam rác hồ sơ. Cấu hình Rate Limit: **3 requests / 5 phút / IP & UserID**.
- **Upload File endpoint**: Có khả năng ngốn băng thông/Storage. Thiết lập **5 requests / phút / IP**.
- Tải danh sách JD (GET `/api/jobs`): Giới hạn **100 requests / 1 phút / IP**. Bắt buộc dùng Pagination `Limit / Offset` tránh tràn RAM hệ thống.
