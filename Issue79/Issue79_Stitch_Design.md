# Phân tích & Thiết kế UI - Issue79: Enterprise Management (SupAdmin)

## BƯỚC 1 — Phân tích Issue

**Stitch Project ID:** `8794011393717868851`
**Design Project Name:** `IOCv2 - Issue79 Enterprise Management (SupAdmin)`
**Design System:** Desktop Layout | Light Mode | Be Vietnam Pro | ROUND_FULL | Primary #d52020

**User goal + Business objective:**
- **User Goal (SupAdmin):** Có một trung tâm điều khiển (Admin Control Panel) duy nhất để tạo mới, theo dõi, và thay đổi trạng thái hoạt động của các doanh nghiệp kết nối với nền tảng.
- **Business Objective:** Chuẩn hóa quy trình onboard doanh nghiệp, quản lý rủi ro bằng cách kiểm soát trạng thái (khóa/mở) của doanh nghiệp, làm nguồn dữ liệu chuẩn (SSOT) cho UniAdmin và toàn hệ thống để từ đó phân bổ sinh viên chính xác.

**Functional requirements:**
1. **Xem danh sách doanh nghiệp:** Hiển thị dạng bảng (Table/DataGrid) với các thông tin cốt lõi (Tên, MST, Website, Ngày tham gia, Trạng thái). Hỗ trợ tìm kiếm, lọc và phân trang.
2. **Onboard doanh nghiệp mới:** Màn hình Form (hoặc Drawer) cho phép nhập thông tin công ty và thông tin nhân sự đại diện (HR).
3. **Quản lý trạng thái:** Thay đổi trạng thái doanh nghiệp (từ Active sang Suspended/Inactive và ngược lại), chặn toàn bộ HR đăng nhập.

**Edge cases + Validation rules:**
- **Validation:** Mã số thuế (Chỉ số/chữ, Max 50, Unique), Tên (Bắt buộc, Max 255), Email HR (Đúng định dạng, Unique).
- **Edge Cases:** Chặn thao tác trùng lặp (ví dụ: submit form liên tục), xác nhận trước khi suspend một công ty (ngăn chặn thao tác nhầm lẫn).
- **Phân quyền:** Chỉ `SupAdmin` mới được thực hiện POST/PATCH. `UniAdmin` xem list (Read-only). Nếu Role không hợp lệ $\to$ 403 Forbidden.

---

## BƯỚC 2 — Thiết kế UX Flow

**User flow step-by-step:**
1. **Truy cập:** SupAdmin click "Quản lý Doanh nghiệp" trên Sidebar $\to$ Xem màn hình Danh sách Doanh nghiệp.
2. **Liệt kê & Tìm kiếm:** Hệ thống hiển thị Data Grid. User nhập "MST" vào ô tìm kiếm hoặc bộ lọc $\to$ Trạng thái Loading hiển thị $\to$ Kết quả cập nhật.
3. **Onboard Form:** Click "Thêm mới Doanh nghiệp" $\to$ Mở Drawer "Onboard Enterprise". User điền 2 Cụm thông tin: Hồ sơ Công ty & Thông tin Đại diện (HR).
4. **Submit & Toast:** Click "Tạo" $\to$ Nếu lỗi Validation, hiển thị Inline Error. Nếu thành công, đóng Drawer, reload Table, hiển thị Toast "Tạo mới thành công - Email đã được gửi".
5. **Đổi trạng thái:** Tại Table, ở cột Actions của công ty XYZ, click "Ngừng hoạt động" $\to$ Modal Cảnh báo hiện lên $\to$ Xác nhận $\to$ Toast Success, Row đổi sang trạng thái (badge màu xám/đỏ).

**Danh sách toàn bộ màn hình/trạng thái cần có:**
1. **Screen 1: Enterprise List (Data Grid View)** - Trang chính.
2. **Screen 2: Enterprise List (Empty State)** - Khi hệ thống chưa có công ty nào, hoặc filter không ra kết quả.
3. **Screen 3: Onboard New Enterprise (Side Drawer / Full Form)** - Nhập liệu thông tin tạo mới.
4. **Screen 4: Warning Modal (Suspend Action)** - Xác nhận để đổi trạng thái.
5. **Screen 5: Form Validation Error & Notification Toast** - Form có các lỗi inline.
6. **Screen 6: Mobile View (Responsive)** - Giao diện Card-based cho danh sách thay vì Table.

---

## BƯỚC 3 & 4 — Output (Cấu trúc & Component Tree)

### 1. Màn hình: Enterprise List (Data Grid View)
- **Vị trí trong flow:** Màn hình chính sau khi click từ Sidebar (SupAdmin).
- **Layout structure:**
  - **Sidebar:** Left navigation (Active item: "Quản lý Doanh nghiệp").
  - **Header:** Title (Quản lý Doanh nghiệp) + Breadcrumb + User Profile Avatar (Top right).
  - **Action Bar (Top Content):** Search input (Tên, MST), Filter Dropdown (Ngành nghề, Trạng thái), Nút Primary "Thêm mới Doanh nghiệp" (+ icon).
  - **Content Area:** Data Table + Pagination footer.
