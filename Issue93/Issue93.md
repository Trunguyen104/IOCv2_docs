# 1. Tổng quan & Bối cảnh (Overview & Context)

## 1.1 Vấn đề (Problem)

Mentor tại Doanh nghiệp cần một công cụ tập trung để quản lý toàn bộ vòng đời đánh giá sinh viên thực tập: từ việc thiết kế các đợt đánh giá (Giữa kỳ, Cuối kỳ), thiết lập bộ tiêu chí chấm điểm, đến thực hiện chấm điểm và công bố kết quả. Hiện tại, nếu không có công cụ này, Mentor phải quản lý điểm số thủ công bên ngoài hệ thống, dẫn đến dữ liệu rời rạc, dễ sai sót, và không đồng bộ được với hệ thống báo cáo mà Sinh viên tra cứu (Issue 41).

Hệ thống cần hỗ trợ 2 hình thức chấm điểm song song:
- **Chấm nhanh hàng loạt (Batch)**: Giao diện Grid tương tự Excel, cho phép Mentor nhập điểm cho nhiều sinh viên cùng lúc, phù hợp khi chấm các tiêu chí đơn giản hoặc số lượng lớn.
- **Chấm chi tiết cá nhân (Individual)**: Form riêng lẻ cho từng sinh viên, hỗ trợ nhập nhận xét (Feedback) chuyên sâu cho từng tiêu chí.

## 1.2 Giá trị Nghiệp vụ (Business Value)

- **Quản lý linh hoạt**: Mentor chủ động định nghĩa các Chu kỳ đánh giá (Evaluation Cycles) và bộ Tiêu chí (Criteria) phù hợp với đặc thù dự án của doanh nghiệp.
- **Tối ưu năng suất**: Giảm thiểu thời gian nhập liệu thông qua giao diện chấm điểm hàng loạt dạng Grid-view.
- **Phản hồi chất lượng**: Khuyến khích Mentor đưa ra nhận xét chi tiết thông qua luồng chấm điểm cá nhân, giúp sinh viên cải thiện.
- **Dữ liệu tập trung**: Chốt điểm và công bố (Publish) đồng loạt. Khi trạng thái chuyển sang `Published`, sinh viên tự động nhìn thấy kết quả (kết nối với Issue 41).

## 1.3 Đối tượng (Actor)

- **Primary Actor**: Mentor (Doanh nghiệp) — Người quản lý chu kỳ, thiết lập tiêu chí, thực hiện chấm điểm và công bố kết quả.
- **Secondary Actor**: Sinh viên (Student) — Người nhận kết quả sau khi Mentor công bố (xem tại Issue 41).

---

# 2. Luồng Người dùng (User Flow)

## 2.1 Luồng Quản lý Chu kỳ Đánh giá (Cycle Management)

1. Mentor đăng nhập, truy cập menu **"Quản lý Đánh giá"**.
2. Hệ thống gọi API lấy danh sách các Chu kỳ đánh giá (`evaluation_cycles`) thuộc `term_id` liên kết với nhóm thực tập (`internship_groups`) mà Mentor quản lý.
3. Mentor thực hiện CRUD Chu kỳ:
   - **Tạo mới**: Nhập Tên chu kỳ (`name`), Ngày bắt đầu (`start_date`), Ngày kết thúc (`end_date`).
   - **Sửa**: Chỉnh sửa thông tin Chu kỳ đang ở trạng thái `Upcoming` hoặc `Ongoing`.
   - **Xóa**: Soft-delete Chu kỳ (set `deleted_at`). Chỉ cho phép xóa khi chưa có dữ liệu đánh giá (`evaluations`) liên kết.

## 2.2 Luồng Quản lý Tiêu chí (Criteria Management)

