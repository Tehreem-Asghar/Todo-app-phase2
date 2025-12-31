# Implementation Plan: Phase II Full-Stack Todo Web Application

**Branch**: `001-todo-app` | **Date**: 2025-12-31 | **Spec**: [spec.md](./spec.md)
**Status**: Ready for Implementation

## Summary

Build a secure, production-ready full-stack Todo application that demonstrates spec-driven development and strong security practices for hackathon evaluation. The application enables authenticated users to create, read, update, and delete their personal todo items with complete data isolation.

**Technical Approach**:
- Next.js 14 App Router frontend with Better Auth for JWT-based authentication
- FastAPI backend with SQLModel ORM for type-safe database operations
- Neon Serverless PostgreSQL as shared database (Better Auth + FastAPI tables)
- JWT verification via JWKS endpoint ensures stateless, secure API access
- UUID identifiers prevent enumeration attacks; 404 responses prevent information leakage

**Key Architectural Decisions**:
1. Better Auth handles password hashing and user management (reduces security risk)
2. Shared Neon database simplifies infrastructure and enables native foreign keys
3. JWT-based stateless authentication scales well for API-first architecture
4. User sync flow (`POST /api/users/sync`) bridges Better Auth and FastAPI user records
5. Client-side logout acceptable for hackathon scope (1-hour token expiration limits risk)

## Technical Context

**Language/Version**: Python 3.9+, Node.js 18+, TypeScript 5+
**Primary Dependencies**: FastAPI, SQLModel, Alembic, Next.js 14, Better Auth, TailwindCSS
**Storage**: Neon Serverless PostgreSQL (shared by Better Auth and FastAPI)
**Testing**: pytest (backend), Jest/React Testing Library (frontend)
**Target Platform**: Web application (local development + cloud deployment)
**Project Type**: Web (full-stack monorepo)
**Performance Goals**: <500ms API response time, <2s page load, <1s todo creation
**Constraints**: <200ms JWT verification (p95), stateless authentication, UUID identifiers only
**Scale/Scope**: Hackathon MVP (5 user stories, ~10 API endpoints, 5-10 frontend pages/components)

## Constitution Check

✅ **GATE: Passed - Proceed with implementation**

**Verified Against Constitution** (`.specify/memory/constitution.md` v1.1.0):

### Core Principles
- ✅ **Spec-first development**: Feature spec exists and is complete before implementation begins
- ✅ **Security by default**: All APIs authenticated (JWT), user isolation enforced (filter by user_id)
- ✅ **Test-driven iteration**: Plan includes TDD workflow in Phase 5
- ✅ **Fast feedback loops**: Hackathon-scoped, prioritized by P1/P2/P3 user stories
- ✅ **AI-assisted but human-reviewed**: Plan reviewed before task breakdown

### Key Standards
#### API Design
- ✅ RESTful APIs following FastAPI conventions (GET/POST/PUT/DELETE)
- ✅ All backend routes in dedicated `api` folder, exposed under `/api/` prefix
- ✅ OpenAPI/Swagger auto-documentation (FastAPI built-in)
- ✅ Standard error format: `{"detail": "message", "error_code": "CODE"}` with correct HTTP status codes

#### Security Standards
- ✅ JWT authentication mandatory (all endpoints except `/api/health`)
- ✅ Password hashing: bcrypt via Better Auth (12+ rounds, no custom implementation)
- ✅ User data isolation: All queries filter by `user_id` from JWT (enforced at dependency level)
- ✅ Input validation: Pydantic models for all request bodies (title length, email format, etc.)
- ✅ CORS policy: Whitelist via env vars (no wildcard `*`)
- ✅ SQL injection prevention: SQLModel parameterized queries (no string concatenation)
- ✅ XSS prevention: React default escaping (user content never rendered as raw HTML)

#### Data Layer
- ✅ SQLModel for all database interactions (type-safe ORM)
- ✅ Neon Serverless PostgreSQL (single database, shared schema)
- ✅ Timestamps: `created_at`, `updated_at` on all entities
- ✅ Foreign keys: `todos.user_id` → `users_sync.id` → `users.id`

#### Code Quality
- ✅ Environment variables: All secrets in `.env` (DATABASE_URL, NEXTAUTH_SECRET, etc.)
- ✅ No inline styles: 100% Tailwind CSS (utility classes only)
- ✅ Type safety: TypeScript (frontend), SQLModel/Pydantic (backend)
- ✅ Max function length: 50 lines (extract helpers if longer)
- ✅ Docstrings: All functions documented (purpose, params, return values)

### Constraints
- ✅ Frontend: Next.js App Router, TypeScript, Tailwind CSS
- ✅ Backend: FastAPI, SQLModel
- ✅ Database: Neon Serverless PostgreSQL
- ✅ Authentication: Better Auth with JWT
- ✅ Architecture: Monorepo with shared specs
- ✅ Deployment-ready: Environment-agnostic configuration

**No violations detected. Constitution compliance: 100%**

## Project Structure

### Documentation (this feature)

```text
specs/001-todo-app/
├── spec.md               # Feature specification (complete)
├── plan.md               # This file - implementation plan
├── research.md           # Architecture decisions and research (Phase 0 output)
├── data-model.md         # Entity definitions and schemas (Phase 1 output)
├── quickstart.md         # Local development setup guide (Phase 1 output)
├── contracts/            # API contracts (Phase 1 output)
│   ├── openapi.yaml      # OpenAPI 3.1 specification
│   └── README.md         # Endpoint documentation and examples
└── tasks.md              # Atomic tasks (Phase 2 output - NOT created by /sp.plan)
```

### Source Code (repository root)

**Selected Structure**: Option 2 - Web application (frontend + backend)

