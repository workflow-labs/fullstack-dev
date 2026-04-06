# /fullstack-init -- Analyze PRD and Design System Architecture

**Purpose:** Analyze a PRD document, break it into system modules and features, design database schema and API contracts, and produce a complete development roadmap. This command performs NO code generation -- it only produces the plan.

**Invocation:**
```
/fullstack-init <prd-file-path>
```

---

## Pre-Flight Checks

1. If `$ARGUMENTS` is empty, search for PRD files: `*prd*`, `*PRD*`, `*requirement*`, `*spec*` in `docs/`, `doc/`, or project root. If multiple found, ask user to choose via `AskUserQuestion`. If none found, ask for path.

2. Check if `.fullstack/config.json` already exists:
   - If exists with `status: "completed"`: Ask user if they want to reinitialize (clean `.fullstack/` and start fresh) or abort.
   - If exists with `status` other than "completed": Ask user if they want to continue the existing workflow or reinitialize.
   - If not exists: Proceed normally.

3. Read the PRD file in full. If the file is very large (>500 lines), read it in sections but ensure complete coverage.

---

## Step 1: PRD Analysis

Dispatch the **prd-analyzer** agent via the `Agent` tool with `subagent_type: "fullstack-dev:prd-analyzer"`.

**Construct the agent prompt with:**
- The complete PRD content (paste it in full, do NOT reference by path)
- Instruction to extract: project name, description, target users, all functional modules, features per module, data entities, UI screens, module dependencies, cross-cutting concerns
- Instruction to classify each feature's complexity (simple/moderate/complex)
- Instruction to output structured markdown

**Read the agent's response** and save the raw analysis as `.fullstack/prd-analysis.md`.

---

## Step 2: System Architecture Design

Dispatch the **system-architect** agent via the `Agent` tool with `subagent_type: "fullstack-dev:system-architect"`.

**Construct the agent prompt with:**
- The PRD analysis from Step 1 (paste in full)
- The original PRD content (paste in full)
- Knowledge of the two boilerplates:

**Frontend boilerplate** (adamchins/react-vite-boilerplate):
- React 18 + TypeScript + Vite 5
- Ant Design + Tailwind CSS
- Zustand (state) + TanStack Query (server state)
- React Router v6 with lazy loading
- Axios HTTP client with JWT interceptor
- i18next internationalization
- Vitest + Testing Library
- Structure: `src/{apis, components, hooks, locales, pages, router, store, utils}`

**Backend boilerplate** (adamchins/zetos-fastapi):
- FastAPI + SQLAlchemy 2.0 (async) + PostgreSQL
- Dual-client API: `/api/v1/` (client) + `/api/v1/backoffice/` (admin)
- JWT auth with refresh tokens, Celery + Redis background tasks
- Alembic migrations, Docker Compose deployment
- Structure: `app/{api, core, db, models, schemas, services, common, utils, exceptions}`
- Centralized route registry pattern

**Instruction to produce:**
1. Complete database schema (all tables, columns, types, relationships, indexes)
2. API contracts per module (endpoint paths, methods, request/response schemas, auth requirements, client vs backoffice classification)
3. Module execution order (topological sort based on dependencies)
4. Feature ordering within each module (foundational features first)
5. All design decisions with rationale
6. Development roadmap

**Read the agent's response** and parse its outputs.

---

## Step 3: Clarification Gate

Review the analysis and architecture for blocking ambiguities. An ambiguity is **blocking** if it affects:
- Business rules or workflow logic
- MVP scope boundaries
- Database schema decisions (table structure, relationships)
- API contract definitions
- Authentication/authorization behavior
- Module dependencies

If blocking ambiguities exist (max 4), ask user via a single `AskUserQuestion` call. Record answers and update the design documents accordingly.

**Non-blocking ambiguities** (internal code organization, naming, file structure) should be resolved using conservative defaults documented in design-decisions.md.

---

## Step 4: Create `.fullstack/` Directory

Create the following files:

### `.fullstack/config.json`
```json
{
  "project_name": "{extracted from PRD}",
  "prd_path": "{path to PRD file}",
  "status": "analyzed",
  "created_at": "{ISO-8601 timestamp}",
  "tech_stack": {
    "frontend": {
      "framework": "React 18 + TypeScript",
      "build": "Vite 5",
      "ui": "Ant Design + Tailwind CSS",
      "state": "Zustand + TanStack Query",
      "routing": "React Router v6",
      "http": "Axios",
      "i18n": "i18next",
      "testing": "Vitest + Testing Library",
      "package_manager": "pnpm"
    },
    "backend": {
      "framework": "FastAPI",
      "orm": "SQLAlchemy 2.0 (async)",
      "database": "PostgreSQL",
      "auth": "JWT with refresh tokens",
      "tasks": "Celery + Redis",
      "migrations": "Alembic",
      "deployment": "Docker Compose"
    }
  },
  "boilerplates": {
    "frontend": "https://github.com/adamchins/react-vite-boilerplate",
    "backend": "https://github.com/adamchins/zetos-fastapi"
  },
  "total_modules": 0,
  "total_features": 0,
  "current_module": null,
  "current_feature": null,
  "conventions": {
    "claude_md_path": "./CLAUDE.md",
    "frontend_test_command": "cd frontend && pnpm test",
    "backend_test_command": "cd backend && pytest",
    "frontend_lint_command": "cd frontend && pnpm lint",
    "backend_lint_command": "cd backend && ruff check .",
    "frontend_typecheck_command": "cd frontend && pnpm tsc --noEmit",
    "backend_typecheck_command": "cd backend && mypy app/"
  }
}
```

Update `total_modules` and `total_features` with actual counts.

### `.fullstack/design-decisions.md`
Write all architectural decisions with rationale from the System Architect agent, including clarification answers.

### `.fullstack/system-architecture.md`
Write the module/feature breakdown with dependency graph.

### `.fullstack/database-schema.md`
Write the complete database schema design.

### `.fullstack/api-contracts.md`
Write all API endpoint contracts organized by module.

### `.fullstack/development-roadmap.md`
Write the ordered development plan: modules in execution order, features within each module in execution order.

### Per-module directories
For each module, create:
```
.fullstack/module-{N}-{slug}/
└── module-config.json
```

Where `module-config.json`:
```json
{
  "module_name": "{name}",
  "module_index": {N},
  "status": "pending",
  "description": "{module description}",
  "dependencies": ["{list of module slugs this depends on}"],
  "total_features": 0,
  "completed_features": 0,
  "current_feature": null,
  "features": [
    {
      "index": 1,
      "name": "{feature-name}",
      "slug": "{feature-slug}",
      "status": "pending",
      "complexity": "simple|moderate|complex",
      "completed_at": null
    }
  ]
}
```

---

## Step 5: User Approval Gate

Present the full plan to the user in a clear format:

```
Fullstack Dev: {project-name}
===================================================

Modules ({total_modules}):

  Module 1: {name} ({feature_count} features)
    Dependencies: none
    Features:
      1.1 {feature-name} [simple]
      1.2 {feature-name} [moderate]
      1.3 {feature-name} [complex]

  Module 2: {name} ({feature_count} features)
    Dependencies: Module 1
    Features:
      2.1 {feature-name} [simple]
      ...

Database: {table_count} tables
API Endpoints: {endpoint_count} endpoints ({client_count} client + {backoffice_count} backoffice)

Development Order:
  1. Module 1: {name}  →  2. Module 2: {name}  →  ...

Total Features: {total_features}
```

Ask user to approve, modify, or reject via `AskUserQuestion`:
- **Approve**: Proceed to save state
- **Modify**: Ask what to change, update accordingly
- **Reject**: Clean up `.fullstack/`, exit

---

## Step 6: Output

```
Plan saved to .fullstack/

Next step: Run /fullstack-setup to clone boilerplates and initialize the project.
```

---

## Error Handling

| Error | Response |
|-------|----------|
| PRD file not found | Ask user for correct path |
| PRD is too vague | Ask clarifying questions (max 2 rounds) |
| Circular module dependencies detected | Flag to user, ask for resolution |
| `.fullstack/` already exists | Follow pre-flight check logic |
| Agent returns incomplete analysis | Re-dispatch with more specific prompt |
