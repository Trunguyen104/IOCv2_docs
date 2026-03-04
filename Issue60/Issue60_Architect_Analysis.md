# Báo Cáo Phân Tích Kiến Trúc - Issue 60: Term Management

**Chế độ hoạt động:** IOCv2 INTERNAL ARCHITECT MODE

## BƯỚC 1 — Phân tích Feature
- **Module:** Term (Quản lý Kỳ thực tập).
- **Loại tác vụ (Command/Query):** Bao gồm cả **Command** (Create, Update, Change Status, Delete) và **Query** (Get List, Get Detail).
- **Phạm vi ảnh hưởng:** Là module dữ liệu Master Data. Rất nhiều module khác như Sinh viên, Nhóm, Điểm chác sẽ phụ thuộc vào sinh mệnh của Term.
- **Yêu cầu kỹ thuật đặc thù:**
  - **Transaction:** Bắt buộc áp dụng UnitOfWork / Transaction Pipeline trên các endpoint thuộc nhóm Command (Đặc biệt lúc xoá).
  - **Cache:** Không quá cần thiết do Master Data có thể sửa bởi Uni Admin, nhưng nếu muốn chống query nặng, có thể cache Output list theo `university_id` (kèm Invalidation khi Command trigger).
  - **Domain Event:** Sẽ rất cần thiết. Ví dụ `TermStatusChangedEvent` hoặc `TermClosedEvent`. Khi Term đổi sang Closed, hệ thống chốt sổ, module Enrolment hay Evaluation có thể lắng nghe sự kiện này làm các side-effects (như ngừng nhận sinh viên tiếp).
  - **External Integration:** Không có.

## BƯỚC 2 — Đối chiếu kiến trúc (Architecture Validation)
- **Clean Architecture & Dependency:** 
  - Toàn bộ validation (`start_date < end_date`) phải quy định tại Application (FluentValidation) hoặc Domain Entity (qua constructor/method). Tuyệt đối không để Controller IF/ELSE kiểm tra ngày.
  - Phân tách read models (DTO) và write models (Entities), không phơi entity nguyên bản ra Controller.
- **Separation of Concern (Module boundary):** 
  - Issue 60 nhắc đến việc IDOR qua trường (`university_id`). Logic lọc và xác thực quyền này phải do Application layer (Handler) đảm trách dựa vào `ICurrentUserService`. Tránh rò logic chứng thực xuống query thô.
- **Nguy cơ sai layer:** 
  - Delete vật lý rất dễ phát sinh lỗi Foreign Key Constraint Error (DbUpdateException) ở hạ tầng DB. Nếu cho Controller ném thẳng lỗi SQL ra ngoài sẽ vi phạm kiến trúc nghiêm trọng. Tầng Infrastructure / Repository phải Handle nó hoặc tầng Application bắt trước logic reference để chủ động báo `409 Conflict`.

## BƯỚC 3 — Đề xuất kiến trúc triển khai
### 1. Cấu trúc Thư mục và File
Triển khai CQRS, đặt trong `src/Core/Application/Terms/`:
- **Commands:**
  - `CreateTerm/` (`CreateTermCommand`, `CreateTermHandler`, `CreateTermValidator`)
  - `UpdateTerm/` (`UpdateTermCommand`, `UpdateTermHandler`, `UpdateTermValidator`)
  - `ChangeTermStatus/` (`ChangeTermStatusCommand`, `ChangeTermStatusHandler`)
  - `DeleteTerm/` (`DeleteTermCommand`, `DeleteTermHandler`)
- **Queries:**
  - `GetTerms/` (`GetTermsQuery`, `GetTermsHandler`)
  - `GetTermById/` (`GetTermByIdQuery`, `GetTermByIdHandler`)

### 2. Transaction Boundary & Logging
- **Transaction:** Gói tất cả Command trong một Transaction Pipeline chung của hệ thống.
- **Logging (FFA-LOG):** Phải lưu Audit log cho hành vi Write/Update/Delete tại mức Warning hoặc Info. Nếu có người cố truy cập Term khác trường $\to$ ghi log `AccessDenied Warning` kèm theo UniversityId.

