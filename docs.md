# Context & Problem Statement

In the era of digital transformation and globalization, businesses increasingly need access to high-quality IT talent while optimizing costs.

The Offshore Development Center (ODC) model allows businesses to hire remote development teams from different countries, facilitating flexible and efficient project implementation.

However, recruiting, training, and managing IT personnel under the ODC model still faces many challenges due to the lack of a comprehensive support system:

- Businesses struggle to find and select IT candidates who meet project requirements.
- There are gaps in training and enhancing IT staff skills to meet international business standards.
- Managing remote work processes, contracts, and performance evaluations becomes complex without an integrated system.
- The absence of a unified platform to connect businesses, candidates, and training providers leads to inefficiency.

# Our Solution

## LabOdc: Next-Generation ODC Platform

The **Lab-based ODC Platform (LabOdc)** model is a unique combination of high-quality personnel training and software development team provision following the **Offshore Development Center (ODC)** model for businesses. Instead of merely providing personnel, LabOdc creates a closed-loop process, ensuring quality from start to finish.

* **Building a Talent Pool:** LabOdc proactively recruits and nurtures a young, potential workforce (students, freshers, junior developers).
* **Specialized Training on Demand:** They receive in-depth training according to specific business requirements regarding skills, technology (tech stack), tools, and work processes (Agile/Scrum).
* **Operating mini-ODC Teams:** Trainees are organized into development teams (mini-ODC Teams) right within the Lab, directly guided by experienced **mentors/seniors** to ensure output quality.

### Benefits for Businesses
With LabOdc, businesses can:
* **Save Time and Cost:** Quickly obtain a development team with the right expertise without spending initial training costs and time.
* **Flexible Management:** Directly manage team progress and work through the platform.
* **Quality Assurance:** LabOdc serves as a bridge and guarantor, managing the entire process from recruitment and training to handover.
* **Flexible Contracts:** Diverse contracts suitable for the needs of each project (monthly hire, project-based, or official recruitment).

## Key Features of LabODC

### 1. Skill-Based Matching System
This system helps optimize the connection between candidates from the Talent Pool and business requirements.
* **Classification & Assessment:** Classify and evaluate candidate skills based on standardized criteria and automated tests.
* **Intelligent Recommendations:** Suggest the most suitable candidates based on skill match and job requirements.
* **Intuitive Dashboard:** Provide a dashboard for businesses to easily compare and select.
* **Matching Algorithm:** Analyze and match Talent Pool data with detailed requirements for skills and projects.
* **Matching Ratio:** Display a list of the most suitable candidates with specific matching ratios.

**Value Delivered:** Reduced search time, increased success rate of ODC teams.

### 2. Marketplace for Mini-Projects
This is where businesses can post and hire ODC Teams for small and medium-sized projects.
* **Project Posting:** Businesses post projects with detailed technical requirements.
* **Bidding Mechanism:** ODC Teams can bid and propose solutions.
* **Integrated Management:** Project management system integrated with popular tools.
* **Evaluation & Ranking:** Businesses can evaluate teams after project completion.
* **Project Approval:** Lab Management reviews and approves projects before posting to the marketplace.

**Value Delivered:** Expands client base, creates practical opportunities for the Talent Pool, increases revenue from small projects.

### 3. Interactive Learning Platform
A modern learning platform that accelerates the training process and standardizes knowledge.
* **Diverse Content:** Rich library of courses and materials for each technology.
* **Automated Practice:** Practical exercises with automated checking systems and sandbox environments.
* **Gamification:** Point systems, leaderboards, and rewards to increase learning motivation.
* **Automated Assessment:** System automatically evaluates and provides feedback to learners.
* **Certificates and Badges:** Issue certificates and badges when learners complete courses.

**Value Delivered:** Improves training quality, standardizes processes, increases course completion rates.
# Actors

