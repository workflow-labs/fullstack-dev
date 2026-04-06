# Fullstack-Dev: Comprehensive Workflow Instructions

> Version 0.1.0 | Last Updated: 2026-04-06

---

## Table of Contents

1. [What is fullstack-dev?](#1-what-is-fullstack-dev)
2. [When to Use fullstack-dev vs sprint-flow](#2-when-to-use-fullstack-dev-vs-sprint-flow)
3. [Architecture Overview](#3-architecture-overview)
4. [Prerequisites](#4-prerequisites)
5. [Complete Workflow Guide](#5-complete-workflow-guide)
6. [Command Reference](#6-command-reference)
7. [Agent Reference](#7-agent-reference)
8. [State Machine & Lifecycle](#8-state-machine--lifecycle)
9. [Writing an Effective PRD](#9-writing-an-effective-prd)
10. [Best Practices for Utilization](#10-best-practices-for-utilization)
11. [Troubleshooting](#11-troubleshooting)
12. [Comparison with sprint-flow (Detailed)](#12-comparison-with-sprint-flow-detailed)
13. [Appendix: File Reference](#13-appendix-file-reference)

---

## 1. What is fullstack-dev?

fullstack-dev is a Claude Code plugin that orchestrates the entire lifecycle of building a fullstack web application -- from analyzing a Product Requirements Document (PRD) to delivering a working monorepo with a React frontend and FastAPI backend.

### Core Philosophy

- **Plan before you code** -- the system designs the complete architecture (database schema, API contracts, module dependencies) before writing a single line of code
- **Backend first, frontend second** -- for each feature, the backend is implemented and tested before the frontend begins, ensuring the frontend always integrates against real, working endpoints
- **One feature at a time** -- the workflow pauses after each feature for manual testing, preventing error compounding and ensuring quality at every step
- **Boilerplate-powered** -- projects start from production-tested boilerplates, not empty directories, giving you working auth, routing, state management, and Docker configuration from minute one

### What It Produces

Given a PRD document, fullstack-dev produces:

```
my-project/
├── frontend/          # React 18 + TypeScript + Vite 5
│   ├── src/
│   │   ├── apis/      # Axios API clients per feature
│   │   ├── components/# Ant Design + Tailwind components
│   │   ├── hooks/     # TanStack Query hooks
│   │   ├── pages/     # Route-based pages
│   │   ├── store/     # Zustand state stores
│   │   └── router/    # React Router v6 config
│   └── ...
├── backend/           # FastAPI + SQLAlchemy 2.0 (async)
│   ├── app/
│   │   ├── api/       # Versioned endpoints (client + backoffice)
│   │   ├── models/    # SQLAlchemy ORM models
│   │   ├── schemas/   # Pydantic v2 validation
│   │   ├── services/  # Business logic layer
│   │   └── ...
│   └── ...
├── .fullstack/        # Workflow state and documentation
├── docker-compose.yml # Combined orchestration
├── CLAUDE.md          # Project conventions
└── README.md
```

---

## 2. When to Use fullstack-dev vs sprint-flow

### Use fullstack-dev When

- Building a **web application with both frontend and backend**
- You want a **monorepo** with React + FastAPI
- You have a **PRD** (or can write one) describing modules and features
- You want **structured, incremental delivery** with testing gates
- You want the system to **design the database schema and API contracts** upfront
- You value **cross-stack integration verification** (frontend calls match backend endpoints)

### Use sprint-flow When

- Building a **single-stack project** (backend only, CLI tool, data pipeline, mobile app)
- The tech stack is **not React + FastAPI** (sprint-flow is stack-agnostic)
- The project is **small** (fewer than 5 features) and doesn't justify the overhead
- You want **continuous execution** without pausing after each unit of work
- You're doing a **proof of concept** or prototype where speed matters more than structure

### Use feature-flow When

- Adding features to an **existing codebase** (1-to-N development)
- The project is already initialized and you need **incremental feature delivery**
- You need **codebase analysis** of an existing project before implementing

### Decision Matrix

| Scenario | Recommended |
|----------|-------------|
| New fullstack web app from PRD | **fullstack-dev** |
| New backend-only API from PRD | sprint-flow |
| New CLI tool from PRD | sprint-flow |
| Add feature to existing React+FastAPI app | feature-flow |
| Add feature to existing single-stack app | feature-flow |
| Quick prototype, any stack | sprint-flow |
| Complex system, 10+ modules | **fullstack-dev** |

---

## 3. Architecture Overview

### Plugin Structure

```
fullstack-dev/
├── .claude-plugin/marketplace.json      # Plugin marketplace registry
├── plugins/fullstack-dev/
│   ├── .claude-plugin/plugin.json       # Plugin metadata
│   ├── commands/                        # 6 user-facing slash commands
│   │   ├── fullstack-init.md
│   │   ├── fullstack-setup.md
│   │   ├── fullstack-dev.md
│   │   ├── fullstack-test.md
│   │   ├── fullstack-status.md
│   │   └── fullstack-done.md
│   └── agents/                          # 6 specialized sub-agents
│       ├── prd-analyzer.md
│       ├── system-architect.md
│       ├── backend-developer.md
│       ├── frontend-developer.md
│       ├── integration-verifier.md
│       └── quality-engineer.md
└── templates/                           # PRD templates
```

### Tech Stack (Fixed)

| Layer | Technology |
|-------|-----------|
| Frontend framework | React 18 + TypeScript (strict) |
| Frontend build | Vite 5 |
| UI library | Ant Design 5 |
| CSS framework | Tailwind CSS 3 |
| Client state | Zustand with Immer |
| Server state | TanStack Query (React Query) |
| HTTP client | Axios with JWT interceptor |
| Routing | React Router v6 (lazy loading) |
| i18n | i18next |
| Frontend testing | Vitest + Testing Library |
| Backend framework | FastAPI |
| ORM | SQLAlchemy 2.0 (async) |
| Database | PostgreSQL |
| Auth | JWT with refresh tokens |
| Background tasks | Celery + Redis |
| Migrations | Alembic |
| Backend testing | pytest |
| Containerization | Docker Compose |

### Boilerplate Sources

| Boilerplate | Repository | Purpose |
|------------|-----------|---------|
| Frontend | [adamchins/react-vite-boilerplate](https://github.com/adamchins/react-vite-boilerplate) | Cloned to `/frontend` |
| Backend | [adamchins/zetos-fastapi](https://github.com/adamchins/zetos-fastapi) | Cloned to `/backend` |

Both boilerplates come with working authentication, Docker configuration, linting, testing infrastructure, and established patterns that the agents follow.

---

## 4. Prerequisites

### Environment Requirements

- **Claude Code** CLI installed and authenticated
- **Git** installed
- **Node.js** >= 18.0.0
- **pnpm** >= 8.0.0 (frontend package manager)
- **Python** >= 3.10 (backend runtime)
- **Docker** and **Docker Compose** (for PostgreSQL, Redis)
- **GitHub CLI** (`gh`) for repo operations

### Plugin Installation

Add to `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "fullstack-dev-marketplace": {
      "source": { "source": "github", "repo": "workflow-labs/fullstack-dev" }
    }
  },
  "enabledPlugins": {
    "fullstack-dev@fullstack-dev-marketplace": true
  }
}
```

### Before Starting a Project

1. **Write a PRD** -- use the templates in `templates/` as a starting point
2. **Create a project directory** -- `mkdir my-project && cd my-project`
3. **Ensure Docker is running** -- PostgreSQL and Redis will be needed

---

## 5. Complete Workflow Guide

### Phase 1: Planning (`/fullstack-init`)

```
/fullstack-init docs/prd.md
```

**What happens:**
1. The PRD Analyzer agent reads your PRD and extracts modules, features, data entities, and dependencies
2. The System Architect agent designs the database schema, API contracts, and development roadmap
3. You're asked to clarify any ambiguities (max 4 questions)
4. The complete plan is presented for your approval

**What you get:**
- `.fullstack/system-architecture.md` -- module/feature breakdown
- `.fullstack/database-schema.md` -- all tables, columns, relationships
- `.fullstack/api-contracts.md` -- every endpoint specification
- `.fullstack/development-roadmap.md` -- ordered execution plan
- `.fullstack/design-decisions.md` -- architectural rationale

**Your responsibility:**
- Review the plan carefully
- Verify modules and features are complete (nothing missing from the PRD)
- Check the dependency ordering makes sense
- Approve, request modifications, or reject

**Tips:**
- The more detailed your PRD, the better the plan. Vague PRDs produce vague architectures.
- If you disagree with the module breakdown, say so now. It's much cheaper to reorganize modules at this stage than after code is written.
- Pay special attention to the database schema -- adding a missing column later requires a migration, but getting it right now is free.

---

### Phase 2: Initialization (`/fullstack-setup`)

```
/fullstack-setup
```

**What happens:**
1. Clones the frontend boilerplate to `/frontend`
2. Clones the backend boilerplate to `/backend`
3. Customizes configs (project name, env vars, API proxy, CORS, database name)
4. Sets up the monorepo root (docker-compose, CLAUDE.md, scripts)
5. Installs dependencies
6. Verifies both services start and respond
7. Creates the initial git commit

**Your responsibility:**
- Verify both services start correctly
- If using Docker, ensure containers are healthy
- If using local development, ensure PostgreSQL and Redis are running

**Tips:**
- If the health check fails, check: is Docker running? Is port 8001 (backend) or 3000 (frontend) already in use?
- Review the generated `CLAUDE.md` -- it becomes the convention guide for all future development
- This is a good time to set up your IDE workspace with both `/frontend` and `/backend`

---

### Phase 3: Feature Development (Repeat)

This is the core loop. It repeats for every feature across all modules.

#### Step 3a: Develop (`/fullstack-dev`)

```
/fullstack-dev              # Auto-picks next feature
/fullstack-dev 2/3          # Specific: module 2, feature 3
```

**What happens (per feature):**

```
Analyzing        Extract feature requirements, create backend + frontend plans
     |
Backend Dev      Models -> Schemas -> Services -> Endpoints -> Migration -> Tests -> Commit
     |
Frontend Dev     API clients -> Store -> Hooks -> Components -> Pages -> Router -> Tests -> Commit
     |
Integration      Verify frontend calls match backend endpoints (field names, shapes, auth)
     |
Testing          Run all tests, lint, type checks. Self-heal failures (2 attempts).
     |
PAUSE            Present results. Wait for user to test manually.
```

**Your responsibility:**
- Test the feature manually after the PAUSE
- Check every acceptance criterion listed in the output
- Test edge cases the automated tests might miss (UI interactions, error states, auth flows)

#### Step 3b: Test (`/fullstack-test`)

```
/fullstack-test             # Run all tests
/fullstack-test backend     # Backend only
/fullstack-test frontend    # Frontend only
```

Use this anytime to re-run tests and see a formatted summary.

#### Step 3c: Complete or Fix (`/fullstack-done`)

```
/fullstack-done             # Approve: generate reports, advance to next feature
/fullstack-done fix         # Report issues: enter fix cycle
```

**If everything works:** Run `/fullstack-done` to approve. This generates a completion report, creates a handoff document for the next feature, and advances the state.

**If you found issues:** Run `/fullstack-done fix` to describe the problem. The appropriate agent (backend/frontend/integration) will fix it, tests will re-run, and you'll be asked to test again. You can run `/fullstack-done fix` as many times as needed -- the feature won't advance until you explicitly approve with `/fullstack-done`.

#### Step 3d: Check Progress (`/fullstack-status`)

```
/fullstack-status           # Overview of all modules
/fullstack-status 2         # Detailed view of module 2
```

Shows a progress dashboard with completion percentages, current feature phase, and next steps.

---

### Phase 4: Completion

When the last feature of the last module is completed via `/fullstack-done`, the project status changes to `"completed"` and a project completion report is generated.

---

## 6. Command Reference

### `/fullstack-init <prd-path>`

| Aspect | Detail |
|--------|--------|
| **Purpose** | Analyze PRD, design system architecture, produce development roadmap |
| **Code generation** | None -- planning only |
| **Outputs** | `.fullstack/` directory with config, schema, contracts, roadmap |
| **Next command** | `/fullstack-setup` |
| **Can re-run?** | Yes -- will ask to reinitialize or continue existing |
| **Agents used** | PRD Analyzer, System Architect |

### `/fullstack-setup`

| Aspect | Detail |
|--------|--------|
| **Purpose** | Clone boilerplates, customize, initialize monorepo, verify services |
| **Prerequisites** | `/fullstack-init` completed (status: `"analyzed"`) |
| **Outputs** | `/frontend`, `/backend`, root configs, initial git commit |
| **Next command** | `/fullstack-dev` |
| **Can re-run?** | Will warn if already set up |
| **Agents used** | None (direct execution) |

### `/fullstack-dev [module/feature]`

| Aspect | Detail |
|--------|--------|
| **Purpose** | Develop ONE feature: backend -> frontend -> integration -> tests -> PAUSE |
| **Prerequisites** | `/fullstack-setup` completed (status: `"setup"` or `"developing"`) |
| **Outputs** | Backend + frontend code, tests, commits, implementation logs |
| **Next command** | `/fullstack-done` (after manual testing) |
| **Can re-run?** | Resumes interrupted feature or picks next pending |
| **Agents used** | Backend Developer, Frontend Developer, Integration Verifier, Quality Engineer |

### `/fullstack-test [backend|frontend|all]`

| Aspect | Detail |
|--------|--------|
| **Purpose** | Run test suites, linters, type checks for current feature |
| **Prerequisites** | A feature must be in progress |
| **Outputs** | Formatted test results summary, updated `test-results.md` |
| **Next command** | `/fullstack-done` or `/fullstack-done fix` |
| **Can re-run?** | Yes, anytime |
| **Agents used** | None (direct execution) |

### `/fullstack-status [module-index]`

| Aspect | Detail |
|--------|--------|
| **Purpose** | Display progress dashboard |
| **Prerequisites** | `/fullstack-init` completed |
| **Outputs** | Formatted progress with module/feature status |
| **Next command** | Contextual (shown in output) |
| **Can re-run?** | Yes, anytime -- read-only |
| **Agents used** | None |

### `/fullstack-done [fix]`

| Aspect | Detail |
|--------|--------|
| **Purpose** | Complete current feature OR enter fix cycle |
| **Prerequisites** | Feature in `"testing"` phase (PAUSE state) |
| **Without arg** | Generates completion report + handoff, advances to next feature |
| **With `fix`** | Collects issues, dispatches fix agent, re-tests, pauses again |
| **Agents used** | Backend Developer or Frontend Developer (for fixes) |

---

## 7. Agent Reference

### PRD Analyzer

**When:** During `/fullstack-init` (Step 1)

**What it does:**
- Reads the complete PRD
- Identifies system modules (bounded functional areas)
- Breaks modules into features (smallest testable units)
- Maps dependencies between modules and features
- Classifies feature complexity (simple/moderate/complex)
- Extracts data entities and their relationships

**Quality depends on:** PRD completeness and specificity. Vague PRDs produce vague breakdowns.

---

### System Architect

**When:** During `/fullstack-init` (Step 2)

**What it does:**
- Designs complete database schema (all tables, columns, types, relationships, indexes)
- Defines API contracts (endpoints, methods, request/response shapes, auth requirements)
- Classifies endpoints as client API or backoffice API (following zetos-fastapi dual-API pattern)
- Determines module execution order (topological sort)
- Orders features within modules (foundational first)
- Documents all design decisions with rationale

**Quality depends on:** PRD analysis quality and clarity of data entities and business rules.

---

### Backend Developer

**When:** During `/fullstack-dev` (backend phase) and `/fullstack-done fix`

**What it does:**
1. Creates SQLAlchemy models following existing patterns
2. Creates Pydantic schemas (Create/Update/Response variants)
3. Creates service layer with async methods and transaction management
4. Creates API endpoints with proper auth, validation, and error handling
5. Registers routes in the centralized route registry
6. Creates and applies Alembic migrations
7. Writes pytest tests (happy path, validation, auth, edge cases)

**Key constraint:** Every pattern decision must cite an existing file in the backend codebase as a precedent.

---

### Frontend Developer

**When:** During `/fullstack-dev` (frontend phase) and `/fullstack-done fix`

**What it does:**
1. Creates Axios API client functions matching backend endpoints exactly
2. Creates/updates Zustand stores for client-side state
3. Creates TanStack Query hooks for server state (queries + mutations)
4. Creates Ant Design + Tailwind UI components
5. Creates pages with proper loading, error, and empty states
6. Updates React Router with lazy-loaded routes
7. Adds i18n translation keys
8. Writes Vitest + Testing Library tests

**Key constraint:** Endpoint URLs, field names, and response shapes must match the backend implementation exactly. The backend is the source of truth.

---

### Integration Verifier

**When:** During `/fullstack-dev` (integration phase)

**What it does:**
- Reads actual backend source files (endpoints, Pydantic schemas)
- Reads actual frontend source files (API clients, hooks)
- Verifies every frontend API call targets a real backend endpoint
- Checks HTTP methods, request payloads, response handling, auth headers, query parameters
- Fixes mismatches (frontend adapts to backend, unless backend has a clear bug)

**Why it matters:** Without this, you discover frontend-backend mismatches at runtime. With it, they're caught before you even test manually.

---

### Quality Engineer

**When:** During `/fullstack-dev` (testing phase) and `/fullstack-test`

**What it does:**
- Runs backend tests (`pytest`)
- Runs frontend tests (`vitest`)
- Runs linters (`ruff` for backend, `eslint` for frontend)
- Runs type checks (`mypy`, `tsc`)
- Attempts self-healing for failures (2 attempts max)
- Verifies acceptance criteria have corresponding tests
- Writes additional tests if coverage gaps found

---

## 8. State Machine & Lifecycle

### Three-Level State Model

```
PROJECT
  uninitialized ──> analyzed ──> setup ──> developing ──> completed
                    (init)     (setup)    (dev loop)

MODULE (per module)
  pending ──> in_progress ──> completed
              (first feature starts)  (last feature completes)

FEATURE (per feature within a module)
  pending ──> analyzing ──> backend_dev ──> frontend_dev ──> integrating ──> testing ──> completed
                                                                              |
                                                                         PAUSE HERE
                                                                     (user tests manually)
```

### State Files

| File | Level | Purpose |
|------|-------|---------|
| `.fullstack/config.json` | Project | Overall status, current position, tech stack, conventions |
| `.fullstack/module-{N}-{slug}/module-config.json` | Module | Module status, feature list with statuses |
| `.fullstack/module-{N}-{slug}/feature-{M}-{slug}/feature-config.json` | Feature | Phase tracking, timestamps, user feedback |

### State Transitions

| Trigger | State Change |
|---------|-------------|
| `/fullstack-init` completes | Project: `uninitialized` -> `analyzed` |
| `/fullstack-setup` completes | Project: `analyzed` -> `setup` |
| First `/fullstack-dev` starts | Project: `setup` -> `developing` |
| `/fullstack-dev` starts a module's first feature | Module: `pending` -> `in_progress` |
| `/fullstack-dev` starts a feature | Feature: `pending` -> `analyzing` -> ... -> `testing` |
| `/fullstack-done` (approve) | Feature: `testing` -> `completed` |
| Last feature in module completed | Module: `in_progress` -> `completed` |
| Last feature in last module completed | Project: `developing` -> `completed` |

### Interruption & Recovery

If the workflow is interrupted mid-feature (e.g., session ends, network drops):

- `/fullstack-dev` reads `feature-config.json` phase status
- Resumes from the last incomplete phase
- Backend code already written? Skips to frontend phase.
- Frontend done but integration not verified? Skips to integration phase.

No work is lost. The file-based state is durable across sessions.

---

## 9. Writing an Effective PRD

The quality of the fullstack-dev output is directly proportional to the quality of your PRD. Here's how to write one that produces excellent results.

### Must-Have Sections

1. **Project overview** -- what it is, who it's for, what problem it solves
2. **User types** -- distinct personas (end user, admin, etc.)
3. **Modules with features** -- functional areas broken into specific capabilities
4. **Data entities** -- what objects exist, their fields, and relationships
5. **Acceptance criteria** -- specific, testable conditions per feature

### Good vs Bad PRD Examples

**Bad:**
> "The app should have user management."

**Good:**
> Module: User Management
> - Feature: User Registration -- users can sign up with email and password. Fields: name, email, password. Sends verification email. Acceptance: user receives email within 60 seconds, clicking link activates account.
> - Feature: User Profile -- users can view and edit their name, avatar, and bio. Avatar upload max 2MB, JPEG/PNG only. Acceptance: profile updates are saved immediately, avatar appears in header.

### PRD Checklist

Before running `/fullstack-init`, verify your PRD includes:

- [ ] Clear project name and description
- [ ] Target user personas
- [ ] Every functional module with description
- [ ] Every feature with specific requirements (not vague)
- [ ] Data entities with key fields and relationships
- [ ] Which features require authentication
- [ ] Which features are admin-only (backoffice API)
- [ ] Acceptance criteria that are testable (not subjective)
- [ ] Out-of-scope items (what is NOT included)
- [ ] Any business rules or constraints

### PRD Templates

Use the provided templates as starting points:

- `templates/prd-template-simple.md` -- for small projects (3-5 modules)
- `templates/prd-template-full.md` -- for complex projects (5+ modules, detailed requirements)

---

## 10. Best Practices for Utilization

### Before You Start

1. **Invest time in the PRD** -- 2 hours writing a detailed PRD saves 10 hours of rework. Include acceptance criteria for every feature.

2. **Review the boilerplates** -- run the frontend and backend boilerplates standalone first. Understand their patterns, file structure, and conventions. The agents follow these patterns, so knowing them helps you review agent output.

3. **Set up Docker early** -- the backend needs PostgreSQL and Redis. Get `docker-compose up -d` working before you start.

### During Planning (`/fullstack-init`)

4. **Scrutinize the database schema** -- this is the hardest thing to change later. Look for:
   - Missing relationships (many-to-many needs a join table)
   - Missing fields (status fields, soft-delete flags, foreign keys)
   - Wrong types (string vs enum, integer vs UUID)

5. **Validate the API contracts** -- check that:
   - Every UI action has a corresponding endpoint
   - Auth requirements are correct (public vs authenticated vs admin-only)
   - Response shapes include all fields the frontend needs

6. **Challenge the module order** -- the dependency graph should make logical sense. Auth before User Management before Dashboard before Billing is typical. If the architect suggests a different order, understand why.

### During Development (`/fullstack-dev` loop)

7. **Test manually after every feature** -- don't skip this. The PAUSE exists for a reason. Automated tests catch code-level bugs; manual testing catches UX issues, incorrect business logic, and integration problems.

8. **Use `/fullstack-done fix` generously** -- if anything is wrong, report it. The fix cycle is designed for iteration. It's better to fix 3 small issues now than discover them compounded in the next feature.

9. **Test the full flow, not just the new feature** -- after feature 1.3 is built, re-test features 1.1 and 1.2 to check for regressions. Especially test after database migrations.

10. **Read the implementation logs** -- after each `/fullstack-dev`, skim the implementation log (`.fullstack/module-X/feature-Y/implementation-log.md`). It shows exactly what was built, which patterns were followed, and any deviations from the plan.

11. **Read the integration report** -- the integration verification catches mismatches, but reviewing the report helps you understand how the frontend and backend connect.

### During Fixes

12. **Be specific when reporting issues** -- instead of "the form doesn't work," say "the create task form submits but returns a 422 error. The backend expects `due_date` as ISO string but the frontend sends a Date object."

13. **One issue per fix cycle** -- if you found 3 issues, report the most critical one first. After it's fixed and re-tested, report the next one. This gives the agent focused context.

### General

14. **Don't edit `.fullstack/` files manually** -- the state directory is managed by the workflow. Manual edits can corrupt state and cause unpredictable behavior.

15. **Commit frequently** -- the workflow commits after each development phase, but you can make additional commits between commands for your own changes.

16. **Use `/fullstack-status` to stay oriented** -- especially in large projects with many modules. It shows exactly where you are and what's next.

17. **Save the design documents** -- the files in `.fullstack/` (especially `design-decisions.md`, `database-schema.md`, and `api-contracts.md`) are valuable project documentation even after development is complete.

---

## 11. Troubleshooting

### Common Issues

#### `/fullstack-init` produces a poor plan

**Cause:** PRD is too vague or missing key sections.
**Fix:** Improve your PRD. Add specific acceptance criteria, data entity definitions, and explicit feature descriptions. Re-run `/fullstack-init`.

#### `/fullstack-setup` fails to clone boilerplates

**Cause:** Network issue or repository access.
**Fix:** Verify you can access the boilerplate repos (`gh repo view adamchins/react-vite-boilerplate`). If private, ensure GitHub CLI is authenticated.

#### Backend health check fails

**Cause:** PostgreSQL or Redis not running, or port conflict.
**Fix:**
```bash
docker-compose up -d postgres redis    # Start services
lsof -i :8001                          # Check port conflict
```

#### Frontend dev server fails

**Cause:** Node version too old, pnpm not installed, or port conflict.
**Fix:**
```bash
node --version    # Must be >= 18
pnpm --version    # Must be >= 8
lsof -i :3000     # Check port conflict
```

#### Alembic migration fails

**Cause:** Database state doesn't match expected schema (previous migration failed or was manually altered).
**Fix:**
```bash
cd backend
alembic history         # Check migration history
alembic current         # Check current state
alembic upgrade head    # Retry
```
If corrupted, consider: `alembic stamp head` to reset (destructive -- only in development).

#### Tests fail after agent implementation

**Cause:** Agent produced code with a bug, or test expectations are wrong.
**Fix:** The Quality Engineer agent attempts 2 self-healing rounds. If still failing, review the error in `test-results.md` and:
- If it's a real bug: `/fullstack-done fix` with the error description
- If it's a test issue: manually fix the test file

#### Integration verification finds mismatches

**Cause:** Backend and frontend were developed with slightly different assumptions.
**Fix:** The Integration Verifier automatically fixes most mismatches. If it can't, the issue is reported in `integration-report.md` with specific file paths and line numbers.

#### Context too large for agent

**Cause:** Feature is too complex, too many files to include in the agent prompt.
**Fix:** This is rare. If it happens, the orchestrator should split the feature into sub-tasks. You may need to manually break the feature into smaller features in the module config.

### Recovery Commands

| Situation | Recovery |
|-----------|----------|
| Interrupted mid-feature | Run `/fullstack-dev` again -- resumes from last phase |
| Want to skip a feature | Edit `module-config.json`, set feature status to `"completed"` |
| Want to redo a feature | Edit `feature-config.json`, reset status to `"pending"` |
| Want to restart entirely | Delete `.fullstack/`, `/frontend`, `/backend`, run `/fullstack-init` |

---

## 12. Comparison with sprint-flow (Detailed)

### Architectural Differences

| Dimension | sprint-flow | fullstack-dev |
|-----------|-------------|---------------|
| **Scope** | Single codebase, any stack | Monorepo, React + FastAPI |
| **State levels** | 2 (project, sprint) | 3 (project, module, feature) |
| **Planning unit** | Sprint (arbitrary chunk) | Module + Feature (domain-aligned) |
| **Execution unit** | Sprint (multiple tasks) | Feature (backend + frontend + integration) |
| **Stack awareness** | None (generic executor) | Full (6 specialized agents) |
| **API contracts** | None (emergent) | Designed upfront, verified per feature |
| **DB schema** | Evolves per sprint | Designed holistically upfront |
| **Integration check** | None | Dedicated verification phase per feature |
| **Boilerplate** | None (starts from scratch) | Clones production-tested templates |

### Workflow Differences

| Dimension | sprint-flow | fullstack-dev |
|-----------|-------------|---------------|
| **Init output** | Sprint plan + handoffs | Schema + API contracts + roadmap |
| **Setup phase** | None (bundled with Sprint 0) | Separate command with verification |
| **Execution model** | Auto-loops all sprints | Pauses after each feature |
| **User testing** | After all sprints | After each feature |
| **Fix mechanism** | None (manual intervention) | `/fullstack-done fix` with targeted agents |
| **Agent model** | 1 generic executor | 6 specialists (PRD, architect, backend, frontend, integration, quality) |
| **Clarification** | Pre-sprint gate | Pre-init gate + per-feature context |

### Quality Improvements

| Area | sprint-flow gap | fullstack-dev solution |
|------|----------------|----------------------|
| **Frontend-backend alignment** | No verification. Mismatches found at runtime. | Integration Verifier checks every API call against every endpoint before manual testing. |
| **Error compounding** | All sprints run automatically. Bug in sprint 2 silently breaks sprints 3-8. | Hard pause after each feature. User verifies before advancing. |
| **Pattern consistency** | Generic executor may drift from patterns across sprints. | Specialized agents read and cite existing pattern files as precedents. |
| **Feedback loop** | No structured way to report and fix issues. | `/fullstack-done fix` cycle with targeted agents, tracked in `user_feedback[]`. |
| **Planning depth** | Sprint-level task lists. No schema or contract design. | Complete DB schema, API contracts, and module dependency graph before code. |
| **Context handoff** | Handoff documents between sprints (can lose context). | Handoff per feature + cumulative implementation logs. Each agent gets full context. |

### When sprint-flow Wins

| Scenario | Why sprint-flow is better |
|----------|--------------------------|
| **Non-web projects** (CLI, data pipeline, mobile) | fullstack-dev is locked to React + FastAPI |
| **Single-stack projects** | fullstack-dev's frontend/backend split is unnecessary overhead |
| **Rapid prototyping** | sprint-flow's continuous execution is faster when quality gates aren't needed |
| **Unknown tech stack** | sprint-flow adapts to any stack; fullstack-dev requires specific boilerplates |
| **Small projects** (< 5 features) | fullstack-dev's 3-level state machine is overkill |

### When fullstack-dev Wins

| Scenario | Why fullstack-dev is better |
|----------|----------------------------|
| **Fullstack web apps** | Purpose-built for the React + FastAPI monorepo pattern |
| **Complex systems** (10+ features) | Module/feature hierarchy keeps work organized |
| **Team projects** | API contracts serve as shared documentation between frontend and backend |
| **Production apps** | Testing gates, integration verification, and boilerplate patterns produce production-quality code |
| **Projects requiring compliance** | Every decision documented, every feature has completion reports |

---

## 13. Appendix: File Reference

### `.fullstack/` Directory (Complete)

```
.fullstack/
├── config.json                          # Project-level state and configuration
├── prd-analysis.md                      # Raw PRD analysis output
├── design-decisions.md                  # All architectural decisions with rationale
├── system-architecture.md               # Module/feature breakdown with dependencies
├── database-schema.md                   # Complete database schema
├── api-contracts.md                     # All endpoint specifications
├── development-roadmap.md               # Ordered execution plan
│
├── module-1-{slug}/
│   ├── module-config.json               # Module state, feature list
│   ├── module-design.md                 # Module-specific design notes (optional)
│   │
│   ├── feature-1-{slug}/
│   │   ├── feature-config.json          # Feature state with phase tracking
│   │   ├── feature-spec.md              # Extracted requirements for this feature
│   │   ├── backend-plan.md              # Ordered backend task list
│   │   ├── frontend-plan.md             # Ordered frontend task list
│   │   ├── implementation-log.md        # What was actually built (both stacks)
│   │   ├── integration-report.md        # Frontend-backend contract verification
│   │   ├── test-results.md              # Test/lint/typecheck results
│   │   ├── completion-report.md         # Final summary (generated on /fullstack-done)
│   │   └── handoff.md                   # Context for next feature's agents
│   │
│   ├── feature-2-{slug}/
│   │   └── ... (same structure)
│   │
│   └── module-completion-report.md      # Generated when all features complete
│
├── module-2-{slug}/
│   └── ... (same structure)
│
└── project-completion-report.md         # Generated when all modules complete
```

### Key JSON Schemas

#### `config.json` (Project)
```json
{
  "project_name": "string",
  "prd_path": "string",
  "status": "uninitialized | analyzed | setup | developing | completed",
  "created_at": "ISO-8601",
  "total_modules": "number",
  "total_features": "number",
  "current_module": "number | null",
  "current_feature": "string | null",
  "tech_stack": { "frontend": {...}, "backend": {...} },
  "boilerplates": { "frontend": "URL", "backend": "URL" },
  "conventions": { "test commands, lint commands, etc." }
}
```

#### `module-config.json`
```json
{
  "module_name": "string",
  "module_index": "number",
  "status": "pending | in_progress | completed",
  "description": "string",
  "dependencies": ["module slugs"],
  "total_features": "number",
  "completed_features": "number",
  "current_feature": "number | null",
  "features": [
    {
      "index": "number",
      "name": "string",
      "slug": "string",
      "status": "pending | analyzing | backend_dev | frontend_dev | integrating | testing | completed",
      "complexity": "simple | moderate | complex",
      "completed_at": "ISO-8601 | null"
    }
  ]
}
```

#### `feature-config.json`
```json
{
  "feature_name": "string",
  "module_name": "string",
  "module_index": "number",
  "feature_index": "number",
  "status": "string (same as feature status in module-config)",
  "created_at": "ISO-8601",
  "completed_at": "ISO-8601 | null",
  "phases": {
    "analyzing": { "status": "pending | in_progress | completed", "timestamp": "ISO-8601 | null" },
    "backend_dev": { "status": "...", "timestamp": "..." },
    "frontend_dev": { "status": "...", "timestamp": "..." },
    "integrating": { "status": "...", "timestamp": "..." },
    "testing": { "status": "...", "timestamp": "..." }
  },
  "user_feedback": [
    {
      "timestamp": "ISO-8601",
      "issue": "string",
      "type": "backend | frontend | integration | other",
      "resolution": "string",
      "files_changed": ["string"]
    }
  ]
}
```

---

## Quick Reference Card

```
COMMANDS                          PURPOSE
/fullstack-init <prd>             Analyze PRD, design system (no code)
/fullstack-setup                  Clone boilerplates, initialize project
/fullstack-dev [M/F]              Develop next feature (or specific M/F)
/fullstack-test [scope]           Run tests (backend|frontend|all)
/fullstack-status [module]        Show progress dashboard
/fullstack-done                   Approve feature, advance to next
/fullstack-done fix               Report issue, enter fix cycle

STATE FLOW
Project:  uninitialized -> analyzed -> setup -> developing -> completed
Module:   pending -> in_progress -> completed
Feature:  pending -> analyzing -> backend_dev -> frontend_dev ->
          integrating -> testing -> [PAUSE] -> completed

DEVELOPMENT ORDER (per feature)
Backend:  Models -> Schemas -> Services -> Endpoints -> Migration -> Tests
Frontend: APIs -> Store -> Hooks -> Components -> Pages -> Router -> Tests
Then:     Integration verification -> Quality checks -> PAUSE
```
