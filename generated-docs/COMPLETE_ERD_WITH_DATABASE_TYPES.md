# COMPLETE ERD WITH DATABASE TYPES

## Tổng quan kiến trúc Database

### Database Distribution:
- **PostgreSQL**: Core business data, relationships, transactions
- **MongoDB**: Unstructured data, logs, flexible schemas
- **Elasticsearch**: Search indexes, analytics, real-time queries

---

## 1. CORE SYSTEM - PostgreSQL

### 1.1. Authentication & Authorization

```
┌─────────────────────────────┐       ┌─────────────────────────────┐
│           ROLES             │       │           USERS             │
│        [PostgreSQL]         │       │        [PostgreSQL]         │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │
│  name (VARCHAR)             │       │  email (VARCHAR) [UNIQUE]   │
│  description (TEXT)         │       │  password_hash (VARCHAR)    │
│  permissions (JSONB)        │       │  full_name (VARCHAR)        │
│  created_at (TIMESTAMP)     │       │  phone (VARCHAR)            │
│  updated_at (TIMESTAMP)     │ 1   * │ *role_id (UUID) [FK]        │
└─────────────────────────────┘───────│  status (VARCHAR)           │
                                      │  created_at (TIMESTAMP)     │
                                      │  updated_at (TIMESTAMP)     │
                                      └─────────────────────────────┘

**Ví dụ ROLES:**
- LAB_ADMIN: Quản trị viên Lab (toàn quyền)
- LAB_MANAGER: Quản lý Lab (quản lý talent, team)
- BUSINESS_OWNER: Chủ doanh nghiệp (quản lý công ty)
- BUSINESS_MANAGER: Quản lý doanh nghiệp (quản lý dự án)
- BUSINESS_CONTACT: Liên hệ doanh nghiệp (xem báo cáo)
- TALENT: Nhân tài trong talent pool
- MENTOR: Mentor/Senior hướng dẫn
- TEAM_LEAD: Trưởng nhóm ODC
- FREELANCER: Freelancer trên marketplace
- INSTRUCTOR: Giảng viên nền tảng học tập
- STUDENT: Học viên tham gia khóa học
```

---

## 2. FLOW 1: Business Registration & Onboarding - PostgreSQL

```
┌─────────────────────────────┐       ┌─────────────────────────────┐
│   BUSINESS_REGISTRATIONS    │       │         BUSINESSES          │
│        [PostgreSQL]         │       │        [PostgreSQL]         │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │
│  business_name (VARCHAR)    │       │ *registration_id (UUID) [FK]│
│  business_email (VARCHAR)   │       │  name (VARCHAR)             │
│  contact_person_name (VAR)  │       │  email (VARCHAR)            │
│  contact_person_email (VAR) │       │  phone (VARCHAR)            │
│  industry (VARCHAR)         │       │  address (TEXT)             │
│  company_size (VARCHAR)     │       │  industry (VARCHAR)         │
│  registration_documents     │ 1   1 │  verification_status (VAR)  │
│  status (VARCHAR)           │───────│  subscription_plan (VAR)    │
│  submitted_at (TIMESTAMP)   │       │  lab_setup_completed (BOOL) │
│  reviewed_by (UUID) [FK]    │       │  onboarding_completed (BOOL)│
│  created_at (TIMESTAMP)     │       │  status (VARCHAR)           │
│  updated_at (TIMESTAMP)     │       │  created_at (TIMESTAMP)     │
└─────────────────────────────┘       │  updated_at (TIMESTAMP)     │
              │                       └─────────────────────────────┘
              │                                       │
              │                                       │ 1
              │                                       │
              │                                       │ *
              │                       ┌─────────────────────────────┐
              │                       │      BUSINESS_CONTACTS      │
              │                       │        [PostgreSQL]         │
              │                       ├─────────────────────────────┤
              │                       │ *id (UUID) [PK]             │
              │                       │ *business_id (UUID) [FK]    │
              │                       │ *user_id (UUID) [FK]        │
              │                       │  position (VARCHAR)         │
              │                       │  is_primary (BOOLEAN)       │
              │                       │  created_at (TIMESTAMP)     │
              │                       │  updated_at (TIMESTAMP)     │
              │                       └─────────────────────────────┘
              │                                       │
              │                                       │
              │                       ┌─────────────────────────────┐
              │                       │           USERS             │
              │                       │        [PostgreSQL]         │
              │                       │        (contact user)       │
              │                       └─────────────────────────────┘
              │
              │
┌─────────────────────────────┐
│           USERS             │
│        [PostgreSQL]         │
│         (reviewer)          │
└─────────────────────────────┘


┌─────────────────────────────┐       ┌─────────────────────────────┐
│   BUSINESS_REQUIREMENTS     │       │     REQUIREMENT_SKILLS      │
│        [PostgreSQL]         │       │        [PostgreSQL]         │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │
│ *business_id (UUID) [FK]    │       │ *requirement_id (UUID) [FK] │
│  title (VARCHAR)            │ 1   * │ *skill_id (UUID) [FK]       │
│  description (TEXT)         │───────│  experience_level (VARCHAR) │
│  team_size (INT)            │       │  created_at (TIMESTAMP)     │
│  duration (VARCHAR)         │       └─────────────────────────────┘
│  contract_type (VARCHAR)    │                       │
│  status (VARCHAR)           │                       │
│  created_at (TIMESTAMP)     │                       │
│  updated_at (TIMESTAMP)     │       ┌─────────────────────────────┐
└─────────────────────────────┘       │           SKILLS            │
              │                       │        [PostgreSQL]         │
              │                       ├─────────────────────────────┤
              │                       │ *id (UUID) [PK]             │
              │                       │  name (VARCHAR) [UNIQUE]    │

**Ví dụ BUSINESS_REQUIREMENTS contract_type:**
- MONTHLY_RENTAL: Thuê theo tháng
- PROJECT_BASED: Theo dự án
- FULL_TIME_HIRE: Tuyển dụng chính thức
- PART_TIME: Bán thời gian
- CONSULTING: Tư vấn

**Ví dụ BUSINESS_REQUIREMENTS status:**
- DRAFT: Bản nháp
- SUBMITTED: Đã gửi
- UNDER_REVIEW: Đang xem xét
- APPROVED: Đã phê duyệt
- IN_PROGRESS: Đang thực hiện
- COMPLETED: Hoàn thành
- CANCELLED: Đã hủy

**Ví dụ SKILLS name:**
- Java, Python, JavaScript, TypeScript, C#, PHP
- React, Angular, Vue.js, Node.js, Spring Boot
- MySQL, PostgreSQL, MongoDB, Redis
- Docker, Kubernetes, AWS, Azure, GCP
- Agile, Scrum, DevOps, CI/CD
- UI/UX Design, Figma, Adobe XD
- Project Management, Team Leadership
- English Communication, Technical Writing
              │                       │  category (VARCHAR)         │
              │                       │  description (TEXT)         │
              │                       │  created_at (TIMESTAMP)     │
              │                       └─────────────────────────────┘
              │
              │
┌─────────────────────────────┐
│          CONTRACTS          │
│        [PostgreSQL]         │
├─────────────────────────────┤
│ *id (UUID) [PK]             │
│ *business_id (UUID) [FK]    │
│  requirement_id (UUID) [FK] │
│  title (VARCHAR)            │
│  description (TEXT)         │
│  start_date (DATE)          │
│  end_date (DATE)            │
│  contract_type (VARCHAR)    │
│  status (VARCHAR)           │
│  file_path (VARCHAR)        │
│  created_at (TIMESTAMP)     │
│  updated_at (TIMESTAMP)     │
└─────────────────────────────┘
```