1. Mentor chọn một Chu kỳ cụ thể, truy cập tab **"Tiêu chí đánh giá"**.
2. Hệ thống gọi API lấy danh sách tiêu chí (`evaluation_criteria`) thuộc `cycle_id` đó.
3. Mentor thực hiện CRUD Tiêu chí:
   - **Tạo mới**: Nhập Tên tiêu chí (`name`), Mô tả (`description`), Điểm tối đa (`max_score`), Trọng số (`weight`).
   - **Sửa**: Chỉnh sửa tiêu chí. Chỉ cho phép nếu chu kỳ chưa `Completed`.
   - **Xóa**: Soft-delete (set `deleted_at`). Chỉ khi chưa có `evaluation_details` tham chiếu.

## 2.3 Luồng Chấm điểm Hàng loạt (Batch Evaluation — Grid View)

1. Tại danh sách Chu kỳ, Mentor chọn **"Chấm điểm nhanh"** cho một Chu kỳ cụ thể.
2. Hệ thống chuyển sang giao diện Grid (tương tự Excel):
   - **Cột cố định**: Họ tên (`users.full_name`), MSSV (`users.user_code`).
   - **Cột động**: Mỗi cột tương ứng với một Tiêu chí (`evaluation_criteria.name`) của chu kỳ đó.
3. Mentor nhập điểm trực tiếp vào các ô trong Grid.
4. Nhấn **"Submit Tất cả"**:
   - Client gửi một JSON Array chứa danh sách điểm cho toàn bộ sinh viên.
   - Server bóc tách JSON, validate và thực hiện Transactional Upsert vào bảng `evaluations` (tạo/cập nhật phiếu điểm tổng) và `evaluation_details` (ghi điểm từng tiêu chí).

## 2.4 Luồng Chấm điểm Cá nhân (Individual Evaluation)

1. Mentor nhấn vào nút tùy chọn (ba chấm) bên cạnh tên sinh viên trong danh sách nhóm.
2. Chọn **"Chấm điểm chi tiết"**.
3. Hệ thống hiển thị Form chấm điểm riêng lẻ:
   - Nhập điểm cho từng tiêu chí (`score`).
   - Nhập nhận xét chung (`general_comment` vào `evaluations.general_comment`).
   - Nhập nhận xét cho từng tiêu chí (`comment` vào `evaluation_details.comment`).
4. Nhấn **"Lưu"**: Gửi request upsert cho duy nhất một sinh viên.

## 2.5 Luồng Công bố Điểm (Publish)

1. Sau khi chấm điểm hoàn tất (trạng thái `Draft` hoặc `Submitted`), Mentor chọn **"Công bố điểm"** cho toàn bộ Chu kỳ hoặc từng cá nhân.
2. Hệ thống chuyển `evaluations.status` sang `3` (Published).
3. Từ thời điểm này, Sinh viên mới có thể xem điểm (theo logic Issue 41: chỉ trả data khi `status == 3`).

---

# 3. Tiêu chí Chấp nhận (Acceptance Criteria)

## AC-01: CRUD Chu kỳ Đánh giá (Evaluation Cycles)

- **Given**: Mentor đã đăng nhập và đang quản lý ít nhất một nhóm thực tập (`internship_groups`).
- **When**: Mentor tạo mới, sửa, hoặc xóa một Chu kỳ đánh giá.
- **Then**:
  - Dữ liệu lưu/cập nhật vào bảng `evaluation_cycles` với `term_id` là Term của nhóm thực tập Mentor đang quản lý.
  - Trường `created_by` / `updated_by` ghi nhận `enterprise_user_id` của Mentor hiện tại.
  - Xóa sử dụng Soft-delete (set `deleted_at`). Hệ thống từ chối xóa nếu đã có `evaluations` nằm trong Chu kỳ đó, trả `409 Conflict`.

## AC-02: CRUD Tiêu chí Đánh giá (Evaluation Criteria)

