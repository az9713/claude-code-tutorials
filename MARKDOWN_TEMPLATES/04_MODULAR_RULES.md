# Claude Code Modular Rules - Complete Guide

> **Modular Rules** are path-specific coding standards that automatically load when working with matching files. They allow you to define different conventions for different parts of your codebase.

---

## Quick Reference

| Property | Value |
|----------|-------|
| **Location** | `.claude/rules/*.md` |
| **Frontmatter** | Optional (for path filtering) |
| **Loading** | Automatic when matching files are active |
| **Scope** | Path-specific via glob patterns |

---

## Template Anatomy

```yaml
---
# OPTIONAL - Path filtering
paths:
  - "src/components/**/*.tsx"     # React components
  - "src/components/**/*.ts"      # Component utilities
  - "!src/components/**/*.test.*" # Exclude tests
---

# Rule Content (Markdown Body)

Rules defined here apply only when working with files
matching the paths above. If no paths specified,
rules apply globally.
```

### Path Pattern Syntax

| Pattern | Matches |
|---------|---------|
| `**/*.ts` | All TypeScript files |
| `src/**/*.tsx` | TSX files under src |
| `*.config.js` | Config files in root |
| `!**/*.test.*` | Exclude test files |
| `src/{components,hooks}/**` | Multiple directories |

---

## Templates

### Minimal Template

```markdown
---
paths:
  - "**/*.ts"
---

Use strict TypeScript. Avoid `any` types.
```

### Standard Template

```markdown
---
paths:
  - "src/components/**/*.tsx"
  - "src/components/**/*.ts"
---

# React Component Rules

## Structure
- One component per file
- Props interface above component
- Export at end of file

## Naming
- Component: PascalCase (matches filename)
- Props: `{ComponentName}Props`
- Hooks: `use{HookName}`

## Patterns
- Prefer composition over inheritance
- Use custom hooks for shared logic
- Keep components under 200 lines
```

### Advanced Template

```markdown
---
paths:
  - "src/api/**/*.ts"
  - "src/services/**/*.ts"
---

# Backend/API Rules

## Error Handling

All API functions must:
1. Use typed error classes
2. Log errors with context
3. Return consistent error format

\`\`\`typescript
// GOOD
try {
  const result = await operation();
  return { success: true, data: result };
} catch (error) {
  logger.error('Operation failed', { error, context });
  throw new ApiError('OPERATION_FAILED', error);
}

// BAD
try {
  return await operation();
} catch (e) {
  throw e;
}
\`\`\`

## Validation

All inputs must be validated:
\`\`\`typescript
// GOOD - Validate at boundary
const validated = schema.parse(input);

// BAD - Trust external input
const data = req.body as UserData;
\`\`\`

## Database Access

- Use repository pattern
- Never raw SQL in controllers
- Always use transactions for multi-step operations

## Testing Requirements

- Unit tests for all service methods
- Integration tests for API endpoints
- Minimum 80% coverage
```

---

## Examples

### Example 1: TypeScript Rules

```markdown
---
paths:
  - "**/*.ts"
  - "**/*.tsx"
  - "!**/*.test.*"
  - "!**/*.spec.*"
---

# TypeScript Coding Standards

## Type Safety

### Strict Mode
- Project uses `strict: true`
- Never use `// @ts-ignore` without explanation
- Never use `any` - use `unknown` if type is truly unknown

