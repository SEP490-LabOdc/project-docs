# Main Flow 2: Talent Pool Management - Hướng Dẫn Nhiệm Vụ Chi Tiết

## 📋 Tổng Quan Flow

**Mục tiêu**: Quản lý toàn bộ quy trình xây dựng và đào tạo talent pool từ tuyển dụng, đào tạo đến phân bổ vào các ODC team.

**Các bước chính**:
1. Tuyển dụng và sàng lọc ứng viên (Student/Fresh graduate)
2. Đánh giá kỹ năng và phân loại level
3. Thiết kế và thực hiện chương trình đào tạo
4. Theo dõi tiến độ học tập và đánh giá
5. Quản lý talent pool và matching với yêu cầu dự án
6. Phân bổ talent vào ODC teams

**Microservices liên quan**:
- Talent Service (chính)
- User Service (authentication)
- Learning Service (đào tạo)
- Assessment Service (đánh giá)
- Notification Service (thông báo)
- Analytics Service (báo cáo)

**Thời gian ước tính**: 3-4 tuần
**Độ ưu tiên**: Cao (Core business flow)

---

## 🔧 NHIỆM VỤ BACKEND TEAM

### 1. Talent Service - Core Implementation

#### 1.1 Database Design & Models
**Nhiệm vụ**:
- Thiết kế bảng `talent_applications` để lưu đơn ứng tuyển
- Thiết kế bảng `talent_profiles` để lưu thông tin chi tiết talent
- Thiết kế bảng `skill_assessments` để lưu kết quả đánh giá kỹ năng
- Thiết kế bảng `training_progress` để theo dõi tiến độ đào tạo
- Thiết kế bảng `talent_assignments` để quản lý phân bổ vào teams
- Tạo các enum cho status, skill levels, training status

**Các trường cần có**:
- **talent_applications**: full_name, email, phone, university, major, graduation_year, gpa, cv_url, cover_letter, preferred_technologies, status, applied_at, reviewed_at, reviewer_id, interview_notes
- **talent_profiles**: user_id (FK), full_name, email, phone, university, major, graduation_year, gpa, current_level, skills (JSON), certifications, portfolio_url, github_url, linkedin_url, status, created_at, updated_at
- **skill_assessments**: talent_id (FK), skill_name, assessment_type, score, max_score, assessed_at, assessor_id, notes
- **training_progress**: talent_id (FK), course_id, enrollment_date, completion_date, progress_percentage, current_module, quiz_scores (JSON), project_scores (JSON), status
- **talent_assignments**: talent_id (FK), team_id, project_id, role, start_date, end_date, status, performance_rating

#### 1.2 Repository Layer
**Nhiệm vụ**:
- **TalentApplicationRepository**:
  - `findByStatus(ApplicationStatus status)` - Lấy đơn theo trạng thái
  - `findByReviewerId(Long reviewerId)` - Lấy đơn được assign cho reviewer
  - `findByEmail(String email)` - Kiểm tra email đã apply
  - `findByUniversityAndMajor()` - Filter theo trường và ngành
  - `findRecentApplications()` - Lấy đơn gần đây

- **TalentProfileRepository**:
  - `findByCurrentLevel(TalentLevel level)` - Lấy talent theo level
  - `findBySkillsContaining(String skill)` - Tìm theo kỹ năng
  - `findAvailableForAssignment()` - Lấy talent available
  - `findByStatus(TalentStatus status)` - Lấy theo trạng thái

- **SkillAssessmentRepository**:
  - `findByTalentId(Long talentId)` - Lấy đánh giá của talent
  - `findBySkillName(String skillName)` - Lấy đánh giá theo skill
  - `findLatestAssessmentByTalentAndSkill()` - Lấy đánh giá mới nhất

