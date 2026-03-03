## 1. Tổng quan & Bối cảnh (Overview & Context)

## Vấn đề (Problem):

Trong mô hình thực tập doanh nghiệp, việc báo cáo tiến độ hàng ngày (Logbook) là bắt buộc để Mentor và Nhà trường nắm bắt được sinh viên đang làm gì, gặp khó khăn gì. Tuy nhiên, việc báo cáo qua email hay chat thường trôi tin, khó thống kê và khó kiểm soát tính chuyên cần (nộp đúng hạn/muộn).

## Giá trị Nghiệp vụ (Business Value):

- **Kỷ luật & Chuyên nghiệp:** Rèn luyện thói quen báo cáo công việc định kỳ, đúng hạn cho sinh viên.

- **Theo dõi sát sao:** Giúp Mentor thấy rõ khối lượng công việc hàng ngày và phát hiện sớm nếu sinh viên bị tắc nghẽn (block).

- **Dữ liệu minh chứng:** Là cơ sở để đánh giá điểm thái độ và quá trình thực tập cuối kỳ.

## Đối tượng (Actor):

- **Primary Actor:** Sinh viên (Student) — người tạo, chỉnh sửa và nộp báo cáo.

- **Secondary Actors:** Mentor, HR — xem danh sách báo cáo.

---

## 2. Luồng Người dùng (User Flow)

## 2.1. Luồng Xem Danh sách Báo cáo

1. Student truy cập menu **"Báo cáo hàng ngày"** (Logbooks) của một dự án. Hệ thống gọi API `GET /api/projects/{projectId:guid}/logbooks`.

2. Hệ thống hiển thị danh sách các báo cáo đã nộp có phân trang.

3. Dữ liệu hiển thị bao gồm: Tên sinh viên (StudentName), Ngày báo cáo (DateReport), Tóm tắt công việc (Summary), Vấn đề gặp phải (Issue), Trạng thái (Status), và Số lượng công việc liên kết (TotalWorkItems).

4. Mặc định danh sách được sắp xếp theo thời gian tạo mới nhất, hoặc theo Tên sinh viên.

## 2.2. Luồng Tạo Báo cáo (Create Logbook)

1. Student nhấn nút **"Tạo báo cáo"**.

2. Hệ thống hiển thị Form tạo báo cáo. Student nhập:

   - **Ngày báo cáo (DateReport):** Bắt buộc, không được lớn hơn thời điểm hiện tại.
   - **Tóm tắt công việc (Summary):** Bắt buộc, tối đa 200 ký tự.
   - **Kế hoạch (Plan):** Bắt buộc, tối đa 200 ký tự.
   - **Vấn đề gặp phải (Issue/Blockers):** Tùy chọn, tối đa 200 ký tự.
  [Missing] - **WorkItems:** Chọn từ danh sách Issue đã tạo.

3. Nhấn **"Nộp báo cáo"**. Hệ thống gọi API `POST /api/projects/{projectId:guid}/logbooks`.

4. Hệ thống Backend (Domain Logic) tự động kiểm tra xem `DateReport` có trùng với ngày hiện tại (`CreatedAt`) không. Nếu trùng, gán trạng thái `PUNCTUAL` (Đúng hạn), ngược lại gán `LATE` (Muộn). Dữ liệu tính toán dựa trên Server Time.

## 2.3. Luồng Chỉnh sửa (Update) & Xóa (Delete)

1. **Sửa:** Student click vào một Logbook cụ thể → Form hiện lên. Student có thể cập nhật `Summary`, `Issue`, `Plan`, `DateReport` qua API `PUT /api/projects/{projectId}/logbooks/{logbookId}`. Khi gọi Update, Backend tự xác định lại trạng thái (Đúng hạn / Muộn) tương ứng.

2. **Xóa:** Student có thể thực hiện Soft Delete bản ghi qua API `DELETE /api/projects/{projectId}/logbooks/{logbookId}`.

## 2.4. Luồng Tìm kiếm & Lọc

1. **Lọc (Filter):** Student/Mentor có thể truyền tham số `Status` (tương ứng với Enum `LogbookStatus` dạng chuỗi) để lọc.

2. **Sắp xếp (Sort):** Chọn sắp xếp theo `studentname` hoặc `createdat` (asc/desc).

---

## 3. Tiêu chí Chấp nhận (Acceptance Criteria)

## AC-RPT-01 — Hiển thị Danh sách Báo cáo chuẩn

- **Given:** Student truy cập trang danh sách Logbooks.

- **When:** Hệ thống gọi API Lấy danh sách thành công.