### Type Definitions
\`\`\`typescript
// GOOD - Explicit interfaces
interface User {
  id: string;
  name: string;
  email: string;
}

// GOOD - Discriminated unions
type Result<T> =
  | { success: true; data: T }
  | { success: false; error: Error };

// BAD - Avoid any
function process(data: any) { ... }

// GOOD - Use unknown
function process(data: unknown) {
  if (isValidData(data)) { ... }
}
\`\`\`

### Return Types
\`\`\`typescript
// GOOD - Explicit return types on public functions
export function calculateTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}

// OK - Inferred for private/internal functions
const helper = (x: number) => x * 2;
\`\`\`

## Null Handling

\`\`\`typescript
// GOOD - Use optional chaining
const name = user?.profile?.name;

// GOOD - Nullish coalescing
const displayName = user.name ?? 'Anonymous';

// BAD - Loose equality
if (value == null) { ... }

// GOOD - Strict checks
if (value === null || value === undefined) { ... }
\`\`\`

## Enums vs Union Types

\`\`\`typescript
// GOOD - Const objects (tree-shakeable, no runtime)
const Status = {
  PENDING: 'pending',
  ACTIVE: 'active',
  COMPLETED: 'completed',
} as const;
type Status = typeof Status[keyof typeof Status];

// OK - String literal unions (simpler)
type Status = 'pending' | 'active' | 'completed';

// AVOID - Enums (can have runtime issues)
enum Status { PENDING, ACTIVE, COMPLETED }
\`\`\`

## Imports

\`\`\`typescript
// Order:
// 1. External packages
// 2. Internal packages (@company/*)
// 3. Relative imports

import { useState } from 'react';
import { Button } from '@company/ui';
import { formatDate } from '../utils';
\`\`\`
```

### Example 2: Python Rules

```markdown
---
paths:
  - "**/*.py"
  - "!**/*_test.py"
  - "!**/test_*.py"
---

# Python Coding Standards

## Style

### PEP 8 Compliance
- Max line length: 88 (Black default)
- Use Black for formatting
- Use isort for import sorting

### Type Hints (PEP 484)
\`\`\`python
# GOOD - All function signatures typed
def calculate_total(items: list[Item]) -> Decimal:
    return sum(item.price for item in items)

# GOOD - Use Optional for nullable
def find_user(user_id: str) -> Optional[User]:
    return db.users.get(user_id)

# GOOD - TypedDict for complex dicts
class UserData(TypedDict):
    name: str
    email: str
    age: NotRequired[int]

# BAD - Untyped function
def process(data):
    return data['value']
\`\`\`

## Error Handling

\`\`\`python
# GOOD - Specific exceptions
try:
    result = operation()
except ValueError as e:
    logger.error(f"Invalid value: {e}")
    raise
except ConnectionError as e:
    logger.error(f"Connection failed: {e}")
    raise ServiceUnavailable() from e

# BAD - Bare except
try:
    result = operation()
except:
    pass  # Silent failure

# BAD - Too broad
except Exception as e:
    ...
\`\`\`

## Docstrings

\`\`\`python
def calculate_discount(
    price: Decimal,
    discount_percent: int,
    *,
    max_discount: Decimal = Decimal("100.00"),
) -> Decimal:
    """Calculate discounted price.

    Args:
        price: Original price before discount.
        discount_percent: Discount percentage (0-100).
        max_discount: Maximum discount amount allowed.

    Returns:
        Final price after discount applied.

    Raises:
        ValueError: If discount_percent not in range 0-100.

    Example:
        >>> calculate_discount(Decimal("100.00"), 20)
        Decimal("80.00")
    """
\`\`\`

## Imports

\`\`\`python
# Order (isort):
# 1. Standard library
# 2. Third-party
# 3. Local

from collections import defaultdict
from typing import Optional

import pytest
from sqlalchemy import select

from app.models import User
from app.services import user_service
\`\`\`

## Testing

\`\`\`python
# Use pytest fixtures
@pytest.fixture
def user():
    return User(name="Test", email="test@example.com")

# Descriptive test names
def test_calculate_discount_applies_percentage():
    assert calculate_discount(Decimal("100"), 20) == Decimal("80")

# Use parametrize for multiple cases
@pytest.mark.parametrize("input,expected", [
    (0, 0),
    (100, 80),
    (50, 40),
])
def test_calculate_discount_cases(input, expected):
    assert calculate_discount(Decimal(input), 20) == Decimal(expected)
\`\`\`
```

### Example 3: React Component Rules