---

## 3. FLOW 2: Talent Management & Skill Assessment - PostgreSQL

```
┌─────────────────────────────┐       ┌─────────────────────────────┐
│     TALENT_APPLICATIONS     │       │          TALENTS            │
│        [PostgreSQL]         │       │        [PostgreSQL]         │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │
│  applicant_name (VARCHAR)   │       │ *application_id (UUID) [FK] │
│  email (VARCHAR)            │       │ *user_id (UUID) [FK]        │
│  phone (VARCHAR)            │       │  employee_id (VARCHAR)      │
│  education_level (VARCHAR)  │       │  education_level (VARCHAR)  │
│  university (VARCHAR)       │       │  university (VARCHAR)       │
│  graduation_year (INT)      │       │  graduation_year (INT)      │
│  years_of_experience (INT)  │ 1   1 │  years_of_experience (INT)  │
│  resume_url (VARCHAR)       │───────│  current_position (VARCHAR) │
│  portfolio_url (VARCHAR)    │       │  department (VARCHAR)       │
│  expected_salary_min (DEC)  │       │  manager_id (UUID) [FK]     │
│  expected_salary_max (DEC)  │       │  hire_date (DATE)           │
│  status (VARCHAR)           │       │  salary (DECIMAL)           │
│  submitted_at (TIMESTAMP)   │       │  performance_rating (DEC)   │
│  reviewed_by (UUID) [FK]    │       │  availability (VARCHAR)     │
│  created_at (TIMESTAMP)     │       │  location (VARCHAR)         │
│  updated_at (TIMESTAMP)     │       │  status (VARCHAR)           │
└─────────────────────────────┘       │  created_at (TIMESTAMP)     │
                                      │  updated_at (TIMESTAMP)     │
                                      └─────────────────────────────┘

**Ví dụ TALENT_APPLICATIONS status:**
- SUBMITTED: Đã nộp hồ sơ
- UNDER_REVIEW: Đang xem xét
- INTERVIEW_SCHEDULED: Đã lên lịch phỏng vấn
- INTERVIEWED: Đã phỏng vấn
- SKILL_ASSESSMENT: Đang đánh giá kỹ năng
- APPROVED: Được chấp nhận
- REJECTED: Bị từ chối
- WAITLISTED: Trong danh sách chờ

**Ví dụ TALENTS status:**
- ACTIVE: Đang hoạt động
- INACTIVE: Tạm ngưng
- ON_PROJECT: Đang làm dự án
- AVAILABLE: Sẵn sàng nhận việc
- TRAINING: Đang đào tạo
- SUSPENDED: Bị đình chỉ
- TERMINATED: Đã chấm dứt

**Ví dụ TALENTS availability:**
- FULL_TIME: Toàn thời gian
- PART_TIME: Bán thời gian
- FREELANCE: Tự do
- CONTRACT: Hợp đồng
- REMOTE: Làm việc từ xa
- ON_SITE: Tại văn phòng
- HYBRID: Kết hợp

                                                      │
                                                      │ 1
                                                      │
                                                      │ *
                                      ┌─────────────────────────────┐
                                      │       TALENT_SKILLS         │
                                      │        [PostgreSQL]         │
                                      ├─────────────────────────────┤
                                      │ *id (UUID) [PK]             │
                                      │ *talent_id (UUID) [FK]      │
                                      │ *skill_id (UUID) [FK]       │
                                      │  proficiency_level (INT)    │
                                      │  years_of_experience (DEC)  │
                                      │  verified (BOOLEAN)         │
                                      │  created_at (TIMESTAMP)     │
                                      │  updated_at (TIMESTAMP)     │
                                      └─────────────────────────────┘
                                                      │
                                                      │
                                      ┌─────────────────────────────┐
                                      │           SKILLS            │
                                      │        [PostgreSQL]         │
                                      │         (reference)         │
                                      └─────────────────────────────┘


┌─────────────────────────────┐       ┌─────────────────────────────┐
│     SKILL_ASSESSMENTS       │       │     TRAINING_PROGRESS       │
│        [PostgreSQL]         │       │        [PostgreSQL]         │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │
│ *talent_id (UUID) [FK]      │       │ *talent_id (UUID) [FK]      │
│ *skill_id (UUID) [FK]       │       │  course_id (UUID)           │
│  assessment_type (VARCHAR)  │       │  course_name (VARCHAR)      │
│  assessment_date (DATE)     │       │  training_type (VARCHAR)    │
│  score (DECIMAL)            │       │  status (VARCHAR)           │
│  max_score (DECIMAL)        │       │  start_date (DATE)          │
│  percentage (DECIMAL)       │       │  completion_date (DATE)     │
│  assessor_id (UUID) [FK]    │       │  progress_percentage (DEC)  │
│  assessment_details (JSONB) │       │  final_score (DECIMAL)      │
│  feedback (TEXT)            │       │  certification_obtained     │
│  certification_earned (VAR) │       │  trainer_id (UUID) [FK]     │
│  valid_until (DATE)         │       │  training_hours (DECIMAL)   │
│  created_at (TIMESTAMP)     │       │  cost (DECIMAL)             │
│  updated_at (TIMESTAMP)     │       │  notes (TEXT)               │
└─────────────────────────────┘       │  created_at (TIMESTAMP)     │
                                      │  updated_at (TIMESTAMP)     │
                                      └─────────────────────────────┘
```

