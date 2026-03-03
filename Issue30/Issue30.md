# 1. Tổng quan & Bối cảnh (Overview & Context)

**Vấn đề (Problem):**

Sinh viên cần một nơi tập trung để nắm bắt đầy đủ thông tin về các dự án thực tập trước khi quyết định đăng ký. Hiện tại, nếu thông tin bị chia nhỏ hoặc thiếu các tài liệu mô tả đính kèm, sinh viên sẽ khó đánh giá được mức độ phù hợp của dự án với năng lực bản thân.

**Giá trị Nghiệp vụ (Business Value):**

- **Minh bạch thông tin:** Cung cấp cái nhìn toàn diện về mục tiêu và yêu cầu của dự án.

- **Tăng trải nghiệm người dùng:** Giúp sinh viên dễ dàng tiếp cận các tài nguyên bổ trợ (file mô tả, hướng dẫn) ngay tại trang chi tiết.

- **Hỗ trợ ra quyết định:** Giúp tỷ lệ khớp lệnh (match) giữa sinh viên và dự án cao hơn nhờ hiểu rõ yêu cầu.

**Đối tượng (Actor):**

- **Primary Actor:** Sinh viên (Student) — Người xem thông tin để định hướng thực tập.

- **Secondary Actors:** Mentor/Admin — Người cung cấp và cấu hình nội dung hiển thị.

# 2. Luồng Người dùng (User Flow)

#### 2.1. Luồng Truy cập & Xem chi tiết dự án

1. Sinh viên đăng nhập vào hệ thống Cổng thông tin thực tập.

2. Từ trang "Danh sách dự án" hoặc "Dự án của tôi", sinh viên nhấn vào **Tên dự án** hoặc nút **"Xem chi tiết"**.

3. Hệ thống điều hướng đến màn hình **"Chi tiết dự án"**.

4. Hệ thống thực hiện gọi API lấy dữ liệu và hiển thị các thành phần:

   - Thông tin cơ bản: Tên dự án, Lĩnh vực, Người phụ trách, Trạng thái (Status), Ngày bắt đầu - kết thúc, Ngày tạo. [Missing field]

   - Nội dung chuyên sâu: Mô tả dự án.

   - Tài nguyên: Danh sách file đính kèm/Liên kết.

#### 2.2. Luồng Tương tác với tài nguyên đính kèm

1. Tại mục "Tài nguyên đính kèm", sinh viên xem danh sách các tệp tin.

2. Sinh viên nhấn vào tên file hoặc biểu tượng hành động:

   - **Đối với file (PDF, Hình ảnh):** Hệ thống mở tab mới để xem trước.

   - **Đối với file (Zip, Docx):** Hệ thống thực hiện tải xuống máy cá nhân.

   - **Đối với liên kết (URL):** Hệ thống điều hướng đến trang đích ở tab mới.
    [Missing] - Hệ thống sử dụng trực tiếp Resource URL trả về từ API để tải file hoặc mở liên kết trên tab mới phụ thuộc vào loại tài nguyên.
---

# 3. Tiêu chí Chấp nhận (Acceptance Criteria)

**AC-IPM-01 — Hiển thị đầy đủ các trường dữ liệu**

- **Given:** Sinh viên truy cập vào một dự án cụ thể.

- **When:** Hệ thống trả về dữ liệu thành công.

- **Then:** Hệ thống phải hiển thị đầy đủ các trường thông tin sau:

  - **Tên dự án:** Tên dự án hiển thị nổi bật ở đầu trang.

  - **Thời gian:** Hiển thị Ngày bắt đầu (StartDate), Ngày kết thúc (EndDate) và Ngày tạo dự án (CreatedAt).

  - **Trạng thái:** Current lifecycle status của dự án.

  - **Mô tả:** Mô tả dự án (Description).

  - **Người phụ trách:** Hiển thị tên Mentor hoặc Giảng viên hướng dẫn.

  - **Tài nguyên:** Danh sách các tài nguyên (ProjectResources) chứa Tên, Loại (ResourceType), và Link URL trực tiếp.

**AC-IPM-02 — Xử lý trạng thái không có tài nguyên**

- **Given:** Dự án không có tệp đính kèm nào được tải lên (ProjectResources rỗng).

- **When:** Sinh viên cuộn đến phần "Tài nguyên đính kèm".

- **Then:** Hệ thống hiển thị thông báo "Không có tài liệu đính kèm" hoặc ẩn toàn bộ section này để tối ưu giao diện.

**AC-IPM-03 — Tính năng tải và xem tài liệu**

- **Given:** Danh sách tài nguyên trả về URL tương ứng.

- **When:** Sinh viên nhấn vào tệp tin.

- **Then:** \* Các định dạng xem được trên trình duyệt (.pdf, .jpg, .png) phải mở ở tab mới.

  - Các định dạng khác (.docx, .zip) phải kích hoạt tiến trình download của trình duyệt.

  - Phải đảm bảo an toàn (không thực thi các file .exe, .sh).

[Missing] - Hệ thống sử dụng trực tiếp Resource URL trả về từ API để tải file hoặc mở liên kết trên tab mới phụ thuộc vào loại tài nguyên.

**AC-IPM-04 — Kiểm soát quyền truy cập chi tiết [⚠️ Architecture Violation]**

- **Given:** Sinh viên có liên kết (URL) trực tiếp đến một dự án.

- **When:** Dự án đó đang ở trạng thái "Bản nháp" (Draft) hoặc "Đã đóng" (Closed).

- **Then:** Hệ thống hiển thị thông báo lỗi "Dự án hiện không khả dụng" và điều hướng sinh viên quay lại trang danh sách.

[Missing] - Code Backend hiện tại đang cho phép xem chi tiết dự án bất kể trạng thái của dự án đó là gì (Vi phạm FFA-SEC Rule SEC-4 vì thiếu kiểm tra logic phân quyền/trạng thái trong Handler). Bất cứ ai có token (kể cả Student) khi gọi `GetProjectById` đều có thể xem dự án dù là Draft/Closed.
---

# 4. Quy tắc nghiệp vụ & Ràng buộc (Business Rules & Constraints)

Thuộc tính Quy tắc kỹ thuật / Nghiệp vụ

Trạng thái dự án Chỉ dự án ở trạng thái ACTIVE hoặc PUBLIC mới được hiển thị trang chi tiết cho Student.

Định dạng mô tả Phải hỗ trợ Render HTML/Markdown từ Rich Text Editor của Mentor để giữ nguyên layout.

Bảo mật tài liệu Link tải tài liệu phải có Token/Signature giới hạn thời gian để tránh bị rò rỉ link trực tiếp ra ngoài.

Giới hạn file Chỉ cho phép hiển thị các định dạng: .pdf, .docx, .pptx, .zip, .rar, .jpg, .png.

Performance Thời gian tải trang chi tiết phải &lt; 1.5s trong điều kiện mạng bình thường.

[Missing] - Hiện tại Backend không có cơ chế kiểm soát định dạng file tải lên (FFA-SEC Rule SEC-5: Input Validation) và không giới hạn các định dạng file được phép. Bất kỳ file nào được upload lên S3 đều có thể được truy cập nếu biết URL.