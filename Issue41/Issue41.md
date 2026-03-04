# 1. Tổng quan & Bối cảnh (Overview & Context)

## 1.1 Vấn đề (Problem)
Đánh giá năng lực (Evaluation) là kết quả quan trọng nhất của kỳ thực tập. Hiện tại, việc thiếu thông tin minh bạch về các đợt đánh giá (Giữa kỳ, Cuối kỳ) và tiêu chí chấm điểm khiến sinh viên bị động, không kịp thời nhận diện điểm yếu để cải thiện trước khi kết thúc kỳ thực tập.

## 1.2 Giá trị Nghiệp vụ (Business Value)
- **Minh bạch hóa kết quả**: Giúp sinh viên nắm bắt lộ trình đánh giá và kết quả hiện tại một cách rõ ràng.
- **Theo dõi tiến độ**: Nhận biết chu kỳ nào đang diễn ra, chu kỳ nào đã kết thúc để chuẩn bị báo cáo phù hợp.
- **Phản hồi kịp thời**: Điểm số chi tiết và nhận xét từ Mentor giúp sinh viên nhận ra điểm mạnh/yếu để khắc phục.

## 1.3 Đối tượng (Actor)
- **Primary Actor**: Sinh viên (Student) — người xem kết quả đánh giá của nhóm và của cá nhân.
- **Secondary Actors**: Mentor/Enterprise — người thực hiện chấm điểm và tạo dữ liệu (Overview data).

---

# 2. Luồng Người dùng (User Flow)

## 2.1 Luồng Xem Danh sách Chu kỳ
1. Sinh viên truy cập vào menu "Đánh giá" (Evaluations).
2. Hệ thống hiển thị danh sách các chu kỳ đánh giá (Evaluation Cycles) áp dụng cho dự án/nhóm của sinh viên.
3. Thông tin hiển thị: Tên chu kỳ, Thời gian (Bắt đầu - Kết thúc), Trạng thái (Sắp/Đang/Đã kết thúc) và tiến độ chấm điểm của Mentor.

## 2.2 Luồng Xem Đánh giá Chung của Nhóm
1. Sinh viên click chọn một chu kỳ cụ thể.
2. Hệ thống hiển thị danh sách tất cả thành viên trong nhóm thực tập.
3. Thông tin hiển thị cho từng thành viên: Họ tên, MSSV, Trạng thái chấm (Đã chấm/Chưa chấm), Ngày chấm gần nhất.
4. Điểm tổng chỉ hiển thị cho bản thân, điểm của người khác hiển thị "--" hoặc bị ẩn đi.

## 2.3 Luồng Xem Chi tiết Phiếu điểm Cá nhân
1. Sinh viên click vào mũi tên xem chi tiết tại dòng tên của chính mình.
2. Hệ thống hiển thị phiếu điểm chi tiết gồm: Các tiêu chí (VD: Kỷ luật, Chuyên môn...), Điểm số từng phần, và Nhận xét chung (Feedback) từ Mentor.
3. Nếu sinh viên cố gắng truy cập chi tiết điểm của người khác, hệ thống sẽ chặn truy cập (Bảo mật).

---

# 3. Tiêu chí Chấp nhận (Acceptance Criteria)

## AC-01: Hiển thị Danh sách Chu kỳ
- **Given**: Sinh viên đã đăng nhập và thuộc một nhóm thực tập, truy cập trang "Đánh giá".
- **When**: Trang tải dữ liệu thành công.
- **Then**:
  - Hệ thống hiển thị danh sách các đợt đánh giá thuộc nhóm/dự án của sinh viên.
  - Trạng thái được BE tự động tính toán dựa trên ngày tháng (Upcoming, Active, Ended) để hiển thị.
  - Hiển thị tiến độ chấm điểm chung của nhóm (Ví dụ: "Đã chấm 3/5 sinh viên").

## AC-02: Xem Danh sách Nhóm thuộc Chu kỳ
- **Given**: Sinh viên đang ở danh sách chu kỳ.
- **When**: Click vào một chu kỳ (VD: "Đánh giá Giữa kỳ").
- **Then**:
  - Chuyển hướng đến bảng danh sách các thành viên trong nhóm dự án cho chu kỳ đó.
  - Cột "Điểm tổng" của các sinh viên khác bị ẩn hoặc hiển thị `--` trừ điểm của bản thân mình (nếu đã được chấm).

