# Schema Database ODC Platform - Mô tả dạng Text

## 1. CORE SYSTEM - Quản lý người dùng và phân quyền

### Entity: ROLES
**Fields:**
- id (UUID, Primary Key)
- name (VARCHAR) - tên role
- description (TEXT) - mô tả role
- permissions (JSONB) - quyền hạn dạng JSON
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Role 1-N với Users

### Entity: USERS
**Fields:**
- id (UUID, Primary Key)
- email (VARCHAR, UNIQUE) - email đăng nhập
- password_hash (VARCHAR) - mật khẩu đã hash
- full_name (VARCHAR) - họ tên đầy đủ
- phone (VARCHAR) - số điện thoại
- role_id (UUID, Foreign Key → ROLES) - liên kết với role
- status (VARCHAR) - trạng thái tài khoản
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** User N-1 với Role

---

## 2. BUSINESS MANAGEMENT - Quản lý doanh nghiệp

### Entity: BUSINESS_REGISTRATIONS
**Fields:**
- id (UUID, Primary Key)
- business_name (VARCHAR) - tên doanh nghiệp
- business_email (VARCHAR) - email doanh nghiệp
- contact_person_name (VARCHAR) - tên người liên hệ
- contact_person_email (VARCHAR) - email người liên hệ
- contact_person_phone (VARCHAR) - SĐT người liên hệ
- industry (VARCHAR) - ngành nghề
- company_size (VARCHAR) - quy mô công ty
- website (VARCHAR) - website công ty
- business_license (VARCHAR) - giấy phép kinh doanh
- tax_id (VARCHAR) - mã số thuế
- address (TEXT) - địa chỉ
- city (VARCHAR) - thành phố
- country (VARCHAR) - quốc gia
- registration_documents (JSONB) - tài liệu đăng ký
- status (VARCHAR) - trạng thái đăng ký
- submitted_at (TIMESTAMP) - thời gian nộp
- reviewed_at (TIMESTAMP) - thời gian duyệt
- reviewed_by (UUID, Foreign Key → USERS) - người duyệt
- rejection_reason (TEXT) - lý do từ chối
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Business_Registration 1-1 với Business, N-1 với User (reviewer)

### Entity: BUSINESSES
**Fields:**
- id (UUID, Primary Key)
- registration_id (UUID, Foreign Key → BUSINESS_REGISTRATIONS) - liên kết đăng ký
- name (VARCHAR) - tên công ty
- email (VARCHAR) - email công ty
- phone (VARCHAR) - số điện thoại
- address (TEXT) - địa chỉ
- industry (VARCHAR) - ngành nghề
- size (VARCHAR) - quy mô
- website (VARCHAR) - website
- business_license (VARCHAR) - giấy phép
- tax_id (VARCHAR) - mã số thuế
- logo_url (VARCHAR) - URL logo
- description (TEXT) - mô tả công ty
- verification_status (VARCHAR) - trạng thái xác minh
- subscription_plan (VARCHAR) - gói đăng ký
- subscription_expires_at (TIMESTAMP) - hết hạn đăng ký
- billing_address (TEXT) - địa chỉ thanh toán
- billing_contact_email (VARCHAR) - email thanh toán
- payment_method_id (VARCHAR) - phương thức thanh toán
- lab_setup_completed (BOOLEAN) - hoàn thành thiết lập lab
- onboarding_completed (BOOLEAN) - hoàn thành onboarding
- status (VARCHAR) - trạng thái
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Business 1-1 với Business_Registration, 1-N với Business_Contacts, 1-N với Business_Requirements, 1-N với Contracts, 1-N với ODC_Teams

### Entity: BUSINESS_CONTACTS
**Fields:**
- id (UUID, Primary Key)
- business_id (UUID, Foreign Key → BUSINESSES) - liên kết doanh nghiệp
- user_id (UUID, Foreign Key → USERS) - liên kết người dùng
- position (VARCHAR) - chức vụ
- is_primary (BOOLEAN) - liên hệ chính
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Business_Contact N-1 với Business, N-1 với User

### Entity: BUSINESS_REQUIREMENTS
**Fields:**
- id (UUID, Primary Key)
- business_id (UUID, Foreign Key → BUSINESSES) - liên kết doanh nghiệp
- title (VARCHAR) - tiêu đề yêu cầu
- description (TEXT) - mô tả yêu cầu
- team_size (INT) - quy mô team
- duration (VARCHAR) - thời gian
- contract_type (VARCHAR) - loại hợp đồng
- status (VARCHAR) - trạng thái
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Business_Requirement N-1 với Business, 1-N với Requirement_Skills, 1-N với Contracts

### Entity: REQUIREMENT_SKILLS
**Fields:**
- id (UUID, Primary Key)
- requirement_id (UUID, Foreign Key → BUSINESS_REQUIREMENTS) - liên kết yêu cầu
- skill_id (UUID, Foreign Key → SKILLS) - liên kết kỹ năng
- experience_level (VARCHAR) - mức độ kinh nghiệm
- created_at (TIMESTAMP)

**Relationship:** Requirement_Skill N-1 với Business_Requirement, N-1 với Skill

### Entity: CONTRACTS
**Fields:**
- id (UUID, Primary Key)
- business_id (UUID, Foreign Key → BUSINESSES) - liên kết doanh nghiệp
- requirement_id (UUID, Foreign Key → BUSINESS_REQUIREMENTS) - liên kết yêu cầu
- title (VARCHAR) - tiêu đề hợp đồng
- description (TEXT) - mô tả
- start_date (DATE) - ngày bắt đầu
- end_date (DATE) - ngày kết thúc
- contract_type (VARCHAR) - loại hợp đồng
- status (VARCHAR) - trạng thái
- file_path (VARCHAR) - đường dẫn file
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Contract N-1 với Business, N-1 với Business_Requirement, 1-N với ODC_Teams

---

## 3. TALENT MANAGEMENT - Quản lý nhân tài

