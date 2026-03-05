# Issue 93 — Evaluation Management: UI/UX Design Document

> **Stitch Project:** `IOCv2 - Issue93 Evaluation Management UI`
> **Design System:** Light Mode | Be Vietnam Pro | ROUND_FULL | Primary #d52020

---

## BƯỚC 1 — Phân tích Issue

### User Goal + Business Objective

| Dimension | Detail |
|---|---|
| **User Goal** | Mentor cần công cụ linh hoạt để tự thiết lập các đợt đánh giá (Chu kỳ), rập khuôn các Tiêu chí, và tiến hành chấm điểm cho sinh viên (cả chấm nhanh hàng loạt lẫn chấm chi tiết cá nhân) |
| **Business Objective** | Số hóa toàn bộ quá trình đánh giá thực tập thay vì Excel rời rạc. Tối ưu thời gian cho Mentor (Batch Upsert) nhưng vẫn giữ được chất lượng phản hồi chuyên sâu (Individual Feedback) |
| **Primary Actor** | Mentor (Doanh nghiệp) |
| **Secondary Actor** | Sinh viên (Người xem điểm sau khi Publish) |

### Functional Requirements

| # | Requirement | Trọng tâm UI |
|---|---|---|
| FR-01 | Quản lý Chu kỳ đánh giá (CRUD Cycle) | Data Table liệt kê chu kỳ, Drawer thêm/sửa, Dialog xóa |
| FR-02 | Quản lý Bộ tiêu chí (CRUD Criteria) | Màn hình chi tiết Chu kỳ chứa Tab Tiêu chí, quản lý danh sách tiêu chí và tổng điểm |
| FR-03 | Chấm điểm nhanh hàng loạt (Batch Grading) | Giao diện **Grid (Excel-like)**. Cột là Tiêu chí, Dòng là Sinh viên. Editable cells cho phép gõ số trực tiếp. |
| FR-04 | Chấm điểm chi tiết cá nhân (Individual) | Slide-over Drawer chứa Form dài để Mentor nhập bình luận cho từng tiêu chí và nhận xét chung. |
| FR-05 | Công bố điểm (Publish) | Nút "Công bố" (có thể chọn toàn bộ hoặc từng sinh viên) để chốt điểm cho SV xem. |

### Edge Cases + Validation Rules

| Rule | Behavior tại UI |
|---|---|
| `score` > `max_score` | Grid cell đổi viền đỏ, show tooltip "Vượt quá điểm tối đa". Nút `Lưu tất cả` bị disabled. |
| Đang nhập Grid mà thoát trang | Hiện Browser Alert "Bạn có thay đổi chưa lưu, xác nhận rời đi?" |
| Xóa Chu kỳ/Tiêu chí đã có điểm | Nút Xóa bị disabled (hoặc call API bị lỗi thì hiện Error Toast 🔴). |
| Chu kỳ chưa có Tiêu chí nào | Màn hình Chấm điểm hiện Empty State nhắc nhở "Vui lòng tạo tiêu chí trước khi chấm điểm". |
| Công bố (Publish) | Trạng thái chuyển từ Draft/Submitted sang Published (Màu sắc badge thay đổi rõ rệt). |

---

## BƯỚC 2 — Thiết kế UX Flow

### User Flow Step-by-step

```mermaid
flowchart TD
    A[Mentor đăng nhập] --> B[Sidebar: Đánh giá & Điểm số]
    B --> C[Danh sách Chu kỳ (Cycles)]
    
    C --> D[Tạo/Sửa Chu kỳ qua Drawer]
    
    C -->|Click vào 1 Chu kỳ| E[Màn hình Chi tiết Chu kỳ]
    E --> F[Tabs Content]
    
    F -->|Tab 1: Tiêu chí| G[Quản lý Criteria List]
    G --> H[Thêm/Sửa Tiêu chí qua Drawer]
    
    F -->|Tab 2: Chấm điểm| I[Danh sách Sinh viên & Bảng Grid]
    I --> J{Mentor chọn cách chấm?}
    
    J -->|Gõ thẳng vào cell| K[Trạng thái bảng: Có edits chưa lưu]
    K --> L[Click 'Lưu Tất Cả' → Batch API → Success Toast]
    
    J -->|Click ⋮ chọn Chấm chi tiết| M[Mở Drawer: Chấm điểm Cá nhân]
    M --> N[Nhập điểm & Comment → Save → Success Toast]
    
    I --> O[Chọn SV đã chấm (Draft) → Click Publish]
    O --> P[Xác nhận Công bố → Trạng thái thành Published]
```

### Danh sách màn hình cần có

