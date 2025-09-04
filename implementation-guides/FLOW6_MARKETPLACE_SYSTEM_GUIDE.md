# Main Flow 6: Mini-Project Marketplace - Hướng Dẫn Nhiệm Vụ Chi Tiết

## 📋 Tổng Quan Flow

**Mục tiêu**: Xây dựng marketplace cho mini-projects nơi businesses có thể post projects nhỏ và talents có thể bid/apply để thực hiện, tạo ecosystem linh hoạt cho short-term engagements.

**Các bước chính**:
1. Project posting và categorization
2. Project discovery và search
3. Bidding và proposal submission
4. Talent selection và negotiation
5. Contract creation và agreement
6. Project execution và milestone tracking
7. Payment processing và escrow
8. Review và rating system

**Microservices liên quan**:
- Marketplace Service (chính)
- Project Service (project management)
- Bidding Service (bid management)
- Payment Service (payment processing)
- Contract Service (contract management)
- Review Service (rating system)
- Notification Service (alerts)
- Search Service (project discovery)

**Thời gian ước tính**: 4-5 tuần
**Độ ưu tiên**: Trung bình (Revenue generation feature)

---

## 🔧 NHIỆM VỤ BACKEND TEAM

### 1. Marketplace Service - Core Implementation

#### 1.1 Database Design & Models
**Nhiệm vụ**:
- Thiết kế bảng `marketplace_projects` để lưu mini-projects
- Thiết kế bảng `project_categories` để categorize projects
- Thiết kế bảng `project_bids` để manage bids
- Thiết kế bảng `project_proposals` để lưu detailed proposals
- Thiết kế bảng `marketplace_contracts` để track contracts
- Thiết kế bảng `project_milestones` để milestone tracking
- Thiết kế bảng `escrow_payments` để payment management
- Thiết kế bảng `marketplace_reviews` để review system

**Các trường cần có**:
- **marketplace_projects**: title, description, business_id (FK), category_id (FK), required_skills (JSON), budget_min, budget_max, currency, duration_days, complexity_level, status, posting_date, deadline, application_deadline, featured, view_count, application_count, created_at, updated_at
- **project_categories**: name, description, parent_category_id (FK), icon, color, skill_tags (JSON), average_budget_range, typical_duration, is_active
- **project_bids**: project_id (FK), talent_id (FK), bid_amount, currency, proposed_duration, bid_description, portfolio_links (JSON), availability_start, confidence_level, status, submitted_at, updated_at
- **project_proposals**: bid_id (FK), detailed_approach, timeline_breakdown (JSON), deliverables (JSON), additional_services (JSON), terms_conditions, revision_policy, communication_plan
- **marketplace_contracts**: project_id (FK), business_id (FK), talent_id (FK), contract_amount, currency, start_date, end_date, milestones (JSON), payment_schedule (JSON), terms (JSON), status, signed_date
- **project_milestones**: contract_id (FK), milestone_name, description, due_date, completion_date, payment_amount, status, deliverables (JSON), approval_status, feedback
- **escrow_payments**: contract_id (FK), milestone_id (FK), amount, currency, status, deposited_date, released_date, dispute_status, transaction_id
- **marketplace_reviews**: contract_id (FK), reviewer_id (FK), reviewee_id (FK), rating, review_text, review_categories (JSON), is_public, created_at, response_text, response_date

#### 1.2 Repository Layer
**Nhiệm vụ**:
- **MarketplaceProjectRepository**:
  - `findByStatus(ProjectStatus status)` - Lấy projects theo status
  - `findByCategory(Long categoryId)` - Lấy projects theo category
  - `findByBudgetRange(Double min, Double max)` - Lấy projects theo budget
  - `findBySkills(List<String> skills)` - Tìm projects theo skills
  - `findFeaturedProjects()` - Lấy featured projects
  - `findByBusinessId(Long businessId)` - Lấy projects của business
  - `findExpiringSoon()` - Lấy projects sắp hết hạn
  - `searchProjects(String query)` - Full-text search

- **ProjectBidRepository**:
  - `findByProjectId(Long projectId)` - Lấy bids của project
  - `findByTalentId(Long talentId)` - Lấy bids của talent
  - `findByStatus(BidStatus status)` - Lấy bids theo status
  - `findWinningBids(Long talentId)` - Lấy winning bids
  - `findByBudgetRange(Double min, Double max)` - Lấy bids theo budget
  - `findTopBidsForProject(Long projectId, int limit)` - Top bids

