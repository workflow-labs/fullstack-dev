# /fullstack-test -- Run Tests and Verify Current Feature

**Purpose:** Run backend and frontend test suites, linters, and type checks for the current feature. Outputs a formatted results summary.

**Invocation:**
```
/fullstack-test [backend|frontend|all]
```

Default: `all` (run both backend and frontend tests).

---

## Pre-Flight Checks

1. Read `.fullstack/config.json`. Verify `status` is `"developing"`.
2. Identify current feature from `current_feature` field.
3. If no current feature is active, output: "No feature in progress. Run `/fullstack-dev` first."

---

## Step 1: Run Tests

Based on `$ARGUMENTS` (default: `all`):

### Backend Tests
```bash
cd backend && pytest -v 2>&1
```

### Frontend Tests
```bash
cd frontend && pnpm test 2>&1
```

### Backend Lint
```bash
cd backend && ruff check . 2>&1
```

### Frontend Lint
```bash
cd frontend && pnpm lint 2>&1
```

### Type Checks (optional, run if tools available)
```bash
cd backend && mypy app/ 2>&1
cd frontend && pnpm tsc --noEmit 2>&1
```

Run independent commands in parallel where possible (e.g., backend tests and frontend tests can run simultaneously).

---

## Step 2: Read Integration Status

Read the current feature's `integration-report.md` if it exists. Summarize contract verification status.

---

## Step 3: Output Results

```
Test Results: {module-name} / {feature-name}
===================================================

Backend:
  Tests:      {pass}/{total} passing       {PASS|FAIL}
  Lint:       {clean | N issues}           {PASS|FAIL}
  Type Check: {clean | N errors}           {PASS|WARN}

Frontend:
  Tests:      {pass}/{total} passing       {PASS|FAIL}
  Lint:       {clean | N issues}           {PASS|FAIL}
  Type Check: {clean | N errors}           {PASS|WARN}

Integration:
  Contracts:  {verified | N mismatches}    {PASS|FAIL}

Overall: {ALL PASS | ISSUES FOUND}
```

If there are failures, include the first 20 lines of each failure output for quick diagnosis.

---

## Step 4: Update Test Results File

Write/update `.fullstack/module-{N}-{slug}/feature-{M}-{slug}/test-results.md` with the full results.

---

## Error Handling

| Error | Response |
|-------|----------|
| Test command not found | Check if dependencies are installed, suggest fix |
| No test files exist | Report: "No tests found for this feature" |
| Backend not running (for integration tests) | Suggest starting services |
