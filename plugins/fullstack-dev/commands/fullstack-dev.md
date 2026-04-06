# /fullstack-dev -- Develop Next Feature

**Purpose:** Develop exactly ONE feature with a backend-first approach (models -> schemas -> services -> endpoints -> tests, then frontend components -> pages -> API integration -> tests), verify integration contracts, run all tests, then PAUSE for user testing.

**Invocation:**
```
/fullstack-dev [module-index/feature-index]
```

If no argument is provided, automatically pick the next pending feature from the development roadmap.

---

## Pre-Flight Checks

1. Read `.fullstack/config.json`. Verify `status` is `"setup"` or `"developing"`. If `"analyzed"`, instruct: "Run `/fullstack-setup` first."
2. If status is `"setup"`, update to `"developing"`.
3. Determine the target feature:
   - If `$ARGUMENTS` provided (e.g., `"2/3"` for module 2, feature 3): parse and locate that feature
   - If no argument: find the first feature across all modules with `status: "pending"`, following the order in `development-roadmap.md`
4. If no pending features remain, output: "All features complete! Run `/fullstack-status` to see the full report."
5. Read the target module's `module-config.json`. If module status is `"pending"`, update to `"in_progress"`.

---

## Step 1: Load Context

Read these files and hold their content for constructing agent prompts:

1. `.fullstack/config.json` -- project config
2. `.fullstack/design-decisions.md` -- architectural decisions
3. `.fullstack/system-architecture.md` -- module/feature overview
4. `.fullstack/database-schema.md` -- DB schema for relevant tables
5. `.fullstack/api-contracts.md` -- API contracts for relevant endpoints
6. `.fullstack/development-roadmap.md` -- execution order
7. The original PRD (path from config.json) -- relevant sections
8. Module's `module-config.json`
9. Previous feature's `handoff.md` (if this is not the first feature)
10. Previous feature's `completion-report.md` (if exists)

---

## Step 2: Analyzing Phase

Update `feature-config.json` status to `"analyzing"`.

Create the feature directory:
```
.fullstack/module-{N}-{slug}/feature-{M}-{slug}/
```

### Create `feature-spec.md`
Extract from the PRD and system architecture only the requirements relevant to this feature:
- Functional requirements
- Data entities involved
- UI screens involved
- API endpoints needed
- Acceptance criteria (specific, testable)
- Dependencies on prior features

### Create `backend-plan.md`
Ordered task list for backend implementation:

```markdown
# Backend Plan: {feature-name}

## Tasks

| ID | Task | Target Files | Priority |
|----|------|-------------|----------|
| B1 | Create/update SQLAlchemy model for {entity} | backend/app/models/{entity}.py | High |
| B2 | Create Pydantic schemas (Create/Update/Response) | backend/app/schemas/{scope}/{entity}.py | High |
| B3 | Create service with business logic | backend/app/services/{scope}/{entity}_service.py | High |
| B4 | Create API endpoint(s) | backend/app/api/{scope}/v1/{entity}.py | High |
| B5 | Register route in route registry | backend/app/route/router_registry.py | High |
| B6 | Create Alembic migration | backend/migrations/versions/ | High |
| B7 | Write pytest tests | backend/tests/ | High |

## API Endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | /api/v1/{entity} | Required | Create {entity} |
| GET | /api/v1/{entity} | Required | List {entities} |
| ... | ... | ... | ... |

## Database Changes

{Table definitions from database-schema.md relevant to this feature}
```

### Create `frontend-plan.md`
Ordered task list for frontend implementation:

```markdown
# Frontend Plan: {feature-name}

## Tasks

| ID | Task | Target Files | Priority |
|----|------|-------------|----------|
| F1 | Create API client functions | frontend/src/apis/{entity}.ts | High |
| F2 | Create/update Zustand store | frontend/src/store/use{Entity}Store.ts | High |
| F3 | Create TanStack Query hooks | frontend/src/hooks/use{Entity}.ts | High |
| F4 | Create UI components | frontend/src/components/{entity}/ | High |
| F5 | Create/update page(s) | frontend/src/pages/{Entity}.tsx | High |
| F6 | Update router config | frontend/src/router/routes.tsx | Medium |
| F7 | Add i18n translation keys | frontend/src/locales/{lang}/ | Medium |
| F8 | Write Vitest tests | frontend/src/components/{entity}/__tests__/ | High |

## Pages & Components

| Component | Type | Description |
|-----------|------|-------------|
| {EntityList} | Page | List view with table/cards |
| {EntityForm} | Component | Create/edit form |
| ... | ... | ... |
```

Save `feature-config.json`:
```json
{
  "feature_name": "{name}",
  "module_name": "{module-name}",
  "module_index": {N},
  "feature_index": {M},
  "status": "analyzing",
  "created_at": "{ISO timestamp}",
  "completed_at": null,
  "phases": {
    "analyzing": { "status": "completed", "timestamp": "{now}" },
    "backend_dev": { "status": "pending", "timestamp": null },
    "frontend_dev": { "status": "pending", "timestamp": null },
    "integrating": { "status": "pending", "timestamp": null },
    "testing": { "status": "pending", "timestamp": null }
  }
}
```

---

## Step 3: Backend Development Phase

Update feature status to `"backend_dev"`.

Dispatch the **backend-developer** agent via `Agent` tool with `subagent_type: "fullstack-dev:backend-developer"`.

### Construct the agent prompt

**CRITICAL: Prompt quality = output quality.** Include ALL of the following in the prompt:

1. **Feature spec** -- paste the full content of `feature-spec.md`
2. **Backend plan** -- paste the full content of `backend-plan.md`
3. **Database schema** -- paste the relevant tables from `database-schema.md`
4. **API contracts** -- paste the relevant endpoints from `api-contracts.md`
5. **Design decisions** -- paste decisions relevant to this feature
6. **Pattern references** -- read 1-2 existing backend files that demonstrate the patterns the agent should follow:
   - Read an existing model file from `backend/app/models/` (e.g., `admin.py` or `user.py`)
   - Read an existing service file from `backend/app/services/`
   - Read an existing endpoint file from `backend/app/api/`
   - Read the route registry file
   - Paste these file contents with their paths into the prompt
7. **CLAUDE.md conventions** -- paste relevant backend conventions
8. **Previous handoff** -- if exists, paste the handoff notes from the previous feature

### After agent completes

1. Verify all target files from `backend-plan.md` were created/modified
2. Run Alembic migration: `cd backend && alembic upgrade head`
3. Run backend tests: `cd backend && pytest -v`
4. If tests fail: read the error, attempt to fix (max 2 attempts). If still failing after 2 attempts, report to user and ask how to proceed.
5. Commit: `git add backend/ && git commit -m "feat(backend): implement {feature-name} for {module-name}"`
6. Update `implementation-log.md` with backend section
7. Update feature phase: `backend_dev.status = "completed"`

---

## Step 4: Frontend Development Phase

Update feature status to `"frontend_dev"`.

Dispatch the **frontend-developer** agent via `Agent` tool with `subagent_type: "fullstack-dev:frontend-developer"`.

### Construct the agent prompt

Include ALL of the following:

1. **Feature spec** -- paste full content of `feature-spec.md`
2. **Frontend plan** -- paste full content of `frontend-plan.md`
3. **Backend implementation log** -- paste the backend section from `implementation-log.md` so the frontend agent knows exact endpoint URLs and response shapes
4. **Pattern references** -- read 1-2 existing frontend files:
   - Read an existing API client from `frontend/src/apis/`
   - Read an existing Zustand store from `frontend/src/store/`
   - Read an existing hook from `frontend/src/hooks/`
   - Read an existing page component from `frontend/src/pages/`
   - Read the router config from `frontend/src/router/routes.tsx` (or `.jsx`)
   - Paste all with file paths
