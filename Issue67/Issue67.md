> **Note:** FE có thể edit lại AC và tùy chỉnh UI/UX cho phù hợp với dự án. BE cũng có thể chỉnh lại AC khi thấy chỗ nào không ổn hoặc thiếu logic

# **AC-01: Hiển thị thông tin kỳ thực tập hiện tại**

**Given**: Student đã đăng nhập thành công và đang được enrolled vào một kỳ thực tập có trạng thái ACTIVE

**When**: Student truy cập trang Dashboard/Home

**Then**: Hệ thống hiển thị thông tin **kỳ thực tập hiện tại** gồm:

- Tên kỳ thực tập

- Ngày bắt đầu và ngày kết thúc

- Trạng thái kỳ thực tập (ACTIVE)

- Số ngày còn lại đến khi kỳ kết thúc

- Trạng thái placement của bản thân (Placed / Unplaced)

- Tên Enterprise (nếu đã Placed)

# **AC-02: Hiển thị thông tin khi kỳ thực tập sắp bắt đầu (UPCOMING)**

**Given**: Student đã được enrolled vào một kỳ thực tập có trạng thái UPCOMING

**When**: Student truy cập trang Dashboard/Home

**Then**:

- Hệ thống hiển thị thông tin kỳ thực tập sắp tới

- Số ngày còn lại đến khi kỳ bắt đầu

- Trạng thái hiển thị: *"Sắp bắt đầu"*

- Trạng thái placement hiển thị Unplaced (vì kỳ chưa bắt đầu)

# **AC-03: Countdown timer hoạt động chính xác**

**Given**: Student đang xem thông tin kỳ thực tập trên Dashboard

**When**: Xem phần đếm ngược

**Then**:

- Nếu kỳ đang ACTIVE: hiển thị *"Còn \[X\] ngày đến khi kết thúc"*

- Nếu kỳ đang UPCOMING: hiển thị *"Còn \[X\] ngày đến khi bắt đầu"*

- Nếu còn dưới 7 ngày: số ngày hiển thị với màu cảnh báo (vàng hoặc đỏ) để nhấn mạnh deadline

- Số ngày được tính chính xác dựa trên ngày hiện tại của hệ thống

# **AC-04: Không hiển thị thông tin khi không có kỳ thực tập**

- **Given**: Student chưa được enrolled vào bất kỳ kỳ thực tập nào, hoặc tất cả kỳ thực tập đã ENDED hoặc CLOSED

- **When**: Student truy cập trang Dashboard/Home

- **Then**:

  - Hệ thống hiển thị thông báo: *"Bạn hiện không có kỳ thực tập nào đang diễn ra"*

  - Không hiển thị phần đếm ngược

# **AC-05: Hiển thị đúng trạng thái Placed / Unplaced**

- **Given**: Student đang được enrolled vào kỳ thực tập ACTIVE

- **When**: Student xem thông tin kỳ thực tập trên Dashboard

- **Then**:

  - Nếu **Placed**: hiển thị badge *"Đã có công ty"* kèm tên Enterprise

  - Nếu **Unplaced**: hiển thị badge cảnh báo *"Chưa có công ty"* kèm gợi ý: *"Hãy apply vào các vị trí tuyển dụng để tìm công ty thực tập"* và nút dẫn đến trang Job Postings

# **AC-06: Điều hướng đến trang chi tiết kỳ thực tập**

- **Given**: Student đang xem thông tin kỳ thực tập trên Dashboard

- **When**: Click vào tên kỳ thực tập hoặc nút "Xem chi tiết"

- **Then**:

  - Hệ thống điều hướng đến trang chi tiết kỳ thực tập của Student

  - Hiển thị đầy đủ thông tin: timeline, trạng thái placement, tên Enterprise (nếu có)

# **AC-07: Cập nhật trạng thái khi placement thay đổi**

- **Given**: Student đang xem Dashboard và vừa Accept offer hoặc được Uni Admin gán Enterprise

- **When**: Trạng thái placement thay đổi từ Unplaced → Placed

- **Then**:

  - Thông tin tự động cập nhật trạng thái sang Placed và hiển thị tên Enterprise

  - Nếu không hỗ trợ realtime: thông tin cập nhật sau tối đa 30 giây hoặc khi Student chuyển tab và quay lại

# **AC-08: Quyền truy cập theo vai trò**

- **Given**: Có các tài khoản với role khác nhau

- **When**: Mỗi role truy cập Dashboard

- **Then**:

  - **Student**: chỉ thấy thông tin kỳ thực tập của bản thân

  - **Uni Admin / Enterprise HR / Sup Admin**: không hiển thị mục thông tin này trên Dashboard của họ (mỗi role có Dashboard riêng)