---

## 4. FLOW 3: ODC Team Formation - PostgreSQL

```
┌─────────────────────────────┐       ┌─────────────────────────────┐
│  TEAM_FORMATION_REQUESTS    │       │         ODC_TEAMS           │
│        [PostgreSQL]         │       │        [PostgreSQL]         │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │
│ *business_id (UUID) [FK]    │       │ *formation_request_id [FK]  │
│  request_title (VARCHAR)    │       │  name (VARCHAR)             │
│  project_description (TEXT) │       │ *business_id (UUID) [FK]    │
│  team_size (INT)            │       │  contract_id (UUID) [FK]    │
│  required_skills (JSONB)    │ 1   1 │  team_lead_id (UUID) [FK]   │
│  experience_level (VARCHAR) │───────│  workspace_id (UUID) [FK]   │
│  project_duration (VARCHAR) │       │  team_type (VARCHAR)        │
│  start_date (DATE)          │       │  methodology (VARCHAR)      │
│  budget_range (VARCHAR)     │       │  max_capacity (INT)         │
│  work_arrangement (VARCHAR) │       │  current_size (INT)         │
│  location (VARCHAR)         │       │  formation_date (DATE)      │
│  priority (VARCHAR)         │       │  performance_rating (DEC)   │
│  status (VARCHAR)           │       │  budget_allocated (DECIMAL) │
│  submitted_at (TIMESTAMP)   │       │  budget_spent (DECIMAL)     │
│  reviewed_by (UUID) [FK]    │       │  status (VARCHAR)           │
│  created_at (TIMESTAMP)     │       │  created_at (TIMESTAMP)     │
│  updated_at (TIMESTAMP)     │       │  updated_at (TIMESTAMP)     │
└─────────────────────────────┘       └─────────────────────────────┘
                                                      │
                                                      │ 1
                                                      │
                                                      │ *
                                      ┌─────────────────────────────┐
                                      │       TEAM_MEMBERS          │
                                      │        [PostgreSQL]         │
                                      ├─────────────────────────────┤
                                      │ *id (UUID) [PK]             │
                                      │ *team_id (UUID) [FK]        │
                                      │ *talent_id (UUID) [FK]      │
                                      │  role_id (UUID) [FK]        │
                                      │  role (VARCHAR)             │
                                      │  is_team_lead (BOOLEAN)     │
                                      │  allocation_percentage (INT)│
                                      │  hourly_rate (DECIMAL)      │
                                      │  join_date (DATE)           │
                                      │  end_date (DATE)            │
                                      │  performance_rating (DEC)   │
                                      │  status (VARCHAR)           │
                                      │  created_at (TIMESTAMP)     │
                                      │  updated_at (TIMESTAMP)     │
                                      └─────────────────────────────┘

**Ví dụ TEAM_MEMBERS role:**
- DEVELOPER: Lập trình viên
- SENIOR_DEVELOPER: Lập trình viên senior
- TECH_LEAD: Trưởng nhóm kỹ thuật
- PROJECT_MANAGER: Quản lý dự án
- BUSINESS_ANALYST: Phân tích nghiệp vụ
- QA_ENGINEER: Kỹ sư kiểm thử
- DEVOPS_ENGINEER: Kỹ sư DevOps
- UI_UX_DESIGNER: Thiết kế UI/UX
- SCRUM_MASTER: Scrum Master

**Ví dụ TEAM_MEMBERS status:**
- ACTIVE: Đang hoạt động
- INACTIVE: Tạm ngưng
- ON_LEAVE: Đang nghỉ phép
- TRANSFERRED: Đã chuyển team
- COMPLETED: Hoàn thành nhiệm vụ

                                                      │
                                                      │
                                      ┌─────────────────────────────┐
                                      │          TALENTS            │
                                      │        [PostgreSQL]         │
                                      │         (reference)         │
                                      └─────────────────────────────┘


┌─────────────────────────────┐       ┌─────────────────────────────┐
│        TEAM_ROLES           │       │      TEAM_WORKSPACES        │
│        [PostgreSQL]         │       │        [PostgreSQL]         │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │
│  role_name (VARCHAR)        │       │  workspace_name (VARCHAR)   │
│  role_description (TEXT)    │       │  location (VARCHAR)         │
│  required_skills (JSONB)    │       │  capacity (INT)             │
│  experience_level (VARCHAR) │       │  facilities (JSONB)         │
│  responsibilities (TEXT)    │       │  availability_status (VAR)  │
│  is_leadership_role (BOOL)  │       │  cost_per_month (DECIMAL)   │
│  created_at (TIMESTAMP)     │       │  created_at (TIMESTAMP)     │
│  updated_at (TIMESTAMP)     │       │  updated_at (TIMESTAMP)     │
└─────────────────────────────┘       └─────────────────────────────┘
```

