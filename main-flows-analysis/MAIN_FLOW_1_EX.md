# Phân tích UI/UX cho Main Flow 1: Onboarding doanh nghiệp mới

## 1. Các bước trong Main Flow 1

Main Flow 1 bao gồm 4 bước chính:
1.  Doanh nghiệp đăng ký sử dụng dịch vụ.
2.  Lab Management tiếp nhận và phân tích yêu cầu.
3.  Thỏa thuận về quy mô team, kỹ năng cần thiết, thời gian và chi phí.
4.  Ký kết hợp đồng và thiết lập tài khoản trên platform.

## 2. Các màn hình cần thiết

Dựa vào các bước trên, chúng ta cần thiết kế các màn hình sau:

### 2.1. Màn hình Landing Page

**Mục đích**: Giới thiệu dịch vụ và thu hút doanh nghiệp đăng ký.

**Yêu cầu UI/UX**:
* **Bố cục**: Layout hiện đại, responsive với header, hero section, giới thiệu dịch vụ, lợi ích, testimonials và footer.
* **Màu sắc**: Sử dụng màu chủ đạo xanh dương (**#1E88E5**) kết hợp với trắng (**#FFFFFF**) và xám nhạt (**#F5F5F5**) làm nền.
* **Các thành phần**:
    * Logo và menu navigation ở header.
    * Hero section với hình ảnh minh họa và slogan nổi bật.
    * Nút **"Đăng ký dịch vụ"** màu cam nổi bật (**#FF5722**) ở vị trí dễ thấy.
    * Phần giới thiệu dịch vụ với các icon minh họa.
    * Phần testimonials từ các doanh nghiệp đã sử dụng dịch vụ.
    * Footer với thông tin liên hệ và social media links.

**Tương tác**: Khi click vào nút **"Đăng ký dịch vụ"**, chuyển đến màn hình đăng ký.

### 2.2. Màn hình Đăng ký tài khoản doanh nghiệp

**Mục đích**: Thu thập thông tin cơ bản của doanh nghiệp.

**Yêu cầu UI/UX**:
* **Bố cục**: Form đăng ký đơn giản, rõ ràng với progress indicator.
* **Màu sắc**: Giữ nhất quán với màu chủ đạo, nền trắng để dễ đọc.
* **Các thành phần**:
    * Progress bar hiển thị bước 1/4 của quy trình đăng ký.
    * Form với các trường thông tin:
        * Tên doanh nghiệp (bắt buộc).
        * Email liên hệ (bắt buộc).
        * Số điện thoại (bắt buộc).
        * Địa chỉ.
        * Lĩnh vực hoạt động (dropdown).
        * Quy mô doanh nghiệp (dropdown: <50, 50-200, >200 nhân viên).
    * Checkbox đồng ý với điều khoản dịch vụ.
    * Nút **"Tiếp tục"** màu xanh dương (**#1E88E5**).
    * Link **"Đã có tài khoản? Đăng nhập"**.

**Tương tác**: Khi hoàn thành form và click **"Tiếp tục"**, chuyển đến màn hình yêu cầu dịch vụ.

### 2.3. Màn hình Yêu cầu dịch vụ

**Mục đích**: Thu thập thông tin chi tiết về nhu cầu của doanh nghiệp.

**Yêu cầu UI/UX**:
* **Bố cục**: Form với các section rõ ràng, có thể mở rộng/thu gọn.
* **Màu sắc**: Nhất quán với màu chủ đạo.
* **Các thành phần**:
    * Progress bar hiển thị bước 2/4.
    * Form với các section:
        1.  **Nhu cầu nhân sự**:
            * Số lượng nhân sự cần (input số).
            * Thời gian dự kiến (dropdown: 1-3 tháng, 3-6 tháng, 6-12 tháng, >12 tháng).
            * Loại hợp đồng (radio buttons: theo dự án, theo tháng, tuyển dụng chính thức).
        2.  **Yêu cầu kỹ năng**:
            * Multi-select dropdown cho các tech stack.
            * Multi-select dropdown cho các công cụ.
            * Multi-select dropdown cho quy trình làm việc.
            * Text area cho yêu cầu khác.
        3.  **Mô tả dự án** (text area).
    * Nút **"Quay lại"** (màu xám).
    * Nút **"Tiếp tục"** (màu xanh dương).

**Tương tác**: Sau khi điền đầy đủ thông tin và click **"Tiếp tục"**, chuyển đến màn hình xác nhận thông tin.

### 2.4. Màn hình Xác nhận thông tin

**Mục đích**: Hiển thị tổng hợp thông tin đã nhập để doanh nghiệp xác nhận.

**Yêu cầu UI/UX**:
* **Bố cục**: Layout hiển thị thông tin theo từng section rõ ràng.
* **Màu sắc**: Nhất quán với màu chủ đạo.
* **Các thành phần**:
    * Progress bar hiển thị bước 3/4.
    * Card hiển thị thông tin doanh nghiệp.
    * Card hiển thị thông tin yêu cầu dịch vụ.
    * Ước tính chi phí sơ bộ (nếu có thể).
    * Checkbox xác nhận thông tin chính xác.
    * Nút **"Chỉnh sửa"** (màu xám) cho từng section.
    * Nút **"Quay lại"** (màu xám).
    * Nút **"Xác nhận và gửi"** (màu xanh dương).

**Tương tác**: Khi click **"Xác nhận và gửi"**, hiển thị thông báo đã gửi yêu cầu thành công và chuyển đến màn hình chờ phản hồi.

### 2.5. Màn hình Chờ phản hồi

**Mục đích**: Thông báo cho doanh nghiệp về trạng thái yêu cầu.

**Yêu cầu UI/UX**:
* **Bố cục**: Trang đơn giản với thông báo và hướng dẫn.
* **Màu sắc**: Nhất quán với màu chủ đạo.
* **Các thành phần**:
    * Progress bar hiển thị bước 4/4.
    * Icon hoặc hình ảnh minh họa (như đồng hồ cát hoặc biểu tượng chờ).
    * Thông báo **"Yêu cầu của bạn đã được gửi thành công"**.
    * Thông tin về thời gian phản hồi dự kiến.
    * Thông tin liên hệ hỗ trợ.
    * Nút **"Xem trạng thái yêu cầu"**.
    * Nút **"Quay về trang chủ"**.

**Tương tác**: Doanh nghiệp có thể click **"Xem trạng thái yêu cầu"** để theo dõi tiến trình.

### 2.6. Dashboard cho Lab Management

**Mục đích**: Giúp Lab Management xem và xử lý các yêu cầu từ doanh nghiệp.

**Yêu cầu UI/UX**:
* **Bố cục**: Dashboard với sidebar navigation và khu vực hiển thị nội dung chính.
* **Màu sắc**: Phối màu chuyên nghiệp, sử dụng màu xanh dương đậm (**#0D47A1**) cho sidebar.
* **Các thành phần**:
    * Sidebar với các menu: Dashboard, Yêu cầu mới, Đang xử lý, Đã hoàn thành, Báo cáo, Cài đặt.
    * Khu vực hiển thị danh sách yêu cầu với các cột: ID, Tên doanh nghiệp, Ngày gửi, Loại yêu cầu, Trạng thái, Hành động.
    * Bộ lọc và tìm kiếm.
    * Nút **"Xem chi tiết"** cho mỗi yêu cầu.
    * Biểu đồ thống kê số lượng yêu cầu theo trạng thái.

**Tương tác**: Khi click **"Xem chi tiết"**, hiển thị popup hoặc chuyển đến trang chi tiết yêu cầu.

### 2.7. Màn hình Chi tiết yêu cầu (cho Lab Management)

**Mục đích**: Hiển thị thông tin chi tiết về yêu cầu và cho phép xử lý.

**Yêu cầu UI/UX**:
* **Bố cục**: Layout chia thành các tab hoặc section.
* **Màu sắc**: Nhất quán với dashboard.
* **Các thành phần**:
    * Thông tin doanh nghiệp.
    * Thông tin yêu cầu dịch vụ.
    * Form đánh giá và phân tích yêu cầu với các trường:
        * Đánh giá khả thi (dropdown: Khả thi, Cần điều chỉnh, Không khả thi).
        * Ghi chú đánh giá (text area).
        * Đề xuất quy mô team (input số).
        * Đề xuất kỹ năng cần thiết (multi-select).
        * Ước tính thời gian (input).
        * Ước tính chi phí (input).
    * Nút **"Lưu nháp"** (màu xám).
    * Nút **"Gửi đề xuất cho doanh nghiệp"** (màu xanh dương).

**Tương tác**: Khi click **"Gửi đề xuất"**, hệ thống gửi email thông báo cho doanh nghiệp.

### 2.8. Màn hình Đề xuất từ Lab Management (cho doanh nghiệp)

**Mục đích**: Hiển thị đề xuất từ Lab Management và cho phép doanh nghiệp phản hồi.

**Yêu cầu UI/UX**:
* **Bố cục**: Layout hiển thị thông tin đề xuất rõ ràng.
* **Màu sắc**: Nhất quán với màu chủ đạo.
* **Các thành phần**:
    * Card hiển thị thông tin yêu cầu ban đầu.
    * Card hiển thị đề xuất từ Lab Management.
    * So sánh giữa yêu cầu và đề xuất (nếu có sự khác biệt).
    * Các tùy chọn phản hồi:
        * Radio buttons: Đồng ý, Đề xuất điều chỉnh, Từ chối.
        * Text area cho ý kiến bổ sung.
    * Nút **"Gửi phản hồi"** (màu xanh dương).

**Tương tác**: Khi click **"Gửi phản hồi"**, hệ thống gửi thông báo cho Lab Management.

### 2.9. Màn hình Ký kết hợp đồng

**Mục đích**: Cho phép doanh nghiệp xem và ký kết hợp đồng.

**Yêu cầu UI/UX**:
* **Bố cục**: Layout hiển thị nội dung hợp đồng và khu vực ký kết.
* **Màu sắc**: Nhất quán với màu chủ đạo, sử dụng màu trắng làm nền cho nội dung hợp đồng.
* **Các thành phần**:
    * Viewer hiển thị nội dung hợp đồng (PDF).
    * Checkbox xác nhận đã đọc và đồng ý với điều khoản.
    * Khu vực ký điện tử hoặc tải lên hợp đồng đã ký.
    * Nút **"Tải hợp đồng"** (màu xám).
    * Nút **"Ký kết"** (màu xanh dương).

**Tương tác**: Khi click **"Ký kết"**, hiển thị thông báo xác nhận và chuyển đến màn hình thiết lập tài khoản.

### 2.10. Màn hình Thiết lập tài khoản trên platform

**Mục đích**: Hoàn tất quá trình onboarding và thiết lập tài khoản.

**Yêu cầu UI/UX**:
* **Bố cục**: Form thiết lập với các bước.
* **Màu sắc**: Nhất quán với màu chủ đạo.
* **Các thành phần**:
    * Form thiết lập tài khoản với các trường:
        * Tên người quản lý (bắt buộc).
        * Email người quản lý (bắt buộc).
        * Mật khẩu và xác nhận mật khẩu (bắt buộc).
        * Số điện thoại liên hệ.
        * Ảnh đại diện (upload).
    * Thiết lập quyền truy cập cho các thành viên khác (nếu có).
    * Nút **"Hoàn tất"** (màu xanh dương).

**Tương tác**: Khi click **"Hoàn tất"**, chuyển đến dashboard dành cho doanh nghiệp.

## 3. Đề xuất về màu sắc và bố cục chung

### 3.1. Bảng màu chính

* **Màu chủ đạo**: Xanh dương (**#1E88E5**) - Thể hiện sự chuyên nghiệp, tin cậy.
* **Màu nhấn**: Cam (**#FF5722**) - Dùng cho các nút call-to-action quan trọng.
* **Màu nền**: Trắng (**#FFFFFF**) và xám nhạt (**#F5F5F5**) - Tạo không gian thoáng đãng, dễ đọc.
* **Màu văn bản**: Đen (**#212121**) cho nội dung chính, xám đậm (**#757575**) cho nội dung phụ.
* **Màu thành công**: Xanh lá (**#4CAF50**) - Cho thông báo thành công.
* **Màu cảnh báo**: Đỏ (**#F44336**) - Cho thông báo lỗi hoặc cảnh báo.

### 3.2. Typography

* **Font chữ**: Sans-serif như **Roboto** hoặc **Open Sans**.
* **Kích thước**:
    * Tiêu đề lớn: 24-32px.
    * Tiêu đề phụ: 18-24px.
    * Nội dung: 14-16px.
    * Chú thích: 12px.

### 3.3. Bố cục chung

* **Responsive design**: Đảm bảo giao diện hoạt động tốt trên desktop, tablet và mobile.
* **Grid system**: Sử dụng grid 12 cột để bố trí các thành phần.
* **Spacing**: Sử dụng padding và margin nhất quán (8px, 16px, 24px, 32px).
* **Card-based design**: Sử dụng card để nhóm thông tin liên quan.
* **Consistent navigation**: Đảm bảo điều hướng nhất quán giữa các màn hình.

## 4. Tương tác và luồng người dùng

### 4.1. Luồng chính cho doanh nghiệp

1.  Truy cập Landing Page → Click **"Đăng ký dịch vụ"**.
2.  Điền form đăng ký → Click **"Tiếp tục"**.
3.  Điền form yêu cầu dịch vụ → Click **"Tiếp tục"**.
4.  Xác nhận thông tin → Click **"Xác nhận và gửi"**.
5.  Xem màn hình chờ phản hồi.
6.  Nhận email thông báo → Click link trong email.
7.  Xem đề xuất từ Lab Management → Phản hồi.
8.  Nếu đồng ý → Chuyển đến màn hình ký kết hợp đồng.
9.  Ký kết hợp đồng → Thiết lập tài khoản.
10. Hoàn tất → Truy cập dashboard.

### 4.2. Luồng chính cho Lab Management

1.  Đăng nhập vào dashboard.
2.  Xem danh sách yêu cầu mới → Click **"Xem chi tiết"**.
3.  Phân tích yêu cầu và điền form đánh giá.
4.  Click **"Gửi đề xuất cho doanh nghiệp"**.
5.  Chờ phản hồi từ doanh nghiệp.
6.  Nếu doanh nghiệp đồng ý → Chuẩn bị hợp đồng.
7.  Sau khi doanh nghiệp ký kết → Thiết lập tài khoản và quyền truy cập.

### 4.3. Các tương tác đặc biệt

* **Form validation**: Hiển thị thông báo lỗi ngay dưới trường nhập liệu không hợp lệ.
* **Tooltips**: Cung cấp thông tin bổ sung khi hover vào icon thông tin.
* **Drag & drop**: Cho phép kéo thả file khi upload tài liệu.
* **Auto-save**: Tự động lưu form khi người dùng nhập liệu (cho Lab Management).
* **Notifications**: Hiển thị thông báo khi có cập nhật mới.

---