### Entity: TALENT_APPLICATIONS
**Fields:**
- id (UUID, Primary Key)
- applicant_name (VARCHAR) - tên ứng viên
- email (VARCHAR) - email ứng viên
- phone (VARCHAR) - số điện thoại
- education_level (VARCHAR) - trình độ học vấn
- university (VARCHAR) - trường đại học
- graduation_year (INT) - năm tốt nghiệp
- major (VARCHAR) - chuyên ngành
- gpa (DECIMAL) - điểm GPA
- years_of_experience (INT) - số năm kinh nghiệm
- current_position (VARCHAR) - vị trí hiện tại
- current_company (VARCHAR) - công ty hiện tại
- resume_url (VARCHAR) - URL CV
- portfolio_url (VARCHAR) - URL portfolio
- cover_letter (TEXT) - thư xin việc
- preferred_technologies (JSONB) - công nghệ ưa thích
- expected_salary_min (DECIMAL) - lương mong muốn tối thiểu
- expected_salary_max (DECIMAL) - lương mong muốn tối đa
- availability_date (DATE) - ngày có thể bắt đầu
- application_source (VARCHAR) - nguồn ứng tuyển
- status (VARCHAR) - trạng thái đơn
- submitted_at (TIMESTAMP) - thời gian nộp
- reviewed_at (TIMESTAMP) - thời gian duyệt
- reviewed_by (UUID, Foreign Key → USERS) - người duyệt
- interview_scheduled_at (TIMESTAMP) - lịch phỏng vấn
- rejection_reason (TEXT) - lý do từ chối
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Talent_Application 1-1 với Talent, N-1 với User (reviewer)

### Entity: TALENTS
**Fields:**
- id (UUID, Primary Key)
- application_id (UUID, Foreign Key → TALENT_APPLICATIONS) - liên kết đơn ứng tuyển
- user_id (UUID, Foreign Key → USERS) - liên kết người dùng
- employee_id (VARCHAR, UNIQUE) - mã nhân viên
- education_level (VARCHAR) - trình độ học vấn
- university (VARCHAR) - trường đại học
- graduation_year (INT) - năm tốt nghiệp
- major (VARCHAR) - chuyên ngành
- gpa (DECIMAL) - điểm GPA
- years_of_experience (INT) - số năm kinh nghiệm
- current_position (VARCHAR) - vị trí hiện tại
- department (VARCHAR) - phòng ban
- manager_id (UUID, Foreign Key → USERS) - quản lý trực tiếp
- hire_date (DATE) - ngày tuyển dụng
- salary (DECIMAL) - lương
- performance_rating (DECIMAL) - đánh giá hiệu suất
- availability (VARCHAR) - tình trạng sẵn sàng
- location (VARCHAR) - địa điểm
- timezone (VARCHAR) - múi giờ
- remote_work_preference (VARCHAR) - ưu tiên làm việc từ xa
- linkedin_url (VARCHAR) - URL LinkedIn
- github_url (VARCHAR) - URL GitHub
- portfolio_url (VARCHAR) - URL portfolio
- bio (TEXT) - tiểu sử
- status (VARCHAR) - trạng thái
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Talent 1-1 với Talent_Application, 1-1 với User, N-1 với User (manager), 1-N với Talent_Skills, 1-N với Skill_Assessments, 1-N với Training_Progress, 1-N với Talent_Assignments, 1-N với Talent_Evaluations, 1-N với Team_Members

### Entity: SKILLS
**Fields:**
- id (UUID, Primary Key)
- name (VARCHAR, UNIQUE) - tên kỹ năng
- category (VARCHAR) - danh mục kỹ năng
- description (TEXT) - mô tả
- created_at (TIMESTAMP)

**Relationship:** Skill 1-N với Talent_Skills, 1-N với Requirement_Skills, 1-N với Skill_Assessments

### Entity: TALENT_SKILLS
**Fields:**
- id (UUID, Primary Key)
- talent_id (UUID, Foreign Key → TALENTS) - liên kết talent
- skill_id (UUID, Foreign Key → SKILLS) - liên kết kỹ năng
- proficiency_level (INT) - mức độ thành thạo
- years_of_experience (DECIMAL) - số năm kinh nghiệm
- verified (BOOLEAN) - đã xác minh
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Talent_Skill N-1 với Talent, N-1 với Skill

### Entity: SKILL_ASSESSMENTS
**Fields:**
- id (UUID, Primary Key)
- talent_id (UUID, Foreign Key → TALENTS) - liên kết talent
- skill_id (UUID, Foreign Key → SKILLS) - liên kết kỹ năng
- assessment_type (VARCHAR) - loại đánh giá
- assessment_date (DATE) - ngày đánh giá
- score (DECIMAL) - điểm số
- max_score (DECIMAL) - điểm tối đa
- percentage (DECIMAL) - phần trăm (tự động tính)
- assessor_id (UUID, Foreign Key → USERS) - người đánh giá
- assessment_details (JSONB) - chi tiết đánh giá
- feedback (TEXT) - phản hồi
- certification_earned (VARCHAR) - chứng chỉ đạt được
- valid_until (DATE) - có hiệu lực đến
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Skill_Assessment N-1 với Talent, N-1 với Skill, N-1 với User (assessor)

### Entity: TRAINING_PROGRESS
**Fields:**
- id (UUID, Primary Key)
- talent_id (UUID, Foreign Key → TALENTS) - liên kết talent
- course_id (UUID) - ID khóa học
- course_name (VARCHAR) - tên khóa học
- training_type (VARCHAR) - loại đào tạo
- status (VARCHAR) - trạng thái
- start_date (DATE) - ngày bắt đầu
- completion_date (DATE) - ngày hoàn thành
- progress_percentage (DECIMAL) - phần trăm tiến độ
- final_score (DECIMAL) - điểm cuối
- certification_obtained (VARCHAR) - chứng chỉ đạt được
- trainer_id (UUID, Foreign Key → USERS) - người đào tạo
- training_hours (DECIMAL) - số giờ đào tạo
- cost (DECIMAL) - chi phí
- notes (TEXT) - ghi chú
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Training_Progress N-1 với Talent, N-1 với User (trainer)

### Entity: TALENT_ASSIGNMENTS
**Fields:**
- id (UUID, Primary Key)
- talent_id (UUID, Foreign Key → TALENTS) - liên kết talent
- project_id (UUID) - ID dự án
- team_id (UUID) - ID team
- assignment_type (VARCHAR) - loại phân công
- role (VARCHAR) - vai trò
- start_date (DATE) - ngày bắt đầu
- end_date (DATE) - ngày kết thúc
- allocation_percentage (DECIMAL) - phần trăm phân bổ
- hourly_rate (DECIMAL) - giá theo giờ
- status (VARCHAR) - trạng thái
- assigned_by (UUID, Foreign Key → USERS) - người phân công
- notes (TEXT) - ghi chú
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Talent_Assignment N-1 với Talent, N-1 với User (assigned_by)