```markdown
---
paths:
  - "src/components/**/*.tsx"
  - "src/app/**/components/**/*.tsx"
---

# React Component Rules

## File Structure

\`\`\`typescript
// 1. Imports
import { useState, useCallback } from 'react';
import { Button } from '@/components/ui';
import type { User } from '@/types';

// 2. Types
interface UserCardProps {
  user: User;
  onSelect?: (user: User) => void;
  className?: string;
}

// 3. Component
export function UserCard({ user, onSelect, className }: UserCardProps) {
  // 3a. Hooks
  const [isExpanded, setIsExpanded] = useState(false);

  // 3b. Handlers
  const handleClick = useCallback(() => {
    onSelect?.(user);
  }, [user, onSelect]);

  // 3c. Render
  return (
    <div className={className}>
      {/* JSX */}
    </div>
  );
}

// 4. Sub-components (if small and only used here)
function UserAvatar({ src, alt }: { src: string; alt: string }) {
  return <img src={src} alt={alt} className="avatar" />;
}
\`\`\`

## Component Patterns

### Props Interface
\`\`\`typescript
// GOOD - Named {Component}Props
interface ButtonProps {
  variant: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
  children: React.ReactNode;
  onClick?: () => void;
}

// GOOD - Extend HTML attributes when wrapping native elements
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant: 'primary' | 'secondary';
}
\`\`\`

### Conditional Rendering
\`\`\`typescript
// GOOD - Early return for simple conditions
if (!user) return null;
if (isLoading) return <Spinner />;

// GOOD - Ternary for inline
{isActive ? <Active /> : <Inactive />}

// GOOD - && for presence
{error && <ErrorMessage error={error} />}

// BAD - Nested ternaries
{a ? b ? <C /> : <D /> : <E />}
\`\`\`

### Event Handlers
\`\`\`typescript
// GOOD - useCallback for handlers passed to children
const handleSubmit = useCallback((data: FormData) => {
  onSubmit(data);
}, [onSubmit]);

// OK - Inline for simple, not passed down
<button onClick={() => setOpen(true)}>
\`\`\`

## State Management

### Local State
\`\`\`typescript
// Use useState for UI state
const [isOpen, setIsOpen] = useState(false);

// Use useReducer for complex state
const [state, dispatch] = useReducer(reducer, initialState);
\`\`\`

### Server State
\`\`\`typescript
// Use React Query for server data
const { data: users, isLoading } = useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers,
});
\`\`\`

## Accessibility

\`\`\`typescript
// GOOD - Semantic HTML
<button onClick={handleClick}>Submit</button>
<nav aria-label="Main navigation">

// GOOD - ARIA when needed
<div role="dialog" aria-modal="true" aria-labelledby="dialog-title">

// BAD - Click handler on div without keyboard support
<div onClick={handleClick}>Clickable</div>

// GOOD - If must use div, add keyboard support
<div
  role="button"
  tabIndex={0}
  onClick={handleClick}
  onKeyDown={(e) => e.key === 'Enter' && handleClick()}
>
\`\`\`

## Testing

\`\`\`typescript
// Test file: UserCard.test.tsx

describe('UserCard', () => {
  it('renders user name', () => {
    render(<UserCard user={mockUser} />);
    expect(screen.getByText(mockUser.name)).toBeInTheDocument();
  });

  it('calls onSelect when clicked', async () => {
    const onSelect = vi.fn();
    render(<UserCard user={mockUser} onSelect={onSelect} />);

    await userEvent.click(screen.getByRole('button'));

    expect(onSelect).toHaveBeenCalledWith(mockUser);
  });
});
\`\`\`
```

### Example 4: Security Rules