- **Given**: Mentor đang ở trang chi tiết một Chu kỳ đánh giá.
- **When**: Mentor tạo mới, sửa, hoặc xóa Tiêu chí.
- **Then**:
  - Dữ liệu lưu/cập nhật vào bảng `evaluation_criteria` với FK `cycle_id` tương ứng.
  - Validate: `max_score` > 0, `weight` >= 0 (nếu có trọng số).
  - Xóa Soft-delete (set `deleted_at`). Từ chối nếu đã có `evaluation_details` tham chiếu `criteria_id` đó, trả `409 Conflict`.

## AC-03: Chấm điểm Hàng loạt (Batch Evaluation via Grid View)

- **Given**: Danh sách sinh viên thuộc nhóm thực tập của Mentor, kèm danh sách Tiêu chí của Chu kỳ đã chọn.
- **When**: Mentor nhập điểm vào Grid và nhấn **"Submit Tất cả"**.
- **Then**:
  - Hệ thống validate: Mỗi `score` nhập vào **không vượt quá** `evaluation_criteria.max_score` của tiêu chí tương ứng, `score >= 0`.
  - Server thực hiện **Transactional Upsert**: Nếu `evaluations` record chưa tồn tại (check unique constraint `cycle_id + internship_id + student_id`) → INSERT. Nếu đã tồn tại → UPDATE.
  - Tương tự cho `evaluation_details` (unique constraint `evaluation_id + criteria_id`).
  - Tính `total_score` = SUM(score * weight / SUM(weight)) hoặc đơn giản SUM(score) tùy cấu hình Chu kỳ.
  - Nếu **bất kỳ** bản ghi nào trong List bị lỗi validation (sai `student_id`, điểm vượt max) → toàn bộ batch **Rollback**, không lưu.
  - Trạng thái `evaluations.status` mặc định được set thành `1` (Draft) sau khi tạo mới.

## AC-04: Chấm điểm Cá nhân và Feedback Chuyên sâu (Individual Evaluation)

- **Given**: Mentor chọn chấm điểm riêng cho một sinh viên cụ thể trong một Chu kỳ.
- **When**: Nhập điểm từng tiêu chí, nhập `general_comment`, nhập `comment` cho từng tiêu chí và nhấn **"Lưu"**.
- **Then**:
  - Hệ thống Upsert bản ghi `evaluations` cho sinh viên đó (bao gồm `general_comment`, `total_score`, `graded_at`, `evaluator_id`).
  - Upsert từng bản ghi `evaluation_details` (bao gồm `score`, `comment`).
  - Trạng thái phiếu điểm chuyển sang `2` (Submitted) nếu Mentor chọn Submit; hoặc giữ `1` (Draft) nếu chỉ Save nháp.

## AC-05: Kiểm soát Trạng thái Công bố (Publish)

- **Given**: Điểm số đã được nhập (Trạng thái `Draft` hoặc `Submitted`).
- **When**: Mentor chọn **"Công bố điểm"** (Publish) cho một hoặc nhiều sinh viên trong Chu kỳ.
- **Then**:
  - Trạng thái `evaluations.status` chuyển sang `3` (Published).
  - Hệ thống chỉ cho phép Publish các phiếu có trạng thái `>= 1` (Draft trở lên). Phiếu `Pending` (chưa chấm) không thể Publish → trả `400 Bad Request`.
  - Sau khi Published, Sinh viên sẽ nhìn thấy điểm tại luồng Issue 41.
  - **Lưu ý Business Rule**: Sau khi Published, Mentor vẫn có thể chỉnh sửa điểm (re-grade). Trạng thái tự động quay về `Draft` khi có thay đổi, yêu cầu Publish lại.

## AC-06: Bảo mật Quyền sở hữu Dữ liệu (Data Ownership Guard)

