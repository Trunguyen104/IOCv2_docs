## 1. Tổng quan & Bối cảnh (Overview & Context)

## Vấn đề (Problem):

Trong quản lý dự án Agile/Scrum, việc phân định rõ ràng giữa "Việc cần làm trong tương lai" (Product Backlog) và "Việc cam kết làm trong kỳ này" (Sprint Backlog) là cốt lõi. Hiện tại, nếu thiếu một màn hình Backlog chuyên biệt, sinh viên sẽ gặp khó khăn trong việc lập kế hoạch, sắp xếp thứ tự ưu tiên và gom nhóm công việc theo Sprint hoặc Epic.

## Giá trị Nghiệp vụ (Business Value):

- **Lập kế hoạch hiệu quả:** Giúp team dễ dàng chọn lọc và cam kết khối lượng công việc phù hợp cho từng Sprint.

- **Tổ chức khoa học:** Phân tầng công việc rõ ràng (Epic > Story > Task) giúp quản lý scope tốt hơn.

- **Linh hoạt:** Thao tác kéo thả (Drag & Drop) giúp việc điều chuyển task giữa các Sprint hoặc trả về Backlog diễn ra nhanh chóng.

## Đối tượng (Actor):

- **Primary Actor:** Sinh viên (Student) — đặc biệt là vai trò Scrum Master/Team Leader (để start/complete sprint) và Team Member.

- **Secondary Actors:** Mentor — xem backlog để đánh giá lộ trình dự án.

---

## 2. Luồng Người dùng (User Flow)

## 2.1. Luồng Xem & Quản lý Backlog (Mặc định)

1. Student truy cập tab **"Backlog"**.

2. Hệ thống hiển thị giao diện chia làm 3 phần chính:

   - **Panel trái (Epic Panel):** Danh sách các Epic (có thể ẩn/hiện). Chọn một Epic để lọc các issue thuộc Epic đó bên phải.

   - **Khu vực Sprint (Sprint Backlog):** Hiển thị các Sprint đang Active (Đang chạy) hoặc Future (Planned/Sắp tới).

   - **Khu vực Backlog chung (Product Backlog):** Hiển thị danh sách các issue chưa được gán vào Sprint nào (nằm dưới cùng).

3. Trên mỗi issue row (hàng): Hiển thị Tóm tắt, Type, Priority, Story Points, Assignee, Trạng thái.

## 2.2. Luồng Điều phối Công việc (Drag & Drop)

1. Student nhấn giữ một hàng issue.

2. Kéo thả hàng đó:

   - Từ Product Backlog lên Sprint Backlog (Lên kế hoạch) và ngược lại.

   - Từ Sprint này sang Sprint khác.

   - Sắp xếp lại thứ tự (Reorder) trong cùng một danh sách để thay đổi độ ưu tiên.

3. Hệ thống cập nhật tức thì vị trí và thuộc tính Sprint của issue.

## 2.3. Luồng Quản trị Sprint (Lifecycle)

1. **Tạo Sprint:** Tại khu vực Product Backlog, nhấn nút "Tạo Sprint" → Hệ thống tạo ra một box Sprint mới rỗng (ví dụ: Sprint 2).

2. **Bắt đầu Sprint:** Với Sprint chưa bắt đầu (Planned), nhấn nút "Bắt đầu Sprint" → Hiển thị modal nhập thông tin (Ngày bắt đầu, Ngày kết thúc) → Xác nhận → Sprint chuyển sang trạng thái Active. (⚠️ Chú ý: Thời gian Sprint phải kéo dài từ 7 đến 28 ngày)

3. **Hoàn thành Sprint:** Với Sprint đang chạy (Active Sprint), nhấn nút "Hoàn thành Sprint" → Hiển thị modal tổng kết (Số task hoàn thành, Số task chưa xong) → Chọn hành động với task chưa xong (chuyển sang Sprint mới, Sprint kế tiếp, hoặc trả về Backlog) → Xác nhận → Sprint đóng lại (Completed).

## 2.4. Luồng Tìm kiếm & Lọc

