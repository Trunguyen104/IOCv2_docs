# 1. Tổng quan & Bối cảnh (Overview & Context)

## 1.1 Vấn đề (Problem)
Hiện tại, hệ thống IOC cần quản lý các kỳ thực tập (Term) một cách tập trung và có cấu trúc để:
- Định nghĩa rõ ràng các kỳ thực tập theo từng trường học (thời gian bắt đầu, kết thúc, trạng thái).
- Gắn kết sinh viên, doanh nghiệp, nhóm thực tập và các luồng đánh giá vào đúng kỳ thực tập tương ứng.
- Kiểm soát vòng đời của kỳ thực tập: từ khởi tạo (Upcoming) → đang hoạt động (Active) → kết thúc tự nhiên (Ended) → đóng (Closed).
- Đảm bảo dữ liệu thực tập không bị rải rác hoặc nhầm lẫn giữa các kỳ khác nhau.

## 1.2 Pain Points hiện tại
- Chưa có màn hình tập trung để tạo và quản lý kỳ thực tập.
- Không kiểm soát được vòng đời kỳ thực tập: kỳ đã kết thúc vẫn có thể bị nhầm lẫn với kỳ hiện tại.
- Sinh viên, HR/Mentor không có nơi tra cứu thông tin kỳ thực tập đang diễn ra.
- Dữ liệu gán doanh nghiệp, nhóm thực tập, đánh giá không được ràng buộc rõ ràng với một kỳ thực tập cụ thể.
- Không có cơ chế xóa kỳ thực tập tạo nhầm, dẫn đến rác dữ liệu.

## 1.3 Giá trị Nghiệp vụ (Business Value)
- **Quản lý vòng đời**: Uni Admin tạo, theo dõi, cập nhật, đóng và mở lại kỳ thực tập.
- **Dữ liệu sạch & cấu trúc**: Các luồng nghiệp vụ (gán doanh nghiệp, báo cáo...) hoạt động chính xác theo từng kỳ. Soft delete dọn rác.
- **Minh bạch**: Phân quyền truy cập rõ ràng và hỗ trợ tra cứu dễ dàng cho Student và HR.
- **Kiểm soát quy trình**: Đóng kỳ thực tập để chốt sổ, ngăn chặn thay đổi.

## 1.4 Đối tượng (Actor)
- **Primary Actor: Quản trị viên trường (Uni Admin)**: Tạo, xem, cập nhật, khóa/mở, xóa dữ liệu kỳ. Thực hiện các thao tác quản lý vòng đời.
- **Secondary Actors:**
  - **Student (Sinh viên)**: Xem thông tin kỳ thực tập áp dụng cho mình (Chỉ đọc).
  - **HR / Mentor**: Xem thông tin kỳ liên quan đến doanh nghiệp/nhóm thực tập (Chỉ đọc).

---

# 2. Luồng Người dùng (User Flow)

## 2.1 Luồng Quản lý danh sách
1. **Đăng nhập**: Uni Admin đăng nhập thành công.
2. **Truy cập**: Điều hướng tới trang "Quản lý Kỳ thực tập".
3. **Hiển thị**: Hệ thống hiển thị danh sách các kỳ thực tập của trường (hỗ trợ phân trang, tìm kiếm, lọc theo Trạng thái & Năm, sắp xếp).

## 2.2 Luồng Tạo mới Kỳ thực tập
1. **Thao tác**: Từ trang danh sách, Uni Admin click "Tạo kỳ thực tập". Hệ thống mở Modal nhập liệu.
2. **Nhập liệu**: Điền Tên kỳ thực tập, Ngày bắt đầu, Ngày kết thúc.
3. **Xử lý**: Hệ thống validate dữ liệu (không trùng tên, độ dài, ngày hợp lệ, không overlap với kỳ đang diễn ra).
4. **Kết quả**: Tạo thành công, trạng thái ban đầu là `Upcoming` hoặc `Active`. Danh sách cập nhật.