- **Given**: Bất kỳ người dùng nào trên hệ thống.
- **When**: Cố gắng truy cập API đánh giá qua Postman hoặc đổi UUID trên URL.
- **Then**:
  - Chỉ User có Role `Mentor` (hoặc `EnterpriseHR` nếu mở rộng) mới được truy cập.
  - Backend kiểm tra **Ownership**: `internship_groups.mentor_id` phải khớp `Token.EnterpriseUserId`. Nếu Mentor A cố thao tác trên nhóm của Mentor B → trả `403 Forbidden`.
  - Sinh viên, UniAdmin hoặc Role khác gọi API → `403 Forbidden`.

---

# 4. Đặc tả Kỹ thuật (Technical Specifications)

Dựa theo chuẩn **FFA Framework (IOCv2 Agent Skills)** và ERD **DB.md**:

## 4.1 Danh sách API Endpoints

### 4.1.1 Quản lý Chu kỳ (Evaluation Cycles)

| API Endpoint | Method | Path | Request Body / Query | Response Structure (JSON) | Status Codes & Messages |
| --- | --- | --- | --- | --- | --- |
| Lấy danh sách Chu kỳ | **GET** | `/api/evaluation-cycles` | Query: `internshipId` (uuid, bắt buộc), `pageIndex`, `pageSize` | `{ data: [ { cycleId, name, startDate, endDate, status, criteriaCount } ], totalCount }` | **200 OK**<br>**400 Bad Request**: `"internshipId": "Trường này là bắt buộc"`<br>**401 Unauthorized**<br>**403 Forbidden** |
| Xem chi tiết Chu kỳ | **GET** | `/api/evaluation-cycles/{cycleId}` | Path: `cycleId` (uuid) | `{ cycleId, name, startDate, endDate, status, criteria: [ { criteriaId, name, description, maxScore, weight } ] }` | **200 OK**<br>**403 Forbidden**<br>**404 Not Found**: `"Không tìm thấy Chu kỳ đánh giá"` |
| Tạo Chu kỳ | **POST** | `/api/evaluation-cycles` | `{ internshipId: uuid, name: string, startDate: timestamptz, endDate: timestamptz }` | `{ cycleId: uuid }` | **201 Created**<br>**400 Bad Request** (validation)<br>**403 Forbidden** |
| Cập nhật Chu kỳ | **PUT** | `/api/evaluation-cycles/{cycleId}` | `{ name: string, startDate: timestamptz, endDate: timestamptz }` | `204 No Content` | **204 No Content**<br>**400 Bad Request**<br>**404 Not Found** |
| Xóa Chu kỳ (Soft-delete) | **DELETE** | `/api/evaluation-cycles/{cycleId}` | Không có | `204 No Content` | **204 No Content**<br>**403 Forbidden**<br>**404 Not Found**<br>**409 Conflict**: `"Không thể xóa chu kỳ đã có dữ liệu đánh giá"` |

### 4.1.2 Quản lý Tiêu chí (Evaluation Criteria)

| API Endpoint | Method | Path | Request Body / Query | Response Structure (JSON) | Status Codes & Messages |
| --- | --- | --- | --- | --- | --- |
| Lấy danh sách Tiêu chí | **GET** | `/api/evaluation-cycles/{cycleId}/criteria` | Path: `cycleId` (uuid) | `{ data: [ { criteriaId, name, description, maxScore, weight } ] }` | **200 OK**<br>**403 Forbidden**<br>**404 Not Found** |
| Tạo Tiêu chí | **POST** | `/api/evaluation-cycles/{cycleId}/criteria` | `{ name: string, description?: string, maxScore: decimal, weight?: decimal }` | `{ criteriaId: uuid }` | **201 Created**<br>**400 Bad Request**<br>**403 Forbidden** |
| Cập nhật Tiêu chí | **PUT** | `/api/evaluation-cycles/{cycleId}/criteria/{criteriaId}` | `{ name: string, description?: string, maxScore: decimal, weight?: decimal }` | `204 No Content` | **204 No Content**<br>**400 Bad Request**<br>**404 Not Found** |
| Xóa Tiêu chí (Soft-delete) | **DELETE** | `/api/evaluation-cycles/{cycleId}/criteria/{criteriaId}` | Không có | `204 No Content` | **204 No Content**<br>**404 Not Found**<br>**409 Conflict**: `"Không thể xóa tiêu chí đã có điểm số"` |

