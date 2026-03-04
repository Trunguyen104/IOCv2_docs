# 1. Tổng quan & Bối cảnh (Overview & Context)

## 1.1 Vấn đề (Problem)
Sau khi kỳ thực tập được tạo, hệ thống IOC cần một cơ chế để Uni Admin có thể đưa sinh viên vào kỳ thực tập một cách có kiểm soát:
- Số lượng sinh viên tham gia mỗi kỳ thực tập có thể lên đến hàng trăm, việc thêm thủ công từng sinh viên là không khả thi.
- Cần theo dõi rõ ràng sinh viên nào đã được gán doanh nghiệp (Placed) và sinh viên nào chưa (Unplaced) trong từng kỳ.
- Cần có khả năng rút sinh viên khỏi kỳ khi có sai sót import hoặc sinh viên được duyệt nghỉ phép.
- Thông tin enrollment của từng sinh viên cần được xem và kiểm tra chi tiết để phục vụ các luồng gán doanh nghiệp, đánh giá và báo cáo.

## 1.2 Pain Points hiện tại
- Chưa có cơ chế import hàng loạt sinh viên vào kỳ thực tập, gây tốn thời gian và dễ sai sót khi thêm thủ công.
- Không có màn hình tổng hợp để theo dõi trạng thái Placed/Unplaced của từng sinh viên trong kỳ.
- Không kiểm soát được sinh viên đã enroll nhầm kỳ hoặc cần rút ra do nghỉ phép.
- Dữ liệu enrollment bị rải rác, không đồng bộ, thiếu nơi xem chi tiết thông tin enrollment của từng sinh viên.

## 1.3 Giá trị Nghiệp vụ (Business Value)
- **Import hàng loạt hiệu quả**: Tiết kiệm thời gian với Excel import, giảm thiểu sai sót. Thêm tay thủ công khi cần thiết.
- **Theo dõi trạng thái tập trung**: Uni Admin nắm bắt nhanh tiến độ Placed/Unplaced để điều phối.
- **Dữ liệu enrollment chính xác**: Rút sinh viên linh hoạt giúp số liệu đánh giá/báo cáo luôn đúng thực tế.
- **Kiểm soát vòng đời**: Quản lý toàn vẹn thao tác enroll, xem chi tiết và gỡ bỏ sinh viên khỏi một Term.

## 1.4 Đối tượng (Actor)
- **Primary Actor**: Quản trị viên trường (Uni Admin) - Import, xem chi tiết, thống kê, rút/gỡ sinh viên, gán doanh nghiệp.
- **Secondary Actors**:
  - **Student (Sinh viên)**: Xem thông tin Enrollment của bản thân (Chỉ đọc).
  - **Enterprise HR / Mentor**: Xem sinh viên được gán (Placed) vào công ty mình (Chỉ đọc).

---

# 2. Luồng Người dùng (User Flow)

## 2.1 Luồng Import Sinh Viên Bằng Excel
1. Uni Admin vào chi tiết Term -> Tab "Danh sách Sinh viên" -> Click "Import Excel".
2. Tải file template mẫu hoặc chọn file `.xlsx` của trường.
3. Gửi file lên server. Hệ thống validate từng dòng, preview kết quả dòng lỗi và dòng thành công.
4. Xác nhận Import -> Hệ thống đưa hàng loạt sinh viên hợp lệ vào Term.

## 2.2 Luồng Thêm Sinh Viên Thủ Công
1. Uni Admin click nút "Thêm sinh viên" -> Form (Họ Tên, MSSV, Email, SĐT, Ngày sinh).
2. Điền thông tin -> Save. Server validate trùng mã sinh viên hoặc email.
3. Thành công: Ghi danh Student vào Term với trạng thái `Unplaced`.

## 2.3 Luồng Xem Chi Tiết & Chỉnh sửa / Gán Doanh nghiệp (Placement)
1. Trong danh sách (hỗ trợ phân trang, lọc theo Trạng thái/Chuyên ngành, tìm kiếm Debounce), Uni Admin click "Xem chi tiết" 1 sinh viên.
2. Form cho phép xem trạng thái Placed/Unplaced.
3. Để thay đổi sang Placed -> Select chọn Enterprise -> Lưu.
4. Có thể xem các Feedback giữa kỳ/cuối kỳ (Read-only từ góc nhìn Enterprise gửi lên).