---

## 5. FLOW 4: Project Management - PostgreSQL

```
┌─────────────────────────────┐       ┌─────────────────────────────┐
│          PROJECTS           │       │      PROJECT_PHASES         │
│        [PostgreSQL]         │       │        [PostgreSQL]         │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │
│  name (VARCHAR)             │       │ *project_id (UUID) [FK]     │
│ *business_id (UUID) [FK]    │       │  phase_name (VARCHAR)       │
│  team_id (UUID) [FK]        │       │  description (TEXT)         │
│  project_manager_id [FK]    │       │  sequence_order (INT)       │
│  description (TEXT)         │ 1   * │  start_date (DATE)          │
│  objectives (TEXT)          │───────│  end_date (DATE)            │
│  scope (TEXT)               │       │  actual_start_date (DATE)   │
│  deliverables (JSONB)       │       │  actual_end_date (DATE)     │
│  start_date (DATE)          │       │  budget (DECIMAL)           │
│  end_date (DATE)            │       │  actual_cost (DECIMAL)      │
│  budget (DECIMAL)           │       │  progress_percentage (DEC)  │
│  actual_cost (DECIMAL)      │       │  deliverables (JSONB)       │
│  priority (VARCHAR)         │       │  acceptance_criteria (TEXT) │
│  methodology (VARCHAR)      │       │  status (VARCHAR)           │
│  progress_percentage (DEC)  │       │  created_at (TIMESTAMP)     │
│  health_status (VARCHAR)    │       │  updated_at (TIMESTAMP)     │
│  status (VARCHAR)           │       └─────────────────────────────┘
│  created_at (TIMESTAMP)     │
│  updated_at (TIMESTAMP)     │
└─────────────────────────────┘

**Ví dụ PROJECTS status:**
- PLANNING: Đang lập kế hoạch
- IN_PROGRESS: Đang thực hiện
- ON_HOLD: Tạm dừng
- COMPLETED: Hoàn thành
- CANCELLED: Đã hủy
- DELAYED: Bị trễ
- UNDER_REVIEW: Đang xem xét

**Ví dụ PROJECTS priority:**
- LOW: Thấp
- MEDIUM: Trung bình
- HIGH: Cao
- CRITICAL: Khẩn cấp

**Ví dụ PROJECT_TASKS status:**
- TODO: Chưa làm
- IN_PROGRESS: Đang làm
- IN_REVIEW: Đang review
- DONE: Hoàn thành
- BLOCKED: Bị chặn
- CANCELLED: Đã hủy
              │
              │ 1
              │
              │ *
┌─────────────────────────────┐       ┌─────────────────────────────┐
│       PROJECT_TASKS         │       │    PROJECT_MILESTONES       │
│        [PostgreSQL]         │       │        [PostgreSQL]         │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │
│ *project_id (UUID) [FK]     │       │ *project_id (UUID) [FK]     │
│  phase_id (UUID) [FK]       │       │  milestone_name (VARCHAR)   │
│  parent_task_id (UUID) [FK] │       │  description (TEXT)         │
│  title (VARCHAR)            │       │  target_date (DATE)         │
│  description (TEXT)         │       │  actual_date (DATE)         │
│  assignee_id (UUID) [FK]    │       │  deliverables (JSONB)       │
│  reporter_id (UUID) [FK]    │       │  success_criteria (TEXT)    │
│  start_date (DATE)          │       │  completion_percentage (DEC)│
│  due_date (DATE)            │       │  status (VARCHAR)           │
│  estimated_hours (DECIMAL)  │       │  created_at (TIMESTAMP)     │
│  actual_hours (DECIMAL)     │       │  updated_at (TIMESTAMP)     │
│  progress_percentage (DEC)  │       └─────────────────────────────┘
│  story_points (INT)         │
│  task_type (VARCHAR)        │
│  status (VARCHAR)           │
│  priority (VARCHAR)         │
│  labels (JSONB)             │
│  acceptance_criteria (TEXT) │
│  dependencies (JSONB)       │
│  created_at (TIMESTAMP)     │
│  updated_at (TIMESTAMP)     │
└─────────────────────────────┘
```