## 2.3 Luồng Xem Chi tiết & Chỉnh sửa
1. **Xem chi tiết**: Nhấp vào "Xem chi tiết" để xem toàn bộ thông tin, timeline, enrollment summary.
2. **Chỉnh sửa**: 
   - Nếu kỳ `Upcoming`/`Closed`: Cho phép đổi Tên, Start Date, End Date.
   - Nếu kỳ `Active`/`Ended`: Start Date bị khóa (Read-only).
   - Validation & Optimistic Locking kiểm tra song song trước khi ghi đè dữ liệu.

## 2.4 Luồng Đóng/Mở lại & Xóa
1. **Đóng kỳ**: Với kỳ `Active`/`Ended`, Uni Admin chọn "Đóng kỳ", qua cảnh báo, đổi trạng thái thành `Closed`. Khóa toàn bộ dữ liệu enrollment bên trong.
2. **Mở lại kỳ**: Với kỳ `Closed`, Uni Admin chọn "Mở lại", hệ thống kích hoạt về `Active` để mở sửa chữa dữ liệu nội bộ.
3. **Xóa kỳ**: Chỉ hỗ trợ cho kỳ `Upcoming`. Xác nhận cảnh báo soft-delete. Đánh dấu xóa và ẩn khỏi view.

---

# 3. Tiêu chí Chấp nhận (Acceptance Criteria)

## AC-01: Hiển thị \u0026 Quản lý Danh sách Kỳ thực tập
- **Given:** Uni Admin truy cập trang "Quản lý Kỳ thực tập".
- **When:** Dữ liệu tải trang hoàn tất.
- **Then:**
  - Hiển thị danh sách bảng: Tên, Ngày bắt đầu, Ngày kết thúc, Trạng thái (Upcoming/Active/Ended/Closed), Số lượng SV (Enrolled, Placed, Unplaced).
  - Phân trang 10/20/50 bản ghi. Mặc định 10.
  - Hỗ trợ Tìm kiếm theo tên (debounce 300ms).
  - Dễ dàng Lọc theo năm của startDate, Trạng thái.
  - Sắp xếp asc/desc dựa trên các header.
  - **[UI/UX]** Trạng thái trống (Empty state) khi không có dữ liệu với nút "Tạo kỳ thực tập".

## AC-02: Tạo mới Kỳ thực tập thành công
- **Given:** Uni Admin mở Modal "Tạo kỳ thực tập".
- **When:** Nhập đầy đủ & hợp lệ Tên kỳ (<= 255 ký tự), Ngày bắt đầu (>= hôm nay), Ngày kết thúc (> Ngày bắt đầu) và click "Lưu".
- **Then:**
  - Tạo dữ liệu thành công, model ẩn. Hệ thống thông báo toast-message thành công.
  - Tự động gán trạng thái `Upcoming` hoặc `Active` tùy ngày.
  - Render lại danh sách bản ghi.

## AC-03: Cập nhật Kỳ thực tập \u0026 Xác thực Dữ liệu
- **Given:** Uni Admin mở Modal "Chỉnh sửa".
- **When:** Admin thay đổi dữ liệu hoặc nhấn lưu.
- **Then:**
  - Nếu kỳ `Active`/`Ended`: `startDate` bị disabled, không cho thao tác.
  - Xảy ra lỗi validation (400 Bad Request) nếu:
    - Bỏ trống bất kỳ trường bắt buộc.
    - `endDate` < `startDate`.
    - Trùng tên với một kỳ thực tập khác trong trường.
    - Trùng khoảng thời gian (overlap) với kỳ đang active của trường.
  - Hệ thống tính toán lại trạng thái tự động dựa trên date.

## AC-04: Đóng \u0026 Mở lại Kỳ thực tập
- **Given:** Kỳ thực tập đã tồn tại.
- **When:** Admin thực thi thao tác tương ứng theo quyền.
- **Then:**
  - Nút **Đóng kỳ**: Chỉ xuất hiện tại `Active`, có cảnh báo khi còn SV `unplaced`. Sau khi đóng -> `Closed` -> Không thể enroll/import sinh viên.
  - Nút **Mở lại kỳ**: Chỉ xuất hiện tại `Closed`. Sau khi mở -> `Active` -> Các tính năng enrollment trở lại bình thường.