### 4.1.3 Chấm điểm (Evaluation Grading)

| API Endpoint | Method | Path | Request Body / Query | Response Structure (JSON) | Status Codes & Messages |
| --- | --- | --- | --- | --- | --- |
| Lấy Grid chấm điểm (dữ liệu hiện tại) | **GET** | `/api/evaluation-cycles/{cycleId}/evaluations` | Query: `internshipId` (uuid, bắt buộc) | `{ criteria: [ { criteriaId, name, maxScore } ], students: [ { studentId, fullName, studentCode, evaluationId?, status?, scores: [ { criteriaId, score, comment? } ], totalScore?, generalComment? } ] }` | **200 OK**<br>**403 Forbidden**<br>**404 Not Found** |
| Chấm điểm Hàng loạt (Batch Upsert) | **POST** | `/api/evaluation-cycles/{cycleId}/evaluations/batch` | `{ internshipId: uuid, evaluations: [ { studentId: uuid, scores: [ { criteriaId: uuid, score: decimal } ] } ] }` | `204 No Content` | **204 No Content**<br>**400 Bad Request**: `"scores[0].score": "Điểm không được vượt quá X"` hoặc `"studentId": "Sinh viên không thuộc nhóm thực tập"`<br>**403 Forbidden**<br>**429 Too Many Requests** |
| Chấm điểm Cá nhân (Individual Upsert) | **POST** | `/api/evaluation-cycles/{cycleId}/evaluations/individual` | `{ internshipId: uuid, studentId: uuid, generalComment?: string, scores: [ { criteriaId: uuid, score: decimal, comment?: string } ] }` | `{ evaluationId: uuid }` | **201 Created / 200 OK**<br>**400 Bad Request**<br>**403 Forbidden** |
| Công bố Điểm (Publish) | **PATCH** | `/api/evaluation-cycles/{cycleId}/evaluations/publish` | `{ internshipId: uuid, studentIds?: uuid[] }` (nếu null → publish toàn bộ) | `204 No Content` | **204 No Content**<br>**400 Bad Request**: `"Không thể công bố phiếu điểm chưa được chấm (Pending)"`<br>**403 Forbidden** |

## 4.2 [DB Alignment] Đối chiếu Database

### 4.2.1 Mapping Dữ liệu

| Trường Response/Request | DB Table | DB Column | DB Type | Ghi chú |
| --- | --- | --- | --- | --- |
| `cycleId` | `evaluation_cycles` | `cycle_id` | uuid | PK |
| `name` (Cycle) | `evaluation_cycles` | `name` | varchar(255) | |
| `startDate` (Cycle) | `evaluation_cycles` | `start_date` | timestamptz | |
| `endDate` (Cycle) | `evaluation_cycles` | `end_date` | timestamptz | |
| `status` (Cycle) | `evaluation_cycles` | `status` | smallint | 0=Upcoming, 1=Ongoing, 2=Completed |
| `criteriaId` | `evaluation_criteria` | `criteria_id` | uuid | PK |
| `name` (Criteria) | `evaluation_criteria` | `name` | varchar(255) | |
| `description` | `evaluation_criteria` | `description` | text | Nullable |
| `maxScore` | `evaluation_criteria` | `max_score` | decimal(5,2) | |
| `weight` | `evaluation_criteria` | `weight` | decimal(5,2) | Nullable |
| `evaluationId` | `evaluations` | `evaluation_id` | uuid | PK |
| `studentId` | `evaluations` | `student_id` | uuid | FK → `students` |
| `internshipId` | `evaluations` | `internship_id` | uuid | FK → `internship_groups` |
| `evaluatorId` | `evaluations` | `evaluator_id` | uuid | FK → `enterprise_users` |
| `totalScore` | `evaluations` | `total_score` | decimal(5,2) | |
| `generalComment` | `evaluations` | `general_comment` | text | Nullable |
| `status` (Evaluation) | `evaluations` | `status` | smallint | 0=Pending, 1=Draft, 2=Submitted, 3=Published |
| `gradedAt` | `evaluations` | `graded_at` | timestamptz | Set khi chấm điểm |
| `detailId` | `evaluation_details` | `detail_id` | uuid | PK |
| `score` (Detail) | `evaluation_details` | `score` | decimal(5,2) | |
| `comment` (Detail) | `evaluation_details` | `comment` | text | Nullable |
| `fullName` | `users` | `full_name` | varchar(100) | Join `students` → `users` |
| `studentCode` | `users` | `user_code` | varchar(10) | Join `students` → `users` |

