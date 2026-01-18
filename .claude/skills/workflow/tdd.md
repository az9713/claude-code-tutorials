---
name: tdd
description: Test-Driven Development workflow - Red, Green, Refactor
---

# Test-Driven Development (TDD)

## The TDD Cycle

```
┌─────────────────────────────────────────────┐
│                                             │
│    ┌─────┐    ┌───────┐    ┌──────────┐    │
│    │ RED │ → │ GREEN │ → │ REFACTOR │    │
│    └─────┘    └───────┘    └──────────┘    │
│       ↑                          │          │
│       └──────────────────────────┘          │
│                                             │
└─────────────────────────────────────────────┘
```

### 1. RED: Write a Failing Test

Write the test FIRST, before any implementation:

```typescript
// ❌ Test fails - function doesn't exist yet
describe('calculateDiscount', () => {
  it('applies 10% discount for orders over $100', () => {
    expect(calculateDiscount(150)).toBe(135);
  });
});
```

### 2. GREEN: Make It Pass

Write the MINIMUM code to make the test pass:

```typescript
// ✅ Simplest implementation that passes
function calculateDiscount(amount: number): number {
  if (amount > 100) {
    return amount * 0.9;
  }
  return amount;
}
```

### 3. REFACTOR: Improve the Code

Clean up while keeping tests green:

```typescript
// ✨ Refactored with constants and clarity
const DISCOUNT_THRESHOLD = 100;
const DISCOUNT_RATE = 0.1;

function calculateDiscount(amount: number): number {
  if (amount <= DISCOUNT_THRESHOLD) return amount;
  return amount * (1 - DISCOUNT_RATE);
}
```

## TDD Best Practices

### Test Structure (Arrange-Act-Assert)

```typescript
it('should apply discount for premium users', () => {
  // Arrange
  const user = createUser({ tier: 'premium' });
  const order = createOrder({ total: 100 });

  // Act
  const result = applyUserDiscount(user, order);

  // Assert
  expect(result.total).toBe(85);
});
```

### Test Naming

Use descriptive names that document behavior:

```typescript
// ❌ Bad
it('test1', () => {});
it('works', () => {});

// ✅ Good
it('returns empty array when no items match filter', () => {});
it('throws ValidationError when email format is invalid', () => {});
```

### One Assertion Per Test (Generally)

```typescript
// ❌ Multiple unrelated assertions
it('processes order', () => {
  expect(order.status).toBe('processed');
  expect(inventory.count).toBe(99);
  expect(email.sent).toBe(true);
});

// ✅ Focused assertions
it('sets order status to processed', () => {
  expect(order.status).toBe('processed');
});

it('decrements inventory count', () => {
  expect(inventory.count).toBe(99);
});
```

## When to Use TDD

**Good candidates:**
- Business logic with clear rules
- Data transformations
- Utility functions
- API endpoints
- State management

**Consider alternatives for:**
- UI layout/styling
- Exploratory prototypes
- Third-party integrations
- Performance optimizations

## TDD Workflow with Claude

1. Describe the feature/function you need
2. I'll write failing tests first
3. Review and adjust test cases
4. I'll implement to make tests pass
5. We refactor together
6. Repeat for edge cases

## Test Doubles

```typescript
// Stub: Returns canned data
const userService = {
  getUser: jest.fn().mockReturnValue({ id: 1, name: 'Test' })
};

// Mock: Verifies interactions
const emailService = { send: jest.fn() };
expect(emailService.send).toHaveBeenCalledWith('user@example.com');

// Spy: Wraps real implementation
const spy = jest.spyOn(console, 'log');
```
