## 1. Tổng quan & Bối cảnh (Overview & Context)

## Vấn đề (Problem):

Đánh giá năng lực là kết quả quan trọng nhất của kỳ thực tập. Sinh viên thường lo lắng không biết mình đang được đánh giá ở giai đoạn nào, điểm số ra sao và tiêu chí là gì. Việc thiếu thông tin minh bạch về các đợt đánh giá (Giữa kỳ, Cuối kỳ) khiến sinh viên bị động, không kịp cải thiện kết quả trước khi kỳ thực tập kết thúc.

## Giá trị Nghiệp vụ (Business Value):

- **Minh bạch hóa kết quả:** Giúp sinh viên nắm bắt được lộ trình đánh giá và kết quả hiện tại của mình.

- **Theo dõi tiến độ:** Biết được chu kỳ nào đang diễn ra, chu kỳ nào đã kết thúc để chuẩn bị tâm thế/báo cáo phù hợp.

- **Phản hồi kịp thời:** Nhìn thấy điểm số chi tiết giúp sinh viên nhận ra điểm mạnh/yếu để khắc phục.

## Đối tượng (Actor):

- **Primary Actor:** Sinh viên (Student) — người xem kết quả.

- **Secondary Actors:** Mentor/Enterprise — người thực hiện chấm điểm (tạo dữ liệu cho view này).

---

## 2. Luồng Người dùng (User Flow)

## 2.1. Luồng Xem Danh sách Chu kỳ (Overview Level)

1. Student truy cập menu **"Đánh giá"**.

2. Hệ thống hiển thị **Bảng thông tin chung các chu kỳ** (Evaluation Cycles).

3. Bảng gồm các cột: **Tên chu kỳ** (Giữa kỳ, Cuối kỳ...), **Thời gian** (Bắt đầu - Kết thúc), **Trạng thái** (Badge: Sắp diễn ra/Đang diễn ra/Kết thúc), **Tiến độ chấm** (Số SV đã chấm/Tổng SV).

4. Student click vào tên một chu kỳ để xem chi tiết.

## 2.2. Luồng Xem Chi tiết Chu kỳ & Danh sách Sinh viên (Cycle Detail Level)

1. Sau khi click chọn chu kỳ, hệ thống chuyển vào màn hình chi tiết.

2. **Phần trên:** Hiển thị lại thông tin chung của chu kỳ đó.

3. **Phần dưới:** Hiển thị **Danh sách sinh viên trong nhóm** được chấm trong chu kỳ này.

   - Cột: Họ tên, MSSV, **Điểm tổng** (nếu đã công bố), **Trạng thái chấm** (Đã chấm/Chưa chấm), **Ngày chấm gần nhất**.

4. Student click vào dòng tên của chính mình (hoặc nút "Xem chi tiết") để xem bảng điểm cụ thể.

   - *Lưu ý:* Thường sinh viên chỉ được xem điểm chi tiết của bản thân (quyền riêng tư), nhưng xem danh sách trạng thái (ai đã được chấm) của cả nhóm thì được.

## 2.3. Luồng Xem Chi tiết Điểm (Score Breakdown Level)

1. Hệ thống hiển thị phiếu điểm chi tiết của Student trong chu kỳ đó.

2. Nội dung gồm các tiêu chí đánh giá (ví dụ: Kỷ luật, Chuyên môn, Teamwork...) và điểm số/nhận xét tương ứng cho từng mục.

3. Hiển thị Điểm tổng kết và Nhận xét chung của Mentor.

---

## 3. Tiêu chí Chấp nhận (Acceptance Criteria)

## AC-EVAL-01 — Hiển thị Danh sách Chu kỳ

- **Given:** Student truy cập trang Đánh giá.

- **When:** Trang tải xong.

- **Then:**

  - Hiển thị danh sách các đợt đánh giá của dự án.

  - Cột Status hiển thị đúng màu sắc (Sắp diễn ra: Xám/Vàng, Đang diễn ra: Xanh dương, Kết thúc: Xanh lá/Đỏ).

  - Hiển thị đúng số lượng sinh viên đã được chấm (ví dụ: "3/5").

## AC-EVAL-02 — Xem Chi tiết Chu kỳ (Drill-down)

- **Given:** Student click vào "Đánh giá Giữa kỳ".

- **When:** Vào trang chi tiết.

- **Then:**

  - Hiển thị danh sách sinh viên thuộc nhóm.

  - Cột Điểm tổng chỉ hiển thị giá trị số nếu Trạng thái là "Đã chấm" (và có thể thêm điều kiện "Đã công bố"). Nếu chưa chấm, hiển thị "--" hoặc "Chưa có".

## AC-EVAL-03 — Xem Phiếu điểm Chi tiết (My Score)

- **Given:** Student muốn xem mình bị trừ điểm ở đâu.

- **When:** Click vào tên mình trong danh sách (hoặc nút "Xem điểm").

- **Then:**

  - Hiển thị Modal hoặc trang con chứa bảng tiêu chí.

  - Liệt kê rõ: Tiêu chí A (8/10), Tiêu chí B (7/10)...

  - Hiển thị lời nhận xét (Feedback) của Mentor nếu có.

## AC-EVAL-04 — Quyền Riêng tư (Privacy)

- **Given:** Student A đang xem danh sách sinh viên của chu kỳ.

- **When:** Thử click vào tên của Student B để xem chi tiết điểm.

- **Then:**

  - Hệ thống **KHÔNG** cho phép (nút bị disable hoặc click vào báo "Bạn không có quyền xem chi tiết điểm người khác").

  - Student chỉ được xem chi tiết điểm của chính mình (My View).

---

## 4. Đặc tả kỹ thuật (Technical Notes)

- **Privacy Logic:** API lấy chi tiết điểm `GET /api/evaluations/{cycleId}/students/{studentId}` cần check quyền: Nếu `current_user.id != studentId` (và không phải Mentor/Admin) thì trả về 403 Forbidden.

- **Status Calculation:** Trạng thái chu kỳ (Sắp/Đang/Xong) có thể tính toán dựa trên `Current Date` so với `StartDate` và `EndDate` của chu kỳ, hoặc dựa trên một trường status cứng trong DB nếu Admin set thủ công.