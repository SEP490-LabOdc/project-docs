# Main Flow 1: Business Onboarding - Hướng Dẫn Nhiệm Vụ Chi Tiết

## 📋 Tổng Quan Flow

**Mục tiêu**: Xử lý quy trình onboarding doanh nghiệp mới từ đăng ký đến ký kết hợp đồng và thiết lập tài khoản.

**Các bước chính**:
1. Business đăng ký sử dụng dịch vụ qua website/mobile
2. Lab Management tiếp nhận và phân tích yêu cầu
3. Thỏa thuận về quy mô team, kỹ năng cần thiết, thời gian và chi phí
4. Ký kết hợp đồng điện tử và thiết lập tài khoản trên platform

**Microservices liên quan**:
- Business Service (chính)
- User Service (authentication)
- Contract Service (tạo hợp đồng)
- Notification Service (thông báo)

**Thời gian ước tính**: 2-3 tuần
**Độ ưu tiên**: Cao (Core business flow)

---

## 🔧 NHIỆM VỤ BACKEND TEAM

### 1. Business Service - Core Implementation

#### 1.1 Database Design & Models
**Nhiệm vụ**:
- Thiết kế bảng `business_registrations` để lưu thông tin đăng ký doanh nghiệp
- Thiết kế bảng `business_profiles` để lưu profile doanh nghiệp sau khi được approve
- Tạo các enum cho status (PENDING, UNDER_REVIEW, APPROVED, REJECTED, CONTRACT_SENT, COMPLETED)
- Thiết kế relationship giữa các bảng

**Các trường cần có**:
- **business_registrations**: company_name, contact_email, contact_phone, contact_person, industry, company_size, project_description, required_skills, estimated_team_size, project_duration, estimated_budget, status, created_at, updated_at, assigned_manager_id, reviewed_at, manager_notes
- **business_profiles**: user_id (FK), company_name, tax_code, address, website, description, company_size, industry, status

#### 1.2 Repository Layer
**Nhiệm vụ**:
- Tạo BusinessRegistrationRepository với các method:
  - `findByStatus(RegistrationStatus status)` - Lấy danh sách theo trạng thái
  - `findByAssignedManagerId(Long managerId)` - Lấy danh sách được phân công cho manager
  - `findByContactEmail(String email)` - Kiểm tra email đã tồn tại
  - `findRecentByStatus()` - Lấy danh sách gần đây theo status
- Tạo BusinessProfileRepository với các method cơ bản

#### 1.3 Service Layer
**Nhiệm vụ**:
- **BusinessRegistrationService**:
  - `submitRegistration()` - Xử lý đăng ký mới, validate email unique, gửi notification
  - `assignToManager()` - Phân công cho manager, cập nhật status
  - `updateManagerNotes()` - Cập nhật ghi chú của manager
  - `approveRegistration()` - Phê duyệt đăng ký, trigger tạo contract
  - `rejectRegistration()` - Từ chối đăng ký với lý do
  - `getPendingRegistrations()` - Lấy danh sách chờ xử lý
  - `getRegistrationsByManager()` - Lấy danh sách theo manager

- **BusinessProfileService**:
  - `createBusinessProfile()` - Tạo profile sau khi approve
  - `updateBusinessProfile()` - Cập nhật thông tin profile
  - `getBusinessProfile()` - Lấy thông tin profile

#### 1.4 Controller Layer
**Nhiệm vụ**:
- **BusinessRegistrationController**:
  - `POST /api/v1/business/registrations` - Submit đăng ký mới
  - `GET /api/v1/business/registrations/pending` - Lấy danh sách chờ xử lý (Lab Manager)
  - `GET /api/v1/business/registrations/{id}` - Lấy chi tiết đăng ký
  - `PUT /api/v1/business/registrations/{id}/assign` - Phân công manager
  - `PUT /api/v1/business/registrations/{id}/approve` - Phê duyệt
  - `PUT /api/v1/business/registrations/{id}/reject` - Từ chối
  - `PUT /api/v1/business/registrations/{id}/notes` - Cập nhật ghi chú

- **BusinessProfileController**:
  - `GET /api/v1/business/profile` - Lấy profile của business đang login
  - `PUT /api/v1/business/profile` - Cập nhật profile

#### 1.5 Integration Tasks
**Nhiệm vụ**:
- **Với Contract Service**:
  - Gọi API tạo contract khi approve registration
  - Xử lý callback khi contract được ký
  - Cập nhật status registration thành COMPLETED

