# Phân tích & Thiết kế UI - Issue84: Enterprise Management (My Profile)

## BƯỚC 1 — Phân tích Issue

**Stitch Project ID:** `8347161656204564046`
**Design Project Name:** `IOCv2 - Issue84 Enterprise Profile UI`
**Design System:** Desktop Layout | Light Mode | Be Vietnam Pro | ROUND_BUTTON | Primary #d52020

**User goal + Business objective:**
- **User Goal (Enterprise HR/Admin):** Dễ dàng tra cứu và cập nhật hồ sơ thương hiệu của chính công ty mình để phục vụ quá trình thu hút, tuyển chọn sinh viên thực tập.
- **Business Objective:** Tăng tần suất cập nhật thông tin chuẩn xác, tối ưu hóa tỷ lệ "Match" và tự động hóa luồng nghiệp vụ thay vì phụ thuộc vào Uni Admin. Giữ an toàn cho các dữ liệu nhạy cảm (như Mã số thuế) không bị thay đổi.

**Functional requirements:**
1. **Xem hồ sơ (My Profile):** Hiển thị màn hình chi tiết như Cover Photo (Background), Logo, Tên công ty, Website, Mã số thuế, Ngành nghề, Địa chỉ trụ sở và Nội dung Giới thiệu (Description).
2. **Cập nhật hồ sơ (Edit Mode):** Giao diện chỉnh sửa cho các trường dữ liệu trên. `TaxCode` bắt buộc khóa.
3. **Validation & Phản hồi:** Hiển thị lỗi ngay form (Inline Error) nếu Website sai định dạng. Khi lưu thành công, Toast message xuất hiện và cập nhật dữ liệu DOM cục bộ (Tránh tải lại trang).

**Edge cases + Validation rules:**
- `Tax Code` không được đẩy lọt qua API, và không được bật nhập ở UI (Trạng thái `Disabled` / Read-only / Read-only badge).
- `Website`: Nhập đúng Regex `http/https`.
- Tránh phơi bày tham số `enterpriseId` trên thanh URL (ẩn `Id`). Chỉ cung cấp ID dưới nền API `Authorization Token` theo dạng `enterprises/me`.

---

## BƯỚC 2 — Thiết kế UX Flow

**User flow step-by-step:**
1. **Truy cập:** HR click menu "Hồ sơ Công ty" (My Profile) $\to$ Router gọi lấy thông tin `enterprises/me`.
2. **Liệt kê (Read Mode):** Hiển thị Banner cover image, Logo ở Header Content, phía dưới là Label-Value Pairs cho các dữ liệu. Có nút "Chỉnh sửa hồ sơ" gọn gàng ở góc.
3. **Mở Form (Edit Mode):** User nhấn "Chỉnh sửa" $\to$ Mở Side Drawer hoặc Modal có form chi tiết.
4. **Validation (Lỗi Input):** User gõ `website` kiểu `www.congty.com` $\to$ Cảnh báo "Vui lòng nhập định dạng http:// hoặc https://". Gửi đi sẽ bắt thẻ đỏ, form không lưu.
5. **Thành công (Update & Toast):** Điền hợp lệ, nhấn "Lưu thay đổi". Load spinner ở Button. Gửi lên BE $\to$ Nhận 204. Toast màu xanh "Cập nhật hồ sơ thành công" $\to$ Trở về mành hiển thị (Read Mode) với dữ liệu mới.

**Danh sách toàn bộ màn hình/trạng thái cần có:**
1. **Screen 1: Enterprise Profile Detail (Read Mode)** - Bố cục thông tin trang chính giống CV công ty.
2. **Screen 2: Edit Profile (Side Drawer/Modal)** - Nhập liệu thông tin thay đổi.
3. **Screen 3: Form Error State & Disabled Fields** - Text box "Mã số thuế" khóa xám và Text box "Website" báo viền đỏ.
4. **Screen 4: Success State (Toast Notification)** - Màn hình vừa cập nhật xong hiện thông báo.
5. **Screen 5: Skeleton Loading State** - Cho lúc bắt đầu truy cập router, load nội dung.

---

## BƯỚC 3 & 4 — Output (Cấu trúc & Component Tree)

*(Quy mô BATCH GENERATION trên Stitch gồm toàn bộ 5 khía cạnh kết hợp trên Master View Layout để quan sát trọn luồng)*