### Entity: TALENT_EVALUATIONS
**Fields:**
- id (UUID, Primary Key)
- talent_id (UUID, Foreign Key → TALENTS) - liên kết talent
- evaluator_id (UUID, Foreign Key → USERS) - người đánh giá
- evaluation_period (VARCHAR) - kỳ đánh giá
- evaluation_date (DATE) - ngày đánh giá
- overall_score (DECIMAL) - điểm tổng thể
- technical_score (DECIMAL) - điểm kỹ thuật
- communication_score (DECIMAL) - điểm giao tiếp
- teamwork_score (DECIMAL) - điểm làm việc nhóm
- leadership_score (DECIMAL) - điểm lãnh đạo
- problem_solving_score (DECIMAL) - điểm giải quyết vấn đề
- strengths (TEXT) - điểm mạnh
- areas_for_improvement (TEXT) - điểm cần cải thiện
- goals_next_period (TEXT) - mục tiêu kỳ tới
- comments (TEXT) - nhận xét
- status (VARCHAR) - trạng thái
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Talent_Evaluation N-1 với Talent, N-1 với User (evaluator)

---

## 4. ODC TEAM MANAGEMENT - Quản lý đội nhóm ODC

### Entity: TEAM_FORMATION_REQUESTS
**Fields:**
- id (UUID, Primary Key)
- business_id (UUID, Foreign Key → BUSINESSES) - liên kết doanh nghiệp
- request_title (VARCHAR) - tiêu đề yêu cầu
- project_description (TEXT) - mô tả dự án
- team_size (INT) - quy mô team
- required_skills (JSONB) - kỹ năng yêu cầu
- experience_level (VARCHAR) - mức độ kinh nghiệm
- project_duration (VARCHAR) - thời gian dự án
- start_date (DATE) - ngày bắt đầu
- budget_range (VARCHAR) - khoảng ngân sách
- work_arrangement (VARCHAR) - hình thức làm việc
- location (VARCHAR) - địa điểm
- special_requirements (TEXT) - yêu cầu đặc biệt
- priority (VARCHAR) - độ ưu tiên
- status (VARCHAR) - trạng thái
- submitted_at (TIMESTAMP) - thời gian nộp
- reviewed_at (TIMESTAMP) - thời gian duyệt
- reviewed_by (UUID, Foreign Key → USERS) - người duyệt
- approval_notes (TEXT) - ghi chú phê duyệt
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Team_Formation_Request N-1 với Business, N-1 với User (reviewer), 1-1 với ODC_Team

### Entity: TEAM_ROLES
**Fields:**
- id (UUID, Primary Key)
- role_name (VARCHAR, UNIQUE) - tên vai trò
- role_description (TEXT) - mô tả vai trò
- required_skills (JSONB) - kỹ năng yêu cầu
- experience_level (VARCHAR) - mức độ kinh nghiệm
- responsibilities (TEXT) - trách nhiệm
- is_leadership_role (BOOLEAN) - vai trò lãnh đạo
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Team_Role 1-N với Team_Members

### Entity: TEAM_WORKSPACES
**Fields:**
- id (UUID, Primary Key)
- workspace_name (VARCHAR) - tên không gian làm việc
- location (VARCHAR) - địa điểm
- capacity (INT) - sức chứa
- facilities (JSONB) - tiện nghi
- availability_status (VARCHAR) - trạng thái sẵn sàng
- cost_per_month (DECIMAL) - chi phí hàng tháng
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Team_Workspace 1-N với ODC_Teams

### Entity: ODC_TEAMS
**Fields:**
- id (UUID, Primary Key)
- formation_request_id (UUID, Foreign Key → TEAM_FORMATION_REQUESTS) - liên kết yêu cầu thành lập
- name (VARCHAR) - tên team
- business_id (UUID, Foreign Key → BUSINESSES) - liên kết doanh nghiệp
- contract_id (UUID, Foreign Key → CONTRACTS) - liên kết hợp đồng
- team_lead_id (UUID, Foreign Key → TALENTS) - trưởng nhóm
- workspace_id (UUID, Foreign Key → TEAM_WORKSPACES) - không gian làm việc
- team_type (VARCHAR) - loại team
- methodology (VARCHAR) - phương pháp luận
- max_capacity (INT) - sức chứa tối đa
- current_size (INT) - quy mô hiện tại
- formation_date (DATE) - ngày thành lập
- planned_end_date (DATE) - ngày kết thúc dự kiến
- actual_end_date (DATE) - ngày kết thúc thực tế
- performance_rating (DECIMAL) - đánh giá hiệu suất
- budget_allocated (DECIMAL) - ngân sách phân bổ
- budget_spent (DECIMAL) - ngân sách đã chi
- status (VARCHAR) - trạng thái
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** ODC_Team N-1 với Business, N-1 với Contract, N-1 với Talent (team_lead), N-1 với Team_Workspace, 1-1 với Team_Formation_Request, 1-N với Team_Members, 1-N với Projects, 1-N với Team_Mentors, 1-N với Team_Performance_Metrics, 1-N với Team_Equipment_Assignments

### Entity: TEAM_MEMBERS
**Fields:**
- id (UUID, Primary Key)
- team_id (UUID, Foreign Key → ODC_TEAMS) - liên kết team
- talent_id (UUID, Foreign Key → TALENTS) - liên kết talent
- role_id (UUID, Foreign Key → TEAM_ROLES) - liên kết vai trò
- role (VARCHAR) - vai trò
- is_team_lead (BOOLEAN) - là trưởng nhóm
- allocation_percentage (INT) - phần trăm phân bổ thời gian
- hourly_rate (DECIMAL) - giá theo giờ
- join_date (DATE) - ngày tham gia
- end_date (DATE) - ngày kết thúc
- performance_rating (DECIMAL) - đánh giá hiệu suất
- last_evaluation_date (DATE) - ngày đánh giá cuối
- notes (TEXT) - ghi chú
- status (VARCHAR) - trạng thái
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Team_Member N-1 với ODC_Team, N-1 với Talent, N-1 với Team_Role

### Entity: TEAM_MENTORS
**Fields:**
- id (UUID, Primary Key)
- team_id (UUID, Foreign Key → ODC_TEAMS) - liên kết team
- mentor_id (UUID, Foreign Key → USERS) - liên kết mentor
- mentorship_type (VARCHAR) - loại mentorship
- specialization (VARCHAR) - chuyên môn
- start_date (DATE) - ngày bắt đầu
- end_date (DATE) - ngày kết thúc
- hours_per_week (INT) - số giờ mỗi tuần
- mentorship_goals (TEXT) - mục tiêu mentorship
- progress_notes (TEXT) - ghi chú tiến độ
- status (VARCHAR) - trạng thái
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Team_Mentor N-1 với ODC_Team, N-1 với User (mentor)

