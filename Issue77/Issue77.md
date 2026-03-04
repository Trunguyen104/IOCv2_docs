**Feat:** Xem danh sách sinh viên đã tiếp nhận

Đủ thông tin rồi, mình viết AC luôn:

---

## User Story

> Là Enterprise, tôi muốn xem danh sách sinh viên được gán vào công ty mình và trạng thái tiếp nhận của họ, để tôi có thể quản lý và đánh giá các sinh viên đó.

---

**AC-01: Hiển thị danh sách sinh viên**

- **Given**: Enterprise HR/Admin đã login và truy cập trang "Danh sách sinh viên thực tập"

- **When**: Xem danh sách

- **Then**:

  - Hiển thị tất cả sinh viên đã được Uni Admin Placed vào Enterprise trong kỳ hiện tại

  - Mỗi dòng hiển thị: họ tên, MSSV, email, SĐT, trường, chuyên ngành, trạng thái tiếp nhận, mentor được assign, vị trí/dự án

  - Hiển thị tổng số sinh viên

  - Nếu chưa có sinh viên nào: hiển thị empty state *"Chưa có sinh viên nào được gán vào công ty"*

---

**AC-02: Trạng thái tiếp nhận**

- **Given**: Uni Admin vừa Placed sinh viên vào Enterprise

- **When**: Enterprise HR/Admin xem danh sách

- **Then**: Sinh viên hiển thị với các trạng thái:

| Trạng thái | Ý nghĩa | Badge |
| --- | --- | --- |
| Pending | Uni Admin đã Placed, Enterprise chưa xác nhận | Vàng |
| Accepted | Enterprise đã xác nhận tiếp nhận | Xanh |
| Rejected | Enterprise từ chối tiếp nhận | Đỏ |

---

**AC-03: Accept sinh viên**

- **Given**: Sinh viên đang ở trạng thái **Pending**

- **When**: HR/Admin click nút "Accept"

- **Then**:

  - Hệ thống hiển thị confirm dialog: *"Bạn có chắc muốn tiếp nhận \[Tên sinh viên\] không?"*

  - Nếu confirm: trạng thái chuyển sang **Accepted**

  - Hệ thống gửi notification cho Uni Admin và Student về việc tiếp nhận thành công

  - Hiển thị toast: *"Đã tiếp nhận sinh viên thành công"*

---

**AC-04: Reject sinh viên**

- **Given**: Sinh viên đang ở trạng thái **Pending**

- **When**: HR/Admin click nút "Reject"

- **Then**:

  - Hệ thống hiển thị dialog yêu cầu nhập **lý do từ chối** (bắt buộc)

  - Nếu confirm: trạng thái chuyển sang **Rejected**

  - Hệ thống gửi notification cho **Uni Admin** kèm lý do từ chối để xử lý lại

  - Hệ thống gửi notification cho **Student** thông báo bị từ chối

  - Hiển thị toast: *"Đã từ chối tiếp nhận sinh viên"*

---

**AC-05: Assign Mentor và Dự án**

- **Given**: Sinh viên đang ở trạng thái **Accepted**

- **When**: HR click nút "Assign" trên dòng sinh viên

- **Then**:

  - Hiển thị form assign gồm: dropdown chọn Mentor (danh sách Mentor thuộc Enterprise), tên vị trí/dự án

  - Sau khi lưu: thông tin Mentor và dự án hiển thị trên dòng sinh viên

  - Hệ thống gửi notification cho **Mentor** được assign và **Student** biết mentor của mình

- **And**: Chỉ **HR** mới có quyền thực hiện thao tác này, Enterprise Admin và Mentor không assign được

---

**AC-06: Tìm kiếm sinh viên**

- **Given**: Enterprise HR/Admin đang xem danh sách sinh viên

- **When**: Nhập từ khóa vào ô tìm kiếm

- **Then**:

  - Sau 300ms (debounce), danh sách lọc theo: họ tên, MSSV, email, tên trường

  - Hiển thị số kết quả tìm được

  - Nếu không có kết quả: hiển thị *"Không tìm thấy sinh viên phù hợp"*

---

**AC-07: Lọc danh sách**

- **Given**: Enterprise HR/Admin đang xem danh sách sinh viên

- **When**: Áp dụng bộ lọc

- **Then**: Hệ thống hỗ trợ lọc theo:

  - Trạng thái tiếp nhận (Pending / Accepted / Rejected)

  - Mentor được assign / chưa assign

  - Trường / Chuyên ngành

- **And**: Có thể kết hợp nhiều bộ lọc cùng lúc, có nút Reset để xóa toàn bộ bộ lọc

---

**AC-08: Sắp xếp danh sách**

- **Given**: Enterprise HR/Admin đang xem danh sách sinh viên

- **When**: Click vào tiêu đề cột

- **Then**: Hệ thống hỗ trợ sắp xếp theo:

  - Họ tên (A-Z / Z-A)

  - Ngày được Placed (mới nhất / cũ nhất)

  - Trạng thái tiếp nhận (gom theo nhóm)

- **And**: Mặc định sắp xếp theo ngày Placed mới nhất

---

**AC-09: Xem chi tiết sinh viên**

- **Given**: Enterprise HR/Admin đang xem danh sách

- **When**: Click "Xem chi tiết" trên 1 sinh viên

- **Then**: Trang chi tiết hiển thị đầy đủ:

  - Thông tin cá nhân: họ tên, MSSV, email, SĐT, trường, chuyên ngành

  - Trạng thái tiếp nhận, ngày Placed, Mentor được assign, vị trí/dự án

  - Feedback giữa kỳ / cuối kỳ (nếu đã có)

---

**AC-10: Mentor xem sinh viên được assign**

- **Given**: Mentor đã được HR assign cho sinh viên

- **When**: Mentor login và truy cập danh sách sinh viên

- **Then**:

  - Mentor chỉ thấy danh sách sinh viên được assign cho mình

  - Không thể xem sinh viên của Mentor khác

  - Không có quyền Accept/Reject/Assign

---

**AC-11: Phân trang**

- **Given**: Danh sách có hơn 10 sinh viên

- **When**: Enterprise HR/Admin xem danh sách

- **Then**:

  - Hiển thị 10 sinh viên mỗi trang theo mặc định

  - Có điều khiển phân trang và tùy chọn số bản ghi/trang (10, 20, 50)

  - Hiển thị: *"Hiển thị X–Y của Z sinh viên"*

---

**AC-12: Quyền truy cập**

| Role | Quyền |
| --- | --- |
| Enterprise Admin | Xem tất cả sinh viên, Accept/Reject |
| HR | Xem tất cả sinh viên, Accept/Reject, Assign mentor/dự án |
| Mentor | Chỉ xem sinh viên được assign cho mình |
| Uni Admin / Student / role khác | 403 Forbidden |
