# API Contracts: Todo App

**Feature**: Phase II Full-Stack Todo Web Application
**Date**: 2025-12-31
**API Version**: 1.0.0

## Overview

This directory contains formal API contracts for the Todo application backend. All endpoints are exposed under the `/api/` prefix and require JWT authentication (except health check).

## Contract Files

- **openapi.yaml**: OpenAPI 3.1 specification (machine-readable, used for code generation and Swagger UI)

## Authentication

All endpoints (except `/api/health`) require JWT Bearer token authentication:

```http
Authorization: Bearer <jwt_token>
```

**JWT Payload Structure**:
```json
{
  "id": "user-uuid-string",
  "email": "user@example.com",
  "iat": 1735635000,
  "exp": 1735638600
}
```

**Token Expiration**: 1 hour (3600 seconds)

**Verification**: FastAPI verifies JWT signatures via Better Auth's JWKS endpoint (`/api/auth/jwks`)

## Endpoint Summary

### Health Check

```http
GET /api/health
```

**Authentication**: None required

**Response**: `200 OK`
```json
{
  "status": "ok"
}
```

---

### User Sync

```http
POST /api/users/sync
Authorization: Bearer <token>
```

**Purpose**: Create minimal user record in FastAPI after Better Auth registration

**Request Body**: None (user info extracted from JWT)

**Response**: `200 OK`
```json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "email": "alice@example.com",
  "created_at": "2025-12-31T10:30:00Z"
}
```

**Notes**:
- Idempotent (safe to call multiple times)
- Automatically extracts `user_id` and `email` from JWT payload
- Should be called immediately after Better Auth registration completes

---

### List Todos

```http
GET /api/todos?skip=0&limit=50
Authorization: Bearer <token>
```

**Query Parameters**:
- `skip` (optional, default: 0): Number of records to skip
- `limit` (optional, default: 50, max: 100): Number of records to return

**Response**: `200 OK`
```json
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "user_id": "123e4567-e89b-12d3-a456-426614174000",
      "title": "Buy groceries",
      "description": "Milk, eggs, bread",
      "completed": false,
      "created_at": "2025-12-31T10:00:00Z",
      "updated_at": "2025-12-31T10:00:00Z"
    }
  ],
  "total": 42
}
```

**Notes**:
- Returns only todos owned by authenticated user (data isolation enforced)
- Results ordered by `created_at DESC` (newest first)

---

### Create Todo

```http
POST /api/todos
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body**:
```json
{
  "title": "Buy groceries",
  "description": "Milk, eggs, bread"  // optional
}
```

**Validation**:
- `title`: Required, 1-500 characters, cannot be empty/whitespace
- `description`: Optional, max 2000 characters

**Response**: `201 Created`
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "user_id": "123e4567-e89b-12d3-a456-426614174000",
  "title": "Buy groceries",
  "description": "Milk, eggs, bread",
  "completed": false,
  "created_at": "2025-12-31T10:00:00Z",
  "updated_at": "2025-12-31T10:00:00Z"
}
```

---

### Get Todo

```http
GET /api/todos/{todo_id}
Authorization: Bearer <token>
```

**Path Parameters**:
- `todo_id`: UUID string

**Response**: `200 OK`
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "user_id": "123e4567-e89b-12d3-a456-426614174000",
  "title": "Buy groceries",
  "description": "Milk, eggs, bread",
  "completed": false,
  "created_at": "2025-12-31T10:00:00Z",
  "updated_at": "2025-12-31T10:00:00Z"
}
```

**Error**: `404 Not Found` (if todo doesn't exist OR belongs to another user)

---

### Update Todo

```http
PUT /api/todos/{todo_id}
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body** (all fields optional):
```json
{
  "title": "Buy groceries and cook dinner",
  "description": "Updated shopping list",
  "completed": true
}
```

**Validation**:
- `title`: Optional, 1-500 characters if provided
- `description`: Optional, max 2000 characters if provided
- `completed`: Optional boolean