- **MarketplaceContractRepository**:
  - `findByBusinessId(Long businessId)` - Lấy contracts của business
  - `findByTalentId(Long talentId)` - Lấy contracts của talent
  - `findByStatus(ContractStatus status)` - Lấy contracts theo status
  - `findActiveContracts()` - Lấy active contracts
  - `findCompletedContracts()` - Lấy completed contracts
  - `findContractsNeedingPayment()` - Contracts cần payment

#### 1.3 Service Layer
**Nhiệm vụ**:
- **ProjectPostingService**:
  - `createProject()` - Tạo mini-project
  - `validateProjectDetails()` - Validate project info
  - `categorizeProject()` - Auto-categorize project
  - `estimateBudgetRange()` - Suggest budget range
  - `publishProject()` - Publish project
  - `updateProject()` - Update project details
  - `closeProject()` - Close project
  - `featureProject()` - Feature project (paid)

- **BiddingService**:
  - `submitBid()` - Submit bid
  - `validateBid()` - Validate bid details
  - `updateBid()` - Update existing bid
  - `withdrawBid()` - Withdraw bid
  - `rankBids()` - Rank bids for project
  - `selectWinningBid()` - Select winning bid
  - `notifyBidders()` - Notify về bid status
  - `generateBidAnalytics()` - Bid analytics

- **ContractManagementService**:
  - `createContract()` - Tạo contract từ winning bid
  - `negotiateTerms()` - Negotiate contract terms
  - `signContract()` - Digital contract signing
  - `trackMilestones()` - Track milestone progress
  - `processMilestonePayment()` - Process payments
  - `handleDisputes()` - Handle contract disputes
  - `completeContract()` - Complete contract
  - `archiveContract()` - Archive completed contract

- **PaymentEscrowService**:
  - `createEscrow()` - Tạo escrow account
  - `depositFunds()` - Deposit funds to escrow
  - `releaseFunds()` - Release funds to talent
  - `refundFunds()` - Refund to business
  - `handleDispute()` - Handle payment disputes
  - `calculateFees()` - Calculate platform fees
  - `generatePaymentReport()` - Payment reports
  - `processRefunds()` - Process refunds

#### 1.4 Controller Layer
**Nhiệm vụ**:
- **MarketplaceController**:
  - `POST /api/v1/marketplace/projects` - Post new project
  - `GET /api/v1/marketplace/projects` - Browse projects với filters
  - `GET /api/v1/marketplace/projects/{id}` - Project details
  - `PUT /api/v1/marketplace/projects/{id}` - Update project
  - `DELETE /api/v1/marketplace/projects/{id}` - Delete project
  - `POST /api/v1/marketplace/projects/{id}/feature` - Feature project
  - `GET /api/v1/marketplace/categories` - Lấy categories
  - `GET /api/v1/marketplace/search` - Search projects

- **BiddingController**:
  - `POST /api/v1/marketplace/projects/{projectId}/bids` - Submit bid
  - `GET /api/v1/marketplace/projects/{projectId}/bids` - Lấy bids (business)
  - `GET /api/v1/marketplace/bids/my-bids` - Lấy bids của talent
  - `PUT /api/v1/marketplace/bids/{id}` - Update bid
  - `DELETE /api/v1/marketplace/bids/{id}` - Withdraw bid
  - `POST /api/v1/marketplace/bids/{id}/select` - Select winning bid
  - `GET /api/v1/marketplace/bids/{id}/proposal` - Detailed proposal

- **ContractController**:
  - `POST /api/v1/marketplace/contracts` - Create contract
  - `GET /api/v1/marketplace/contracts` - Lấy contracts
  - `GET /api/v1/marketplace/contracts/{id}` - Contract details
  - `PUT /api/v1/marketplace/contracts/{id}` - Update contract
  - `POST /api/v1/marketplace/contracts/{id}/sign` - Sign contract
  - `POST /api/v1/marketplace/contracts/{id}/milestones/{milestoneId}/complete` - Complete milestone
  - `POST /api/v1/marketplace/contracts/{id}/dispute` - Raise dispute