- **Then:**

  - Hiển thị danh sách báo cáo bao gồm thông tin chi tiết: `StudentName`, `DateReport`, `Summary`, `Issue`, `Status` (Enum) và `TotalWorkItems`.
  
  - Danh sách hỗ trợ phân trang với `PageNumber` và `PageSize`.

## AC-RPT-02 — Tạo Báo cáo & Logic Validation

- **Given:** Student mở form thêm mới Logbook.

- **When:** Submit form gửi API Create.

- **Then:**

  - Đầu vào `ProjectId`, `Summary`, `Plan`, `DateReport` không được bỏ trống.
  
  - API Controller sử dụng FluentValidation trả về HTTP 400 nếu `Summary`, `Issue`, `Plan` > 200 ký tự, hoặc `DateReport` ở tương lai.
  
  - Nếu thành công, trả về HTTP 201 Created. Backend thực hiện ghi nhận Audit Log.

## AC-RPT-03 — Cập nhật Trạng thái tự động (Auto-Status Logic)

- **Given:** Student gửi yêu cầu tạo hoặc chỉnh sửa Logbook.

- **When:** Backend tiếp nhận thông tin lưu trữ.

- **Then:**

  - Status (PUNCTUAL hay LATE) phải được quyết định tại Domain Entity (`Logbook.DetermineStatus()`) bằng cách so sánh `DateReport.Date` và `CreatedAt.Date` (Server UTC time). Sẽ không cho phép User truyền tham số ép buộc thay đổi Status từ Client.

## AC-RPT-04 — WorkItem Relation Link (TotalWorkItems)

- **Given:** Sinh viên xem danh sách báo cáo.

- **When:** Một Logbook được trả về.

- **Then:**

  - Có trường chọn "Issue đã làm" cho phép search và chọn các task đang có trong dự án.
  [Missing] - Hiển thị số lượng Task liên kết với báo cáo trong trường `TotalWorkItems` (chỉ đếm, API Create/Update mặc định hiện tại chưa hỗ trợ móc nối thêm trực tiếp).

## AC-RPT-05 — Logic ngày nghỉ [Missing]

- **Given:** Sinh viên tạo báo cáo vào ngày nghỉ.

- **When:** Hệ thống ghi nhận báo cáo.

- **Then:**

  [Missing] - Hệ thống phải kiểm tra xem ngày báo cáo có phải là ngày nghỉ không.
  [Missing] - Nếu là ngày nghỉ, hệ thống phải đánh dấu báo cáo là "Nghỉ" và không tính là báo cáo hàng ngày.

## AC-RPT-06 — Bảo mật Endpoint

- **Given:** Bất kỳ endpoint Logbook nào.

- **When:** Đầu cuối nhận Request.

- **Then:**

  - API phải yêu cầu `[Authorize]`. User phải liên kết tới `Student` Entity. Trả về `ResultErrorType.Unauthorized` nếu profile không phải Student hợp lệ.

---

## 4. Đặc tả kỹ thuật (Technical Notes)

- **Domain Driven Design:** Entity `Logbook` áp dụng tốt `Factory Method` Pattern thông qua phương thức `Logbook.Create()` đóng gói hoàn chỉnh logic Status, thoả mãn kiến trúc FFA-CAG.
- **Transaction & Audit:** Transaction Blocked `BeginTransactionAsync`, `CommitTransactionAsync` kết hợp cùng `AuditLog` tạo vết lưu vết kiểm toán rất tốt (Thỏa mãn FFA-TXG).
- **Status Value:** `LogbookStatus` định nghĩa bằng Enum (`0: SUBMITTED`, `1: APPROVED`, `2: NEEDS_REVISION`, `3: PUNCTUAL`, `4: LATE`). Việc Filter Status dựa trên Enum Parse case-insensitive.
- **[⚠️ Architecture Violation]:** Tại Handler `GetLogbooksHandler`, API hiện tại cho phép lấy **toàn bộ** danh sách báo cáo của cả Project mà không giới hạn quyền, nghĩa là mọi nhóm / mentor trong dự án cũng có thể đọc trừ khi Controller/Handler bổ sung Custom Permission (VD check `StudentId` hoặc policy phù hợp). Tương tự, thao tác Delete không kiểm tra `LogbookId` đó có phải của sinh viên gọi hay không, dẫn đến rủi ro xoá chéo dữ liệu của bạn học. Cần bổ sung Role/RBAC hoặc Data Isolation validation.
- API Route: `GET|POST|PUT|DELETE /api/projects/{projectId:guid}/logbooks` là thiết kế URL RESTful chuẩn (chứa cha là project -> con là logbooks).
[Missing] - **Holiday Calendar** - Cần có danh sách ngày nghỉ của dự án để kiểm tra xem ngày báo cáo có phải là ngày nghỉ không.