```text
phase_3/                      # Repository root
├── .env                      # Environment variables (DATABASE_URL, secrets)
├── .gitignore                # Ignore .env, node_modules/, __pycache__/, etc.
├── README.md                 # Project overview and setup instructions
│
├── frontend/                 # Next.js 14 App Router frontend
│   ├── src/
│   │   ├── app/              # Next.js App Router pages
│   │   │   ├── layout.tsx    # Root layout (providers, auth context)
│   │   │   ├── page.tsx      # Landing page (redirect to login or todos)
│   │   │   ├── (auth)/       # Auth route group (login, register)
│   │   │   │   ├── login/
│   │   │   │   │   └── page.tsx
│   │   │   │   └── register/
│   │   │   │       └── page.tsx
│   │   │   ├── (dashboard)/  # Protected route group (requires auth)
│   │   │   │   └── todos/
│   │   │   │       └── page.tsx
│   │   │   └── api/
│   │   │       └── auth/[...auth]/route.ts  # Better Auth handler
│   │   ├── components/       # Reusable UI components
│   │   │   ├── TodoList.tsx  # Todo list display
│   │   │   ├── TodoItem.tsx  # Individual todo card
│   │   │   ├── TodoForm.tsx  # Create/edit todo form
│   │   │   └── Navbar.tsx    # Navigation with logout
│   │   ├── lib/              # Utilities and configuration
│   │   │   ├── auth.ts       # Better Auth config
│   │   │   ├── api.ts        # API client (fetch wrapper)
│   │   │   └── types.ts      # TypeScript interfaces
│   │   └── middleware.ts     # Auth middleware (protected routes)
│   ├── public/               # Static assets
│   ├── package.json
│   ├── tsconfig.json
│   ├── tailwind.config.ts
│   └── next.config.js
│
├── backend/                  # FastAPI backend
│   ├── src/
│   │   ├── main.py           # FastAPI app entry point, CORS, middleware
│   │   ├── api/              # API route handlers
│   │   │   ├── dependencies.py   # Dependency injection (get_db, get_current_user)
│   │   │   ├── todos.py      # Todo CRUD endpoints
│   │   │   └── users.py      # User sync endpoint
│   │   ├── models/           # SQLModel database models
│   │   │   ├── todo.py       # Todo model
│   │   │   └── user.py       # UserSync model
│   │   ├── schemas/          # Pydantic request/response schemas
│   │   │   ├── todo.py       # TodoCreate, TodoUpdate, TodoResponse
│   │   │   └── user.py       # UserSyncResponse
│   │   └── core/             # Core configuration and utilities
│   │       ├── config.py     # Settings (DATABASE_URL, CORS, etc.)
│   │       └── security.py   # JWT verification (JWKS fetch, token decode)
│   ├── alembic/              # Database migrations
│   │   ├── versions/         # Migration scripts
│   │   └── env.py            # Alembic configuration
│   ├── tests/                # Backend tests
│   │   ├── conftest.py       # Pytest fixtures (test client, test DB)
│   │   ├── test_auth.py      # JWT verification tests
│   │   ├── test_todos.py     # Todo CRUD tests
│   │   └── test_users.py     # User sync tests
│   ├── requirements.txt      # Python dependencies
│   ├── alembic.ini           # Alembic config file
│   └── pytest.ini            # Pytest configuration
│
└── specs/                    # Feature specifications
    └── 001-todo-app/         # This feature's documentation
```

**Structure Decision**: Web application structure chosen because the project requires both a Next.js frontend and FastAPI backend. Separation into `frontend/` and `backend/` directories provides clear boundaries, independent dependency management, and deployment flexibility.

## Complexity Tracking

**No Constitution violations detected.** This section intentionally left empty as no complexity justification is required.

---

## Implementation Phases

### Phase 0: Environment Setup and Dependencies ✅ (Completed)

**Objective**: Establish local development environment and install all required dependencies.

**Tasks**:
1. ✅ Provision Neon PostgreSQL database
2. ✅ Configure `.env` file with `DATABASE_URL`, `NEXTAUTH_SECRET`, etc.
3. ✅ Initialize Next.js frontend project with TypeScript and Tailwind
4. ✅ Install Better Auth (`npm install better-auth`)
5. ✅ Initialize FastAPI backend project with uv
6. ✅ Install FastAPI dependencies (`fastapi`, `sqlmodel`, `alembic`, `pyjwt`, `httpx`)
7. ✅ Run Better Auth migrations (`npx better-auth migrate`)
8. ✅ Initialize Alembic for FastAPI migrations (`alembic init alembic`)

**Deliverables**:
- ✅ Neon database accessible via connection string
- ✅ `.env` file with all required environment variables
- ✅ Frontend project structure with dependencies installed
- ✅ Backend project structure with dependencies installed
- ✅ Better Auth tables created in database (`users`, `sessions`, etc.)

**Acceptance Criteria**:
- ✅ `npm run dev` starts Next.js on http://localhost:3000
- ✅ `uvicorn src.main:app --reload` starts FastAPI on http://localhost:8000
- ✅ Database connection successful (verify with `psql <DATABASE_URL>`)

---

### Phase 1: Authentication Foundation

**Objective**: Implement Better Auth for user registration/login and JWT issuance; enable FastAPI JWT verification via JWKS.

**Dependencies**: Phase 0 complete

#### 1.1 Better Auth Configuration (Frontend)

**Tasks**:
1. Configure Better Auth with Neon PostgreSQL connection
2. Enable email/password authentication plugin
3. Enable JWT plugin with 1-hour expiration and RS256 algorithm
4. Create Better Auth API handler at `/api/auth/[...auth]/route.ts`
5. Create auth client wrapper in `lib/auth.ts`
6. Configure JWKS endpoint exposure at `/api/auth/jwks`

**Files to Create/Modify**:
- `frontend/src/lib/auth.ts`: Better Auth configuration
- `frontend/src/app/api/auth/[...auth]/route.ts`: Better Auth API handler
- `frontend/.env`: Add `NEXTAUTH_URL`, `NEXTAUTH_SECRET`, `DATABASE_URL`

**Acceptance Criteria**:
- Better Auth configured with correct database URL
- JWT tokens include `{id, email, iat, exp}` claims
- JWKS endpoint accessible at `http://localhost:3000/api/auth/jwks`
- JWT expiration set to 3600 seconds (1 hour)

#### 1.2 User Registration and Login (Frontend)

**Tasks**:
1. Create registration page at `/register` (email + password form)
2. Implement registration form with validation (min 8 chars password, email format)
3. Create login page at `/login` (email + password form)
4. Implement login form with error handling (invalid credentials)
5. Store JWT token in httpOnly cookie or localStorage (decision: httpOnly preferred)
6. Redirect to `/todos` after successful login/registration

**Files to Create/Modify**:
- `frontend/src/app/(auth)/register/page.tsx`: Registration page
- `frontend/src/app/(auth)/login/page.tsx`: Login page
- `frontend/src/lib/api.ts`: API client with JWT token handling

**Acceptance Criteria**:
- Users can register with valid email/password (min 8 chars)
- Registration fails with error "Email already registered" for duplicate emails
- Users can login with correct credentials
- Login fails with error "Invalid email or password" for incorrect credentials
- JWT token stored securely after successful authentication
- Users redirected to `/todos` dashboard after login

#### 1.3 JWT Verification in FastAPI (Backend)

**Tasks**:
1. Implement JWKS fetching from Better Auth endpoint
2. Create JWT verification function (verify signature, expiration, claims)
3. Implement `get_current_user` dependency (extract user_id from JWT)
4. Create exception handlers for 401 Unauthorized errors
5. Add CORS middleware with whitelist for frontend URL

