# 1. Tổng quan & Bối cảnh (Overview & Context)

## 1.1 Vấn đề (Problem)
Đánh giá quá trình và điểm số (Evaluation) là kết quả phản ánh trực tiếp chất lượng thực tập của sinh viên tại doanh nghiệp. Hiện tại, sinh viên thiếu bối cảnh và công cụ để theo dõi xem có những đợt đánh giá nào (Giữa kỳ, Cuối kỳ) ứng với kỳ thực tập của mình, trạng thái đánh giá (đã chấm hay chưa) của mình và đồng đội trong nhóm ra sao. Sự thiếu hụt này làm sinh viên mất chủ động trong việc đón nhận Feedback từ Doanh nghiệp (Mentor).

## 1.2 Giá trị Nghiệp vụ (Business Value)
- **Minh bạch hóa lộ trình**: Sinh viên nắm bắt được lịch đánh giá của Trường/Doanh nghiệp để chuẩn bị tâm lý và báo cáo công việc kịp thời.
- **Tiếp thu phản hồi (Feedback)**: Cho phép sinh viên xem chi tiết điểm số từng tiêu chí và nhận xét từ Mentor để khắc phục điểm yếu.
- **Tính riêng tư**: Đảm bảo quyền bảo mật điểm số cá nhân, sinh viên chỉ thấy điểm tổng quát của trạng thái đánh giá, không nhìn trộm được điểm của bạn bè.

## 1.3 Đối tượng (Actor)
- **Primary Actor**: Sinh viên (Student) — Người vào hệ thống để tra cứu đợt đánh giá và xem kết quả của chính mình.
- **Secondary Actor**: Không có (Mentor/HR là người chấm điểm nhưng thao tác ở luồng issue khác).

---

# 2. Luồng Người dùng (User Flow)

## 2.1 Luồng Khám phá Chu kỳ Đánh giá
1. Sinh viên đăng nhập, truy cập menu **"Đánh giá & Điểm số"** (Evaluations).
2. Hệ thống gọi API liệt kê tất cả các Chu kỳ đánh giá (`Evaluation Cycles`) áp dụng cho kỳ thực tập hiện tại của Sinh viên.
3. Giao diện hiển thị danh sách các Chu kỳ (VD: Đánh giá giữa kỳ, Đánh giá cuối kỳ) kèm thời gian diễn ra và số lượng sinh viên trong nhóm đã được chấm điểm.

## 2.2 Luồng Xem Tổng quan Đánh giá Nhóm
1. Sinh viên click chọn một Chu kỳ cụ thể để đi vào chi tiết.
2. Hệ thống gọi API lấy danh sách toàn bộ các thành viên thuộc nhóm thực tập (`internship_group`) của sinh viên này.
3. Màn hình liệt kê Grid: Họ tên thành viên, MSSV, Trạng thái (Chưa chấm / Đã chấm).
4. **Quy tắc hiển thị điểm**: Nếu là bản ghi của **Chính Sinh Viên Đang Xem**, hiển thị Tống Điểm. Tất cả các sinh viên khác hiện `***` hoặc `--` để bảo mật.

## 2.3 Luồng Xem Chi tiết Phiếu điểm Cá nhân
1. Tại dòng của chính mình (chỉ dòng này mới có nút hành động), sinh viên bấm **"Xem chi tiết Phiếu điểm"**.
2. Hệ thống fetch API lấy phiếu mẫu chi tiết.
3. Màn hình trượt (Slider) hiển thị:
   - Thông tin Mentor đánh giá và Ngày đánh giá.
   - Điểm tổng & Nhận xét chung (General Comment).
   - Danh sách điểm thành phần (Criteria Scores) liệt kê từng tiêu chí (Criteria), số điểm đạt được / điểm tối đa và nhận xét chi tiết cho tiêu chí đó.

---

# 3. Tiêu chí Chấp nhận (Acceptance Criteria)

## AC-01: Hiển thị danh sách Chu kỳ
- **Given**: Sinh viên đăng nhập và đang tiến hành thực tập theo nhóm.
- **When**: Truy cập trang "Đánh giá".
- **Then**:
  - Liệt kê các `Evaluation Cycles` có liên đới đến `term_id` của nhóm thực tập này.
  - Mỗi thẻ Chu kỳ ghi rõ: Tên chu kỳ, Timeline Bắt đầu - Kết thúc, và Trạng thái của chu kỳ theo logic ngày tháng (Upcoming / Ongoing / Completed).

## AC-02: Xem Danh sách Thành viên Nhóm
- **Given**: Sinh viên đang ở trang Danh sách chu kỳ.
- **When**: Click vào một Chu kỳ đang Ongoing / Completed.
- **Then**:
  - Giao diện liệt kê tất cả sinh viên thuộc nhóm thực tập của mình.
  - Cột trạng thái hiển thị: Pending, Draft (Chưa public), Submitted, Published.
  - **Che dấu điểm số**: Cột điểm số hiển thị `***` cho mọi người khác, chỉ duy nhất chính mình nhìn thấy điểm số thật của mình nếu phiếu điểm mang trạng thái `Published`.

## AC-03: Xem chi tiết Phiếu điểm (My Score)
- **Given**: Phiếu điểm của sinh viên mang trạng thái `Published` (Đã công bố).
- **When**: Sinh viên bấm nút Xem chi tiết điểm của mình.
- **Then**:
  - Giao diện trải ra Bảng điểm chi tiết.
  - Bao gồm: General feedback, và bảng Detail cho từng Criteria (Mỗi dòng có Tên tiêu chí, Điểm nhận/Điểm tối đa, Comment riêng rẽ).

