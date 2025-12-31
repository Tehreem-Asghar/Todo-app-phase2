# Tasks: Phase II Full-Stack Todo Web Application

**Input**: Design documents from `/specs/001-todo-app/`
**Prerequisites**: plan.md (complete), spec.md (complete), research.md, data-model.md, contracts/

**Tests**: Tests are NOT explicitly requested in the specification. Focus on implementation tasks and manual validation as described in spec.md acceptance scenarios.

**Organization**: Tasks are grouped by user story (US1-US5) to enable independent implementation and testing of each story according to priority (P1 → P2 → P3).

## Format: `- [ ] [ID] [P?] [Story?] Description`

- **- [ ]**: Markdown checkbox (REQUIRED for all tasks)
- **[ID]**: Task ID (T001, T002, T003... in execution order)
- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: User story label (US1, US2, US3, US4, US5) - ONLY for user story phase tasks
- **Description**: Clear action with exact file path

## Path Conventions

This is a web application with frontend and backend:
- **Backend**: `backend/src/` (FastAPI, SQLModel, Alembic)
- **Frontend**: `frontend/src/` (Next.js 14 App Router, TypeScript, Tailwind CSS)
- **Database**: Neon Serverless PostgreSQL (provisioned externally)
- **Environment**: `.env` at repository root

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization, directory structure, and dependency installation