**Files to Create/Modify**:
- `backend/src/core/security.py`: JWT verification logic
- `backend/src/api/dependencies.py`: `get_current_user` dependency
- `backend/src/main.py`: CORS configuration, exception handlers

**Acceptance Criteria**:
- FastAPI fetches JWKS from `http://localhost:3000/api/auth/jwks`
- JWT signatures verified using public key from JWKS
- Expired tokens rejected with 401 Unauthorized
- Invalid tokens rejected with 401 Unauthorized
- `get_current_user` dependency extracts `user_id` from JWT `"id"` claim
- CORS allows requests from `http://localhost:3000`

#### 1.4 User Sync Endpoint (Backend)

**Tasks**:
1. Create `UserSync` SQLModel (id, email, created_at)
2. Create Alembic migration for `users_sync` table with foreign key to `users.id`
3. Implement `POST /api/users/sync` endpoint (authenticated)
4. Extract user_id and email from JWT payload
5. Upsert user record (idempotent: `ON CONFLICT DO NOTHING`)

**Files to Create/Modify**:
- `backend/src/models/user.py`: `UserSync` SQLModel
- `backend/src/schemas/user.py`: `UserSyncResponse` Pydantic schema
- `backend/src/api/users.py`: `POST /api/users/sync` endpoint
- `backend/alembic/versions/xxx_create_users_sync.py`: Alembic migration

**Acceptance Criteria**:
- `users_sync` table created with foreign key constraint to `users.id`
- `POST /api/users/sync` requires valid JWT token
- Endpoint extracts `user_id` and `email` from JWT claims
- User record created if not exists (first sync)
- Multiple calls to sync endpoint are idempotent (no duplicate inserts)
- Returns `UserSyncResponse` with id, email, created_at

#### 1.5 Frontend User Sync Integration

**Tasks**:
1. Call `POST /api/users/sync` immediately after Better Auth registration
2. Handle sync errors (display error message, retry logic)
3. Store sync status in local state (prevent duplicate calls)

**Files to Create/Modify**:
- `frontend/src/app/(auth)/register/page.tsx`: Add user sync call after registration
- `frontend/src/lib/api.ts`: `syncUser()` function

**Acceptance Criteria**:
- `POST /api/users/sync` called automatically after successful registration
- Sync errors displayed to user ("Failed to sync user. Please try logging in again.")
- User redirected to `/todos` only after successful sync
- Sync call includes JWT token in Authorization header

**Phase 1 Deliverables**:
- ✅ Better Auth configured with JWT issuance
- ✅ User registration and login pages functional
- ✅ FastAPI JWT verification via JWKS working
- ✅ `POST /api/users/sync` endpoint implemented
- ✅ Frontend calls user sync after registration

**Phase 1 Acceptance Criteria**:
- Users can register and login successfully
- JWT tokens issued with correct claims and 1-hour expiration
- FastAPI verifies JWT signatures and rejects invalid tokens
- User records synced to FastAPI database after registration
- All authentication flows tested end-to-end

---

### Phase 2: Core Todo CRUD Backend

**Objective**: Implement Todo model, database migrations, and protected CRUD API endpoints with user isolation.

**Dependencies**: Phase 1 complete (authentication working)

#### 2.1 Todo Database Model and Migration

**Tasks**:
1. Create `Todo` SQLModel (id, user_id, title, description, completed, created_at, updated_at)
2. Add foreign key constraint: `user_id` → `users_sync.id`
3. Create database indexes on `user_id` and `(user_id, created_at DESC)`
4. Generate Alembic migration for `todos` table
5. Create database trigger for auto-updating `updated_at` timestamp

**Files to Create/Modify**:
- `backend/src/models/todo.py`: `Todo` SQLModel
- `backend/alembic/versions/xxx_create_todos.py`: Alembic migration

**Acceptance Criteria**:
- `todos` table created with all required columns
- Foreign key constraint enforces `user_id` references `users_sync.id`
- Indexes created on `user_id` and `(user_id, created_at DESC)`
- `updated_at` automatically updates on row modification
- Migration applies cleanly with `alembic upgrade head`

#### 2.2 Todo Pydantic Schemas

**Tasks**:
1. Create `TodoCreate` schema (title: required 1-500 chars, description: optional)
2. Create `TodoUpdate` schema (title, description, completed: all optional)
3. Create `TodoResponse` schema (all fields, from_attributes=True)
4. Add Pydantic validators (title not empty/whitespace, length constraints)

**Files to Create/Modify**:
- `backend/src/schemas/todo.py`: `TodoCreate`, `TodoUpdate`, `TodoResponse`

**Acceptance Criteria**:
- `TodoCreate` requires non-empty title (1-500 chars)
- `TodoUpdate` allows partial updates (all fields optional)
- Title validator rejects empty strings and whitespace-only input
- Description limited to 2000 characters
- `TodoResponse` includes all database fields with correct types

#### 2.3 List Todos Endpoint

**Tasks**:
1. Implement `GET /api/todos` endpoint (authenticated)
2. Accept `skip` (default: 0) and `limit` (default: 50, max: 100) query params
3. Filter by `user_id` from JWT (data isolation)
4. Order by `created_at DESC` (newest first)
5. Return `{"data": [todos], "total": count}` format

**Files to Create/Modify**:
- `backend/src/api/todos.py`: `GET /api/todos` endpoint

**Acceptance Criteria**:
- Endpoint requires valid JWT token (401 if missing/invalid)
- Returns only todos owned by authenticated user
- Pagination works correctly (skip=0&limit=50, skip=50&limit=50, etc.)
- Results ordered by creation date (newest first)
- Response includes `total` count for pagination UI
- Empty list returned if user has no todos

#### 2.4 Create Todo Endpoint

**Tasks**:
1. Implement `POST /api/todos` endpoint (authenticated)
2. Accept `TodoCreate` request body
3. Validate title (non-empty, 1-500 chars)
4. Set `user_id` from JWT (prevent user spoofing)
5. Set default `completed=False`
6. Return `TodoResponse` with 201 Created status

**Files to Create/Modify**:
- `backend/src/api/todos.py`: `POST /api/todos` endpoint

**Acceptance Criteria**:
- Endpoint requires valid JWT token
- Title validation rejects empty/whitespace-only input (400 Bad Request)
- Title validation enforces 1-500 character limit
- User cannot create todos for other users (user_id set from JWT)
- Returns created todo with generated UUID and timestamps
- HTTP status 201 Created on success

#### 2.5 Get Todo Endpoint

**Tasks**:
1. Implement `GET /api/todos/{todo_id}` endpoint (authenticated)
2. Validate `todo_id` is valid UUID format (400 if invalid)
3. Filter by `user_id` from JWT AND `todo_id` (data isolation)
4. Return 404 if todo not found OR not owned by user (prevent enumeration)