```markdown
---
paths:
  - "**/*.ts"
  - "**/*.js"
  - "**/*.py"
  - "**/*.java"
---

# Security Coding Rules

## Input Validation

### Never Trust User Input
\`\`\`typescript
// GOOD - Validate with schema
const userInput = userSchema.parse(req.body);

// GOOD - Sanitize before use
const sanitized = DOMPurify.sanitize(htmlInput);

// BAD - Direct use of user input
const query = \`SELECT * FROM users WHERE id = \${userId}\`;
\`\`\`

### Parameterized Queries
\`\`\`typescript
// GOOD - Parameterized
const user = await db.query(
  'SELECT * FROM users WHERE id = $1',
  [userId]
);

// GOOD - ORM with proper escaping
const user = await User.findOne({ where: { id: userId } });

// BAD - String concatenation
const user = await db.query(\`SELECT * FROM users WHERE id = \${userId}\`);
\`\`\`

## Authentication & Authorization

### Password Handling
\`\`\`typescript
// GOOD - Use bcrypt or argon2
const hash = await bcrypt.hash(password, 12);
const valid = await bcrypt.compare(password, hash);

// BAD - Weak hashing
const hash = crypto.createHash('md5').update(password).digest('hex');

// BAD - Storing plaintext
user.password = password;
\`\`\`

### Session Management
\`\`\`typescript
// GOOD - Secure session configuration
app.use(session({
  secret: process.env.SESSION_SECRET,
  cookie: {
    secure: true,        // HTTPS only
    httpOnly: true,      // No JS access
    sameSite: 'strict',  // CSRF protection
    maxAge: 3600000,     // 1 hour
  },
  resave: false,
  saveUninitialized: false,
}));
\`\`\`

### Authorization Checks
\`\`\`typescript
// GOOD - Check on every request
async function updatePost(postId: string, userId: string, data: PostData) {
  const post = await Post.findById(postId);
  if (!post) throw new NotFoundError();
  if (post.authorId !== userId) throw new ForbiddenError();
  return post.update(data);
}

// BAD - No authorization check
async function updatePost(postId: string, data: PostData) {
  return Post.findByIdAndUpdate(postId, data);
}
\`\`\`

## Secrets Management

### Never Hardcode Secrets
\`\`\`typescript
// GOOD - Environment variables
const apiKey = process.env.API_KEY;

// GOOD - Secret manager
const secret = await secretManager.getSecret('api-key');

// BAD - Hardcoded
const apiKey = 'sk_live_abc123xyz';

// BAD - In config file (even if gitignored)
const config = { apiKey: 'sk_live_abc123xyz' };
\`\`\`

### Environment Variable Validation
\`\`\`typescript
// GOOD - Validate at startup
const config = z.object({
  DATABASE_URL: z.string().url(),
  API_KEY: z.string().min(32),
  NODE_ENV: z.enum(['development', 'production', 'test']),
}).parse(process.env);
\`\`\`

## Output Encoding

### XSS Prevention
\`\`\`typescript
// GOOD - Framework auto-escaping (React)
return <div>{userContent}</div>;

// GOOD - Explicit sanitization when needed
const clean = DOMPurify.sanitize(htmlContent);

// BAD - dangerouslySetInnerHTML without sanitization
return <div dangerouslySetInnerHTML={{ __html: userContent }} />;
\`\`\`

## Logging

### Sensitive Data
\`\`\`typescript
// GOOD - Redact sensitive fields
logger.info('User login', {
  userId: user.id,
  email: '[REDACTED]',
  ip: request.ip,
});

// BAD - Logging sensitive data
logger.info('User login', { user });  // May include password, tokens
\`\`\`

## Dependencies

### Keep Updated
\`\`\`bash
# Regular audit
npm audit
pip-audit
\`\`\`

### Minimal Permissions
\`\`\`typescript
// GOOD - Request only needed permissions
const client = new S3Client({
  credentials: {
    // Use IAM role with minimal S3 permissions
  }
});
\`\`\`
```

### Example 5: Testing Rules