| # | Màn hình | Device |
|---|---|---|
| 1 | **Cycle Management List** | Desktop / Tablet |
| 2 | **Cycle Details - Criteria Tab** | Desktop / Tablet |
| 3 | **Cycle Details - Grading Grid Tab** | Desktop (Cần không gian rộng) |
| 4 | **Form Drawers** (Tạo Chu kỳ, Tạo Tiêu chí, Form Chấm riêng) | Desktop / Mobile |
| 5 | **Confirmation Dialogs** (Xóa, Rời trang chưa lưu, Publish) | All |

### Trạng thái đặc biệt
- **Unsaved Changes State**: Nổi bật thông báo "Có thay đổi chưa lưu" trên Header của Grid. Nút Save sáng lên (Primary).
- **Read-only State**: Khi điểm đã `Published`, các cell trong Grid biến thành text box chỉ xếp để hiển thị, không gõ được trừ khi bấm nút "Chỉnh sửa lại" (đưa về Draft).

---

## BƯỚC 3 — Thiết kế UI trong Stitch

### Design Tokens (Kế thừa từ Design System)
- **Font Family**: Be Vietnam Pro
- **Color Primary**: `#d52020` (Nút hành động chính, active tab, focus border).
- **Shape**: `ROUND_FULL` (9999px) cho Button, Badge, Input.
- **Surface**: Background Trắng (White) và Xám nhạt (Gray-50) để tách biệt Layout.

### Mở rộng Component Library (Đề xuất thêm)

Do tính chất "nhập điểm hàng loạt", cần bổ sung/tối ưu các component sau để không phá vỡ UX:

| Component | Status | Props / Specs |
|---|---|---|
| **`EditableDataGrid`** | 🆕 New | Bảng Excel-like. Có input number ẩn trong cell. Khi hover hiện outline, khi click focus hiện cursor. `columns`, `data`, `onChange`, `validationRules` |
| **`ScoreInputCell`** | 🆕 New | `<input type="number">` không có spinner arrows, căn phải (Right aligned). Background transparent. Error state: text đỏ sậm, bg đỏ nhạt (red-50). |
| **`TabsLayout`** | ✅ Existing | `items: { label, key }[]`, `activeKey`. Dùng để chuyển tab Tiêu chí và Danh sách Sinh viên. |
| **`StatusBadge`** | ✅ Cập nhật | `Draft` (Gray), `Submitted` (Blue), `Published` (Green). |

---

## BƯỚC 4 — Component Tree & Layout Structure

### 1. Layout Màn hình Chi tiết Chu kỳ (Cycle Details)

```text
┌─────────────────────────────────────────────────────────┐
│                    BROWSER VIEWPORT                      │
├──────────┬──────────────────────────────────────────────┤
│  SIDEBAR │  Breadcrumb: Đánh giá / [Tên Chu kỳ]           │
│  (240px) │──────────────────────────────────────────────│
│          │  [H3] Đánh giá Giữa kỳ (Ngày bắt đầu - kết thúc)│
│          │  [Tabs: 1. Bộ Tiêu chí  |  2. Bảng Chấm Điểm] │
│          ├──────────────────────────────────────────────┤
│          │  ▼ TAB 2: BẢNG CHẤM ĐIỂM (GRID VIEW)          │
│          │  [🔍 Tìm kiếm]                [💾 Lưu Tât Cả] │
│          │                                               │
│          │  ┌──────────────────────────────────────┐     │
│          │  │ Thí sinh   │ Khái niệm│ Code │ Thái độ │     │
│          │  ├──────────────────────────────────────┤     │
│          │  │ Nguyễn V.A │  [ 8 ]   │ [ 7 ]│  [ 9 ]  │ ⋮ │
│          │  ├──────────────────────────────────────┤     │
│          │  │ Lê Thị B   │  [ 9 ]   │ [ 9 ]│  [10 ]  │ ⋮ │
│          │  └──────────────────────────────────────┘     │
└──────────┴──────────────────────────────────────────────┘
* Nút (⋮) ở cuối mở ra "Chấm chi tiết" (Drawer) hoặc "Công bố điểm".
```