**Files to Create/Modify**:
- `backend/src/api/todos.py`: `GET /api/todos/{todo_id}` endpoint

**Acceptance Criteria**:
- Endpoint requires valid JWT token
- Invalid UUID format returns 400 Bad Request
- Returns todo if owned by user (200 OK)
- Returns 404 if todo doesn't exist
- Returns 404 if todo exists but owned by another user (not 403)
- Error message does not reveal existence of other users' todos

#### 2.6 Update Todo Endpoint

**Tasks**:
1. Implement `PUT /api/todos/{todo_id}` endpoint (authenticated)
2. Accept `TodoUpdate` request body (partial updates allowed)
3. Filter by `user_id` from JWT AND `todo_id` (data isolation)
4. Update only provided fields (title, description, completed)
5. Auto-update `updated_at` timestamp
6. Return 404 if not found/not owned (prevent enumeration)

**Files to Create/Modify**:
- `backend/src/api/todos.py`: `PUT /api/todos/{todo_id}` endpoint

**Acceptance Criteria**:
- Endpoint requires valid JWT token
- Partial updates work (can update only title, only completed, etc.)
- Title validation enforced if title provided (non-empty, 1-500 chars)
- Cannot update other users' todos (404 response)
- `updated_at` timestamp automatically updated
- Returns updated todo with new `updated_at` value

#### 2.7 Delete Todo Endpoint

**Tasks**:
1. Implement `DELETE /api/todos/{todo_id}` endpoint (authenticated)
2. Filter by `user_id` from JWT AND `todo_id` (data isolation)
3. Delete todo from database
4. Return 204 No Content on success
5. Return 404 if not found/not owned (prevent enumeration)

**Files to Create/Modify**:
- `backend/src/api/todos.py`: `DELETE /api/todos/{todo_id}` endpoint

**Acceptance Criteria**:
- Endpoint requires valid JWT token
- Todo deleted permanently from database
- HTTP status 204 No Content on success (no response body)
- Cannot delete other users' todos (404 response)
- Subsequent GET requests for deleted todo return 404

**Phase 2 Deliverables**:
- ✅ `Todo` database model with migrations applied
- ✅ All CRUD endpoints implemented (`GET`, `POST`, `PUT`, `DELETE`)
- ✅ User isolation enforced on all endpoints
- ✅ Pydantic validation on all request bodies
- ✅ 404 responses for cross-user access attempts

**Phase 2 Acceptance Criteria**:
- All todo endpoints require authentication
- Users can only access their own todos
- Validation errors return 400 with descriptive messages
- Pagination works correctly for list endpoint
- 404 responses do not leak information about other users' data
- All endpoints documented in auto-generated Swagger at `/docs`

---

### Phase 3: Frontend Todo UI

**Objective**: Build React components for todo list display, create/edit forms, and integrate with backend API.

**Dependencies**: Phase 2 complete (backend CRUD endpoints working)

#### 3.1 Protected Routes Middleware

**Tasks**:
1. Create Next.js middleware to check authentication status
2. Redirect unauthenticated users to `/login`
3. Protect `/todos` route (requires valid JWT)
4. Allow access to `/login` and `/register` without auth

**Files to Create/Modify**:
- `frontend/src/middleware.ts`: Auth middleware

**Acceptance Criteria**:
- Unauthenticated users redirected to `/login` when accessing `/todos`
- Authenticated users can access `/todos`
- Login/register pages accessible without authentication
- Middleware checks JWT token validity before allowing access

#### 3.2 API Client Setup

**Tasks**:
1. Create API client wrapper with base URL configuration
2. Add JWT token to all requests (Authorization header)
3. Implement error handling (401 → redirect to login, 500 → display error)
4. Create typed functions for each API endpoint

**Files to Create/Modify**:
- `frontend/src/lib/api.ts`: API client with typed functions
- `frontend/src/lib/types.ts`: TypeScript interfaces for Todo, User, API responses

**Acceptance Criteria**:
- API client includes JWT token in all requests
- 401 responses trigger automatic redirect to login
- Network errors display user-friendly error messages
- TypeScript types match backend Pydantic schemas

#### 3.3 Todo List Component

**Tasks**:
1. Create `TodoList` component to display todos
2. Fetch todos on component mount (`GET /api/todos`)
3. Handle empty state ("No todos yet. Add your first task!")
4. Handle loading state (spinner or skeleton)
5. Handle error state (display error message)
6. Order todos by creation date (newest first)

**Files to Create/Modify**:
- `frontend/src/components/TodoList.tsx`: Todo list component
- `frontend/src/app/(dashboard)/todos/page.tsx`: Todos page

**Acceptance Criteria**:
- Todos fetched and displayed on page load
- Empty state shown when user has no todos
- Loading spinner displayed while fetching
- Error message displayed if API request fails
- Todos ordered by creation date (newest first)

#### 3.4 Todo Item Component

**Tasks**:
1. Create `TodoItem` component for individual todo display
2. Display title, description, completion status
3. Add checkbox for toggling completion (calls `PUT /api/todos/{id}`)
4. Add "Edit" button (opens edit form)
5. Add "Delete" button with confirmation prompt
6. Apply visual styling (strikethrough for completed todos)

**Files to Create/Modify**:
- `frontend/src/components/TodoItem.tsx`: Todo item card component

**Acceptance Criteria**:
- Todo displays title, description, and completion status
- Checkbox toggles completion status (API call + UI update)
- Completed todos have strikethrough text styling
- Edit button opens edit form modal/inline
- Delete button shows confirmation prompt ("Are you sure?")
- Delete confirmation triggers API call and removes from list

#### 3.5 Todo Create Form

**Tasks**:
1. Create `TodoForm` component for creating new todos
2. Add input field for title (required, 1-500 chars)
3. Add textarea for description (optional, 2000 chars max)
4. Implement client-side validation (title non-empty)
5. Call `POST /api/todos` on form submit
6. Add new todo to list on success (optimistic update)
7. Display error message if API call fails

**Files to Create/Modify**:
- `frontend/src/components/TodoForm.tsx`: Create/edit form component
- `frontend/src/app/(dashboard)/todos/page.tsx`: Integrate form in todos page

**Acceptance Criteria**:
- Form requires title (cannot submit empty)
- Title limited to 500 characters (client-side validation)
- Description optional, limited to 2000 characters
- Form submission creates todo via API
- New todo appears at top of list immediately (optimistic update)
- Validation errors displayed inline in form
- API errors displayed to user ("Failed to create todo...")

#### 3.6 Todo Edit Form

**Tasks**:
1. Reuse `TodoForm` component for editing (pass existing todo data)
2. Pre-fill form with existing title and description
3. Call `PUT /api/todos/{id}` on form submit
4. Update todo in list on success (optimistic update)
5. Add "Cancel" button to revert changes

