# fullstack-dev

A Claude Code plugin that orchestrates fullstack development from PRD to working system. Monorepo layout with React+Vite frontend and FastAPI backend.

## What It Does

1. **Analyzes your PRD** -- breaks it into modules and features with dependency ordering
2. **Designs the system** -- database schema, API contracts, development roadmap
3. **Initializes the project** -- clones boilerplates, customizes configs, sets up monorepo
4. **Develops feature-by-feature** -- backend first, then frontend, with integration verification
5. **Pauses for testing** -- after each feature, you test before moving to the next

## Tech Stack

| Layer | Stack |
|-------|-------|
| Frontend | React 18 + TypeScript + Vite 5, Ant Design + Tailwind CSS, Zustand + TanStack Query |
| Backend | FastAPI + SQLAlchemy 2.0 (async) + PostgreSQL, JWT auth, Celery + Redis |
| Layout | Monorepo: `/frontend` + `/backend` |

Based on:
- [react-vite-boilerplate](https://github.com/adamchins/react-vite-boilerplate)
- [zetos-fastapi](https://github.com/adamchins/zetos-fastapi)

## Installation

### Claude Code

#### Step 1: Add marketplace

```bash
/plugin marketplace add workflow-labs/fullstack-dev
```

#### Step 2: Install plugin

```bash
/plugin install fullstack-dev@fullstack-dev-marketplace
```

## Commands

| Command | Purpose |
|---------|---------|
| `/fullstack-init <prd-path>` | Analyze PRD, design system, produce roadmap |
| `/fullstack-setup` | Clone boilerplates, initialize monorepo |
| `/fullstack-dev [module/feature]` | Develop next feature (backend-first) |
| `/fullstack-test [backend\|frontend\|all]` | Run tests for current feature |
| `/fullstack-status [module-index]` | Show development progress |
| `/fullstack-done [fix]` | Complete feature or fix issues |

## Workflow

```
Write PRD
    |
    v
/fullstack-init docs/prd.md     -- Analyze & plan (no code)
    |
    v
/fullstack-setup                 -- Clone boilerplates, initialize project
    |
    v
/fullstack-dev                   -- Develop feature 1.1 (backend + frontend)
    |                               PAUSE -- you test manually
    v
/fullstack-done                  -- Approve & advance
    |                               (or /fullstack-done fix -- report issues)
    v
/fullstack-dev                   -- Develop feature 1.2
    |
    ... repeat for all features ...
    |
    v
PROJECT COMPLETE
```

## State Machine

```
Project:  uninitialized -> analyzed -> setup -> developing -> completed
Module:   pending -> in_progress -> completed
Feature:  pending -> analyzing -> backend_dev -> frontend_dev -> integrating -> testing -> completed
```

## State Directory

All workflow state is stored in `.fullstack/`:

```
.fullstack/
├── config.json                  -- Project state
├── design-decisions.md          -- Architectural decisions
├── system-architecture.md       -- Module/feature breakdown
├── database-schema.md           -- DB schema
├── api-contracts.md             -- API specifications
├── development-roadmap.md       -- Execution order
└── module-{N}-{name}/
    ├── module-config.json
    └── feature-{M}-{name}/
        ├── feature-config.json
        ├── feature-spec.md
        ├── backend-plan.md
        ├── frontend-plan.md
        ├── implementation-log.md
        ├── integration-report.md
        ├── test-results.md
        ├── completion-report.md
        └── handoff.md
```

## Agents

| Agent | Role |
|-------|------|
| PRD Analyzer | Extract modules, features, and dependencies from PRD |
| System Architect | Design DB schema, API contracts, roadmap |
| Backend Developer | Implement backend code following FastAPI patterns |
| Frontend Developer | Implement frontend code following React patterns |
| Integration Verifier | Verify frontend-backend contract alignment |
| Quality Engineer | Run tests, lint, type checks, fix failures |

## Key Design Decisions

- **Backend is source of truth** for API contracts
- **One feature at a time** -- strictly sequential
- **Pause after each feature** -- prevents compounding errors
- **Init separate from Setup** -- plan review before code generation
- **Self-healing** -- 2 fix attempts for test failures, then escalate

## PRD Templates

See `templates/` for PRD templates:
- `prd-template-simple.md` -- minimal PRD for small projects
- `prd-template-full.md` -- comprehensive PRD for complex projects

## License

MIT