- **PaymentController**:
  - `POST /api/v1/marketplace/payments/escrow` - Create escrow
  - `POST /api/v1/marketplace/payments/deposit` - Deposit funds
  - `POST /api/v1/marketplace/payments/release` - Release payment
  - `POST /api/v1/marketplace/payments/refund` - Request refund
  - `GET /api/v1/marketplace/payments/history` - Payment history
  - `GET /api/v1/marketplace/payments/balance` - Account balance

#### 1.5 Integration Tasks
**Nhiệm vụ**:
- **Với Payment Service**:
  - Process credit card payments
  - Handle bank transfers
  - Manage digital wallets
  - Process platform fees

- **Với Search Service**:
  - Index project data
  - Provide search capabilities
  - Auto-complete suggestions
  - Search analytics

- **Với Notification Service**:
  - Bid notifications
  - Contract updates
  - Payment notifications
  - Deadline reminders

- **Với Review Service**:
  - Collect project reviews
  - Calculate ratings
  - Manage reputation scores
  - Handle review disputes

#### 1.6 Business Logic & Algorithms
**Nhiệm vụ**:
- **Bid Ranking Algorithm**:
  - Price competitiveness
  - Talent reputation score
  - Proposal quality
  - Timeline feasibility
  - Past success rate

- **Project Recommendation**:
  - Skill-based matching
  - Budget compatibility
  - Timeline alignment
  - Success probability

- **Pricing Intelligence**:
  - Market rate analysis
  - Budget optimization
  - Price trend tracking
  - Competitive pricing

- **Fraud Detection**:
  - Suspicious bid patterns
  - Fake project detection
  - Payment fraud prevention
  - Account verification

---

## 🎨 NHIỆM VỤ FRONTEND TEAM

### 1. Project Posting Interface

#### 1.1 Project Creation Wizard
**Nhiệm vụ**:
- Multi-step project posting wizard:
  - **Step 1**: Project Basics (title, description, category)
  - **Step 2**: Requirements (skills, experience level)
  - **Step 3**: Budget & Timeline (budget range, duration)
  - **Step 4**: Additional Details (deliverables, milestones)
  - **Step 5**: Review & Publish

**Features cần có**:
- Rich text editor cho descriptions
- Skill autocomplete
- Budget calculator
- Timeline picker
- Preview mode
- Draft saving

#### 1.2 Project Management Dashboard
**Nhiệm vụ**:
- Business project management interface:
  - Posted projects overview
  - Bid management
  - Application tracking
  - Performance analytics
  - Quick actions

#### 1.3 Project Templates
**Nhiệm vụ**:
- Pre-built project templates:
  - Common project types
  - Industry-specific templates
  - Customizable templates
  - Template marketplace

### 2. Project Discovery Interface

#### 2.1 Project Marketplace
**Nhiệm vụ**:
- Main marketplace interface:
  - **Project Grid/List View**: Switchable layouts
  - **Advanced Filtering**: Category, budget, skills, timeline
  - **Search Functionality**: Full-text search với suggestions
  - **Sorting Options**: Date, budget, popularity, deadline
  - **Featured Projects**: Highlighted premium listings
  - **Saved Projects**: Bookmark functionality

#### 2.2 Project Detail Pages
**Nhiệm vụ**:
- Comprehensive project detail view:
  - Complete project description
  - Required skills visualization
  - Budget và timeline info
  - Business profile preview
  - Similar projects
  - Application/bid interface

#### 2.3 Advanced Search & Filters
**Nhiệm vụ**:
- Sophisticated search interface:
  - Multi-criteria filtering
  - Saved search preferences
  - Search alerts
  - Filter combinations
  - Search history

### 3. Bidding & Proposal Interface

#### 3.1 Bid Submission Form
**Nhiệm vụ**:
- Comprehensive bidding interface:
  - **Bid Amount**: Price input với currency selection
  - **Timeline Proposal**: Duration và milestones
  - **Approach Description**: Detailed methodology
  - **Portfolio Samples**: Relevant work examples
  - **Terms & Conditions**: Custom terms
  - **Communication Plan**: How you'll communicate

#### 3.2 Proposal Builder
**Nhiệm vụ**:
- Advanced proposal creation:
  - Template-based proposals
  - Rich media support
  - Interactive timelines
  - Cost breakdowns
  - Revision tracking

#### 3.3 Bid Management Dashboard
**Nhiệm vụ**:
- Talent bid tracking interface:
  - Active bids overview
  - Bid status tracking
  - Win/loss analytics
  - Performance metrics
  - Bid history

### 4. Contract & Payment Interface

