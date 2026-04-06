# /fullstack-setup -- Clone Boilerplates and Initialize Project

**Purpose:** Clone the frontend and backend boilerplates, customize them for the project, set up the monorepo structure, and verify both services can start. This is the transition from planning to code.

**Invocation:**
```
/fullstack-setup
```

---

## Pre-Flight Checks

1. Read `.fullstack/config.json`. If missing, instruct: "Run `/fullstack-init <prd-path>` first."
2. Verify `status` is `"analyzed"`. If `"setup"` or later, warn: "Project already initialized. Run `/fullstack-dev` to start development."
3. Verify no `/frontend` or `/backend` directories exist. If they do, ask user: overwrite or abort.

---

## Step 1: Clone Frontend Boilerplate

```bash
git clone https://github.com/adamchins/react-vite-boilerplate frontend
rm -rf frontend/.git
```

If clone fails (network error, repo unavailable), ask user for alternative: local path or different repo URL.

---

## Step 2: Clone Backend Boilerplate

```bash
git clone https://github.com/adamchins/zetos-fastapi backend
rm -rf backend/.git
```

Same error handling as Step 1.

---

## Step 3: Customize Frontend

Read `.fullstack/config.json` for project name and `.fullstack/design-decisions.md` for any frontend-specific decisions.

1. **`frontend/package.json`**: Update `name` to project slug, update `description`
2. **`frontend/.env.example`** (copy to `.env`): Set `VITE_APP_TITLE` to project name, set `VITE_API_BASE_URL` to `http://localhost:8001/api/v1`
3. **`frontend/vite.config.ts`** (or `.js`): Verify proxy target points to `http://localhost:8001`
4. **`frontend/index.html`**: Update `<title>` to project name
5. **`frontend/src/locales/en/*.json`**: Update app name strings if present

---

## Step 4: Customize Backend

Read `.fullstack/config.json` for project name and `.fullstack/database-schema.md` for base tables.

1. **`backend/.env.example`** (copy to `.env`):
   - Set `PROJECT_NAME` to project name
   - Set `POSTGRES_DB` to project slug (lowercase, underscores)
   - Generate a random `SECRET_KEY` (use `python -c "import secrets; print(secrets.token_hex(32))"`)
   - Set `CORS_ORIGINS` to include `http://localhost:3000` (frontend dev server)
2. **`backend/app/core/config.py`**: Verify project name matches
3. **`backend/docker-compose.yml`** (or `docker-compose.dev.yml`):
   - Update service names to use project slug
   - Update `POSTGRES_DB` to match .env

---

## Step 5: Set Up Monorepo Root

Create these files in the project root:

### Root `package.json` (for convenience scripts)
```json
{
  "name": "{project-slug}",
  "private": true,
  "scripts": {
    "dev:frontend": "cd frontend && pnpm dev",
    "dev:backend": "cd backend && python main.py",
    "test:frontend": "cd frontend && pnpm test",
    "test:backend": "cd backend && pytest",
    "lint:frontend": "cd frontend && pnpm lint",
    "lint:backend": "cd backend && ruff check .",
    "typecheck:frontend": "cd frontend && pnpm tsc --noEmit",
    "typecheck:backend": "cd backend && mypy app/"
  }
}
```

### Root `docker-compose.yml`
Create a combined docker-compose that orchestrates both stacks:
- PostgreSQL service
- Redis service
- Backend service (depends on postgres + redis)
- Frontend dev service (optional, or use local dev server)

Base this on the backend's existing `docker-compose.dev.yml` and extend it.

### Root `.gitignore`
```
# Dependencies
node_modules/
__pycache__/
*.pyc
venv/
.venv/

# Environment
.env
.env.local

# Build
frontend/dist/
backend/*.egg-info/

# IDE
.vscode/
.idea/

# State
.fullstack/

# OS
.DS_Store
Thumbs.db
```

### Root `CLAUDE.md`
Create a project-level CLAUDE.md with:
- Project name and description
- Monorepo structure (`/frontend`, `/backend`)
- Tech stack summary (both stacks)
- Key conventions from both boilerplates
- How to run: dev servers, tests, lints
- Database migration commands
- Directory structure for both stacks

### Root `README.md`
Create a basic README with project name, description, and setup instructions.

---

## Step 6: Create Base Database Migration

Read `.fullstack/database-schema.md` to identify tables that are foundational (shared across all modules). Typically:
- `users` table (if not already in boilerplate)
- `admins` table (if not already in boilerplate)
- Any base config/settings tables

**Important:** The boilerplate already has User and Admin models. Only create migrations for additional base tables identified in the schema that don't already exist.

If the base schema requires new tables:
```bash
cd backend
alembic revision --autogenerate -m "add base tables for {project-name}"
```

If no additional base tables are needed, skip this step.

---

## Step 7: Install Dependencies

```bash
cd frontend && pnpm install
```

```bash
cd backend && pip install -r requirements.txt
```

If using a virtual environment for backend:
```bash
cd backend && python -m venv venv && source venv/bin/activate && pip install -r requirements.txt
```

Report any dependency installation errors to the user.

---

## Step 8: Verify Services Start

### Backend verification:
1. Start backend: `cd backend && python main.py &` (or use docker-compose)
2. Wait for startup (2-3 seconds)
3. Hit health check: `curl -s http://localhost:8001/api/v1/config/health`
4. Verify response includes `"status": "healthy"` or similar
5. Stop the background process

### Frontend verification:
1. Start frontend: `cd frontend && pnpm dev &`
2. Wait for startup (3-5 seconds)
3. Hit dev server: `curl -s http://localhost:3000`
4. Verify HTML response (React app shell)
5. Stop the background process

If either fails, report the specific error and suggest fixes. Common issues:
- Port already in use: suggest alternative port
- Database not running: suggest `docker-compose up -d postgres redis`
- Missing env vars: check .env file

---

## Step 9: Initial Git Commit

```bash
git init
git add -A
git commit -m "chore: initialize {project-name} monorepo with frontend and backend boilerplates"
```

---

## Step 10: Update State

Update `.fullstack/config.json`:
- Set `status` to `"setup"`
- Set `current_module` to `1`
- Set `current_feature` to the first feature path

---

## Output

```
Project Initialized: {project-name}
===================================================

Frontend:  /frontend (React + Vite + TypeScript)
Backend:   /backend  (FastAPI + SQLAlchemy + PostgreSQL)

Services verified:
  Backend:  http://localhost:8001 -- OK
  Frontend: http://localhost:3000 -- OK

Next step: Run /fullstack-dev to start developing the first feature.
  First up: Module 1 ({module-name}) / Feature 1.1 ({feature-name})
```

---

## Error Handling

| Error | Response |
|-------|----------|
| Git clone fails | Report error, ask for local path or different URL |
| Directories already exist | Ask: overwrite or abort |
| Dependency install fails | Report specific error, suggest fix |
| Backend health check fails | Check Docker, DB connection, port conflicts |
| Frontend dev server fails | Check node version, port conflicts, missing deps |
| `.fullstack/config.json` missing | Instruct to run `/fullstack-init` first |