### 4.2.2 Logic Quan hệ (Relationship Tracing)

**Luồng truy xuất Chu kỳ của Mentor:**
```
Token.EnterpriseUserId 
  → internship_groups.mentor_id (WHERE mentor_id = @MentorId)
  → internship_groups.term_id 
  → evaluation_cycles.term_id (WHERE term_id = @TermId)
```

**Luồng truy xuất Sinh viên của Nhóm:**
```
internship_groups.internship_id 
  → internship_students.internship_id (WHERE internship_id = @InternshipId)
  → internship_students.student_id 
  → students.student_id → users.user_id (JOIN để lấy full_name, user_code)
```

**Luồng truy xuất Đánh giá:**
```
evaluations (WHERE cycle_id = @CycleId AND internship_id = @InternshipId AND student_id = @StudentId)
  → evaluation_details (WHERE evaluation_id = @EvaluationId)
  → evaluation_criteria (JOIN criteria_id để lấy name, max_score)
```

### 4.2.3 [⚠️ DB Mismatch] Cảnh báo Sai lệch

1. **Thiếu liên kết trực tiếp `evaluation_cycles` → `internship_groups`**:
   - Issue yêu cầu Mentor quản lý Chu kỳ **theo nhóm thực tập** (`internship_group`). Tuy nhiên, bảng `evaluation_cycles` chỉ có FK `term_id` (ref → `terms`), **KHÔNG có** FK `internship_id` (ref → `internship_groups`).
   - **Hệ quả**: Một Chu kỳ thuộc về một Term, nhưng một Term có thể có nhiều `internship_groups` (nhiều Mentor). Do đó, Chu kỳ đánh giá hiện tại là **dùng chung cho toàn bộ Term**, không phải riêng lẻ từng nhóm.
   - **Hướng xử lý (Giữ nguyên DB)**: Chấp nhận rằng Chu kỳ đánh giá thuộc cấp Term. Mentor tạo Chu kỳ cho Term, và khi chấm điểm thì filter theo `internship_id` tại bảng `evaluations`. Bảng `evaluations` đã có cột `internship_id` để phân biệt nhóm nào.

2. **Thiếu ràng buộc ai được tạo Chu kỳ**:
   - Bảng `evaluation_cycles` có `created_by uuid` nhưng KHÔNG có FK ref cụ thể. Backend cần tự validate rằng `created_by` là `enterprise_user_id` của Mentor thuộc doanh nghiệp có nhóm thực tập trong Term đó.

## 4.3 [FFA-ACV] Quy tắc Validation

### 4.3.1 Validation cho Chu kỳ (Evaluation Cycle)