```markdown
---
paths:
  - "**/*.test.ts"
  - "**/*.test.tsx"
  - "**/*.spec.ts"
  - "**/*.spec.tsx"
  - "**/test_*.py"
  - "**/*_test.py"
  - "tests/**"
  - "__tests__/**"
---

# Testing Standards

## Test Structure

### Arrange-Act-Assert (AAA)
\`\`\`typescript
it('should calculate total correctly', () => {
  // Arrange
  const items = [{ price: 10 }, { price: 20 }];

  // Act
  const result = calculateTotal(items);

  // Assert
  expect(result).toBe(30);
});
\`\`\`

### Descriptive Names
\`\`\`typescript
// GOOD - Describes behavior
it('should return empty array when no items match filter')
it('should throw ValidationError when email is invalid')
it('should retry request up to 3 times on network failure')

// BAD - Vague
it('works')
it('handles error')
it('test case 1')
\`\`\`

## Test Types

### Unit Tests
\`\`\`typescript
// Test single unit in isolation
describe('calculateDiscount', () => {
  it('should apply percentage discount', () => {
    expect(calculateDiscount(100, 20)).toBe(80);
  });

  it('should not exceed maximum discount', () => {
    expect(calculateDiscount(100, 50, { max: 30 })).toBe(70);
  });

  it('should throw for invalid percentage', () => {
    expect(() => calculateDiscount(100, -10)).toThrow();
  });
});
\`\`\`

### Integration Tests
\`\`\`typescript
// Test multiple units together
describe('POST /api/orders', () => {
  it('should create order and update inventory', async () => {
    const response = await request(app)
      .post('/api/orders')
      .send({ productId: '123', quantity: 2 });

    expect(response.status).toBe(201);

    const inventory = await db.inventory.findById('123');
    expect(inventory.quantity).toBe(initialQuantity - 2);
  });
});
\`\`\`

## Mocking

### When to Mock
\`\`\`typescript
// GOOD - Mock external services
vi.mock('./emailService', () => ({
  sendEmail: vi.fn().mockResolvedValue({ sent: true }),
}));

// GOOD - Mock time
vi.useFakeTimers();
vi.setSystemTime(new Date('2024-01-15'));

// BAD - Over-mocking (testing mocks, not code)
vi.mock('./calculator');  // Don't mock the thing you're testing
\`\`\`

### Mock Patterns
\`\`\`typescript
// Partial mock
vi.mock('./api', async () => {
  const actual = await vi.importActual('./api');
  return {
    ...actual,
    fetchUser: vi.fn(),
  };
});

// Spy on method
const consoleSpy = vi.spyOn(console, 'error');
expect(consoleSpy).toHaveBeenCalledWith('Error message');
\`\`\`

## Test Data

### Use Factories
\`\`\`typescript
// factories/user.ts
export const userFactory = {
  build: (overrides?: Partial<User>): User => ({
    id: faker.string.uuid(),
    name: faker.person.fullName(),
    email: faker.internet.email(),
    ...overrides,
  }),
};

// In test
const user = userFactory.build({ name: 'Test User' });
\`\`\`

### Avoid Magic Values
\`\`\`typescript
// GOOD - Clear intent
const VALID_EMAIL = 'user@example.com';
const INVALID_EMAIL = 'not-an-email';

it('should accept valid email', () => {
  expect(validateEmail(VALID_EMAIL)).toBe(true);
});

// BAD - Magic strings
it('should accept valid email', () => {
  expect(validateEmail('abc@def.ghi')).toBe(true);
});
\`\`\`

## Coverage

### Meaningful Coverage
\`\`\`typescript
// GOOD - Test behavior, not implementation
it('should return filtered items', () => {
  const items = [{ active: true }, { active: false }];
  expect(filterActive(items)).toHaveLength(1);
});

// BAD - Testing implementation details
it('should call filter method', () => {
  const spy = vi.spyOn(Array.prototype, 'filter');
  filterActive([]);
  expect(spy).toHaveBeenCalled();
});
\`\`\`

### Coverage Targets
- New code: 80% minimum
- Critical paths: 95%+
- Edge cases: Explicitly tested

## Assertions

\`\`\`typescript
// GOOD - Specific assertions
expect(result).toEqual({ id: '123', name: 'Test' });
expect(array).toHaveLength(3);
expect(fn).toHaveBeenCalledWith('arg1', expect.any(Function));

// BAD - Weak assertions
expect(result).toBeTruthy();
expect(array.length > 0).toBe(true);
\`\`\`
```

### Example 6: API Design Rules