## AC-03: Xem Phiếu điểm Chi tiết (My Score)
- **Given**: Sinh viên ở trang danh sách thuộc chu kỳ.
- **When**: Click vào tên của CHÍNH MÌNH (chức năng "Xem chi tiết").
- **Then**:
  - Hệ thống mở phiếu điểm hoàn chỉnh.
  - Liệt kê rõ: Điểm thành phần theo từng tiêu chí con, Điểm tổng kết và Nhận xét của Mentor.

## AC-04: Đảm bảo Quyền Riêng tư (Privacy Guard)
- **Given**: Sinh viên A đang xem danh sách trạng thái chấm điểm của nhóm.
- **When**: Sinh viên A cố gắng xem chi tiết điểm của Sinh viên B (click UI hoặc gọi trực tiếp API Bypass).
- **Then**:
  - UI không hiển thị nút "Xem điểm" đối với các dòng không phải user đăng nhập.
  - API backend trả lỗi mạnh `403 Forbidden` với message: *"Bạn không có quyền xem chi tiết điểm của người khác."*

---

# 4. Đặc tả Kỹ thuật (Technical Specifications)

Dựa theo chuẩn **FFA Framework (IOCv2 Agent Skills)**:

## 4.1 Danh sách API Endpoints Bắt buộc

| API Endpoint | Method | Path | Request Variables | Response Structure (Thành công 200 OK) | Mã lỗi dự kiến |
| --- | --- | --- | --- | --- | --- |
| Lấy danh sách chu kỳ đánh giá của SV | **GET** | `/api/evaluations-cycles` | *Query*: `projectId` | `[ { cycleId, name, startDate, endDate, status, gradedCount, totalCount } ]` | **401 Unauthorized**<br>**429 Too Many Requests** |
| Xem danh sách sinh viên đánh giá trong chu kỳ | **GET** | `/api/evaluations-cycles/{cycleId}` | Không | `{ cycleInfo: {...}, students: [ { studentId, fullName, isGraded, gradedAt, totalScore } ] }` | **400 Bad Request** (Lỗi Validator)<br>**404 Not Found** (Không tìm thấy chu kỳ) |
| Lấy chi tiết phiếu điểm cá nhân | **GET** | `/api/evaluations-cycles/{cycleId}/students/{studentId}` | Không | `{ studentId, cycleId, totalScore, feedback, criteriaScores: [ { criteriaName, score, maxScore, comments } ] }` | **403 Forbidden** (Truy cập điểm người khác)<br>**404 Not Found** (Chưa có điểm) |

*[⚠️ Architecture Violation]: Đảm bảo Entity `EvaluationCycle` có lưu trữ mối quan hệ với `Project`/`Term`, BE cần logic trích xuất `StudentId` từ `JWT Token` để lọc ra đúng dữ liệu danh sách thuộc về sinh viên đó, tránh việc list ra chu kỳ của người khác.*

## 4.2 [FFA-ACV]  [FFA-ERR] Validation  Response Data
- **Request Validation**:
  - Tham số `cycleId` và `studentId` trên URL Route bắt buộc là UUID hợp lệ. Nếu sai định dạng lập tức văng Exception -> Xử lý thành **400 Bad Request**.
- **Data Protection ở Endpoint List Sinh Viên (`cycles/{cycleId}`)**:
  - Trường `totalScore` trong mảng `students` nếu bản ghi không thuộc về `CurrentUser.Id` thì Backend **BẮT BUỘC** gán giá trị thành `null` trước khi send Response cục bộ (ngăn lộ qua tools Network Dumper).

## 4.3 [FFA-SEC] Authentication  Authorization
- Ràng buộc Authentication (bắt buộc Login): Thêm Header `[Authorize(Roles = "Student")]` trên controller.
- **Authorization Cross-Check (Tầng Handler)**:
  - Ở Query `GetEvaluationDetailQueryHandler` lấy Phiếu điểm cá nhân, thực hiện logic: `if (currentUser.Id != request.StudentId) throw new ForbiddenAccessException("Bạn không có quyền xem chi tiết điểm của người khác");`
  - Global Exception middleware của FFA-ERR sẽ tự chuyển ngoại lệ đó thành mã `403 Forbidden`.

## 4.4 [FFA-PERF] Cấu hình Rate Limit
- **Read APIs Rate Limit**: Sử dụng IP-based Rule Rate Limit để ngăn bot/script tự động crawl điểm của sinh viên liên tục.
  - Config: **60 requests / 1 phút / IP** áp dụng cho toàn bộ scope `EvaluationController`. Vi phạm trả ra lỗi `429 Too Many Requests`.