#### 1.3 Service Layer
**Nhiệm vụ**:
- **TalentApplicationService**:
  - `submitApplication()` - Xử lý đơn ứng tuyển mới
  - `reviewApplication()` - Review và approve/reject đơn
  - `scheduleInterview()` - Lên lịch phỏng vấn
  - `conductInterview()` - Ghi nhận kết quả phỏng vấn
  - `approveApplication()` - Approve và tạo talent profile
  - `rejectApplication()` - Reject với feedback
  - `getApplicationsByReviewer()` - Lấy đơn theo reviewer

- **TalentProfileService**:
  - `createTalentProfile()` - Tạo profile sau khi approve
  - `updateTalentProfile()` - Cập nhật thông tin talent
  - `assessTalentSkills()` - Đánh giá kỹ năng
  - `updateSkillLevel()` - Cập nhật level kỹ năng
  - `searchTalents()` - Tìm kiếm talent theo criteria
  - `getTalentRecommendations()` - Gợi ý talent cho project

- **TalentAssignmentService**:
  - `assignTalentToTeam()` - Phân bổ talent vào team
  - `reassignTalent()` - Chuyển talent sang team khác
  - `completeTalentAssignment()` - Hoàn thành assignment
  - `evaluatePerformance()` - Đánh giá performance
  - `getTeamTalents()` - Lấy danh sách talent trong team

#### 1.4 Controller Layer
**Nhiệm vụ**:
- **TalentApplicationController**:
  - `POST /api/v1/talent/applications` - Submit đơn ứng tuyển
  - `GET /api/v1/talent/applications` - Lấy danh sách đơn (HR)
  - `GET /api/v1/talent/applications/{id}` - Chi tiết đơn
  - `PUT /api/v1/talent/applications/{id}/review` - Review đơn
  - `PUT /api/v1/talent/applications/{id}/interview` - Schedule interview
  - `PUT /api/v1/talent/applications/{id}/approve` - Approve đơn
  - `PUT /api/v1/talent/applications/{id}/reject` - Reject đơn

- **TalentProfileController**:
  - `GET /api/v1/talents` - Lấy danh sách talent với filter
  - `GET /api/v1/talents/{id}` - Chi tiết talent
  - `PUT /api/v1/talents/{id}` - Cập nhật talent
  - `POST /api/v1/talents/{id}/assessments` - Thêm đánh giá kỹ năng
  - `GET /api/v1/talents/{id}/assessments` - Lấy lịch sử đánh giá
  - `GET /api/v1/talents/search` - Tìm kiếm talent
  - `GET /api/v1/talents/recommendations` - Gợi ý talent cho project

- **TalentAssignmentController**:
  - `POST /api/v1/talent/assignments` - Tạo assignment mới
  - `GET /api/v1/talent/assignments` - Lấy danh sách assignments
  - `PUT /api/v1/talent/assignments/{id}` - Cập nhật assignment
  - `DELETE /api/v1/talent/assignments/{id}` - Hủy assignment
  - `GET /api/v1/teams/{teamId}/talents` - Lấy talent trong team

#### 1.5 Integration Tasks
**Nhiệm vụ**:
- **Với Learning Service**:
  - Enroll talent vào training programs
  - Theo dõi training progress
  - Sync completion status
  - Cập nhật skill levels sau training

- **Với Assessment Service**:
  - Tạo assessment sessions
  - Lấy kết quả assessments
  - Cập nhật skill scores
  - Generate skill reports

- **Với User Service**:
  - Tạo user accounts cho talents
  - Quản lý roles và permissions
  - Sync profile information

- **Với Notification Service**:
  - Thông báo application status updates
  - Reminder cho interviews
  - Thông báo training assignments
  - Alert cho performance issues

#### 1.6 Business Logic & Rules
**Nhiệm vụ**:
- Implement skill matching algorithm
- Calculate talent readiness scores
- Auto-assign talents based on criteria
- Performance tracking và rating system
- Talent progression path logic
- Availability management

---

## 🎨 NHIỆM VỤ FRONTEND TEAM

### 1. Talent Application Portal

