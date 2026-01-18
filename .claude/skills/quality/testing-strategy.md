---
name: testing-strategy
description: Testing pyramid, strategies, and best practices
---

# Testing Strategy

## The Testing Pyramid

```
        /\
       /  \
      / E2E \       Few, slow, expensive
     /------\
    /  Integ  \     Some, medium speed
   /----------\
  /    Unit    \    Many, fast, cheap
 /--------------\
```

## Unit Tests

### What to Test

- Pure functions and business logic
- Edge cases and boundary conditions
- Error handling
- State transformations

### Best Practices

```typescript
// Good test structure: Arrange-Act-Assert
describe('calculateDiscount', () => {
  it('applies 10% discount for orders over $100', () => {
    // Arrange
    const order = { total: 150, items: [] };

    // Act
    const result = calculateDiscount(order);

    // Assert
    expect(result).toBe(135);
  });

  // Test edge cases
  it('returns original amount for orders exactly $100', () => {
    const order = { total: 100, items: [] };
    expect(calculateDiscount(order)).toBe(100);
  });

  // Test error conditions
  it('throws error for negative amounts', () => {
    const order = { total: -50, items: [] };
    expect(() => calculateDiscount(order)).toThrow('Invalid amount');
  });
});
```

### Naming Conventions

```typescript
// Describe what, not how
it('returns empty array when no items match filter', () => {});

// Include the condition and expected outcome
it('throws ValidationError when email format is invalid', () => {});

// Be specific
it('calculates tax at 8.5% for California addresses', () => {});
```

## Integration Tests

### API Testing

```typescript
import request from 'supertest';
import { app } from '../app';
import { db } from '../db';

describe('POST /users', () => {
  beforeEach(async () => {
    await db.users.deleteAll();
  });

  it('creates a new user with valid data', async () => {
    const response = await request(app)
      .post('/users')
      .send({ email: 'test@example.com', name: 'Test User' })
      .expect(201);

    expect(response.body).toMatchObject({
      email: 'test@example.com',
      name: 'Test User',
    });
    expect(response.body.id).toBeDefined();
  });

  it('returns 400 for invalid email', async () => {
    const response = await request(app)
      .post('/users')
      .send({ email: 'invalid', name: 'Test' })
      .expect(400);

    expect(response.body.error).toContain('email');
  });

  it('returns 409 for duplicate email', async () => {
    await db.users.create({ email: 'test@example.com', name: 'Existing' });

    await request(app)
      .post('/users')
      .send({ email: 'test@example.com', name: 'New' })
      .expect(409);
  });
});
```

### Database Testing

```typescript
describe('UserRepository', () => {
  let repo: UserRepository;

  beforeAll(async () => {
    await db.migrate.latest();
  });

  beforeEach(async () => {
    await db.users.truncate();
    repo = new UserRepository(db);
  });

  afterAll(async () => {
    await db.destroy();
  });

  it('finds user by email', async () => {
    await repo.create({ email: 'test@example.com', name: 'Test' });

    const user = await repo.findByEmail('test@example.com');

    expect(user).not.toBeNull();
    expect(user?.name).toBe('Test');
  });
});
```

## E2E Tests

```typescript
import { test, expect } from '@playwright/test';

test.describe('User Authentication', () => {
  test('user can log in with valid credentials', async ({ page }) => {
    await page.goto('/login');

    await page.fill('[data-testid="email"]', 'user@example.com');
    await page.fill('[data-testid="password"]', 'password123');
    await page.click('[data-testid="login-button"]');

    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('[data-testid="welcome-message"]'))
      .toContainText('Welcome');
  });

  test('shows error for invalid credentials', async ({ page }) => {
    await page.goto('/login');

    await page.fill('[data-testid="email"]', 'wrong@example.com');
    await page.fill('[data-testid="password"]', 'wrongpassword');
    await page.click('[data-testid="login-button"]');

    await expect(page.locator('[data-testid="error-message"]'))
      .toBeVisible();
  });
});
```

## Test Doubles

### Mocks vs Stubs vs Spies

```typescript
// Stub: Returns predefined data
const userService = {
  getUser: jest.fn().mockResolvedValue({ id: 1, name: 'Test' })
};

// Mock: Verifies interactions
const emailService = { send: jest.fn() };
await sendWelcomeEmail(user);
expect(emailService.send).toHaveBeenCalledWith(
  user.email,
  expect.stringContaining('Welcome')
);

// Spy: Wraps real implementation
const spy = jest.spyOn(userService, 'getUser');
await doSomething();
expect(spy).toHaveBeenCalledTimes(1);
spy.mockRestore(); // Restore original
```

### When to Mock

```typescript
// ✅ Mock external services
jest.mock('../services/stripe');

// ✅ Mock slow operations
jest.mock('../services/emailService');

// ✅ Mock non-deterministic functions
jest.spyOn(Date, 'now').mockReturnValue(1640995200000);
jest.spyOn(Math, 'random').mockReturnValue(0.5);

// ❌ Don't mock what you're testing
// ❌ Don't mock simple utilities
// ❌ Don't over-mock (test becomes meaningless)
```

## Code Coverage

```bash
# Generate coverage report
npm test -- --coverage

# Coverage thresholds in jest.config.js
module.exports = {
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
};
```

### Coverage Guidelines

- **80%+ coverage** is a good target
- **100% coverage ≠ bug-free** - focus on meaningful tests
- Cover critical paths thoroughly
- Don't test implementation details

## Testing Checklist

- [ ] Unit tests for business logic
- [ ] Integration tests for API endpoints
- [ ] E2E tests for critical user flows
- [ ] Edge cases covered
- [ ] Error handling tested
- [ ] Tests are readable and maintainable
- [ ] No flaky tests
- [ ] CI runs all tests