---

## 6. FLOW 6: Marketplace for Mini-Projects - PostgreSQL

```
┌─────────────────────────────┐       ┌─────────────────────────────┐
│    MARKETPLACE_PROJECTS     │       │     PROJECT_CATEGORIES      │
│        [PostgreSQL]         │       │        [PostgreSQL]         │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │
│ *business_id (UUID) [FK]    │       │  name (VARCHAR)             │
│ *category_id (UUID) [FK]    │       │  description (TEXT)         │
│  title (VARCHAR)            │       │  parent_id (UUID) [FK]      │
│  description (TEXT)         │       │  skill_mapping (JSONB)      │
│  requirements (TEXT)        │ *   1 │  created_at (TIMESTAMP)     │
│  deliverables (JSONB)       │───────│  updated_at (TIMESTAMP)     │
│  budget_min (DECIMAL)       │       └─────────────────────────────┘
│  budget_max (DECIMAL)       │
│  duration_weeks (INT)       │
│  complexity_level (VARCHAR) │
│  skills_required (JSONB)    │
│  location_requirement (VAR) │
│  remote_allowed (BOOLEAN)   │
│  deadline (DATE)            │
│  status (VARCHAR)           │
│  created_at (TIMESTAMP)     │
│  updated_at (TIMESTAMP)     │
└─────────────────────────────┘

**Ví dụ MARKETPLACE_PROJECTS status:**
- DRAFT: Bản nháp
- PUBLISHED: Đã đăng
- BIDDING_OPEN: Đang nhận thầu
- BIDDING_CLOSED: Đã đóng thầu
- AWARDED: Đã trao thầu
- IN_PROGRESS: Đang thực hiện
- COMPLETED: Hoàn thành
- CANCELLED: Đã hủy

**Ví dụ MARKETPLACE_PROJECTS complexity_level:**
- BEGINNER: Người mới bắt đầu
- INTERMEDIATE: Trung cấp
- ADVANCED: Nâng cao
- EXPERT: Chuyên gia

**Ví dụ MARKETPLACE_BIDS status:**
- SUBMITTED: Đã gửi
- UNDER_REVIEW: Đang xem xét
- SHORTLISTED: Được chọn vào danh sách ngắn
- ACCEPTED: Được chấp nhận
- REJECTED: Bị từ chối
- WITHDRAWN: Đã rút lại
              │
              │ 1
              │
              │ *
┌─────────────────────────────┐       ┌─────────────────────────────┐
│      MARKETPLACE_BIDS       │       │     PROJECT_PROPOSALS       │
│        [PostgreSQL]         │       │        [PostgreSQL]         │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │
│ *project_id (UUID) [FK]     │       │ *project_id (UUID) [FK]     │
│ *bidder_id (UUID) [FK]      │       │ *freelancer_id (UUID) [FK]  │
│  bid_amount (DECIMAL)       │       │  proposal_text (TEXT)       │
│  proposed_duration (INT)    │       │  estimated_hours (INT)      │
│  proposal_text (TEXT)       │       │  hourly_rate (DECIMAL)      │
│  cover_letter (TEXT)        │       │  total_cost (DECIMAL)       │
│  portfolio_links (JSONB)    │       │  timeline (TEXT)            │
│  availability (VARCHAR)     │       │  methodology (TEXT)         │
│  status (VARCHAR)           │       │  portfolio_samples (JSONB)  │
│  submitted_at (TIMESTAMP)   │       │  status (VARCHAR)           │
│  created_at (TIMESTAMP)     │       │  submitted_at (TIMESTAMP)   │
│  updated_at (TIMESTAMP)     │       │  created_at (TIMESTAMP)     │
└─────────────────────────────┘       │  updated_at (TIMESTAMP)     │
                                      └─────────────────────────────┘


┌─────────────────────────────┐       ┌─────────────────────────────┐
│   MARKETPLACE_CONTRACTS     │       │      ESCROW_PAYMENTS        │
│        [PostgreSQL]         │       │        [PostgreSQL]         │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │
│ *project_id (UUID) [FK]     │       │ *contract_id (UUID) [FK]    │
│ *business_id (UUID) [FK]    │       │  amount (DECIMAL)           │
│ *freelancer_id (UUID) [FK]  │       │  currency (VARCHAR)         │
│  contract_terms (TEXT)      │ 1   * │  payment_method (VARCHAR)   │
│  total_amount (DECIMAL)     │───────│  transaction_id (VARCHAR)   │
│  payment_schedule (JSONB)   │       │  escrow_status (VARCHAR)    │
│  start_date (DATE)          │       │  held_until (DATE)          │
│  end_date (DATE)            │       │  release_conditions (TEXT)  │
│  milestones (JSONB)         │       │  dispute_id (UUID)          │
│  status (VARCHAR)           │       │  created_at (TIMESTAMP)     │
│  signed_at (TIMESTAMP)      │       │  updated_at (TIMESTAMP)     │
│  created_at (TIMESTAMP)     │       └─────────────────────────────┘
│  updated_at (TIMESTAMP)     │
└─────────────────────────────┘
              │
              │ 1
              │
              │ *
┌─────────────────────────────┐
│    MARKETPLACE_REVIEWS      │
│        [PostgreSQL]         │
├─────────────────────────────┤
│ *id (UUID) [PK]             │
│ *contract_id (UUID) [FK]    │
│ *reviewer_id (UUID) [FK]    │
│ *reviewee_id (UUID) [FK]    │
│  rating (DECIMAL)           │
│  review_text (TEXT)         │
│  review_type (VARCHAR)      │
│  created_at (TIMESTAMP)     │
│  updated_at (TIMESTAMP)     │
└─────────────────────────────┘
```