#### 1.1 Application Form
**Nhiệm vụ**:
- Tạo form ứng tuyển với các section:
  - **Thông tin cá nhân**: Họ tên, email, phone, địa chỉ
  - **Thông tin học vấn**: Trường, ngành, năm tốt nghiệp, GPA
  - **Kỹ năng và kinh nghiệm**: Programming languages, frameworks, projects
  - **Documents**: Upload CV, portfolio, certificates
  - **Preferences**: Preferred technologies, career goals

**Features cần có**:
- File upload với drag & drop
- Auto-save draft functionality
- Progress indicator
- Skill tags với autocomplete
- Portfolio showcase section

#### 1.2 Application Tracking
**Nhiệm vụ**:
- Dashboard cho candidates track application status
- Timeline hiển thị các stages
- Interview scheduling interface
- Feedback display từ reviewers
- Notification center

### 2. HR Management Dashboard

#### 2.1 Application Review Interface
**Nhiệm vụ**:
- Danh sách applications với filtering:
  - Status (New, Under Review, Interview Scheduled, etc.)
  - University, Major, GPA range
  - Applied date range
  - Assigned reviewer
- Bulk actions (assign reviewer, schedule interviews)
- Application detail view với:
  - Candidate information
  - CV viewer
  - Review form
  - Interview notes
  - Decision buttons (Approve/Reject)

#### 2.2 Interview Management
**Nhiệm vụ**:
- Calendar integration cho interview scheduling
- Interview room booking
- Interviewer assignment
- Interview feedback forms
- Video call integration

### 3. Talent Pool Management

#### 3.1 Talent Directory
**Nhiệm vụ**:
- Grid/List view của tất cả talents
- Advanced filtering:
  - Skills và skill levels
  - Availability status
  - Current assignments
  - Performance ratings
  - Training completion
- Search functionality với multiple criteria
- Export functionality

#### 3.2 Talent Profile Pages
**Nhiệm vụ**:
- Comprehensive talent profile với:
  - Personal information
  - Skills matrix với visual indicators
  - Training history và progress
  - Project assignment history
  - Performance metrics
  - Assessment results
- Edit profile functionality
- Skill assessment scheduling

#### 3.3 Team Assignment Interface
**Nhiệm vụ**:
- Drag & drop interface cho talent assignment
- Team composition view
- Skill gap analysis
- Talent recommendations cho projects
- Assignment history tracking

### 4. Analytics & Reporting

#### 4.1 Talent Analytics Dashboard
**Nhiệm vụ**:
- Key metrics visualization:
  - Total talents by level
  - Skill distribution
  - Training completion rates
  - Assignment utilization
  - Performance trends
- Interactive charts và graphs
- Date range filtering
- Export reports functionality

#### 4.2 Recruitment Analytics
**Nhiệm vụ**:
- Application funnel analysis
- Source tracking (university, referrals, etc.)
- Time-to-hire metrics
- Rejection reason analysis
- Interviewer performance metrics

---

## 📱 NHIỆM VỤ MOBILE TEAM

### 1. Talent Mobile App

#### 1.1 Application Submission
**Nhiệm vụ**:
- Mobile-optimized application form
- Camera integration cho document scanning
- Voice-to-text cho cover letter
- Offline draft saving
- Push notifications cho status updates

#### 1.2 Talent Dashboard
**Nhiệm vụ**:
- Personal dashboard với:
  - Current assignments
  - Training progress
  - Upcoming assessments
  - Performance metrics
- Quick actions (update availability, view schedule)
- Notification center

#### 1.3 Learning Integration
**Nhiệm vụ**:
- Access training materials
- Complete quizzes và assignments
- Track learning progress
- Download certificates
- Offline content access

### 2. HR Mobile Interface

#### 2.1 Application Review
**Nhiệm vụ**:
- Mobile-friendly application review
- Quick approve/reject actions
- Interview scheduling
- Push notifications cho new applications

#### 2.2 Talent Search
**Nhiệm vụ**:
- Quick talent search
- Filter by availability
- Contact talents directly
- View talent profiles

---

## 🔄 INTEGRATION & WORKFLOWS

