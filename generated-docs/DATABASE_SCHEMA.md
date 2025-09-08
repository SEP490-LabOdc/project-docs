# Database Schema cho Lab-based ODC Platform

## 1. Tổng quan

Dựa trên phân tích các main flows và kiến trúc hệ thống, database schema được thiết kế để hỗ trợ nền tảng Lab-based ODC với các thành phần chính sau:

- Quản lý người dùng và phân quyền
- Quản lý Talent Pool và kỹ năng
- Quản lý doanh nghiệp và yêu cầu
- Quản lý ODC Team và dự án
- Hệ thống ghép nối dựa trên kỹ năng
- Marketplace cho các dự án mini
- Nền tảng học tập tương tác
- Audit trail và logging
- Notification system

### Cải tiến so với phiên bản trước:
- Bổ sung audit tables cho tracking thay đổi
- Thêm notification system
- Cải thiện indexing và constraints
- Tối ưu hóa cấu trúc MongoDB collections
- ERD được vẽ bằng ASCII art thay vì PlantUML

## 2. Lựa chọn cơ sở dữ liệu

### PostgreSQL
Dùng cho dữ liệu quan hệ, cần tính toàn vẹn cao:
- Thông tin người dùng và phân quyền
- Thông tin doanh nghiệp và hợp đồng
- Quản lý dự án và ODC Team
- Quản lý tài chính và thanh toán

### MongoDB
Dùng cho dữ liệu phi cấu trúc hoặc cần linh hoạt:
- Hồ sơ cá nhân của Talent
- Feedback và đánh giá
- Log hệ thống
- Nội dung khóa học và tài liệu học tập

### Elasticsearch
Dùng cho các chức năng tìm kiếm toàn văn và phân tích dữ liệu:
- Tìm kiếm tài năng theo kỹ năng
- Tìm kiếm dự án và khóa học
- Phân tích dữ liệu và báo cáo

## 3. Database Schema (PostgreSQL)

### Users Schema

```sql
-- Roles Table
CREATE TABLE roles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(50) NOT NULL UNIQUE,
    description TEXT,
    permissions JSONB, -- Store permissions as JSON array
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Users Table with direct role reference (1-N relationship)
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    full_name VARCHAR(255) NOT NULL,
    phone VARCHAR(20),
    role_id UUID NOT NULL REFERENCES roles(id),
    status VARCHAR(20) NOT NULL DEFAULT 'active',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Insert default roles
INSERT INTO roles (name, description, permissions) VALUES 
('admin', 'System Administrator', '["user:read", "user:write", "user:delete", "business:read", "business:write", "business:delete", "talent:read", "talent:write", "talent:delete", "project:read", "project:write", "project:delete", "system:admin"]'),
('business_owner', 'Business Owner', '["user:read", "business:read", "business:write", "talent:read", "project:read", "project:write", "contract:read", "contract:write"]'),
('business_manager', 'Business Manager', '["user:read", "business:read", "talent:read", "project:read", "project:write"]'),
('talent', 'Talent/Developer', '["user:read", "user:write_own", "talent:read", "talent:write_own", "project:read", "skill:read", "course:read", "assessment:read"]'),
('team_lead', 'Team Lead', '["user:read", "talent:read", "project:read", "project:write", "team:read", "team:write"]'),
('hr_manager', 'HR Manager', '["user:read", "talent:read", "talent:write", "skill:read", "assessment:read", "training:read"]'),
('instructor', 'Course Instructor', '["user:read", "course:read", "course:write", "assessment:read", "assessment:write", "student:read"]'),
('student', 'Course Student', '["user:read", "user:write_own", "course:read", "assessment:read", "progress:read"]');
```

### Businesses Schema

-- Business Registration Table for FLOW1 Support
```sql
CREATE TABLE business_registrations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    business_name VARCHAR(255) NOT NULL,
    business_email VARCHAR(255) NOT NULL,
    contact_person_name VARCHAR(255) NOT NULL,
    contact_person_email VARCHAR(255) NOT NULL,
    contact_person_phone VARCHAR(20),
    industry VARCHAR(100),
    company_size VARCHAR(50),
    website VARCHAR(255),
    business_license VARCHAR(255),
    tax_id VARCHAR(100),
    address TEXT,
    city VARCHAR(100),
    country VARCHAR(100),
    registration_documents JSONB,
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    submitted_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    reviewed_at TIMESTAMP,
    reviewed_by UUID REFERENCES users(id),
    rejection_reason TEXT,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Enhanced Businesses Table
CREATE TABLE businesses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    registration_id UUID REFERENCES business_registrations(id),
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL,
    phone VARCHAR(20) NOT NULL,
    address TEXT,
    industry VARCHAR(100),
    size VARCHAR(50),
    website VARCHAR(255),
    business_license VARCHAR(255),
    tax_id VARCHAR(100),
    logo_url VARCHAR(500),
    description TEXT,
    verification_status VARCHAR(20) DEFAULT 'pending',
    subscription_plan VARCHAR(50) DEFAULT 'basic',
    subscription_expires_at TIMESTAMP,
    billing_address TEXT,
    billing_contact_email VARCHAR(255),
    payment_method_id VARCHAR(255),
    lab_setup_completed BOOLEAN DEFAULT false,
    onboarding_completed BOOLEAN DEFAULT false,
    status VARCHAR(20) NOT NULL DEFAULT 'active',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE business_contacts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    business_id UUID NOT NULL REFERENCES businesses(id),
    user_id UUID NOT NULL REFERENCES users(id),
    position VARCHAR(100),
    is_primary BOOLEAN NOT NULL DEFAULT false,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE business_requirements (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    business_id UUID NOT NULL REFERENCES businesses(id),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    team_size INT,
    duration VARCHAR(50),
    contract_type VARCHAR(50),
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE requirement_skills (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    requirement_id UUID NOT NULL REFERENCES business_requirements(id),
    skill_id UUID NOT NULL REFERENCES skills(id),
    experience_level VARCHAR(50),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE contracts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    business_id UUID NOT NULL REFERENCES businesses(id),
    requirement_id UUID REFERENCES business_requirements(id),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    start_date DATE NOT NULL,
    end_date DATE,
    contract_type VARCHAR(50) NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'draft',
    file_path VARCHAR(255),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### Talents Schema

-- Talent Applications Table for FLOW2 Support
```sql
CREATE TABLE talent_applications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    applicant_name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL,
    phone VARCHAR(20),
    education_level VARCHAR(100),
    university VARCHAR(255),
    graduation_year INT,
    major VARCHAR(255),
    gpa DECIMAL(3,2),
    years_of_experience INT DEFAULT 0,
    current_position VARCHAR(255),
    current_company VARCHAR(255),
    resume_url VARCHAR(500),
    portfolio_url VARCHAR(500),
    cover_letter TEXT,
    preferred_technologies JSONB,
    expected_salary_min DECIMAL(12,2),
    expected_salary_max DECIMAL(12,2),
    availability_date DATE,
    application_source VARCHAR(100),
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    submitted_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    reviewed_at TIMESTAMP,
    reviewed_by UUID REFERENCES users(id),
    interview_scheduled_at TIMESTAMP,
    rejection_reason TEXT,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Enhanced Talents Table