### Entity: TEAM_PERFORMANCE_METRICS
**Fields:**
- id (UUID, Primary Key)
- team_id (UUID, Foreign Key → ODC_TEAMS) - liên kết team
- metric_date (DATE) - ngày đo lường
- productivity_score (DECIMAL) - điểm năng suất
- quality_score (DECIMAL) - điểm chất lượng
- collaboration_score (DECIMAL) - điểm hợp tác
- client_satisfaction (DECIMAL) - sự hài lòng khách hàng
- delivery_timeliness (DECIMAL) - tính đúng hạn
- budget_efficiency (DECIMAL) - hiệu quả ngân sách
- team_velocity (INT) - velocity của team
- defect_rate (DECIMAL) - tỷ lệ lỗi
- knowledge_sharing_score (DECIMAL) - điểm chia sẻ kiến thức
- innovation_score (DECIMAL) - điểm đổi mới
- notes (TEXT) - ghi chú
- recorded_by (UUID, Foreign Key → USERS) - người ghi nhận
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Team_Performance_Metric N-1 với ODC_Team, N-1 với User (recorded_by)

### Entity: TEAM_EQUIPMENT_ASSIGNMENTS
**Fields:**
- id (UUID, Primary Key)
- team_id (UUID, Foreign Key → ODC_TEAMS) - liên kết team
- equipment_type (VARCHAR) - loại thiết bị
- equipment_name (VARCHAR) - tên thiết bị
- serial_number (VARCHAR) - số serial
- assigned_date (DATE) - ngày giao
- return_date (DATE) - ngày trả
- condition_notes (TEXT) - ghi chú tình trạng
- cost (DECIMAL) - chi phí
- status (VARCHAR) - trạng thái
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Team_Equipment_Assignment N-1 với ODC_Team

---

## 5. PROJECT MANAGEMENT - Quản lý dự án

### Entity: PROJECTS
**Fields:**
- id (UUID, Primary Key)
- name (VARCHAR) - tên dự án
- business_id (UUID, Foreign Key → BUSINESSES) - liên kết doanh nghiệp
- team_id (UUID, Foreign Key → ODC_TEAMS) - liên kết team
- project_manager_id (UUID, Foreign Key → USERS) - quản lý dự án
- description (TEXT) - mô tả
- objectives (TEXT) - mục tiêu
- scope (TEXT) - phạm vi
- deliverables (JSONB) - sản phẩm bàn giao
- start_date (DATE) - ngày bắt đầu
- end_date (DATE) - ngày kết thúc
- actual_start_date (DATE) - ngày bắt đầu thực tế
- actual_end_date (DATE) - ngày kết thúc thực tế
- budget (DECIMAL) - ngân sách
- actual_cost (DECIMAL) - chi phí thực tế
- currency (VARCHAR) - đơn vị tiền tệ
- priority (VARCHAR) - độ ưu tiên
- methodology (VARCHAR) - phương pháp luận
- progress_percentage (DECIMAL) - phần trăm tiến độ
- health_status (VARCHAR) - tình trạng sức khỏe dự án
- risk_level (VARCHAR) - mức độ rủi ro
- client_satisfaction (DECIMAL) - sự hài lòng khách hàng
- status (VARCHAR) - trạng thái
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Project N-1 với Business, N-1 với ODC_Team, N-1 với User (project_manager), 1-N với Project_Phases, 1-N với Project_Tasks, 1-N với Project_Milestones

### Entity: PROJECT_PHASES
**Fields:**
- id (UUID, Primary Key)
- project_id (UUID, Foreign Key → PROJECTS) - liên kết dự án
- phase_name (VARCHAR) - tên giai đoạn
- description (TEXT) - mô tả
- sequence_order (INT) - thứ tự
- start_date (DATE) - ngày bắt đầu
- end_date (DATE) - ngày kết thúc
- actual_start_date (DATE) - ngày bắt đầu thực tế
- actual_end_date (DATE) - ngày kết thúc thực tế
- budget (DECIMAL) - ngân sách
- actual_cost (DECIMAL) - chi phí thực tế
- progress_percentage (DECIMAL) - phần trăm tiến độ
- deliverables (JSONB) - sản phẩm bàn giao
- acceptance_criteria (TEXT) - tiêu chí chấp nhận
- status (VARCHAR) - trạng thái
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Project_Phase N-1 với Project, 1-N với Project_Tasks

### Entity: PROJECT_TASKS
**Fields:**
- id (UUID, Primary Key)
- project_id (UUID, Foreign Key → PROJECTS) - liên kết dự án
- phase_id (UUID, Foreign Key → PROJECT_PHASES) - liên kết giai đoạn
- parent_task_id (UUID, Foreign Key → PROJECT_TASKS) - task cha
- title (VARCHAR) - tiêu đề
- description (TEXT) - mô tả
- assignee_id (UUID, Foreign Key → USERS) - người được giao
- reporter_id (UUID, Foreign Key → USERS) - người báo cáo
- start_date (DATE) - ngày bắt đầu
- due_date (DATE) - ngày đến hạn
- actual_start_date (DATE) - ngày bắt đầu thực tế
- actual_completion_date (DATE) - ngày hoàn thành thực tế
- estimated_hours (DECIMAL) - số giờ ước tính
- actual_hours (DECIMAL) - số giờ thực tế
- progress_percentage (DECIMAL) - phần trăm tiến độ
- story_points (INT) - điểm story
- task_type (VARCHAR) - loại task
- status (VARCHAR) - trạng thái
- priority (VARCHAR) - độ ưu tiên
- labels (JSONB) - nhãn
- acceptance_criteria (TEXT) - tiêu chí chấp nhận
- dependencies (JSONB) - phụ thuộc
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Project_Task N-1 với Project, N-1 với Project_Phase, N-1 với Project_Task (parent), N-1 với User (assignee), N-1 với User (reporter), 1-N với Project_Task (children)

### Entity: PROJECT_MILESTONES
**Fields:**
- id (UUID, Primary Key)
- project_id (UUID, Foreign Key → PROJECTS) - liên kết dự án
- milestone_name (VARCHAR) - tên milestone
- description (TEXT) - mô tả
- target_date (DATE) - ngày mục tiêu
- actual_date (DATE) - ngày thực tế
- deliverables (JSONB) - sản phẩm bàn giao
- success_criteria (TEXT) - tiêu chí thành công
- status (VARCHAR) - trạng thái
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Project_Milestone N-1 với Project

---

## 6. MARKETPLACE SYSTEM - Hệ thống marketplace