1. Student sử dụng thanh công cụ để tìm kiếm theo từ khóa hoặc filter (Assignee, Type, Priority...).

2. Kết quả lọc được áp dụng đồng thời cho cả khu vực Sprint và Product Backlog để dễ dàng tìm kiếm task ở mọi nơi.

---

## 3. Tiêu chí Chấp nhận (Acceptance Criteria)

## AC-BL-01 — Hiển thị Giao diện Backlog chuẩn

- **Given:** Student truy cập tab Backlog.

- **When:** Trang tải xong.

- **Then:**

  - Hiển thị đúng cấu trúc phân tầng: Sprint Active (trên cùng) > Future Sprints (Planned) > Product Backlog (dưới cùng).

  - Mỗi issue hiển thị trên một hàng ngang với đủ thông tin tóm tắt.

  - Tổng số Story Points (hoặc issue count) được tính tổng và hiển thị trên header của mỗi Sprint.

## AC-BL-02 — Epic Panel (Danh sách ẩn)

- **Given:** Student muốn lọc theo Epic.

- **When:** Click nút "Epic" (hoặc toggle panel trái).

- **Then:**

  - Panel trái mở ra, hiển thị danh sách các Epic. [⚠️ Architecture Violation: Tài liệu yêu cầu "màu sắc đại diện" nhưng Model trong Code không lưu trường Color cho Epic/WorkItem]

  - Click vào một Epic → Toàn bộ list bên phải chỉ hiển thị các issue (nội dung sprint backlog và product backlog) thuộc Epic đó.

  - Có thể đóng panel để mở rộng không gian làm việc.

  - Khi tạo epic cần có Tên epic (bắt buộc, tối đa 255 ký tự) và Mô tả epic (optional, tối đa 2000 ký tự). [⚠️ Architecture Violation: Tài liệu cũ yêu cầu "ngày kết thúc" nhưng Command `CreateEpicCommand` không hỗ trợ trường này, khi tạo Epic Date/Due Date sẽ null]

## AC-BL-03 — Thao tác Kéo thả (Cross-Backlog Drag & Drop)

- **Given:** Có ít nhất 1 Sprint và Product Backlog.

- **When:** Kéo một issue từ danh sách này sang danh sách khác.

- **Then:**

  - Issue di chuyển mượt mà.

  - Dữ liệu `sprint_id` của issue được cập nhật tương ứng.

  - Tổng Story Points/Count của Sprint nguồn và đích được tính toán lại ngay lập tức.

  - Cho phép sắp xếp lại vị trí (rank) trong cùng một list.

## AC-BL-04 — Tạo Sprint (Create Sprint)

- **Given:** Đang ở màn hình Backlog.

- **When:** Nhấn nút "Tạo Sprint".

- **Then:**

  - Một Sprint mới được tạo ra nằm phía trên Product Backlog.

  - Tên mặc định tăng dần (ví dụ: Sprint 1, Sprint 2...).

  - Trạng thái mặc định là "Planned" (Sắp bắt đầu/Chưa bắt đầu).

  - Hiển thị modal nhập: Tên sprint (bắt buộc, tối đa 200 ký tự), mục tiêu sprint (optional, tối đa 1000 ký tự).

## AC-BL-05 — Bắt đầu Sprint (Start Sprint)

- **Given:** Một Sprint ở trạng thái Planned và có ít nhất 1 issue bên trong.

- **When:** Nhấn nút "Bắt đầu Sprint".

- **Then:**

  - Hiển thị modal nhập: Thời gian (StartDate và EndDate, bắt buộc).

  - Đảm bảo validation: Thời lượng (EndDate - StartDate) phải từ 7 ngày đến 28 ngày. EndDate phải lớn hơn StartDate.

  - Bắt buộc kiểm tra dự án không có Sprint nào khác đang ở trạng thái Active (chỉ cho phép 1 Sprint Active).

  - Sau khi xác nhận: Sprint chuyển sang trạng thái "Active".

  - Sprint Active sẽ hiển thị trên Bảng công việc (Board).

  - Chỉ cho phép 1 Sprint Active tại một thời điểm. [⚠️ Architecture Violation: Tài liệu cũ yêu cầu "tùy rule dự án" nhưng Code hiện tại Hardcode chỉ cho phép 1 Sprint Active]

