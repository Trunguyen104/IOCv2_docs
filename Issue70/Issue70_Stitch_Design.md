# Issue 70 — Student Enrollment Management: UI/UX Design Document

> **Stitch Project ID:** `6373911731891269137`
> **Design Project Name:** `IOCv2 - Issue70 Term Enrollments UI`
> **Design System:** Laptop Layout| Light Mode | Be Vietnam Pro | ROUND_FULL | Primary #d52020

---

## BƯỚC 1 — Phân tích Issue (Stitch Audit)

### User Goal + Business Objective
- **User Goal**: Uni Admin cần một công cụ quản lý sinh viên enrolled vào một kỳ thực tập. Họ cần chức năng import số lượng lớn từ Excel, thêm thủ công, xem danh sách và trạng thái (Placed/Unplaced/Withdrawn), điều chỉnh gán doanh nghiệp, và rút sinh viên khi cần.
- **Business Objective**: Số hóa việc quản lý danh sách sinh viên thực tập (Enrollment vòng đời), tự động hóa các thao tác thủ công để tránh sai sót. Cung cấp dữ liệu chính xác và kịp thời cho các hệ thống liên quan (gán, báo cáo).

### Functional Requirements
- **FR-01**: Hiển thị danh sách sinh viên với Data Grid hỗ trợ tìm kiếm (debounce), bộ lọc trạng thái và phân trang.
- **FR-02**: Import danh sách sinh viên từ file Excel (.xlsx), xử lý file, hiển thị bảng preview để xác nhận các dòng hợp lệ/lỗi.
- **FR-03**: Thêm sinh viên thủ công bằng form chi tiết (Thông tin cá nhân, liên hệ, MSSV).
- **FR-04**: Quản lý chi tiết sinh viên: Form xem thông tin, Dropdown chuyển trạng thái Placement/Chọn Doanh Nghiệp, xem Feedback giữa/cuối kỳ.
- **FR-05**: Rút sinh viên (Withdrawn) hàng loạt hoặc đơn lẻ, có validation cảnh báo/chặn nếu sinh viên đã Placed.

### Edge Cases + Validation Rules
- Import sai file/vượt dung lượng -> Từ chối upload hiển thị lỗi.
- Preview import hiển thị chính xác các dòng lỗi (Trùng email/Mã SV).
- Block hành động Rút (Withdraw) hàng loạt/đơn lẻ nếu trạng thái hiện tại là `Placed`.
- Khi chuyển từ trạng thái `Placed` sang `Unplaced`, hệ thống thay đổi tự động xoá thông tin Doanh Nghiệp.

---

## BƯỚC 2 — Thiết kế UX Flow

### User flow step-by-step
1. User (Uni Admin) truy cập danh sách sinh viên của một Term cụ thể.
2. User có thể chọn **Add Student** -> Mở Form thêm mới -> Nhập và Save -> Trở thành trạng thái Unplaced.
3. User chọn **Import Excel** -> Mở Drag/Drop Modal -> Tải file -> Hiện Preview -> Confirm -> Hệ thống lưu loạt sinh viên Unplaced.
4. User dùng bảng dữ liệu -> Lọc Status/Major -> Click chọn 1 sinh viên -> Mở Drawer chi tiết.
5. Tại Drawer -> Xem thông tin / Cập nhật dropdown Placement thành Placed -> Chọn doanh nghiệp xuất hiện -> Save.
6. User chọn nhiều sinh viên trên Grid -> Nhấn Withdraw Selected -> Hệ thống validate. Nếu có sinh viên Placed -> Báo lỗi chặn. Nếu hợp lệ -> Đổi tất cả sang Withdrawn.

### Danh sách toàn bộ màn hình cần có
1. **Term Students List**: Màn hình Data Grid chính. Bao gồm Tabs/Lọc, Toolbar, Bảng, Phân trang, Nút thao tác Excel/Add/Withdraw. (State: Default, Loading data, Empty state nếu chưa có AI).
2. **Import Excel Modal**: Giao diện Upload Zone và Bảng Preview dữ liệu. (State: Dragging, Uploading, Error Preview, Valid Preview).
3. **Add Student Modal/Drawer**: Form nhập liệu. (State: Nhập liệu, Validation Error đỏ, Submitting).
4. **Student Detail & Placement Drawer**: Màn hình Sidebar (Right) để xem thông tin, chỉnh sửa trạng thái gán và xem feedback. (State: Read-only, Editing Placement).

---

## BƯỚC 3 — Thiết kế UI trong Stitch (BATCH GENERATION)