## AC-04: Bảo vệ an toàn dữ liệu (Privacy Guard)
- **Given**: Sinh viên A biết UUID phiếu điểm của bạn B trong nhóm hoặc ngoài trường.
- **When**: Sinh viên A giả lập Request lấy chi tiết điểm của bạn B bằng URL path.
- **Then**: 
  - Hệ thống BE tự động lọc quyền Ownership. Trả về mã lỗi HTTP `403 Forbidden` do truy cập trái phép. Không tiết lộ bất cứ mảng data nào.

---

# 4. Đặc tả Kỹ thuật (Technical Specifications)

Dựa theo chuẩn **FFA Framework (IOCv2 Agent Skills)** và ERD **DB.md**:

## 4.1 Danh sách API Endpoints

| API Endpoint | Method | Path | Request Variables | Response Structure (JSON) | Status Codes & Messages |
| --- | --- | --- | --- | --- | --- |
| Lấy các Chu kỳ đánh giá | **GET** | `/api/students/me/evaluation-cycles` | Query: `internshipId` (Bắt buộc nếu tham gia nhiều nhóm) | `{ data: [ { cycleId, name, startDate, endDate, status } ] }` | **200 OK**<br>**404 Not Found**<br>**403 Forbidden** |
| Xem overview nhóm thuộc Chu kỳ | **GET** | `/api/students/me/evaluation-cycles/{cycleId}/team-evaluations` | Path: `cycleId` | `{ data: [ { studentId, fullName, studentCode, evaluationStatus, totalScore (Null nếu khác mình) } ] }` | **200 OK**<br>**403 Forbidden** |
| Chi tiết phiếu điểm cá nhân | **GET** | `/api/students/me/evaluation-cycles/{cycleId}/my-evaluation` | Path: `cycleId` | `{ evaluationId, cycleName, evaluatorName, gradedAt, totalScore, generalComment, criteriaScores: [ { criteriaName, score, maxScore, comment } ] }` | **200 OK**<br>**404 Not Found**<br>**403 Forbidden** |

## 4.2 [DB Alignment] Đối chiếu Database
- **Thiết kế API GET My-Evaluation**:
  - Bảng chính: `evaluations` kết hợp `evaluation_details`.
  - Mapping: `evaluationId` $\to$ `evaluations.evaluation_id`. `evaluatorName` $\to$ join qua `enterprise_users.user_id` $\to$ `users.full_name`. `criteriaScores` lấy từ `evaluation_details` join với `evaluation_criteria` để bóc `max_score` và `name`.
- **Trạng thái Điểm (evaluations.status)**: Bảng db.md quy định `0=Pending,1=Draft,2=Submitted,3=Published`. Backend chỉ trả dữ liệu mảng Detail Criteria và `totalScore` nếu Status `== 3` (Published). Nếu `<= 2`, tổng điểm và chi tiết trả `Null`.
- **Logic Quan hệ**: Không có **[⚠️ DB Mismatch]**. Lược đồ Database dư sức đáp ứng quan hệ `internship_groups` -> `terms` -> `evaluation_cycles` cho truy xuất cấp nhóm.

## 4.3 [FFA-ACV] Quy tắc Validation Data Response
- Ở API `/team-evaluations` (Danh sách nhóm), Backend có nghĩa vụ phải can thiệp chặn dữ liệu tại RAM (DTO Mapping) trước khi Serialize thành JSON:
  - Vòng lặp: Nếu `row.studentId != Token.StudentId`, thiết lập `totalScore = null`. Tránh tuyệt đối để điểm người khác lọt xuống Chrome DevTools (Network Payload).

## 4.4 [FFA-SEC] Authentication & Authorization
- **Ràng buộc Quyền hạn (RBAC)**: Toàn bộ Controller/Endpoint bị khóa bởi Attribute `[Authorize(Roles = "Student")]`.
- **Ownership Identity (Chống IDOR ngầm)**:
  - Ở mọi hành động GET, Backend lọc tự động bằng việc lấy `Token.UserId -> StudentId`.
  - Tại API `my-evaluation`: Câu lệnh query Entity Framework BẮT BUỘC có `.Where(e => e.student_id == currentStudentId && e.cycle_id == requestedCycleId)`. Nếu Count = 0, ném `404 Not Found` (Vì bản chất là chưa có điểm, hoặc gọi sai cycle của term khác).

## 4.5 [FFA-PERF] Cấu hình Rate Limit & Tối ưu 
- **Read APIs Rate Limit (GET)**: Giao diện dành cho sinh viên, lượng sinh viên dồn vào đọc điểm lúc cuối kỳ rất cao, thiết lập IP-based Limit: `60 requests / 1 phút / IP`.
- **EF Core Database Plan**: 
  - Toàn bộ query SELECT phải đính kèm `.AsNoTracking()`. Đặc biệt ở API List Team-Evaluations (tránh lưu cache Object hàng nghìn sinh viên vào Memory Server).
  - Tích hợp Lazy Loading / Split Queries cẩn thận khi móc nối `evaluations` $\to$ `evaluation_details` $\to$ `evaluation_criteria` ở màn hình xem phiếu điểm cá nhân. Đề xuất `.Include(e => e.Details).ThenInclude(d => d.Criteria).AsSplitQuery()`.