## AC-BL-06 — Hoàn thành Sprint (Complete Sprint)

- **Given:** Một Sprint đang Active.

- **When:** Nhấn nút "Hoàn thành Sprint".

- **Then:**

  - Hiển thị modal tổng kết đếm số task đã Done và chưa Done.

  - Chặn các thay đổi nếu validation đầu vào không hợp lệ. Yêu cầu chọn 1 trong 3 hành động với Issue chưa hoàn thành:
    1. Trả về Backlog (`ToBacklog`).
    2. Chuyển vào Sprint tiếp theo (`ToNextPlannedSprint` - có thể chỉ định chính xác TargetSprintId, nếu không backend sẽ lấy Sprint Planned đầu tiên theo StartDate).
    3. Tạo Sprint mới (`CreateNewSprint` - có thể nhập tên, nếu không backend tự đặt là "[Tên Sprint cũ] (Continued)").

  - Sau khi xác nhận: Sprint cũ chuyển sang trạng thái "Completed".

## AC-BL-07 — Tìm kiếm & Bộ lọc (Search & Filter)

- **Given:** Backlog có nhiều issue.

- **When:** Nhập từ khóa hoặc chọn filter (Assignee, Type, Priority, Status...).

- **Then:**

  - Lọc hiển thị issue trên TOÀN BỘ các danh sách (cả Sprint lẫn Product Backlog)

  - Các option có trong filter: tóm tắt, người được giao, loại, priority, status, ngày tạo, ngày hoàn thành. 

  - Các Sprint không chứa issue nào khớp với bộ lọc có thể được ẩn đi hoặc hiển thị rỗng (tùy UX).

## AC-BL-08 — CRUD Issue trên Backlog

- **Given:** Student muốn thêm nhanh task.

- **When:** Nhấn nút "+ Tạo issue" ở dưới cùng của một Sprint hoặc Product Backlog.

- **Then:**

  - Hiển thị modal nhập: Tóm tắt (Title, bắt buộc, max 255), Mô tả (Description), Loại (Type: Epic | UserStory | Task | Subtask), Người thực hiện (AssigneeId), Độ ưu tiên (Priority: Low | Medium | High | Critical), Epic (ParentId), Hạn chót (DueDate), và Story Point (lớn hơn hoặc bằng 0). 

  - [⚠️ Architecture Violation: Tài liệu yêu cầu "Trạng thái", "Thời gian (Start/End Date)" và "Tag" nhưng Backend Command (`CreateWorkItemCommand`) không nhận các input này. "Start/End Date" được thay bằng "Hạn chót (DueDate)". Trạng thái mặc định được backend Hardcode khởi tạo là Todo].

  - Task mới tạo sẽ tự động được đưa vào Sprint tương ứng nếu tạo trong danh sách Sprint (thông qua truyền `SprintId`).

  - Có menu context (ba chấm) trên từng row để Sửa/Xóa task.

---

## 4. Đặc tả kỹ thuật (Technical Notes)

- **Drag & Drop Library:** Tiếp tục sử dụng `react-beautiful-dnd` hoặc `dnd-kit`. Lưu ý xử lý logic `Droppable` cho nhiều container (mỗi Sprint là một Droppable area). Mọi thao tác đổi vị trí sẽ được map qua `BacklogOrder` và `BoardOrder`.

- **Sprint Logic:** Cần validate logic Scrum theo API: Đảm bảo thời gian 7 - 28 ngày khi Start, không cho phép có 2 Sprint active trong một Project, xử lý logic Complete Sprint (3 options gán với string nhận trên server).

- **Performance:** Khi kéo thả giữa 2 list lớn, cần tối ưu render (dùng `React.memo` hoặc `virtualization` nếu backlog có hàng trăm item).