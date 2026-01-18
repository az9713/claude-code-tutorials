---
name: documentation
description: Documentation best practices for code and projects
---

# Documentation Best Practices

## Code Documentation

### When to Comment

```typescript
// ❌ Don't: Explain obvious code
// Increment counter by 1
counter++;

// ✅ Do: Explain WHY, not WHAT
// Using exponential backoff to avoid overwhelming the API during outages
const delay = Math.min(1000 * Math.pow(2, retryCount), 30000);

// ✅ Do: Document workarounds and edge cases
// Safari doesn't support `gap` in flexbox before v14.1
// Using margin as fallback for older browsers
margin: 0 -8px;

// ✅ Do: Reference external resources
// Algorithm from: https://en.wikipedia.org/wiki/Fisher-Yates_shuffle
function shuffle<T>(array: T[]): T[] {
```

### JSDoc for Public APIs

```typescript
/**
 * Calculates the total price including tax and discounts.
 *
 * @param items - Array of items in the cart
 * @param options - Calculation options
 * @param options.taxRate - Tax rate as decimal (e.g., 0.085 for 8.5%)
 * @param options.discountCode - Optional discount code to apply
 * @returns The calculated total with breakdown
 *
 * @example
 * const total = calculateTotal(items, { taxRate: 0.085 });
 * console.log(total.grandTotal); // 108.50
 *
 * @throws {ValidationError} If items array is empty
 * @throws {InvalidDiscountError} If discount code is invalid
 */
function calculateTotal(
  items: CartItem[],
  options: CalculationOptions
): TotalBreakdown {
  // ...
}
```

### Type Documentation

```typescript
/**
 * Represents a user in the system.
 */
interface User {
  /** Unique identifier (UUID v4) */
  id: string;

  /** User's email address (unique, used for login) */
  email: string;

  /** Display name shown in UI */
  displayName: string;

  /**
   * User's role determining permissions
   * @default "user"
   */
  role: 'admin' | 'user' | 'guest';

  /** ISO 8601 timestamp of account creation */
  createdAt: string;

  /**
   * Account status
   * - active: Normal account
   * - suspended: Temporarily disabled
   * - deleted: Soft-deleted, data retained
   */
  status: 'active' | 'suspended' | 'deleted';
}
```

## README Structure

```markdown
# Project Name

One-line description of what this project does.

## Features

- Feature 1
- Feature 2
- Feature 3

## Quick Start

\`\`\`bash
npm install
npm run dev
\`\`\`

## Installation

Detailed installation instructions...

## Usage

\`\`\`typescript
import { Client } from 'my-library';

const client = new Client({ apiKey: 'xxx' });
const result = await client.doSomething();
\`\`\`

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `API_KEY` | Your API key | - |
| `DEBUG` | Enable debug mode | `false` |

## API Reference

Link to detailed API docs or inline documentation.

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md)

## License

MIT
```

## API Documentation

### OpenAPI/Swagger

```yaml
openapi: 3.0.0
info:
  title: My API
  version: 1.0.0
  description: API for managing users and orders

paths:
  /users:
    get:
      summary: List all users
      tags: [Users]
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
      responses:
        '200':
          description: List of users
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
                  meta:
                    $ref: '#/components/schemas/Pagination'

    post:
      summary: Create a new user
      tags: [Users]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserRequest'
      responses:
        '201':
          description: User created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '400':
          description: Validation error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: string
          format: uuid
        email:
          type: string
          format: email
        name:
          type: string
      required: [id, email, name]
```

## Architecture Documentation

### ADR (Architecture Decision Record)

```markdown
# ADR 001: Use PostgreSQL for Primary Database

## Status
Accepted

## Context
We need a primary database for our application. Options considered:
- PostgreSQL
- MySQL
- MongoDB

## Decision
We will use PostgreSQL.

## Rationale
- Strong ACID compliance
- Excellent JSON support for flexible schemas
- Team familiarity
- Good ORM support (Prisma, TypeORM)

## Consequences
### Positive
- Reliable transactions
- Rich query capabilities
- Strong community support

### Negative
- Horizontal scaling more complex than NoSQL
- Requires more upfront schema design

## References
- PostgreSQL docs: https://postgresql.org/docs/
```

## Documentation Checklist

### Code
- [ ] Public APIs have JSDoc/TSDoc
- [ ] Complex logic is explained
- [ ] Edge cases documented
- [ ] Types are well-documented

### Project
- [ ] README with quick start
- [ ] Installation instructions
- [ ] Configuration documented
- [ ] Contributing guidelines

### API
- [ ] OpenAPI/Swagger spec
- [ ] Request/response examples
- [ ] Error codes documented
- [ ] Authentication explained

### Architecture
- [ ] System overview diagram
- [ ] Key decisions recorded (ADRs)
- [ ] Data flow documented
- [ ] Deployment process described