### Entity: MARKETPLACE_PROJECTS
**Fields:**
- id (UUID, Primary Key)
- title (VARCHAR) - tiêu đề dự án
- description (TEXT) - mô tả
- business_id (UUID, Foreign Key → BUSINESSES) - liên kết doanh nghiệp
- category_id (UUID, Foreign Key → PROJECT_CATEGORIES) - danh mục
- budget_min (DECIMAL) - ngân sách tối thiểu
- budget_max (DECIMAL) - ngân sách tối đa
- currency (VARCHAR) - đơn vị tiền tệ
- duration_weeks (INT) - thời gian (tuần)
- required_skills (JSONB) - kỹ năng yêu cầu
- experience_level (VARCHAR) - mức độ kinh nghiệm
- project_type (VARCHAR) - loại dự án
- work_arrangement (VARCHAR) - hình thức làm việc
- location (VARCHAR) - địa điểm
- deadline (DATE) - hạn chót
- attachments (JSONB) - file đính kèm
- special_requirements (TEXT) - yêu cầu đặc biệt
- bid_deadline (TIMESTAMP) - hạn chót đấu thầu
- max_bids (INT) - số bid tối đa
- auto_award (BOOLEAN) - tự động trao thưởng
- featured (BOOLEAN) - nổi bật
- views_count (INT) - số lượt xem
- bids_count (INT) - số lượt bid
- status (VARCHAR) - trạng thái
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Marketplace_Project N-1 với Business, N-1 với Project_Category, 1-N với Marketplace_Bids, 1-N với Project_Proposals, 1-1 với Marketplace_Contract

### Entity: PROJECT_CATEGORIES
**Fields:**
- id (UUID, Primary Key)
- name (VARCHAR, UNIQUE) - tên danh mục
- description (TEXT) - mô tả
- parent_category_id (UUID, Foreign Key → PROJECT_CATEGORIES) - danh mục cha
- icon_url (VARCHAR) - URL icon
- is_active (BOOLEAN) - đang hoạt động
- sort_order (INT) - thứ tự sắp xếp
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Project_Category 1-N với Marketplace_Projects, N-1 với Project_Category (parent), 1-N với Project_Category (children)

### Entity: MARKETPLACE_BIDS
**Fields:**
- id (UUID, Primary Key)
- project_id (UUID, Foreign Key → MARKETPLACE_PROJECTS) - liên kết dự án
- talent_id (UUID, Foreign Key → TALENTS) - liên kết talent
- bid_amount (DECIMAL) - số tiền bid
- currency (VARCHAR) - đơn vị tiền tệ
- delivery_time_weeks (INT) - thời gian giao hàng
- proposal_text (TEXT) - văn bản đề xuất
- attachments (JSONB) - file đính kèm
- milestones (JSONB) - các milestone
- payment_terms (VARCHAR) - điều khoản thanh toán
- additional_services (JSONB) - dịch vụ bổ sung
- questions_for_client (TEXT) - câu hỏi cho khách hàng
- portfolio_samples (JSONB) - mẫu portfolio
- estimated_start_date (DATE) - ngày bắt đầu ước tính
- is_featured (BOOLEAN) - nổi bật
- status (VARCHAR) - trạng thái
- submitted_at (TIMESTAMP) - thời gian nộp
- reviewed_at (TIMESTAMP) - thời gian duyệt
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Marketplace_Bid N-1 với Marketplace_Project, N-1 với Talent

### Entity: PROJECT_PROPOSALS
**Fields:**
- id (UUID, Primary Key)
- project_id (UUID, Foreign Key → MARKETPLACE_PROJECTS) - liên kết dự án
- talent_id (UUID, Foreign Key → TALENTS) - liên kết talent
- proposal_title (VARCHAR) - tiêu đề đề xuất
- proposal_text (TEXT) - nội dung đề xuất
- technical_approach (TEXT) - phương pháp kỹ thuật
- timeline (JSONB) - timeline
- budget_breakdown (JSONB) - phân tích ngân sách
- team_composition (JSONB) - cấu trúc team
- risk_mitigation (TEXT) - giảm thiểu rủi ro
- quality_assurance (TEXT) - đảm bảo chất lượng
- post_delivery_support (TEXT) - hỗ trợ sau bàn giao
- references (JSONB) - tham khảo
- certifications (JSONB) - chứng chỉ
- attachments (JSONB) - file đính kèm
- status (VARCHAR) - trạng thái
- submitted_at (TIMESTAMP) - thời gian nộp
- reviewed_at (TIMESTAMP) - thời gian duyệt
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Project_Proposal N-1 với Marketplace_Project, N-1 với Talent

### Entity: MARKETPLACE_CONTRACTS
**Fields:**
- id (UUID, Primary Key)
- project_id (UUID, Foreign Key → MARKETPLACE_PROJECTS) - liên kết dự án
- talent_id (UUID, Foreign Key → TALENTS) - liên kết talent
- business_id (UUID, Foreign Key → BUSINESSES) - liên kết doanh nghiệp
- contract_title (VARCHAR) - tiêu đề hợp đồng
- contract_amount (DECIMAL) - số tiền hợp đồng
- currency (VARCHAR) - đơn vị tiền tệ
- payment_schedule (JSONB) - lịch thanh toán
- start_date (DATE) - ngày bắt đầu
- end_date (DATE) - ngày kết thúc
- deliverables (JSONB) - sản phẩm bàn giao
- terms_and_conditions (TEXT) - điều khoản
- intellectual_property_terms (TEXT) - điều khoản sở hữu trí tuệ
- confidentiality_terms (TEXT) - điều khoản bảo mật
- dispute_resolution (TEXT) - giải quyết tranh chấp
- contract_file_path (VARCHAR) - đường dẫn file hợp đồng
- signed_by_business_at (TIMESTAMP) - ký bởi doanh nghiệp
- signed_by_talent_at (TIMESTAMP) - ký bởi talent
- witness_id (UUID, Foreign Key → USERS) - người chứng kiến
- status (VARCHAR) - trạng thái
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Marketplace_Contract 1-1 với Marketplace_Project, N-1 với Talent, N-1 với Business, N-1 với User (witness), 1-N với Marketplace_Milestones, 1-N với Escrow_Payments

### Entity: MARKETPLACE_MILESTONES
**Fields:**
- id (UUID, Primary Key)
- contract_id (UUID, Foreign Key → MARKETPLACE_CONTRACTS) - liên kết hợp đồng
- milestone_name (VARCHAR) - tên milestone
- description (TEXT) - mô tả
- deliverables (JSONB) - sản phẩm bàn giao
- amount (DECIMAL) - số tiền
- due_date (DATE) - ngày đến hạn
- completion_date (DATE) - ngày hoàn thành
- acceptance_criteria (TEXT) - tiêu chí chấp nhận
- review_notes (TEXT) - ghi chú đánh giá
- status (VARCHAR) - trạng thái
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Marketplace_Milestone N-1 với Marketplace_Contract