- [ ] T001 Provision Neon PostgreSQL database and obtain connection string at https://neon.tech
- [ ] T002 Create `.env` file at repository root with DATABASE_URL, NEXTAUTH_URL, NEXTAUTH_SECRET, FASTAPI_URL
- [ ] T003 Create `.gitignore` file at repository root (ignore .env, node_modules/, __pycache__/, .venv/, .next/, alembic/versions/*.pyc)
- [ ] T004 [P] Initialize Next.js frontend with TypeScript and Tailwind at frontend/
- [ ] T005 [P] Initialize FastAPI backend with uv at backend/
- [ ] T006 Install Better Auth in frontend/ (npm install better-auth)
- [ ] T007 Install FastAPI dependencies in backend/ (fastapi, sqlmodel, alembic, pyjwt[crypto], httpx, cryptography, uvicorn)
- [ ] T008 Create frontend directory structure: src/app/, src/components/, src/lib/
- [ ] T009 Create backend directory structure: src/api/, src/models/, src/schemas/, src/core/
- [ ] T010 [P] Configure Tailwind CSS in frontend/tailwind.config.ts
- [ ] T011 [P] Configure TypeScript in frontend/tsconfig.json (strict mode, path aliases)
- [ ] T012 [P] Initialize Alembic for database migrations in backend/ (alembic init alembic)
- [ ] T013 Configure Alembic env.py to use SQLModel metadata and DATABASE_URL from environment

**Checkpoint**: Project structure created, dependencies installed

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core authentication infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

### Backend Core Configuration

- [ ] T014 Create Settings class in backend/src/core/config.py (load DATABASE_URL, CORS_ORIGINS from env)
- [ ] T015 Create database connection helper in backend/src/api/dependencies.py (get_db session generator)
- [ ] T016 Initialize FastAPI app in backend/src/main.py with CORS middleware (whitelist http://localhost:3000)
- [ ] T017 Add health check endpoint GET /api/health in backend/src/main.py (returns {"status": "ok"})

### Better Auth Configuration (Frontend)

- [ ] T018 Configure Better Auth in frontend/src/lib/auth.ts (PostgreSQL adapter, JWT plugin, 1-hour expiration, RS256 algorithm)
- [ ] T019 Create Better Auth API handler in frontend/src/app/api/auth/[...auth]/route.ts
- [ ] T020 Run Better Auth migrations to create users table in Neon database (npx better-auth migrate)

### JWT Verification (Backend)

- [ ] T021 Implement JWKS fetching logic in backend/src/core/security.py (fetch from http://localhost:3000/api/auth/jwks)
- [ ] T022 Implement JWT verification function in backend/src/core/security.py (verify signature, expiration, extract claims)
- [ ] T023 Create get_current_user dependency in backend/src/api/dependencies.py (extract user_id from JWT "id" claim)
- [ ] T024 Add 401 exception handler in backend/src/main.py (return {"detail": "Invalid or expired token", "error_code": "UNAUTHORIZED"})

### User Sync Infrastructure (Backend)

- [ ] T025 Create UserSync SQLModel in backend/src/models/user.py (id: UUID FK to users.id, email: str, created_at: datetime)
- [ ] T026 Create UserSyncResponse schema in backend/src/schemas/user.py (id, email, created_at)
- [ ] T027 Generate Alembic migration for users_sync table in backend/alembic/versions/ (foreign key to users.id, ON DELETE CASCADE)
- [ ] T028 Apply Alembic migration to create users_sync table (alembic upgrade head)
- [ ] T029 Implement POST /api/users/sync endpoint in backend/src/api/users.py (authenticated, upsert user from JWT payload)
- [ ] T030 Register users router in backend/src/main.py (app.include_router with /api prefix)

**Checkpoint**: Foundation ready - authentication working, user sync functional, all user stories can now begin in parallel

---

## Phase 3: User Story 1 - User Registration and Login (Priority: P1) 🎯 MVP

**Goal**: Enable users to register with email/password and login to receive JWT tokens for authenticated access

**Independent Test**: Create new account with email/password, logout, login again. Verify JWT token issued and stored. Access protected route.

### Frontend Registration Page

- [ ] T031 [P] [US1] Create registration page in frontend/src/app/(auth)/register/page.tsx (email input, password input min 8 chars, submit button)
- [ ] T032 [P] [US1] Implement registration form validation in frontend/src/app/(auth)/register/page.tsx (email format, password min 8 chars)
- [ ] T033 [US1] Implement registration handler in frontend/src/app/(auth)/register/page.tsx (call Better Auth signup, handle errors)
- [ ] T034 [US1] Call POST /api/users/sync after successful registration in frontend/src/app/(auth)/register/page.tsx
- [ ] T035 [US1] Redirect to /todos after successful registration and sync in frontend/src/app/(auth)/register/page.tsx

### Frontend Login Page

- [ ] T036 [P] [US1] Create login page in frontend/src/app/(auth)/login/page.tsx (email input, password input, submit button)
- [ ] T037 [US1] Implement login handler in frontend/src/app/(auth)/login/page.tsx (call Better Auth signin, handle errors)
- [ ] T038 [US1] Display error message "Invalid email or password" for incorrect credentials in frontend/src/app/(auth)/login/page.tsx
- [ ] T039 [US1] Redirect to /todos after successful login in frontend/src/app/(auth)/login/page.tsx

### Frontend Logout Functionality

- [ ] T040 [US1] Create Navbar component in frontend/src/components/Navbar.tsx with logout button (Tailwind styling)
- [ ] T041 [US1] Implement logout handler in frontend/src/components/Navbar.tsx (call Better Auth signout, remove JWT token)
- [ ] T042 [US1] Redirect to /login after logout in frontend/src/components/Navbar.tsx
- [ ] T043 [US1] Include Navbar in root layout at frontend/src/app/layout.tsx

### Frontend Protected Routes

- [ ] T044 [US1] Create auth middleware in frontend/src/middleware.ts (check JWT token, redirect unauthenticated to /login)
- [ ] T045 [US1] Protect /todos route in frontend/src/middleware.ts (require valid JWT)
- [ ] T046 [US1] Create empty todos dashboard page in frontend/src/app/(dashboard)/todos/page.tsx (placeholder for US2)

### Error Handling

- [ ] T047 [US1] Display "Email already registered" error when duplicate email in frontend/src/app/(auth)/register/page.tsx
- [ ] T048 [US1] Handle sync errors with message "Failed to sync user. Please try logging in again." in frontend/src/app/(auth)/register/page.tsx

**Checkpoint**: User Story 1 complete - Users can register, login, logout. JWT authentication working end-to-end.

**Acceptance Validation** (from spec.md):
- ✅ User can register with valid email/password (min 8 chars)
- ✅ User receives "Email already registered" error for duplicate emails
- ✅ User can login with correct credentials and receives JWT token
- ✅ User sees "Invalid email or password" for incorrect credentials
- ✅ User can logout and JWT token is invalidated
- ✅ User redirected to /todos dashboard after successful login/registration

---

## Phase 4: User Story 2 - Create and View Personal Todos (Priority: P1) 🎯 MVP

**Goal**: Enable authenticated users to create new todos and view their complete list of todos with data isolation

**Independent Test**: Login, create multiple todos, verify they appear in list. Create second user, verify they see only their own todos (data isolation).

### Backend Todo Model and Migration

- [ ] T049 [P] [US2] Create Todo SQLModel in backend/src/models/todo.py (id: UUID, user_id: UUID FK, title: str max 500, description: str optional, completed: bool, created_at, updated_at)
- [ ] T050 [P] [US2] Create TodoCreate schema in backend/src/schemas/todo.py (title: 1-500 chars required, description: optional max 2000 chars)
- [ ] T051 [P] [US2] Create TodoResponse schema in backend/src/schemas/todo.py (all fields, from_attributes=True)
- [ ] T052 [P] [US2] Add Pydantic validator to TodoCreate in backend/src/schemas/todo.py (title not empty/whitespace only)
- [ ] T053 [US2] Generate Alembic migration for todos table in backend/alembic/versions/ (foreign key to users_sync.id, indexes on user_id and (user_id, created_at DESC))
- [ ] T054 [US2] Apply Alembic migration to create todos table and indexes (alembic upgrade head)

### Backend List Todos Endpoint

- [ ] T055 [US2] Implement GET /api/todos endpoint in backend/src/api/todos.py (authenticated, filter by user_id from JWT)
- [ ] T056 [US2] Add pagination params skip and limit to GET /api/todos in backend/src/api/todos.py (default limit=50, max=100)
- [ ] T057 [US2] Return {"data": [todos], "total": count} format in GET /api/todos in backend/src/api/todos.py
- [ ] T058 [US2] Order todos by created_at DESC in GET /api/todos in backend/src/api/todos.py (newest first)
- [ ] T059 [US2] Register todos router in backend/src/main.py (app.include_router with /api prefix)

### Backend Create Todo Endpoint

- [ ] T060 [US2] Implement POST /api/todos endpoint in backend/src/api/todos.py (authenticated, accept TodoCreate body)
- [ ] T061 [US2] Set user_id from JWT in POST /api/todos in backend/src/api/todos.py (prevent user spoofing)
- [ ] T062 [US2] Set completed=False default in POST /api/todos in backend/src/api/todos.py
- [ ] T063 [US2] Return TodoResponse with 201 Created status in POST /api/todos in backend/src/api/todos.py
- [ ] T064 [US2] Validate title non-empty and 1-500 chars in POST /api/todos in backend/src/api/todos.py (return 400 on failure)

### Frontend API Client

- [ ] T065 [P] [US2] Create API client wrapper in frontend/src/lib/api.ts (base URL from env, add JWT to all requests)
- [ ] T066 [P] [US2] Create TypeScript Todo interface in frontend/src/lib/types.ts (matches TodoResponse schema)
- [ ] T067 [US2] Implement getTodos() function in frontend/src/lib/api.ts (GET /api/todos with pagination)
- [ ] T068 [US2] Implement createTodo() function in frontend/src/lib/api.ts (POST /api/todos with title, description)
- [ ] T069 [US2] Add 401 error handler in frontend/src/lib/api.ts (redirect to /login on unauthorized)

### Frontend Todo List Display

- [ ] T070 [US2] Implement todo list fetch on mount in frontend/src/app/(dashboard)/todos/page.tsx (call getTodos() via useEffect)
- [ ] T071 [US2] Display loading spinner while fetching todos in frontend/src/app/(dashboard)/todos/page.tsx (Tailwind spinner)
- [ ] T072 [US2] Display "No todos yet. Add your first task!" for empty list in frontend/src/app/(dashboard)/todos/page.tsx
- [ ] T073 [US2] Display error message if getTodos() fails in frontend/src/app/(dashboard)/todos/page.tsx
- [ ] T074 [US2] Create TodoList component in frontend/src/components/TodoList.tsx (map over todos array, Tailwind styling)
- [ ] T075 [US2] Render TodoList component in frontend/src/app/(dashboard)/todos/page.tsx

### Frontend Todo Create Form

- [ ] T076 [US2] Create TodoForm component in frontend/src/components/TodoForm.tsx (title input, description textarea, submit button)
- [ ] T077 [US2] Add client-side validation to TodoForm in frontend/src/components/TodoForm.tsx (title required, max 500 chars)
- [ ] T078 [US2] Implement form submit handler in frontend/src/components/TodoForm.tsx (call createTodo(), optimistic update)
- [ ] T079 [US2] Display validation errors inline in TodoForm in frontend/src/components/TodoForm.tsx (Tailwind error styling)
- [ ] T080 [US2] Clear form after successful todo creation in frontend/src/components/TodoForm.tsx
- [ ] T081 [US2] Render TodoForm component in frontend/src/app/(dashboard)/todos/page.tsx

**Checkpoint**: User Story 2 complete - Users can create and view their own todos. Data isolation enforced (users only see their own todos).

**Acceptance Validation** (from spec.md):
- ✅ User can create todo with title, appears at top of list with status "incomplete"
- ✅ User sees only their own todos (data isolation tested with two users)
- ✅ Unauthenticated visitor redirected to /login when accessing /todos
- ✅ User with no todos sees "No todos yet. Add your first task!" message
- ✅ User sees validation error "Todo title cannot be empty" for empty title

---

## Phase 5: User Story 3 - Update Todo Status (Priority: P2)

**Goal**: Enable authenticated users to mark todos as completed or incomplete to track progress

**Independent Test**: Login, create todo, mark as completed (verify strikethrough styling), mark as incomplete, refresh page (verify status persists).

### Backend Update Todo Endpoint

- [ ] T082 [P] [US3] Create TodoUpdate schema in backend/src/schemas/todo.py (title: optional 1-500 chars, description: optional, completed: optional bool)
- [ ] T083 [P] [US3] Add Pydantic validator to TodoUpdate in backend/src/schemas/todo.py (title not empty/whitespace if provided)
- [ ] T084 [US3] Implement PUT /api/todos/{todo_id} endpoint in backend/src/api/todos.py (authenticated, accept TodoUpdate body)
- [ ] T085 [US3] Filter by user_id from JWT AND todo_id in PUT /api/todos/{todo_id} in backend/src/api/todos.py (data isolation)
- [ ] T086 [US3] Return 404 if todo not found or not owned by user in PUT /api/todos/{todo_id} in backend/src/api/todos.py
- [ ] T087 [US3] Update only provided fields in PUT /api/todos/{todo_id} in backend/src/api/todos.py (partial updates allowed)
- [ ] T088 [US3] Auto-update updated_at timestamp in PUT /api/todos/{todo_id} in backend/src/api/todos.py
- [ ] T089 [US3] Return updated TodoResponse in PUT /api/todos/{todo_id} in backend/src/api/todos.py

### Frontend Todo Item Component

- [ ] T090 [US3] Create TodoItem component in frontend/src/components/TodoItem.tsx (display title, description, completed status)
- [ ] T091 [US3] Add checkbox for toggling completion in TodoItem in frontend/src/components/TodoItem.tsx (Tailwind styling)
- [ ] T092 [US3] Implement updateTodo() function in frontend/src/lib/api.ts (PUT /api/todos/{id} with partial update)
- [ ] T093 [US3] Implement checkbox toggle handler in TodoItem in frontend/src/components/TodoItem.tsx (call updateTodo with completed field)
- [ ] T094 [US3] Apply strikethrough styling to completed todos in TodoItem in frontend/src/components/TodoItem.tsx (Tailwind line-through)
- [ ] T095 [US3] Apply optimistic update for completion toggle in TodoItem in frontend/src/components/TodoItem.tsx (instant UI feedback)
- [ ] T096 [US3] Update TodoList component in frontend/src/components/TodoList.tsx to render TodoItem for each todo

**Checkpoint**: User Story 3 complete - Users can toggle todo completion status. Status persists after page refresh.

**Acceptance Validation** (from spec.md):
- ✅ User can click checkbox to mark todo as completed (strikethrough styling applied)
- ✅ User can click checkbox to mark completed todo as incomplete (normal styling restored)
- ✅ Completion status persists after page refresh
- ✅ Unauthenticated visitor receives 401 Unauthorized when attempting to update todo via API

---

## Phase 6: User Story 5 - Delete Todos (Priority: P2)

**Goal**: Enable authenticated users to permanently delete todos to keep list clean and manageable

**Independent Test**: Login, create multiple todos, delete specific ones with confirmation, verify they no longer appear in list.

### Backend Delete Todo Endpoint

- [ ] T097 [US5] Implement DELETE /api/todos/{todo_id} endpoint in backend/src/api/todos.py (authenticated, filter by user_id from JWT)
- [ ] T098 [US5] Return 404 if todo not found or not owned by user in DELETE /api/todos/{todo_id} in backend/src/api/todos.py
- [ ] T099 [US5] Delete todo from database in DELETE /api/todos/{todo_id} in backend/src/api/todos.py
- [ ] T100 [US5] Return 204 No Content on success in DELETE /api/todos/{todo_id} in backend/src/api/todos.py

### Frontend Delete Functionality

- [ ] T101 [US5] Implement deleteTodo() function in frontend/src/lib/api.ts (DELETE /api/todos/{id})
- [ ] T102 [US5] Add delete button to TodoItem in frontend/src/components/TodoItem.tsx (Tailwind danger button styling)
- [ ] T103 [US5] Implement delete confirmation dialog in TodoItem in frontend/src/components/TodoItem.tsx ("Are you sure you want to delete this todo?")
- [ ] T104 [US5] Implement delete handler in TodoItem in frontend/src/components/TodoItem.tsx (call deleteTodo on confirm)
- [ ] T105 [US5] Remove todo from list on successful deletion in TodoItem in frontend/src/components/TodoItem.tsx (optimistic update)

**Checkpoint**: User Story 5 complete - Users can delete todos with confirmation. Deleted todos removed from list permanently.

**Acceptance Validation** (from spec.md):
- ✅ User can click delete button to remove todo from list
- ✅ User sees confirmation prompt "Are you sure you want to delete this todo?" before deletion
- ✅ Todo permanently removed after confirmation (cannot be recovered)
- ✅ Unauthenticated visitor receives 401 Unauthorized when attempting to delete via API
- ✅ User attempting to delete another user's todo receives 404 Not Found (data isolation enforced)

---

## Phase 7: User Story 4 - Edit Todo Details (Priority: P3)

**Goal**: Enable authenticated users to edit todo title and description to correct mistakes or update information

**Independent Test**: Login, create todo, click edit, change title, save, verify change persists. Test cancel button preserves original content.

### Backend Get Todo Endpoint

- [ ] T106 [US4] Implement GET /api/todos/{todo_id} endpoint in backend/src/api/todos.py (authenticated, filter by user_id from JWT)
- [ ] T107 [US4] Validate todo_id is valid UUID format in GET /api/todos/{todo_id} in backend/src/api/todos.py (return 400 if invalid)
- [ ] T108 [US4] Return 404 if todo not found or not owned by user in GET /api/todos/{todo_id} in backend/src/api/todos.py
- [ ] T109 [US4] Return TodoResponse in GET /api/todos/{todo_id} in backend/src/api/todos.py

### Frontend Edit Functionality

- [ ] T110 [US4] Add edit mode state to TodoItem in frontend/src/components/TodoItem.tsx (useState for isEditing)
- [ ] T111 [US4] Add edit button to TodoItem in frontend/src/components/TodoItem.tsx (Tailwind styling)
- [ ] T112 [US4] Show inline edit form when isEditing=true in TodoItem in frontend/src/components/TodoItem.tsx (title input, description textarea)
- [ ] T113 [US4] Pre-fill edit form with existing todo data in TodoItem in frontend/src/components/TodoItem.tsx
- [ ] T114 [US4] Implement save handler in TodoItem in frontend/src/components/TodoItem.tsx (call updateTodo with title and description)
- [ ] T115 [US4] Implement cancel button in TodoItem in frontend/src/components/TodoItem.tsx (reset form, exit edit mode)
- [ ] T116 [US4] Add validation for empty title in edit form in TodoItem in frontend/src/components/TodoItem.tsx
- [ ] T117 [US4] Apply optimistic update on save in TodoItem in frontend/src/components/TodoItem.tsx (instant UI feedback)

**Checkpoint**: User Story 4 complete - Users can edit todo title and description. Changes persist after save. Cancel preserves original content.

**Acceptance Validation** (from spec.md):
- ✅ User can click edit button, modify title, save, and see updated title displayed
- ✅ User sees validation error "Todo title cannot be empty" when attempting to save empty title
- ✅ User can click cancel button to discard changes and preserve original todo content

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Error handling, validation, performance, and documentation improvements across all user stories

### Error Response Standardization (Backend)

- [ ] T118 [P] Add validation error exception handler in backend/src/main.py (return 400 with {"detail": "...", "error_code": "VALIDATION_ERROR"})
- [ ] T119 [P] Add not found error exception handler in backend/src/main.py (return 404 with {"detail": "...", "error_code": "NOT_FOUND"})
- [ ] T120 [P] Add internal server error exception handler in backend/src/main.py (return 500 with {"detail": "Service temporarily unavailable", "error_code": "INTERNAL_ERROR"})

### Frontend Error Handling

- [ ] T121 [P] Create ErrorBoundary component in frontend/src/components/ErrorBoundary.tsx (catch React errors, display fallback UI)
- [ ] T122 [P] Add network error handling in frontend/src/lib/api.ts (display "Check your connection" message)
- [ ] T123 [P] Add 500 error handling in frontend/src/lib/api.ts (display "Service temporarily unavailable" message)
- [ ] T124 Wrap app with ErrorBoundary in frontend/src/app/layout.tsx

### Validation Enhancements

- [ ] T125 [P] Test title validation with edge cases in backend/src/schemas/todo.py (empty, whitespace, 500+ chars, unicode)
- [ ] T126 [P] Test description validation with edge cases in backend/src/schemas/todo.py (2000+ chars, special characters)
- [ ] T127 [P] Add custom validation error messages in backend/src/schemas/todo.py (user-friendly error text)

### Performance Optimization

- [ ] T128 [P] Verify database indexes exist in backend/alembic/versions/ (user_id, (user_id, created_at DESC) on todos table)
- [ ] T129 [P] Add React.memo to TodoItem in frontend/src/components/TodoItem.tsx (prevent unnecessary re-renders)
- [ ] T130 [P] Add React.memo to TodoList in frontend/src/components/TodoList.tsx (prevent unnecessary re-renders)

### Documentation

- [ ] T131 [P] Add docstrings to all FastAPI endpoints in backend/src/api/*.py (describe purpose, params, return values)
- [ ] T132 [P] Verify Swagger UI accessible at http://localhost:8000/docs (test "Try it out" for all endpoints)
- [ ] T133 Create README.md at repository root with setup instructions (copy key sections from specs/001-todo-app/quickstart.md)
- [ ] T134 Create .env.example at repository root with placeholder values (DATABASE_URL, NEXTAUTH_URL, NEXTAUTH_SECRET, FASTAPI_URL)

### Final Validation

- [ ] T135 Run complete demo flow per quickstart.md (register → create todos → edit → delete → logout → login)
- [ ] T136 Test data isolation with two users (verify each user sees only their own todos)
- [ ] T137 Verify JWT expiration handling (mock expired token, verify 401 response and redirect to login)
- [ ] T138 Test all edge cases from spec.md (expired JWT, duplicate email, invalid todo ID, empty title, etc.)
- [ ] T139 Verify all functional requirements from spec.md (FR-001 through FR-031)

**Checkpoint**: Application ready for hackathon demo. All user stories functional, errors handled gracefully, performance optimized.

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phases 3-7)**: All depend on Foundational phase completion
  - **User Story 1 (P1)**: Can start after Foundational - Independent
  - **User Story 2 (P1)**: Can start after Foundational - Independent (technically depends on US1 for auth, but auth is in Foundation)
  - **User Story 3 (P2)**: Depends on US2 (needs todos to exist to update them)
  - **User Story 5 (P2)**: Depends on US2 (needs todos to exist to delete them)
  - **User Story 4 (P3)**: Depends on US2 and US3 (needs todos and update endpoint)
- **Polish (Phase 8)**: Depends on desired user stories being complete

### User Story Dependencies

- **US1 (Registration & Login)**: No dependencies after Foundation - Can start immediately
- **US2 (Create & View Todos)**: No dependencies after Foundation - Can start immediately (auth from Foundation)
- **US3 (Update Status)**: Depends on US2 (requires todos table and list view)
- **US5 (Delete Todos)**: Depends on US2 (requires todos table and list view)
- **US4 (Edit Details)**: Depends on US2 and US3 (requires todos table, update endpoint)

### Recommended Execution Order (Priority-Based)

1. **Phase 1**: Setup (T001-T013)
2. **Phase 2**: Foundational (T014-T030) - MUST COMPLETE before user stories
3. **Phase 3**: User Story 1 - Registration & Login (T031-T048) - P1, MVP Critical
4. **Phase 4**: User Story 2 - Create & View Todos (T049-T081) - P1, MVP Critical
5. **Phase 5**: User Story 3 - Update Status (T082-T096) - P2, High Value
6. **Phase 6**: User Story 5 - Delete Todos (T097-T105) - P2, High Value
7. **Phase 7**: User Story 4 - Edit Details (T106-T117) - P3, Nice-to-Have
8. **Phase 8**: Polish & Cross-Cutting (T118-T139) - Final touches

### Parallel Opportunities

**Within Setup Phase (Phase 1)**:
- T004, T005 (frontend and backend init) can run in parallel
- T006, T007 (dependency installation) can run in parallel after init
- T008, T009 (directory creation) can run in parallel
- T010, T011, T012 (config files) can run in parallel

**Within Foundational Phase (Phase 2)**:
- T014, T015, T016, T017 (backend core) can run in parallel
- T018, T019 (Better Auth config) can run in parallel
- T021, T022, T023, T024 (JWT verification) can run in parallel after T018-T020
- T025, T026 (models and schemas) can run in parallel

**Within User Story Phases**:
- Models and schemas marked [P] can run in parallel
- Frontend and backend tasks can run in parallel when on different files
- Example US2: T049, T050, T051, T052 (models and schemas) can run in parallel

**Between User Stories** (after Foundational complete):
- US1 and US2 can start in parallel (if team capacity allows)
- US3 and US5 can work in parallel after US2 complete

---

## Parallel Example: User Story 2 (Create and View Todos)

```bash
# After Foundation complete, launch models and schemas in parallel:
Task T049: "Create Todo SQLModel in backend/src/models/todo.py"
Task T050: "Create TodoCreate schema in backend/src/schemas/todo.py"
Task T051: "Create TodoResponse schema in backend/src/schemas/todo.py"
Task T052: "Add Pydantic validator to TodoCreate in backend/src/schemas/todo.py"

# After schemas ready, launch API client functions in parallel:
Task T065: "Create API client wrapper in frontend/src/lib/api.ts"
Task T066: "Create TypeScript Todo interface in frontend/src/lib/types.ts"

# Frontend and backend can work in parallel:
Backend: T055-T064 (list and create endpoints)
Frontend: T070-T081 (list display and create form)
```

---

## Implementation Strategy

### MVP First (Minimum Viable Product)

**Goal**: Deliver working authentication and basic todo functionality as fast as possible

**MVP Scope** (P1 priorities only):
1. Phase 1: Setup (T001-T013)
2. Phase 2: Foundational (T014-T030)
3. Phase 3: User Story 1 - Registration & Login (T031-T048)
4. Phase 4: User Story 2 - Create & View Todos (T049-T081)
5. **STOP and VALIDATE**: Test end-to-end (register → login → create todos → logout)
6. Deploy/demo MVP

**MVP delivers**: Authenticated users can manage their personal todo lists with data isolation

### Incremental Delivery (Full Product)

1. **Foundation**: Complete Setup + Foundational (T001-T030) → Authentication infrastructure ready
2. **MVP**: Add US1 + US2 (T031-T081) → Users can register and create todos → **Demo checkpoint!**
3. **Enhancement 1**: Add US3 (T082-T096) → Users can mark todos complete → **Demo checkpoint!**
4. **Enhancement 2**: Add US5 (T097-T105) → Users can delete todos → **Demo checkpoint!**
5. **Enhancement 3**: Add US4 (T106-T117) → Users can edit todos → **Demo checkpoint!**
6. **Polish**: Add Phase 8 (T118-T139) → Production-ready with error handling → **Final demo!**

Each checkpoint delivers independently testable value without breaking previous functionality.

### Parallel Team Strategy (Multiple Developers)

**Team completes Setup + Foundational together** (T001-T030):
- Establishes shared infrastructure
- Prevents merge conflicts in core files

**Once Foundational complete, split work**:
- **Developer A**: User Story 1 (T031-T048) - Authentication
- **Developer B**: User Story 2 (T049-T081) - Todo CRUD (depends on A completing US1)
- **Developer C**: Polish Phase 8 (T118-T134) - Error handling and docs (can start early)

**After MVP (US1+US2) complete**:
- **Developer A**: User Story 3 (T082-T096) - Update status
- **Developer B**: User Story 5 (T097-T105) - Delete todos
- **Developer C**: User Story 4 (T106-T117) - Edit details

Stories complete and integrate independently with minimal conflicts.

---

## Notes

- **[P] marker**: Tasks marked [P] can run in parallel (different files, no dependencies on incomplete tasks)
- **[Story] label**: Maps task to specific user story (US1-US5) for traceability and independent testing
- **Independent testing**: Each user story should be fully testable on its own after its phase completes
- **Data isolation**: All todo endpoints filter by user_id from JWT (verified in US2, US3, US4, US5)
- **Error handling**: Consistent format {"detail": "...", "error_code": "..."} enforced in Phase 8
- **Validation**: Pydantic validates all inputs (title length, non-empty, email format, etc.)
- **Security**: JWT authentication mandatory, CORS whitelist, SQL injection prevention via SQLModel
- **Commit strategy**: Commit after each task or logical group (e.g., after completing a user story phase)
- **Stop at checkpoints**: Validate user story independently before proceeding to next priority
- **MVP focus**: Phases 1-4 deliver MVP (authentication + basic todo CRUD) - prioritize these first

---

## Total Task Count: 139 tasks

**By Phase**:
- Phase 1 (Setup): 13 tasks
- Phase 2 (Foundational): 17 tasks (BLOCKS all user stories)
- Phase 3 (US1 - Registration & Login): 18 tasks - P1
- Phase 4 (US2 - Create & View Todos): 33 tasks - P1
- Phase 5 (US3 - Update Status): 15 tasks - P2
- Phase 6 (US5 - Delete Todos): 9 tasks - P2
- Phase 7 (US4 - Edit Details): 12 tasks - P3
- Phase 8 (Polish): 22 tasks

**By Priority**:
- P1 (MVP Critical): Phases 1-4 (81 tasks)
- P2 (High Value): Phases 5-6 (24 tasks)
- P3 (Nice-to-Have): Phase 7 (12 tasks)
- Cross-Cutting: Phase 8 (22 tasks)

**Parallel Opportunities Identified**: 47 tasks marked [P] can run in parallel within their phases

**Suggested MVP Scope**: Phases 1-4 (81 tasks) delivers authenticated users creating and viewing their personal todos with complete data isolation.
