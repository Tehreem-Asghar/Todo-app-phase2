# Research: Todo App Implementation Architecture

**Feature**: Phase II Full-Stack Todo Web Application
**Date**: 2025-12-31
**Status**: Completed

## Overview

This document consolidates architectural research and decisions for implementing a secure, full-stack Todo application using Next.js, FastAPI, Better Auth, and Neon PostgreSQL.

## Key Architecture Decisions

### 1. Authentication Strategy: Better Auth + JWT

**Decision**: Use Better Auth for authentication management with JWT token-based session handling

**Rationale**:
- Better Auth is a modern TypeScript-first authentication library designed for Next.js App Router
- Built-in JWT support with JWKS endpoint for public key distribution
- Handles password hashing (bcrypt), email validation, and user management automatically
- Reduces implementation complexity compared to building custom auth from scratch
- JWT tokens enable stateless authentication suitable for API-first architecture

**Alternatives Considered**:
- **NextAuth.js**: More mature but heavier; Better Auth is lighter and designed for modern Next.js
- **Custom JWT implementation**: Higher security risk, more development time, reinventing the wheel
- **Session-based auth with Redis**: Requires additional infrastructure (Redis), stateful sessions don't scale as easily for hackathon demo

**Implementation Details**:
- Better Auth manages the `users` table in Neon PostgreSQL
- JWT tokens issued on login with 1-hour expiration (claim: `{id, email, iat, exp}`)
- FastAPI verifies JWT signatures via Better Auth's `/api/auth/jwks` endpoint
- User sync flow: After registration, frontend calls `POST /api/users/sync` to create minimal FastAPI user record

### 2. Database Architecture: Shared Neon PostgreSQL

**Decision**: Use a single Neon Serverless PostgreSQL instance shared by Better Auth and FastAPI

**Rationale**:
- Simplifies infrastructure - no need to manage multiple databases or synchronization
- Neon's serverless model provides automatic scaling and connection pooling
- Better Auth can own its tables (`users`, `sessions`, etc.) while FastAPI owns application tables (`todos`, `users_sync`)
- Foreign key relationships work natively (FastAPI `todos.user_id` → Better Auth `users.id`)

**Alternatives Considered**:
- **Separate databases**: Adds complexity, requires user data synchronization logic, complicates foreign keys
- **Embedded SQLite**: Not production-ready, no concurrent access support
- **PostgreSQL + Redis**: Overkill for hackathon scope, adds infrastructure complexity

**Schema Strategy**:
- Better Auth tables: `users`, `sessions`, `accounts`, `verifications` (managed by Better Auth migrations)
- FastAPI tables: `users_sync` (minimal: id, email, created_at), `todos` (id, user_id, title, description, completed, created_at, updated_at)
- User sync ensures FastAPI has foreign key reference without duplicating authentication logic

### 3. User Identity Management: UUID Identifiers

**Decision**: Use UUID strings for `user_id` and `todo_id` instead of auto-incrementing integers