### Entity: ESCROW_PAYMENTS
**Fields:**
- id (UUID, Primary Key)
- contract_id (UUID, Foreign Key → MARKETPLACE_CONTRACTS) - liên kết hợp đồng
- milestone_id (UUID, Foreign Key → MARKETPLACE_MILESTONES) - liên kết milestone
- amount (DECIMAL) - số tiền
- currency (VARCHAR) - đơn vị tiền tệ
- payment_method (VARCHAR) - phương thức thanh toán
- transaction_id (VARCHAR) - ID giao dịch
- deposited_at (TIMESTAMP) - thời gian gửi
- released_at (TIMESTAMP) - thời gian giải ngân
- refunded_at (TIMESTAMP) - thời gian hoàn tiền
- fees (DECIMAL) - phí
- status (VARCHAR) - trạng thái
- notes (TEXT) - ghi chú
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Escrow_Payment N-1 với Marketplace_Contract, N-1 với Marketplace_Milestone

### Entity: MARKETPLACE_REVIEWS
**Fields:**
- id (UUID, Primary Key)
- contract_id (UUID, Foreign Key → MARKETPLACE_CONTRACTS) - liên kết hợp đồng
- reviewer_id (UUID, Foreign Key → USERS) - người đánh giá
- reviewee_id (UUID, Foreign Key → USERS) - người được đánh giá
- reviewer_type (VARCHAR) - loại người đánh giá
- overall_rating (DECIMAL) - đánh giá tổng thể
- quality_rating (DECIMAL) - đánh giá chất lượng
- communication_rating (DECIMAL) - đánh giá giao tiếp
- timeliness_rating (DECIMAL) - đánh giá đúng hạn
- professionalism_rating (DECIMAL) - đánh giá chuyên nghiệp
- review_text (TEXT) - nội dung đánh giá
- pros (TEXT) - ưu điểm
- cons (TEXT) - nhược điểm
- would_recommend (BOOLEAN) - có giới thiệu không
- would_work_again (BOOLEAN) - có làm việc lại không
- is_public (BOOLEAN) - công khai
- status (VARCHAR) - trạng thái
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Marketplace_Review N-1 với Marketplace_Contract, N-1 với User (reviewer), N-1 với User (reviewee)

---

## 7. LEARNING PLATFORM - Nền tảng học tập

### Entity: COURSES
**Fields:**
- id (UUID, Primary Key)
- title (VARCHAR) - tiêu đề khóa học
- description (TEXT) - mô tả
- category_id (UUID, Foreign Key → COURSE_CATEGORIES) - danh mục
- instructor_id (UUID, Foreign Key → USERS) - giảng viên
- level (VARCHAR) - mức độ
- duration_hours (DECIMAL) - thời lượng (giờ)
- price (DECIMAL) - giá
- currency (VARCHAR) - đơn vị tiền tệ
- language (VARCHAR) - ngôn ngữ
- prerequisites (JSONB) - điều kiện tiên quyết
- learning_objectives (JSONB) - mục tiêu học tập
- skills_covered (JSONB) - kỹ năng được đề cập
- thumbnail_url (VARCHAR) - URL hình thu nhỏ
- trailer_url (VARCHAR) - URL video giới thiệu
- certificate_template (VARCHAR) - mẫu chứng chỉ
- passing_score (DECIMAL) - điểm đạt
- max_attempts (INT) - số lần thử tối đa
- enrollment_limit (INT) - giới hạn đăng ký
- enrollment_count (INT) - số lượng đăng ký
- average_rating (DECIMAL) - đánh giá trung bình
- total_reviews (INT) - tổng số đánh giá
- is_published (BOOLEAN) - đã xuất bản
- is_featured (BOOLEAN) - nổi bật
- tags (JSONB) - thẻ
- status (VARCHAR) - trạng thái
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Course N-1 với Course_Category, N-1 với User (instructor), 1-N với Course_Modules, 1-N với Course_Enrollments

### Entity: COURSE_CATEGORIES
**Fields:**
- id (UUID, Primary Key)
- name (VARCHAR, UNIQUE) - tên danh mục
- description (TEXT) - mô tả
- parent_category_id (UUID, Foreign Key → COURSE_CATEGORIES) - danh mục cha
- icon_url (VARCHAR) - URL icon
- color_code (VARCHAR) - mã màu
- is_active (BOOLEAN) - đang hoạt động
- sort_order (INT) - thứ tự sắp xếp
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Course_Category 1-N với Courses, N-1 với Course_Category (parent), 1-N với Course_Category (children)

### Entity: COURSE_MODULES
**Fields:**
- id (UUID, Primary Key)
- course_id (UUID, Foreign Key → COURSES) - liên kết khóa học
- module_title (VARCHAR) - tiêu đề module
- description (TEXT) - mô tả
- sequence_order (INT) - thứ tự
- duration_minutes (INT) - thời lượng (phút)
- is_preview (BOOLEAN) - xem trước
- learning_objectives (JSONB) - mục tiêu học tập
- resources (JSONB) - tài nguyên
- status (VARCHAR) - trạng thái
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Course_Module N-1 với Course, 1-N với Course_Lessons

### Entity: COURSE_LESSONS
**Fields:**
- id (UUID, Primary Key)
- module_id (UUID, Foreign Key → COURSE_MODULES) - liên kết module
- lesson_title (VARCHAR) - tiêu đề bài học
- description (TEXT) - mô tả
- sequence_order (INT) - thứ tự
- lesson_type (VARCHAR) - loại bài học
- content_url (VARCHAR) - URL nội dung
- video_url (VARCHAR) - URL video
- duration_minutes (INT) - thời lượng (phút)
- transcript (TEXT) - bản ghi
- attachments (JSONB) - file đính kèm
- quiz_questions (JSONB) - câu hỏi quiz
- is_preview (BOOLEAN) - xem trước
- completion_criteria (JSONB) - tiêu chí hoàn thành
- status (VARCHAR) - trạng thái
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Course_Lesson N-1 với Course_Module

### Entity: COURSE_ENROLLMENTS
**Fields:**
- id (UUID, Primary Key)
- course_id (UUID, Foreign Key → COURSES) - liên kết khóa học
- user_id (UUID, Foreign Key → USERS) - liên kết người dùng
- enrollment_date (TIMESTAMP) - ngày đăng ký
- start_date (TIMESTAMP) - ngày bắt đầu
- completion_date (TIMESTAMP) - ngày hoàn thành
- progress_percentage (DECIMAL) - phần trăm tiến độ
- current_module_id (UUID, Foreign Key → COURSE_MODULES) - module hiện tại
- current_lesson_id (UUID, Foreign Key → COURSE_LESSONS) - bài học hiện tại
- total_time_spent_minutes (INT) - tổng thời gian học (phút)
- quiz_attempts (INT) - số lần làm quiz
- best_quiz_score (DECIMAL) - điểm quiz tốt nhất
- final_score (DECIMAL) - điểm cuối
- certificate_issued (BOOLEAN) - đã cấp chứng chỉ
- certificate_url (VARCHAR) - URL chứng chỉ
- payment_status (VARCHAR) - trạng thái thanh toán
- payment_amount (DECIMAL) - số tiền thanh toán
- enrollment_source (VARCHAR) - nguồn đăng ký
- status (VARCHAR) - trạng thái
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Course_Enrollment N-1 với Course, N-1 với User, N-1 với Course_Module (current), N-1 với Course_Lesson (current), 1-N với Learning_Progress