5. **Design decisions** -- paste relevant frontend decisions
6. **CLAUDE.md conventions** -- paste relevant frontend conventions
7. **Previous handoff** -- if exists, paste handoff notes

### After agent completes

1. Verify all target files from `frontend-plan.md` were created/modified
2. Run frontend tests: `cd frontend && pnpm test`
3. Run lint: `cd frontend && pnpm lint`
4. If tests or lint fail: attempt fix (max 2 attempts)
5. Commit: `git add frontend/ && git commit -m "feat(frontend): implement {feature-name} for {module-name}"`
6. Update `implementation-log.md` with frontend section
7. Update feature phase: `frontend_dev.status = "completed"`

---

## Step 5: Integration Verification Phase

Update feature status to `"integrating"`.

Dispatch the **integration-verifier** agent via `Agent` tool with `subagent_type: "fullstack-dev:integration-verifier"`.

### Construct the agent prompt

Include:
1. **Backend implementation log** -- endpoints, methods, schemas, response shapes
2. **Frontend implementation log** -- API client calls, expected shapes
3. **API contracts** from `api-contracts.md`
4. **Actual source files**:
   - Backend endpoint file(s) for this feature
   - Frontend API client file(s) for this feature
   - Frontend hook file(s) that call the API

### After agent completes

1. Read `integration-report.md`
2. If mismatches found and fixed: commit `git add -A && git commit -m "fix: align frontend-backend contracts for {feature-name}"`
3. Update feature phase: `integrating.status = "completed"`

---

## Step 6: Testing Phase

Update feature status to `"testing"`.

Run comprehensive tests:

```bash
cd backend && pytest -v
cd frontend && pnpm test
cd frontend && pnpm lint
cd backend && ruff check .
```

Save results to `.fullstack/module-{N}-{slug}/feature-{M}-{slug}/test-results.md`:

```markdown
# Test Results: {feature-name}

## Backend
- Tests: {pass}/{total} passing
- Lint (ruff): {pass|fail}
- Type check (mypy): {pass|fail} (optional)

## Frontend
- Tests: {pass}/{total} passing
- Lint (eslint): {pass|fail}
- Type check (tsc): {pass|fail} (optional)

## Integration
- Contract verification: {verified|issues}
```

If any tests fail:
1. Read the error output
2. Attempt self-healing fix (identify file, read it, fix the issue)
3. Re-run the failing test
4. If still failing after 2 attempts, include failure details in the PAUSE output

Update feature phase: `testing.status = "completed"`

---

## Step 7: PAUSE for User Testing

**This is mandatory.** The workflow STOPS here and presents results to the user.

Output:

```
Feature Ready for Testing: {module-name} / {feature-name}
===================================================

Backend:
  - {N} files created, {M} files modified
  - Endpoints: {list endpoint paths}
  - Tests: {pass}/{total} passing

Frontend:
  - {N} files created, {M} files modified
  - Pages: {list new/updated pages}
  - Tests: {pass}/{total} passing

Integration: {verified | N issues found}

Please test the following acceptance criteria:
  [ ] {criterion 1}
  [ ] {criterion 2}
  [ ] {criterion 3}

When done testing:
  /fullstack-done          Mark complete, move to next feature
  /fullstack-done fix      Report issues to fix before proceeding
  /fullstack-test          Re-run tests
  /fullstack-status        View overall progress
```

**DO NOT automatically proceed to the next feature.** Wait for the user to explicitly run `/fullstack-done`.

---

## Error Handling

| Error | Response |
|-------|----------|
| No pending features | Output: "All features complete!" |
| Agent produces incomplete code | Re-dispatch with more specific prompt |
| Backend tests fail (self-heal fails) | Report exact errors, ask user |
| Frontend tests fail (self-heal fails) | Report exact errors, ask user |
| Alembic migration conflict | Read error, resolve, regenerate migration |
| Integration mismatch unresolvable | Report details, ask user which side to fix |
| Context too large for agent | Split into focused sub-calls |
| Feature depends on incomplete feature | Block and report dependency |