| Trường | Required | MaxLength / Type | Rule | Error Message |
| --- | --- | --- | --- | --- |
| `name` | ✅ | MaxLength(255) | Không rỗng, không trùng tên trong cùng Term (optional) | `"Tên chu kỳ là bắt buộc"` / `"Tên không được vượt quá 255 ký tự"` |
| `startDate` | ✅ | timestamptz | Phải là ngày hợp lệ | `"Ngày bắt đầu là bắt buộc"` |
| `endDate` | ✅ | timestamptz | Phải > `startDate` | `"Ngày kết thúc phải lớn hơn ngày bắt đầu"` |
| `internshipId` (Create) | ✅ | uuid | Regex GUID chuẩn, phải tồn tại trong DB và Mentor phải sở hữu | `"internshipId": "Nhóm thực tập không hợp lệ"` |

### 4.3.2 Validation cho Tiêu chí (Evaluation Criteria)

| Trường | Required | MaxLength / Type | Rule | Error Message |
| --- | --- | --- | --- | --- |
| `name` | ✅ | MaxLength(255) | Không rỗng | `"Tên tiêu chí là bắt buộc"` |
| `description` | ❌ | text | Nullable | — |
| `maxScore` | ✅ | decimal(5,2) | > 0 | `"Điểm tối đa phải lớn hơn 0"` |
| `weight` | ❌ | decimal(5,2) | >= 0 nếu có | `"Trọng số không được âm"` |

### 4.3.3 Validation cho Chấm điểm (Evaluation)

| Trường | Required | MaxLength / Type | Rule | Error Message |
| --- | --- | --- | --- | --- |
| `studentId` | ✅ | uuid | Regex GUID, phải thuộc `internship_students` của nhóm | `"Sinh viên không thuộc nhóm thực tập"` |
| `score` | ✅ | decimal(5,2) | `0 <= score <= criteria.max_score` | `"Điểm không được vượt quá {maxScore}"` / `"Điểm không được âm"` |
| `generalComment` | ❌ | text | Nullable | — |
| `comment` (per criteria) | ❌ | text | Nullable | — |
| `criteriaId` | ✅ | uuid | Phải thuộc Chu kỳ đang chấm | `"Tiêu chí không thuộc chu kỳ đánh giá này"` |

### 4.3.4 Validation cho UUID Parameters

- Tất cả `cycleId`, `criteriaId`, `internshipId`, `studentId` trên URL Path / Request Body đều phải khớp Regex GUID chuẩn: `^[0-9a-fA-F]{8}-([0-9a-fA-F]{4}-){3}[0-9a-fA-F]{12}$`. Nếu sai format → `400 Bad Request` dạng `"[field]": "Không đúng định dạng UUID"`.

## 4.4 [FFA-SEC] Authentication & Authorization

### 4.4.1 Ràng buộc Quyền hạn (RBAC)

- **Toàn bộ Endpoint** của module Evaluation Management gắn Attribute `[Authorize(Roles = "Mentor")]`.
- Sinh viên (`Student`), UniAdmin, EnterpriseHR gọi API → `403 Forbidden`.
- Nếu mở rộng cho `EnterpriseHR` quản lý → thêm `[Authorize(Roles = "Mentor, EnterpriseHR")]` và bổ sung ownership check tương ứng.

### 4.4.2 Ownership Check (Chống IDOR)

- **Bước 1**: Trích xuất `Token.EnterpriseUserId` từ JWT Claim.
- **Bước 2**: Tại mọi thao tác:
  - Query `internship_groups` WHERE `mentor_id == Token.EnterpriseUserId` AND `internship_id == request.InternshipId`.
  - Nếu không tìm thấy → `403 Forbidden: "Bạn không có quyền thực hiện hành động này"`.
- **Bước 3**: Khi thao tác trên `evaluation_cycles`:
  - Xác minh Chu kỳ thuộc `term_id` mà Mentor đang quản lý nhóm thực tập.
  - Truy vấn: `evaluation_cycles.term_id` phải nằm trong danh sách `internship_groups.term_id` WHERE `mentor_id == Token.EnterpriseUserId`.
- **Bước 4**: Khi chấm điểm:
  - Xác minh `studentId` thuộc `internship_students` WHERE `internship_id` thuộc quyền quản lý của Mentor.
  - Set `evaluations.evaluator_id = Token.EnterpriseUserId` tự động, ngăn chặn Mentor giả mạo evaluator.