CREATE TABLE talents (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    application_id UUID REFERENCES talent_applications(id),
    user_id UUID NOT NULL REFERENCES users(id),
    employee_id VARCHAR(50) UNIQUE,
    education_level VARCHAR(100),
    university VARCHAR(255),
    graduation_year INT,
    major VARCHAR(255),
    gpa DECIMAL(3,2),
    years_of_experience INT,
    current_position VARCHAR(255),
    department VARCHAR(100),
    manager_id UUID REFERENCES users(id),
    hire_date DATE,
    salary DECIMAL(12,2),
    performance_rating DECIMAL(3,1),
    availability VARCHAR(50) NOT NULL DEFAULT 'available',
    location VARCHAR(255),
    timezone VARCHAR(50),
    remote_work_preference VARCHAR(20) DEFAULT 'hybrid',
    linkedin_url VARCHAR(500),
    github_url VARCHAR(500),
    portfolio_url VARCHAR(500),
    bio TEXT,
    status VARCHAR(20) NOT NULL DEFAULT 'active',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE skills (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL UNIQUE,
    category VARCHAR(50) NOT NULL,
    description TEXT,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE talent_skills (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    talent_id UUID NOT NULL REFERENCES talents(id),
    skill_id UUID NOT NULL REFERENCES skills(id),
    proficiency_level INT NOT NULL,
    years_of_experience DECIMAL(4,1),
    verified BOOLEAN NOT NULL DEFAULT false,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(talent_id, skill_id)
);

-- Skill Assessments Table for FLOW2 Support
CREATE TABLE skill_assessments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    talent_id UUID NOT NULL REFERENCES talents(id),
    skill_id UUID NOT NULL REFERENCES skills(id),
    assessment_type VARCHAR(50) NOT NULL, -- 'technical', 'behavioral', 'project'
    assessment_date DATE NOT NULL,
    score DECIMAL(5,2) NOT NULL,
    max_score DECIMAL(5,2) NOT NULL,
    percentage DECIMAL(5,2) GENERATED ALWAYS AS (score * 100.0 / max_score) STORED,
    assessor_id UUID REFERENCES users(id),
    assessment_details JSONB,
    feedback TEXT,
    certification_earned VARCHAR(255),
    valid_until DATE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Training Progress Table for FLOW2 Support
CREATE TABLE training_progress (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    talent_id UUID NOT NULL REFERENCES talents(id),
    course_id UUID NOT NULL,
    course_name VARCHAR(255) NOT NULL,
    training_type VARCHAR(50) NOT NULL, -- 'internal', 'external', 'certification'
    status VARCHAR(20) NOT NULL DEFAULT 'enrolled', -- 'enrolled', 'in_progress', 'completed', 'failed'
    start_date DATE,
    completion_date DATE,
    progress_percentage DECIMAL(5,2) DEFAULT 0,
    final_score DECIMAL(5,2),
    certification_obtained VARCHAR(255),
    trainer_id UUID REFERENCES users(id),
    training_hours DECIMAL(5,1),
    cost DECIMAL(10,2),
    notes TEXT,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Talent Assignments Table for FLOW2 Support
CREATE TABLE talent_assignments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    talent_id UUID NOT NULL REFERENCES talents(id),
    project_id UUID,
    team_id UUID,
    assignment_type VARCHAR(50) NOT NULL, -- 'project', 'team', 'training', 'bench'
    role VARCHAR(100),
    start_date DATE NOT NULL,
    end_date DATE,
    allocation_percentage DECIMAL(5,2) DEFAULT 100.00,
    hourly_rate DECIMAL(10,2),
    status VARCHAR(20) NOT NULL DEFAULT 'active',
    assigned_by UUID REFERENCES users(id),
    notes TEXT,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE talent_evaluations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    talent_id UUID NOT NULL REFERENCES talents(id),
    evaluator_id UUID NOT NULL REFERENCES users(id),
    evaluation_period VARCHAR(20) NOT NULL, -- 'quarterly', 'annual', 'project_end'
    evaluation_date DATE NOT NULL,
    overall_score DECIMAL(3,1) NOT NULL,
    technical_score DECIMAL(3,1),
    communication_score DECIMAL(3,1),
    teamwork_score DECIMAL(3,1),
    leadership_score DECIMAL(3,1),
    problem_solving_score DECIMAL(3,1),
    strengths TEXT,
    areas_for_improvement TEXT,
    goals_next_period TEXT,
    comments TEXT,
    status VARCHAR(20) DEFAULT 'draft',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### ODC Teams Schema

-- Team Formation Requests Table for FLOW3 Support
```sql
CREATE TABLE team_formation_requests (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    business_id UUID NOT NULL REFERENCES businesses(id),
    request_title VARCHAR(255) NOT NULL,
    project_description TEXT,
    team_size INT NOT NULL,
    required_skills JSONB,
    experience_level VARCHAR(50), -- 'junior', 'mid', 'senior', 'mixed'
    project_duration VARCHAR(50),
    start_date DATE,
    budget_range VARCHAR(100),
    work_arrangement VARCHAR(50), -- 'onsite', 'remote', 'hybrid'
    location VARCHAR(255),
    special_requirements TEXT,
    priority VARCHAR(20) DEFAULT 'medium',
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    submitted_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    reviewed_at TIMESTAMP,
    reviewed_by UUID REFERENCES users(id),
    approval_notes TEXT,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Team Roles Table for FLOW3 Support
CREATE TABLE team_roles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    role_name VARCHAR(100) NOT NULL UNIQUE,
    role_description TEXT,
    required_skills JSONB,
    experience_level VARCHAR(50),
    responsibilities TEXT,
    is_leadership_role BOOLEAN DEFAULT false,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Team Workspaces Table for FLOW3 Support
CREATE TABLE team_workspaces (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    workspace_name VARCHAR(255) NOT NULL,
    location VARCHAR(255),
    capacity INT,
    facilities JSONB, -- equipment, tools, etc.
    availability_status VARCHAR(20) DEFAULT 'available',
    cost_per_month DECIMAL(10,2),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Enhanced ODC Teams Table
CREATE TABLE odc_teams (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    formation_request_id UUID REFERENCES team_formation_requests(id),
    name VARCHAR(100) NOT NULL,
    business_id UUID NOT NULL REFERENCES businesses(id),
    contract_id UUID REFERENCES contracts(id),
    team_lead_id UUID REFERENCES talents(id),
    workspace_id UUID REFERENCES team_workspaces(id),
    team_type VARCHAR(50), -- 'development', 'qa', 'devops', 'mixed'
    methodology VARCHAR(50), -- 'agile', 'waterfall', 'kanban'
    max_capacity INT DEFAULT 10,
    current_size INT DEFAULT 0,
    formation_date DATE,
    planned_end_date DATE,
    actual_end_date DATE,
    performance_rating DECIMAL(3,1),
    budget_allocated DECIMAL(12,2),
    budget_spent DECIMAL(12,2),
    status VARCHAR(20) NOT NULL DEFAULT 'forming',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Enhanced Team Members Table for FLOW3 Support
CREATE TABLE team_members (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    team_id UUID NOT NULL REFERENCES odc_teams(id),
    talent_id UUID NOT NULL REFERENCES talents(id),
    role_id UUID REFERENCES team_roles(id),
    role VARCHAR(50) NOT NULL,
    is_team_lead BOOLEAN DEFAULT false,
    allocation_percentage INT DEFAULT 100, -- % of time allocated to this team
    hourly_rate DECIMAL(8,2),
    join_date DATE NOT NULL,
    end_date DATE,
    performance_rating DECIMAL(3,1),
    last_evaluation_date DATE,
    notes TEXT,
    status VARCHAR(20) NOT NULL DEFAULT 'active',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Enhanced Team Mentors Table for FLOW3 Support
CREATE TABLE team_mentors (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    team_id UUID NOT NULL REFERENCES odc_teams(id),
    mentor_id UUID NOT NULL REFERENCES users(id),
    mentorship_type VARCHAR(50), -- 'technical', 'soft_skills', 'leadership'
    specialization VARCHAR(100),
    start_date DATE NOT NULL,
    end_date DATE,
    hours_per_week INT DEFAULT 5,
    mentorship_goals TEXT,
    progress_notes TEXT,
    status VARCHAR(20) DEFAULT 'active',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Team Performance Metrics Table for FLOW3 Support
CREATE TABLE team_performance_metrics (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    team_id UUID NOT NULL REFERENCES odc_teams(id),
    metric_date DATE NOT NULL,
    productivity_score DECIMAL(3,1),
    quality_score DECIMAL(3,1),
    collaboration_score DECIMAL(3,1),
    client_satisfaction DECIMAL(3,1),
    delivery_timeliness DECIMAL(3,1),
    budget_efficiency DECIMAL(5,2), -- actual vs planned budget %
    team_velocity INT, -- story points or tasks completed
    defect_rate DECIMAL(5,2),
    knowledge_sharing_score DECIMAL(3,1),
    innovation_score DECIMAL(3,1),
    notes TEXT,
    recorded_by UUID REFERENCES users(id),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Team Equipment Assignments Table for FLOW3 Support
CREATE TABLE team_equipment_assignments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    team_id UUID NOT NULL REFERENCES odc_teams(id),
    equipment_type VARCHAR(100), -- 'laptop', 'monitor', 'software_license'
    equipment_name VARCHAR(255),
    serial_number VARCHAR(100),
    assigned_date DATE NOT NULL,
    return_date DATE,
    condition_notes TEXT,
    cost DECIMAL(10,2),
    status VARCHAR(20) DEFAULT 'assigned',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### Projects Schema

-- Enhanced Projects Table for FLOW4 Support
```sql
CREATE TABLE projects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    business_id UUID NOT NULL REFERENCES businesses(id),
    team_id UUID REFERENCES odc_teams(id),
    project_manager_id UUID REFERENCES users(id),
    description TEXT,
    objectives TEXT,
    scope TEXT,
    deliverables JSONB,
    start_date DATE NOT NULL,
    end_date DATE,
    actual_start_date DATE,
    actual_end_date DATE,
    budget DECIMAL(12,2),
    actual_cost DECIMAL(12,2),
    currency VARCHAR(10) DEFAULT 'USD',
    priority VARCHAR(20) NOT NULL DEFAULT 'medium',
    methodology VARCHAR(50), -- 'agile', 'waterfall', 'kanban'
    progress_percentage DECIMAL(5,2) DEFAULT 0,
    health_status VARCHAR(20) DEFAULT 'green', -- 'green', 'yellow', 'red'
    risk_level VARCHAR(20) DEFAULT 'low',
    client_satisfaction DECIMAL(3,1),
    status VARCHAR(20) NOT NULL DEFAULT 'planning',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Project Phases Table for FLOW4 Support
CREATE TABLE project_phases (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL REFERENCES projects(id),
    phase_name VARCHAR(255) NOT NULL,
    description TEXT,
    sequence_order INT NOT NULL,
    start_date DATE,
    end_date DATE,
    actual_start_date DATE,
    actual_end_date DATE,
    budget DECIMAL(10,2),
    actual_cost DECIMAL(10,2),
    progress_percentage DECIMAL(5,2) DEFAULT 0,
    deliverables JSONB,
    acceptance_criteria TEXT,
    status VARCHAR(20) NOT NULL DEFAULT 'not_started',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Enhanced Project Tasks Table for FLOW4 Support
CREATE TABLE project_tasks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL REFERENCES projects(id),
    phase_id UUID REFERENCES project_phases(id),
    parent_task_id UUID REFERENCES project_tasks(id),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    assignee_id UUID REFERENCES users(id),
    reporter_id UUID REFERENCES users(id),
    start_date DATE,
    due_date DATE,
    actual_start_date DATE,
    actual_completion_date DATE,
    estimated_hours DECIMAL(6,2),
    actual_hours DECIMAL(6,2),
    progress_percentage DECIMAL(5,2) DEFAULT 0,
    story_points INT,
    task_type VARCHAR(50), -- 'feature', 'bug', 'improvement', 'task'
    status VARCHAR(20) NOT NULL DEFAULT 'todo',
    priority VARCHAR(20) NOT NULL DEFAULT 'medium',
    labels JSONB, -- tags for categorization
    acceptance_criteria TEXT,
    dependencies JSONB, -- task dependencies
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Project Milestones Table for FLOW4 Support
CREATE TABLE project_milestones (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL REFERENCES projects(id),
    milestone_name VARCHAR(255) NOT NULL,
    description TEXT,
    target_date DATE NOT NULL,
    actual_date DATE,
    deliverables JSONB,
    success_criteria TEXT,
    is_critical BOOLEAN DEFAULT false,
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Project Resources Table for FLOW4 Support
CREATE TABLE project_resources (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL REFERENCES projects(id),
    resource_type VARCHAR(50), -- 'human', 'equipment', 'software', 'budget'
    resource_name VARCHAR(255) NOT NULL,
    resource_id UUID, -- reference to talent, equipment, etc.
    allocation_percentage DECIMAL(5,2),
    cost_per_unit DECIMAL(10,2),
    quantity DECIMAL(8,2),
    total_cost DECIMAL(12,2),
    start_date DATE,
    end_date DATE,
    status VARCHAR(20) DEFAULT 'allocated',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Enhanced Project Reports Table for FLOW4 Support
CREATE TABLE project_reports (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL REFERENCES projects(id),
    reporter_id UUID NOT NULL REFERENCES users(id),
    report_type VARCHAR(50), -- 'daily', 'weekly', 'monthly', 'milestone', 'final'
    report_date DATE NOT NULL,
    reporting_period_start DATE,
    reporting_period_end DATE,
    title VARCHAR(255),
    executive_summary TEXT,
    achievements TEXT,
    challenges TEXT,
    risks_identified TEXT,
    next_steps TEXT,
    budget_status TEXT,
    timeline_status TEXT,
    quality_metrics JSONB,
    kpi_metrics JSONB,
    attachments JSONB,
    status VARCHAR(20) NOT NULL DEFAULT 'draft',
    submitted_at TIMESTAMP,
    approved_at TIMESTAMP,
    approved_by UUID REFERENCES users(id),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Project Risks Table for FLOW4 Support
CREATE TABLE project_risks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL REFERENCES projects(id),
    risk_title VARCHAR(255) NOT NULL,
    risk_description TEXT,
    risk_category VARCHAR(100), -- 'technical', 'resource', 'schedule', 'budget', 'external'
    probability VARCHAR(20), -- 'low', 'medium', 'high'
    impact VARCHAR(20), -- 'low', 'medium', 'high'
    risk_score DECIMAL(3,1), -- calculated score
    mitigation_strategy TEXT,
    contingency_plan TEXT,
    owner_id UUID REFERENCES users(id),
    identified_date DATE NOT NULL,
    target_resolution_date DATE,
    actual_resolution_date DATE,
    status VARCHAR(20) NOT NULL DEFAULT 'identified',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Project Communications Table for FLOW4 Support
CREATE TABLE project_communications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL REFERENCES projects(id),
    communication_type VARCHAR(50), -- 'meeting', 'email', 'report', 'presentation'
    subject VARCHAR(255) NOT NULL,
    content TEXT,
    sender_id UUID NOT NULL REFERENCES users(id),
    recipients JSONB, -- array of user IDs
    communication_date TIMESTAMP NOT NULL,
    meeting_duration INT, -- minutes
    action_items JSONB,
    attachments JSONB,
    status VARCHAR(20) DEFAULT 'sent',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Indexes for better performance
CREATE INDEX idx_projects_business_id ON projects(business_id);
CREATE INDEX idx_projects_team_id ON projects(team_id);
CREATE INDEX idx_projects_status ON projects(status);
CREATE INDEX idx_projects_dates ON projects(start_date, end_date);
CREATE INDEX idx_project_phases_project_id ON project_phases(project_id);
CREATE INDEX idx_project_tasks_project_id ON project_tasks(project_id);
CREATE INDEX idx_project_tasks_assignee ON project_tasks(assignee_id);
CREATE INDEX idx_project_tasks_status ON project_tasks(status);
CREATE INDEX idx_project_milestones_project_id ON project_milestones(project_id);
CREATE INDEX idx_project_resources_project_id ON project_resources(project_id);
CREATE INDEX idx_project_reports_project_id ON project_reports(project_id);
CREATE INDEX idx_project_risks_project_id ON project_risks(project_id);
CREATE INDEX idx_project_communications_project_id ON project_communications(project_id);
```

### Marketplace Schema

-- Enhanced Marketplace Projects for FLOW6 Support
```sql
CREATE TABLE marketplace_projects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    business_id UUID NOT NULL REFERENCES businesses(id),
    category_id UUID REFERENCES project_categories(id),
    title VARCHAR(255) NOT NULL,
    description TEXT NOT NULL,
    required_skills JSONB, -- skill requirements
    budget_min DECIMAL(10,2),
    budget_max DECIMAL(10,2),
    currency VARCHAR(3) DEFAULT 'USD',
    duration_days INT,
    complexity_level VARCHAR(20), -- 'simple', 'medium', 'complex'
    status VARCHAR(20) NOT NULL DEFAULT 'draft', -- 'draft', 'open', 'in_progress', 'completed', 'cancelled'
    posting_date TIMESTAMP,
    deadline DATE,
    application_deadline DATE,
    featured BOOLEAN DEFAULT false,
    view_count INT DEFAULT 0,
    application_count INT DEFAULT 0,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Project Categories for FLOW6 Support
CREATE TABLE project_categories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    description TEXT,
    parent_category_id UUID REFERENCES project_categories(id),
    icon VARCHAR(50),
    color VARCHAR(7), -- hex color code
    skill_tags JSONB,
    average_budget_range JSONB, -- {"min": 1000, "max": 5000}
    typical_duration INT, -- days
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Enhanced Project Skills for FLOW6 Support
CREATE TABLE marketplace_project_skills (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL REFERENCES marketplace_projects(id),
    skill_id UUID NOT NULL REFERENCES skills(id),
    required_level VARCHAR(20), -- 'beginner', 'intermediate', 'advanced', 'expert'
    is_mandatory BOOLEAN DEFAULT true,
    weight DECIMAL(3,2), -- importance weight 0.0-1.0
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(project_id, skill_id)
);

-- Enhanced Project Bids for FLOW6 Support
CREATE TABLE marketplace_bids (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL REFERENCES marketplace_projects(id),
    talent_id UUID REFERENCES talents(id),
    team_id UUID REFERENCES odc_teams(id),
    bid_amount DECIMAL(10,2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'USD',
    proposed_duration INT, -- days
    bid_description TEXT NOT NULL,
    portfolio_links JSONB,
    availability_start DATE,
    confidence_level VARCHAR(20), -- 'low', 'medium', 'high'
    status VARCHAR(20) NOT NULL DEFAULT 'pending', -- 'pending', 'accepted', 'rejected', 'withdrawn'
    submitted_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CHECK ((talent_id IS NOT NULL AND team_id IS NULL) OR (talent_id IS NULL AND team_id IS NOT NULL))
);

-- Project Proposals for FLOW6 Support
CREATE TABLE project_proposals (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    bid_id UUID NOT NULL REFERENCES marketplace_bids(id),
    detailed_approach TEXT,
    timeline_breakdown JSONB, -- detailed timeline
    deliverables JSONB, -- list of deliverables
    additional_services JSONB,
    terms_conditions TEXT,
    revision_policy TEXT,
    communication_plan TEXT,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Marketplace Contracts for FLOW6 Support
CREATE TABLE marketplace_contracts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL REFERENCES marketplace_projects(id),
    business_id UUID NOT NULL REFERENCES businesses(id),
    talent_id UUID REFERENCES talents(id),
    team_id UUID REFERENCES odc_teams(id),
    contract_amount DECIMAL(10,2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'USD',
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    milestones JSONB, -- milestone definitions
    payment_schedule JSONB, -- payment timeline
    terms JSONB, -- contract terms
    status VARCHAR(20) NOT NULL DEFAULT 'draft', -- 'draft', 'active', 'completed', 'terminated', 'disputed'
    signed_date TIMESTAMP,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CHECK ((talent_id IS NOT NULL AND team_id IS NULL) OR (talent_id IS NULL AND team_id IS NOT NULL))
);

-- Marketplace Contract Milestones for FLOW6 Support
CREATE TABLE marketplace_milestones (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    contract_id UUID NOT NULL REFERENCES marketplace_contracts(id),
    milestone_name VARCHAR(255) NOT NULL,
    description TEXT,
    due_date DATE NOT NULL,
    completion_date DATE,
    payment_amount DECIMAL(10,2),
    status VARCHAR(20) NOT NULL DEFAULT 'pending', -- 'pending', 'in_progress', 'completed', 'overdue'
    deliverables JSONB,
    approval_status VARCHAR(20) DEFAULT 'pending', -- 'pending', 'approved', 'rejected', 'revision_requested'
    feedback TEXT,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Escrow Payments for FLOW6 Support
CREATE TABLE escrow_payments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    contract_id UUID NOT NULL REFERENCES marketplace_contracts(id),
    milestone_id UUID REFERENCES marketplace_milestones(id),
    amount DECIMAL(10,2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'USD',
    status VARCHAR(20) NOT NULL DEFAULT 'pending', -- 'pending', 'deposited', 'released', 'refunded', 'disputed'
    deposited_date TIMESTAMP,
    released_date TIMESTAMP,
    dispute_status VARCHAR(20), -- 'none', 'raised', 'in_review', 'resolved'
    transaction_id VARCHAR(255),
    platform_fee DECIMAL(10,2),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Marketplace Reviews for FLOW6 Support
CREATE TABLE marketplace_reviews (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    contract_id UUID NOT NULL REFERENCES marketplace_contracts(id),
    reviewer_id UUID NOT NULL REFERENCES users(id),
    reviewee_id UUID NOT NULL REFERENCES users(id),
    rating DECIMAL(2,1) NOT NULL CHECK (rating >= 1.0 AND rating <= 5.0),
    review_text TEXT,
    review_categories JSONB, -- {"communication": 5, "quality": 4, "timeliness": 5}
    is_public BOOLEAN DEFAULT true,
    response_text TEXT,
    response_date TIMESTAMP,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Marketplace Indexes
CREATE INDEX idx_marketplace_projects_business_id ON marketplace_projects(business_id);
CREATE INDEX idx_marketplace_projects_category_id ON marketplace_projects(category_id);
CREATE INDEX idx_marketplace_projects_status ON marketplace_projects(status);
CREATE INDEX idx_marketplace_projects_budget ON marketplace_projects(budget_min, budget_max);
CREATE INDEX idx_marketplace_projects_deadline ON marketplace_projects(deadline);
CREATE INDEX idx_marketplace_projects_featured ON marketplace_projects(featured);
CREATE INDEX idx_project_categories_parent ON project_categories(parent_category_id);
CREATE INDEX idx_marketplace_bids_project_id ON marketplace_bids(project_id);
CREATE INDEX idx_marketplace_bids_talent_id ON marketplace_bids(talent_id);
CREATE INDEX idx_marketplace_bids_team_id ON marketplace_bids(team_id);
CREATE INDEX idx_marketplace_bids_status ON marketplace_bids(status);
CREATE INDEX idx_marketplace_contracts_project_id ON marketplace_contracts(project_id);
CREATE INDEX idx_marketplace_contracts_business_id ON marketplace_contracts(business_id);
CREATE INDEX idx_marketplace_contracts_talent_id ON marketplace_contracts(talent_id);
CREATE INDEX idx_marketplace_contracts_team_id ON marketplace_contracts(team_id);
CREATE INDEX idx_marketplace_contracts_status ON marketplace_contracts(status);
CREATE INDEX idx_marketplace_milestones_contract_id ON marketplace_milestones(contract_id);
CREATE INDEX idx_marketplace_milestones_status ON marketplace_milestones(status);
CREATE INDEX idx_escrow_payments_contract_id ON escrow_payments(contract_id);
CREATE INDEX idx_escrow_payments_status ON escrow_payments(status);
CREATE INDEX idx_marketplace_reviews_contract_id ON marketplace_reviews(contract_id);
CREATE INDEX idx_marketplace_reviews_reviewer_id ON marketplace_reviews(reviewer_id);
CREATE INDEX idx_marketplace_reviews_reviewee_id ON marketplace_reviews(reviewee_id);
```

### Learning Platform Schema

-- Enhanced Courses for FLOW7 Support
```sql
CREATE TABLE courses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    category_id UUID REFERENCES course_categories(id),
    instructor_id UUID REFERENCES users(id),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    short_description VARCHAR(500),
    level VARCHAR(50) NOT NULL, -- 'beginner', 'intermediate', 'advanced', 'expert'
    duration_hours DECIMAL(5,2),
    estimated_completion_time INT, -- in days
    language VARCHAR(10) DEFAULT 'en',
    price DECIMAL(8,2) DEFAULT 0,
    currency VARCHAR(3) DEFAULT 'USD',
    thumbnail_url VARCHAR(500),
    preview_video_url VARCHAR(500),
    learning_objectives JSONB,
    prerequisites JSONB,
    target_audience TEXT,
    difficulty_score DECIMAL(3,1), -- 1.0-10.0
    rating DECIMAL(2,1) DEFAULT 0, -- average rating
    total_ratings INT DEFAULT 0,
    enrollment_count INT DEFAULT 0,
    completion_rate DECIMAL(5,2) DEFAULT 0,
    status VARCHAR(20) NOT NULL DEFAULT 'draft', -- 'draft', 'published', 'archived', 'under_review'
    is_featured BOOLEAN DEFAULT false,
    tags JSONB,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Course Categories for FLOW7 Support
CREATE TABLE course_categories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    description TEXT,
    parent_category_id UUID REFERENCES course_categories(id),
    icon VARCHAR(50),
    color VARCHAR(7), -- hex color
    sort_order INT DEFAULT 0,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Enhanced Course Modules for FLOW7 Support
CREATE TABLE course_modules (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    course_id UUID NOT NULL REFERENCES courses(id),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    sequence_order INT NOT NULL,
    duration_minutes INT,
    is_mandatory BOOLEAN DEFAULT true,
    unlock_criteria JSONB, -- conditions to unlock this module
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Course Lessons for FLOW7 Support
CREATE TABLE course_lessons (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    module_id UUID NOT NULL REFERENCES course_modules(id),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    content_type VARCHAR(50), -- 'video', 'text', 'quiz', 'assignment', 'interactive'
    content_url VARCHAR(500),
    content_data JSONB, -- lesson content/configuration
    duration_minutes INT,
    sequence_order INT NOT NULL,
    is_mandatory BOOLEAN DEFAULT true,
    points_value INT DEFAULT 0,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Enhanced Course Enrollments for FLOW7 Support
CREATE TABLE course_enrollments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    course_id UUID NOT NULL REFERENCES courses(id),
    user_id UUID NOT NULL REFERENCES users(id),
    enrollment_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    start_date TIMESTAMP,
    completion_date TIMESTAMP,
    last_accessed TIMESTAMP,
    progress_percentage DECIMAL(5,2) DEFAULT 0,
    total_time_spent INT DEFAULT 0, -- minutes
    status VARCHAR(20) NOT NULL DEFAULT 'enrolled', -- 'enrolled', 'in_progress', 'completed', 'dropped', 'suspended'
    completion_certificate_url VARCHAR(500),
    final_score DECIMAL(5,2),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(course_id, user_id)
);

-- Learning Progress Tracking for FLOW7 Support
CREATE TABLE learning_progress (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    enrollment_id UUID NOT NULL REFERENCES course_enrollments(id),
    lesson_id UUID NOT NULL REFERENCES course_lessons(id),
    user_id UUID NOT NULL REFERENCES users(id),
    status VARCHAR(20) NOT NULL DEFAULT 'not_started', -- 'not_started', 'in_progress', 'completed', 'skipped'
    start_time TIMESTAMP,
    completion_time TIMESTAMP,
    time_spent INT DEFAULT 0, -- minutes
    attempts_count INT DEFAULT 0,
    score DECIMAL(5,2),
    notes TEXT,
    bookmarks JSONB,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(enrollment_id, lesson_id)
);

-- Assessments for FLOW7 Support
CREATE TABLE assessments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    course_id UUID REFERENCES courses(id),
    lesson_id UUID REFERENCES course_lessons(id),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    assessment_type VARCHAR(50), -- 'quiz', 'assignment', 'project', 'exam'
    questions JSONB, -- quiz questions and configuration
    max_attempts INT DEFAULT 1,
    time_limit_minutes INT,
    passing_score DECIMAL(5,2) DEFAULT 70,
    points_value INT DEFAULT 0,
    is_graded BOOLEAN DEFAULT true,
    auto_grade BOOLEAN DEFAULT false,
    instructions TEXT,
    resources JSONB,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Assessment Submissions for FLOW7 Support
CREATE TABLE assessment_submissions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    assessment_id UUID NOT NULL REFERENCES assessments(id),
    user_id UUID NOT NULL REFERENCES users(id),
    enrollment_id UUID NOT NULL REFERENCES course_enrollments(id),
    submission_data JSONB, -- answers/submission content
    submitted_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    graded_at TIMESTAMP,
    graded_by UUID REFERENCES users(id),
    score DECIMAL(5,2),
    max_score DECIMAL(5,2),
    feedback TEXT,
    attempt_number INT DEFAULT 1,
    time_taken_minutes INT,
    status VARCHAR(20) DEFAULT 'submitted', -- 'submitted', 'graded', 'returned', 'resubmitted'
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Certificates for FLOW7 Support
CREATE TABLE certificates (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    course_id UUID NOT NULL REFERENCES courses(id),
    user_id UUID NOT NULL REFERENCES users(id),
    enrollment_id UUID NOT NULL REFERENCES course_enrollments(id),
    certificate_number VARCHAR(100) UNIQUE NOT NULL,
    issued_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    expiry_date TIMESTAMP,
    certificate_url VARCHAR(500),
    verification_code VARCHAR(50) UNIQUE,
    final_score DECIMAL(5,2),
    completion_time_hours DECIMAL(6,2),
    skills_earned JSONB,
    is_verified BOOLEAN DEFAULT true,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Learning Paths for FLOW7 Support
CREATE TABLE learning_paths (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    target_role VARCHAR(100),
    difficulty_level VARCHAR(50),
    estimated_duration_weeks INT,
    skills_covered JSONB,
    prerequisites JSONB,
    created_by UUID REFERENCES users(id),
    is_public BOOLEAN DEFAULT true,
    enrollment_count INT DEFAULT 0,
    completion_rate DECIMAL(5,2) DEFAULT 0,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Learning Path Courses for FLOW7 Support
CREATE TABLE learning_path_courses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    learning_path_id UUID NOT NULL REFERENCES learning_paths(id),
    course_id UUID NOT NULL REFERENCES courses(id),
    sequence_order INT NOT NULL,
    is_mandatory BOOLEAN DEFAULT true,
    unlock_criteria JSONB,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(learning_path_id, course_id)
);

-- Course Reviews for FLOW7 Support
CREATE TABLE course_reviews (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    course_id UUID NOT NULL REFERENCES courses(id),
    user_id UUID NOT NULL REFERENCES users(id),
    enrollment_id UUID NOT NULL REFERENCES course_enrollments(id),
    rating DECIMAL(2,1) NOT NULL CHECK (rating >= 1.0 AND rating <= 5.0),
    review_text TEXT,
    review_categories JSONB, -- {"content": 5, "instructor": 4, "difficulty": 3}
    is_verified BOOLEAN DEFAULT false,
    helpful_count INT DEFAULT 0,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(course_id, user_id)
);

-- Learning Platform Indexes
CREATE INDEX idx_courses_category_id ON courses(category_id);
CREATE INDEX idx_courses_instructor_id ON courses(instructor_id);
CREATE INDEX idx_courses_status ON courses(status);
CREATE INDEX idx_courses_level ON courses(level);
CREATE INDEX idx_courses_rating ON courses(rating);
CREATE INDEX idx_courses_featured ON courses(is_featured);
CREATE INDEX idx_course_categories_parent ON course_categories(parent_category_id);
CREATE INDEX idx_course_modules_course_id ON course_modules(course_id);
CREATE INDEX idx_course_lessons_module_id ON course_lessons(module_id);
CREATE INDEX idx_course_enrollments_course_id ON course_enrollments(course_id);
CREATE INDEX idx_course_enrollments_user_id ON course_enrollments(user_id);
CREATE INDEX idx_course_enrollments_status ON course_enrollments(status);
CREATE INDEX idx_learning_progress_enrollment_id ON learning_progress(enrollment_id);
CREATE INDEX idx_learning_progress_user_id ON learning_progress(user_id);
CREATE INDEX idx_learning_progress_lesson_id ON learning_progress(lesson_id);
CREATE INDEX idx_assessments_course_id ON assessments(course_id);
CREATE INDEX idx_assessments_lesson_id ON assessments(lesson_id);
CREATE INDEX idx_assessment_submissions_assessment_id ON assessment_submissions(assessment_id);
CREATE INDEX idx_assessment_submissions_user_id ON assessment_submissions(user_id);
CREATE INDEX idx_certificates_course_id ON certificates(course_id);
CREATE INDEX idx_certificates_user_id ON certificates(user_id);
CREATE INDEX idx_certificates_verification_code ON certificates(verification_code);
CREATE INDEX idx_learning_paths_target_role ON learning_paths(target_role);
CREATE INDEX idx_learning_path_courses_path_id ON learning_path_courses(learning_path_id);
CREATE INDEX idx_learning_path_courses_course_id ON learning_path_courses(course_id);
CREATE INDEX idx_course_reviews_course_id ON course_reviews(course_id);
CREATE INDEX idx_course_reviews_user_id ON course_reviews(user_id);
```

### Skill Matching Schema

-- Enhanced Skill Profiles for FLOW5 Support
```sql
CREATE TABLE skill_profiles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    talent_id UUID NOT NULL REFERENCES talents(id),
    skill_id UUID NOT NULL REFERENCES skills(id),
    proficiency_level VARCHAR(20) NOT NULL, -- 'beginner', 'intermediate', 'advanced', 'expert'
    years_experience DECIMAL(3,1),
    last_used_date DATE,
    certification_level VARCHAR(50),
    self_assessment_score DECIMAL(3,1),
    validated_score DECIMAL(3,1),
    learning_progress DECIMAL(5,2), -- percentage
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(talent_id, skill_id)
);

-- Skill Assessments for FLOW5 Support
CREATE TABLE skill_assessments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    talent_id UUID NOT NULL REFERENCES talents(id),
    skill_id UUID NOT NULL REFERENCES skills(id),
    assessment_type VARCHAR(50), -- 'technical_test', 'peer_review', 'project_based', 'certification'
    score DECIMAL(5,2) NOT NULL,
    max_score DECIMAL(5,2) NOT NULL,
    assessment_date DATE NOT NULL,
    assessor_id UUID REFERENCES users(id),
    validity_period INT, -- days
    notes TEXT,
    certification_earned VARCHAR(255),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Matching Requests for FLOW5 Support
CREATE TABLE matching_requests (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    requester_type VARCHAR(20) NOT NULL, -- 'PROJECT', 'TEAM', 'BUSINESS'
    requester_id UUID NOT NULL,
    request_title VARCHAR(255) NOT NULL,
    required_skills JSONB NOT NULL, -- [{skill_id, min_proficiency, importance}]
    optional_skills JSONB, -- [{skill_id, preferred_proficiency, bonus_weight}]
    experience_level VARCHAR(50), -- 'junior', 'mid', 'senior', 'mixed'
    availability_requirement VARCHAR(100), -- 'full_time', 'part_time', 'contract'
    location_preference VARCHAR(255),
    budget_range VARCHAR(100),
    timeline VARCHAR(100),
    priority VARCHAR(20) DEFAULT 'medium',
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Enhanced Match Results for FLOW5 Support
CREATE TABLE match_results (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    request_id UUID NOT NULL REFERENCES matching_requests(id),
    talent_id UUID NOT NULL REFERENCES talents(id),
    overall_score DECIMAL(5,2) NOT NULL,
    skill_match_score DECIMAL(5,2) NOT NULL,
    experience_score DECIMAL(5,2) NOT NULL,
    availability_score DECIMAL(5,2) NOT NULL,
    compatibility_score DECIMAL(5,2),
    recommendation_rank INT,
    match_confidence DECIMAL(3,1), -- 0-100%
    explanation JSONB, -- detailed explanation of scoring
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Match Feedback for FLOW5 Support
CREATE TABLE match_feedback (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    match_result_id UUID NOT NULL REFERENCES match_results(id),
    feedback_type VARCHAR(50), -- 'hire_success', 'hire_failure', 'interview_feedback'
    rating DECIMAL(3,1), -- 1-5 stars
    comments TEXT,
    feedback_provider_id UUID NOT NULL REFERENCES users(id),
    feedback_date DATE NOT NULL,
    improvement_suggestions JSONB,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Skill Weights Configuration for FLOW5 Support
CREATE TABLE skill_weights (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    skill_category VARCHAR(100),
    skill_id UUID REFERENCES skills(id),
    base_weight DECIMAL(3,2) DEFAULT 1.0,
    project_type_weights JSONB, -- weights for different project types
    experience_multiplier DECIMAL(3,2) DEFAULT 1.0,
    recency_factor DECIMAL(3,2) DEFAULT 1.0,
    certification_bonus DECIMAL(3,2) DEFAULT 0.1,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Matching Rules for FLOW5 Support
CREATE TABLE matching_rules (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    rule_name VARCHAR(255) NOT NULL,
    rule_type VARCHAR(50), -- 'filter', 'boost', 'penalty', 'requirement'
    conditions JSONB NOT NULL, -- rule conditions
    actions JSONB NOT NULL, -- rule actions
    priority INT DEFAULT 1,
    is_active BOOLEAN DEFAULT true,
    created_by UUID NOT NULL REFERENCES users(id),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Compatibility Scores for FLOW5 Support
CREATE TABLE compatibility_scores (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    talent1_id UUID NOT NULL REFERENCES talents(id),
    talent2_id UUID NOT NULL REFERENCES talents(id),
    collaboration_score DECIMAL(3,1),
    communication_score DECIMAL(3,1),
    work_style_score DECIMAL(3,1),
    personality_score DECIMAL(3,1),
    past_project_success DECIMAL(3,1),
    calculated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(talent1_id, talent2_id)
);

-- Indexes for better performance
CREATE INDEX idx_skill_profiles_talent_id ON skill_profiles(talent_id);
CREATE INDEX idx_skill_profiles_skill_id ON skill_profiles(skill_id);
CREATE INDEX idx_skill_profiles_proficiency ON skill_profiles(proficiency_level);
CREATE INDEX idx_skill_assessments_talent_skill ON skill_assessments(talent_id, skill_id);
CREATE INDEX idx_matching_requests_status ON matching_requests(status);
CREATE INDEX idx_matching_requests_requester ON matching_requests(requester_type, requester_id);
CREATE INDEX idx_match_results_request_id ON match_results(request_id);
CREATE INDEX idx_match_results_talent_id ON match_results(talent_id);
CREATE INDEX idx_match_results_score ON match_results(overall_score DESC);
CREATE INDEX idx_match_feedback_result_id ON match_feedback(match_result_id);
CREATE INDEX idx_compatibility_scores_talents ON compatibility_scores(talent1_id, talent2_id);
```

### Audit Trail Schema

```sql
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    table_name VARCHAR(100) NOT NULL,
    record_id UUID NOT NULL,
    action VARCHAR(20) NOT NULL, -- INSERT, UPDATE, DELETE
    old_values JSONB,
    new_values JSONB,
    changed_by UUID NOT NULL REFERENCES users(id),
    changed_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    ip_address INET,
    user_agent TEXT
);

CREATE INDEX idx_audit_logs_table_record ON audit_logs(table_name, record_id);
CREATE INDEX idx_audit_logs_changed_by ON audit_logs(changed_by);
CREATE INDEX idx_audit_logs_changed_at ON audit_logs(changed_at);
```

### Notification System Schema

```sql
CREATE TABLE notifications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    title VARCHAR(255) NOT NULL,
    message TEXT NOT NULL,
    type VARCHAR(50) NOT NULL, -- info, warning, error, success
    entity_type VARCHAR(50), -- project, team, contract, etc.
    entity_id UUID,
    is_read BOOLEAN NOT NULL DEFAULT false,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    read_at TIMESTAMP
);

CREATE INDEX idx_notifications_user_id ON notifications(user_id);
CREATE INDEX idx_notifications_unread ON notifications(user_id, is_read) WHERE is_read = false;
CREATE INDEX idx_notifications_created_at ON notifications(created_at);

CREATE TABLE notification_preferences (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    notification_type VARCHAR(50) NOT NULL,
    email_enabled BOOLEAN NOT NULL DEFAULT true,
    push_enabled BOOLEAN NOT NULL DEFAULT true,
    sms_enabled BOOLEAN NOT NULL DEFAULT false,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(user_id, notification_type)
);
```

### Additional Indexes for Performance Optimization

```sql
-- Users table indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
CREATE INDEX idx_users_status ON users(status);

-- Businesses table indexes
CREATE INDEX idx_businesses_industry ON businesses(industry);
CREATE INDEX idx_businesses_size ON businesses(size);
CREATE INDEX idx_businesses_status ON businesses(status);

-- Talents table indexes
CREATE INDEX idx_talents_user_id ON talents(user_id);
CREATE INDEX idx_talents_availability ON talents(availability);
CREATE INDEX idx_talents_experience ON talents(years_of_experience);

-- Talent skills indexes
CREATE INDEX idx_talent_skills_talent_id ON talent_skills(talent_id);
CREATE INDEX idx_talent_skills_skill_id ON talent_skills(skill_id);
CREATE INDEX idx_talent_skills_proficiency ON talent_skills(proficiency_level);

-- Projects table indexes
CREATE INDEX idx_projects_business_id ON projects(business_id);
CREATE INDEX idx_projects_team_id ON projects(team_id);
CREATE INDEX idx_projects_status ON projects(status);
CREATE INDEX idx_projects_dates ON projects(start_date, end_date);

-- ODC Teams indexes
CREATE INDEX idx_odc_teams_business_id ON odc_teams(business_id);
CREATE INDEX idx_odc_teams_status ON odc_teams(status);

-- Team members indexes
CREATE INDEX idx_team_members_team_id ON team_members(team_id);
CREATE INDEX idx_team_members_talent_id ON team_members(talent_id);
CREATE INDEX idx_team_members_status ON team_members(status);

-- Marketplace projects indexes
CREATE INDEX idx_marketplace_projects_business_id ON marketplace_projects(business_id);
CREATE INDEX idx_marketplace_projects_status ON marketplace_projects(status);
CREATE INDEX idx_marketplace_projects_budget ON marketplace_projects(budget_min, budget_max);

-- Course enrollments indexes
CREATE INDEX idx_course_enrollments_course_id ON course_enrollments(course_id);
CREATE INDEX idx_course_enrollments_user_id ON course_enrollments(user_id);
CREATE INDEX idx_course_enrollments_status ON course_enrollments(status);
```

## 4. MongoDB Collections

### Talent Profiles Collection
```json
{
  "_id": "ObjectId",
  "talent_id": "UUID",
  "personal_info": {
    "bio": "string",
    "interests": ["string"],
    "preferred_work_environment": "string",
    "career_goals": "string",
    "personality_traits": ["string"],
    "work_style_preferences": ["string"]
  },
  "education": [
    {
      "institution": "string",
      "degree": "string",
      "field": "string",
      "start_date": "date",
      "end_date": "date",
      "description": "string",
      "gpa": "number",
      "honors": ["string"]
    }
  ],
  "work_experience": [
    {
      "company": "string",
      "position": "string",
      "start_date": "date",
      "end_date": "date",
      "description": "string",
      "technologies": ["string"],
      "achievements": ["string"],
      "team_size": "number"
    }
  ],
  "projects": [
    {
      "name": "string",
      "description": "string",
      "role": "string",
      "technologies": ["string"],
      "url": "string",
      "duration": "string",
      "team_size": "number",
      "impact": "string"
    }
  ],
  "certifications": [
    {
      "name": "string",
      "issuer": "string",
      "date": "date",
      "expires": "date",
      "url": "string",
      "verification_code": "string",
      "skill_areas": ["string"]
    }
  ],
  "social_links": {
    "linkedin": "string",
    "github": "string",
    "portfolio": "string",
    "stackoverflow": "string",
    "twitter": "string"
  },
  "availability_calendar": {
    "timezone": "string",
    "working_hours": {
      "start": "string",
      "end": "string"
    },
    "available_days": ["string"],
    "vacation_periods": [
      {
        "start_date": "date",
        "end_date": "date",
        "reason": "string"
      }
    ]
  },
  "created_at": "date",
  "updated_at": "date"
}
```

### Project Analytics Collection
```json
{
  "_id": "ObjectId",
  "project_id": "UUID",
  "analytics_date": "date",
  "metrics": {
    "progress_percentage": "number",
    "tasks_completed": "number",
    "tasks_total": "number",
    "team_productivity": "number",
    "budget_utilization": "number",
    "timeline_adherence": "number",
    "quality_score": "number",
    "risk_level": "string"
  },
  "team_performance": [
    {
      "member_id": "UUID",
      "productivity_score": "number",
      "task_completion_rate": "number",
      "collaboration_score": "number"
    }
  ],
  "milestone_progress": [
    {
      "milestone_id": "UUID",
      "completion_percentage": "number",
      "estimated_completion": "date",
      "risk_factors": ["string"]
    }
  ],
  "created_at": "date"
}
```

### Marketplace Analytics Collection
```json
{
  "_id": "ObjectId",
  "project_id": "UUID",
  "analytics_period": {
    "start_date": "date",
    "end_date": "date"
  },
  "bidding_metrics": {
    "total_bids": "number",
    "average_bid_amount": "number",
    "bid_range": {
      "min": "number",
      "max": "number"
    },
    "response_time_avg": "number",
    "conversion_rate": "number"
  },
  "talent_engagement": {
    "views": "number",
    "favorites": "number",
    "questions_asked": "number",
    "proposal_quality_avg": "number"
  },
  "market_trends": {
    "skill_demand": [
      {
        "skill": "string",
        "demand_score": "number",
        "avg_rate": "number"
      }
    ],
    "competition_level": "string",
    "success_probability": "number"
  },
  "created_at": "date"
}
```

### Feedback Collection
```json
{
  "_id": "ObjectId",
  "entity_type": "string",
  "entity_id": "UUID",
  "user_id": "UUID",
  "rating": "number",
  "content": "string",
  "attributes": {
    "communication": "number",
    "quality": "number",
    "timeliness": "number",
    "expertise": "number",
    "collaboration": "number",
    "problem_solving": "number"
  },
  "feedback_context": {
    "project_phase": "string",
    "interaction_type": "string",
    "duration": "number"
  },
  "improvement_suggestions": ["string"],
  "would_recommend": "boolean",
  "created_at": "date"
}
```

### Course Content Collection
```json
{
  "_id": "ObjectId",
  "course_id": "UUID",
  "module_id": "UUID",
  "lesson_id": "UUID",
  "type": "string",
  "title": "string",
  "content": "string",
  "content_format": "string",
  "media": [
    {
      "type": "string",
      "url": "string",
      "description": "string",
      "duration": "number",
      "quality": "string",
      "subtitles": ["string"]
    }
  ],
  "attachments": [
    {
      "name": "string",
      "url": "string",
      "size": "number",
      "type": "string",
      "description": "string"
    }
  ],
  "interactive_elements": [
    {
      "type": "string",
      "config": "object",
      "position": "number"
    }
  ],
  "learning_objectives": ["string"],
  "prerequisites": ["string"],
  "sequence_order": "number",
  "estimated_time": "number",
  "difficulty_level": "string",
  "tags": ["string"],
  "version": "string",
  "created_at": "date",
  "updated_at": "date"
}
```

### Learning Analytics Collection
```json
{
  "_id": "ObjectId",
  "user_id": "UUID",
  "course_id": "UUID",
  "session_id": "string",
  "learning_session": {
    "start_time": "date",
    "end_time": "date",
    "duration": "number",
    "device_type": "string",
    "browser": "string"
  },
  "content_interactions": [
    {
      "content_id": "UUID",
      "interaction_type": "string",
      "timestamp": "date",
      "duration": "number",
      "completion_percentage": "number",
      "engagement_score": "number"
    }
  ],
  "assessment_attempts": [
    {
      "assessment_id": "UUID",
      "attempt_number": "number",
      "score": "number",
      "time_taken": "number",
      "answers": ["object"]
    }
  ],
  "learning_path_progress": {
    "current_step": "number",
    "completed_steps": ["number"],
    "estimated_completion": "date",
    "personalized_recommendations": ["string"]
  },
  "created_at": "date"
}
```

### System Logs Collection
```json
{
  "_id": "ObjectId",
  "timestamp": "date",
  "level": "string",
  "service": "string",
  "message": "string",
  "user_id": "UUID",
  "session_id": "string",
  "ip_address": "string",
  "user_agent": "string",
  "request_path": "string",
  "request_method": "string",
  "status_code": "number",
  "execution_time": "number",
  "memory_usage": "number",
  "cpu_usage": "number",
  "error_details": {
    "error_code": "string",
    "error_message": "string",
    "stack_trace": "string",
    "context": "object"
  },
  "business_context": {
    "feature": "string",
    "flow": "string",
    "operation": "string"
  },
  "additional_data": "object"
}
```

### Real-time Events Collection
```json
{
  "_id": "ObjectId",
  "event_type": "string",
  "event_source": "string",
  "timestamp": "date",
  "user_id": "UUID",
  "entity_type": "string",
  "entity_id": "UUID",
  "event_data": {
    "action": "string",
    "previous_state": "object",
    "new_state": "object",
    "metadata": "object"
  },
  "notification_targets": ["UUID"],
  "processed": "boolean",
  "processing_attempts": "number",
  "created_at": "date"
}
```

## 5. Elasticsearch Indices

### Talent Search Index
```json
{
  "mappings": {
    "properties": {
      "talent_id": { "type": "keyword" },
      "full_name": { "type": "text" },
      "bio": { "type": "text" },
      "skills": {
        "type": "nested",
        "properties": {
          "skill_id": { "type": "keyword" },
          "name": { "type": "text" },
          "category": { "type": "keyword" },
          "proficiency": { "type": "integer" },
          "years": { "type": "float" },
          "validated_score": { "type": "float" },
          "certification_level": { "type": "keyword" }
        }
      },
      "education_level": { "type": "keyword" },
      "years_of_experience": { "type": "integer" },
      "availability": { "type": "keyword" },
      "hourly_rate": { "type": "float" },
      "projects": {
        "type": "nested",
        "properties": {
          "name": { "type": "text" },
          "technologies": { "type": "text" },
          "role": { "type": "keyword" },
          "duration": { "type": "keyword" },
          "team_size": { "type": "integer" }
        }
      },
      "certifications": {
        "type": "nested",
        "properties": {
          "name": { "type": "text" },
          "issuer": { "type": "keyword" },
          "skill_areas": { "type": "keyword" }
        }
      },
      "personality_traits": { "type": "keyword" },
      "work_style_preferences": { "type": "keyword" },
      "location": { "type": "geo_point" },
      "timezone": { "type": "keyword" },
      "rating": { "type": "float" },
      "completed_projects": { "type": "integer" },
      "success_rate": { "type": "float" }
    }
  }
}
```

### Project Search Index
```json
{
  "mappings": {
    "properties": {
      "project_id": { "type": "keyword" },
      "title": { "type": "text" },
      "description": { "type": "text" },
      "status": { "type": "keyword" },
      "priority": { "type": "keyword" },
      "budget_range": {
        "type": "object",
        "properties": {
          "min": { "type": "float" },
          "max": { "type": "float" }
        }
      },
      "required_skills": {
        "type": "nested",
        "properties": {
          "skill_name": { "type": "text" },
          "skill_category": { "type": "keyword" },
          "proficiency_level": { "type": "keyword" },
          "importance": { "type": "keyword" }
        }
      },
      "project_type": { "type": "keyword" },
      "duration": { "type": "keyword" },
      "team_size": { "type": "integer" },
      "location": { "type": "geo_point" },
      "remote_allowed": { "type": "boolean" },
      "business_industry": { "type": "keyword" },
      "created_at": { "type": "date" },
      "deadline": { "type": "date" }
    }
  }
}
```

### Marketplace Project Search Index
```json
{
  "mappings": {
    "properties": {
      "project_id": { "type": "keyword" },
      "title": { "type": "text" },
      "description": { "type": "text" },
      "category": { "type": "keyword" },
      "subcategory": { "type": "keyword" },
      "budget_min": { "type": "float" },
      "budget_max": { "type": "float" },
      "duration_weeks": { "type": "integer" },
      "complexity_level": { "type": "keyword" },
      "required_skills": {
        "type": "nested",
        "properties": {
          "skill_name": { "type": "text" },
          "proficiency_required": { "type": "keyword" }
        }
      },
      "preferred_skills": { "type": "keyword" },
      "location_requirement": { "type": "keyword" },
      "remote_allowed": { "type": "boolean" },
      "business_size": { "type": "keyword" },
      "industry": { "type": "keyword" },
      "urgency": { "type": "keyword" },
      "bid_count": { "type": "integer" },
      "avg_bid_amount": { "type": "float" },
      "posted_date": { "type": "date" },
      "deadline": { "type": "date" },
      "status": { "type": "keyword" }
    }
  }
}
```

### Course Search Index
```json
{
  "mappings": {
    "properties": {
      "course_id": { "type": "keyword" },
      "title": { "type": "text" },
      "description": { "type": "text" },
      "level": { "type": "keyword" },
      "category": { "type": "keyword" },
      "subcategory": { "type": "keyword" },
      "skills_covered": { "type": "keyword" },
      "learning_objectives": { "type": "text" },
      "prerequisites": { "type": "text" },
      "content": { "type": "text" },
      "tags": { "type": "keyword" },
      "instructor_name": { "type": "text" },
      "instructor_rating": { "type": "float" },
      "course_rating": { "type": "float" },
      "enrollment_count": { "type": "integer" },
      "completion_rate": { "type": "float" },
      "duration_hours": { "type": "float" },
      "price": { "type": "float" },
      "is_free": { "type": "boolean" },
      "certification_offered": { "type": "boolean" },
      "language": { "type": "keyword" },
      "created_at": { "type": "date" },
      "updated_at": { "type": "date" }
    }
  }
}
```

### Skill Matching Index
```json
{
  "mappings": {
    "properties": {
      "match_id": { "type": "keyword" },
      "request_id": { "type": "keyword" },
      "talent_id": { "type": "keyword" },
      "overall_score": { "type": "float" },
      "skill_match_score": { "type": "float" },
      "experience_score": { "type": "float" },
      "availability_score": { "type": "float" },
      "compatibility_score": { "type": "float" },
      "match_confidence": { "type": "float" },
      "recommendation_rank": { "type": "integer" },
      "matched_skills": {
        "type": "nested",
        "properties": {
          "skill_name": { "type": "keyword" },
          "required_level": { "type": "keyword" },
          "talent_level": { "type": "keyword" },
          "match_strength": { "type": "float" }
        }
      },
      "talent_profile": {
        "type": "object",
        "properties": {
          "name": { "type": "text" },
          "experience_years": { "type": "integer" },
          "hourly_rate": { "type": "float" },
          "availability": { "type": "keyword" },
          "location": { "type": "geo_point" }
        }
      },
      "request_details": {
        "type": "object",
        "properties": {
          "title": { "type": "text" },
          "budget_range": { "type": "keyword" },
          "timeline": { "type": "keyword" },
          "priority": { "type": "keyword" }
        }
      },
      "created_at": { "type": "date" }
    }
  }
}
```

## 6. Sơ đồ ERD - Cập nhật đầy đủ

### 6.1. Tổng quan hệ thống

```
                    ┌─────────────────────────────┐
                    │        CORE SYSTEM          │
                    │                             │
                    │  ┌─────────────────────┐    │
                    │  │       USERS         │    │
                    │  │   (Authentication)  │    │
                    │  └─────────────────────┘    │
                    │            │                │
                    │            │                │
                    │  ┌─────────┼─────────┐      │
                    │  │         │         │      │
                    │  ▼         ▼         ▼      │
                    │ BUSINESSES TALENTS  ADMINS  │
                    └─────────────────────────────┘
                              │
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│   FLOW 1-2      │  │   FLOW 3-4      │  │   FLOW 5-9      │
│ Business &      │  │ ODC Teams &     │  │ Marketplace &   │
│ Talent Mgmt     │  │ Projects        │  │ Learning        │
└─────────────────┘  └─────────────────┘  └─────────────────┘
```

### 6.2. Quản lý người dùng và phân quyền (Đơn giản hóa: Role 1-N User)

```
┌─────────────────────────────┐       ┌─────────────────────────────┐
│           ROLES             │       │           USERS             │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │
│  name (VARCHAR)             │       │  email (VARCHAR)            │
│  description (TEXT)         │       │  password_hash (VARCHAR)    │
│  permissions (JSONB)        │       │  full_name (VARCHAR)        │
│  created_at (TIMESTAMP)     │       │  phone (VARCHAR)            │
│  updated_at (TIMESTAMP)     │       │ *role_id (UUID) [FK]        │<──┐
└─────────────────────────────┘       │  status (VARCHAR)           │   │
              │                       │  created_at (TIMESTAMP)     │   │
              │ 1                     │  updated_at (TIMESTAMP)     │   │
              │                       └─────────────────────────────┘   │
              │                                                         │
              └─────────────────────────────────────────────────────────┘
              
              Mối quan hệ: 1 Role có thể có nhiều Users (1-N)
              
              Các roles có sẵn:
              • admin - System Administrator
              • business_owner - Business Owner  
              • business_manager - Business Manager
              • talent - Talent/Developer
              • team_lead - Team Lead
              • hr_manager - HR Manager
              • instructor - Course Instructor
              • student - Course Student
```

### 6.3. FLOW 1: Business Onboarding và Registration

```
┌─────────────────────────────┐       ┌─────────────────────────────┐
│   BUSINESS_REGISTRATIONS    │       │         BUSINESSES          │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │
│  business_name (VARCHAR)    │       │ *registration_id (UUID) [FK]│<──┐
│  business_email (VARCHAR)   │       │  name (VARCHAR)             │   │
│  contact_person_name (VAR)  │       │  email (VARCHAR)            │   │
│  contact_person_email (VAR) │       │  phone (VARCHAR)            │   │
│  industry (VARCHAR)         │       │  address (TEXT)             │   │
│  company_size (VARCHAR)     │       │  industry (VARCHAR)         │   │
│  registration_documents     │       │  size (VARCHAR)             │   │
│  status (VARCHAR)           │       │  verification_status (VAR)  │   │
│  submitted_at (TIMESTAMP)   │       │  subscription_plan (VAR)    │   │
│  reviewed_by (UUID) [FK]    │<──┐   │  lab_setup_completed (BOOL) │   │
│  created_at (TIMESTAMP)     │   │   │  onboarding_completed (BOOL)│   │
│  updated_at (TIMESTAMP)     │   │   │  status (VARCHAR)           │   │
└─────────────────────────────┘   │   │  created_at (TIMESTAMP)     │   │
              │                   │   │  updated_at (TIMESTAMP)     │   │
              └───────────────────┼───└─────────────────────────────┘   │
                                  │                   │               │
                                  │                   │ 1             │
                                  │                   │               │
                                  │                   │ *             │
              ┌─────────────────────────────┐         │   ┌─────────────────────────────┐
              │           USERS             │         │   │      BUSINESS_CONTACTS      │
              │         (reviewer)          │─────────┘   ├─────────────────────────────┤
              └─────────────────────────────┘             │ *id (UUID) [PK]             │
                                                          │ *business_id (UUID) [FK]    │<─┘
                                                          │ *user_id (UUID) [FK]        │<──┐
                                                          │  position (VARCHAR)         │   │
                                                          │  is_primary (BOOLEAN)       │   │
                                                          │  created_at (TIMESTAMP)    │   │
                                                          │  updated_at (TIMESTAMP)    │   │
                                                          └─────────────────────────────┘   │
                                                                                            │
              ┌─────────────────────────────┐                                             │
              │           USERS             │                                             │
              │        (contact user)       │─────────────────────────────────────────────┘
              └─────────────────────────────┘

┌─────────────────────────────┐       ┌─────────────────────────────┐
│   BUSINESS_REQUIREMENTS     │       │     REQUIREMENT_SKILLS      │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │
│ *business_id (UUID) [FK]    │<──┐   │ *requirement_id (UUID) [FK] │<──┐
│  title (VARCHAR)            │   │   │ *skill_id (UUID) [FK]       │<──┼─┐
│  description (TEXT)         │   │   │  experience_level (VARCHAR) │   │ │
│  team_size (INT)            │   │   │  created_at (TIMESTAMP)    │   │ │
│  duration (VARCHAR)         │   │   └─────────────────────────────┘   │ │
│  contract_type (VARCHAR)    │   │                                     │ │
│  status (VARCHAR)           │   │   ┌─────────────────────────────┐   │ │
│  created_at (TIMESTAMP)     │   │   │           SKILLS            │   │ │
│  updated_at (TIMESTAMP)     │   │   ├─────────────────────────────┤   │ │
└─────────────────────────────┘   │   │ *id (UUID) [PK]             │   │ │
              │                   │   │  name (VARCHAR)             │   │ │
              └───────────────────┘   │  category (VARCHAR)         │───┘ │
                                      │  description (TEXT)         │     │
                                      │  created_at (TIMESTAMP)    │     │
                                      └─────────────────────────────┘     │
                                                                          │
┌─────────────────────────────┐                                         │
│          CONTRACTS          │                                         │
├─────────────────────────────┤                                         │
│ *id (UUID) [PK]             │                                         │
│ *business_id (UUID) [FK]    │──────> BUSINESSES                      │
│  requirement_id (UUID) [FK] │──────> BUSINESS_REQUIREMENTS ──────────┘
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

### 6.3. FLOW 2: Talent Management và Skill Assessment

```
┌─────────────────────────────┐       ┌─────────────────────────────┐
│     TALENT_APPLICATIONS     │       │          TALENTS            │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │
│  full_name (VARCHAR)        │       │ *application_id (UUID) [FK] │<──┐
│  email (VARCHAR)            │       │ *user_id (UUID) [FK]        │<──┼─┐
│  phone (VARCHAR)            │       │  full_name (VARCHAR)        │   │ │
│  date_of_birth (DATE)       │       │  email (VARCHAR)            │   │ │
│  education_level (VARCHAR)  │       │  phone (VARCHAR)            │   │ │
│  years_of_experience (INT)  │       │  date_of_birth (DATE)       │   │ │
│  cv_file_path (VARCHAR)     │       │  address (TEXT)             │   │ │
│  portfolio_url (VARCHAR)    │       │  education_level (VARCHAR)  │   │ │
│  cover_letter (TEXT)        │       │  years_of_experience (INT)  │   │ │
│  preferred_work_type (VAR)  │       │  cv_file_path (VARCHAR)     │   │ │
│  expected_salary (DECIMAL)  │       │  portfolio_url (VARCHAR)    │   │ │
│  availability_date (DATE)   │       │  availability_status (VAR)  │   │ │
│  status (VARCHAR)           │       │  hourly_rate (DECIMAL)      │   │ │
│  submitted_at (TIMESTAMP)   │       │  preferred_work_type (VAR)  │   │ │
│  reviewed_by (UUID) [FK]    │<──┐   │  lab_access_level (VARCHAR) │   │ │
│  created_at (TIMESTAMP)     │   │   │  onboarding_completed (BOOL)│   │ │
│  updated_at (TIMESTAMP)     │   │   │  status (VARCHAR)           │   │ │
└─────────────────────────────┘   │   │  created_at (TIMESTAMP)    │   │ │
              │                   │   │  updated_at (TIMESTAMP)    │   │ │
              └───────────────────┼───└─────────────────────────────┘   │ │
                                  │                   │               │ │
                                  │                   │ 1             │ │
                                  │                   │               │ │
                                  │                   │ *             │ │
              ┌─────────────────────────────┐         │   ┌─────────────────────────────┐ │
              │           USERS             │         │   │       TALENT_SKILLS         │ │
              │         (reviewer)          │─────────┘   ├─────────────────────────────┤ │
              └─────────────────────────────┘             │ *id (UUID) [PK]             │ │
                                                          │ *talent_id (UUID) [FK]      │<┘ │
                                                          │ *skill_id (UUID) [FK]       │<──┼─┐
                                                          │  proficiency_level (VARCHAR)│   │ │
                                                          │  years_of_experience (INT)  │   │ │
                                                          │  is_primary (BOOLEAN)       │   │ │
                                                          │  created_at (TIMESTAMP)    │   │ │
                                                          │  updated_at (TIMESTAMP)    │   │ │
                                                          └─────────────────────────────┘   │ │
                                                                                            │ │
              ┌─────────────────────────────┐                                             │ │
              │           USERS             │                                             │ │
              │        (talent user)        │─────────────────────────────────────────────┘ │
              └─────────────────────────────┘                                               │
                                                                                            │
┌─────────────────────────────┐       ┌─────────────────────────────┐                     │
│     SKILL_ASSESSMENTS       │       │           SKILLS            │                     │
├─────────────────────────────┤       ├─────────────────────────────┤                     │
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │                     │
│ *talent_id (UUID) [FK]      │<──┐   │  name (VARCHAR)             │                     │
│ *skill_id (UUID) [FK]       │<──┼───│  category (VARCHAR)         │─────────────────────┘
│  assessment_type (VARCHAR)  │   │   │  description (TEXT)         │
│  score (DECIMAL)            │   │   │  assessment_criteria (JSON) │
│  max_score (DECIMAL)        │   │   │  created_at (TIMESTAMP)    │
│  assessment_date (DATE)     │   │   │  updated_at (TIMESTAMP)    │
│  assessor_id (UUID) [FK]    │<──┼─┐ └─────────────────────────────┘
│  assessment_notes (TEXT)    │   │ │
│  certificate_url (VARCHAR)  │   │ │ ┌─────────────────────────────┐
│  validity_period (INT)      │   │ │ │     TRAINING_PROGRESS       │
│  status (VARCHAR)           │   │ │ ├─────────────────────────────┤
│  created_at (TIMESTAMP)     │   │ │ │ *id (UUID) [PK]             │
│  updated_at (TIMESTAMP)     │   │ │ │ *talent_id (UUID) [FK]      │<─┘
└─────────────────────────────┘   │ │ │ *course_id (UUID) [FK]      │<──┐
                                  │ │ │  enrollment_date (DATE)     │   │
              ┌─────────────────────────────┐ │  start_date (DATE)          │   │
              │           USERS             │ │  completion_date (DATE)     │   │
              │         (assessor)          │─┘  progress_percentage (INT) │   │
              └─────────────────────────────┘    status (VARCHAR)           │   │
                                                 │  final_score (DECIMAL)      │   │
                                                 │  certificate_url (VARCHAR)  │   │
                                                 │  created_at (TIMESTAMP)    │   │
                                                 │  updated_at (TIMESTAMP)    │   │
                                                 └─────────────────────────────┘   │
                                                                                   │
                                                 ┌─────────────────────────────┐   │
                                                 │          COURSES            │   │
                                                 ├─────────────────────────────┤   │
                                                 │ *id (UUID) [PK]             │   │
                                                 │  title (VARCHAR)            │───┘
                                                 │  description (TEXT)         │
                                                 │  provider (VARCHAR)         │
                                                 │  duration_hours (INT)       │
                                                 │  difficulty_level (VARCHAR) │
                                                 │  created_at (TIMESTAMP)    │
                                                 └─────────────────────────────┘
```

### 6.4. FLOW 3: ODC Team Formation và Project Management

```
┌─────────────────────────────┐       ┌─────────────────────────────┐
│   TEAM_FORMATION_REQUESTS   │       │         ODC_TEAMS           │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │
│ *business_id (UUID) [FK]    │<──┐   │ *formation_request_id (FK)  │<──┐
│ *requirement_id (UUID) [FK] │<──┼─┐ │ *business_id (UUID) [FK]    │<──┼─┐
│  team_name (VARCHAR)        │   │ │ │  contract_id (UUID) [FK]    │<──┼─┼─┐
│  team_size (INT)            │   │ │ │  name (VARCHAR)             │   │ │ │
│  required_skills (JSON)     │   │ │ │  team_lead_id (UUID) [FK]   │<──┼─┼─┼─┐
│  project_duration (VARCHAR) │   │ │ │  workspace_id (UUID) [FK]   │<──┼─┼─┼─┼─┐
│  budget_allocated (DECIMAL) │   │ │ │  formation_date (DATE)      │   │ │ │ │ │
│  priority_level (VARCHAR)   │   │ │ │  status (VARCHAR)           │   │ │ │ │ │
│  requested_start_date (DATE)│   │ │ │  created_at (TIMESTAMP)     │   │ │ │ │ │
│  status (VARCHAR)           │   │ │ │  updated_at (TIMESTAMP)     │   │ │ │ │ │
│  approved_by (UUID) [FK]    │<──┼─┼─┼─└─────────────────────────────┘   │ │ │ │ │
│  created_at (TIMESTAMP)     │   │ │ │                   │               │ │ │ │ │
│  updated_at (TIMESTAMP)     │   │ │ │                   │ 1             │ │ │ │ │
└─────────────────────────────┘   │ │ │                   │               │ │ │ │ │
              │                   │ │ │                   │ *             │ │ │ │ │
              └───────────────────┼─┼─┼───────────────────┼───────────────┼─┼─┼─┼─┼─┐
                                  │ │ │   ┌─────────────────────────────┐ │ │ │ │ │ │
              ┌─────────────────────────────┐ │       TEAM_MEMBERS          │ │ │ │ │ │ │
              │         BUSINESSES          │ ├─────────────────────────────┤ │ │ │ │ │ │
              │         (reference)         │─┘ *id (UUID) [PK]             │ │ │ │ │ │ │
              └─────────────────────────────┘   │ *team_id (UUID) [FK]        │<┘ │ │ │ │ │
                                              │ *talent_id (UUID) [FK]      │<──┼─┼─┼─┼─┼─┐
              ┌─────────────────────────────┐   │  role_id (UUID) [FK]        │<──┼─┼─┼─┼─┼─┼─┐
              │   BUSINESS_REQUIREMENTS     │   │  join_date (DATE)           │   │ │ │ │ │ │ │
              │         (reference)         │───│  end_date (DATE)            │   │ │ │ │ │ │ │
              └─────────────────────────────┘   │  performance_score (DEC)    │   │ │ │ │ │ │ │
                                              │  status (VARCHAR)           │   │ │ │ │ │ │ │
              ┌─────────────────────────────┐   │  created_at (TIMESTAMP)     │   │ │ │ │ │ │ │
              │           USERS             │   │  updated_at (TIMESTAMP)     │   │ │ │ │ │ │ │
              │         (approver)          │───└─────────────────────────────┘   │ │ │ │ │ │ │
              └─────────────────────────────┘                                   │ │ │ │ │ │ │
                                                                                │ │ │ │ │ │ │
              ┌─────────────────────────────┐   ┌─────────────────────────────┐   │ │ │ │ │ │ │
              │          CONTRACTS          │   │          TALENTS            │   │ │ │ │ │ │ │
              │         (reference)         │───│         (reference)         │───┘ │ │ │ │ │ │
              └─────────────────────────────┘   └─────────────────────────────┘     │ │ │ │ │ │
                                                                                    │ │ │ │ │ │
              ┌─────────────────────────────┐   ┌─────────────────────────────┐     │ │ │ │ │ │
              │           USERS             │   │        TEAM_ROLES           │     │ │ │ │ │ │
              │        (team lead)          │───├─────────────────────────────┤     │ │ │ │ │ │
              └─────────────────────────────┘   │ *id (UUID) [PK]             │     │ │ │ │ │ │
                                              │  name (VARCHAR)             │─────┘ │ │ │ │ │
              ┌─────────────────────────────┐   │  description (TEXT)         │       │ │ │ │ │
              │      TEAM_WORKSPACES        │   │  responsibilities (JSON)    │       │ │ │ │ │
              ├─────────────────────────────┤   │  required_skills (JSON)     │       │ │ │ │ │
              │ *id (UUID) [PK]             │   │  created_at (TIMESTAMP)     │       │ │ │ │ │
              │  name (VARCHAR)             │───└─────────────────────────────┘       │ │ │ │ │
              │  location (VARCHAR)         │                                         │ │ │ │ │
              │  capacity (INT)             │                                         │ │ │ │ │
              │  equipment_list (JSON)      │                                         │ │ │ │ │
              │  availability_status (VAR)  │                                         │ │ │ │ │
              │  created_at (TIMESTAMP)     │                                         │ │ │ │ │
              └─────────────────────────────┘                                         │ │ │ │ │
                                                                                      │ │ │ │ │
┌─────────────────────────────┐       ┌─────────────────────────────┐               │ │ │ │ │
│          PROJECTS           │       │       PROJECT_PHASES        │               │ │ │ │ │
├─────────────────────────────┤       ├─────────────────────────────┤               │ │ │ │ │
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │               │ │ │ │ │
│  name (VARCHAR)             │       │ *project_id (UUID) [FK]     │<──┐           │ │ │ │ │
│ *business_id (UUID) [FK]    │<──────│  name (VARCHAR)             │   │           │ │ │ │ │
│  team_id (UUID) [FK]        │<──────│  description (TEXT)         │   │           │ │ │ │ │
│  contract_id (UUID) [FK]    │<──────│  start_date (DATE)          │   │           │ │ │ │ │
│  description (TEXT)         │       │  end_date (DATE)            │   │           │ │ │ │ │
│  methodology (VARCHAR)      │       │  status (VARCHAR)           │   │           │ │ │ │ │
│  start_date (DATE)          │       │  deliverables (JSON)        │   │           │ │ │ │ │
│  end_date (DATE)            │       │  created_at (TIMESTAMP)     │   │           │ │ │ │ │
│  budget (DECIMAL)           │       │  updated_at (TIMESTAMP)     │   │           │ │ │ │ │
│  status (VARCHAR)           │       └─────────────────────────────┘   │           │ │ │ │ │
│  created_at (TIMESTAMP)     │                   │                     │           │ │ │ │ │
│  updated_at (TIMESTAMP)     │                   │ 1                   │           │ │ │ │ │
└─────────────────────────────┘                   │                     │           │ │ │ │ │
              │                                   │ *                   │           │ │ │ │ │
              │ 1                 ┌─────────────────────────────┐       │           │ │ │ │ │
              │                   │       PROJECT_TASKS         │       │           │ │ │ │ │
              │ *                 ├─────────────────────────────┤       │           │ │ │ │ │
┌─────────────────────────────┐   │ *id (UUID) [PK]             │       │           │ │ │ │ │
│     PROJECT_MILESTONES      │   │ *project_id (UUID) [FK]     │<──────┘           │ │ │ │ │
├─────────────────────────────┤   │  phase_id (UUID) [FK]       │<──────────────────┘ │ │ │ │
│ *id (UUID) [PK]             │   │  title (VARCHAR)            │                     │ │ │ │
│ *project_id (UUID) [FK]     │<──│  description (TEXT)         │                     │ │ │ │
│  name (VARCHAR)             │   │  assignee_id (UUID) [FK]    │<────────────────────┼─┼─┼─┼─┐
│  description (TEXT)         │   │  start_date (DATE)          │                     │ │ │ │ │
│  due_date (DATE)            │   │  due_date (DATE)            │                     │ │ │ │ │
│  completion_date (DATE)     │   │  estimated_hours (INT)      │                     │ │ │ │ │
│  status (VARCHAR)           │   │  actual_hours (INT)         │                     │ │ │ │ │
│  deliverables (JSON)        │   │  status (VARCHAR)           │                     │ │ │ │ │
│  created_at (TIMESTAMP)     │   │  priority (VARCHAR)         │                     │ │ │ │ │
│  updated_at (TIMESTAMP)     │   │  created_at (TIMESTAMP)     │                     │ │ │ │ │
└─────────────────────────────┘   │  updated_at (TIMESTAMP)     │                     │ │ │ │ │
                                  └─────────────────────────────┘                     │ │ │ │ │
                                                                                      │ │ │ │ │
                                  ┌─────────────────────────────┐                     │ │ │ │ │
                                  │           USERS             │                     │ │ │ │ │
                                  │      (task assignee)        │─────────────────────┘ │ │ │ │
                                  └─────────────────────────────┘                       │ │ │ │
                                                                                        │ │ │ │
                                  ┌─────────────────────────────┐                       │ │ │ │
                                  │         BUSINESSES          │                       │ │ │ │
                                  │         (reference)         │───────────────────────┘ │ │ │
                                  └─────────────────────────────┘                         │ │ │
                                                                                          │ │ │
                                  ┌─────────────────────────────┐                         │ │ │
                                  │          CONTRACTS          │                         │ │ │
                                  │         (reference)         │─────────────────────────┘ │ │
                                  └─────────────────────────────┘                           │ │
                                                                                            │ │
                                  ┌─────────────────────────────┐                           │ │
                                  │          CONTRACTS          │                           │ │
                                  │         (reference)         │───────────────────────────┘ │
                                  └─────────────────────────────┘                             │
                                                                                              │
                                  ┌─────────────────────────────┐                             │
                                  │          CONTRACTS          │                             │
                                  │         (reference)         │─────────────────────────────┘
                                  └─────────────────────────────┘
```

### 6.5. FLOW 4: Marketplace và Learning Platform

```
┌─────────────────────────────┐       ┌─────────────────────────────┐
│     MARKETPLACE_PROJECTS    │       │    PROJECT_CATEGORIES       │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │
│ *business_id (UUID) [FK]    │<──┐   │  name (VARCHAR)             │
│ *category_id (UUID) [FK]    │<──┼─┐ │  description (TEXT)         │
│  title (VARCHAR)            │   │ │ │  parent_id (UUID) [FK]      │<─┐
│  description (TEXT)         │   │ │ │  created_at (TIMESTAMP)     │  │
│  required_skills (JSON)     │   │ │ │  updated_at (TIMESTAMP)     │  │
│  budget_min (DECIMAL)       │   │ │ └─────────────────────────────┘  │
│  budget_max (DECIMAL)       │   │ │                   │             │
│  duration_weeks (INT)       │   │ │                   └─────────────┘
│  project_type (VARCHAR)     │   │ │
│  complexity_level (VARCHAR) │   │ │ ┌─────────────────────────────┐
│  status (VARCHAR)           │   │ │ │         BUSINESSES          │
│  posted_at (TIMESTAMP)      │   │ │ │         (reference)         │
│  deadline (TIMESTAMP)       │   │ └─│                             │
│  created_at (TIMESTAMP)     │   │   └─────────────────────────────┘
│  updated_at (TIMESTAMP)     │   │
└─────────────────────────────┘   │   ┌─────────────────────────────┐
              │                   │   │   PROJECT_CATEGORIES        │
              │ 1                 │   │         (reference)         │
              │                   └───│                             │
              │ *                     └─────────────────────────────┘
┌─────────────────────────────┐
│ MARKETPLACE_PROJECT_SKILLS  │       ┌─────────────────────────────┐
├─────────────────────────────┤       │     MARKETPLACE_BIDS        │
│ *id (UUID) [PK]             │       ├─────────────────────────────┤
│ *project_id (UUID) [FK]     │<──┐   │ *id (UUID) [PK]             │
│ *skill_id (UUID) [FK]       │<──┼─┐ │ *project_id (UUID) [FK]     │<──┐
│  proficiency_level (VARCHAR)│   │ │ │ *talent_id (UUID) [FK]      │<──┼─┐
│  is_required (BOOLEAN)      │   │ │ │  bid_amount (DECIMAL)       │   │ │
│  weight (DECIMAL)           │   │ │ │  proposal_text (TEXT)       │   │ │
│  created_at (TIMESTAMP)     │   │ │ │  estimated_duration (INT)   │   │ │
└─────────────────────────────┘   │ │ │  portfolio_links (JSON)     │   │ │
              │                   │ │ │  status (VARCHAR)           │   │ │
              │ *                 │ │ │  submitted_at (TIMESTAMP)   │   │ │
              │                   │ │ │  reviewed_at (TIMESTAMP)    │   │ │
              │ 1                 │ │ │  created_at (TIMESTAMP)     │   │ │
┌─────────────────────────────┐   │ │ │  updated_at (TIMESTAMP)     │   │ │
│           SKILLS            │   │ │ └─────────────────────────────┘   │ │
│         (reference)         │───┘ │                                   │ │
└─────────────────────────────┘     │ ┌─────────────────────────────┐   │ │
                                    │ │    PROJECT_PROPOSALS        │   │ │
                                    │ ├─────────────────────────────┤   │ │
                                    │ │ *id (UUID) [PK]             │   │ │
                                    │ │ *project_id (UUID) [FK]     │<──┘ │
                                    │ │ *talent_id (UUID) [FK]      │<────┼─┐
                                    │ │  proposal_text (TEXT)       │     │ │
                                    │ │  proposed_budget (DECIMAL)  │     │ │
                                    │ │  estimated_timeline (INT)   │     │ │
                                    │ │  technical_approach (TEXT)  │     │ │
                                    │ │  deliverables (JSON)        │     │ │
                                    │ │  status (VARCHAR)           │     │ │
                                    │ │  submitted_at (TIMESTAMP)   │     │ │
                                    │ │  created_at (TIMESTAMP)     │     │ │
                                    │ │  updated_at (TIMESTAMP)     │     │ │
                                    │ └─────────────────────────────┘     │ │
                                    │                                     │ │
                                    │ ┌─────────────────────────────┐     │ │
                                    │ │          TALENTS            │     │ │
                                    │ │         (reference)         │─────┘ │
                                    │ └─────────────────────────────┘       │
                                    │                                       │
                                    │                                       │
                                    └───────────────────────────────────────┘

┌─────────────────────────────┐       ┌─────────────────────────────┐
│   MARKETPLACE_CONTRACTS     │       │      ESCROW_PAYMENTS        │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │
│ *project_id (UUID) [FK]     │<──┐   │ *contract_id (UUID) [FK]    │<──┐
│ *business_id (UUID) [FK]    │<──┼─┐ │ *milestone_id (UUID) [FK]   │<──┼─┐
│ *talent_id (UUID) [FK]      │<──┼─┼─│  amount (DECIMAL)           │   │ │
│  contract_value (DECIMAL)   │   │ │ │  payment_method (VARCHAR)   │   │ │
│  payment_terms (TEXT)       │   │ │ │  transaction_id (VARCHAR)   │   │ │
│  deliverables (JSON)        │   │ │ │  status (VARCHAR)           │   │ │
│  start_date (DATE)          │   │ │ │  held_at (TIMESTAMP)        │   │ │
│  end_date (DATE)            │   │ │ │  released_at (TIMESTAMP)    │   │ │
│  status (VARCHAR)           │   │ │ │  created_at (TIMESTAMP)     │   │ │
│  signed_at (TIMESTAMP)      │   │ │ │  updated_at (TIMESTAMP)     │   │ │
│  created_at (TIMESTAMP)     │   │ │ └─────────────────────────────┘   │ │
│  updated_at (TIMESTAMP)     │   │ │                                   │ │
└─────────────────────────────┘   │ │ ┌─────────────────────────────┐   │ │
              │                   │ │ │   MARKETPLACE_MILESTONES    │   │ │
              │ 1                 │ │ │         (reference)         │───┘ │
              │                   │ │ └─────────────────────────────┘     │
              │ *                 │ │                                     │
┌─────────────────────────────┐   │ │ ┌─────────────────────────────┐     │
│    MARKETPLACE_REVIEWS      │   │ │ │         BUSINESSES          │     │
├─────────────────────────────┤   │ │ │         (reference)         │─────┘
│ *id (UUID) [PK]             │   │ │ └─────────────────────────────┘
│ *contract_id (UUID) [FK]    │<──┘ │
│ *reviewer_id (UUID) [FK]    │<────┼─┐ ┌─────────────────────────────┐
│ *reviewee_id (UUID) [FK]    │<────┼─┼─│          TALENTS            │
│  rating (DECIMAL)           │     │ │ │         (reference)         │
│  review_text (TEXT)         │     │ │ └─────────────────────────────┘
│  review_type (VARCHAR)      │     │ │
│  created_at (TIMESTAMP)     │     │ │
│  updated_at (TIMESTAMP)     │     │ │
└─────────────────────────────┘     │ │
                                    │ │
                                    │ │
┌─────────────────────────────┐     │ │
│           USERS             │     │ │
│        (reviewer)           │─────┘ │
└─────────────────────────────┘       │
                                      │
┌─────────────────────────────┐       │
│           USERS             │       │
│        (reviewee)           │───────┘
└─────────────────────────────┘
```

### 6.6. FLOW 5: Learning Platform và Skill Development

```
┌─────────────────────────────┐       ┌─────────────────────────────┐
│          COURSES            │       │     COURSE_CATEGORIES       │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │
│ *instructor_id (UUID) [FK]  │<──┐   │  name (VARCHAR)             │
│ *category_id (UUID) [FK]    │<──┼─┐ │  description (TEXT)         │
│  title (VARCHAR)            │   │ │ │  parent_id (UUID) [FK]      │<─┐
│  description (TEXT)         │   │ │ │  skill_mapping (JSON)       │  │
│  learning_objectives (JSON) │   │ │ │  created_at (TIMESTAMP)     │  │
│  prerequisites (JSON)       │   │ │ │  updated_at (TIMESTAMP)     │  │
│  level (VARCHAR)            │   │ │ └─────────────────────────────┘  │
│  duration_hours (INT)       │   │ │                   │             │
│  price (DECIMAL)            │   │ │                   └─────────────┘
│  certification_offered (BOOL)│  │ │
│  max_students (INT)         │   │ │ ┌─────────────────────────────┐
│  status (VARCHAR)           │   │ │ │           USERS             │
│  published_at (TIMESTAMP)   │   │ │ │       (instructors)         │
│  created_at (TIMESTAMP)     │   │ └─│                             │
│  updated_at (TIMESTAMP)     │   │   └─────────────────────────────┘
└─────────────────────────────┘   │
              │                   │   ┌─────────────────────────────┐
              │ 1                 │   │     COURSE_CATEGORIES       │
              │                   │   │         (reference)         │
              │ *                 └───│                             │
┌─────────────────────────────┐       └─────────────────────────────┘
│      COURSE_MODULES         │
├─────────────────────────────┤       ┌─────────────────────────────┐
│ *id (UUID) [PK]             │       │     COURSE_ENROLLMENTS      │
│ *course_id (UUID) [FK]      │<──┐   ├─────────────────────────────┤
│  title (VARCHAR)            │   │   │ *id (UUID) [PK]             │
│  description (TEXT)         │   │   │ *user_id (UUID) [FK]        │<──┐
│  learning_objectives (JSON) │   │   │ *course_id (UUID) [FK]      │<──┼─┐
│  order_index (INT)          │   │   │  enrollment_type (VARCHAR)  │   │ │
│  estimated_duration (INT)   │   │   │  payment_status (VARCHAR)   │   │ │
│  is_mandatory (BOOLEAN)     │   │   │  enrolled_at (TIMESTAMP)    │   │ │
│  created_at (TIMESTAMP)     │   │   │  started_at (TIMESTAMP)     │   │ │
│  updated_at (TIMESTAMP)     │   │   │  completed_at (TIMESTAMP)   │   │ │
└─────────────────────────────┘   │   │  progress_percentage (INT)  │   │ │
              │                   │   │  current_module_id (UUID)   │   │ │
              │ 1                 │   │  status (VARCHAR)           │   │ │
              │                   │   │  certificate_issued (BOOL)  │   │ │
              │ *                 │   │  created_at (TIMESTAMP)     │   │ │
┌─────────────────────────────┐   │   │  updated_at (TIMESTAMP)     │   │ │
│      COURSE_LESSONS         │   │   └─────────────────────────────┘   │ │
├─────────────────────────────┤   │                                     │ │
│ *id (UUID) [PK]             │   │   ┌─────────────────────────────┐   │ │
│ *module_id (UUID) [FK]      │<──┘   │           USERS             │   │ │
│  title (VARCHAR)            │       │        (students)           │───┘ │
│  content_type (VARCHAR)     │       └─────────────────────────────┘     │
│  content_url (VARCHAR)      │                                           │
│  content_text (TEXT)        │                                           │
│  duration_minutes (INT)     │                                           │
│  order_index (INT)          │                                           │
│  is_preview (BOOLEAN)       │                                           │
│  quiz_questions (JSON)      │                                           │
│  resources (JSON)           │                                           │
│  created_at (TIMESTAMP)     │                                           │
│  updated_at (TIMESTAMP)     │                                           │
└─────────────────────────────┘                                           │
              │                                                           │
              │ 1                                                         │
              │                                                           │
              │ *                                                         │
┌─────────────────────────────┐       ┌─────────────────────────────┐     │
│     LEARNING_PROGRESS       │       │      SKILL_ASSESSMENTS      │     │
├─────────────────────────────┤       ├─────────────────────────────┤     │
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │     │
│ *user_id (UUID) [FK]        │<──────│ *user_id (UUID) [FK]        │<────┘
│ *lesson_id (UUID) [FK]      │<──────│ *skill_id (UUID) [FK]       │<──┐
│ *enrollment_id (UUID) [FK]  │<──┐   │  assessment_type (VARCHAR)  │   │
│  started_at (TIMESTAMP)     │   │   │  score (DECIMAL)            │   │
│  completed_at (TIMESTAMP)   │   │   │  max_score (DECIMAL)        │   │
│  time_spent_minutes (INT)   │   │   │  passed (BOOLEAN)           │   │
│  quiz_score (DECIMAL)       │   │   │  attempts (INT)             │   │
│  quiz_attempts (INT)        │   │   │  assessment_data (JSON)     │   │
│  notes (TEXT)               │   │   │  assessed_at (TIMESTAMP)    │   │
│  status (VARCHAR)           │   │   │  valid_until (TIMESTAMP)    │   │
│  created_at (TIMESTAMP)     │   │   │  created_at (TIMESTAMP)     │   │
│  updated_at (TIMESTAMP)     │   │   │  updated_at (TIMESTAMP)     │   │
└─────────────────────────────┘   │   └─────────────────────────────┘   │
                                  │                                     │
                                  │   ┌─────────────────────────────┐   │
                                  │   │           SKILLS            │   │
                                  │   │         (reference)         │───┘
                                  │   └─────────────────────────────┘
                                  │
                                  │   ┌─────────────────────────────┐
                                  │   │     COURSE_ENROLLMENTS      │
                                  │   │         (reference)         │
                                  └───│                             │
                                      └─────────────────────────────┘

┌─────────────────────────────┐       ┌─────────────────────────────┐
│    COURSE_CERTIFICATES      │       │      TRAINING_PROGRESS      │
├─────────────────────────────┤       ├─────────────────────────────┤
│ *id (UUID) [PK]             │       │ *id (UUID) [PK]             │
│ *user_id (UUID) [FK]        │<──┐   │ *talent_id (UUID) [FK]      │<──┐
│ *course_id (UUID) [FK]      │<──┼─┐ │ *skill_id (UUID) [FK]       │<──┼─┐
│ *enrollment_id (UUID) [FK]  │<──┼─┼─│  training_type (VARCHAR)    │   │ │
│  certificate_number (VARCHAR)│  │ │ │  current_level (VARCHAR)    │   │ │
│  issued_at (TIMESTAMP)      │   │ │ │  target_level (VARCHAR)     │   │ │
│  valid_until (TIMESTAMP)    │   │ │ │  progress_percentage (INT)  │   │ │
│  verification_url (VARCHAR) │   │ │ │  hours_completed (DECIMAL)  │   │ │
│  certificate_data (JSON)    │   │ │ │  last_activity (TIMESTAMP)  │   │ │
│  created_at (TIMESTAMP)     │   │ │ │  status (VARCHAR)           │   │ │
│  updated_at (TIMESTAMP)     │   │ │ │  created_at (TIMESTAMP)     │   │ │
└─────────────────────────────┘   │ │ │  updated_at (TIMESTAMP)     │   │ │
                                  │ │ └─────────────────────────────┘   │ │
┌─────────────────────────────┐   │ │                                   │ │
│           USERS             │   │ │ ┌─────────────────────────────┐   │ │
│         (students)          │───┘ │ │          TALENTS            │   │ │
└─────────────────────────────┘     │ │         (reference)         │───┘ │
                                    │ └─────────────────────────────┘     │
┌─────────────────────────────┐     │                                     │
│          COURSES            │     │ ┌─────────────────────────────┐     │
│         (reference)         │─────┘ │           SKILLS            │     │
└─────────────────────────────┘       │         (reference)         │─────┘
                                      └─────────────────────────────┘
┌─────────────────────────────┐
│     COURSE_ENROLLMENTS      │
│         (reference)         │
└─────────────────────────────┘
```
```

## 7. Kết luận

Database schema này được thiết kế để hỗ trợ đầy đủ các chức năng của nền tảng ODC lab-based với ERD được cập nhật hoàn chỉnh:

### 7.1. Tổng quan kiến trúc dữ liệu
- **Sử dụng UUID**: Tất cả các bảng chính đều sử dụng UUID làm khóa chính để đảm bảo tính duy nhất và bảo mật
- **Lựa chọn database**: 
  - PostgreSQL cho dữ liệu quan hệ chính
  - MongoDB cho dữ liệu phi cấu trúc (logs, analytics)
  - Elasticsearch cho tìm kiếm và phân tích

### 7.2. Hỗ trợ 5 luồng chính với ERD đầy đủ
1. **FLOW 1: Business Onboarding và Registration**
   - Quản lý đăng ký doanh nghiệp với business_registrations
   - Liên kết với contacts, requirements và contracts
   - Hỗ trợ skill matching và requirement management

2. **FLOW 2: Talent Management và Skill Assessment**
   - Quản lý talent applications và skill assessments
   - Theo dõi training progress và skill development
   - Liên kết với learning platform và certification

3. **FLOW 3: ODC Team Formation và Project Management**
   - Quản lý team formation requests và team assignments
   - Theo dõi project progress và deliverables
   - Hỗ trợ resource allocation và performance tracking

4. **FLOW 4: Marketplace và Learning Platform**
   - Quản lý marketplace projects với categories và bidding
   - Hỗ trợ project proposals và contract management
   - Tích hợp escrow payments và dispute resolution

5. **FLOW 5: Learning Platform và Skill Development**
   - Quản lý courses với categories và instructors
   - Theo dõi learning progress và assessments
   - Hỗ trợ certification và skill validation

### 7.3. Đảm bảo tính toàn vẹn và mở rộng
- **Foreign key constraints**: Đảm bảo tính toàn vẹn dữ liệu giữa các bảng
- **Validation rules**: Kiểm tra dữ liệu đầu vào và business logic
- **Scalability**: Thiết kế hỗ trợ mở rộng theo chiều ngang
- **Performance**: Tối ưu hóa với indexes và partitioning
- **Security**: Implement row-level security và audit trails

### 7.4. Tích hợp và API Support
- **RESTful APIs**: Hỗ trợ đầy đủ CRUD operations
- **GraphQL**: Flexible data querying cho frontend
- **Real-time updates**: WebSocket support cho notifications
- **Data analytics**: Integration với BI tools và reporting

Schema này cung cấp nền tảng vững chắc cho việc phát triển và vận hành nền tảng ODC lab-based với khả năng mở rộng và bảo trì cao.
        