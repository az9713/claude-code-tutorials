---
name: refactoring
description: Safe refactoring patterns - improve code without breaking functionality
---

# Refactoring Patterns

## The Golden Rule

> Make the change easy, then make the easy change.
> — Kent Beck

**Always:**
1. Have tests before refactoring
2. Make small, incremental changes
3. Run tests after each change
4. Commit frequently

## Common Refactoring Patterns

### Extract Function

```typescript
// Before: Long function doing multiple things
function processOrder(order) {
  // Validate
  if (!order.items.length) throw new Error('Empty order');
  if (!order.customer) throw new Error('No customer');

  // Calculate total
  let total = 0;
  for (const item of order.items) {
    total += item.price * item.quantity;
  }

  // Apply discount
  if (order.customer.tier === 'premium') {
    total *= 0.9;
  }

  return { ...order, total };
}

// After: Small, focused functions
function processOrder(order) {
  validateOrder(order);
  const total = calculateTotal(order.items);
  const discountedTotal = applyCustomerDiscount(total, order.customer);
  return { ...order, total: discountedTotal };
}

function validateOrder(order) {
  if (!order.items.length) throw new Error('Empty order');
  if (!order.customer) throw new Error('No customer');
}

function calculateTotal(items) {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}

function applyCustomerDiscount(total, customer) {
  return customer.tier === 'premium' ? total * 0.9 : total;
}
```

### Replace Conditional with Polymorphism

```typescript
// Before: Switch statement
function getShippingCost(order) {
  switch (order.shippingMethod) {
    case 'standard': return 5.99;
    case 'express': return 15.99;
    case 'overnight': return 29.99;
    default: throw new Error('Unknown shipping method');
  }
}

// After: Strategy pattern
const shippingStrategies = {
  standard: { cost: 5.99, days: 5 },
  express: { cost: 15.99, days: 2 },
  overnight: { cost: 29.99, days: 1 },
};

function getShippingCost(order) {
  const strategy = shippingStrategies[order.shippingMethod];
  if (!strategy) throw new Error('Unknown shipping method');
  return strategy.cost;
}
```

### Introduce Parameter Object

```typescript
// Before: Too many parameters
function createUser(name, email, age, country, city, zip, role, department) {
  // ...
}

// After: Parameter object
interface CreateUserParams {
  name: string;
  email: string;
  age: number;
  address: { country: string; city: string; zip: string };
  work: { role: string; department: string };
}

function createUser(params: CreateUserParams) {
  // ...
}
```

### Replace Magic Numbers/Strings

```typescript
// Before: Magic values
if (user.age >= 18) { /* ... */ }
if (order.status === 'PNDG') { /* ... */ }

// After: Named constants
const LEGAL_AGE = 18;
const OrderStatus = {
  PENDING: 'PNDG',
  APPROVED: 'APVD',
  REJECTED: 'RJCT',
} as const;

if (user.age >= LEGAL_AGE) { /* ... */ }
if (order.status === OrderStatus.PENDING) { /* ... */ }
```

### Extract Class

```typescript
// Before: Class doing too much
class Order {
  // Order data
  items: Item[];
  customer: Customer;

  // Shipping logic (should be separate)
  calculateShipping() { /* ... */ }
  getDeliveryDate() { /* ... */ }
  validateAddress() { /* ... */ }

  // Payment logic (should be separate)
  processPayment() { /* ... */ }
  validateCard() { /* ... */ }
}

// After: Single responsibility
class Order {
  items: Item[];
  customer: Customer;
  shipping: ShippingService;
  payment: PaymentService;
}

class ShippingService {
  calculate(order: Order) { /* ... */ }
  getDeliveryDate(order: Order) { /* ... */ }
  validateAddress(address: Address) { /* ... */ }
}
```

## Refactoring Safely

### The Mikado Method

For large refactorings:
1. Try to make the change
2. If it breaks, note what needs to change first
3. Revert
4. Fix prerequisites first
5. Repeat until change is easy

### Strangler Fig Pattern

For replacing legacy code:
1. Build new implementation alongside old
2. Route new features to new code
3. Gradually migrate existing features
4. Remove old code when fully migrated

## Code Smells to Watch For

| Smell | Refactoring |
|-------|-------------|
| Long function | Extract function |
| Long parameter list | Introduce parameter object |
| Duplicate code | Extract function, pull up to parent |
| Feature envy | Move method to appropriate class |
| Data clumps | Extract class |
| Primitive obsession | Replace with value object |
| Switch statements | Replace with polymorphism |
| Parallel inheritance | Collapse hierarchy |

## Before You Refactor

- [ ] Tests exist and pass
- [ ] I understand the current behavior
- [ ] I have a clear goal for the refactoring
- [ ] The refactoring is scoped and time-boxed
- [ ] I can make incremental progress
