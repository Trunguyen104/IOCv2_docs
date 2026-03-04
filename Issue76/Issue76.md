> **Note:** FE có thể edit lại AC và tùy chỉnh UI/UX cho phù hợp với dự án. BE cũng có thể chỉnh lại AC khi thấy chỗ nào không ổn hoặc thiếu logic

**Feat**: Xem trạng thái thực tập của bản thân

Đủ thông tin rồi, mình viết AC luôn:

---

## User Story

> Là Sinh viên, tôi muốn xem trạng thái placement của bản thân về kỳ thực tập hiện tại, để tôi biết mình đã được gán vào công ty nào chưa.

---

**AC-01: Hiển thị tóm tắt trên Dashboard**

- **Given**: Student đã login và đang ở Dashboard

- **When**: Xem Dashboard

- **Then**:

  - Hiển thị widget/card tóm tắt kỳ thực tập **hiện tại** gồm: tên kỳ, trạng thái placement (Placed/Unplaced), tên Enterprise (nếu Placed)

  - Có nút "Xem chi tiết" dẫn đến trang "Kỳ thực tập của tôi"

  - Nếu chưa enroll kỳ nào: hiển thị *"Bạn chưa tham gia kỳ thực tập nào"*

---

**AC-02: Hiển thị danh sách tất cả các kỳ**

- **Given**: Student truy cập trang "Kỳ thực tập của tôi"

- **When**: Xem danh sách

- **Then**:

  - Hiển thị tất cả các kỳ Student đã tham gia, sắp xếp theo thời gian mới nhất lên đầu

  - Mỗi kỳ hiển thị: tên kỳ, ngày enroll, trạng thái enrollment, trạng thái placement, tên Enterprise (nếu Placed)

  - Kỳ hiện tại được đánh dấu nổi bật (badge "Hiện tại")

  - Nếu chưa tham gia kỳ nào: hiển thị empty state *"Bạn chưa tham gia kỳ thực tập nào"*

---

**AC-03: Xem chi tiết 1 kỳ**

- **Given**: Student đang ở trang "Kỳ thực tập của tôi"

- **When**: Click vào 1 kỳ bất kỳ

- **Then**: Hiển thị chi tiết kỳ đó gồm:

| Trường | Ghi chú |
| --- | --- |
| Tên kỳ thực tập |  |
| Ngày enroll |  |
| Trạng thái enrollment | Active / Withdrawn |
| Trạng thái placement | Placed / Unplaced |
| Tên Enterprise | Trống nếu Unplaced |

---

**AC-04: Trạng thái Unplaced**

- **Given**: Student đang ở kỳ hiện tại và chưa được xếp doanh nghiệp

- **When**: Xem chi tiết kỳ

- **Then**:

  - Badge trạng thái hiển thị **"Chưa được xếp doanh nghiệp"** (màu đỏ/xám)

  - Trường Tên Enterprise hiển thị *"Chưa có"*

---

**AC-05: Trạng thái Placed**

- **Given**: Student đã được Uni Admin xếp vào doanh nghiệp

- **When**: Xem chi tiết kỳ

- **Then**:

  - Badge trạng thái hiển thị **"Đã được xếp doanh nghiệp"** (màu xanh)

  - Trường Tên Enterprise hiển thị tên doanh nghiệp tương ứng

---

**AC-06: Notification khi được Placed**

- **Given**: Student đang ở trạng thái Unplaced

- **When**: Uni Admin cập nhật trạng thái sang Placed

- **Then**:

  - Hệ thống gửi email/notification đến Student với nội dung: tên kỳ thực tập, tên Enterprise được xếp

  - Notification hiển thị trong app (notification bell)

  - Student truy cập lại trang sẽ thấy trạng thái đã cập nhật

---

**AC-07: Notification khi bị rút khỏi kỳ**

- **Given**: Student đang enrolled trong kỳ

- **When**: Uni Admin rút Student khỏi kỳ

- **Then**:

  - Hệ thống gửi email/notification đến Student: tên kỳ, ngày rút, lý do (nếu có)

  - Trạng thái enrollment của kỳ đó cập nhật thành **Withdrawn**

  - Kỳ đó vẫn hiển thị trong lịch sử với badge **"Đã rút"**

---

**AC-08: Quyền truy cập**

- **Given**: Có các tài khoản với role khác nhau

- **When**: Truy cập trang "Kỳ thực tập của tôi"

- **Then**:

  - **Student**: chỉ xem được thông tin kỳ thực tập của chính mình

  - **Student khác / Uni Admin / Enterprise HR**: không thể truy cập dữ liệu của Student này, nhận lỗi 403 Forbidden

---

Tổng **8 AC**, cover đầy đủ happy path, empty state, notification và phân quyền. Confirm nhé!