#### 4.1 Contract Management
**Nhiệm vụ**:
- Digital contract interface:
  - Contract creation wizard
  - Terms negotiation
  - Digital signing
  - Milestone tracking
  - Amendment management

#### 4.2 Payment & Escrow
**Nhiệm vụ**:
- Payment management interface:
  - Escrow account dashboard
  - Payment scheduling
  - Transaction history
  - Dispute resolution
  - Fee transparency

#### 4.3 Milestone Tracking
**Nhiệm vụ**:
- Project progress tracking:
  - Milestone visualization
  - Progress indicators
  - Deliverable submission
  - Approval workflows
  - Payment triggers

### 5. Review & Rating System

#### 5.1 Review Interface
**Nhiệm vụ**:
- Comprehensive review system:
  - Multi-criteria ratings
  - Detailed feedback forms
  - Photo/video reviews
  - Response capabilities
  - Review moderation

#### 5.2 Reputation Dashboard
**Nhiệm vụ**:
- Reputation tracking interface:
  - Overall rating display
  - Review breakdown
  - Performance trends
  - Badge system
  - Improvement suggestions

---

## 📱 NHIỆM VỤ MOBILE TEAM

### 1. Mobile Marketplace

#### 1.1 Project Discovery
**Nhiệm vụ**:
- Mobile-optimized project browsing:
  - Swipe-based project cards
  - Quick filter options
  - Voice search capability
  - Offline project saving
  - Push notification alerts

#### 1.2 Quick Project Posting
**Nhiệm vụ**:
- Simplified mobile posting:
  - Voice-to-text descriptions
  - Photo-based requirements
  - Template selection
  - Quick budget setting
  - One-tap publishing

### 2. Mobile Bidding

#### 2.1 Quick Bid Submission
**Nhiệm vụ**:
- Streamlined mobile bidding:
  - Swipe-to-bid interface
  - Voice proposal recording
  - Photo portfolio upload
  - Quick price calculator
  - Instant submission

#### 2.2 Bid Notifications
**Nhiệm vụ**:
- Real-time bid updates:
  - Push notifications
  - In-app alerts
  - Bid status widgets
  - Quick response actions
  - Notification preferences

### 3. Mobile Contract Management

#### 3.1 Contract Tracking
**Nhiệm vụ**:
- Mobile contract interface:
  - Contract overview cards
  - Milestone progress
  - Payment tracking
  - Quick status updates
  - Document access

#### 3.2 Mobile Payments
**Nhiệm vụ**:
- Mobile payment interface:
  - Touch/Face ID payments
  - QR code payments
  - Payment notifications
  - Balance tracking
  - Transaction history

### 4. Mobile Communication

#### 4.1 Project Chat
**Nhiệm vụ**:
- Integrated communication:
  - Project-specific chats
  - File sharing
  - Voice messages
  - Video calls
  - Screen sharing

#### 4.2 Review & Rating
**Nhiệm vụ**:
- Mobile review interface:
  - Quick star ratings
  - Voice reviews
  - Photo reviews
  - Review templates
  - Review reminders

---

## 🔄 WORKFLOWS & PROCESSES

### 1. Project Lifecycle Workflow
**Steps**:
1. Project posting và validation
2. Project goes live
3. Talent discovery và bidding
4. Bid evaluation và selection
5. Contract negotiation
6. Contract signing
7. Project execution
8. Milestone completion
9. Payment processing
10. Project completion
11. Review exchange
12. Project archival

### 2. Bidding Process Workflow
**Steps**:
1. Project discovery
2. Bid preparation
3. Proposal submission
4. Bid evaluation
5. Shortlisting
6. Interview/clarification
7. Final selection
8. Contract initiation

### 3. Payment & Escrow Workflow
**Steps**:
1. Contract signing
2. Escrow account creation
3. Initial payment deposit
4. Milestone completion
5. Deliverable review
6. Payment approval
7. Fund release
8. Platform fee deduction
9. Final settlement

### 4. Dispute Resolution Workflow
**Steps**:
1. Dispute initiation
2. Evidence collection
3. Mediation attempt
4. Arbitration process
5. Decision rendering
6. Resolution implementation
7. Case closure
8. Feedback collection

---

## 📋 CHECKLIST TRIỂN KHAI