---

## 7. FLOW 7: Interactive Learning Platform - PostgreSQL

```
┌─────────────────────────────┐       ┌─────────────────────────────┐
│          COURSES            │       │     COURSE_CATEGORIES       │
│        [PostgreSQL]         │       │        [PostgreSQL]         │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │
│ *instructor_id (UUID) [FK]  │       │  name (VARCHAR)             │
│ *category_id (UUID) [FK]    │       │  description (TEXT)         │
│  title (VARCHAR)            │       │  parent_id (UUID) [FK]      │
│  description (TEXT)         │ *   1 │  skill_mapping (JSONB)      │
│  learning_objectives (JSON) │───────│  created_at (TIMESTAMP)     │
│  prerequisites (JSONB)      │       │  updated_at (TIMESTAMP)     │
│  level (VARCHAR)            │       └─────────────────────────────┘
│  duration_hours (INT)       │
│  price (DECIMAL)            │
│  certification_offered      │
│  max_students (INT)         │
│  status (VARCHAR)           │
│  published_at (TIMESTAMP)   │
│  created_at (TIMESTAMP)     │
│  updated_at (TIMESTAMP)     │
└─────────────────────────────┘
              │
              │ 1
              │
              │ *
┌─────────────────────────────┐       ┌─────────────────────────────┐
│      COURSE_MODULES         │       │     COURSE_ENROLLMENTS      │
│        [PostgreSQL]         │       │        [PostgreSQL]         │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │
│ *course_id (UUID) [FK]      │       │ *user_id (UUID) [FK]        │
│  title (VARCHAR)            │       │ *course_id (UUID) [FK]      │
│  description (TEXT)         │       │  enrollment_type (VARCHAR)  │
│  learning_objectives (JSON) │       │  payment_status (VARCHAR)   │
│  order_index (INT)          │       │  enrolled_at (TIMESTAMP)    │
│  estimated_duration (INT)   │       │  started_at (TIMESTAMP)     │
│  is_mandatory (BOOLEAN)     │       │  completed_at (TIMESTAMP)   │
│  created_at (TIMESTAMP)     │       │  progress_percentage (INT)  │
│  updated_at (TIMESTAMP)     │       │  current_module_id (UUID)   │
└─────────────────────────────┘       │  status (VARCHAR)           │
              │                       │  certificate_issued (BOOL)  │
              │ 1                     │  created_at (TIMESTAMP)     │
              │                       │  updated_at (TIMESTAMP)     │
              │ *                     └─────────────────────────────┘
┌─────────────────────────────┐
│      COURSE_LESSONS         │
│        [PostgreSQL]         │
├─────────────────────────────┤
│ *id (UUID) [PK]             │
│ *module_id (UUID) [FK]      │
│  title (VARCHAR)            │
│  content_type (VARCHAR)     │
│  content_url (VARCHAR)      │
│  content_text (TEXT)        │
│  duration_minutes (INT)     │
│  order_index (INT)          │
│  is_preview (BOOLEAN)       │
│  quiz_questions (JSONB)     │
│  resources (JSONB)          │
│  created_at (TIMESTAMP)     │
│  updated_at (TIMESTAMP)     │
└─────────────────────────────┘
              │
              │ 1
              │
              │ *
┌─────────────────────────────┐
│     LEARNING_PROGRESS       │
│        [PostgreSQL]         │
├─────────────────────────────┤
│ *id (UUID) [PK]             │
│ *user_id (UUID) [FK]        │
│ *lesson_id (UUID) [FK]      │
│ *enrollment_id (UUID) [FK]  │
│  started_at (TIMESTAMP)     │
│  completed_at (TIMESTAMP)   │
│  time_spent_minutes (INT)   │
│  quiz_score (DECIMAL)       │
│  quiz_attempts (INT)        │
│  notes (TEXT)               │
│  status (VARCHAR)           │
│  created_at (TIMESTAMP)     │
│  updated_at (TIMESTAMP)     │
└─────────────────────────────┘
```

