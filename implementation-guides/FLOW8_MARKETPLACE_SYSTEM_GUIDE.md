# FLOW8: Mini-Project Marketplace Implementation Guide

## Overview
The Mini-Project Marketplace is a breakthrough feature that creates a platform for businesses to post small projects and ODC Teams to bid on them. This system expands the client base, provides practical opportunities for the Talent Pool, and generates additional revenue streams.

## Business Flow Description
1. **Project Posting**: Business posts projects with detailed technical requirements
2. **Project Approval**: Lab Management reviews and approves projects before posting
3. **Project Browsing**: ODC Teams review available projects
4. **Proposal Submission**: Teams submit comprehensive solution proposals
5. **Team Selection**: Business selects the most suitable team through bidding process
6. **Project Execution**: Implementation with integrated management process
7. **Evaluation**: Assessment and ranking after completion

## Key Features
- **Project Management**: Detailed technical requirements and specifications
- **Bidding System**: Competitive proposal submission and evaluation
- **Approval Workflow**: Lab Management oversight and quality control
- **Budget Management**: Fair pricing mechanisms and budget ranges
- **Skill Tagging**: Match projects with appropriate team skills
- **Contract Generation**: Automatic contract creation upon bid acceptance
- **Progress Tracking**: Built-in milestone and deliverable tracking
- **Integration**: Seamless connection with Project Service

## Technical Implementation

### Database Schema
```sql
-- Projects table
CREATE TABLE marketplace_projects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    business_id UUID REFERENCES businesses(id),
    title VARCHAR(255) NOT NULL,
    description TEXT NOT NULL,
    technical_requirements JSONB,
    budget_min DECIMAL(10,2),
    budget_max DECIMAL(10,2),
    deadline DATE,
    required_skills TEXT[],
    status VARCHAR(50) DEFAULT 'pending_approval',
    approved_by UUID REFERENCES users(id),
    approved_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Project proposals/bids
CREATE TABLE project_proposals (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID REFERENCES marketplace_projects(id),
    team_id UUID REFERENCES odc_teams(id),
    proposed_budget DECIMAL(10,2),
    timeline_weeks INTEGER,
    solution_approach TEXT,
    team_composition JSONB,
    portfolio_links TEXT[],
    status VARCHAR(50) DEFAULT 'submitted',
    submitted_at TIMESTAMP DEFAULT NOW(),
    evaluated_at TIMESTAMP,
    evaluation_score INTEGER
);

-- Project contracts
CREATE TABLE marketplace_contracts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID REFERENCES marketplace_projects(id),
    proposal_id UUID REFERENCES project_proposals(id),
    contract_terms JSONB,
    signed_at TIMESTAMP,
    status VARCHAR(50) DEFAULT 'draft'
);
```

### API Endpoints

#### Project Management
```typescript
// POST /api/marketplace/projects
// Create new project (Business)
interface CreateProjectRequest {
  title: string;
  description: string;
  technicalRequirements: TechnicalRequirement[];
  budgetRange: { min: number; max: number };
  deadline: string;
  requiredSkills: string[];
}

// GET /api/marketplace/projects
// List projects (filtered by status, skills, budget)
interface ProjectListQuery {
  status?: 'approved' | 'pending' | 'in_progress' | 'completed';
  skills?: string[];
  budgetMin?: number;
  budgetMax?: number;
  page?: number;
  limit?: number;
}

// PUT /api/marketplace/projects/:id/approve
// Approve project (Lab Management)
interface ApproveProjectRequest {
  approved: boolean;
  feedback?: string;
  suggestedChanges?: string[];
}
```

#### Bidding System
```typescript
// POST /api/marketplace/projects/:id/proposals
// Submit proposal (ODC Team)
interface SubmitProposalRequest {
  proposedBudget: number;
  timelineWeeks: number;
  solutionApproach: string;
  teamComposition: TeamMember[];
  portfolioLinks: string[];
}

// GET /api/marketplace/projects/:id/proposals
// Get proposals for project (Business, Lab Management)
interface ProposalResponse {
  id: string;
  teamInfo: TeamInfo;
  proposedBudget: number;
  timeline: number;
  approach: string;
  teamSkills: SkillMatch[];
  portfolioSamples: Portfolio[];
  evaluationScore?: number;
}

// PUT /api/marketplace/proposals/:id/select
// Select winning proposal (Business)
interface SelectProposalRequest {
  selected: boolean;
  feedback?: string;
}
```