### 4.4.3 Response Sensitivity (FFA-SEC SEC-2)

- **KHÔNG** trả về: `password_hash`, `email` (trừ khi cần thiết), `phone_number` của sinh viên.
- Response chỉ chứa các trường cần thiết cho giao diện: `fullName`, `studentCode`, `score`, `comment`.

## 4.5 [FFA-TXG] Transaction Guard — Chấm điểm Hàng loạt

### 4.5.1 Yêu cầu Transaction cho Batch Evaluation

Handler xử lý **Batch Evaluation** bắt buộc tuân thủ `FFA-TXG`:

- **Write Operations**: Upsert N bản ghi `evaluations` + N × M bản ghi `evaluation_details` (N = số sinh viên, M = số tiêu chí).
- **Transaction Flow**:
  ```
  BeginTransactionAsync()
  try {
    foreach (student in batch) {
      Upsert evaluations record
      foreach (criteria in student.scores) {
        Upsert evaluation_details record
      }
    }
    await SaveChangesAsync()
    await CommitTransactionAsync()
  } catch {
    await RollbackTransactionAsync()
    throw
  }
  ```
- **[⚠️ Architecture Violation]**: Nếu backend KHÔNG bọc Transaction cho batch → một sinh viên lỗi sẽ gây dữ liệu bất nhất (một số có điểm, một số không). Đây là vi phạm `TX-2` (CRITICAL).

### 4.5.2 Chấm điểm Cá nhân

- Chấm điểm cá nhân cũng có ≥ 2 write operations (1 `evaluations` + N `evaluation_details`) → **BẮT BUỘC** dùng Transaction.
- Áp dụng pattern `IUnitOfWork` theo Rule `TX-1`.

## 4.6 [FFA-PERF] Cấu hình Rate Limit & Tối ưu Hiệu năng

### 4.6.1 Rate Limit

| Nhóm API | Limit | Lý do |
| --- | --- | --- |
| **GET** (List Cycles, Criteria, Grid) | 60 requests / 1 phút / IP | Read-only, tần suất trung bình |
| **POST/PUT/DELETE** (CRUD Cycle, Criteria) | 20 requests / 1 phút / IP | Mutate API nhạy cảm |
| **POST Batch Evaluation** | 10 requests / 1 phút / IP | Request body lớn, Transaction phức tạp, tốn tài nguyên server |
| **PATCH Publish** | 20 requests / 1 phút / IP | State change, cần bảo vệ khỏi spam |

### 4.6.2 EF Core Database Optimizations

- **GET Grid Chấm điểm**: Query join qua 5 bảng (`evaluation_cycles` → `evaluation_criteria`, `evaluations` → `evaluation_details`, `internship_students` → `students` → `users`). Bắt buộc:
  - `.AsNoTracking()` trên toàn bộ query SELECT (PERF-2).
  - `.Select()` projection chỉ lấy các trường cần thiết, không load toàn bộ Entity (PERF-4, PERF-5).
- **Batch Upsert**: Nếu danh sách sinh viên > 50, cân nhắc sử dụng `ExecuteSqlRawAsync` hoặc EF Core `ExecuteUpdateAsync`/`BulkExtensions` thay vì loop `Add/Update` để giảm roundtrip tới DB.
- **Pagination**: API lấy danh sách Chu kỳ phải dùng `.Skip().Take()`. Danh sách Tiêu chí và Grid chấm điểm thường số lượng nhỏ (< 50 tiêu chí, < 100 sinh viên/nhóm) nên có thể load toàn bộ nhưng vẫn nên đặt ceiling.
- **Split Query**: Khi eager loading `evaluations` → `evaluation_details` → `evaluation_criteria`, sử dụng `.AsSplitQuery()` để tránh Cartesian Explosion.