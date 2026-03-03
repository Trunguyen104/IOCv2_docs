## 1. Tổng quan & Bối cảnh (Overview & Context)

## Vấn đề (Problem):

Trong mô hình thực tập doanh nghiệp, việc báo cáo tiến độ hàng ngày (Daily Standup/Report) là bắt buộc để Mentor và Nhà trường nắm bắt được sinh viên đang làm gì, gặp khó khăn gì. Tuy nhiên, việc báo cáo qua email hay chat thường trôi tin, khó thống kê và khó kiểm soát tính chuyên cần (nộp đúng hạn/muộn).

## Giá trị Nghiệp vụ (Business Value):

- **Kỷ luật & Chuyên nghiệp:** Rèn luyện thói quen báo cáo công việc định kỳ, đúng hạn cho sinh viên.

- **Theo dõi sát sao:** Giúp Mentor thấy rõ khối lượng công việc hàng ngày và phát hiện sớm nếu sinh viên bị tắc nghẽn (block).

- **Dữ liệu minh chứng:** Là cơ sở để đánh giá điểm thái độ và quá trình thực tập cuối kỳ.

## Đối tượng (Actor):

- **Primary Actor:** Sinh viên (Student) — người tạo và nộp báo cáo.

- **Secondary Actors:** Mentor, HR — xem và duyệt báo cáo (nếu có quy trình duyệt).

---

## 2. Luồng Người dùng (User Flow)

## 2.1. Luồng Xem Danh sách Báo cáo

1. Student truy cập menu **"Báo cáo hàng ngày"**.

2. Hệ thống hiển thị danh sách các báo cáo đã nộp.

3. Bảng hiển thị các cột: Ngày báo cáo, Tên sinh viên (nếu xem nhóm), Trạng thái duyệt (Status), Tóm tắt công việc, Issue đã làm, Thời gian nộp (Time), Trạng thái nộp (Đúng hạn/Muộn).

4. Cuối bảng có thanh phân trang và tổng số bản ghi.

## 2.2. Luồng Tạo Báo cáo (Create Daily Report)

1. Student nhấn nút **"Tạo báo cáo"**.

2. Hệ thống hiển thị Form báo cáo cho ngày hiện tại.

   - *Logic chặn:* Nếu hôm nay là T7, CN hoặc Ngày lễ (cấu hình hệ thống), có thể hiển thị cảnh báo hoặc không bắt buộc nộp (tùy policy), nhưng vẫn cho phép tạo nếu sinh viên làm thêm giờ.

3. Student nhập:

   - **Tóm tắt công việc:** Mô tả ngắn gọn những việc đã làm.

   - **Issues đã làm:** Chọn/Link các task từ danh sách công việc (Jira-like linking) hoặc nhập text.

   - **Vấn đề gặp phải (Blockers):** (Optional).

   - **Kế hoạch ngày mai:** (Optional).

4. Nhấn **"Nộp báo cáo"**.

5. Hệ thống ghi nhận thời gian nộp và tự động đánh dấu "Đúng hạn" hay "Muộn" dựa trên giờ quy định (ví dụ: trước 18:00).

## 2.3. Luồng Chỉnh sửa (Update)

1. Student click vào báo cáo của ngày hôm nay (nếu chưa bị Mentor khóa/duyệt).

2. Chỉnh sửa nội dung.

3. Lưu lại (Hệ thống có thể ghi chú "Đã chỉnh sửa").

## 2.4. Luồng Tìm kiếm & Lọc

1. **Tìm kiếm:** Nhập từ khóa (tìm trong Tóm tắt công việc).

2. **Lọc:**

   - Theo **Trạng thái duyệt** (Đã duyệt/Chờ duyệt).

   - Theo **Trạng thái nộp** (Đúng hạn/Muộn).

   - Theo **Khoảng thời gian** (Từ ngày - Đến ngày).

---

## 3. Tiêu chí Chấp nhận (Acceptance Criteria)

## AC-RPT-01 — Hiển thị Danh sách Báo cáo chuẩn

- **Given:** Student truy cập trang Báo cáo.

- **When:** Trang tải xong.

- **Then:**

  - Hiển thị đúng các cột: Ngày báo cáo (dd/MM/yyyy), Tên sinh viên, Status (Duyệt/Chưa), Tóm tắt, Issue liên quan, Thời gian nộp (HH:mm), Status nộp (Đúng hạn/Muộn).

  - Mặc định sắp xếp theo ngày giảm dần (mới nhất lên đầu).

  - Hiển thị Tổng số bản ghi (Total records).

## AC-RPT-02 — Logic Phân trang

- **Given:** Danh sách báo cáo dài &gt; 1 trang.

- **When:** Student chọn hiển thị 50 bản ghi/trang hoặc chuyển trang 2.

- **Then:**

  - Bảng hiển thị đúng số lượng yêu cầu.

  - Dữ liệu tải lại chính xác.

## AC-RPT-03 — Tạo Báo cáo & Đánh dấu Muộn (Late Submission)

- **Given:** Quy định giờ nộp là 09:00 AM sáng hôm sau (hoặc 18:00 PM cùng ngày - tùy config).

- **When:**

  - Student A nộp lúc 08:00 AM → Hệ thống ghi nhận "Đúng hạn" (On Time).

  - Student B nộp lúc 10:00 AM → Hệ thống ghi nhận "Muộn" (Late) và highlight màu đỏ/cam.

- **Then:**

  - Bản ghi được lưu thành công.

  - Trạng thái nộp (Submission Status) được tính toán tự động, không cho sửa thủ công.

## AC-RPT-04 — Link Issue (Liên kết công việc)

- **Given:** Student đang làm việc trên các task ID-101, ID-102.

- **When:** Tạo báo cáo.

- **Then:**

  - Có trường chọn "Issue đã làm" cho phép search và chọn các task đang có trong dự án.

  - Khi xem danh sách báo cáo, cột "Issue đã làm" hiển thị các mã task (click vào sẽ mở chi tiết task đó).

## AC-RPT-05 — Logic Ngày nghỉ (Weekend/Holiday)

- **Given:** Hôm nay là Thứ 7 hoặc Chủ Nhật.

- **When:** Student vào trang báo cáo.

- **Then:**

  - Hệ thống không bắt buộc (không hiện cảnh báo đỏ "Chưa nộp") cho ngày này.

  - Vẫn cho phép nút "Tạo báo cáo" hoạt động (trường hợp làm bù hoặc OT).

  - Các báo cáo tạo vào ngày nghỉ không bị tính là "Muộn" dù nộp giờ nào (Tùy rule dự án, thường là vậy).

## AC-RPT-06 — Tìm kiếm & Filter

- **Given:** Student muốn xem lại các báo cáo bị muộn tháng trước.

- **When:**

  - Filter Status nộp = "Muộn".

  - Filter Date Range = "Tháng trước".

- **Then:**

  - Danh sách chỉ hiển thị các báo cáo thỏa mãn.

---

## 4. Đặc tả kỹ thuật (Technical Notes)

- **Holiday Calendar:** Cần có một bảng cấu hình hoặc API trả về danh sách ngày lễ (Holiday) để Frontend/Backend check logic "Ngày làm việc".

- **Auto-Status Logic:** Logic check "Đúng hạn/Muộn" nên thực hiện ở Backend dựa trên `server_time` khi nhận request để đảm bảo công bằng (tránh trường hợp Student chỉnh giờ máy client).

- **Submission Deadline:** Cần cấu hình được Deadline (ví dụ: `18:00:00`) trong DB cấu hình dự án.