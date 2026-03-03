## 1. Tổng quan & Bối cảnh (Overview & Context)

## Vấn đề (Problem)

Hệ thống IOC cần quản lý tập trung các tài khoản doanh nghiệp (Enterprise) để:

- Cấp quyền truy cập cho doanh nghiệp tham gia vào nền tảng một cách có kiểm soát

- Lưu trữ và cập nhật hồ sơ doanh nghiệp (tên, lĩnh vực, thông tin liên hệ, v.v.) phục vụ cho các luồng gán sinh viên thực tập

- Kiểm soát trạng thái hoạt động của từng doanh nghiệp, đảm bảo chỉ doanh nghiệp hợp lệ mới xuất hiện trong các luồng nghiệp vụ

- Hỗ trợ tìm kiếm, lọc và thống kê doanh nghiệp trên toàn hệ thống

## Pain Points hiện tại

- Chưa có màn hình tập trung để tạo và quản lý tài khoản doanh nghiệp

- Không có quy trình chuẩn để cấp thông tin đăng nhập cho doanh nghiệp mới, dẫn đến việc onboard doanh nghiệp bị rải rác và thiếu nhất quán

- Khó kiểm soát doanh nghiệp nào đang hoạt động, doanh nghiệp nào đã ngừng hợp tác nhưng vẫn xuất hiện trong các luồng gán sinh viên

- Không có nơi tra cứu tập trung hồ sơ doanh nghiệp, gây khó khăn cho việc vận hành và hỗ trợ

## Giá trị Nghiệp vụ (Business Value)

- **Onboard doanh nghiệp có kiểm soát**: Sup Admin có thể tạo tài khoản và gửi thông tin đăng nhập cho doanh nghiệp ngay trên hệ thống, đảm bảo quy trình onboard chuẩn hóa và nhất quán

- **Quản lý hồ sơ tập trung**: Toàn bộ thông tin doanh nghiệp được lưu trữ trong một nguồn duy nhất, phục vụ chính xác cho các luồng gán sinh viên thực tập, đánh giá và báo cáo

- **Kiểm soát trạng thái hoạt động**: Cơ chế vô hiệu hóa/kích hoạt tài khoản đảm bảo chỉ doanh nghiệp đang hoạt động mới xuất hiện trong các luồng nghiệp vụ, tránh gán sinh viên vào doanh nghiệp không còn hợp tác

- **Hỗ trợ vận hành và báo cáo**: Danh sách doanh nghiệp tập trung giúp Sup Admin dễ dàng tra cứu, thống kê và hỗ trợ xử lý sự cố liên quan đến tài khoản doanh nghiệp

## Đối tượng (Actor)

**Primary Actor: Quản trị viên hệ thống (Sup Admin)**

- Tạo tài khoản Enterprise và gửi thông tin đăng nhập cho doanh nghiệp

- Xem, tìm kiếm và quản lý toàn bộ danh sách doanh nghiệp trên hệ thống

- Xem chi tiết hồ sơ từng doanh nghiệp

- Vô hiệu hóa hoặc kích hoạt lại tài khoản doanh nghiệp khi cần

**Secondary Actors:**

- **System Administrator**: Có thể xem và quản lý toàn bộ tài khoản doanh nghiệp, hỗ trợ vận hành cấp cao

- **Uni Admin**: Có thể xem danh sách doanh nghiệp đang hoạt động để phục vụ luồng gán sinh viên thực tập (chỉ đọc, tùy quyền)

- **Enterprise HR**: Quản lý thông tin hồ sơ của doanh nghiệp mình sau khi được cấp tài khoản

## User Story Statements

> Là Sup Admin, tôi muốn có một màn hình quản lý tài khoản doanh nghiệp tập trung (tạo, xem, xem chi tiết, vô hiệu hóa/kích hoạt) để kiểm soát toàn bộ doanh nghiệp tham gia nền tảng, đảm bảo chỉ doanh nghiệp hợp lệ mới được tham gia vào các luồng gán sinh viên thực tập.