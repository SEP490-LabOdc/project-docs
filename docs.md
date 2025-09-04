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

## Breakthrough Extended Features

### 1. Skill-Based Matching System
This system helps optimize the connection between candidates from the Talent Pool and business requirements.
* **Classification & Assessment:** Classify and evaluate candidate skills based on standardized criteria and automated tests.
* **Intelligent Recommendations:** Suggest the most suitable candidates based on skill match and job requirements.
* **Intuitive Dashboard:** Provide a dashboard for businesses to easily compare and select.

**Value Delivered:** Reduced search time, increased success rate of ODC teams.

### 2. Marketplace for Mini-Projects
This is where businesses can post and hire ODC Teams for small and medium-sized projects.
* **Project Posting:** Businesses post projects with detailed technical requirements.
* **Bidding Mechanism:** ODC Teams can bid and propose solutions.
* **Integrated Management:** Project management system integrated with popular tools.
* **Evaluation & Ranking:** Businesses can evaluate teams after project completion.

**Value Delivered:** Expanded customer base, created real-world opportunities for the Talent Pool, increased revenue from small projects.

### 3. Interactive Learning Platform
A modern learning platform that accelerates the training process and standardizes knowledge.
* **Diverse Content:** Rich library of courses and materials for each technology.
* **Automated Practice:** Practical exercises with automated checking systems and sandbox.
* **Gamification:** Using point systems, leaderboards, and rewards to increase learning motivation.

**Value Delivered:** Enhanced training quality, standardized processes, increased course completion rates for trainees.
# Actors

## 1. Lab Management
* **System Management**: Manage system settings, user roles, and access permissions.
* **Financial Monitoring**: Track payment transactions, revenue from recruitment/training services.
* **Content Management**: Approve and manage job postings, training modules, and project proposals from businesses.
* **Overall System Operation**: Responsible for operating the entire platform.

## 2. Business
* **Create Jobs & Projects**: Propose and submit job positions and ODC projects for approval.
* **Team & Project Management**: Track recruitment progress, onboard candidates, assign tasks, and monitor project milestones.
* **Communication**: Send notifications, onboarding documents, and updates to recruited IT personnel.
* **Use ODC Services**: Customers using ODC services.

## 3. Talent Pool
* **Account & Profile Management**: Register and manage professional profiles, skills, certifications, and portfolios.
* **Participate in Recruitment & Training**: Apply for jobs, participate in training modules, and join ODC project teams.
* **Track Rewards**: View achievement points, badges, and monitor reward redemptions.
* **Join the System**: Students, freshers, junior developers participating in the system.

## 4. Mentor/Senior
* **Create Training Modules**: Design, submit, and manage training courses for approval.
* **Provide Training**: Track candidate registrations, progress, and evaluations.
* **Communication**: Send materials, updates, and feedback to candidates.
* **Guide and Ensure Quality**: Mentors and quality assurance for teams.

## 5. ODC Team
* **Implement Projects**: Development teams organized within the Lab.
* **Report Progress**: Update work and project progress.
* **Participate in Bidding**: Participate in bidding for mini-projects on the marketplace.

## 6. Platform System
* **Platform System**: Manage and operate the entire process from recruitment, training to handover.
* **Automation**: Automate matching processes, evaluations, and reporting.
* **Data Analysis**: Provide insights and reports to stakeholders.
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

### Flow 1: Onboarding New Businesses
- Businesses register to use the service
- Lab Management receives and analyzes requirements
- Agreement on team size, necessary skills, timeline, and costs
- Contract signing and account setup on the platform

### Flow 2: Building and Training the Talent Pool
- Lab Management recruits candidates into the Talent Pool
- Skills assessment and candidate classification
- Design of training programs tailored to business requirements
- Mentors/Seniors conduct training and evaluation

### Flow 3: Forming and Operating ODC Teams
- Lab Management selects members from the Talent Pool
- Organization into mini-ODC teams with appropriate structures
- Assignment of Mentors/Seniors to oversee each team
- Establishment of work processes and management tools

### Flow 4: Project Management and Progress Tracking
- Businesses assign tasks and requirements to ODC Teams through the platform
- ODC Teams perform work under the guidance of Mentors/Seniors
- Businesses monitor progress and work quality on the platform
- Lab Management ensures quality and resolves emerging issues

### Flow 5: Evaluation and Improvement
- Businesses evaluate the performance of ODC Teams
- Lab Management collects feedback from stakeholders
- Mentors/Seniors propose improvement measures
- Process and personnel adjustments if necessary

### Flow 6: Personnel Transfer
- Businesses request formal recruitment of members from ODC Teams
- Lab Management evaluates and negotiates transfer conditions
- Processing of procedures and placement fees
- Completion of the transfer process and onboarding support

### Flow 7: Skill-Based Matching
- Businesses provide detailed requirements for skills and projects
- The system analyzes and matches with Talent Pool data
- Proposes a list of the most suitable candidates with matching ratios

### Flow 8: Mini-Project Marketplace
- Businesses post projects on the marketplace with detailed requirements
- ODC Teams review projects and submit solution proposals
- Project implementation with integrated management processes

### Flow 9: Interactive Learning
- Lab Management creates customized training programs
- Talent Pool registers and participates in courses
- The system automatically evaluates and provides feedback
