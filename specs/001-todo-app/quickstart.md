# Quickstart Guide: Todo App Local Development

**Feature**: Phase II Full-Stack Todo Web Application
**Date**: 2025-12-31
**Target**: Local development setup (MacOS, Linux, Windows)

## Prerequisites

Ensure the following tools are installed:

- **Node.js**: v18+ (for Next.js frontend)
- **Python**: 3.9+ (for FastAPI backend)
- **uv**: Python package manager ([installation guide](https://astral.sh/uv))
- **Git**: For version control
- **PostgreSQL Client** (optional): For manual database inspection

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    User Browser                             │
└────────────┬────────────────────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────────────────────┐
│           Next.js Frontend (Port 3000)                      │
│  - Better Auth (JWT issuance, user management)             │
│  - React UI components (TypeScript, Tailwind)              │
└────────────┬────────────────────────────────────────────────┘
             │
             ├──────────► Better Auth API (/api/auth/*)
             │
             └──────────► FastAPI Backend (/api/*)
                          │
                          ▼
             ┌────────────────────────────────┐
             │  FastAPI Backend (Port 8000)   │
             │  - JWT verification via JWKS   │
             │  - Todo CRUD operations        │
             └────────────┬───────────────────┘
                          │
                          ▼
             ┌────────────────────────────────┐
             │  Neon PostgreSQL (Cloud)       │
             │  - Better Auth tables (users)  │
             │  - FastAPI tables (todos)      │
             └────────────────────────────────┘
```

## Step 1: Clone Repository

```bash
# Clone the repository
git clone <repository-url>
cd phase_3

# Checkout feature branch
git checkout 001-todo-app
```

## Step 2: Database Setup (Neon PostgreSQL)

### 2.1 Create Neon Database

1. Sign up at [https://neon.tech](https://neon.tech)
2. Create a new project (e.g., "todo-app-dev")
3. Copy the connection string (format: `postgresql://user:password@host/database?sslmode=require`)

### 2.2 Configure Environment Variables

Create `.env` file in repository root:

```bash
# .env
DATABASE_URL=postgresql://user:password@host/database?sslmode=require
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=<generate-random-secret>
FASTAPI_URL=http://localhost:8000
```

**Generate NEXTAUTH_SECRET**:
```bash
# Option 1: Using Node.js
node -e "console.log(require('crypto').randomBytes(32).toString('base64'))"

# Option 2: Using OpenSSL
openssl rand -base64 32
```

## Step 3: Frontend Setup (Next.js)

```bash
# Navigate to frontend directory
cd frontend

# Install dependencies
npm install

# Install Better Auth
npm install better-auth

# Run database migrations for Better Auth
npx better-auth migrate

# Start development server
npm run dev
```

**Expected Output**:
```
> next dev
  ▲ Next.js 14.x.x
  - Local:        http://localhost:3000
  - Ready in 2.3s
```

**Verify**:
- Open browser: `http://localhost:3000`
- You should see the landing page (or login page if redirected)

## Step 4: Backend Setup (FastAPI)

```bash
# Navigate to backend directory (from repo root)
cd backend

# Create virtual environment with uv
uv venv

# Activate virtual environment
# Windows (PowerShell):
.venv\Scripts\Activate.ps1
# MacOS/Linux:
source .venv/bin/activate

# Install dependencies
uv pip install -r requirements.txt

# Run database migrations (Alembic)
alembic upgrade head

# Start development server
uvicorn src.main:app --reload --port 8000
```

**Expected Output**:
```
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [12345]
INFO:     Started server process [12346]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

**Verify**:
- Open browser: `http://localhost:8000/docs`
- You should see interactive Swagger API documentation

## Step 5: Verify Integration

### 5.1 Test User Registration

1. Open frontend: `http://localhost:3000`
2. Click "Register" (or navigate to `/register`)
3. Enter email: `test@example.com`
4. Enter password: `testpass123` (min 8 characters)
5. Click "Sign Up"

**Expected Behavior**:
- Account created successfully
- Redirected to todo dashboard
- JWT token stored in browser

### 5.2 Test User Sync

After registration, the frontend should automatically call `POST /api/users/sync`.

**Verify in browser DevTools**:
1. Open Network tab
2. Look for request to `http://localhost:8000/api/users/sync`
3. Status should be `200 OK`
4. Response body: `{"id": "...", "email": "test@example.com", "created_at": "..."}`

**Verify in database** (optional):
```sql
-- Connect to Neon database
psql <DATABASE_URL>

-- Check users_sync table
SELECT * FROM users_sync;
-- Should show your test user
```

### 5.3 Test Todo CRUD

1. **Create Todo**:
   - Enter title: "Test Todo"
   - Click "Add Todo"
   - Todo appears in list

2. **Toggle Completion**:
   - Click checkbox next to todo
   - Visual style changes (strikethrough)
   - Refresh page - status persists

3. **Edit Todo**:
   - Click "Edit" button
   - Change title to "Updated Todo"
   - Click "Save"
   - Title updates

4. **Delete Todo**:
   - Click "Delete" button
   - Confirm deletion prompt
   - Todo removed from list

## Step 6: Test Authentication Flow

### 6.1 Logout and Login

1. Click "Logout" in UI
2. Redirected to login page
3. Enter same credentials: `test@example.com` / `testpass123`
4. Click "Sign In"
5. Redirected to dashboard with todos still present

### 6.2 Verify JWT Expiration

JWT tokens expire after 1 hour.

**Test expiration handling**:
1. Login and get JWT token
2. Wait 1 hour (or manually set system clock forward)
3. Try to create a todo
4. Expected: 401 Unauthorized response, redirected to login

**Quick test** (optional):
- Manually modify JWT `exp` claim in browser DevTools
- Next API request should fail with 401

## Step 7: Test Data Isolation

Verify that users can only access their own todos:

### 7.1 Create Two Users

1. Register user 1: `alice@example.com`
2. Create todo: "Alice's Todo"
3. Logout
4. Register user 2: `bob@example.com`
5. Create todo: "Bob's Todo"

### 7.2 Verify Isolation

- Bob should see only "Bob's Todo" (not "Alice's Todo")
- Logout, login as Alice
- Alice should see only "Alice's Todo" (not "Bob's Todo")

### 7.3 Test Cross-User Access (API level)

1. Login as Alice, copy JWT token from DevTools (Application → Cookies or Local Storage)
2. Use curl to try accessing a todo ID owned by Bob:

```bash
curl -X GET http://localhost:8000/api/todos/{bob_todo_id} \
  -H "Authorization: Bearer {alice_jwt_token}"
```

**Expected Response**: `404 Not Found`
```json
{
  "detail": "Todo not found",
  "error_code": "NOT_FOUND"
}
```

## Common Issues & Troubleshooting

### Issue 1: `DATABASE_URL` not found

**Error**:
```
OperationalError: could not translate host name "..." to address
```

**Solution**:
- Verify `.env` file exists in repository root
- Check `DATABASE_URL` is correctly set with Neon connection string
- Restart backend server after updating `.env`

---

### Issue 2: Better Auth migration fails

**Error**:
```
Error: Cannot find module 'better-auth/cli'
```

**Solution**:
```bash
cd frontend
npm install better-auth --save
npx better-auth migrate
```

---

### Issue 3: Port 3000 or 8000 already in use

**Error**:
```
EADDRINUSE: address already in use :::3000
```

**Solution**:

**Option 1: Kill existing process**
```bash
# Windows
netstat -ano | findstr :3000
taskkill /PID <pid> /F

# MacOS/Linux
lsof -ti:3000 | xargs kill -9
```

**Option 2: Use different port**
```bash
# Frontend (Next.js)
npm run dev -- -p 3001

# Backend (FastAPI)
uvicorn src.main:app --reload --port 8001
```

Update `FASTAPI_URL` in `.env` if changing backend port.

---

### Issue 4: CORS errors in browser

**Error**:
```
Access to fetch at 'http://localhost:8000/api/todos' from origin 'http://localhost:3000'
has been blocked by CORS policy
```

**Solution**:

Check FastAPI CORS configuration in `backend/src/main.py`:

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],  # Frontend URL
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

---

### Issue 5: JWT verification fails

**Error**:
```
401 Unauthorized: Invalid or expired token
```

**Possible Causes**:
1. **Better Auth JWKS endpoint not accessible**: Verify `http://localhost:3000/api/auth/jwks` returns public keys
2. **Mismatched JWT issuer**: Check Better Auth `issuer` config matches frontend URL
3. **Clock skew**: Ensure system clocks are synchronized

**Debug Steps**:
```bash
# 1. Check JWKS endpoint
curl http://localhost:3000/api/auth/jwks

# 2. Decode JWT (without verification)
# Use jwt.io or:
node -e "console.log(JSON.stringify(JSON.parse(Buffer.from(process.argv[1].split('.')[1], 'base64').toString()), null, 2))" "YOUR_JWT_TOKEN"

# 3. Check FastAPI logs for verification errors
# Look for error messages in uvicorn output
```

---

### Issue 6: Database schema out of sync

**Error**:
```
OperationalError: relation "todos" does not exist
```

**Solution**:

**Backend (FastAPI)**:
```bash
cd backend
alembic downgrade base  # Reset to clean state
alembic upgrade head    # Reapply all migrations
```

**Frontend (Better Auth)**:
```bash
cd frontend
npx better-auth migrate
```

**Nuclear option** (dev environment only):
```bash
# Drop all tables and recreate
psql <DATABASE_URL> -c "DROP SCHEMA public CASCADE; CREATE SCHEMA public;"
cd backend && alembic upgrade head
cd frontend && npx better-auth migrate
```

---

## Development Workflow

### Making Changes

1. **Frontend changes** (components, pages):
   - Edit files in `frontend/src/`
   - Hot reload updates browser automatically

2. **Backend changes** (API endpoints, models):
   - Edit files in `backend/src/`
   - Uvicorn reloads server automatically (`--reload` flag)

3. **Database schema changes**:
   ```bash
   # Edit model in backend/src/models/
   cd backend
   alembic revision --autogenerate -m "Add new field"
   alembic upgrade head
   ```

### Running Tests

**Frontend**:
```bash
cd frontend
npm test
```

**Backend**:
```bash
cd backend
pytest tests/
```

### Viewing Logs

**Frontend** (Next.js):
- Check terminal where `npm run dev` is running
- Browser DevTools → Console tab

**Backend** (FastAPI):
- Check terminal where `uvicorn` is running
- Logs show request details, errors, and stack traces

**Database** (Neon):
- Use Neon web console to view query logs
- Or connect via `psql` for direct inspection

---

## Next Steps

1. ✅ Local development environment working
2. ✅ User registration and authentication tested
3. ✅ Todo CRUD operations verified
4. ✅ Data isolation confirmed

**Ready for development!**

- Start implementing tasks from `tasks.md` (generated by `/sp.tasks` command)
- Follow TDD workflow: write failing test → implement → refactor
- Commit small, atomic changes following Constitution guidelines

---

## Environment Variables Reference

### Required Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `DATABASE_URL` | Neon PostgreSQL connection string | `postgresql://user:pass@host/db` |
| `NEXTAUTH_URL` | Frontend URL (for Better Auth) | `http://localhost:3000` |
| `NEXTAUTH_SECRET` | Secret for JWT signing (min 32 chars) | `<random-base64-string>` |
| `FASTAPI_URL` | Backend API base URL | `http://localhost:8000` |

### Optional Variables (Production)

| Variable | Description | Default |
|----------|-------------|---------|
| `CORS_ORIGINS` | Allowed CORS origins (comma-separated) | `http://localhost:3000` |
| `LOG_LEVEL` | Logging verbosity | `INFO` |
| `JWT_ALGORITHM` | JWT signing algorithm | `RS256` |

---

## Success Checklist

- [x] Node.js 18+ installed
- [x] Python 3.9+ installed
- [x] uv package manager installed
- [x] Neon PostgreSQL database created
- [x] `.env` file configured with DATABASE_URL
- [x] Frontend running on http://localhost:3000
- [x] Backend running on http://localhost:8000
- [x] Better Auth migrations applied
- [x] FastAPI migrations applied
- [x] User registration works
- [x] User sync to FastAPI successful
- [x] Todo CRUD operations functional
- [x] JWT authentication verified
- [x] Data isolation tested
- [x] Swagger docs accessible at /docs

**You're ready to build!** 🚀
