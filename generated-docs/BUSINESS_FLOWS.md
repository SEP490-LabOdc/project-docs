# Phân tích Business Flows cho Lab-based ODC Platform

## 1. Phân tích yêu cầu

Mô hình Lab-based ODC Platform là sự kết hợp giữa đào tạo nhân sự theo yêu cầu và cung cấp đội ngũ phát triển theo mô hình Offshore Development Center (ODC) cho doanh nghiệp. Mô hình này khác biệt so với ODC truyền thống ở các điểm sau:

- Xây dựng Talent Pool gồm sinh viên, fresher, và junior developer
- Đào tạo theo yêu cầu kỹ năng cụ thể từ doanh nghiệp (stack, công cụ, quy trình Agile/Scrum)
- Tổ chức thành các mini-ODC team ngay trong Lab, có mentor/senior trực tiếp dẫn dắt
- Cung cấp platform để doanh nghiệp quản lý trực tiếp tiến độ, công việc của team
- Hỗ trợ hợp đồng linh hoạt: thuê theo tháng, theo dự án, hoặc tuyển dụng chính thức

## 2. Xác định các Actor

1. **Lab Management**: Đội ngũ quản lý Lab, chịu trách nhiệm vận hành toàn bộ hệ thống
2. **Doanh nghiệp (Business)**: Khách hàng sử dụng dịch vụ ODC
3. **Talent Pool**: Sinh viên, fresher, junior developer tham gia vào hệ thống
4. **Mentor/Senior**: Người hướng dẫn và đảm bảo chất lượng cho các team
5. **ODC Team**: Các nhóm phát triển được tổ chức trong Lab
6. **Platform System**: Hệ thống nền tảng quản lý và vận hành

## 3. Use Case chính

1. **Quản lý Talent Pool**: Tuyển chọn, đánh giá và phân loại nhân sự
2. **Đào tạo theo yêu cầu**: Thiết kế và triển khai chương trình đào tạo theo yêu cầu của doanh nghiệp
3. **Thành lập ODC Team**: Tổ chức và quản lý các mini-ODC team
4. **Quản lý dự án**: Theo dõi tiến độ và chất lượng công việc của các team
5. **Quản lý hợp đồng**: Xử lý các loại hợp đồng khác nhau với doanh nghiệp
6. **Chuyển giao nhân sự**: Quy trình chuyển giao nhân sự cho doanh nghiệp

## 4. Định nghĩa 6 Main Business Flows

### Flow 1: Onboarding doanh nghiệp mới
- Doanh nghiệp đăng ký sử dụng dịch vụ
- Lab Management tiếp nhận và phân tích yêu cầu
- Thỏa thuận về quy mô team, kỹ năng cần thiết, thời gian và chi phí
- Ký kết hợp đồng và thiết lập tài khoản trên platform

### Flow 2: Xây dựng và đào tạo Talent Pool
- Lab Management tuyển chọn ứng viên vào Talent Pool
- Đánh giá kỹ năng và phân loại ứng viên
- Thiết kế chương trình đào tạo phù hợp với yêu cầu doanh nghiệp
- Mentor/Senior thực hiện đào tạo và đánh giá

### Flow 3: Thành lập và vận hành ODC Team
- Lab Management lựa chọn thành viên từ Talent Pool
- Tổ chức thành các mini-ODC team với cấu trúc phù hợp
- Phân công Mentor/Senior phụ trách từng team
- Thiết lập quy trình làm việc và công cụ quản lý

### Flow 4: Quản lý dự án và theo dõi tiến độ
- Doanh nghiệp giao nhiệm vụ và yêu cầu cho ODC Team thông qua platform
- ODC Team thực hiện công việc dưới sự hướng dẫn của Mentor/Senior
- Doanh nghiệp theo dõi tiến độ và chất lượng công việc trên platform
- Lab Management đảm bảo chất lượng và giải quyết các vấn đề phát sinh

### Flow 5: Đánh giá và cải tiến
- Doanh nghiệp đánh giá hiệu suất của ODC Team
- Lab Management thu thập phản hồi từ các bên liên quan
- Mentor/Senior đề xuất các biện pháp cải tiến
- Điều chỉnh quy trình và nhân sự nếu cần thiết

### Flow 6: Chuyển giao nhân sự
- Doanh nghiệp yêu cầu tuyển dụng chính thức thành viên từ ODC Team
- Lab Management đánh giá và thỏa thuận điều kiện chuyển giao
- Xử lý thủ tục và phí placement
- Hoàn tất quá trình chuyển giao và hỗ trợ onboarding

### Flow 7: Skill-Based Matching
- Doanh nghiệp cung cấp yêu cầu chi tiết về kỹ năng và dự án
- Lab Management tạo profile yêu cầu trên hệ thống matching
- Hệ thống phân tích và so khớp với dữ liệu Talent Pool
- Đề xuất danh sách ứng viên phù hợp nhất với tỷ lệ matching
- Doanh nghiệp xem hồ sơ chi tiết và lựa chọn ứng viên
- Lab Management điều chỉnh tiêu chí dựa trên phản hồi

### Flow 8: Mini-Project Marketplace
- Doanh nghiệp đăng dự án lên marketplace với yêu cầu chi tiết
- Lab Management xem xét và phê duyệt dự án
- ODC Teams xem xét dự án và gửi đề xuất giải pháp
- Doanh nghiệp lựa chọn team phù hợp nhất
- Thực hiện dự án với quy trình quản lý tích hợp
- Đánh giá và xếp hạng sau khi hoàn thành

### Flow 9: Interactive Learning
- Lab Management tạo chương trình đào tạo tùy chỉnh
- Talent Pool đăng ký và tham gia khóa học
- Học viên thực hiện các bài học, bài tập và dự án thực hành
- Hệ thống tự động đánh giá và cung cấp phản hồi
- Mentor xem xét tiến độ và hỗ trợ khi cần thiết
- Cấp chứng chỉ và badges khi hoàn thành