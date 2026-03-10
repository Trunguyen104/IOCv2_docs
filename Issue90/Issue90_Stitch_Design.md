# Phân tích & Thiết kế UI - Issue90: Quản lý Ứng tuyển & Job Portal (Student)

## BƯỚC 1 — Phân tích Issue

**Stitch Project ID:** `6023372471380982738`
**Design Project Name:** `IOCv2 - Issue90 Student Internship Portal UI`
**Design System:** Desktop Layout | Light Mode | Be Vietnam Pro (hoặc Inter) | ROUND_FULL | Primary Red (#d52020)

**User goal + Business objective:**
- **User Goal (Student):** Khám phá cơ hội thực tập, dễ dàng đọc yêu cầu công việc (JD), ứng tuyển, quản lý lịch phỏng vấn và chấp nhận Offer tự động hóa từ hệ thống.
- **Business Objective:** Thay thế luồng tuyển dụng thủ công ồn ào bằng luồng State-Machine rõ ràng: `Applied` $\to$ `Interviewing` $\to$ `Offered` $\to$ `Hired`. Hạn chế sinh viên bỏ lỡ cơ hội.

**Functional requirements:**
1. **Explore Jobs:** Giao diện cho phép xem/lọc danh sách vị trí Thực tập đang mở. Nút Bookmark lưu trữ tin. (Trang này bị khóa lập tức hoặc ẩn JD nếu SV đã được Hired).
2. **Apply Form:** Form Upload 2 file riêng biệt: CV (Required, Max 5MB) và Cover Letter (Optional) kèm input ghi chú bổ sung.
3. **Quản lý Đơn (My Applications):** Màn hình hiển thị tình trạng của toàn bộ các Application Sinh viên đã nộp. Các Action động hiện tùy ngữ cảnh (Ví dụ: Đang Interviewing thì hiện Confirm Time/Reschedule).
4. **Reschedule & Withdraw Modal:** Các Panel nổi để nhận thông tin bắt buộc (như Lý do đổi lịch phỏng vấn).

**Edge cases + Validation rules:**
- `CV file`: Trình duyệt phải kiểm tra file đuôi pdf/doc trước khi submit. Không có CV $\to$ Disable nút nộp đơn.
- Giai đoạn **Interviewing** Withdraw: Bắt buộc điền lý do $\to$ Form phải có Text Input bắt buộc.
- Nếu được **Offered**: Sinh viên thấy thẻ nổi bật yêu cầu quyết định (Accept/Decline).
- Chống apply nhiều lần cho cùng 1 Job.

---

## BƯỚC 2 — Thiết kế UX Flow

**User flow step-by-step:**
1. **Khám phá:** Sinh viên mở "Explore Jobs". Xem qua các cơ hội $\to$ Bấm vào JD chi tiết công ty A.
2. **Ứng tuyển (Apply):** Sinh viên bấm nút "Apply Now". Một Form đè lên màn hình cho phép Upload tài liệu và ghi chú $\to$ Nhấn Submit $\to$ Thành công (Chuyển đơn vào My Applications).
3. **Theo dõi trạng thái:** SV sang tab "My Applications" thấy đơn Cty A ở trạng thái `Applied/In Review`.
4. **Phỏng vấn (Interviewing):** Đơn chuyển trạng thái `Interviewing`. SV click "Confirm Time" hoặc "Reschedule" ở danh sách. Nếu "Reschedule", mở Modal nhập lý do, thời gian mới.
5. **Chốt Offer:** Trạng thái lên `Offered`. SV bấm "Accept" $\to$ Mọi ứng tuyển khác bị khóa. Toàn hệ thống ghi nhận SV vào trạng thái `Hired` (Mầu xanh rực rỡ).

**Danh sách toàn bộ màn hình/trạng thái cần có:**
1. **Screen 1: Explore Jobs (Job Board List & Filter)** - Màn hình chứa các JD thẻ (Cards).
2. **Screen 2: Application State (My Applications)** - Dashboard quản lý (List/Grid) thể hiện đơn theo nhiều trạng thái (Status badge).
3. **Screen 3: Apply Form (Drawer/Modal)** - Giao diện File uploader (Bao gồm cảnh báo thiếu file).
4. **Screen 4: Modals for Actions (Reschedule/Withdraw/Accept Offer)** - Các Pop-up xác nhận kèm input lý do.

---

## BƯỚC 3 & 4 — Output (Cấu trúc & Component Tree)

*(Để minh họa tốt nhất trên Stitch, ta sẽ dùng phương án **BATCH GENERATION** - Tạo một màn hình Student Dashboard phức hợp: Nửa trái là Explore Jobs/Application List, và các Overlay/Drawer hiển thị ngữ cảnh)*

### 1. Màn hình: Manage Applications (Trang chủ đích)
- **Vị trí trong flow:** Nơi sinh viên quản lý tiến độ thực tập.
- **Layout structure:**
  - **Sidebar:** Left Navigation (Active: "My Applications").
  - **Header:** Lời chào "Tiến độ ứng tuyển của bạn" + Avatar SV.
  - **Main Content:** Cấu trúc danh sách (DataGrid hoặc Card Stack) phân chia rõ 3 nhóm đơn: "Đang chờ" (Applied), "Phỏng vấn/Offer" (Interviewing/Offered) - Highlight lên để chú ý, và "Lịch sử" (Rejected/Hired).
- **Component tree chi tiết:**
  - `Layout` > `Sidebar`
  - `Main`
    - `ApplicationCard(Status="Offered")` > `Logo Cty`, `Title`, `Date`. Nổi bật bằng Alert Box xanh lá: "Bạn được cấp Offer!". `ActionButtons[Accept, Decline]`.
    - `ApplicationCard(Status="Interviewing")` > Text hiển thị Lịch. `ActionButtons[Confirm, Reschedule, Withdraw]`.
    - `ApplicationCard(Status="Applied")` > Text chờ duyệt. Nút `Button(Ghost, Withdraw)`.

### 2. Màn hình: Apply for a Job (Drawer Form)
- **Vị trí trong flow:** Ngay khi bấm Apply trên JD. Giữ UX bằng Side Drawer để sinh viên không rời khỏi Job Board.
- **Layout structure:** Pannel 450px trượt phải.
- **Component tree chi tiết:**
  - `Drawer`
    - `DrawerHeader` ("Ứng tuyển vào vị trí [Tên Job]")
    - `Form`
      - `FormGroup(CV)` > `Label(Required)`, `FileUploadBox` $\to$ Cảnh báo đỏ nếu trống.
      - `FormGroup(CoverLetter)` > `Label(Optional)`, `FileUploadBox`.
      - `FormGroup(Ghi chú)` > `TextArea`.
    - `DrawerFooter` > `Button(Submit)`

### 3. Màn hình: Validation Error & Reschedule Modal
- **Vị trí trong flow:** Trigger khi cần input thêm lý do hoặc file thiếu.
- **Layout structure:** Dialog ở giữa màn hình.
- **Component tree chi tiết (Reschedule):**
  - `Modal`
    - `ModalHeader` (Đề xuất hẹn lại lịch)
    - `Form` > `InputDate`, `TextArea`(Lý do dời lịch)
    - `Footer` > `Cancel`, `Submit Request`.

### 4. Bố cục Responsive & Accessibility
- **Desktop $\to$ Tablet:** Card view cho danh sách tự dãn lưới (Grid 3 cols $\to$ 2 cols).
- **Mobile:** Menu Hamburger, Drawer đổi thành Bottom Sheet trượt lên.
- **Accessibility:** Chú ý các `aria-live` cho các Toast notification, Focus bẫy (Trapping Focus) khi bật các thẻ Modal/Drawer để Sinh viên dùng bàn phím dễ dàng (Tab index).
- Các Badge màu tuân thủ contrast (Xanh dương - Interviewing, Xanh lục - Offered, Đỏ/Vàng - Warning/Withdraw).

---

## BƯỚC 5 — Self-check
- **Vi phạm Design System?** Không. Dùng đúng bo tròn thẻ, Primary #d52020 cho các Action chính, xám nhạt cho nền SaaS. Typography phân cấp rõ tiêu đề (Title JD) và phụ đề (Tên DN, Lịch trình).
- **Bỏ sót màn hình nào?** Phủ đủ 3 mốc: Tìm tin, Điền Form, và Quản lý Đơn (Giao diện thay đổi theo 4 state status khác nhau).
- **Trải nghiệm Clean & Minimal?** Bỏ các bảng dữ liệu quá khô khan, dùng giao diện `Kanban board` hoặc `Card` đối với quản lý tín trình để tạo cảm giác khích lệ người trẻ ứng tuyển (Sinh viên).

> **BATCH GENERATION Command:** MCP Generator sẽ gom các state này vào một Layout Dashboard chuyên nghiệp trên nền tảng hiển thị.

https://stitch.withgoogle.com/projects/6023372471380982738