- **Với User Service**:
  - Gọi API tạo user account cho business
  - Liên kết user_id với business_profile
  - Xử lý authentication flow

- **Với Notification Service**:
  - Gửi email thông báo đăng ký mới cho Lab Management
  - Gửi email thông báo phân công cho Manager
  - Gửi email thông báo approve/reject cho Business
  - Gửi email hướng dẫn setup account

#### 1.6 Validation & Security
**Nhiệm vụ**:
- Validate input data (email format, phone format, required fields)
- Kiểm tra duplicate email registration
- Implement authorization (chỉ Lab Manager mới được approve)
- Implement rate limiting cho registration endpoint
- Log tất cả các thao tác quan trọng

#### 1.7 Error Handling
**Nhiệm vụ**:
- Xử lý lỗi email đã tồn tại
- Xử lý lỗi khi gọi external services
- Xử lý lỗi validation
- Trả về error response chuẩn theo format API

---

## 🎨 NHIỆM VỤ FRONTEND TEAM

### 1. Business Registration Form (Web)

#### 1.1 Registration Form Component
**Nhiệm vụ**:
- Tạo form đăng ký với các section:
  - **Thông tin công ty**: Tên công ty, ngành nghề, quy mô
  - **Thông tin liên hệ**: Người liên hệ, email, số điện thoại
  - **Yêu cầu dự án**: Mô tả dự án, kỹ năng cần thiết, quy mô team, thời gian, ngân sách

**Validation cần có**:
- Email format validation
- Phone number format validation
- Required field validation
- Team size minimum validation
- Budget range validation

**UX Requirements**:
- Form được chia thành các section rõ ràng
- Progress indicator cho multi-step form
- Auto-save draft functionality
- Responsive design cho mobile
- Loading states khi submit

#### 1.2 Success/Error Handling
**Nhiệm vụ**:
- Hiển thị success message sau khi submit thành công
- Hiển thị error message khi có lỗi
- Redirect đến trang thank you với thông tin tracking
- Gửi email confirmation cho business

#### 1.3 API Integration
**Nhiệm vụ**:
- Tạo API service cho business registration
- Handle API errors và hiển thị user-friendly messages
- Implement retry mechanism cho failed requests
- Cache form data trong localStorage

### 2. Lab Management Dashboard

#### 2.1 Registration List View
**Nhiệm vụ**:
- Hiển thị danh sách đăng ký với các thông tin:
  - Tên công ty, người liên hệ, ngành nghề
  - Status với color coding
  - Ngày đăng ký
  - Manager được assign (nếu có)
- Implement filtering theo status, ngày, manager
- Implement sorting theo các trường
- Implement pagination

#### 2.2 Registration Detail Modal/Page
**Nhiệm vụ**:
- Hiển thị đầy đủ thông tin đăng ký
- Form để assign manager
- Form để approve/reject với lý do
- Text area để ghi chú
- History log các thao tác

#### 2.3 Manager Assignment
**Nhiệm vụ**:
- Dropdown list các manager available
- Bulk assignment cho multiple registrations
- Notification khi assign thành công

#### 2.4 Approval/Rejection Flow
**Nhiệm vụ**:
- Form approve với các trường:
  - Approved team size
  - Approved duration
  - Approved budget
  - Contract terms
  - Approval notes
- Form reject với required reason
- Confirmation dialog trước khi approve/reject

### 3. Business Portal

#### 3.1 Registration Status Tracking
**Nhiệm vụ**:
- Trang tracking status của registration
- Timeline hiển thị các bước đã hoàn thành
- Notification khi có update status
- Download contract khi available

#### 3.2 Business Profile Management
**Nhiệm vụ**:
- Form cập nhật thông tin công ty
- Upload company logo
- Manage company documents
- View contract history

---

## 📱 NHIỆM VỤ MOBILE TEAM

### 1. Business Registration Flow

#### 1.1 Mobile Registration Form
**Nhiệm vụ**:
- Thiết kế form registration tối ưu cho mobile
- Multi-step wizard với progress indicator
- Camera integration để chụp business license
- Location picker cho company address
- File upload cho company documents

#### 1.2 Mobile-Specific Features
**Nhiệm vụ**:
- Push notification khi có update status
- Offline mode để save draft
- Biometric authentication setup
- QR code scanning cho quick registration

### 2. Business Dashboard Mobile

