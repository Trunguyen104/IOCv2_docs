# Issue 77 — Enterprise HR Management: UI/UX Design Document (Updated)

> **Stitch Project ID:** `8794011393717868851`
> ****Design Project Name:** `IOCv2 - Issue77 Enterprise HR & Group Management`
> ****Design System:** Desktop Layout | Light Mode | Be Vietnam Pro | ROUND_FULL | Primary #d52020

---

## BƯỚC 1 — Phân tích Issue (Stitch Audit)

### User Goal + Business Objective

- **User Goal**: Enterprise HR cần công cụ để phê duyệt sinh viên, **tổ chức sinh viên thành các nhóm làm việc**, và gán Mentor quản lý theo từng nhóm hoặc cá nhân.

- **Business Objective**: Minh bạch hóa việc phân bổ nhân sự; tối ưu hóa quản lý diện rộng bằng cách gán Mentor theo đơn vị nhóm thay vì thao tác thủ công từng sinh viên.

### Functional Requirements

- **FR-01**: Data Grid hiển thị Application sinh viên, lọc theo trạng thái và ngành học.

- **FR-02**: Flow duyệt nhanh (Accept) và từ chối kèm lý do (Reject).

- **FR-03 (New)**: **Group Management**: Tạo/Sửa/Xóa nhóm thực tập. Cho phép gán sinh viên vào nhóm (Bulk Action).

- **FR-04 (Update)**: **Assign Mentor**: Hỗ trợ gán Mentor cho toàn bộ một Nhóm hoặc gán lẻ cho sinh viên đã có nhóm.

- **FR-05**: Role-based Access: Mentor chỉ xem được sinh viên thuộc nhóm mình phụ trách.

---

## BƯỚC 2 — Thiết kế UX Flow

### User flow step-by-step

1. **Duyệt**: HR duyệt sinh viên từ tab **Pending** sang **Accepted**.

2. **Phân nhóm**: HR chọn các sinh viên trong danh sách `Accepted` $\\rightarrow$ Sử dụng **Bulk Action** để "Thêm vào nhóm" (Chọn nhóm có sẵn hoặc tạo mới).

3. **Gán Mentor**: HR vào giao diện **Quản lý Nhóm** $\\rightarrow$ Chọn nhóm $\\rightarrow$ Mở Modal **Assign Mentor** để gán người phụ trách cho cả nhóm.

4. **Kiểm tra**: Mentor đăng nhập sẽ thấy danh sách sinh viên được lọc tự động theo nhóm mà họ phụ trách.

### Danh sách màn hình cần có

1. **Internship Students List Grid**: Quản lý danh sách tổng và thực hiện Bulk Action phân nhóm.

2. **Group Management Dashboard**: Giao diện quản lý các nhóm thực tập (Card View hoặc Kanban).

3. **Reject Application Modal**: Popup lấy lý do từ chối.

4. **Assign Mentor to Group/Student Modal**: Popup gán Mentor với tùy chọn gán theo đơn vị Nhóm.

---

## BƯỚC 4 — Output (Cấu trúc màn hình chi tiết)

### Màn hình 1: Internship Students List Grid

- **Tabs**: `Pending`, `Accepted (Unassigned)`, `Grouped`.

- **Bulk Action Toolbar**: Xuất hiện khi tick chọn &gt; 1 sinh viên. Gồm các nút: "Create Group", "Add to Group".

- **Data Grid Column**: Thêm cột `Group Name` (Badge màu xám nhạt) để nhận diện sinh viên đã có nhóm hay chưa.

### Màn hình 2: Group Management Dashboard (Mới)

- **Layout Structure**:

  - **Header**: Nút "Create New Group".

  - **Main Content**: Danh sách các Nhóm dạng Card. Mỗi card hiển thị: Tên nhóm, Mentor phụ trách, Danh sách Avatar sinh viên (max 5), số lượng thành viên.

- **Actions**: Nút "Assign Mentor" và "View Details" trên mỗi Card.

### Màn hình 3: Assign Mentor Modal (Cập nhật)

- **Vị trí**: Nổi Center khi nhấn "Assign" từ Card Nhóm hoặc dòng Sinh viên.

- **Body**:

  - Hiển thị thông tin đối tượng được gán (Ví dụ: "Assigning for: Group Web Frontend").

  - **Dropdown**: Searchable Select chọn Mentor.

  - **Input**: Tên dự án/Vị trí thực tập.

- **Footer**: Nút Cancel và Primary Button "Confirm Assignment".

---

## BƯỚC 5 — Self-check

1. **Tính nhất quán**: Sử dụng cùng Design System #d52020 và bo tròn cho các Card nhóm.

2. **Logic quản lý**: Đã giải quyết được vấn đề "cần quản lý phân nhóm trước khi assign mentor". Sinh viên có thể được gán lẻ, nhưng ưu tiên luồng gán theo nhóm để giảm tải cho HR.

3. **Hierarchy**: Phân cấp rõ ràng giữa Danh sách sinh viên (Cá nhân) và Dashboard nhóm (Tổ chức).

https://stitch.withgoogle.com/projects/7264289854991260113