### 1. Màn hình: Enterprise Profile Detail (Read Mode)
- **Vị trí trong flow:** Mặc định của HR Dashboard.
- **Layout structure:**
  - **Sidebar:** Left navigation (Active item: "Hồ sơ Công ty").
  - **Header:** "Tên của doanh nghiệp" (hoặc Breadcrumb "My Profile") + User Profile.
  - **Cover/Header Card:** Một Card rộng tràn trang chứa Image Cover (height 200px) và Profile Picture tròn nổi (Logo) lệch lên ranh giới + Tên to nổi bật kèm Badge "Active" / "Verified".
  - **Content Area:** 
    - Lưới thông tin cột trái: `Description` Text block.
    - Lưới thông tin cột phải: List Label: Value cho Mã Số Thuế, Industry, Địa chỉ, Website.
  - **Action Area:** Nút `Button[variant=outline]` có text "Chỉnh sửa hồ sơ" góc trên phải (trong Card chính).
- **Component tree chi tiết:**
  - `Layout` > `Sidebar` + `TopNav`
  - `ProfilePage`
    - `BannerCover` > `Image`
    - `ProfileHeader` > `Avatar(Logo)`, `Heading(Tên)`, `Badge`
    - `Grid(cols=12)`
      - `Col(span=8)` > `Card` > `RichText(Description)`
      - `Col(span=4)` > `Card` > `List` > `ListItem(Icon, Label, Value)`
    - `Button[primary]` (Edit Profile)

### 2. Màn hình: Edit Profile (Side Drawer Form)
- **Vị trí trong flow:** Ngay khi click "Chỉnh sửa hồ sơ". Mở Drawer giúp giữ nguyên Layout nền (Modal không mất context).
- **Layout structure:** Panel cố định phải.
  - Sticky Header, Scroll Body (Các Input Fields), Sticky Footer.
- **Component tree chi tiết:**
  - `Drawer` (Width: 500px)
    - `DrawerHeader` > "Cập nhật Thông tin" 
    - `Form`
      - `UploadField` (Background/Logo)
      - `FormGroup` > `Label`, `Input` (Tên công ty)
      - `FormGroup` > `Label`, `Input[disabled=true]` (Mã số thuế - Read only)
      - `FormGroup` > `Label`, `Input[type=url, error=true]` (Website) + `InlineErrorMessage` ("Phải chứa http/https")
      - `FormGroup` > `Label`, `Select` (Industry)
      - `FormGroup` > `Label`, `TextArea` (Address/Description)
    - `DrawerFooter` > `Button[variant=ghost]`, `Button[variant=primary] (Lưu thay đổi)`
- **State variations:**
  - Input TaxCode phông nền màu `#F3F4F6`, không thể trigger Focus State (Disabled).
  - Button Submit: Sẽ hiện Loading spinner bên trong nếu quá trình update kéo dài.
- **Accessibility notes:** Input errors cần thẻ `aria-describedby` và Label cần link với thẻ `htmlFor`.

### 3. Màn hình: Skeleton Loading
- **Responsive behavior:** Hiển thị khối xám Skeleton dạng thẻ Box trước khi Content API `/me` trả kết quả để giữ cảm giác App tức thì (Snappy).

### 4. Bố cục Mobile View
- **Tablet/Mobile:** Form Side Drawer đổi thành full screen Panel để dễ gõ phím ảo. Bố cục lưới Read-Mode gộp từ cột (span 8+4) sang stack Rows theo chiều dọc.

---

## BƯỚC 5 — Self-check
- **Vi phạm Design System?** Các quy tắc Spacing và bo góc (`ROUND_EIGHT` hay `ROUND_FULL`) được duy trì. Màu Primary được tập trung để Call to Action (Chỉnh sửa/Lưu) chứ không tán sắc bừa bãi.
- **Bỏ sót màn hình nào?** Đã tích hợp luồng View Profile, Panel Edit, Trạng thái Input (Error/Disabled), Loading, Thông báo.
- **Tính Consistent:** Có sự phân cấp mạnh giữa Dữ liệu cho phép sửa (Input active outline) và Pháp lý khóa cứng (Mã số thuế disabled solid background).

> Quá trình Gen UI lên Stitch đang được tự động BATCH bằng MCP theo mô tả ở trên.

https://stitch.withgoogle.com/projects/8347161656204564046