```markdown
---
paths:
  - "src/api/**/*.ts"
  - "src/routes/**/*.ts"
  - "app/api/**/*.ts"
  - "pages/api/**/*.ts"
---

# API Design Standards

## REST Conventions

### URL Structure
\`\`\`typescript
// GOOD - Resource-based URLs
GET    /users           // List users
GET    /users/:id       // Get user
POST   /users           // Create user
PUT    /users/:id       // Replace user
PATCH  /users/:id       // Update user
DELETE /users/:id       // Delete user

// GOOD - Nested resources
GET    /users/:id/posts       // User's posts
POST   /users/:id/posts       // Create user's post

// BAD - Action-based URLs
POST   /users/create
GET    /users/getAll
POST   /users/:id/delete
\`\`\`

### HTTP Methods
\`\`\`typescript
// GET - Read only, no side effects
router.get('/users', async (req, res) => {
  const users = await userService.list();
  res.json(users);
});

// POST - Create new resource
router.post('/users', async (req, res) => {
  const user = await userService.create(req.body);
  res.status(201).json(user);
});

// PUT - Replace entire resource
router.put('/users/:id', async (req, res) => {
  const user = await userService.replace(req.params.id, req.body);
  res.json(user);
});

// PATCH - Partial update
router.patch('/users/:id', async (req, res) => {
  const user = await userService.update(req.params.id, req.body);
  res.json(user);
});

// DELETE - Remove resource
router.delete('/users/:id', async (req, res) => {
  await userService.delete(req.params.id);
  res.status(204).send();
});
\`\`\`

## Request Validation

\`\`\`typescript
// GOOD - Validate all inputs
const createUserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
  role: z.enum(['user', 'admin']).default('user'),
});

router.post('/users', async (req, res) => {
  const data = createUserSchema.parse(req.body);
  const user = await userService.create(data);
  res.status(201).json(user);
});
\`\`\`

## Response Format

### Success Response
\`\`\`typescript
// Single resource
{
  "data": {
    "id": "123",
    "name": "John Doe",
    "email": "john@example.com"
  }
}

// Collection with pagination
{
  "data": [...],
  "pagination": {
    "page": 1,
    "perPage": 20,
    "total": 150,
    "totalPages": 8
  }
}
\`\`\`

### Error Response
\`\`\`typescript
// Consistent error format
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      { "field": "email", "message": "Invalid email format" }
    ]
  }
}

// Implementation
class ApiError extends Error {
  constructor(
    public code: string,
    message: string,
    public statusCode: number = 400,
    public details?: unknown
  ) {
    super(message);
  }
}

// Error handler
app.use((err, req, res, next) => {
  if (err instanceof ApiError) {
    return res.status(err.statusCode).json({
      error: {
        code: err.code,
        message: err.message,
        details: err.details,
      }
    });
  }
  // ... handle other errors
});
\`\`\`

## Status Codes

\`\`\`typescript
// Success
200 OK              // GET, PUT, PATCH success
201 Created         // POST success (include Location header)
204 No Content      // DELETE success

// Client Error
400 Bad Request     // Validation error
401 Unauthorized    // Authentication required
403 Forbidden       // Authenticated but not authorized
404 Not Found       // Resource doesn't exist
409 Conflict        // Duplicate, version conflict
422 Unprocessable   // Valid syntax, semantic error

// Server Error
500 Internal Error  // Unexpected server error
502 Bad Gateway     // Upstream service error
503 Unavailable     // Maintenance, overloaded
\`\`\`

## Pagination

\`\`\`typescript
// Query parameters
GET /users?page=2&perPage=20&sort=createdAt:desc

// Implementation
router.get('/users', async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const perPage = Math.min(parseInt(req.query.perPage) || 20, 100);

  const { users, total } = await userService.list({
    skip: (page - 1) * perPage,
    take: perPage,
    orderBy: parseSort(req.query.sort),
  });

  res.json({
    data: users,
    pagination: {
      page,
      perPage,
      total,
      totalPages: Math.ceil(total / perPage),
    }
  });
});
\`\`\`

## Authentication

\`\`\`typescript
// Bearer token in Authorization header
const authMiddleware = async (req, res, next) => {
  const token = req.headers.authorization?.replace('Bearer ', '');
  if (!token) {
    throw new ApiError('UNAUTHORIZED', 'Token required', 401);
  }

  try {
    const payload = await verifyToken(token);
    req.user = payload;
    next();
  } catch {
    throw new ApiError('UNAUTHORIZED', 'Invalid token', 401);
  }
};

// Protected route
router.get('/me', authMiddleware, async (req, res) => {
  const user = await userService.getById(req.user.id);
  res.json({ data: user });
});
\`\`\`
```

### Example 7: Git Conventions