---

## 8. FLOW 8: Notification System - PostgreSQL

```
┌─────────────────────────────┐       ┌─────────────────────────────┐
│       NOTIFICATIONS         │       │  NOTIFICATION_PREFERENCES   │
│        [PostgreSQL]         │       │        [PostgreSQL]         │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │
│ *user_id (UUID) [FK]        │       │ *user_id (UUID) [FK]        │
│  title (VARCHAR)            │       │  notification_type (VARCHAR)│
│  message (TEXT)             │       │  email_enabled (BOOLEAN)    │
│  notification_type (VARCHAR)│       │  sms_enabled (BOOLEAN)      │
│  priority (VARCHAR)         │       │  push_enabled (BOOLEAN)     │
│  channel (VARCHAR)          │       │  in_app_enabled (BOOLEAN)   │
│  read_at (TIMESTAMP)        │       │  frequency (VARCHAR)        │
│  action_url (VARCHAR)       │       │  quiet_hours_start (TIME)   │
│  metadata (JSONB)           │       │  quiet_hours_end (TIME)     │
│  expires_at (TIMESTAMP)     │       │  timezone (VARCHAR)         │
│  created_at (TIMESTAMP)     │       │  created_at (TIMESTAMP)     │
│  updated_at (TIMESTAMP)     │       │  updated_at (TIMESTAMP)     │
└─────────────────────────────┘       └─────────────────────────────┘


┌─────────────────────────────┐       ┌─────────────────────────────┐
│   NOTIFICATION_TEMPLATES    │       │     NOTIFICATION_LOGS       │
│        [PostgreSQL]         │       │        [PostgreSQL]         │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │
│  template_name (VARCHAR)    │       │ *notification_id (UUID) [FK]│
│  template_type (VARCHAR)    │       │  channel (VARCHAR)          │
│  subject_template (VARCHAR) │       │  recipient (VARCHAR)        │
│  body_template (TEXT)       │       │  status (VARCHAR)           │
│  variables (JSONB)          │       │  sent_at (TIMESTAMP)        │
│  is_active (BOOLEAN)        │       │  delivered_at (TIMESTAMP)   │
│  created_at (TIMESTAMP)     │       │  error_message (TEXT)       │
│  updated_at (TIMESTAMP)     │       │  retry_count (INT)          │
└─────────────────────────────┘       │  created_at (TIMESTAMP)     │
                                      └─────────────────────────────┘

**Ví dụ NOTIFICATION_TEMPLATES type:**
- EMAIL_WELCOME: Email chào mừng
- EMAIL_APPLICATION_RECEIVED: Thông báo nhận hồ sơ
- EMAIL_INTERVIEW_INVITATION: Mời phỏng vấn
- EMAIL_PROJECT_ASSIGNED: Giao dự án
- EMAIL_PAYMENT_CONFIRMATION: Xác nhận thanh toán
- SMS_REMINDER: Nhắc nhở qua SMS
- PUSH_NOTIFICATION: Thông báo đẩy

**Ví dụ NOTIFICATION_QUEUE status:**
- PENDING: Chờ gửi
- SENT: Đã gửi
- DELIVERED: Đã nhận
- FAILED: Gửi thất bại
- RETRY: Đang thử lại

**Ví dụ AUDIT_LOGS action:**
- CREATE: Tạo mới
- UPDATE: Cập nhật
- DELETE: Xóa
- LOGIN: Đăng nhập
- LOGOUT: Đăng xuất
- VIEW: Xem
- DOWNLOAD: Tải xuống
- APPROVE: Phê duyệt
- REJECT: Từ chối
```

---

## 9. FLOW 9: Analytics & Audit - PostgreSQL + MongoDB

