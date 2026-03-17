## Quản trị Hệ thống & Vận hành (Admin & Operations Management)

## 1. Tổng quan & Bối cảnh (Overview & Context)

### Vấn đề (Problem):

Hiện tại, hệ thống IOC cần một module chuyên biệt để quản lý các tài khoản hành chính và vận hành (MASTER, Admin Nhà trường, Admin Doanh nghiệp, VẬN HÀNH) nhằm:

- Tạo và quản lý tài khoản hành chính nhanh chóng, trực quan
- Kiểm soát phân quyền và trạng thái hoạt động của các tài khoản hành chính
- Hỗ trợ công cụ tìm kiếm, lọc và audit cho các hoạt động quản trị

### Giá trị Nghiệp vụ (Business Value):

1. **Tăng hiệu quả vận hành**: Quy trình tạo/sửa/xóa tài khoản hành chính rõ ràng, giảm công việc thủ công.
2. **Bảo mật và kiểm soát**: Có thể tạm khoá/khôi phục tài khoản kịp thời khi phát hiện rủi ro.
3. **Quản lý quyền hạn**: Phân tách quyền cho MASTER, MODERATOR, Admin nhà trường/ doanh nghiệp.
4. **Truy vết hoạt động**: Lưu lịch sử thao tác để phục vụ audit và điều tra sự cố.

### Đối tượng (Actor):

- **Primary Actor**: Quản trị hệ thống (MASTER) — có quyền cao nhất để tạo/sửa/xóa và phân quyền.
- **Secondary Actors**:
- Admin Nhà trường, Admin Doanh nghiệp: tương tự master nhưng chỉ có thể thực hiện trong đơn vị của mình
- VẬN HÀNH: Tương tự Master nhưng chỉ có thể xem

### User Story Statement

Là Quản trị hệ thống, tôi muốn quản lý các tài khoản hành chính và vận hành (tạo, phê duyệt, cập nhật, vô hiệu/hồi phục, phân quyền, audit) để đảm bảo an toàn và hiệu quả quản trị hệ thống.

---

## 2. Luồng Người dùng (User Flow)

### 2.1. Luồng Xem & Quản lý danh sách tài khoản hành chính

```
1. MASTER truy cập trang "Bảng Quản trị Hệ thống & Vận hành"
2. Hệ thống hiển thị danh sách tài khoản hành chính: STT, Mã, Họ tên, Email, Đơn vị, Vai trò, Trạng thái, Ngày tạo
3. Hỗ trợ:
   - Tìm kiếm (debounce 300ms) theo từ khóa
   - Bộ lọc nâng cao (Advanced Filter) cho phép lọc theo: Email, Mã, Đơn vị, Vai trò, Trạng thái, Ngày tạo
   - Sắp xếp theo nhiều cột: Họ tên, Email, Đơn vị, Ngày tạo
   - Phân trang: 10/20/50/100 bản ghi mỗi trang
4. Các hành động: Tạo, Chỉnh sửa, Đặt lại mật khẩu, Vô hiệu/Hồi phục, Xóa
5. Mọi thao tác hiển thị thông báo thành công/lỗi và ghi log cho audit
```

### 2.2. Luồng Tạo tài khoản hành chính

```
1. MASTER hoặc người có quyền nhấn nút "Tạo mới" trên trang quản lý
2. Hiển thị modal "Tạo tài khoản hành chính" với form gồm các trường:
   - Họ tên (bắt buộc, text input)
   - Email (bắt buộc, email input)
   - Vai trò (bắt buộc, select dropdown - chọn từ danh sách roles hệ thống)
   - Đơn vị (bắt buộc, phụ thuộc vào vai trò được chọn):
     * Vai trò INTERNAL: Tự động hiển thị "Nội bộ" (disabled input)
     * Vai trò SCHOOL: Hiển thị Combobox để chọn từ danh sách trường học
     * Vai trò ENTERPRISE: Hiển thị Combobox để chọn từ danh sách doanh nghiệp
     * Chưa chọn vai trò: Hiển thị input disabled với placeholder "Vui lòng chọn vai trò"
   - Mã
   - Mật khẩu (bắt buộc, password input)
   - Xác nhận mật khẩu (bắt buộc, password input)
3. Validate form:
   - Validate khi blur (onBlur mode)
   - Re-validate khi thay đổi sau lần validate đầu tiên (onChange reValidateMode)
   - Kiểm tra các điều kiện:
     * Họ tên không được để trống
     * Email hợp lệ (format email) và không được để trống
     * Mật khẩu đủ mạnh (theo password schema)
     * Xác nhận mật khẩu phải khớp với mật khẩu
     * Vai trò phải được chọn
     * Đơn vị phải được chọn (nếu vai trò yêu cầu)
4. Khi nhấn "Tạo":
   - Nếu form hợp lệ: Gửi request tạo tài khoản với trạng thái mặc định ACTIVE
   - Nếu email trùng: Hiển thị modal lỗi "Email đã tồn tại trong hệ thống", form vẫn mở
   - Nếu thành công: Modal đóng, danh sách tự động cập nhật (React Query invalidation), hiển thị toast thông báo thành công, ghi log hành động
```