**Files to Create/Modify**:
- `frontend/src/components/TodoForm.tsx`: Add edit mode support
- `frontend/src/components/TodoItem.tsx`: Integrate edit form

**Acceptance Criteria**:
- Edit form pre-filled with existing todo data
- Form submission updates todo via API
- Updated todo reflects changes immediately in UI
- Cancel button discards changes and closes form
- Validation rules same as create form

#### 3.7 Logout Functionality

**Tasks**:
1. Create `Navbar` component with logout button
2. Implement logout handler (remove JWT token, call Better Auth logout)
3. Redirect to `/login` after logout
4. Clear any cached user data

**Files to Create/Modify**:
- `frontend/src/components/Navbar.tsx`: Navigation bar with logout
- `frontend/src/app/layout.tsx`: Include Navbar in root layout

**Acceptance Criteria**:
- Logout button visible when user is authenticated
- Clicking logout removes JWT token from storage
- User redirected to `/login` after logout
- Subsequent API requests fail with 401 (token invalidated)

**Phase 3 Deliverables**:
- ✅ Protected routes middleware implemented
- ✅ API client with typed functions
- ✅ Todo list component with CRUD operations
- ✅ Todo create/edit forms with validation
- ✅ Logout functionality working

**Phase 3 Acceptance Criteria**:
- Unauthenticated users cannot access `/todos` (redirected to login)
- Users can create, read, update, and delete their todos via UI
- All user interactions trigger appropriate API calls
- Loading, error, and empty states handled gracefully
- Optimistic updates provide instant feedback
- All styling uses Tailwind CSS utility classes (no inline styles)

---

### Phase 4: Validation, Security, and Error Handling

**Objective**: Harden application security, add comprehensive validation, and ensure consistent error handling.

**Dependencies**: Phase 3 complete (full CRUD flow working)

#### 4.1 Backend Validation Enhancement

**Tasks**:
1. Add comprehensive Pydantic validators for edge cases
2. Test title validation (empty, whitespace, exceeds 500 chars, unicode)
3. Test description validation (exceeds 2000 chars, special characters)
4. Add custom error messages for validation failures
5. Test UUID validation for todo_id path parameter

**Files to Modify**:
- `backend/src/schemas/todo.py`: Add/enhance validators
- `backend/tests/test_validation.py`: Create validation test suite

**Acceptance Criteria**:
- Empty title rejected (400 Bad Request)
- Whitespace-only title rejected (400 Bad Request)
- Title > 500 chars rejected (400 Bad Request)
- Description > 2000 chars rejected (400 Bad Request)
- Invalid UUID format rejected (400 Bad Request)
- All validation errors return descriptive messages

#### 4.2 Security Audit