### Entity: LEARNING_PROGRESS
**Fields:**
- id (UUID, Primary Key)
- enrollment_id (UUID, Foreign Key → COURSE_ENROLLMENTS) - liên kết đăng ký
- lesson_id (UUID, Foreign Key → COURSE_LESSONS) - liên kết bài học
- user_id (UUID, Foreign Key → USERS) - liên kết người dùng
- started_at (TIMESTAMP) - thời gian bắt đầu
- completed_at (TIMESTAMP) - thời gian hoàn thành
- time_spent_minutes (INT) - thời gian học (phút)
- progress_percentage (DECIMAL) - phần trăm tiến độ
- quiz_score (DECIMAL) - điểm quiz
- notes (TEXT) - ghi chú
- bookmarks (JSONB) - đánh dấu
- status (VARCHAR) - trạng thái
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Learning_Progress N-1 với Course_Enrollment, N-1 với Course_Lesson, N-1 với User

---

## 8. NOTIFICATION SYSTEM - Hệ thống thông báo

### Entity: NOTIFICATIONS
**Fields:**
- id (UUID, Primary Key)
- user_id (UUID, Foreign Key → USERS) - liên kết người dùng
- title (VARCHAR) - tiêu đề thông báo
- message (TEXT) - nội dung thông báo
- notification_type (VARCHAR) - loại thông báo
- priority (VARCHAR) - độ ưu tiên
- category (VARCHAR) - danh mục
- related_entity_type (VARCHAR) - loại entity liên quan
- related_entity_id (UUID) - ID entity liên quan
- action_url (VARCHAR) - URL hành động
- action_text (VARCHAR) - text hành động
- metadata (JSONB) - metadata
- is_read (BOOLEAN) - đã đọc
- read_at (TIMESTAMP) - thời gian đọc
- is_archived (BOOLEAN) - đã lưu trữ
- archived_at (TIMESTAMP) - thời gian lưu trữ
- expires_at (TIMESTAMP) - thời gian hết hạn
- delivery_channels (JSONB) - kênh gửi
- delivery_status (JSONB) - trạng thái gửi
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Notification N-1 với User

### Entity: NOTIFICATION_PREFERENCES
**Fields:**
- id (UUID, Primary Key)
- user_id (UUID, Foreign Key → USERS) - liên kết người dùng
- notification_type (VARCHAR) - loại thông báo
- email_enabled (BOOLEAN) - bật email
- sms_enabled (BOOLEAN) - bật SMS
- push_enabled (BOOLEAN) - bật push notification
- in_app_enabled (BOOLEAN) - bật thông báo trong app
- frequency (VARCHAR) - tần suất
- quiet_hours_start (TIME) - giờ bắt đầu im lặng
- quiet_hours_end (TIME) - giờ kết thúc im lặng
- timezone (VARCHAR) - múi giờ
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Notification_Preference N-1 với User

### Entity: NOTIFICATION_TEMPLATES
**Fields:**
- id (UUID, Primary Key)
- template_name (VARCHAR, UNIQUE) - tên template
- notification_type (VARCHAR) - loại thông báo
- channel (VARCHAR) - kênh gửi
- subject_template (VARCHAR) - template tiêu đề
- body_template (TEXT) - template nội dung
- variables (JSONB) - biến
- language (VARCHAR) - ngôn ngữ
- is_active (BOOLEAN) - đang hoạt động
- version (INT) - phiên bản
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Notification_Template 1-N với Notification_Logs

### Entity: NOTIFICATION_LOGS
**Fields:**
- id (UUID, Primary Key)
- notification_id (UUID, Foreign Key → NOTIFICATIONS) - liên kết thông báo
- template_id (UUID, Foreign Key → NOTIFICATION_TEMPLATES) - liên kết template
- user_id (UUID, Foreign Key → USERS) - liên kết người dùng
- channel (VARCHAR) - kênh gửi
- recipient (VARCHAR) - người nhận
- subject (VARCHAR) - tiêu đề
- content (TEXT) - nội dung
- delivery_status (VARCHAR) - trạng thái gửi
- sent_at (TIMESTAMP) - thời gian gửi
- delivered_at (TIMESTAMP) - thời gian giao
- opened_at (TIMESTAMP) - thời gian mở
- clicked_at (TIMESTAMP) - thời gian click
- error_message (TEXT) - thông báo lỗi
- retry_count (INT) - số lần thử lại
- external_id (VARCHAR) - ID bên ngoài
- cost (DECIMAL) - chi phí
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

**Relationship:** Notification_Log N-1 với Notification, N-1 với Notification_Template, N-1 với User

---

## 9. AUDIT SYSTEM - Hệ thống audit

### Entity: AUDIT_LOGS
**Fields:**
- id (UUID, Primary Key)
- user_id (UUID, Foreign Key → USERS) - liên kết người dùng
- action (VARCHAR) - hành động
- entity_type (VARCHAR) - loại entity
- entity_id (UUID) - ID entity
- old_values (JSONB) - giá trị cũ
- new_values (JSONB) - giá trị mới
- ip_address (VARCHAR) - địa chỉ IP
- user_agent (VARCHAR) - user agent
- session_id (VARCHAR) - ID session
- request_id (VARCHAR) - ID request
- additional_data (JSONB) - dữ liệu bổ sung
- created_at (TIMESTAMP)

**Relationship:** Audit_Log N-1 với User

---

## TÓM TẮT CÁC RELATIONSHIP CHÍNH:

### 1. Core System:
- **Role 1-N User**: Một role có thể có nhiều users

### 2. Business Management:
- **Business_Registration 1-1 Business**: Một đăng ký tương ứng một doanh nghiệp
- **Business 1-N Business_Contact**: Một doanh nghiệp có nhiều liên hệ
- **Business 1-N Business_Requirement**: Một doanh nghiệp có nhiều yêu cầu
- **Business_Requirement 1-N Requirement_Skill**: Một yêu cầu có nhiều kỹ năng
- **Business 1-N Contract**: Một doanh nghiệp có nhiều hợp đồng

### 3. Talent Management:
- **Talent_Application 1-1 Talent**: Một đơn ứng tuyển tương ứng một talent
- **User 1-1 Talent**: Một user tương ứng một talent
- **Talent 1-N Talent_Skill**: Một talent có nhiều kỹ năng
- **Skill 1-N Talent_Skill**: Một kỹ năng có nhiều talent
- **Talent 1-N Skill_Assessment**: Một talent có nhiều đánh giá kỹ năng