### 2.3. Luồng Cập nhật tài khoản hành chính

```
1. MASTER hoặc người có quyền nhấn menu trên dòng tài khoản cần chỉnh sửa
2. Chọn "Chỉnh sửa" từ menu
3. Hiển thị modal "Chỉnh sửa tài khoản hành chính" với form đã điền sẵn dữ liệu hiện tại:
   - Họ tên (bắt buộc, text input, có giá trị hiện tại)
   - Email (disabled, hiển thị email hiện tại - không thể chỉnh sửa)
   - Vai trò (bắt buộc, select dropdown - có giá trị hiện tại)
   - Đơn vị (bắt buộc, phụ thuộc vào vai trò được chọn - có giá trị hiện tại):
     * Vai trò INTERNAL: Tự động hiển thị "Nội bộ" (disabled input)
     * Vai trò SCHOOL: Hiển thị Combobox với trường học hiện tại được chọn
     * Vai trò ENTERPRISE: Hiển thị Combobox với doanh nghiệp hiện tại được chọn
   - Mã
   - Không hiển thị trường Mật khẩu và Xác nhận mật khẩu
4. Validate form:
   - Validate khi blur (onBlur mode)
   - Re-validate khi thay đổi sau lần validate đầu tiên (onChange reValidateMode)
   - Kiểm tra các điều kiện:
     * Họ tên không được để trống
     * Vai trò phải được chọn
     * Đơn vị phải được chọn (nếu vai trò yêu cầu)
5. Khi nhấn "Cập nhật":
   - Nếu form hợp lệ: Gửi request cập nhật (không gửi email, không gửi mật khẩu)
   - Nếu thành công: Modal đóng, danh sách tự động cập nhật, hiển thị toast thông báo thành công, ghi log hành động
```

### 2.4. Luồng Xóa tài khoản hành chính

```
1. MASTER nhấn menu trên dòng tài khoản cần xóa
2. Chọn "Xóa" từ menu
3. Hiển thị modal xác nhận xóa:
   - Tiêu đề: "Xóa tài khoản hành chính"
   - Nội dung: Thông báo xác nhận xóa
   - Nút: "Xóa" và "Hủy"
4. Khi nhấn "Xóa" trong modal xác nhận:
   - Gửi request xóa tài khoản
   - Nếu thành công: 
     * Modal xác nhận đóng
     * Danh sách tự động cập nhật
     * Hiển thị toast thông báo thành công
     * Ghi log audit
   - Nếu có lỗi ràng buộc (constraint/foreign key):
     * Hiển thị modal lỗi với thông báo "Không thể xóa tài khoản do có dữ liệu liên kết"
     * Modal xác nhận đóng
     * Tài khoản không bị xóa
```

### 2.5. Luồng Vô hiệu hóa / Kích hoạt tài khoản hành chính