### 3. Validation & Exception Strategy (FFA-ERR)
- **Validation (FluentValidation):** Đảm bảo `endDate > startDate`, `name` không rỗng và dài vừa phải.
- **Exceptions Mapping:**
  - `ValidationException` lọt thành `HTTP 400 Bad Request`.
  - `ForbiddenException` lọt thành `HTTP 403 Forbidden` do IDOR.
  - `NotFoundException` $\to$ `HTTP 404`.
  - Giữa chừng bắt lỗi check reference khi xoá: Ném custom `ConflictException` (chẳng hạn `TermInUseException`) $\to$ `HTTP 409 Conflict`.

### 4. Test Strategy (FFA-TST)
**Cần viết các bài Unit Test trọng yếu cho Handler:**
- **[Validation Rule Test]:** Pass `CreateTermCommand` có `startDate` 15-05-2026 và `endDate` 01-05-2026 $\to$ Verify handler throw `ValidationException`.
- **[Security / IDOR Test]:** Mock Token có `UniversityId` là X, thực hiện `UpdateTermCommand` truyền id term của Uni Y $\to$ Verify bay lỗi `ForbiddenException`.
- **[Delete Constraint Test]:** Thay vì phụ thuộc DB quăng lỗi Constraint, mock Repository báo Term đã có Sinh viên (has References) $\to$ `DeleteTermHandler` phải bắt được và quăng ra rào cản `ConflictException`.

## BƯỚC 4 — Đánh giá rủi ro
- 🔴 **Critical Risk (Security - Khoảng cách sở hữu):** Vấn đề Data Ownership là tử huyệt. Bất cứ Admin nào cũng có quyền Admin, nhưng API phải chặt đứt ngạch `university_id` để tránh trường A thao túng dữ liệu rác vào trường B hoặc phá hoại kỳ thực tập đang diễn ra.
- 🟠 **Architectural Risk (Delete Vật lý vs Mất Toàn vẹn Schema):** Document đã cảnh báo về trạng thái thiết kế lệch DB (không có `IsDeleted`). Khi Database không có Soft Delete, xóa nhầm Term (chưa có reference FK) sẽ làm mất vĩnh viễn dữ liệu thiết lập. Backend DEV CẦN chủ động viết lệnh Domain Service truy xuất đếm trước số lượng các entity phụ thuộc vào để chặn việc xóa một kỳ đã đi vào sử dụng lâu.
- 🟡 **Maintainability Risk:** Trạng thái lưu theo kiểu `smallint` (0,1,2). DEV cần tạo rõ một cấu trúc `public enum TermStatus { Draft=0, Open=1, Closed=2 }` và dùng Enum cho toàn bộ Logic Flow. Tránh tuyệt đối "Magic Numbers" `if (status == 1)` trong code.
- 🔵 **Performance Risk:** Tối huỷ/thấp. Truy vấn Query danh sách dùng chuẩn `.AsNoTracking()` trong EF là an toàn. Lại thêm Rate Limiting đã kẹp thì khó sụp tải Master Data.

## BƯỚC 5 — Kết luận
- **Trạng thái:** Tốt và đã được document khá chuẩn để implementation, tuy nhiên cần cẩn trọng cao độ phần thao tác Cập nhật Database vì bảng này là Foundation quan trọng.
- **Đề xuất nâng cấp (Refinement):**
  - **Tạo Custom Enum:** Chắc chắn phải có `TermStatus` thay vì số cứng dập khuôn.
  - **Về Behavior Delete:** Xin nhấn mạnh lại việc hệ thống sẽ dùng Physical Delete vì DB không đổi được. Trong `DeleteTermHandler`, trước lúc gọi `_repo.Remove(term)`, DEV CẦN thiết kế một lệnh `await _checkingService.HasAnyReferencesAsync(term.Id)` (kiểm tra bảng `internship_groups` hay tương tự) để quăng lỗi 409 từ Application Layer, ĐỪNG chờ DB chửi mắng rồi mới Handle Exception của Entity Framework, vì như thế rất rườm rà và dính coupling. 
  - **Áp dụng Domain Events:** Dùng event `TermStatusChanged` (VD: truyền sang trạng thái Closed) để tạo các pipeline background trigger chốt dữ liệu (nếu business flow tương lai cần tới).

**KHUYẾN NGHỊ:** ĐÃ SẴN SÀNG IMPLEMENT. DEV theo sát CQRS Pattern và chú ý Handle Exception 409 cho sạch.
