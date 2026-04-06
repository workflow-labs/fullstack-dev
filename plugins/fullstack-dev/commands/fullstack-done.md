# /fullstack-done -- Complete Current Feature or Fix Issues

**Purpose:** Either mark the current feature as complete (generating reports and advancing to next feature) or enter a fix cycle to address issues found during user testing.

**Invocation:**
```
/fullstack-done [fix]
```

- No argument: Mark current feature as complete, advance to next
- `fix`: Enter fix cycle -- collect issues, fix them, re-test, pause again

---

## Pre-Flight Checks

1. Read `.fullstack/config.json`. Verify `status` is `"developing"`.
2. Identify current feature from config.
3. Read current feature's `feature-config.json`.
4. Verify feature status is `"testing"` (meaning `/fullstack-dev` has completed all phases and paused).

If feature status is not `"testing"`:
- If `"pending"` or `"analyzing"`: "Feature not started. Run `/fullstack-dev` first."
- If `"backend_dev"` or `"frontend_dev"` or `"integrating"`: "Feature still in development. Run `/fullstack-dev` to continue."
- If `"completed"`: "Feature already completed. Run `/fullstack-dev` for the next feature."

---

## Path A: Fix Mode (`/fullstack-done fix`)

### Step A1: Collect Issue Description

Ask the user to describe the issues via `AskUserQuestion`:

```
What issues did you find during testing?

Options:
1. Backend bug (API returns wrong data, error, or doesn't work)
2. Frontend bug (UI doesn't render correctly, interactions broken)
3. Integration issue (frontend and backend don't communicate properly)
4. Other (describe in detail)
```

Allow free-text for detailed description.

### Step A2: Diagnose and Fix

Based on the issue type:

**Backend bug:**
- Read the relevant backend files (endpoint, service, model)
- Read the implementation log
- Dispatch **backend-developer** agent with:
  - Issue description
  - Relevant source files (pasted)
  - Feature spec
  - Instruction to FIX the specific issue, not rewrite
- After fix: run `cd backend && pytest -v`
- Commit: `fix(backend): {brief description of fix}`

**Frontend bug:**
- Read the relevant frontend files (component, page, hook, store)
- Read the implementation log
- Dispatch **frontend-developer** agent with:
  - Issue description
  - Relevant source files (pasted)
  - Feature spec
  - Instruction to FIX the specific issue, not rewrite
- After fix: run `cd frontend && pnpm test`
- Commit: `fix(frontend): {brief description of fix}`

**Integration issue:**
- Read both backend endpoint and frontend API client files
- Dispatch **integration-verifier** agent with:
  - Issue description
  - Both source files
  - API contracts
- After fix: run both test suites
- Commit: `fix: resolve integration issue for {feature-name}`

### Step A3: Record Feedback

Append to feature-config.json `user_feedback` array:
```json
{
  "timestamp": "{ISO timestamp}",
  "issue": "{description}",
  "type": "backend|frontend|integration|other",
  "resolution": "{what was fixed}",
  "files_changed": ["{list of files}"]
}
```

### Step A4: PAUSE Again

Re-run tests and output results. The user must test again:

```
Fix Applied: {brief description}
===================================================

Files changed: {list}
Tests: {pass}/{total} passing

Please re-test. When done:
  /fullstack-done          Mark complete
  /fullstack-done fix      Report more issues
```

**DO NOT auto-advance.** The user must explicitly run `/fullstack-done` without `fix` to proceed.

---

## Path B: Complete Feature (`/fullstack-done` without argument)

### Step B1: Generate Completion Report

Create `.fullstack/module-{N}-{slug}/feature-{M}-{slug}/completion-report.md`:

```markdown
# Completion Report: {feature-name}

## Summary
{1-2 sentence summary of what was built}

## Backend Changes
| File | Action | Description |
|------|--------|-------------|
| backend/app/models/{x}.py | Created | {entity} model with {fields} |
| backend/app/schemas/{x}.py | Created | Create/Update/Response schemas |
| backend/app/services/{x}.py | Created | Business logic for {feature} |
| backend/app/api/{x}.py | Created | {N} endpoints |
| backend/migrations/versions/{x}.py | Created | Migration for {tables} |
| backend/tests/{x}.py | Created | {N} tests |

## Frontend Changes
| File | Action | Description |
|------|--------|-------------|
| frontend/src/apis/{x}.ts | Created | API client for {feature} |
| frontend/src/store/{x}.ts | Created | Zustand store for {state} |
| frontend/src/hooks/{x}.ts | Created | Query/mutation hooks |
| frontend/src/pages/{x}.tsx | Created | {page} page |
| frontend/src/components/{x}/ | Created | {N} components |
| frontend/src/router/routes.tsx | Modified | Added route for {page} |

## API Endpoints Implemented
| Method | Path | Auth | Status |
|--------|------|------|--------|
| POST | /api/v1/{x} | Required | Working |
| GET | /api/v1/{x} | Required | Working |

## Test Results
- Backend: {pass}/{total} passing
- Frontend: {pass}/{total} passing
- Integration: verified

## Issues Fixed During Testing
{list from user_feedback array, or "None"}

## Deviations from Plan
{any differences between the plan and what was actually implemented, or "None"}
```

### Step B2: Generate Handoff Document

Create `.fullstack/module-{N}-{slug}/feature-{M}-{slug}/handoff.md`:

```markdown
# Handoff: {feature-name} -> Next Feature

## What Was Built
{brief description of implemented feature}

## Database State
{tables added/modified in this feature, with key columns}

## API Endpoints Available
| Method | Path | Description |
|--------|------|-------------|
| ... | ... | ... |

## Frontend State
- New routes: {list}
- New pages: {list}
- New components: {list}
- Store changes: {list}

## Patterns Established
{any new patterns introduced that the next feature should follow}

## Known Limitations
{anything deferred or simplified that might affect future features}

## Notes for Next Feature
{specific things the next feature's agents need to know}
```

### Step B3: Update State

1. Update `feature-config.json`:
   - `status` -> `"completed"`
   - `completed_at` -> current ISO timestamp

2. Update `module-config.json`:
   - Increment `completed_features`
   - Update the feature entry: `status` -> `"completed"`, `completed_at` -> now
   - If this was the LAST feature in the module:
     - Set module `status` -> `"completed"`
     - Generate `module-completion-report.md` summarizing all features

3. Update `.fullstack/config.json`:
   - Advance `current_feature` to the next pending feature
   - If this was the last feature of the current module, advance `current_module`
   - If ALL modules are complete:
     - Set `status` -> `"completed"`
     - Generate `.fullstack/project-completion-report.md`

4. Commit state files: `git add .fullstack/ && git commit -m "docs: complete {feature-name}, prepare handoff"`

### Step B4: Output Next Steps

**If more features remain:**
```
Feature Completed: {module-name} / {feature-name}
===================================================

Progress: [{completed}/{total}] features complete

Next: Module {N} / Feature {M} -- {next-feature-name}

Run /fullstack-dev to start the next feature.
```

**If current module is complete but more modules remain:**
```
Module Completed: {module-name}
===================================================

All {N} features in this module are done.

Progress: [{completed-modules}/{total-modules}] modules complete
          [{completed-features}/{total-features}] features complete

Next module: Module {N} -- {next-module-name} ({feature-count} features)

Run /fullstack-dev to start the next module.
```

**If all modules are complete:**
```
Project Complete: {project-name}
===================================================

All {total-modules} modules and {total-features} features are implemented.

Summary:
  - Backend: {total-backend-files} files
  - Frontend: {total-frontend-files} files
  - Tests: {total-tests} passing
  - Endpoints: {total-endpoints}

Full report: .fullstack/project-completion-report.md

Congratulations!
```

---

## Error Handling

| Error | Response |
|-------|----------|
| Feature not in "testing" status | Guide user to correct command |
| Fix attempt fails (agent error) | Report specific error, ask user |
| State file corrupted | Attempt to reconstruct from git history |
| No more features but status not updated | Force-complete remaining state |
