# 1. Tổng quan & Bối cảnh (Overview & Context)

## Vấn đề (Problem)

Sau khi kỳ thực tập được tạo, hệ thống IOC cần một cơ chế để Uni Admin có thể đưa sinh viên vào kỳ thực tập một cách có kiểm soát:

- Số lượng sinh viên tham gia mỗi kỳ thực tập có thể lên đến hàng trăm, việc thêm thủ công từng sinh viên là không khả thi

- Cần theo dõi rõ ràng sinh viên nào đã được gán doanh nghiệp (Placed) và sinh viên nào chưa (Unplaced) trong từng kỳ

- Cần có khả năng rút sinh viên khỏi kỳ khi có sai sót import hoặc sinh viên được duyệt nghỉ phép

- Thông tin enrollment của từng sinh viên cần được xem và kiểm tra chi tiết để phục vụ các luồng gán doanh nghiệp, đánh giá và báo cáo

## Pain Points hiện tại

- Chưa có cơ chế import hàng loạt sinh viên vào kỳ thực tập, gây tốn thời gian và dễ sai sót khi thêm thủ công

- Không có màn hình tổng hợp để theo dõi trạng thái Placed/Unplaced của từng sinh viên trong kỳ

- Không kiểm soát được sinh viên đã enroll nhầm kỳ hoặc cần rút ra do nghỉ phép

- Dữ liệu enrollment bị rải rác, không đồng bộ giữa các module (gán doanh nghiệp, đánh giá, báo cáo)

- Không có nơi xem chi tiết thông tin enrollment của từng sinh viên trong một kỳ cụ thể

## Giá trị Nghiệp vụ (Business Value)

- **Import hàng loạt hiệu quả**: Uni Admin có thể import hàng trăm sinh viên vào kỳ thực tập chỉ bằng một file Excel, tiết kiệm thời gian và giảm thiểu sai sót nhập liệu thủ công

- **Theo dõi trạng thái tập trung**: Cung cấp màn hình tổng hợp Placed/Unplaced giúp Uni Admin nắm bắt nhanh tiến độ gán doanh nghiệp, từ đó kịp thời xử lý các sinh viên chưa có nơi thực tập

- **Dữ liệu enrollment chính xác**: Khả năng xem chi tiết và rút sinh viên khỏi kỳ giúp đảm bảo danh sách enrollment luôn phản ánh đúng thực tế, tránh sai lệch ảnh hưởng đến các luồng đánh giá và báo cáo

- **Kiểm soát vòng đời enrollment**: Uni Admin có đầy đủ công cụ để quản lý toàn bộ vòng đời của một enrollment — từ import, theo dõi, xem chi tiết đến rút sinh viên khi cần

## Đối tượng (Actor)

**Primary Actor: Quản trị viên trường (Uni Admin)**

- Import sinh viên vào kỳ thực tập thông qua file Excel

- Xem, lọc và theo dõi danh sách sinh viên theo trạng thái Placed/Unplaced trong từng kỳ

- Xem chi tiết thông tin enrollment và placement của từng sinh viên

- Rút sinh viên khỏi kỳ khi cần thiết (sai sót import, sinh viên nghỉ phép)

**Secondary Actors:**

- **Student (Sinh viên)**: Được enroll vào kỳ thực tập, có thể xem thông tin enrollment của bản thân (chỉ đọc)

- **Enterprise HR / Mentor**: Xem danh sách sinh viên được gán vào doanh nghiệp mình trong kỳ (chỉ đọc)