### Frontend Components

#### Business Interface
```typescript
// ProjectPostingForm.tsx
interface ProjectPostingFormProps {
  onSubmit: (project: ProjectData) => void;
  skillOptions: Skill[];
}

// ProjectProposalsList.tsx
interface ProjectProposalsListProps {
  projectId: string;
  proposals: Proposal[];
  onSelectProposal: (proposalId: string) => void;
}

// ProjectDashboard.tsx
interface ProjectDashboardProps {
  projects: Project[];
  onViewDetails: (projectId: string) => void;
}
```

#### ODC Team Interface
```typescript
// MarketplaceBrowser.tsx
interface MarketplaceBrowserProps {
  projects: Project[];
  filters: ProjectFilters;
  onFilterChange: (filters: ProjectFilters) => void;
}

// ProposalSubmissionForm.tsx
interface ProposalSubmissionFormProps {
  project: Project;
  teamInfo: TeamInfo;
  onSubmit: (proposal: ProposalData) => void;
}

// TeamProposalsTracker.tsx
interface TeamProposalsTrackerProps {
  proposals: Proposal[];
  onViewFeedback: (proposalId: string) => void;
}
```

#### Lab Management Interface
```typescript
// ProjectApprovalQueue.tsx
interface ProjectApprovalQueueProps {
  pendingProjects: Project[];
  onApprove: (projectId: string, approved: boolean) => void;
}

// MarketplaceAnalytics.tsx
interface MarketplaceAnalyticsProps {
  metrics: MarketplaceMetrics;
  timeRange: DateRange;
}
```

## Integration Points

### With Project Service
```typescript
// Auto-create project in Project Service when proposal is accepted
interface ProjectServiceIntegration {
  createProjectFromMarketplace(contractId: string): Promise<ProjectResponse>;
  syncProjectProgress(marketplaceProjectId: string): Promise<void>;
  generateProjectReports(projectId: string): Promise<ReportData>;
}
```

### With Skill Matching Service
```typescript
// Match projects with suitable teams based on skills
interface SkillMatchingIntegration {
  findSuitableTeams(projectRequirements: SkillRequirement[]): Promise<TeamMatch[]>;
  calculateTeamProjectFit(teamId: string, projectId: string): Promise<MatchScore>;
}
```

### With Contract Service
```typescript
// Generate contracts automatically
interface ContractServiceIntegration {
  generateMarketplaceContract(proposalId: string): Promise<ContractData>;
  processContractSigning(contractId: string): Promise<SigningStatus>;
}
```

## Business Rules

### Project Approval
- All projects must be approved by Lab Management before posting
- Projects must have clear technical requirements and realistic budgets
- Minimum project duration: 1 week
- Maximum project duration: 12 weeks
- Budget range must be within acceptable limits

### Bidding Process
- Teams can submit multiple proposals with different approaches
- Proposal deadline: 1 week after project posting
- Teams must demonstrate relevant skills and experience
- Proposals are evaluated on: technical approach, budget, timeline, team experience

### Contract Management
- Contracts are auto-generated upon proposal acceptance
- Payment milestones are based on project deliverables
- Lab Management takes 10-15% commission
- Dispute resolution through Lab Management mediation

## Quality Assurance

### Testing Strategy
- Unit tests for all API endpoints
- Integration tests for bidding workflow
- End-to-end tests for complete project lifecycle
- Performance tests for concurrent bidding scenarios

### Monitoring
- Track project posting to completion rates
- Monitor average time from posting to team selection
- Measure client satisfaction scores
- Track revenue generated through marketplace

## Success Metrics
- Number of projects posted monthly
- Average time to fill projects
- Client satisfaction rating (>4.5/5)
- Team utilization rate increase
- Revenue growth from marketplace activities
- Repeat client rate (>60%)

## Implementation Timeline
- **Week 13**: Database schema and core APIs
- **Week 14**: Frontend interfaces and bidding system
- **Integration**: Throughout development with other services
- **Testing**: Continuous testing and quality assurance
- **Deployment**: Gradual rollout with pilot projects