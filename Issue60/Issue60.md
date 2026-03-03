# Tổng quan & Bối cảnh (Overview & Context)

## Vấn đề (Problem)

Hiện tại, hệ thống IOC cần quản lý các kỳ thực tập (Internship Term) một cách tập trung và có cấu trúc để:

- Định nghĩa rõ ràng các kỳ thực tập theo từng trường học (thời gian bắt đầu, kết thúc, trạng thái)

- Gắn kết sinh viên, doanh nghiệp, nhóm thực tập và các luồng đánh giá vào đúng kỳ thực tập tương ứng

- Kiểm soát vòng đời của kỳ thực tập: từ khởi tạo → đang hoạt động → đóng

- Đảm bảo dữ liệu thực tập không bị rải rác hoặc nhầm lẫn giữa các kỳ khác nhau

## Pain Points hiện tại

- Chưa có màn hình tập trung để tạo và quản lý kỳ thực tập

- Không kiểm soát được vòng đời kỳ thực tập: kỳ đã kết thúc vẫn có thể bị nhầm lẫn với kỳ hiện tại

- Sinh viên, HR/Mentor không có nơi tra cứu thông tin kỳ thực tập đang diễn ra

- Dữ liệu gán doanh nghiệp, nhóm thực tập, đánh giá không được ràng buộc rõ ràng với một kỳ thực tập cụ thể

- Không có cơ chế xóa kỳ thực tập tạo nhầm, dẫn đến dữ liệu rác tồn tại trong hệ thống

## Giá trị Nghiệp vụ (Business Value)

- **Quản lý vòng đời kỳ thực tập**: Uni Admin có thể tạo, theo dõi, cập nhật và đóng kỳ thực tập một cách có kiểm soát, đảm bảo các luồng nghiệp vụ liên quan (gán doanh nghiệp, đánh giá, báo cáo) hoạt động đúng kỳ

- **Dữ liệu sạch và có cấu trúc**: Mỗi kỳ thực tập là một đơn vị dữ liệu độc lập, giúp hạn chế nhầm lẫn giữa các kỳ và hỗ trợ báo cáo thống kê chính xác

- **Minh bạch thông tin cho các bên liên quan**: Sinh viên và HR/Mentor có thể chủ động tra cứu thông tin kỳ thực tập, giảm tải công việc hỏi đáp cho Uni Admin

- **Kiểm soát trạng thái nghiệp vụ**: Cơ chế đóng kỳ thực tập đảm bảo các luồng phát sinh mới (gán doanh nghiệp, nộp báo cáo) không thể thực hiện sau khi kỳ đã kết thúc

- **Dọn dẹp dữ liệu**: Cho phép xóa kỳ thực tập tạo nhầm (chưa có dữ liệu liên quan), giữ hệ thống gọn gàng

## Đối tượng (Actor)

**Primary Actor: Quản trị viên trường (Uni Admin)**

- Tạo, xem, cập nhật, đóng và xóa kỳ thực tập thuộc trường mình

- Chịu trách nhiệm đảm bảo thông tin kỳ thực tập luôn chính xác và đúng tiến độ

**Secondary Actors:**

- **Student (Sinh viên)**: Xem thông tin kỳ thực tập hiện tại đang áp dụng cho mình (chỉ đọc)

- **HR / Mentor**: Xem thông tin kỳ thực tập liên quan đến doanh nghiệp hoặc nhóm thực tập mình phụ trách (chỉ đọc)

## User Story Statement

> Là Quản trị viên trường (Uni Admin), tôi muốn quản lý kỳ thực tập tập trung (tạo, xem, cập nhật, đóng, xóa) để kiểm soát vòng đời từng kỳ thực tập, đảm bảo các luồng gán doanh nghiệp, đánh giá và báo cáo được thực hiện đúng kỳ và đúng thời điểm.