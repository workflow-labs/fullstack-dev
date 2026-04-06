# /fullstack-status -- Show Development Progress

**Purpose:** Display the current project development progress across all modules and features, with phase-level detail for the active feature.

**Invocation:**
```
/fullstack-status [module-index]
```

If a module index is provided, show detailed feature-level view for that module.

---

## Pre-Flight Checks

1. Read `.fullstack/config.json`. If missing, output: "No project found. Run `/fullstack-init <prd-path>` first."
2. Read the project status field.

---

## Step 1: Load All Module States

Use `Glob` to find all `module-config.json` files:
```
.fullstack/module-*/module-config.json
```

Read each one to collect module names, statuses, feature counts, and completion data.

---

## Step 2: Display Overview (no argument)

If no `$ARGUMENTS` provided, show the overview dashboard:

```
Fullstack Dev: {project-name}
===================================================
Status: {status}
Progress: [{completed-features}/{total-features}] {progress-bar} {percentage}%

Module 1: {name}                    [{completed}/{total}] {bar} {STATUS}
  1.1 {feature-name}               completed   {date}
  1.2 {feature-name}               completed   {date}
  1.3 {feature-name}               completed   {date}

Module 2: {name}                    [{completed}/{total}] {bar} IN PROGRESS
  2.1 {feature-name}               completed   {date}
  2.2 {feature-name}               frontend_dev  <-- CURRENT
  2.3 {feature-name}               pending
  2.4 {feature-name}               pending

Module 3: {name}                    [{completed}/{total}] {bar} PENDING
  (features hidden -- not started)

Module 4: {name}                    [{completed}/{total}] {bar} PENDING
  (features hidden -- not started)

---
Current: Module 2 / Feature 2.2 ({feature-name}) -- frontend_dev phase

Commands:
  /fullstack-dev          Continue development (next feature)
  /fullstack-test         Run tests for current feature
  /fullstack-done         Complete current feature
```

**Progress bar format:** Use `=` for completed and `.` for remaining, 10 chars wide.
- `[3/3] ==========` COMPLETED
- `[1/4] ==........` IN PROGRESS
- `[0/3] ..........` PENDING

**Status labels:**
- `pending` -- not yet started
- `analyzing` -- extracting requirements
- `backend_dev` -- implementing backend
- `frontend_dev` -- implementing frontend
- `integrating` -- verifying contracts
- `testing` -- running tests
- `completed` -- done, with date

---

## Step 3: Display Module Detail (with argument)

If `$ARGUMENTS` contains a module index, show detailed view:

```
Module {N}: {name}
===================================================
Status: {status}
Description: {description}
Dependencies: {list or "none"}
Progress: [{completed}/{total}] features

Feature {N}.1: {name}                         COMPLETED
  Complexity: {simple|moderate|complex}
  Status: completed (2026-04-06)
  Backend:  5 files created
  Frontend: 4 files created
  Tests:    12/12 passing

Feature {N}.2: {name}                         IN PROGRESS
  Complexity: {moderate}
  Phase: frontend_dev
  Phases:
    [x] analyzing      -- 2026-04-07 10:00
    [x] backend_dev    -- 2026-04-07 11:30
    [ ] frontend_dev   -- in progress
    [ ] integrating    -- pending
    [ ] testing        -- pending

Feature {N}.3: {name}                         PENDING
  Complexity: {simple}
  Status: pending
```

Read each feature's `feature-config.json` and `completion-report.md` (if exists) for detailed data.

---

## Error Handling

| Error | Response |
|-------|----------|
| `.fullstack/config.json` missing | "No project found. Run `/fullstack-init` first." |
| Module index out of range | "Module {N} not found. Available: 1-{max}" |
| Module config file corrupted | Report error, suggest re-reading |