**Rationale**:
- **Security**: Prevents ID enumeration attacks (attacker can't guess valid IDs by incrementing)
- **Better Auth default**: Better Auth generates UUIDs for user IDs by default
- **Scalability**: UUIDs avoid ID collision issues in distributed systems or multi-tenant scenarios
- **RESTful best practices**: Obscures resource count and access patterns

**Alternatives Considered**:
- **Auto-incrementing integers**: Simpler but exposes resource count, enables enumeration attacks
- **Snowflake IDs**: Overkill for single-database hackathon app

**Implementation**:
- PostgreSQL `uuid` type for both `users.id` and `todos.id`
- Python: Use `str` type in Pydantic/SQLModel (UUID stored as string)
- Frontend: Pass UUIDs as strings in API requests

### 4. API Routing Structure: Dedicated `/api/` Prefix

**Decision**: All backend routes must be organized in a dedicated `api` routes folder and exposed under `/api/` URL prefix

**Rationale**:
- **Constitution requirement**: Explicitly mandates this structure (constitution.md:36)
- **Separation of concerns**: Clear boundary between frontend routes and API endpoints
- **API versioning**: Easy to add `/api/v2/` if needed in future
- **Security**: Simplifies CORS, rate limiting, and authentication middleware configuration

**Implementation**:
- FastAPI app mounted at `/api` prefix
- Example routes: `GET /api/todos`, `POST /api/users/sync`, `PUT /api/todos/{todo_id}`
- OpenAPI docs available at `/docs` and `/redoc`

### 5. JWT Verification: JWKS-Based Signature Validation

**Decision**: FastAPI verifies JWT tokens by fetching public keys from Better Auth's JWKS endpoint

**Rationale**:
- **Security best practice**: Public key cryptography ensures tokens aren't forged
- **Decoupling**: FastAPI doesn't need access to Better Auth's secret key
- **Standard protocol**: JWKS (JSON Web Key Set) is RFC 7517 standard used by Auth0, Okta, etc.
- **Key rotation support**: If Better Auth rotates keys, FastAPI automatically fetches new keys

**Alternatives Considered**:
- **Shared secret verification**: Requires sharing secret key between frontend and backend (security risk)
- **Database session lookup**: Stateful, requires additional database queries, doesn't scale

**Implementation**:
- FastAPI dependency: `get_current_user()` extracts JWT from `Authorization: Bearer <token>` header
- On first request, fetch JWKS from `https://frontend-url/api/auth/jwks` and cache public keys
- Verify JWT signature using `pyjwt` with fetched public key
- Extract `user_id = payload["id"]` for database filtering

### 6. Error Response Strategy: 404 vs 403 for Cross-User Access

**Decision**: Return 404 Not Found (not 403 Forbidden) when users attempt to access other users' todos

**Rationale**:
- **Security**: Prevents information leakage - 403 confirms the resource exists but user lacks permission
- **Privacy**: Attacker can't enumerate valid todo IDs by checking status codes
- **Spec requirement**: FR-015 explicitly requires 404 for cross-user access attempts

**Alternatives Considered**:
- **403 Forbidden**: More semantically accurate but reveals existence of resources
- **Generic 400 Bad Request**: Loses semantic meaning, makes debugging harder

**Implementation**:
- Database queries always filter by `user_id` from JWT: `WHERE user_id = {current_user_id} AND id = {todo_id}`
- If no rows returned, raise `HTTPException(status_code=404, detail="Todo not found")`
- Same error message whether todo doesn't exist or belongs to another user

### 7. Logout Strategy: Client-Side Token Removal

**Decision**: Implement logout by removing JWT token from client-side storage (no server-side token blacklist)

**Rationale**:
- **Simplicity**: No additional infrastructure (Redis blacklist, database tracking)
- **Stateless architecture**: Maintains JWT's stateless benefits
- **Hackathon scope**: Acceptable trade-off for demo timeline
- **Token expiry**: 1-hour expiration limits risk window if token is compromised

**Alternatives Considered**:
- **Token blacklist in Redis**: Adds infrastructure complexity, defeats stateless JWT benefits
- **Short-lived tokens + refresh tokens**: More secure but adds implementation complexity

**Implementation**:
- Frontend: Remove JWT from localStorage/sessionStorage on logout
- Better Auth: Call Better Auth's logout handler to clear any server-side sessions
- Security note: Expired tokens are already rejected by JWT verification

### 8. User Sync Flow: POST /api/users/sync

**Decision**: After Better Auth registration, frontend immediately calls `POST /api/users/sync` to create a minimal user record in FastAPI

**Rationale**:
- **Foreign key requirement**: FastAPI `todos.user_id` needs a foreign key reference
- **Separation of concerns**: Better Auth owns authentication, FastAPI owns application data
- **Minimal duplication**: Only sync essential fields (id, email, created_at) - no password or sensitive data
- **Idempotent**: Calling sync multiple times is safe (upsert behavior)

**Implementation**:
- Endpoint: `POST /api/users/sync` (authenticated, extracts user_id from JWT)
- Request body: None (user info extracted from JWT payload)
- FastAPI logic: `INSERT INTO users_sync (id, email, created_at) VALUES (...) ON CONFLICT (id) DO NOTHING`
- Call timing: Immediately after Better Auth registration completes

## Technology Best Practices

### Next.js 14 App Router

**Key Patterns**:
- Server Components by default (better performance, reduced JavaScript bundle)
- Client Components ("`use client`") only when needed (forms, interactive state)
- Server Actions for form submissions (eliminates need for separate API routes for some operations)
- Middleware for authentication checks on protected routes

**File Structure**:
```
app/
├── (auth)/
│   ├── login/page.tsx
│   └── register/page.tsx
├── (dashboard)/
│   └── todos/page.tsx
├── api/
│   └── auth/[...auth]/route.ts  # Better Auth handler
└── layout.tsx
```

### FastAPI + SQLModel

**Key Patterns**:
- SQLModel for ORM (combines SQLAlchemy + Pydantic)
- Dependency injection for database sessions and current user
- Response models separate from database models (DTO pattern)
- Alembic for database migrations

**File Structure**:
```
backend/
├── src/
│   ├── api/
│   │   ├── dependencies.py  # get_db, get_current_user
│   │   ├── todos.py         # Todo CRUD routes
│   │   └── users.py         # User sync route
│   ├── models/
│   │   ├── todo.py          # Todo SQLModel
│   │   └── user.py          # UserSync SQLModel
│   ├── schemas/
│   │   ├── todo.py          # TodoCreate, TodoUpdate, TodoResponse
│   │   └── user.py          # UserResponse
│   ├── core/
│   │   ├── config.py        # Settings from env vars
│   │   └── security.py      # JWT verification logic
│   └── main.py              # FastAPI app
├── alembic/
└── tests/
```

### Better Auth Configuration

**Key Settings**:
```typescript
// lib/auth.ts
export const authConfig = {
  database: {
    type: "postgres",
    url: process.env.DATABASE_URL,
  },
  plugins: [
    jwt({
      expiresIn: 3600, // 1 hour
      algorithm: "RS256", // RSA for JWKS support
    }),
  ],
  emailAndPassword: {
    enabled: true,
    minPasswordLength: 8,
  },
}
```

### Pydantic Validation

**Key Patterns**:
- Field validators for length, format constraints
- Custom validators for business logic
- Consistent error responses via exception handlers

**Example**:
```python
class TodoCreate(BaseModel):
    title: str = Field(..., min_length=1, max_length=500)
    description: Optional[str] = Field(None, max_length=2000)

    @validator('title')
    def title_not_empty(cls, v):
        if not v.strip():
            raise ValueError('Title cannot be empty or whitespace')
        return v.strip()
```

## Security Implementation Checklist

- [x] JWT signature verification via JWKS
- [x] All todo queries filter by user_id from JWT
- [x] Pydantic validation on all request bodies
- [x] Password hashing handled by Better Auth (bcrypt, 12+ rounds)
- [x] SQL injection prevention via SQLModel parameterized queries
- [x] CORS whitelist (configured via env vars)
- [x] Environment variables for secrets (no hardcoded credentials)
- [x] XSS prevention via React's default escaping
- [x] 404 responses for cross-user access (prevent enumeration)
- [x] JWT expiration enforced (1 hour)

## Performance Considerations

- **Database indexing**: Index on `todos.user_id` for fast filtering
- **Connection pooling**: Neon provides automatic pooling
- **Response times**: Target <500ms for API responses (spec SC-004)
- **Pagination**: Default limit=50, max limit=100 (spec FR-009)

## Open Questions Resolved

| Question | Resolution |
|----------|-----------|
| How to sync Better Auth users with FastAPI? | POST /api/users/sync after registration |
| JWT claim name for user ID? | Use `"id"` claim (Better Auth default) |
| UUID vs integer IDs? | UUID strings for security (prevents enumeration) |
| Logout implementation? | Client-side token removal (acceptable for hackathon scope) |
| JWT expiration duration? | 1 hour (3600 seconds) - balances security and usability |
| 403 vs 404 for cross-user access? | 404 to prevent information leakage |
| Shared vs separate database? | Shared Neon PostgreSQL (simpler, native foreign keys) |

## Risk Analysis

| Risk | Mitigation |
|------|-----------|
| JWT token theft (XSS attack) | Store in httpOnly cookies (not localStorage) if possible; React default escaping prevents XSS |
| Expired token during long session | Frontend detects 401 responses, redirects to login |
| Better Auth JWKS endpoint unavailable | FastAPI caches public keys; return 503 if verification fails |
| Database connection failure | Neon's connection pooling + retry logic; return 500 with generic error |
| User enumeration via timing attacks | Use constant-time password comparison (Better Auth handles this) |

## Next Steps

This research phase is complete. Proceed to Phase 1:
1. Generate `data-model.md` with entity definitions
2. Create API contracts in `contracts/` directory
3. Generate `quickstart.md` for local development setup
4. Update agent context files with technology choices