### 2. Component Tree: Grading Grid Tab
```
CycleDetailsPage
├── PageHeader (Breadcrumbs, Title, Summary info)
├── TabsNav (Active: "Bảng Chấm Điểm")
└── TabContent
    ├── Toolbar
    │   ├── SearchInput (Filter sinh viên)
    │   ├── FilterBadge (Lọc theo trạng thái: Đã chấm, Chưa chấm)
    │   └── ActionGroup
    │       ├── Text: "Có 3 thay đổi chưa lưu" (Vàng/cam báo động nhẹ)
    │       ├── Button (variant="outline", label="Hủy")
    │       └── Button (variant="primary", label="Lưu Tất Cả", disabled=!hasChanges)
    └── EditableDataGrid
        ├── TableHead
        │   ├── Th: "Sinh viên" (fixed width)
        │   ├── Th (Dynamic loop qua criteria): "Tên tiêu chí (Max: 10)"
        │   ├── Th: "Tổng điểm"
        │   ├── Th: "Trạng thái"
        │   └── Th: "Hành động"
        └── TableBody
            └── Tr (loop students)
                ├── Td: UserCard (Avatar + Name + MSSV)
                ├── Td (Dynamic): ScoreInputCell (value, onChange, max=criteria.max)
                ├── Td: TotalScore Text
                ├── Td: StatusBadge (Draft/Published)
                └── Td: ActionDropdown ("Chấm chi tiết bằng form", "Công bố")
```

### 3. Component Tree: Individual Grading Drawer (Chấm chi tiết cá nhân)
```
SlideoverDrawer (width: 600px - Rộng để dễ viết comment)
├── DrawerHeader
│   ├── UserCard (Tên SV, MSSV)
│   └── StatusBadge (Trạng thái phiếu điểm)
├── DrawerBody
│   ├── CriteriaSection (Loop theo tiêu chí)
│   │   ├── TitleRow (Tên tiêu chí + "Điểm tối đa: 10")
│   │   ├── NumberInput (Label: "Điểm đạt được")
│   │   └── TextArea (Label: "Nhận xét tiêu chí (Không bắt buộc)", rows=3)
│   ├── Divider
│   └── GeneralSection
│       └── TextArea (Label: "Nhận xét tổng quan", rows=4, placeholder="Đánh giá thái độ và năng lực chung...")
└── DrawerFooter
    ├── Button (variant="outline", label="Lưu Nháp (Draft)")
    └── Button (variant="primary", label="Lưu & Xác nhận (Submitted)")
```

### 4. Responsive Behavior
- **Desktop (>=1024px)**: Grid view hiển thị đủ cột. Form Drawer chiếm 600px bám phải.
- **Tablet (768px - 1024px)**: Bảng Grid có horizontal scroll (cuộn ngang). Cột "Sinh viên" bị sticky (đóng băng) bên trái để không mất context.
- **Mobile (<768px)**: 
  - KHÔNG MỞ GRID. UX Grid trên mobile là ác mộng do quá chật.
  - Thay bằng **Card View**. Mỗi sinh viên là một Card. Bấm vào Card sẽ nhảy thẳng sang Drawer "Chấm điểm chi tiết". Không hỗ trợ Batch Upsert trên Mobile.

### 5. Accessibility Notes
- **Keyboard Navigation trên Grid**: Phải hỗ trợ dùng phím `Tab`, `Shift+Tab`, `Arrow Cùn`, `Enter` để di chuyển focus giữa các ô nhập điểm (như Excel).
- **ARIA Live**: Khi nhập quá tay lố điểm Max, đọc thông báo lỗi ngay lập tức bằng `aria-live="assertive"`.
- **Contrast**: `ScoreInputCell` đang Focus phải đổi màu viền sang đỏ `#d52020` đủ tương phản thay vì outline xanh xám mặc định của trình duyệt. 

---

## BƯỚC 5 — Self-check Visual & System Compliance

1. **Có vi phạm design system không?**
   - Không. Outline focus dùng chính màu brand `--color-primary`. Các radius dùng `ROUND_FULL` cho Input/Button theo đúng thiết lập Stitch project. Tab dùng gạch chân đỏ tinh tế.
2. **Có sử dụng sai token không?**
   - Màu validation error dùng `#dc2626` (Danger / Red-600) chứ không xài lẫn `#d52020` (Brand Primary). Rất rõ ràng ngữ nghĩa.
3. **SaaS Professional & Data Centric?**
   - Chức năng Grid Data cực kỳ chuyên nghiệp (Data-centric), loại bỏ các padding thừa trong Table để nén được nhiều Cell nhất có thể (SaaS density high).
4. **Clarity > Flashy?**
   - Tránh việc màu mè ô nhấp nháy khi có điểm chưa lưu. Chỉ dùng header banner "Có 3 thay đổi chưa lưu" và nhả disable nút Lưu. Giao diện phẳng, tinh gọn. Mọi thứ quy củ như Excel.

**Kết luận:** Design structure hoàn toàn đạt chuẩn IOCv2 và sẵn sàng cho Frontend Dev (FFE-CMP) chuyển hóa thành code Next.js / Tailwind.

https://stitch.withgoogle.com/projects/5519768010363165592