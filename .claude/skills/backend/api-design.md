---
name: api-design
description: RESTful API design principles and best practices
---

# API Design Best Practices

## REST Principles

### Resource Naming

```
# ✅ Good - Nouns, plural, lowercase
GET    /users
GET    /users/123
GET    /users/123/orders
POST   /users
PUT    /users/123
DELETE /users/123

# ❌ Bad - Verbs, actions in URL
GET    /getUsers
POST   /createUser
GET    /getUserOrders/123
```

### HTTP Methods

| Method | Purpose | Idempotent | Safe |
|--------|---------|------------|------|
| GET | Retrieve resource | Yes | Yes |
| POST | Create resource | No | No |
| PUT | Replace resource | Yes | No |
| PATCH | Partial update | No | No |
| DELETE | Remove resource | Yes | No |

### Status Codes

```
# Success
200 OK              - General success
201 Created         - Resource created (POST)
204 No Content      - Success, no body (DELETE)

# Client Errors
400 Bad Request     - Invalid input
401 Unauthorized    - Not authenticated
403 Forbidden       - Not authorized
404 Not Found       - Resource doesn't exist
409 Conflict        - State conflict (duplicate, etc.)
422 Unprocessable   - Validation failed

# Server Errors
500 Internal Error  - Unexpected server error
503 Service Unavailable - Temporary overload
```

## Request/Response Design

### Request Format

```typescript
// POST /users
{
  "email": "user@example.com",
  "name": "John Doe",
  "role": "admin"
}

// PATCH /users/123 (partial update)
{
  "name": "Jane Doe"
}

// Query parameters for filtering/pagination
GET /users?role=admin&status=active&page=2&limit=20&sort=-createdAt
```

### Response Format

```typescript
// Single resource
{
  "data": {
    "id": "123",
    "type": "user",
    "attributes": {
      "email": "user@example.com",
      "name": "John Doe",
      "createdAt": "2024-01-15T10:30:00Z"
    },
    "relationships": {
      "organization": {
        "data": { "type": "organization", "id": "456" }
      }
    }
  }
}

// Collection
{
  "data": [...],
  "meta": {
    "total": 100,
    "page": 2,
    "limit": 20,
    "totalPages": 5
  },
  "links": {
    "self": "/users?page=2",
    "first": "/users?page=1",
    "prev": "/users?page=1",
    "next": "/users?page=3",
    "last": "/users?page=5"
  }
}

// Error response
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ]
  }
}
```

## Versioning

```
# URL versioning (recommended for simplicity)
GET /v1/users
GET /v2/users

# Header versioning
GET /users
Accept: application/vnd.api+json;version=2

# Query parameter
GET /users?version=2
```

## Filtering, Sorting, Pagination

```typescript
// Filtering
GET /products?category=electronics&price[gte]=100&price[lte]=500

// Sorting (prefix with - for descending)
GET /products?sort=price          // Ascending
GET /products?sort=-createdAt     // Descending
GET /products?sort=-price,name    // Multiple fields

// Pagination
GET /products?page=2&limit=20     // Offset-based
GET /products?cursor=abc123&limit=20  // Cursor-based (better for large datasets)

// Field selection (sparse fieldsets)
GET /users?fields=id,name,email
GET /users/123?include=orders,profile
```

## Authentication & Security

```typescript
// Bearer token (JWT)
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

// API Key
X-API-Key: your-api-key

// Rate limiting headers
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640995200

// Security headers
Strict-Transport-Security: max-age=31536000
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
```

## API Design Checklist

### URLs
- [ ] Use nouns, not verbs
- [ ] Plural resource names
- [ ] Lowercase, hyphen-separated
- [ ] No trailing slashes
- [ ] Version in URL path

### Responses
- [ ] Consistent response structure
- [ ] Proper status codes
- [ ] Meaningful error messages
- [ ] Include pagination metadata
- [ ] ISO 8601 date format

### Security
- [ ] HTTPS only
- [ ] Authentication required
- [ ] Rate limiting
- [ ] Input validation
- [ ] No sensitive data in URLs

### Documentation
- [ ] OpenAPI/Swagger spec
- [ ] Example requests/responses
- [ ] Error code reference
- [ ] Authentication guide
- [ ] Rate limit documentation