### Backend Tasks:
- [ ] Thiết kế database schema cho marketplace
- [ ] Implement project posting system
- [ ] Implement bidding mechanism
- [ ] Setup contract management
- [ ] Implement payment & escrow system
- [ ] Setup review & rating system
- [ ] Implement search functionality
- [ ] Setup notification system
- [ ] Implement fraud detection
- [ ] Setup analytics tracking
- [ ] Implement dispute resolution
- [ ] Setup integration với Payment Service
- [ ] Viết comprehensive tests

### Frontend Tasks:
- [ ] Thiết kế project posting interface
- [ ] Implement marketplace browsing
- [ ] Tạo bidding interface
- [ ] Implement contract management UI
- [ ] Tạo payment interface
- [ ] Implement review system UI
- [ ] Setup search và filtering
- [ ] Implement real-time updates
- [ ] Setup responsive design
- [ ] Implement data visualization
- [ ] Setup performance optimization
- [ ] Viết unit tests cho components

### Mobile Tasks:
- [ ] Thiết kế mobile marketplace interface
- [ ] Implement mobile bidding
- [ ] Setup mobile payments
- [ ] Implement push notifications
- [ ] Setup offline functionality
- [ ] Implement camera integration
- [ ] Setup voice features
- [ ] Implement gesture controls
- [ ] Optimize performance
- [ ] Viết mobile-specific tests

### Payment Integration Tasks:
- [ ] Setup payment gateway integration
- [ ] Implement escrow functionality
- [ ] Setup multi-currency support
- [ ] Implement fee calculation
- [ ] Setup fraud prevention
- [ ] Implement refund processing
- [ ] Setup payment analytics
- [ ] Test payment workflows

### Integration Tasks:
- [ ] Test complete marketplace workflow
- [ ] Verify payment integration
- [ ] Test search functionality
- [ ] Verify notification delivery
- [ ] Test review system
- [ ] Performance testing với high load
- [ ] Security testing cho financial data
- [ ] User acceptance testing

---

## 🎯 DEFINITION OF DONE

### Backend:
- [ ] Project posting system functional
- [ ] Bidding mechanism working
- [ ] Contract management operational
- [ ] Payment & escrow system secure
- [ ] Review system comprehensive
- [ ] Search functionality effective
- [ ] All integrations stable
- [ ] Performance requirements met
- [ ] Security implemented
- [ ] Fraud detection active

### Frontend:
- [ ] Intuitive project posting flow
- [ ] Effective marketplace browsing
- [ ] Smooth bidding experience
- [ ] Clear contract management
- [ ] Secure payment interface
- [ ] Comprehensive review system
- [ ] Responsive design working
- [ ] Real-time updates functional
- [ ] Accessibility standards met

### Mobile:
- [ ] Smooth mobile marketplace experience
- [ ] Effective mobile bidding
- [ ] Secure mobile payments
- [ ] Push notifications working
- [ ] Offline functionality stable
- [ ] Performance optimized
- [ ] Platform guidelines followed

### Payment & Security:
- [ ] Payment processing secure
- [ ] Escrow system reliable
- [ ] Fraud detection effective
- [ ] Data encryption implemented
- [ ] PCI compliance achieved
- [ ] Financial audit passed
- [ ] Dispute resolution working

### Integration:
- [ ] End-to-end workflows functioning
- [ ] Payment integration stable
- [ ] Search performance acceptable
- [ ] Notification delivery reliable
- [ ] Data consistency maintained
- [ ] Security requirements satisfied
- [ ] User acceptance criteria met

---

## 🚀 SUCCESS METRICS

### Marketplace Activity:
- Project posting rate > 50 projects/month
- Bid submission rate > 5 bids/project
- Project completion rate > 80%
- User retention rate > 70%

### Financial Performance:
- Transaction volume growth > 20%/month
- Average project value > $500
- Platform fee collection > 95%
- Payment success rate > 99%

### User Satisfaction:
- Business satisfaction > 4.2/5
- Talent satisfaction > 4.0/5
- Review participation rate > 60%
- Dispute rate < 5%

### System Performance:
- Search response time < 2 seconds
- Payment processing time < 30 seconds
- Page load time < 3 seconds
- System uptime > 99.5%

---

**Lưu ý quan trọng**:
- Marketplace là revenue-generating feature quan trọng
- Payment security là critical priority
- User trust là foundation của marketplace success
- Fraud prevention cần được implement từ đầu
- Scalability cần được consider cho future growth
- Legal compliance với payment regulations là mandatory