## 2.4 Luồng Rút Sinh Viên (Withdrawn)
1. Uni Admin click chọn 1 hoặc nhiều sinh viên (Tick chọn). Nhấn "Rút sinh viên".
2. Hệ thống cảnh báo. Nếu là sinh viên đang `Placed`, hệ thống chặn lệnh rút, yêu cầu hủy Placed trước.
3. Trạng thái sinh viên được đổi thành `Withdrawn` (Soft Delete / Hủy hoạt động).
4. Cung cấp bộ lọc xem danh sách sinh viên đã rút để có thể "Khôi phục" (Re-enroll).

---

# 3. Tiêu chí Chấp nhận (Acceptance Criteria)

## AC-01: Upload & Process Import Excel
- **Given**: Uni Admin chọn file `.xlsx` (<5MB, <1000 records).
- **When**: Bấm Tải lên.
- **Then**: 
  - Preview kiểm tra Regex Email, validate Mã sinh viên không được trống, Ngày sinh hợp logic.
  - Các row trùng lặp Email/Mã sinh viên hoặc đã enroll thì báo thẻ Error, skip import dòng đó.

## AC-02: Thêm Sinh viên thủ công
- **Given**: Form thêm sinh viên thủ công đang mở.
- **When**: Nhập liệu đầy đủ, click Lưu.
- **Then**: 
  - Block nếu Email đã có trong hệ thống hoặc Mã Sinh Viên hiện hữu ở sinh viên khác trong trường. Trả lỗi validation.
  - Nếu thông tin valid, tự động Insert Student và Enroll vào Term.

## AC-03: Xem Tìm kiếm Danh sách (Grid)
- **Given**: Trang danh sách sinh viên của Term mở lên.
- **When**: Đổi trang, tìm kiếm, sắp xếp.
- **Then**:
  - Hỗ trợ sắp xếp (A-Z Họ tên, Mã SV, Ngày enroll...).
  - Tìm debounce (300ms) trên Text, Lọc filter mảng Trạng thái Placed/Unplaced/Withdrawn.
  - Bảng duy trì state bộ lọc ngay cả khi click sang trang 2, 3.

## AC-04: Xem Chi tiết Enrollment  Placement Update
- **Given**: Uni Admin mở Form Chi Tiết một Enrollment cụ thể.
- **When**: Cập nhật trạng thái `Unplaced` sang `Placed`.
- **Then**: Bắt buộc chọn dropdown Enterprise. Khi chuyển từ `Placed` sang `Unplaced`, tên Enterprise tự động xóa (Null).

## AC-05: Rút/Khôi phục Sinh viên (Withdrawn Flow)
- **Given**: Uni Admin click Unenroll sinh viên.
- **When**: Sinh viên đang `Placed`.
- **Then**: Báo lỗi `400 Bad request` chặn việc rút (yêu cầu chuyển Unplaced trước).
- **When**: Sinh viên đang `Unplaced`.
- **Then**: Chuyển trạng thái sang `Withdrawn`. Gửi thông báo Email cho sinh viên. Cho phép "Khôi phục" lại `Unplaced` tại mảng list Filter Withdrawn.

## AC-06: Quyền truy cập (RBAC Guard)
- **Given**: User cố gắng thao tác (Import/Thêm/Sửa/Rút).
- **When**: Người này là HR, Student hoặc Uni Admin trường khác.
- **Then**: Trả về `403 Forbidden` do thiếu Permission.

---

# 4. Đặc tả Kỹ thuật (Technical Specifications)

Dựa theo chuẩn **FFA Framework (IOCv2 Agent Skills)**:

## 4.1 Bảng API Endpoints Bắt buộc