```markdown
---
paths:
  - ".git/**"
  - ".github/**"
  - "**/*.md"
---

# Git Conventions

## Branch Naming

### Pattern
\`\`\`
<type>/<description>
\`\`\`

### Types
| Type | Use Case |
|------|----------|
| `feature/` | New features |
| `fix/` | Bug fixes |
| `refactor/` | Code refactoring |
| `docs/` | Documentation |
| `test/` | Adding tests |
| `chore/` | Maintenance |

### Examples
\`\`\`bash
feature/user-authentication
fix/login-redirect-loop
refactor/simplify-auth-logic
docs/api-documentation
test/user-service-coverage
chore/upgrade-dependencies
\`\`\`

## Commit Messages

### Format (Conventional Commits)
\`\`\`
<type>(<scope>): <subject>

<body>

<footer>
\`\`\`

### Types
| Type | Description |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation |
| `style` | Formatting (no code change) |
| `refactor` | Code change (no feature/fix) |
| `perf` | Performance improvement |
| `test` | Adding tests |
| `chore` | Maintenance |

### Examples
\`\`\`bash
# Simple
feat: add user registration

# With scope
feat(auth): add password reset flow

# With body
fix(api): handle null response from external service

The external service occasionally returns null instead of an empty
array. This fix adds a null check and defaults to an empty array.

Fixes #123

# Breaking change
feat(api)!: change user response format

BREAKING CHANGE: User responses now use camelCase instead of snake_case.
Migration guide: https://example.com/migration
\`\`\`

### Rules
- Subject: imperative mood, no period, max 50 chars
- Body: wrap at 72 chars, explain what and why
- Footer: reference issues, note breaking changes

## Pull Requests

### Title
\`\`\`
[TYPE] Brief description

Examples:
[FEAT] Add user authentication
[FIX] Resolve login redirect issue
[DOCS] Update API documentation
\`\`\`

### Description Template
\`\`\`markdown
## Summary
Brief description of changes.

## Changes
- Change 1
- Change 2

## Testing
How to test these changes.

## Screenshots (if UI)
Before/after screenshots.

## Checklist
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] No new warnings
\`\`\`

### Size Guidelines
- Small PRs (< 200 lines) preferred
- Large changes: split into smaller PRs
- Feature flags for incremental deployment

## Workflow

### Main Branches
- `main` - Production code
- `develop` - Integration branch (if using GitFlow)

### Feature Development
\`\`\`bash
# 1. Create branch from main
git checkout main
git pull
git checkout -b feature/my-feature

# 2. Make commits
git add .
git commit -m "feat: add feature"

# 3. Push and create PR
git push -u origin feature/my-feature

# 4. After approval, squash merge to main
\`\`\`

### Keeping Up to Date
\`\`\`bash
# Rebase on main (preferred for feature branches)
git fetch origin
git rebase origin/main

# Or merge (if branch is shared)
git merge origin/main
\`\`\`

## Code Review

### As Author
- Keep PRs focused and small
- Write clear descriptions
- Respond to all comments
- Request re-review after changes

### As Reviewer
- Review within 24 hours
- Be constructive and specific
- Approve when ready (don't over-nitpick)
- Use suggestions for minor fixes
```

---

## Patterns

### Pattern 1: Language-Specific Rules

```markdown
---
paths:
  - "**/*.ts"
---
TypeScript-specific rules here.
```

Separate rules per language for clarity.

### Pattern 2: Layer-Based Rules

```markdown
---
paths:
  - "src/api/**"
---
API layer rules.
```

```markdown
---
paths:
  - "src/components/**"
---
UI layer rules.
```

Different rules for different architectural layers.

### Pattern 3: Include/Exclude Patterns

```markdown
---
paths:
  - "**/*.ts"
  - "!**/*.test.ts"
  - "!**/*.d.ts"
---
```

Exclude test files from production code rules.

### Pattern 4: Code Examples in Rules

```markdown
## Naming Convention

\`\`\`typescript
// GOOD
const userCount = users.length;
function calculateTotal(items) {}

// BAD
const n = users.length;
function calc(i) {}
\`\`\`
```

Show concrete examples of what to do and not do.

### Pattern 5: Tables for Quick Reference

