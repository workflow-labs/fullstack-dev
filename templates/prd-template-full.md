# PRD: {Project Name}

## 1. Executive Summary
{What is this product? What problem does it solve? Who is it for?}

## 2. Goals & Objectives
- {Goal 1}
- {Goal 2}
- {Goal 3}

## 3. Target Users

### 3.1 User Personas
- **{Persona Name}**: {Role, goals, pain points}
- **{Persona Name}**: {Role, goals, pain points}

### 3.2 User Stories
- As a {user type}, I want to {action} so that {benefit}.
- As a {user type}, I want to {action} so that {benefit}.

## 4. System Modules

### 4.1 {Module Name}
**Purpose:** {What this module does}
**Priority:** {Must-have | Should-have | Nice-to-have}

#### Features:
1. **{Feature Name}**
   - Description: {What this feature does}
   - User Story: As a {user}, I want to {action} so that {benefit}
   - Acceptance Criteria:
     - [ ] {Criterion 1}
     - [ ] {Criterion 2}
     - [ ] {Criterion 3}
   - UI: {Describe screens/components needed}

2. **{Feature Name}**
   - Description: {What this feature does}
   - Acceptance Criteria:
     - [ ] {Criterion 1}

### 4.2 {Module Name}
**Purpose:** {What this module does}
**Dependencies:** {Which modules must exist first}

#### Features:
1. **{Feature Name}**
   - ...

## 5. Data Model

### 5.1 Entities
| Entity | Description | Key Fields | Relationships |
|--------|-------------|------------|---------------|
| {Name} | {Purpose} | {field1, field2, ...} | belongs_to: {Entity} |
| {Name} | {Purpose} | {field1, field2, ...} | has_many: {Entity} |

### 5.2 Business Rules
- {Rule 1: e.g., "A user can only have one active subscription"}
- {Rule 2: e.g., "Deleted items are soft-deleted, not removed"}

## 6. API Requirements

### 6.1 Client API (Public)
| Endpoint | Method | Description | Auth |
|----------|--------|-------------|------|
| /api/v1/{resource} | GET | List resources | Required |
| /api/v1/{resource} | POST | Create resource | Required |

### 6.2 Backoffice API (Admin)
| Endpoint | Method | Description | Auth |
|----------|--------|-------------|------|
| /api/v1/backoffice/{resource} | GET | Admin list | Required |

## 7. UI/UX Requirements

### 7.1 Pages
| Page | Route | Description | Auth |
|------|-------|-------------|------|
| Dashboard | /dashboard | Main overview | Required |
| {Page} | /{path} | {Description} | Required |

### 7.2 Design Guidelines
- {Color scheme, typography, or branding notes}
- {Mobile responsiveness requirements}
- {Accessibility requirements}

## 8. Non-Functional Requirements

### 8.1 Performance
- {Response time targets}
- {Concurrent user expectations}

### 8.2 Security
- {Authentication method: JWT}
- {Authorization: role-based}
- {Data protection requirements}

### 8.3 Deployment
- {Hosting: Docker Compose}
- {CI/CD requirements}
- {Environment: dev, staging, production}

## 9. Constraints & Assumptions
- {Constraint 1}
- {Assumption 1}

## 10. Out of Scope
- {What is explicitly NOT included in this version}
