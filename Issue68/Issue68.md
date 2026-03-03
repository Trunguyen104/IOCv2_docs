> **Note:** FE có thể edit lại AC và tùy chỉnh UI/UX cho phù hợp với dự án. BE cũng có thể chỉnh lại AC khi thấy chỗ nào không ổn hoặc thiếu logic

# **AC-01: Truy cập thông tin "Kỳ thực tập đang diễn ra"**

**Given**: Enterprise HR hoặc Mentor đã đăng nhập thành công

**When**: Truy cập Dashboard hoặc menu "Internship Term / Kỳ thực tập"

**Then**:

- Hệ thống hiển thị thông tin **"Kỳ thực tập đang diễn ra"** cho các trường mà Enterprise đang có sinh viên thực tập

- Thông tin hiển thị đúng theo quyền của người dùng (HR/Mentor)

# AC-02: Hiển thị thông tin cơ bản & timeline của kỳ Active

**Given**: Enterprise HR hoặc Mentor đang xem “Kỳ thực tập đang diễn ra”\
**When:** Dữ liệu tải xong\
**Then:**

- Hiển thị thông tin cơ bản của kỳ: Tên kỳ, Ngày bắt đầu, Ngày kết thúc, Trạng thái = Active
- Hiển thị timeline trực quan gồm: mốc Start → End, vị trí “Hôm nay” trên timeline, số ngày còn lại đến khi kết thúc

# **AC-03: Hiển thị các deadline liên quan chấm điểm/đánh giá**

**Given**: Kỳ thực tập ACTIVE có cấu hình các mốc deadline đánh giá/chấm điểm (do Uni Admin thiết lập)

**When**: Enterprise HR/Mentor xem phần "Deadlines" trong term

**Then**:

- Hiển thị tối thiểu các deadline cần thiết cho Enterprise HR/Mentor:

  - Deadline nộp đánh giá (evaluation submission deadline)

  - Deadline chốt điểm/grade (grading deadline) — nếu tách riêng

- Mỗi deadline hiển thị: ngày đến hạn và countdown (còn X ngày/giờ)

- Nếu còn dưới ngưỡng cảnh báo (≤ 7 ngày): deadline được highlight để nhấn mạnh

- Nếu Uni Admin chưa cấu hình deadline: hiển thị thông báo "Chưa có thông tin deadline cho kỳ thực tập này"

# AC-04: Trường hợp có nhiều kỳ Active (nhiều trường)

**Given:** Enterprise đang có sinh viên thực tập thuộc nhiều trường và có thể có nhiều term Active đồng thời\
**When:** Enterprise HR/Mentor mở màn xem term\
**Then:**

- Hệ thống hiển thị danh sách các term Active theo từng trường (mỗi item có timeline + deadlines)

- Có bộ lọc theo Trường (School) để HR/Mentor chọn xem nhanh đúng term cần theo dõi

# **AC-05: Không có kỳ ACTIVE**

**Given**: Enterprise HR/Mentor đăng nhập nhưng hiện không có term nào ACTIVE liên quan (chưa có sinh viên được tiếp nhận, hoặc term đã **ENDED hoặc CLOSED**)

**When**: Truy cập Dashboard/Term page

**Then**:

- Hiển thị empty state: *"Hiện chưa có kỳ thực tập nào đang diễn ra"*

- Không hiển thị countdown/deadlines

# AC-06: Quyền truy cập & phạm vi dữ liệu

**Given:** Enterprise HR/Mentor đang đăng nhập với tài khoản thuộc Enterprise A\
**When:** Xem thông tin term\
**Then:**

- Chỉ xem được term của các trường mà Enterprise A có sinh viên thực tập/được gán/đã xác nhận tiếp nhận
- Không xem được term của các trường/Enterprise khác; nếu truy cập URL không hợp lệ thì trả 403 hoặc thông báo “Không có quyền truy cập”

# **AC-07: Cập nhật khi Uni Admin thay đổi term**

**Given**: Uni Admin cập nhật ngày kết thúc hoặc đóng kỳ thực tập

**When**: Enterprise HR/Mentor refresh trang hoặc tải lại

**Then**:

- Timeline và countdown phản ánh đúng dữ liệu mới

- Nếu term chuyển sang **ENDED hoặc CLOSED**: thông tin "Kỳ thực tập đang diễn ra" không còn hiển thị term đó — chuyển sang empty state hoặc hiển thị term ACTIVE khác nếu còn tồn tại