```markdown
## HTTP Methods

| Method | Idempotent | Use Case |
|--------|------------|----------|
| GET | Yes | Read |
| POST | No | Create |
| PUT | Yes | Replace |
\`\`\`
```

Tables are scannable and clear.

---

## Anti-Patterns

### Anti-Pattern 1: Conflicting Rules Across Files

```markdown
# typescript.md
Use semicolons at end of statements.
```

```markdown
# code-style.md
Never use semicolons.
```

**Why**: Conflicting rules cause confusion. Keep rules consistent.

### Anti-Pattern 2: Overly Broad Paths

```markdown
---
paths:
  - "**/*"
---
```

**Why**: Applies to all files. Use global CLAUDE.md instead.

### Anti-Pattern 3: Rules Without Examples

```markdown
# BAD
Use proper naming conventions.
Write clean code.
Follow best practices.
```

```markdown
# GOOD
## Naming Conventions

\`\`\`typescript
// GOOD - Descriptive
const userEmailAddress = user.email;

// BAD - Abbreviated
const uea = u.e;
\`\`\`
```

**Why**: Abstract rules are hard to follow without concrete examples.

### Anti-Pattern 4: Circular @import Chains

```markdown
# rules/a.md
@import ./b.md
```

```markdown
# rules/b.md
@import ./a.md
```

**Why**: Creates infinite loops. Keep imports linear.

### Anti-Pattern 5: Stating the Obvious

```markdown
# BAD
## Rules
- Write code that works
- Don't introduce bugs
- Make sure tests pass
```

**Why**: These are universal expectations, not project-specific rules.

### Anti-Pattern 6: Rules That Are Too Detailed

```markdown
# BAD - Micro-managing
Always put one space after commas.
Always use 2-space indentation.
Always put opening brace on same line.
```

**Why**: Use a linter (ESLint, Prettier) for formatting rules.

---

## Troubleshooting

### Rules Not Loading

**Symptoms**: Rules ignored when editing matching files

**Causes**:
1. Path patterns don't match
2. Invalid YAML frontmatter
3. File not in `.claude/rules/`

**Solution**:
```bash
# Test your glob pattern
ls src/components/**/*.tsx

# Verify frontmatter syntax
head -10 .claude/rules/react.md
```

### Rules Conflicting

**Symptoms**: Inconsistent suggestions

**Causes**:
1. Multiple rule files with overlapping paths
2. Rules contradict each other

**Solution**:
- Review all rules with overlapping paths
- Ensure consistent guidance
- Use more specific paths to avoid overlap

### Rules Not Specific Enough

**Symptoms**: Rules apply to wrong files

**Causes**:
1. Path patterns too broad
2. Missing exclusion patterns

**Solution**:
```yaml
---
paths:
  - "src/components/**/*.tsx"    # Specific
  - "!**/*.test.tsx"             # Exclude tests
  - "!**/*.stories.tsx"          # Exclude stories
---
```

---

## Cheat Sheet

### File Location
```
.claude/rules/*.md
```

### Basic Frontmatter
```yaml
---
paths:
  - "pattern/**/*.ext"
---
```

### Path Pattern Syntax
```
*           # Match any characters (not /)
**          # Match any characters (including /)
?           # Match single character
[abc]       # Match a, b, or c
{a,b}       # Match a or b
!pattern    # Exclude matches
```

### Common Path Patterns
```yaml
paths:
  - "**/*.ts"                    # All TypeScript
  - "src/**/*.tsx"               # React components
  - "**/*.test.ts"               # Test files
  - "src/api/**"                 # API layer
  - "!**/node_modules/**"        # Exclude node_modules
```

### Rule Structure
```markdown
# Category Name

## Subcategory

### Rule Name

\`\`\`typescript
// GOOD
goodExample();

// BAD
badExample();
\`\`\`
```

---

## Related Documentation

- [CLAUDE.md](./03_CLAUDE_MD.md) - For project-wide context
- [Skills](./02_SKILLS.md) - For repeatable workflows
- [Subagents](./01_SUBAGENTS.md) - For specialized tasks
- [Official Docs](https://code.claude.com/docs/memory)