```
┌─────────────────────────────┐       ┌─────────────────────────────┐
│         AUDIT_LOGS          │       │      USER_ACTIVITIES        │
│        [PostgreSQL]         │       │         [MongoDB]           │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *_id (ObjectId) [PK]        │
│ *user_id (UUID) [FK]        │       │  user_id (UUID)             │
│  action (VARCHAR)           │       │  session_id (String)        │
│  resource_type (VARCHAR)    │       │  activity_type (String)     │
│  resource_id (UUID)         │       │  page_url (String)          │
│  old_values (JSONB)         │       │  action_details (Object)    │
│  new_values (JSONB)         │       │  ip_address (String)        │
│  ip_address (INET)          │       │  user_agent (String)        │
│  user_agent (TEXT)          │       │  device_info (Object)       │
│  session_id (VARCHAR)       │       │  location (Object)          │
│  created_at (TIMESTAMP)     │       │  duration_seconds (Number)  │
└─────────────────────────────┘       │  timestamp (Date)           │
                                      │  metadata (Object)          │
                                      └─────────────────────────────┘


┌─────────────────────────────┐       ┌─────────────────────────────┐
│      SYSTEM_METRICS         │       │      BUSINESS_ANALYTICS     │
│         [MongoDB]           │       │         [MongoDB]           │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *_id (ObjectId) [PK]        │       │ *_id (ObjectId) [PK]        │
│  metric_name (String)       │       │  business_id (UUID)         │
│  metric_type (String)       │       │  metric_date (Date)         │
│  value (Number)             │       │  revenue (Number)           │
│  unit (String)              │       │  active_projects (Number)   │
│  tags (Array)               │       │  team_utilization (Number)  │
│  dimensions (Object)        │       │  client_satisfaction (Num)  │
│  timestamp (Date)           │       │  cost_efficiency (Number)   │
│  source (String)            │       │  delivery_performance (Num) │
│  aggregation_level (String) │       │  talent_retention (Number)  │
│  metadata (Object)          │       │  skill_demand (Array)       │
└─────────────────────────────┘       │  market_trends (Object)     │
                                      │  competitive_analysis (Obj) │
                                      │  created_at (Date)          │
                                      └─────────────────────────────┘
```

---

## 10. Elasticsearch Indexes

### 10.1. Talent Search Index
```
Index: talent_search
{
  "mappings": {
    "properties": {
      "talent_id": { "type": "keyword" },
      "full_name": { "type": "text" },
      "email": { "type": "keyword" },
      "skills": {
        "type": "nested",
        "properties": {
          "skill_name": { "type": "text" },
          "proficiency_level": { "type": "integer" },
          "years_experience": { "type": "float" }
        }
      },
      "location": { "type": "geo_point" },
      "availability": { "type": "keyword" },
      "hourly_rate": { "type": "float" },
      "experience_years": { "type": "integer" },
      "education_level": { "type": "keyword" },
      "certifications": { "type": "keyword" },
      "performance_rating": { "type": "float" },
      "last_active": { "type": "date" }
    }
  }
}
```

### 10.2. Project Search Index
```
Index: project_search
{
  "mappings": {
    "properties": {
      "project_id": { "type": "keyword" },
      "title": { "type": "text" },
      "description": { "type": "text" },
      "required_skills": {
        "type": "nested",
        "properties": {
          "skill_name": { "type": "text" },
          "proficiency_required": { "type": "keyword" }
        }
      },
      "budget_range": {
        "type": "object",
        "properties": {
          "min": { "type": "float" },
          "max": { "type": "float" }
        }
      },
      "location": { "type": "geo_point" },
      "remote_allowed": { "type": "boolean" },
      "urgency": { "type": "keyword" },
      "status": { "type": "keyword" },
      "created_at": { "type": "date" },
      "deadline": { "type": "date" }
    }
  }
}
```

### 10.3. Course Search Index
```
Index: course_search
{
  "mappings": {
    "properties": {
      "course_id": { "type": "keyword" },
      "title": { "type": "text" },
      "description": { "type": "text" },
      "skills_covered": { "type": "keyword" },
      "level": { "type": "keyword" },
      "category": { "type": "keyword" },
      "instructor_name": { "type": "text" },
      "rating": { "type": "float" },
      "duration_hours": { "type": "float" },
      "price": { "type": "float" },
      "certification_offered": { "type": "boolean" },
      "enrollment_count": { "type": "integer" },
      "completion_rate": { "type": "float" }
    }
  }
}
```

---

## 11. Tổng kết Database Architecture

### 11.1. PostgreSQL Tables (Core Business Logic)
- **Authentication**: roles, users
- **Business Management**: business_registrations, businesses, business_contacts, business_requirements, requirement_skills, contracts
- **Talent Management**: talent_applications, talents, talent_skills, skill_assessments, training_progress, talent_assignments, talent_evaluations
- **Team Management**: team_formation_requests, odc_teams, team_members, team_roles, team_workspaces, team_performance_metrics
- **Project Management**: projects, project_phases, project_tasks, project_milestones
- **Marketplace Management**: marketplace_projects, marketplace_bids, marketplace_contracts, marketplace_milestones, escrow_payments
- **Marketplace**: marketplace_projects, project_categories, marketplace_bids, project_proposals, marketplace_contracts, escrow_payments, marketplace_reviews
- **Learning Platform**: courses, course_categories, course_modules, course_lessons, course_enrollments, learning_progress
- **Notifications**: notifications, notification_preferences, notification_templates, notification_logs
- **Audit**: audit_logs
- **Master Data**: skills

### 11.2. MongoDB Collections (Flexible/Unstructured Data)
- **Analytics**: user_activities, system_metrics, business_analytics
- **Logs**: application_logs, error_logs, performance_logs
- **User Profiles**: talent_profiles, business_profiles
- **Content**: course_content, project_documents

### 11.3. Elasticsearch Indexes (Search & Analytics)
- **Search**: talent_search, project_search, course_search
- **Analytics**: skill_matching, performance_analytics
- **Real-time**: activity_streams, notification_feeds