## 1. Lab Management
* **System Management**: Manage system settings, user roles, and access permissions.
* **Financial Monitoring**: Track payment transactions, revenue from recruitment/training services.
* **Content Management**: Approve and manage job postings, training modules, and project proposals from businesses.
* **System Operations**: Responsible for operating the entire platform.
* **Personnel Selection**: Recruit candidates into the Talent Pool and evaluate skills.

## 2. Business
* **Create Jobs & Projects**: Propose and submit job positions and ODC projects for approval.
* **Team & Project Management**: Track recruitment progress, onboard candidates, assign tasks, and monitor project milestones.
* **Communication**: Send notifications, onboarding documents, and updates to recruited IT personnel.
* **Use ODC Services**: Clients using ODC services.
* **Performance Evaluation**: Evaluate ODC Team performance and provide feedback.

## 3. Talent Pool
* **Account & Profile Management**: Register and manage professional profiles, skills, certificates, and portfolios.
* **Participate in Recruitment & Training**: Apply for jobs, participate in training modules, and join ODC project teams.
* **Track Rewards**: View achievement points, badges, and monitor reward redemption.
* **Join the System**: Students, freshers, junior developers joining the system.
* **Skill Development**: Participate in courses and practice to enhance skills.

## 4. Mentor/Senior
* **Create Training Modules**: Design, submit, and manage training courses for approval.
* **Provide Training**: Monitor candidate registrations, progress, and evaluations.
* **Communication**: Send materials, updates, and feedback to candidates.
* **Guide and Ensure Quality**: Guide and ensure quality for teams.
* **Suggest Improvements**: Propose measures to improve processes and personnel.

## 5. ODC Team
* **Implement Projects**: Development teams organized within the Lab.
* **Report Progress**: Update work and project progress.
* **Participate in Bidding**: Participate in bidding for small projects on the marketplace.
* **Follow Processes**: Adhere to work processes and use management tools.

## 6. Platform System
* **Platform System**: Manage and operate the entire process from recruitment, training to handover.
* **Automation**: Automate matching processes, evaluations, and reporting.
* **Data Analysis**: Provide insights and reports to stakeholders.
* **Permission Management**: Assign permissions to different user roles in the system.
# Non-functional requirements

* **System Reliability**: 99.0% uptime for both the candidate/business portal and the admin dashboard.
* **Performance**: Portal and dashboard must be highly responsive, with job/project listings loading in under 5 seconds; payment processing under 3 seconds.
* **Scalability**: The system must support hundreds of simultaneous users and a large volume of applications and training sessions during peak recruitment or onboarding periods.
* **Security**: All user data, especially payment information, must be encrypted.
* **User Experience (UX)**: The portal must have an intuitive, professional interface for candidates and businesses. The admin dashboard must be simple for administrators and trainers.
# Main Proposal Content (System Requirements and Deliverables)

## Documentation Requirements
* Complete software development lifecycle documentation using UML 2.0.
* Required documents: 
  * User Requirements Document (URD)
  * Software Requirements Specification (SRS)
  * Software Architecture Document (SAD)
  * Detailed Design Document (DDD)
  * Implementation Documentation
  * Test Documentation
  * Installation Guide
  * Source Code Documentation
  * Deployment Package Documentation

## Technical Stack
* **Backend**: Spring Boot
* **Database**: PostgreSQL, MongoDB, Redis, Elasticsearch
* **Mobile App (Frontend)**: Flutter
* **Web (Frontend)**: React
* **Message Broker**: Apache Kafka
* **Third-party Services**: PayOS, Cloudinary
* **Deployment**: Docker, AWS

## System Components

### Applications
* **LabOdc Portal**: For candidates and businesses (web-based).
* **Admin Dashboard**: Web application for administrators and trainers.
* **Mobile App**: Mobile application for candidates and mentors.
* **Backend Services**: RESTful APIs to process all data and business logic.

### Data Storage
* **PostgreSQL**: Stores user data, job/project information, transactions (payments/rewards), and training modules.
* **MongoDB**: Stores unstructured data such as personal profiles, feedback, and evaluations.
* **Redis**: Stores cache to improve performance.
* **Elasticsearch**: Used for full-text search and data analysis.
* **Cloudinary**: Stores user avatars, training materials, and portfolio images.