### 4. ODC Team Management:
- **Business 1-N Team_Formation_Request**: Một doanh nghiệp có nhiều yêu cầu thành lập team
- **Team_Formation_Request 1-1 ODC_Team**: Một yêu cầu tương ứng một team
- **ODC_Team 1-N Team_Member**: Một team có nhiều thành viên
- **Talent 1-N Team_Member**: Một talent có thể tham gia nhiều team

### 5. Project Management:
- **Business 1-N Project**: Một doanh nghiệp có nhiều dự án
- **ODC_Team 1-N Project**: Một team có thể làm nhiều dự án
- **Project 1-N Project_Phase**: Một dự án có nhiều giai đoạn
- **Project_Phase 1-N Project_Task**: Một giai đoạn có nhiều task

### 6. Marketplace System:
- **Business 1-N Marketplace_Project**: Một doanh nghiệp có nhiều dự án marketplace
- **Marketplace_Project 1-N Marketplace_Bid**: Một dự án có nhiều bid
- **Talent 1-N Marketplace_Bid**: Một talent có thể bid nhiều dự án
- **Marketplace_Project 1-1 Marketplace_Contract**: Một dự án có một hợp đồng

### 7. Learning Platform:
- **Course 1-N Course_Module**: Một khóa học có nhiều module
- **Course_Module 1-N Course_Lesson**: Một module có nhiều bài học
- **User 1-N Course_Enrollment**: Một user có thể đăng ký nhiều khóa học
- **Course_Enrollment 1-N Learning_Progress**: Một đăng ký có nhiều tiến độ học tập

### 8. Notification System:
- **User 1-N Notification**: Một user có nhiều thông báo
- **User 1-N Notification_Preference**: Một user có nhiều thiết lập thông báo
- **Notification_Template 1-N Notification_Log**: Một template có nhiều log

### 9. Audit System:
- **User 1-N Audit_Log**: Một user có nhiều log audit

---

## CÁC ENTITY CHÍNH VÀ SỐ LƯỢNG FIELD:

1. **ROLES** (6 fields): Quản lý vai trò hệ thống
2. **USERS** (9 fields): Quản lý người dùng
3. **BUSINESS_REGISTRATIONS** (21 fields): Đăng ký doanh nghiệp
4. **BUSINESSES** (24 fields): Thông tin doanh nghiệp
5. **BUSINESS_CONTACTS** (6 fields): Liên hệ doanh nghiệp
6. **BUSINESS_REQUIREMENTS** (9 fields): Yêu cầu doanh nghiệp
7. **REQUIREMENT_SKILLS** (5 fields): Kỹ năng yêu cầu
8. **CONTRACTS** (11 fields): Hợp đồng
9. **TALENT_APPLICATIONS** (26 fields): Đơn ứng tuyển
10. **TALENTS** (25 fields): Thông tin nhân tài
11. **SKILLS** (5 fields): Kỹ năng
12. **TALENT_SKILLS** (7 fields): Kỹ năng của talent
13. **SKILL_ASSESSMENTS** (15 fields): Đánh giá kỹ năng
14. **TRAINING_PROGRESS** (15 fields): Tiến độ đào tạo
15. **TALENT_ASSIGNMENTS** (13 fields): Phân công talent
16. **TALENT_EVALUATIONS** (16 fields): Đánh giá talent
17. **TEAM_FORMATION_REQUESTS** (19 fields): Yêu cầu thành lập team
18. **TEAM_ROLES** (8 fields): Vai trò trong team
19. **TEAM_WORKSPACES** (8 fields): Không gian làm việc
20. **ODC_TEAMS** (19 fields): Đội nhóm ODC
21. **TEAM_MEMBERS** (15 fields): Thành viên team
22. **TEAM_MENTORS** (11 fields): Mentor team
23. **TEAM_PERFORMANCE_METRICS** (15 fields): Chỉ số hiệu suất team
24. **TEAM_EQUIPMENT_ASSIGNMENTS** (10 fields): Phân bổ thiết bị team
25. **PROJECTS** (25 fields): Dự án
26. **PROJECT_PHASES** (15 fields): Giai đoạn dự án
27. **PROJECT_TASKS** (25 fields): Nhiệm vụ dự án
28. **PROJECT_MILESTONES** (9 fields): Cột mốc dự án
29. **MARKETPLACE_PROJECTS** (25 fields): Dự án marketplace
30. **PROJECT_CATEGORIES** (8 fields): Danh mục dự án
31. **MARKETPLACE_BIDS** (19 fields): Đấu thầu marketplace
32. **PROJECT_PROPOSALS** (17 fields): Đề xuất dự án
33. **MARKETPLACE_CONTRACTS** (21 fields): Hợp đồng marketplace
34. **MARKETPLACE_MILESTONES** (10 fields): Cột mốc marketplace
35. **ESCROW_PAYMENTS** (13 fields): Thanh toán ký quỹ
36. **MARKETPLACE_REVIEWS** (17 fields): Đánh giá marketplace
37. **COURSES** (25 fields): Khóa học
38. **COURSE_CATEGORIES** (8 fields): Danh mục khóa học
39. **COURSE_MODULES** (10 fields): Module khóa học
40. **COURSE_LESSONS** (15 fields): Bài học
41. **COURSE_ENROLLMENTS** (21 fields): Đăng ký khóa học
42. **LEARNING_PROGRESS** (12 fields): Tiến độ học tập
43. **NOTIFICATIONS** (19 fields): Thông báo
44. **NOTIFICATION_PREFERENCES** (11 fields): Thiết lập thông báo
45. **NOTIFICATION_TEMPLATES** (10 fields): Template thông báo
46. **NOTIFICATION_LOGS** (17 fields): Log thông báo
47. **AUDIT_LOGS** (12 fields): Log audit

**TỔNG CỘNG: 47 entities với tổng cộng 647 fields**

---

## DATABASE DISTRIBUTION:

### PostgreSQL (Primary Database):
- Tất cả các entity chính của hệ thống
- Quản lý dữ liệu có cấu trúc và quan hệ phức tạp
- Đảm bảo tính toàn vẹn dữ liệu với foreign keys

### MongoDB (Document Store):
- Lưu trữ dữ liệu phi cấu trúc như documents, files
- Content management cho courses, projects
- Flexible schema cho metadata

### Elasticsearch (Search Engine):
- Full-text search cho projects, talents, courses
- Analytics và reporting
- Real-time search suggestions

---

*Schema này mô tả đầy đủ cấu trúc database của ODC Platform với 47 entities, 647 fields và các mối quan hệ phức tạp giữa các thành phần trong hệ thống.*