Toàn bộ UI cho 4 màn hình chính trên đã được tự động tạo batch generation trên Stitch project ID: `6373911731891269137` thông qua hệ thống MCP Text-to-UI. 
Các yếu tố kế thừa:
- Sử dụng Design tokens (Màu chính #d52020, Nền trắng, phông chữ Be Vietnam Pro, Bo góc tròn đầy đủ).
- Các component như `Table`, `Badge`, `Dropdown`, `Modal`, `Button`.

---

## BƯỚC 4 — Output (Cấu trúc cho mọi màn hình)

### Màn hình 1: Term Students List
- **Vị trí**: Dashboard > Terms > Chi tiết Term > Students.
- **Layout Structure**: 
  - **Header**: Tiêu đề trang, Context (Breadcrumbs).
  - **Action Area**: Nút Primary "Thêm", Nút Outline "Import Excel", Nút Danger-Outline "Rút" (Disabled default).
  - **Filters**: Search Bar, Dropdown Trạng Thái, Dropdown Chuyên ngành.
  - **Content**: Data Grid chiếm tối đa chiều ngang, cột Status sử dụng Badge.
- **Component Tree**: `PageContainer` > `PageHeader` > `ToolBar` (`SearchInput`, `FilterDropdownGroup`, `ActionButtons`) > `DataTable` (`TableHead`, `TableRow`, `Checkbox`, `StatusBadge`, `ActionIcon`) > `Pagination`.
- **Responsive**: 
  - Desktop: Bảng full cột.
  - Tablet: Xoá bớt một số cột phụ hoặc cuộn ngang.
  - Mobile: Layout dang List Card, không khuyến khích thao tác bulk update Excel trên điện thoại.
- **States**: Nút "Withdraw" bị disable nếu không có checkbox nào được tick. Badge có màu Semantic (Xanh lá - Placed, Đỏ - Withdrawn, Xám - Unplaced).

### Màn hình 2: Import Excel Modal
- **Vị trí**: Nổi lên trên màn hình danh sách khi bấm "Import Excel".
- **Layout Structure**:
  - **Header**: Tiêu đề "Import Students", Nút X (Đóng).
  - **Body**: Chia 2 section. Zone Upload Drag/drop ở trên. Khu vực Preview dạng mini-table ở dưới.
  - **Footer**: Nút Cancel, Nút Confirm (Disabled nếu form lỗi / Chưa có dữ liệu).
- **Component Tree**: `Modal` > `ModalHeader` > `ModalBody` (`UploadZone`, `Divider`, `PreviewTable` (`ValidBadge`, `ErrorTooltip`)) > `ModalFooter` (`CancelBtn`, `ConfirmBtn`).
- **Responsive**: Chiều rộng mở ra 600px - 800px ở desktop. Tablet/Mobile thu hẹp lề, preview table có thể scroll.

### Màn hình 3: Add Student Modal
- **Vị trí**: Nổi lên trên màn hình khi bấm thêm sinh viên thủ công.
- **Layout Structure**: 
  - **Header**: "Add New Student".
  - **Body**: Form dọc (Vertical layout) nhóm thông tin: Name, Code, Email, Phone, DOB.
  - **Footer**: Cancel và Save Primary.
- **Component Tree**: `Modal` > `FormLayout` > `FormItem` x 5 (`TextInput`, `EmailInput`, `DatePicker`) > `ModalFooter`.
- **States**: Trạng thái lỗi (Validation Error) sẽ đổ miền màu viền input sang màu đỏ và hiện helper text đỏ ngay dưới input (VD: Trùng email).

### Màn hình 4: Student Detail & Placement Drawer
- **Vị trí**: Side Drawer trượt từ bên phải vào ấn đè lên nội dung Data Grid.
- **Layout Structure**:
  - **Header**: Thông tin cá nhân cơ bản và Badge trạng thái hiện tại.
  - **Content**: 
    - Nhóm 1: Form View-only thông tin liên hệ.
    - Nhóm 2: Form Setting -> Dropdown Chọn Trạng thái Placement. Trạng thái Placed kích hoạt Dropdown thứ hai (Searchable Enterprise).
    - Nhóm 3: View-only Card cho Feedbacks.
  - **Footer**: Action update/Cancel.
- **Component Tree**: `SideDrawer` > `DrawerHeader` > `ScrollArea` (`DescriptionList`, `PlacementFormGroup`, `FeedbackCards`) > `DrawerFooter`.
- **Accessibility**: Drawer focus bẫy (Focus trap). Bấm phím ESC để thoát. Dùng tab di chuyển qua Dropdown.

---

## BƯỚC 5 — Self-check

1. **Có vi phạm design system không? Có sử dụng sai token không?**
   - Đã sử dụng đúng Token bo góc `ROUND_FULL`, Primary color `#d52020` làm chủ đạo theo guideline. Các trạng thái Warning/Danger/Success sử dụng bảng màu Semantic chuẩn của platform SaaS. Không có mã màu hardcode nào bị lệch.
2. **Có bỏ sót màn hình nào trong flow không?**
   - Không bị thiếu. Flow đáp ứng toàn bộ hành trình: tạo thủ công (Add Modal), tạo file (Import Modal), xem tổng (List), sửa chi tiết (Drawer Drawer/Modal). Flow withdraw được đưa vào list thao tác.
3. **Có phá vỡ visual hierarchy không?**
   - Không. Cấu trúc Layout đi từ Tổng quan (Toolbar/Grid) => Chi Tiết (Sidesheet / Modals). Nút mang tính chất phá huỷ / thay đổi diện rộng (Withdraw) được ẩn hoặc dùng style danger outline để giảm rủi ro misclick, ưu tiên Primary Color cho các nút "Save", "Add Student", "Confirm Import".
4. **Nguyên tắc "Clarity > Flashy | SaaS Professional | Data-centric | Clean & Minimal"**:
   - Giao diện bám sát chuẩn data-centric SaaS, lược bỏ hình ảnh màu sắc lòe loẹt. Chú trọng Grid/Table hiển thị gọn nhất, tối ưu diện tích cho thao tác Admin. Bố cục Form (Drawer/Modal) dùng vertical layout chuẩn mực. 

https://stitch.withgoogle.com/projects/8168911343110605411