| API Endpoint | Method | Path | Request Variables | Response Structure (Thành công 200/201) | Mã lỗi dự kiến |
| --- | --- | --- | --- | --- | --- |
| Import thông qua Excel | **POST** | `/api/terms/{termId}/enrollments/import` | *Path*: `termId`<br>*Form*: `file` | `{ successCount, errorCount, errors: [{ row, message }] }` | **400 Bad Request**<br>**403 Forbidden**<br>**429 Throttled** |
| Thêm SV thủ công | **POST** | `/api/terms/{termId}/enrollments` | *Path*: `termId`<br>*Body*: `fullName`, `studentCode`, `email`, `phone`, `dob` | `{ enrollmentId, message: "Added successfully" }` (201 Created) | **400 Bad Request**<br>**404 Not Found**<br>**403 Forbidden** |
| Get danh sách Enrollments | **GET** | `/api/terms/{termId}/enrollments` | *Path*: `termId`<br>*Query*: `page`, `pageSize`, `search`, `placementStatus`, `sortBy`, `sortDir` | `{ list: [ { enrollmentId, student: {...}, placementStatus, enterpriseName,... } ], totalCount }` | **400 Bad Request**<br>**403 Forbidden** |
| Chi tiết Enrollment | **GET** | `/api/terms/{termId}/enrollments/{studentId}` | *Path*: `termId`, `studentId` | `{ enrollmentId, termId, student: {...}, placementStatus, enterpriseId, feedbackMid, feedbackFinal... }` | **404 Not Found**<br>**403 Forbidden** |
| Update Enrollment (Placement) | **PUT** | `/api/terms/{termId}/enrollments/{studentId}` | *Path*: `termId`, `studentId`<br>*Body*: `placementStatus`, `enterpriseId` (opt) | `{ message: "Updated successfully" }` | **400 Bad Request**<br>**404 Not Found**<br>**409 Conflict** |
| Rút 1 Sinh viên (Withdraw) | **DELETE**| `/api/terms/{termId}/enrollments/{studentId}`| *Path*: `termId`, `studentId` | `{ message: "Unenrolled successfully" }` | **400 Bad Request** (Đang Placed)<br>**404 Not Found** |
| Rút/Khôi phục Hàng Loạt | **PATCH** | `/api/terms/{termId}/enrollments/bulk-status` | *Path*: `termId`<br>*Body*: `studentIds[]`, `actionType` (Withdrawn/Unplaced) | `{ successCount, failedCount }` | **400 Bad Request**<br>**403 Forbidden** |

*[⚠️ Architecture Violation]: Flow Upload Excel cần thiết kế qua hệ thống Batch Operation bằng IFormFile -> MemoryStream. Nên giới hạn size (5MB). Nếu File quá to, xử lý bất đồng bộ Background Job, tuy nhiên Issue yêu cầu Preview nên bắt buộc BE phải đọc file trả Validation ngay lập tức.*

## 4.2 [FFA-ACV]  [FFA-ERR] Validation  Error Handling
- **Upload Validation**: Extension bắt buộc là `.xlsx`. Nếu upload `.csv` hoặc `.pdf`, Middleware văng thẳng `400 Bad Request` dạng `{"file": ["Chỉ hỗ trợ file Excel XLSX"]}`.
- **Manual Add Validation**: FluentValidation kiểm tra Email `.EmailAddress()`, SĐT Regex `^\+?[0-9]{10,11}$`, Name không có Script. Trả `400 Bad Request` khi lỗi, detail properties.
- **Withdraw Logic Check**: Tại Command Handler của Delete, Nếu `Enrollment.PlacementStatus == Placed`, ném exception `DomainViolationException` để Global Filter bọc lại bằng HTTP `400` với thông điệp: `"Sinh viên đang được xếp vị trí, vui lòng huỷ placement trước khi rút."`.

## 4.3 [FFA-SEC] Authentication  Authorization
- Ràng buộc **Authenticate** ở mọi API bằng token `Bearer`.
- Kiểm tra Authorize: `[Authorize(Roles = "UniAdmin, SupAdmin")]`.
- **Database Ownership Check**: Trong Handler, lấy `Term` ra check `Term.UniversityId == CurrentUser.UniversityId` (nếu role không phải SupAdmin). Kẻ xấu gọi URL chéo trường sẽ ăn mã `403 Forbidden`.

## 4.4 [FFA-PERF] Cấn hình DB  Caching
- **Rate Limit**: API Import file Excel dễ ngốn Memory/CPU, cần config Limit: `5 requests / phút / IP` (trả về 429 nếu spam). Write Limit thủ công là `20 requests / phút`. Read Limit List là `60 req / phút`.
- Lọc List với **IQueryable** và Indexing Db trên `(TermId, PlacementStatus)` để support truy vấn `Placed/Unplaced` siêu nhanh. Không fetch Full list nếu Paginate. Mặc định Sorted by `DateEnrolled DESC`.