### 1. Application to Talent Pipeline
**Workflow**:
1. Candidate submits application
2. Auto-assignment to reviewer based on workload
3. Initial screening và CV review
4. Interview scheduling
5. Interview conduct và feedback
6. Final decision (Approve/Reject)
7. If approved: Create talent profile và user account
8. Enroll in onboarding training program

### 2. Skill Assessment Workflow
**Workflow**:
1. Schedule assessment session
2. Conduct technical assessment
3. Record scores và feedback
4. Update talent skill matrix
5. Recommend training if needed
6. Update talent level if qualified

### 3. Team Assignment Workflow
**Workflow**:
1. Project manager requests talents với specific skills
2. System suggests matching talents
3. Review talent availability
4. Assign talents to team
5. Notify talents và team leads
6. Track assignment performance
7. Complete assignment và update records

---

## 📋 CHECKLIST TRIỂN KHAI

### Backend Tasks:
- [ ] Thiết kế database schema cho talent management
- [ ] Implement talent application processing
- [ ] Implement skill assessment system
- [ ] Implement talent search và matching algorithms
- [ ] Setup integration với Learning Service
- [ ] Setup integration với Assessment Service
- [ ] Implement assignment management
- [ ] Setup notification workflows
- [ ] Implement analytics và reporting APIs
- [ ] Setup file upload cho CVs và documents
- [ ] Implement security và data privacy
- [ ] Viết comprehensive tests

### Frontend Tasks:
- [ ] Thiết kế UI cho application form
- [ ] Implement application tracking interface
- [ ] Tạo HR dashboard cho application review
- [ ] Implement talent directory với advanced search
- [ ] Tạo talent profile management
- [ ] Implement team assignment interface
- [ ] Tạo analytics dashboard
- [ ] Setup file upload và preview
- [ ] Implement responsive design
- [ ] Setup real-time notifications
- [ ] Viết unit tests cho components

### Mobile Tasks:
- [ ] Thiết kế mobile application flow
- [ ] Implement document scanning
- [ ] Tạo talent mobile dashboard
- [ ] Setup push notifications
- [ ] Implement offline functionality
- [ ] Integrate với learning modules
- [ ] Setup biometric authentication
- [ ] Implement deep linking
- [ ] Optimize performance cho low-end devices
- [ ] Viết mobile-specific tests

### Integration Tasks:
- [ ] Test complete application-to-talent pipeline
- [ ] Verify skill assessment integration
- [ ] Test team assignment workflows
- [ ] Verify notification delivery
- [ ] Test file upload và storage
- [ ] Performance testing với large datasets
- [ ] Security testing cho sensitive data
- [ ] User acceptance testing

---

## 🎯 DEFINITION OF DONE

### Backend:
- [ ] All APIs documented và tested
- [ ] Skill matching algorithm working accurately
- [ ] File upload và storage secure
- [ ] Integration với external services stable
- [ ] Performance requirements met
- [ ] Data privacy compliance
- [ ] Comprehensive error handling

### Frontend:
- [ ] Intuitive user experience cho all user types
- [ ] Advanced search và filtering working
- [ ] Real-time updates functioning
- [ ] File upload với progress indicators
- [ ] Responsive design across devices
- [ ] Accessibility standards met
- [ ] Performance optimized

### Mobile:
- [ ] Smooth application submission flow
- [ ] Document scanning working reliably
- [ ] Offline functionality stable
- [ ] Push notifications delivered correctly
- [ ] App performance optimized
- [ ] Platform-specific guidelines followed

### Integration:
- [ ] End-to-end workflows functioning
- [ ] Data consistency across services
- [ ] Error scenarios handled gracefully
- [ ] Performance under load acceptable
- [ ] Security requirements satisfied
- [ ] User acceptance criteria met

---

**Lưu ý quan trọng**:
- Talent pool là tài sản quan trọng nhất của platform
- Cần đảm bảo data quality và consistency
- Privacy và security của talent data là ưu tiên hàng đầu
- Performance cần được optimize cho large datasets
- User experience phải smooth cho cả talents và HR team