**Tasks**:
1. Verify all todo endpoints filter by `user_id` (no hardcoded queries without filter)
2. Test cross-user access attempts (user A tries to access user B's todo)
3. Verify 404 responses don't leak information (same error for not found vs not owned)
4. Test JWT expiration handling (expired tokens rejected)
5. Test CORS configuration (only whitelisted origins allowed)
6. Verify SQL injection protection (test malicious inputs)

**Files to Create/Modify**:
- `backend/tests/test_security.py`: Security test suite

**Acceptance Criteria**:
- 100% of todo queries include `user_id` filter
- Cross-user access attempts return 404 (not 403)
- Expired JWT tokens rejected with 401
- Invalid JWT tokens rejected with 401
- CORS allows only configured origins
- SQL injection attempts fail safely (parameterized queries prevent injection)

#### 4.3 Error Response Standardization

**Tasks**:
1. Create FastAPI exception handler for validation errors (400)
2. Create exception handler for authentication errors (401)
3. Create exception handler for not found errors (404)
4. Create exception handler for internal server errors (500)
5. Ensure all errors return `{"detail": "message", "error_code": "CODE"}` format

**Files to Modify**:
- `backend/src/main.py`: Add exception handlers
- `backend/tests/test_errors.py`: Create error response test suite

**Acceptance Criteria**:
- All validation errors return 400 with `VALIDATION_ERROR` code
- All auth errors return 401 with `UNAUTHORIZED` code
- All not found errors return 404 with `NOT_FOUND` code
- All internal errors return 500 with `INTERNAL_ERROR` code
- Error responses follow consistent `{"detail", "error_code"}` format
- No stack traces exposed to clients (logged server-side only)

#### 4.4 Frontend Error Handling

**Tasks**:
1. Add error boundary component for React errors
2. Display user-friendly error messages for API failures
3. Handle network errors (offline, timeout)
4. Handle 401 errors (redirect to login)
5. Handle 500 errors (display "Service unavailable" message)
6. Add retry logic for transient failures (optional)

**Files to Modify**:
- `frontend/src/lib/api.ts`: Enhanced error handling
- `frontend/src/components/ErrorBoundary.tsx`: React error boundary

**Acceptance Criteria**:
- Network errors display "Check your connection" message
- 401 errors redirect to login automatically
- 500 errors display "Service temporarily unavailable" message
- React errors caught by error boundary (app doesn't crash)
- All error messages are user-friendly (no technical jargon)

#### 4.5 Edge Case Testing

**Tasks**:
1. Test simultaneous registration with same email (database constraint prevents duplicate)
2. Test creating todo with extremely long title (rejected by validation)
3. Test marking non-existent todo as completed (404 response)
4. Test database connection failure (500 response)
5. Test JWT token expired during active session (401 on next request)

**Files to Create/Modify**:
- `backend/tests/test_edge_cases.py`: Edge case test suite
- `frontend/tests/edge-cases.test.ts`: Frontend edge case tests

**Acceptance Criteria**:
- Duplicate email registration fails with "Email already registered" error
- Long titles rejected with validation error
- Non-existent todo operations return 404
- Database failures return 500 with generic error
- Expired tokens trigger automatic redirect to login

**Phase 4 Deliverables**:
- ✅ Comprehensive validation on all inputs
- ✅ Security audit passed (no data leakage, proper isolation)
- ✅ Consistent error response format across all endpoints
- ✅ Frontend error handling for all failure scenarios
- ✅ Edge case testing complete

**Phase 4 Acceptance Criteria**:
- All validation rules enforced (title length, non-empty, etc.)
- 100% of queries filter by user_id (data isolation verified)
- Cross-user access attempts return 404 (information leakage prevented)
- All errors follow standard format with correct HTTP status codes
- Frontend handles all error types gracefully (no unhandled rejections)
- Edge cases tested and handled correctly

---

### Phase 5: Polish, Performance, and Hackathon Readiness

**Objective**: Optimize performance, finalize UI polish, ensure demo readiness, and complete documentation.

**Dependencies**: Phase 4 complete (core functionality secure and validated)

#### 5.1 Performance Optimization

**Tasks**:
1. Add database indexes if missing (`user_id`, `created_at`)
2. Test API response times under load (target: <500ms p95)
3. Optimize frontend bundle size (lazy loading, code splitting)
4. Add React memoization for expensive components (`useMemo`, `React.memo`)
5. Test page load time (target: <2s)

**Files to Modify**:
- `backend/alembic/versions/xxx_add_performance_indexes.py`: Index migration
- `frontend/src/app/(dashboard)/todos/page.tsx`: Add lazy loading

**Acceptance Criteria**:
- All API responses return within 500ms (excluding network latency)
- Todo creation completes in <1s (from click to UI update)
- Page load time <2s (measured with Lighthouse)
- Database queries use indexes efficiently (verify with EXPLAIN)
- Frontend bundle size <1MB (gzipped)

#### 5.2 UI/UX Polish

**Tasks**:
1. Add loading spinners for async operations (create, update, delete)
2. Add success toast notifications ("Todo created!", "Todo deleted!")
3. Improve form UX (disable submit button while loading, clear form after success)
4. Add keyboard shortcuts (Enter to submit form, Esc to cancel)
5. Ensure responsive design (mobile, tablet, desktop)
6. Verify 100% Tailwind CSS usage (no inline styles)

**Files to Modify**:
- `frontend/src/components/*`: Add loading states and animations
- `frontend/tailwind.config.ts`: Customize theme if needed

**Acceptance Criteria**:
- Loading indicators visible during API calls
- Success/error toast notifications appear for user actions
- Forms have smooth UX (button states, clear on success)
- Keyboard shortcuts work (Enter, Esc)
- UI responsive on all screen sizes (tested in DevTools)
- Zero inline styles (verified with code search)

#### 5.3 Demo Flow Validation

**Tasks**:
1. Test complete demo flow (register → create todos → edit → delete → logout → login)
2. Verify JWT expiration handling (wait 1 hour or mock)
3. Test data isolation (create two users, verify they see only their todos)
4. Test error scenarios (wrong password, network failure, etc.)
5. Record demo video (optional but recommended for hackathon)

**Acceptance Criteria**:
- Complete demo flow works end-to-end without errors
- JWT expiration triggers redirect to login
- Two separate users see only their own todos (data isolation verified)
- Error scenarios handled gracefully (user-friendly messages)
- Demo video recorded (optional) showing key features

#### 5.4 Documentation and Swagger Review

**Tasks**:
1. Verify Swagger UI is accessible at `http://localhost:8000/docs`
2. Test all endpoints in Swagger UI (use "Try it out" feature)
3. Ensure all endpoints have docstrings and descriptions
4. Verify request/response schemas are complete and accurate
5. Add README.md with setup instructions (copy from quickstart.md)

**Files to Create/Modify**:
- `README.md`: Project overview and setup guide
- `backend/src/api/*.py`: Add/improve docstrings

**Acceptance Criteria**:
- Swagger UI accessible and displays all endpoints
- All endpoints have clear descriptions and parameter documentation
- Request/response schemas match actual API behavior
- README.md provides clear setup instructions
- Quickstart.md integrated into main documentation

#### 5.5 Final Testing and Cleanup

**Tasks**:
1. Run full test suite (backend: `pytest`, frontend: `npm test`)
2. Fix any failing tests
3. Run linters (backend: `ruff`, frontend: `eslint`)
4. Remove debug code, console.logs, and commented-out code
5. Verify `.env.example` file exists with placeholder values
6. Verify `.gitignore` includes `.env`, `node_modules/`, `__pycache__/`, etc.

**Files to Create/Modify**:
- `.env.example`: Template environment file
- `.gitignore`: Ensure sensitive files excluded

**Acceptance Criteria**:
- All tests passing (0 failures)
- Linters pass with no errors or warnings
- No debug code or console.logs in production code
- `.env.example` provides clear template for setup
- `.gitignore` prevents secrets from being committed

**Phase 5 Deliverables**:
- ✅ Performance optimizations applied
- ✅ UI polished with loading states and animations
- ✅ Demo flow validated end-to-end
- ✅ Swagger documentation complete and accurate
- ✅ All tests passing, codebase clean

**Phase 5 Acceptance Criteria**:
- API response times meet targets (<500ms p95)
- Page load time <2s, todo creation <1s
- Complete demo flow works flawlessly
- Swagger UI fully functional and documented
- All tests passing, linters clean, no debug code

---

## Dependencies and Sequencing

### Blocking Dependencies (Must Complete in Order)

1. **Phase 0 → Phase 1**: Environment setup must complete before authentication can be implemented
2. **Phase 1.1-1.3 → Phase 1.4**: Better Auth and JWT verification must work before user sync endpoint
3. **Phase 1 → Phase 2**: Authentication must be working before implementing protected CRUD endpoints
4. **Phase 2.1-2.2 → Phase 2.3-2.7**: Database model and schemas must exist before implementing endpoints
5. **Phase 2 → Phase 3**: Backend CRUD endpoints must be working before building frontend UI
6. **Phase 3 → Phase 4**: Basic functionality must work before hardening security and validation
7. **Phase 4 → Phase 5**: Security and validation must be solid before final polish

### Parallel Work Opportunities (Can Be Done Simultaneously)

1. **Phase 1.2 (Frontend registration/login) ∥ Phase 1.3 (Backend JWT verification)**: Independent work on frontend and backend
2. **Phase 2.3-2.7 (Backend CRUD endpoints)**: All CRUD endpoints can be implemented in parallel once model/schemas are done
3. **Phase 3.3-3.6 (Frontend components)**: Todo list, item, and form components can be built in parallel once API client is ready
4. **Phase 4.1-4.4 (Validation, security, error handling)**: Backend and frontend hardening can happen in parallel
5. **Phase 5.1-5.4 (Performance, UI polish, docs)**: Most Phase 5 tasks are independent and can be parallelized

### Critical Path (Longest Dependency Chain)

```
Phase 0 (Setup)
  ↓
Phase 1.1 (Better Auth Config)
  ↓
Phase 1.3 (JWT Verification)
  ↓
Phase 1.4 (User Sync Endpoint)
  ↓
Phase 2.1 (Todo Model)
  ↓
Phase 2.3-2.7 (CRUD Endpoints)
  ↓
Phase 3.3 (Todo List Component)
  ↓
Phase 4.2 (Security Audit)
  ↓
Phase 5.3 (Demo Flow Validation)
```

**Estimated Critical Path Duration**: ~8-12 hours for experienced developer (hackathon-appropriate)

---

## Design Decisions to Highlight (ADR Candidates)

The following architectural decisions are significant and should be documented as ADRs if requested:

### 1. Better Auth + JWT vs Custom Authentication

**Decision**: Use Better Auth library for authentication management with JWT tokens

**Significance**:
- **Long-term consequences**: Delegates password hashing, email validation, and session management to well-tested library
- **Alternatives considered**: Custom JWT implementation, NextAuth.js, session-based auth with Redis
- **Cross-cutting impact**: Affects frontend (Better Auth client), backend (JWT verification), and database (user schema)

**ADR Recommended**: Yes - affects security architecture and developer workflow

**Suggestion to User**:
📋 Architectural decision detected: **Using Better Auth + JWT for authentication**
   Document reasoning and tradeoffs? Run `/sp.adr better-auth-jwt-authentication`

---

### 2. JWT-Based Stateless Authentication vs Session-Based Auth

**Decision**: Use stateless JWT authentication verified via JWKS endpoint

**Significance**:
- **Long-term consequences**: No server-side session storage required; scales horizontally easily
- **Alternatives considered**: Session-based auth with Redis, cookie-based sessions, custom token database
- **Cross-cutting impact**: Affects API design (Bearer token headers), deployment (no Redis required), logout strategy (client-side only)

**ADR Recommended**: Yes - fundamental authentication architecture choice

**Suggestion to User**:
📋 Architectural decision detected: **Stateless JWT authentication via JWKS**
   Document reasoning and tradeoffs? Run `/sp.adr jwt-stateless-authentication`

---

### 3. UUID Identifiers for Users and Todos

**Decision**: Use UUID strings for `user_id` and `todo_id` instead of auto-incrementing integers

**Significance**:
- **Long-term consequences**: Prevents ID enumeration attacks, obscures resource count
- **Alternatives considered**: Auto-incrementing integers, Snowflake IDs, ULIDs
- **Cross-cutting impact**: Affects database schema, API contracts, frontend TypeScript types

**ADR Recommended**: Yes - security and data model decision

**Suggestion to User**:
📋 Architectural decision detected: **UUID identifiers for security**
   Document reasoning and tradeoffs? Run `/sp.adr uuid-identifiers-for-security`

---

### 4. Client-Side Logout vs Server-Side Token Revocation

**Decision**: Implement logout by removing JWT from client storage (no server-side token blacklist)

**Significance**:
- **Long-term consequences**: Simpler implementation but tokens valid until expiration
- **Alternatives considered**: Redis token blacklist, short-lived tokens + refresh tokens, database session tracking
- **Cross-cutting impact**: Affects security model (1-hour expiration window), infrastructure (no Redis needed), logout UX

**ADR Recommended**: Yes - security and architecture tradeoff

**Suggestion to User**:
📋 Architectural decision detected: **Client-side logout without token revocation**
   Document reasoning and tradeoffs? Run `/sp.adr client-side-logout-strategy`

---

### 5. 404 Instead of 403 for Cross-User Access Attempts

**Decision**: Return 404 Not Found (not 403 Forbidden) when users try to access other users' resources

**Significance**:
- **Long-term consequences**: Prevents information leakage about resource existence
- **Alternatives considered**: 403 Forbidden (more semantically accurate), generic 400 Bad Request
- **Cross-cutting impact**: Affects API contract, security posture, frontend error handling

**ADR Recommended**: Yes - security and API design decision

**Suggestion to User**:
📋 Architectural decision detected: **404 vs 403 for cross-user access (prevents enumeration)**
   Document reasoning and tradeoffs? Run `/sp.adr 404-vs-403-cross-user-access`

---

### 6. Shared Database Between Better Auth and FastAPI

**Decision**: Use single Neon PostgreSQL database for both Better Auth tables and FastAPI application tables

**Significance**:
- **Long-term consequences**: Simpler infrastructure, native foreign keys, potential schema conflicts
- **Alternatives considered**: Separate databases (Better Auth + FastAPI), embedded SQLite, multi-tenant schema
- **Cross-cutting impact**: Affects deployment, migrations (Better Auth + Alembic), database access patterns

**ADR Recommended**: Yes - infrastructure and data architecture decision

**Suggestion to User**:
📋 Architectural decision detected: **Shared database between Better Auth and FastAPI**
   Document reasoning and tradeoffs? Run `/sp.adr shared-database-architecture`

---

## Validation and Quality Strategy

### How Acceptance Criteria Will Be Validated

#### Security Validation
1. **JWT Expiry**: Create token with past `exp` claim, verify 401 response
2. **User Isolation**: Create two users, attempt cross-user todo access, verify 404
3. **SQL Injection**: Submit malicious SQL in title field, verify parameterized query prevents injection
4. **XSS Prevention**: Submit `<script>alert('xss')</script>` in title, verify React escapes HTML

**Tools**: pytest (backend tests), Playwright (frontend E2E tests), manual Swagger UI testing

#### API Correctness
1. **Status Codes**: Verify 200/201/204/400/401/404/500 returned correctly for each scenario
2. **Error Format**: Validate all errors return `{"detail", "error_code"}` format
3. **Response Schemas**: Use OpenAPI schema validation to verify responses match contracts
4. **Data Integrity**: Verify timestamps, user_id, UUID formats in responses

**Tools**: pytest (API tests), OpenAPI validator, Postman/Insomnia (manual testing)

#### Swagger/OpenAPI Verification
1. **Documentation Completeness**: All endpoints have descriptions and parameter docs
2. **Schema Accuracy**: Request/response schemas match actual API behavior
3. **Interactive Testing**: All endpoints testable via "Try it out" in Swagger UI
4. **JWKS Endpoint**: Verify `/api/auth/jwks` accessible and returns valid JSON

**Tools**: Swagger UI (`http://localhost:8000/docs`), OpenAPI spec validator

#### Hackathon Demo Validation Checklist
- [ ] User can register and login successfully
- [ ] User can create multiple todos
- [ ] User can mark todos as completed (visual feedback)
- [ ] User can edit todo title and description
- [ ] User can delete todos with confirmation
- [ ] User can logout and login again (todos persist)
- [ ] Two users cannot see each other's todos (data isolation)
- [ ] JWT expiration handled gracefully (redirect to login)
- [ ] All API endpoints documented in Swagger
- [ ] No console errors in browser DevTools
- [ ] Application loads in <2 seconds
- [ ] Todo creation completes in <1 second
- [ ] Mobile-responsive (tested on 3 screen sizes)
- [ ] No inline styles (100% Tailwind CSS)

---

## Deliverables

### Phase 1 Deliverables
- ✅ `frontend/src/lib/auth.ts`: Better Auth configuration
- ✅ `frontend/src/app/api/auth/[...auth]/route.ts`: Better Auth API handler
- ✅ `frontend/src/app/(auth)/register/page.tsx`: Registration page
- ✅ `frontend/src/app/(auth)/login/page.tsx`: Login page
- ✅ `backend/src/core/security.py`: JWT verification via JWKS
- ✅ `backend/src/api/dependencies.py`: `get_current_user` dependency
- ✅ `backend/src/models/user.py`: `UserSync` model
- ✅ `backend/src/api/users.py`: `POST /api/users/sync` endpoint

### Phase 2 Deliverables
- ✅ `backend/src/models/todo.py`: `Todo` model
- ✅ `backend/src/schemas/todo.py`: `TodoCreate`, `TodoUpdate`, `TodoResponse`
- ✅ `backend/src/api/todos.py`: All CRUD endpoints (`GET`, `POST`, `PUT`, `DELETE`)
- ✅ `backend/alembic/versions/xxx_create_todos.py`: Todo table migration

### Phase 3 Deliverables
- ✅ `frontend/src/middleware.ts`: Protected routes middleware
- ✅ `frontend/src/lib/api.ts`: Typed API client
- ✅ `frontend/src/components/TodoList.tsx`: Todo list component
- ✅ `frontend/src/components/TodoItem.tsx`: Todo item card
- ✅ `frontend/src/components/TodoForm.tsx`: Create/edit form
- ✅ `frontend/src/components/Navbar.tsx`: Navigation with logout
- ✅ `frontend/src/app/(dashboard)/todos/page.tsx`: Todos dashboard page

### Phase 4 Deliverables
- ✅ `backend/tests/test_validation.py`: Validation test suite
- ✅ `backend/tests/test_security.py`: Security test suite
- ✅ `backend/tests/test_errors.py`: Error response test suite
- ✅ `frontend/src/components/ErrorBoundary.tsx`: React error boundary
- ✅ Enhanced error handling in `frontend/src/lib/api.ts`

### Phase 5 Deliverables
- ✅ `README.md`: Project setup guide
- ✅ `.env.example`: Environment variable template
- ✅ Performance optimizations (indexes, lazy loading)
- ✅ UI polish (loading states, toast notifications)
- ✅ Complete test suite passing (backend + frontend)

---

## Risks and Mitigation

| Risk | Impact | Probability | Mitigation |
|------|--------|------------|-----------|
| **Better Auth JWKS endpoint unavailable** | High (auth breaks) | Low | Cache JWKS public keys; add retry logic with exponential backoff |
| **JWT token theft via XSS** | High (account compromise) | Medium | Use httpOnly cookies instead of localStorage; React default escaping prevents XSS |
| **Database connection failure** | High (service unavailable) | Low | Neon provides connection pooling and automatic failover; return 500 with generic error |
| **User enumeration via timing attacks** | Medium (privacy leak) | Low | Better Auth uses constant-time password comparison; enforce 404 for cross-user access |
| **Expired token during long session** | Medium (UX interruption) | High | Frontend detects 401 responses, redirects to login with return URL |
| **Duplicate email registration** | Low (UX issue) | Medium | Database unique constraint prevents duplicates; Better Auth returns clear error message |
| **Large pagination queries (skip=10000)** | Medium (performance) | Low | Enforce max limit=100; add query timeout; consider cursor-based pagination for v2 |
| **Slow API responses** | Medium (UX degradation) | Low | Add database indexes; use connection pooling; monitor with Neon query logs |

---

## Implementation Timeline (Hackathon Context)

**Total Estimated Time**: 8-12 hours for experienced full-stack developer

| Phase | Tasks | Estimated Time | Priority |
|-------|-------|----------------|----------|
| Phase 0 | Environment setup, dependencies | 1 hour | P0 (Blocker) |
| Phase 1 | Authentication foundation | 2-3 hours | P1 (Core) |
| Phase 2 | Backend CRUD endpoints | 2-3 hours | P1 (Core) |
| Phase 3 | Frontend UI components | 2-3 hours | P1 (Core) |
| Phase 4 | Validation, security, errors | 1-2 hours | P2 (Important) |
| Phase 5 | Polish, performance, docs | 1-2 hours | P3 (Nice-to-have) |

**Hackathon Strategy**:
- Focus on P1 tasks first (Phases 0-3): Complete demo-able CRUD flow
- Implement P2 tasks if time allows (Phase 4): Security and validation hardening
- P3 tasks are polish (Phase 5): Only if extra time available

**Minimum Viable Demo** (6 hours):
- Phase 0: Setup (1 hour)
- Phase 1: Auth (2 hours)
- Phase 2: Backend CRUD (2 hours)
- Phase 3: Frontend UI (1 hour - basic list + create only)

**Complete Demo** (12 hours):
- All phases including polish and performance optimization

---

## Next Steps

1. ✅ **Planning Complete**: This plan is ready for task breakdown
2. ⏭️ **Run `/sp.tasks`**: Generate atomic, testable tasks from this plan
3. ⏭️ **Consider ADRs**: If any design decisions need documentation, run `/sp.adr <decision-title>`
4. ⏭️ **Begin Implementation**: Follow TDD workflow (write test → implement → refactor)
5. ⏭️ **Commit Often**: Small, atomic commits with descriptive messages

**Branch**: `001-todo-app`
**Ready for**: Task generation (`/sp.tasks` command)

---

## Appendix: Technology Stack Details

### Frontend Stack
- **Next.js 14**: App Router, Server Components, Server Actions
- **TypeScript 5+**: Type safety, interfaces for API responses
- **Tailwind CSS 3+**: Utility-first styling, no inline styles
- **Better Auth**: Authentication library with JWT support
- **React 18**: Component-based UI, hooks (useState, useEffect, etc.)

### Backend Stack
- **FastAPI**: Async Python web framework, auto-generates OpenAPI docs
- **SQLModel**: ORM combining SQLAlchemy + Pydantic for type safety
- **Alembic**: Database migration tool
- **Pydantic**: Data validation and serialization
- **pyjwt**: JWT token verification
- **httpx**: HTTP client for fetching JWKS from Better Auth
- **uvicorn**: ASGI server for FastAPI

### Database
- **Neon Serverless PostgreSQL**: Cloud-hosted PostgreSQL with auto-scaling
- **PostgreSQL 15+**: Supports UUID types, foreign keys, triggers

### Testing
- **pytest**: Backend testing framework
- **Jest**: Frontend unit testing
- **React Testing Library**: Component testing
- **Playwright** (optional): End-to-end testing

### Development Tools
- **uv**: Fast Python package manager
- **npm**: Frontend package manager
- **ESLint**: Frontend linting
- **Ruff**: Backend linting
- **Prettier**: Code formatting

---

**Plan Status**: ✅ Complete and ready for task breakdown
**Next Command**: `/sp.tasks` to generate atomic implementation tasks