- **Component tree chi tiết:**
  - `Layout` > `Sidebar`
  - `Layout` > `MainContent`
    - `PageHeader` (Title, Breadcrum)
    - `FilterBar` > `InputRoot(SearchIcon)`, `Select(Industry)`, `Select(Status)`, `Button[variant=primary]`
    - `DataTable` > `TableHeader` + `TableRow`
      - `TableCell` (Enterprise Info: Logo, Name, TaxCode)
      - `TableCell` (Industry Badge)
      - `TableCell` (Status Badge: Active=Green, Suspended=Red)
      - `TableCell` (Actions: IconButton (Edit), IconButton (Suspend/Lock))
    - `Pagination`
- **Responsive behavior:**
  - **Desktop:** Layout đầy đủ, bảng chia cột.
  - **Tablet:** Thu gọn sidebar (chỉ icon), bảng có thể hiển thị thanh cuộn ngang (overflow-x).
- **State variations:**
  - Hover trên Table Row (đổi màu nhẹ). Loading State (Skeleton loading cho Table thay vì trống trơn).
- **Accessibility notes:**
  - Bảng cần có `aria-label` và `role="table"`.
  - Input search có `aria-placeholder`.

### 2. Màn hình: Empty State (Kèm Loading)
- **Layout structure:** Giống màn hình List (Sidebar, Top header) nhưng vùng Table thay bằng illustration.
- **Component tree chi tiết:**
  - `EmptyState` > `Illustration/Icon`
  - `EmptyState` > Title ("Chưa có doanh nghiệp nào") + Subtitle ("Bấm thêm mới để onboard một doanh nghiệp vào hệ thống").
  - `EmptyState` > `Button[variant=primary]` (Tạo mới).

### 3. Màn hình: Onboard New Enterprise (Side Drawer Form)
- **Vị trí trong flow:** Ngay khi click "Thêm mới" từ trang List. Dùng Drawer trượt từ bên phải để user không mất context trang List.
- **Layout structure:**
  - **Drawer Header:** Title "Tạo doanh nghiệp mới" + Nút Close (X).
  - **Drawer Body (Scrollable):** Chia làm 2 Section (Company Profile & Representative HR).
  - **Drawer Footer (Sticky):** Nút Cancel + Nút Submit.
- **Component tree chi tiết:**
  - `Overlay`
  - `Drawer` (Width: 480px)
    - `DrawerHeader` (Title, X Icon)
    - `Form`
      - `SectionTitle` (1. Thông tin Doanh nghiệp)
      - `FormGroup` > `Label`, `Input` (Tên công ty)
      - `FormGroup` > `Label`, `Input` (Mã số thuế)
      - `FormGroup` > `Label`, `Select` (Ngành nghề)
      - `SectionTitle` (2. Tài khoản Đại diện)
      - `FormGroup` > `Label`, `Input` (Họ Tên HR)
      - `FormGroup` > `Label`, `Input` (Email HR)
      - `ErrorMessage` (Inline error text cho Email bị trùng).
    - `DrawerFooter` > `Button[variant=ghost]`, `Button[variant=primary, type=submit]`
- **State variations:**
  - Submit Button: `disabled` nếu chưa điền đủ field bắt buộc, hoặc có trạng thái `loading` (spinner) khi call API.
  - Form Fields: `focus` outline màu primary, `error` outline màu đỏ (khi trùng MST).

### 4. Màn hình: Suspend Confirmation Modal
- **Vị trí trong flow:** Khi click icon "Lock" tại một hàng.
- **Layout structure:** Modal Pop-up ở giữa màn hình, đè Overlay.
- **Component tree chi tiết:**
  - `Dialog`
    - `DialogHeader` > `Icon(Warning - Red/Orange)` + Title ("Ngừng hợp tác với [A]?").
    - `DialogBody` > Text ghi rõ hậu quả ("Mọi tài khoản HR từ công ty này sẽ bị cấm đăng nhập lập tức...").
    - `DialogFooter` > `Button[variant=ghost]` (Hủy), `Button[variant=destructive]` (Ngừng hoạt động).
- **Accessibility notes:** Trap focus bên trong Modal, Escape để đóng.

### 5. Màn hình: Mobile Responsive List (Card-based)
- **Responsive behavior:**
  - Khi breakpoint < 768px, Table giấu đi, thay bằng List các Card component.
  - Floating Action Button (FAB) ở góc phải dưới thay cho Nút Add ở góc trên.
- **Component tree chi tiết:**
  - `MobileLayout` > `TopNavigation` (Hamburger Menu)
  - `CardList`
    - `Card` > `Flex(Row)`: Logo + Name + Status Badge.
    - `Card` > Divider.
    - `Card` > `Flex(Row)`: TaxCode / Ngành nghề.
    - `Card` > `ActionRow`: Nút Edit, Suspend.

---

## BƯỚC 5 — Self-check
- **Vi phạm Design System?** Không. Các component như Button, Table, Drawer, Modal, Input, Badge sử dụng đúng chuẩn chung (SaaS Professional style), phân cấp semantic (Primary, Destructive, Ghost).
- **Bỏ sót màn hình?** Đã bao phủ toàn bộ luồng tạo lưới, Thêm mới, Ngừng hoạt động, và Empty/Mobile state.
- **Tính Consistent:** Luồng Drawer (cho Input) và Modal (cho Cảnh báo nguy hiểm) tạo sự tách biệt UX cần thiết. Tính Responsive (Card cho Mobile) đúng với UX hiện đại. Tiêu chí Clean & Minimal được duy trì rõ ràng qua Whitespace và DataGrid Layout chuyên nghiệp.

> Quá trình Gen UI lên Stitch đang được tự động BATCH bằng MCP theo mô tả ở trên.

https://stitch.withgoogle.com/projects/5670065673026564724