## System Architecture

### Microservices Architecture
* **API Gateway**: Spring Cloud Gateway - Single entry point for all requests.
* **Service Registry**: Netflix Eureka - Service registration and discovery.
* **Configuration Server**: Spring Cloud Config - Centralized configuration management.
* **Message Broker**: Apache Kafka - Asynchronous communication between microservices.
* **Authentication**: JWT (JSON Web Tokens) - User session security.

### Microservices
1. **User Service**: User management, authentication, and authorization
2. **Talent Pool Service**: Human resource management, skills, and training
3. **Business Service**: Business information and requirements management
4. **ODC Team Service**: ODC team management, assignment, and tracking
5. **Project Service**: Project management, progress, and reporting
6. **Skill Matching Service**: Skill-based matching system
7. **Marketplace Service**: Mini-project marketplace management
8. **Learning Service**: Interactive learning platform
9. **Notification Service**: Notification and alert management
10. **Analytics Service**: Data analysis and reporting

## Main Business Flows

### 1. New Business Onboarding
- Business registers to use the service
- Lab Management receives and analyzes requirements
- Agreement on team size, necessary skills, timeline, and costs
- Contract signing and platform account setup
- Business Service calls Contract Service to create a new contract
- Contract Service creates PDF file and adds signature fields
- Notification Service sends notifications to the business

### 2. Building and Training Talent Pool
- Lab Management recruits candidates into the Talent Pool
- Skills assessment and candidate classification
- Design of training programs tailored to business requirements
- Mentor/Senior conducts training and evaluation
- Learning Service manages training content and learning progress
- Talent Pool Service stores skill information and evaluations

### 3. Establishing and Operating ODC Teams
- Lab Management selects members from the Talent Pool
- Organizes into mini-ODC teams with appropriate structure
- Assigns Mentor/Senior to oversee each team
- Establishes work processes and management tools
- ODC Team Service manages team structure and members
- Project Service integrates with project management tools

### 4. Project Management and Progress Tracking
- Business assigns tasks and requirements to ODC Team through the platform
- ODC Team performs work under Mentor/Senior guidance
- Business monitors progress and work quality on the platform
- Lab Management ensures quality and resolves arising issues
- Project Service provides dashboards and progress reports
- Analytics Service analyzes performance and quality

### 5. Evaluation and Improvement
- Business evaluates ODC Team performance
- Lab Management collects feedback from stakeholders
- Mentor/Senior proposes improvement measures
- Adjusts processes and personnel if necessary
- Analytics Service creates detailed evaluation reports
- Notification Service sends evaluation result notifications

### 6. Personnel Transfer
- Business requests official recruitment of members from ODC Team
- Lab Management evaluates and negotiates transfer conditions
- Processes procedures and placement fees
- Completes transfer process and supports onboarding
- Contract Service creates transfer agreement
- Business Service and Talent Pool Service update status

### 7. Skill-Based Matching
- Business provides detailed requirements for skills and projects
- Lab Management creates requirement profile on matching system
- System analyzes and matches with Talent Pool data
- Suggests list of most suitable candidates with matching ratios
- Business views detailed profiles and selects candidates
- Lab Management adjusts criteria based on feedback
- Skill Matching Service implements matching algorithm

### 8. Mini-Project Marketplace
- Business posts projects on marketplace with detailed requirements
- Lab Management reviews and approves projects
- ODC Teams review projects and submit solution proposals
- Business selects the most suitable team
- Implements project with integrated management process
- Evaluation and ranking after completion
- Marketplace Service manages project listings and bidding

### 9. Interactive Learning
- Lab Management creates customized training programs
- Talent Pool registers and participates in courses
- Learners complete lessons, exercises, and practical projects
- System automatically evaluates and provides feedback
- Mentor reviews progress and provides support when needed
- Issues certificates and badges upon completion
- Learning Service manages content and automated assessment