```
1. MASTER nhấn menu trên dòng tài khoản cần vô hiệu hóa/kích hoạt
2. Tùy theo trạng thái hiện tại của tài khoản:
   - Nếu trạng thái ACTIVE: Hiển thị option "Vô hiệu" trong menu
   - Nếu trạng thái INACTIVE: Hiển thị option "Hồi phục" trong menu
3. Chọn "Vô hiệu" hoặc "Hồi phục"
4. Hiển thị modal xác nhận:
   - Nếu vô hiệu hóa (từ ACTIVE → INACTIVE):
     * Tiêu đề: "Vô hiệu tài khoản hành chính"
     * Nội dung: Thông báo xác nhận vô hiệu hóa
     * Nút: "Vô hiệu" và "Hủy"
   - Nếu kích hoạt (từ INACTIVE → ACTIVE):
     * Tiêu đề: "Kích hoạt tài khoản hành chính"
     * Nội dung: Thông báo xác nhận kích hoạt
     * Nút: "Kích hoạt" và "Hủy"
5. Khi nhấn "Vô hiệu" hoặc "Kích hoạt":
   - Gửi request cập nhật trạng thái (ACTIVE ↔ INACTIVE)
   - Nếu thành công:
     * Modal xác nhận đóng
     * Danh sách tự động cập nhật
     * Hiển thị thông báo thành công tương ứng
     * Trạng thái trong bảng cập nhật (ACTIVE/INACTIVE)
     * Ghi log audit
   - Nếu có lỗi:
     * Hiển thị modal lỗi với thông báo lỗi
     * Modal xác nhận đóng
```

### 2.6. Luồng Đặt lại mật khẩu tài khoản hành chính

```
1. MASTER hoặc người có quyền nhấn menu trên dòng tài khoản cần đặt lại mật khẩu
2. Chọn "Đặt lại mật khẩu" từ menu
3. Hiển thị modal "Đặt lại mật khẩu" với form:
   - Thông tin tài khoản (Họ tên, Email - readonly)
   - Xác nhận đặt lại mật khẩu
4. Khi xác nhận đặt lại mật khẩu:
   - Gửi request đặt lại mật khẩu về mật khẩu mặc định an toàn
   - Nếu thành công:
     * Modal đóng
     * Hiển thị toast thông báo thành công
     * Ghi log audit
```

### 2.7. Quyền truy cập & Phê duyệt

```
1. MASTER có thể cấp/revoke quyền cho MODERATOR và các admin khác
2. Các thay đổi quyền được ghi lại và có thể truy vấn trong phần audit
```

---

## 3. Tiêu chí Chấp nhận (Acceptance Criteria)

**AC-AOM-01** — Hiển thị danh sách tài khoản hành chính\
**Given**: MASTER truy cập trang "Bảng Quản trị Hệ thống & Vận hành"\
**When**: Trang được tải\
**Then**:

- Hiển thị bảng với các cột: STT, Mã, Họ tên, Email, Đơn vị, Vai trò, Trạng thái, Ngày tạo
- Có nút "Tạo mới" để thêm tài khoản
- Có thanh tìm kiếm và bộ lọc nâng cao
- Có phân trang hoạt động với các tùy chọn: 10/20/50/100 bản ghi mỗi trang

**AC-AOM-02** — Tạo tài khoản hành chính thành công\
**Given**: MASTER mở modal "Tạo tài khoản hành chính"\
**When**:

- Điền form hợp lệ với các trường: Họ tên, Email, Vai trò, Đơn vị (tùy theo vai trò), Mật khẩu, Xác nhận mật khẩu
- Trường "Đơn vị" thay đổi dựa trên vai trò được chọn:
  - Vai trò INTERNAL: Tự động hiển thị "Nội bộ" (disabled)
  - Vai trò SCHOOL: Chọn trường học từ Combobox
  - Vai trò ENTERPRISE: Chọn doanh nghiệp từ Combobox
- Nhấn "Tạo"\
  **Then**:
- Tài khoản được tạo thành công
- Modal đóng
- Danh sách tự động cập nhật (React Query invalidation)
- Hiển thị thông báo thành công
- Ghi log hành động

**AC-AOM-03** — Tạo tài khoản thất bại khi email trùng\
**Given**: Email đã tồn tại trong hệ thống\
**When**: MASTER cố gắng tạo tài khoản với email trùng và submit form\
**Then**:

- Hiển thị modal lỗi với thông báo "Email đã tồn tại trong hệ thống"
- Không tạo tài khoản
- Modal form vẫn mở, giữ nguyên nội dung đã nhập
- Có thể đóng modal lỗi và sửa lại email

**AC-AOM-04** — Chỉnh sửa tài khoản thành công\
**Given**: MASTER/Moderator mở modal chỉnh sửa từ danh sách\
**When**:

- Modal hiển thị dữ liệu hiện tại của tài khoản
- Trường Email bị disabled (không thể chỉnh sửa)
- Cập nhật các trường hợp lệ (Họ tên, Vai trò, Đơn vị) và lưu\
  **Then**:
- Dữ liệu cập nhật thành công (Email không thay đổi)
- Modal đóng
- Danh sách tự động cập nhật phản ánh thay đổi
- Hiển thị thông báo thành công
- Ghi log hành động

**AC-AOM-05** — Đặt lại mật khẩu hoạt động đúng\
**Given**: MASTER/Moderator chọn "Đặt lại mật khẩu"\
**When**: Xác nhận đặt lại mật khẩu\
**Then**: Mật khẩu tài khoản được đặt về mặc định an toàn, ghi log và thông báo thành công.

**AC-AOM-06** — Vô hiệu/hồi phục tài khoản thay đổi trạng thái và ghi log audit\
**Given**: MASTER muốn vô hiệu hóa hoặc kích hoạt tài khoản\
**When**:

- Nhấn menu "..." trên dòng tài khoản
- Chọn "Vô hiệu" (nếu trạng thái ACTIVE) hoặc "Hồi phục" (nếu trạng thái INACTIVE)
- Xác nhận trong modal xác nhận\
  **Then**:
- Nếu vô hiệu hóa (ACTIVE → INACTIVE):
  - Hiển thị modal xác nhận với tiêu đề "Vô hiệu tài khoản hành chính"
  - Khi xác nhận: Trạng thái tài khoản cập nhật thành INACTIVE
  - Trạng thái trong bảng cập nhật (hiển thị INACTIVE)
  - Hiển thị thông báo "Đã vô hiệu hóa tài khoản thành công"
  - Ghi log audit
- Nếu kích hoạt (INACTIVE → ACTIVE):
  - Hiển thị modal xác nhận với tiêu đề "Kích hoạt tài khoản hành chính"
  - Khi xác nhận: Trạng thái tài khoản cập nhật thành ACTIVE
  - Trạng thái trong bảng cập nhật (hiển thị ACTIVE)
  - Hiển thị thông báo "Đã kích hoạt tài khoản thành công"
  - Ghi log audit
- Modal xác nhận đóng sau khi hoàn tất
- Danh sách tự động cập nhật (React Query invalidation)

**AC-AOM-07** — Xóa tài khoản và xử lý lỗi ràng buộc\
**Given**: MASTER thực hiện xóa tài khoản\
**When**:

- Nhấn nút "Xóa" từ menu dropdown
- Xác nhận xóa trong modal xác nhận
- Thao tác được xác nhận\
  **Then**:
- Nếu xóa thành công: Tài khoản bị xóa, modal đóng, danh sách tự động cập nhật, hiển thị thông báo thành công, ghi log audit
- Nếu có ràng buộc (constraint/foreign key): Hiển thị modal lỗi với thông báo "Không thể xóa tài khoản do có dữ liệu liên kết", modal xác nhận đóng, tài khoản không bị xóa

**AC-AOM-08** — Tìm kiếm/ Lọc/ Sắp xếp hoạt động đúng (debounce 300ms)\
**Given**: Nhiều tài khoản trong hệ thống\
**When**:

- Nhập từ khóa vào thanh tìm kiếm (debounce 300ms)
- Áp dụng bộ lọc nâng cao (Advanced Filter) với các trường: Email, Đơn vị, Vai trò, Trạng thái, Ngày tạo
- Sắp xếp theo các cột: Họ tên, Email, Đơn vị, Ngày tạo (hỗ trợ multi-column sorting)\
  **Then**:
- Danh sách hiển thị đúng kết quả phù hợp với từ khóa tìm kiếm (debounce 300ms)
- Bộ lọc nâng cao hoạt động đúng với các điều kiện đã chọn
- Sắp xếp hoạt động đúng theo các cột đã chọn

**AC-AOM-09** — Phân quyền granular (MASTER vs MODERATOR)\
**Given**: Một người dùng có quyền MODERATOR\
**When**: Truy cập trang quản lý tài khoản\
**Then**: MODERATOR chỉ thấy/ chỉnh sửa các trường được phép; thao tác tạo/xóa chỉ dành cho MASTER.

**AC-AOM-10** — Performance: tải danh sách trong &lt; 2s với 10k tài khoản\
**Given**: Hệ thống có lượng lớn tài khoản (ví dụ 10k)\
**When**: Tải trang danh sách\
**Then**: Trang trả về dữ liệu/hiển thị trong thời gian hợp lý (&lt; 2s) theo SLA.