#### 2.1 Dashboard Overview
**Nhiệm vụ**:
- Card-based layout hiển thị registration status
- Quick actions (view details, contact support)
- Notification center
- Profile quick edit

#### 2.2 Document Management
**Nhiệm vụ**:
- View và download contracts
- Upload additional documents
- Digital signature integration
- Document sharing với team members

---

## 🔄 INTEGRATION & TESTING

### 1. End-to-End Testing Scenarios
**Nhiệm vụ**:
- Test complete registration flow từ submit đến approve
- Test email notification delivery
- Test contract generation và signing
- Test error scenarios (duplicate email, service down)
- Test mobile responsiveness

### 2. Performance Requirements
**Nhiệm vụ**:
- Registration form load time < 2 seconds
- Form submission response time < 5 seconds
- Dashboard load time < 3 seconds
- Support 100 concurrent registrations

### 3. Security Testing
**Nhiệm vụ**:
- Test input validation và sanitization
- Test authorization (chỉ Lab Manager approve được)
- Test rate limiting
- Test data encryption

---

## 📋 CHECKLIST TRIỂN KHAI

### Backend Tasks:
- [ ] Thiết kế database schema và tạo migration
- [ ] Implement entities và repositories
- [ ] Implement business logic trong service layer
- [ ] Tạo REST controllers với validation
- [ ] Setup integration với Contract Service
- [ ] Setup integration với User Service
- [ ] Setup integration với Notification Service
- [ ] Implement error handling và logging
- [ ] Viết unit tests và integration tests
- [ ] Setup API documentation (Swagger)
- [ ] Configure security và authorization
- [ ] Setup monitoring và alerting

### Frontend Tasks:
- [ ] Thiết kế UI/UX cho registration form
- [ ] Implement registration form với validation
- [ ] Implement success/error handling
- [ ] Setup API integration
- [ ] Tạo lab management dashboard
- [ ] Implement registration list với filtering/sorting
- [ ] Tạo registration detail modal
- [ ] Implement manager assignment flow
- [ ] Implement approval/rejection flow
- [ ] Setup routing và navigation
- [ ] Implement responsive design
- [ ] Viết unit tests cho components
- [ ] Setup error boundary và error tracking

### Mobile Tasks:
- [ ] Thiết kế mobile UI cho registration
- [ ] Implement multi-step registration wizard
- [ ] Setup camera và file upload
- [ ] Implement push notifications
- [ ] Setup offline mode
- [ ] Implement business dashboard
- [ ] Setup document management
- [ ] Implement biometric authentication
- [ ] Setup deep linking
- [ ] Viết unit tests và UI tests

### Integration Tasks:
- [ ] Test end-to-end registration flow
- [ ] Verify email notification delivery
- [ ] Test contract generation integration
- [ ] Test user account creation
- [ ] Performance testing
- [ ] Security testing
- [ ] User acceptance testing
- [ ] Load testing

---

## 🎯 DEFINITION OF DONE

### Backend:
- [ ] Tất cả APIs hoạt động đúng theo spec
- [ ] Unit test coverage >= 80%
- [ ] Integration tests pass
- [ ] API documentation hoàn chỉnh
- [ ] Error handling đầy đủ
- [ ] Logging và monitoring setup
- [ ] Security review passed

### Frontend:
- [ ] UI match với design mockups
- [ ] Responsive trên tất cả devices
- [ ] Form validation hoạt động đúng
- [ ] Error handling user-friendly
- [ ] Performance requirements đạt
- [ ] Accessibility standards đạt
- [ ] Cross-browser testing passed

### Mobile:
- [ ] App hoạt động trên iOS và Android
- [ ] UI/UX tối ưu cho mobile
- [ ] Push notifications hoạt động
- [ ] Offline mode hoạt động
- [ ] App store review guidelines đạt
- [ ] Performance trên low-end devices

### Integration:
- [ ] End-to-end flow hoạt động hoàn chỉnh
- [ ] Tất cả external integrations stable
- [ ] Error scenarios được handle đúng
- [ ] Performance requirements đạt
- [ ] Security requirements đạt
- [ ] User acceptance criteria đạt

---

**Lưu ý quan trọng**:
- Đây là flow cơ bản nhất của hệ thống, cần ưu tiên cao
- Cần coordinate chặt chẽ giữa các team để đảm bảo integration smooth
- Testing cần được thực hiện liên tục trong quá trình development
- Documentation cần được update real-time