## AC-05: Xóa Kỳ thực tập (Soft-Delete)
- **Given:** Một kỳ thực tập ở trạng thái `Upcoming`.
- **When:** Admin ấn nút "Xóa" và vượt qua bước "Xác nhận xóa".
- **Then:**
  - Nếu đã có dữ liệu bên trong (student, placement): Gói lệnh Xóa 2 lớp, cần xác nhận: `"Toàn bộ dữ liệu liên quan sẽ bị giấu. Không thể hoàn tác."`
  - Hành động API chỉ thực hiện **soft delete** (`isDeleted=true`, update `deletedAt`, `deletedBy`). 
  - Ẩn hoàn toàn khỏi query hiển thị của application.
  - Khóa API không cho phép xóa đối với trạng thái `Active`/`Ended`/`Closed`.

## AC-06: Quyền Truy Cập (Authorization Control)
- **Given:** Người dùng cố truy cập API hoặc UI.
- **When:** Tương tác tạo/sửa/xóa/xem danh sách InternshipTerm.
- **Then:**
  - Uni Admin chỉ được phép tác động lên Term thuộc `UniversityId` của mình. Truy cập Term khác -> 403 Forbidden.
  - Các roles như Student, Mentor, HR/Enterprise không có nút thao tác. Call direct API trả về 403 Forbidden.

---

# 4. Đặc tả Kỹ thuật (Technical Specifications)

Dựa theo chuẩn **FFA Framework (IOCv2 Agent Skills)**:

## 4.1. [FFA-CTL] \u0026 [FFA-CAG] Clean Architecture 
- **Controller layer**: `InternshipTermController` phải thuộc chuẩn Thin Controller, chỉ thực hiện route routing và gửi payload tới `IMediator`. Không xử lý business rules.
- Nên chia nhỏ Feature thành các thư mục Command / Query riêng:
  - `CreateTermCommand` / `CreateTermCommandHandler`
  - `UpdateTermCommand` / `UpdateTermCommandHandler`
  - `CloseTermCommand` / `CloseTermCommandHandler`
  - `DeleteTermCommand` / `DeleteTermCommandHandler`
  - `GetTermsQuery` / `GetTermsQueryHandler`
  - `GetTermByIdQuery` / `GetTermByIdQueryHandler`
- Return Type qua Global response wrapper, chứa Data, Pagination properties.

## 4.2. [FFA-ERR] \u0026 [FFA-ACV] Validation \u0026 Error Handling
- **Request Validation**: Dùng FluentValidation cho mọi Command.
  - Rule: string max length 255 cho `TermName`.
  - Rule: `EndDate` Must be Greater Than `StartDate`.
- **Global Exception Filter**:
  - Validation failures văng thẳng Exception để bắt bằng `400 Bad Request`.
  - Các lỗi Not Found Exception trả `404 Not Found`.

## 4.3. [FFA-SEC] Authentication \u0026 Authorization
- Cấp quyền trên Controller Endpoint bằng `[Authorize(Roles = "UniAdmin")]` (hoặc Policy-based Authorize Role).
- Ở Handler (ví dụ Delete), khi FindById ra được InternshipTerm, phải kiểm tra tiếp `Term.UniversityId == CurentUser.UniversityId`. Bất hợp lệ quăng `ForbiddenAccessException` -> `403`.
- Bắt buộc Rate Limiting tại các API POST/PUT/DELETE chống spam tác vụ.

## 4.4. [FFA-PERF] \u0026 DTO 
- Câu Query List kèm Filtering/Sorting phải apply Index ở database cho `UniversityId`, `Status` tĩnh và `StartDate`. 
- Tránh lazy loading. EF Core LINQ `Select` ánh xạ thẳng ra DTO để giảm size object từ Database lên. Cấm sử dụng toàn vẹn Entity trên API Response.

> **⚠️ LƯU Ý KHI CODE**: Model Term cần các trường: `TermId`, `UniversityId`, `Name`, `StartDate`, `EndDate`, `Status` (Enum/Int tính toán hoặc trigger lưu), Audit Fields (Kế thừa Base Entity).