**Response**: `200 OK`
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "user_id": "123e4567-e89b-12d3-a456-426614174000",
  "title": "Buy groceries and cook dinner",
  "description": "Updated shopping list",
  "completed": true,
  "created_at": "2025-12-31T10:00:00Z",
  "updated_at": "2025-12-31T10:30:00Z"
}
```

**Error**: `404 Not Found` (if todo doesn't exist OR belongs to another user)

---

### Delete Todo

```http
DELETE /api/todos/{todo_id}
Authorization: Bearer <token>
```

**Path Parameters**:
- `todo_id`: UUID string

**Response**: `204 No Content`

**Error**: `404 Not Found` (if todo doesn't exist OR belongs to another user)

---

## Error Response Format

All errors follow this standard format:

```json
{
  "detail": "Human-readable error message",
  "error_code": "MACHINE_READABLE_CODE"
}
```

### Common Error Codes

| HTTP Status | Error Code | Description |
|------------|------------|-------------|
| 400 | `VALIDATION_ERROR` | Invalid request body or parameters |
| 401 | `UNAUTHORIZED` | Missing or invalid JWT token |
| 404 | `NOT_FOUND` | Resource not found or not owned by user |
| 500 | `INTERNAL_ERROR` | Internal server error |

### Example Error Responses

**400 Validation Error**:
```json
{
  "detail": "Title cannot be empty or whitespace",
  "error_code": "VALIDATION_ERROR"
}
```

**401 Unauthorized**:
```json
{
  "detail": "Invalid or expired token",
  "error_code": "UNAUTHORIZED"
}
```

**404 Not Found**:
```json
{
  "detail": "Todo not found",
  "error_code": "NOT_FOUND"
}
```

**500 Internal Server Error**:
```json
{
  "detail": "Service temporarily unavailable. Please try again.",
  "error_code": "INTERNAL_ERROR"
}
```

---

## Data Isolation

**Critical Security Requirement**: All todo queries filter by `user_id` extracted from JWT token.

- Users can ONLY access their own todos
- Cross-user access attempts return `404 Not Found` (not `403 Forbidden`)
- This prevents information leakage about other users' data

**Example Scenario**:
- Alice (user_id: `aaa...`) creates a todo with ID `todo-123`
- Bob (user_id: `bbb...`) attempts `GET /api/todos/todo-123`
- Server returns `404 Not Found` (even though todo exists, Bob doesn't own it)

---

## Pagination

List endpoints support pagination via `skip` and `limit` query parameters:

**Parameters**:
- `skip`: Number of records to skip (default: 0)
- `limit`: Maximum records to return (default: 50, max: 100)

**Response Format**:
```json
{
  "data": [...],
  "total": 150  // Total count of user's todos
}
```

**Example**:
```bash
# Get first page (records 1-50)
GET /api/todos?skip=0&limit=50

# Get second page (records 51-100)
GET /api/todos?skip=50&limit=50

# Get third page (records 101-150)
GET /api/todos?skip=100&limit=50
```

---

## Testing with Swagger UI

Once the FastAPI server is running, access interactive API documentation at:

- **Swagger UI**: `http://localhost:8000/docs`
- **ReDoc**: `http://localhost:8000/redoc`

**Testing authenticated endpoints**:
1. Register/login via Better Auth frontend to get JWT token
2. Click "Authorize" button in Swagger UI
3. Enter `Bearer <your_jwt_token>`
4. Test endpoints interactively

---

## Contract Validation Checklist

- [x] All endpoints documented with request/response schemas
- [x] Authentication requirements specified
- [x] Error responses follow consistent format
- [x] Data isolation enforced (404 for cross-user access)
- [x] Validation rules match functional requirements (FR-008, FR-016)
- [x] HTTP status codes follow REST conventions
- [x] UUID identifiers for todos and users
- [x] Pagination parameters documented
- [x] Auto-generated OpenAPI spec available at `/docs`

---

## Implementation Notes

**FastAPI Code Generation**:
- FastAPI automatically generates OpenAPI spec from code
- Ensure all endpoints have docstrings and type hints
- Use Pydantic models for request/response validation

**Frontend Client Generation**:
- Use OpenAPI generators (e.g., `openapi-typescript`) to generate TypeScript client
- Ensures type safety between frontend and backend

**Contract Testing**:
- Use `openapi.yaml` to generate contract tests
- Validate that implementation matches specification
