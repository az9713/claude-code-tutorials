# Tutorial 08: Claude Code Development Workflows

## Table of Contents

1. [Introduction](#1-introduction)
2. [Workflow Fundamentals](#2-workflow-fundamentals)
3. [Feature Development Workflow](#3-feature-development-workflow)
4. [Bug Fixing Workflow](#4-bug-fixing-workflow)
5. [Refactoring Workflow](#5-refactoring-workflow)
6. [Hand-Holding Examples](#6-hand-holding-examples)
7. [Real-World Examples](#7-real-world-examples)
8. [Workflow Automation](#8-workflow-automation)
9. [Git Workflows](#9-git-workflows)
10. [Testing Workflows](#10-testing-workflows)
11. [Documentation Workflows](#11-documentation-workflows)
12. [Team Workflows](#12-team-workflows)
13. [Workflow Templates](#13-workflow-templates)
14. [Pros and Cons](#14-pros-and-cons)
15. [Quick Reference](#15-quick-reference)

---

## 1. Introduction

### What Are Development Workflows?

Development workflows are structured processes that guide how you interact with Claude Code to accomplish coding tasks. They define the sequence of steps, checkpoints, and verification points that ensure consistent, high-quality results.

### Why Workflows Matter

**Without Workflows:**
```
You: "Add user authentication"
Claude: *makes changes*
You: "It's broken"
Claude: *makes more changes*
You: "Now something else is broken"
# Cycle continues...
```

**With Workflows:**
```
You: "Add user authentication using the feature development workflow"
Claude:
1. Clarifies requirements
2. Creates implementation plan
3. Implements with tests
4. Verifies everything works
5. Documents changes
# Predictable, high-quality results
```

### Core Workflow Principles

1. **Plan Before Code** - Understand requirements before implementation
2. **Verify Continuously** - Check work at each step
3. **Document As You Go** - Capture decisions and rationale
4. **Test Everything** - Prove that code works
5. **Review Before Commit** - Quality gates prevent issues

### When to Use Formal Workflows

| Task Complexity | Recommended Approach |
|-----------------|---------------------|
| Simple fix (typo, config) | Direct execution |
| Medium task (new function) | Lightweight workflow |
| Large feature | Full workflow with planning |
| Critical system change | Full workflow + review |

---

## 2. Workflow Fundamentals

### The Plan-Build-Review-Fix Cycle

Every development task follows this fundamental cycle:

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│    ┌──────────┐    ┌──────────┐    ┌──────────┐   │
│    │   PLAN   │───▶│  BUILD   │───▶│  REVIEW  │   │
│    └──────────┘    └──────────┘    └──────────┘   │
│         ▲                               │          │
│         │          ┌──────────┐         │          │
│         └──────────│   FIX    │◀────────┘          │
│                    └──────────┘                    │
│                         │                          │
│                         ▼                          │
│                    ┌──────────┐                    │
│                    │   DONE   │                    │
│                    └──────────┘                    │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Phase 1: Plan

**Objective:** Understand what needs to be done and how to do it.

```bash
# Start planning phase
You: "I need to add email notifications. Let's plan this out."

Claude: "I'll help plan the email notification feature. Let me ask some clarifying questions:

1. What events should trigger notifications?
2. What email service will we use (SendGrid, SES, SMTP)?
3. Should users be able to configure notification preferences?
4. What templates do we need?

Once I understand these, I'll create a detailed implementation plan."
```

**Planning Outputs:**
- Clear requirements list
- Technical approach
- File changes needed
- Testing strategy
- Risk assessment

### Phase 2: Build

**Objective:** Implement the planned changes systematically.

```bash
# Execute the plan
You: "Proceed with the implementation plan"

Claude: "I'll implement the email notification system following our plan.

Starting with Task 1: Set up email service configuration...
[Creates configuration files]

Task 1 complete. Moving to Task 2: Create notification templates...
[Creates template files]

Task 2 complete. Moving to Task 3: Implement notification service...
[Implements service class with tests]

All implementation tasks complete. Ready for review."
```

**Build Best Practices:**
- Implement one component at a time
- Write tests alongside code
- Commit logical units of work
- Document non-obvious decisions

### Phase 3: Review

**Objective:** Verify the implementation meets requirements.

```bash
# Review implementation
You: "Review what we've built"

Claude: "I'll conduct a comprehensive review:

## Code Review Checklist

✅ All planned features implemented
✅ Tests written and passing (12 tests, 100% pass rate)
✅ Error handling in place
✅ Configuration externalized
⚠️ Minor: Could add rate limiting
✅ Documentation updated

## Test Results
$ npm test -- --grep 'email'
  EmailService
    ✓ sends welcome email (45ms)
    ✓ handles template rendering (12ms)
    ✓ retries on failure (102ms)
    ...

## Recommendations
1. Consider adding rate limiting before production
2. Add monitoring for email delivery rates

Overall: Ready for merge with minor improvements suggested."
```

### Phase 4: Fix

**Objective:** Address issues found during review.

```bash
# Address review findings
You: "Add the rate limiting you mentioned"

Claude: "I'll add rate limiting to the email service.

[Implements rate limiter]
[Adds tests for rate limiting]
[Updates documentation]

Rate limiting added. Re-running review...

✅ All previous checks still pass
✅ Rate limiting implemented (100 emails/minute default)
✅ New tests passing

Ready for final approval."
```

### Workflow State Management

Claude Code tracks workflow state to ensure nothing is missed:

```bash
# Check current workflow state
You: "Where are we in the workflow?"

Claude: "Current Workflow Status:

Feature: Email Notifications
Phase: Review (3 of 4)
Progress: 85%

Completed:
✅ Planning (requirements, design)
✅ Building (service, templates, tests)
🔄 Reviewing (in progress)

Remaining:
- Address rate limiting
- Final verification
- Create PR

Blockers: None"
```

---

## 3. Feature Development Workflow

### Complete Feature Lifecycle

```
Requirements → Design → Implement → Test → Review → Deploy
```

### Step 1: Requirements Gathering

```bash
You: "I need to add a user dashboard feature"

Claude: "I'll help you build a user dashboard. Let me understand the requirements:

**Functional Requirements:**
1. What data should the dashboard display?
2. Should it have real-time updates?
3. What user actions are needed (filters, exports)?
4. Are there role-based views?

**Technical Requirements:**
5. What's your frontend framework?
6. Do you have existing API endpoints to use?
7. Any performance requirements (load time targets)?

**Constraints:**
8. Timeline constraints?
9. Must integrate with existing systems?

Please answer these so I can create a comprehensive plan."
```

### Step 2: Design Phase

```bash
You: "Dashboard shows user stats, orders, notifications. React frontend,
existing REST API. Should load in under 2 seconds."

Claude: "Based on your requirements, here's the design:

## Dashboard Architecture

### Components
```
src/components/dashboard/
├── Dashboard.tsx           # Main container
├── StatsPanel.tsx          # Key metrics
├── OrdersWidget.tsx        # Recent orders
├── NotificationsWidget.tsx # User notifications
└── hooks/
    └── useDashboardData.ts # Data fetching hook
```

### API Endpoints Needed
- GET /api/dashboard/stats - Aggregated statistics
- GET /api/dashboard/orders?limit=5 - Recent orders
- GET /api/dashboard/notifications - Unread notifications

### Data Flow
```
User Login → Dashboard Mount → Parallel API Calls → Render Widgets
                                    ↓
                            Cache in React Query
                                    ↓
                            Background Refresh (30s)
```

### Performance Strategy
- Parallel API calls (not sequential)
- React Query for caching
- Skeleton loading states
- Lazy load charts

Shall I proceed with implementation?"
```

### Step 3: Implementation

```bash
You: "Yes, proceed with implementation"

Claude: "Starting implementation. I'll work through each component:

## Task 1: Backend API Endpoints

Creating dashboard controller...

```typescript
// src/controllers/dashboardController.ts
import { Request, Response } from 'express';
import { DashboardService } from '../services/dashboardService';

export class DashboardController {
  constructor(private dashboardService: DashboardService) {}

  async getStats(req: Request, res: Response) {
    const userId = req.user.id;
    const stats = await this.dashboardService.getUserStats(userId);
    res.json(stats);
  }

  async getRecentOrders(req: Request, res: Response) {
    const userId = req.user.id;
    const limit = parseInt(req.query.limit as string) || 5;
    const orders = await this.dashboardService.getRecentOrders(userId, limit);
    res.json(orders);
  }

  async getNotifications(req: Request, res: Response) {
    const userId = req.user.id;
    const notifications = await this.dashboardService.getUnreadNotifications(userId);
    res.json(notifications);
  }
}
```

Task 1 complete. Moving to Task 2: Frontend components..."
```

### Step 4: Testing

```bash
Claude: "## Task 4: Writing Tests

### Backend Tests
```typescript
// tests/dashboard.test.ts
describe('Dashboard API', () => {
  describe('GET /api/dashboard/stats', () => {
    it('returns user statistics', async () => {
      const response = await request(app)
        .get('/api/dashboard/stats')
        .set('Authorization', `Bearer ${testToken}`);

      expect(response.status).toBe(200);
      expect(response.body).toHaveProperty('totalOrders');
      expect(response.body).toHaveProperty('totalSpent');
      expect(response.body).toHaveProperty('memberSince');
    });

    it('requires authentication', async () => {
      const response = await request(app)
        .get('/api/dashboard/stats');

      expect(response.status).toBe(401);
    });
  });
});
```

### Frontend Tests
```typescript
// src/components/dashboard/__tests__/Dashboard.test.tsx
describe('Dashboard', () => {
  it('renders loading state initially', () => {
    render(<Dashboard />);
    expect(screen.getByTestId('dashboard-skeleton')).toBeInTheDocument();
  });

  it('displays stats after loading', async () => {
    mockApi.getStats.mockResolvedValue({ totalOrders: 42 });
    render(<Dashboard />);

    await waitFor(() => {
      expect(screen.getByText('42')).toBeInTheDocument();
    });
  });

  it('handles API errors gracefully', async () => {
    mockApi.getStats.mockRejectedValue(new Error('Network error'));
    render(<Dashboard />);

    await waitFor(() => {
      expect(screen.getByText(/failed to load/i)).toBeInTheDocument();
    });
  });
});
```

Running all tests...
✅ 18 tests passed
⏱️ Total time: 3.2s"
```

### Step 5: Review and Polish

```bash
Claude: "## Implementation Review

### Checklist
✅ All API endpoints implemented and tested
✅ Frontend components with proper error handling
✅ Loading states for better UX
✅ Performance: Initial load measured at 1.4s (under 2s target)
✅ Responsive design for mobile
✅ Accessibility: ARIA labels, keyboard navigation

### Code Quality
- ESLint: 0 errors, 0 warnings
- TypeScript: No type errors
- Test coverage: 94%

### Files Changed
- 8 new files created
- 3 existing files modified
- 412 lines added

Ready to create PR or would you like any adjustments?"
```

---

## 4. Bug Fixing Workflow

### The Reproduce-Analyze-Fix-Test Cycle

```
Report → Reproduce → Analyze → Fix → Test → Verify → Deploy
```

### Step 1: Understand the Bug Report

```bash
You: "Users report that checkout fails intermittently with 'Invalid cart' error"

Claude: "I'll help investigate this checkout issue. Let me gather information:

**Bug Report Analysis:**
- Symptom: Checkout fails with 'Invalid cart' error
- Frequency: Intermittent (not consistent)
- Scope: Multiple users affected

**Questions to narrow down:**
1. When did this start? (recent deploy?)
2. Any pattern to failures? (time of day, user type, cart size?)
3. Do you have error logs or stack traces?
4. Can you share steps that sometimes reproduce it?

**Initial Investigation Plan:**
1. Search codebase for 'Invalid cart' error
2. Review recent changes to checkout flow
3. Check error logs for patterns
4. Attempt reproduction"
```

### Step 2: Reproduce the Issue

```bash
You: "Started after last week's deploy. Here's a stack trace: [trace]"

Claude: "I'll analyze the stack trace and reproduce the issue.

**Stack Trace Analysis:**
```
Error: Invalid cart
    at CartService.validateCart (src/services/cartService.ts:142)
    at CheckoutService.processCheckout (src/services/checkoutService.ts:67)
    at CheckoutController.checkout (src/controllers/checkoutController.ts:34)
```

**Searching for the error source...**

Found the validation logic:
```typescript
// src/services/cartService.ts:138-150
async validateCart(cartId: string): Promise<boolean> {
  const cart = await this.cartRepository.findById(cartId);

  if (!cart || cart.items.length === 0) {
    throw new Error('Invalid cart');  // Line 142
  }

  // Check item availability
  for (const item of cart.items) {
    const product = await this.productRepository.findById(item.productId);
    if (!product || product.stock < item.quantity) {
      throw new Error('Invalid cart');  // Same error, different cause!
    }
  }
  return true;
}
```

**Issue Identified:** Same error message for different failure conditions makes debugging hard.

**Reproduction Steps:**
1. Add item to cart
2. Wait for item to go out of stock (or manually set stock to 0)
3. Attempt checkout
4. Error occurs

Let me verify this theory..."
```

### Step 3: Root Cause Analysis

```bash
Claude: "**Root Cause Analysis Complete**

## The Problem

The 'Invalid cart' error has multiple causes but one message:
1. Cart doesn't exist
2. Cart is empty
3. Product unavailable
4. Insufficient stock

## Why It's Intermittent

Race condition between cart validation and checkout:
```
Timeline:
T0: User adds item (stock: 5)
T1: Another user buys last items (stock: 0)
T2: First user clicks checkout
T3: Validation fails - 'Invalid cart'
```

## Recent Change That Caused This

Last week's deploy added inventory check (good), but:
- Reused existing error message (bad)
- No user-friendly feedback (bad)
- No cart refresh on checkout page (bad)

## Proposed Fix

1. **Specific error messages** - Tell users exactly what's wrong
2. **Stock check on page load** - Warn before checkout attempt
3. **Graceful handling** - Offer alternatives (remove item, waitlist)

Shall I implement these fixes?"
```

### Step 4: Implement the Fix

```bash
You: "Yes, implement the fixes"

Claude: "Implementing the bug fixes:

## Fix 1: Specific Error Messages

```typescript
// src/services/cartService.ts
export class CartValidationError extends Error {
  constructor(
    message: string,
    public code: CartErrorCode,
    public details?: { itemId?: string; productName?: string }
  ) {
    super(message);
    this.name = 'CartValidationError';
  }
}

export enum CartErrorCode {
  CART_NOT_FOUND = 'CART_NOT_FOUND',
  CART_EMPTY = 'CART_EMPTY',
  PRODUCT_UNAVAILABLE = 'PRODUCT_UNAVAILABLE',
  INSUFFICIENT_STOCK = 'INSUFFICIENT_STOCK',
}

async validateCart(cartId: string): Promise<ValidationResult> {
  const cart = await this.cartRepository.findById(cartId);

  if (!cart) {
    throw new CartValidationError(
      'Cart not found',
      CartErrorCode.CART_NOT_FOUND
    );
  }

  if (cart.items.length === 0) {
    throw new CartValidationError(
      'Cart is empty',
      CartErrorCode.CART_EMPTY
    );
  }

  const issues: CartIssue[] = [];

  for (const item of cart.items) {
    const product = await this.productRepository.findById(item.productId);

    if (!product || !product.active) {
      issues.push({
        itemId: item.id,
        productName: item.name,
        code: CartErrorCode.PRODUCT_UNAVAILABLE,
        message: `${item.name} is no longer available`,
      });
    } else if (product.stock < item.quantity) {
      issues.push({
        itemId: item.id,
        productName: item.name,
        code: CartErrorCode.INSUFFICIENT_STOCK,
        message: `Only ${product.stock} of ${item.name} available`,
        availableStock: product.stock,
      });
    }
  }

  return { valid: issues.length === 0, issues };
}
```

## Fix 2: Frontend Stock Check

```typescript
// src/pages/Checkout.tsx
const CheckoutPage: React.FC = () => {
  const { cartId } = useCart();
  const [cartIssues, setCartIssues] = useState<CartIssue[]>([]);

  // Check stock on page load
  useEffect(() => {
    const checkStock = async () => {
      const result = await cartService.validateCart(cartId);
      if (!result.valid) {
        setCartIssues(result.issues);
      }
    };
    checkStock();
  }, [cartId]);

  if (cartIssues.length > 0) {
    return (
      <CartIssuesPanel
        issues={cartIssues}
        onRemoveItem={handleRemoveItem}
        onUpdateQuantity={handleUpdateQuantity}
      />
    );
  }

  // ... rest of checkout
};
```

Fixes implemented. Running tests..."
```

### Step 5: Verify the Fix

```bash
Claude: "## Verification Results

### Unit Tests
```
$ npm test -- --grep 'CartService'
  CartService
    validateCart
      ✓ returns valid for cart with available items
      ✓ returns CART_NOT_FOUND for missing cart
      ✓ returns CART_EMPTY for empty cart
      ✓ returns PRODUCT_UNAVAILABLE for discontinued items
      ✓ returns INSUFFICIENT_STOCK with available quantity
      ✓ returns multiple issues for multiple problems

  6 passing (45ms)
```

### Integration Tests
```
$ npm run test:integration -- --grep 'checkout'
  Checkout Flow
    ✓ completes successfully with valid cart
    ✓ shows specific error for out-of-stock item
    ✓ allows removing problematic items
    ✓ refreshes stock on page load

  4 passing (1.2s)
```

### Manual Verification
Tested the original reproduction steps:
1. Added item to cart ✅
2. Set stock to 0 in database ✅
3. Navigated to checkout ✅
4. **Result:** User sees 'Only 0 of [Product] available' with option to remove ✅

Bug is fixed. Ready for code review and deployment."
```

---

## 5. Refactoring Workflow

### The Identify-Plan-Change-Verify Cycle

```
Identify Problem → Plan Changes → Incremental Changes → Verify Behavior
```

### Step 1: Identify Refactoring Needs

```bash
You: "The UserService is getting hard to maintain. Can you analyze it?"

Claude: "I'll analyze the UserService for refactoring opportunities.

**Code Analysis: UserService**

```typescript
// Current state: src/services/userService.ts (487 lines)
```

## Issues Identified

### 1. God Class (High Severity)
The service handles too many responsibilities:
- User CRUD operations
- Authentication
- Password management
- Email verification
- Profile management
- Notification preferences
- Activity logging

**Metric:** 487 lines, 23 methods

### 2. Mixed Abstraction Levels (Medium Severity)
```typescript
async createUser(data: CreateUserDTO) {
  // High level: business logic
  const user = new User(data);

  // Low level: direct database calls
  await this.db.query('INSERT INTO users...');

  // Infrastructure: sending emails
  await this.sendEmail(user.email, 'Welcome!');

  // Cross-cutting: logging
  console.log(`User created: ${user.id}`);
}
```

### 3. Tight Coupling (Medium Severity)
Direct dependencies on:
- Database queries (no repository)
- Email service (instantiated internally)
- Logger (console.log)

### 4. No Dependency Injection
```typescript
class UserService {
  private emailService = new EmailService(); // Hard to test
  private db = new Database(); // Hard to mock
}
```

## Refactoring Recommendation

Split into focused services:
- `UserRepository` - Data access
- `AuthService` - Authentication
- `UserProfileService` - Profile management
- `UserNotificationService` - Notification preferences

Shall I create a detailed refactoring plan?"
```

### Step 2: Create Refactoring Plan

```bash
You: "Yes, create the plan"

Claude: "## Refactoring Plan: UserService Decomposition

### Phase 1: Preparation (No Behavior Change)
1. Add comprehensive tests for current behavior
2. Document current public API
3. Set up feature flag for gradual rollout

### Phase 2: Extract Repository Layer
**Goal:** Separate data access from business logic

```
Before:
UserService → Database (direct)

After:
UserService → UserRepository → Database
```

**Files to create:**
- `src/repositories/userRepository.ts`
- `src/repositories/interfaces/IUserRepository.ts`

**Changes:**
1. Create IUserRepository interface
2. Implement UserRepository
3. Inject repository into UserService
4. Replace direct DB calls with repository calls
5. Verify all tests still pass

### Phase 3: Extract Authentication
**Goal:** Separate auth concerns

**Files to create:**
- `src/services/authService.ts`
- `src/services/interfaces/IAuthService.ts`

**Methods to move:**
- `login()`
- `logout()`
- `validateToken()`
- `refreshToken()`
- `hashPassword()`
- `verifyPassword()`

### Phase 4: Extract Profile Service
**Goal:** Separate profile management

**Files to create:**
- `src/services/userProfileService.ts`

**Methods to move:**
- `updateProfile()`
- `uploadAvatar()`
- `getPublicProfile()`
- `updatePreferences()`

### Phase 5: Dependency Injection
**Goal:** Make services testable and loosely coupled

```typescript
// Before
class UserService {
  private emailService = new EmailService();
}

// After
class UserService {
  constructor(
    private userRepository: IUserRepository,
    private emailService: IEmailService,
    private logger: ILogger
  ) {}
}
```

### Phase 6: Cleanup
1. Remove dead code
2. Update imports across codebase
3. Update documentation
4. Remove feature flags

### Verification at Each Phase
- All existing tests pass
- No behavior changes
- Performance unchanged
- API contracts maintained

Ready to begin Phase 1?"
```

### Step 3: Incremental Implementation

```bash
You: "Start Phase 1"

Claude: "## Phase 1: Adding Tests for Current Behavior

First, I'll ensure we have tests covering all current functionality.

**Current Test Coverage:**
```
$ npm test -- --coverage src/services/userService.ts
  UserService: 62% coverage
  Uncovered: login edge cases, password reset flow, notification methods
```

**Adding Missing Tests:**

```typescript
// tests/services/userService.test.ts

describe('UserService', () => {
  // Existing tests...

  // NEW: Login edge cases
  describe('login', () => {
    it('handles account lockout after 5 failed attempts', async () => {
      const user = await createTestUser();

      for (let i = 0; i < 5; i++) {
        await userService.login(user.email, 'wrong-password');
      }

      await expect(
        userService.login(user.email, 'correct-password')
      ).rejects.toThrow('Account locked');
    });

    it('resets failed attempts after successful login', async () => {
      const user = await createTestUser();

      // 3 failed attempts
      for (let i = 0; i < 3; i++) {
        await userService.login(user.email, 'wrong').catch(() => {});
      }

      // Successful login
      await userService.login(user.email, 'correct-password');

      // Should allow more attempts
      const result = await userService.login(user.email, 'wrong').catch(e => e);
      expect(result.message).not.toContain('locked');
    });
  });

  // NEW: Password reset flow
  describe('resetPassword', () => {
    it('generates unique reset token', async () => {
      const user = await createTestUser();
      const token1 = await userService.requestPasswordReset(user.email);
      const token2 = await userService.requestPasswordReset(user.email);

      expect(token1).not.toBe(token2);
    });

    it('invalidates old tokens when new one requested', async () => {
      const user = await createTestUser();
      const oldToken = await userService.requestPasswordReset(user.email);
      await userService.requestPasswordReset(user.email);

      await expect(
        userService.resetPassword(oldToken, 'new-password')
      ).rejects.toThrow('Invalid or expired token');
    });
  });
});
```

**Updated Coverage:**
```
$ npm test -- --coverage src/services/userService.ts
  UserService: 94% coverage ✅
  All 47 tests passing
```

Phase 1 complete. Test baseline established. Proceeding to Phase 2?"
```

### Step 4: Continue Through Phases

```bash
You: "Continue with Phase 2"

Claude: "## Phase 2: Extracting Repository Layer

### Step 2.1: Create Interface

```typescript
// src/repositories/interfaces/IUserRepository.ts
export interface IUserRepository {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  create(data: CreateUserData): Promise<User>;
  update(id: string, data: Partial<User>): Promise<User>;
  delete(id: string): Promise<void>;

  // Query methods
  findByStatus(status: UserStatus): Promise<User[]>;
  findWithPagination(options: PaginationOptions): Promise<PaginatedResult<User>>;
}
```

### Step 2.2: Implement Repository

```typescript
// src/repositories/userRepository.ts
import { IUserRepository } from './interfaces/IUserRepository';
import { Database } from '../database';
import { User } from '../models/User';

export class UserRepository implements IUserRepository {
  constructor(private db: Database) {}

  async findById(id: string): Promise<User | null> {
    const row = await this.db.query(
      'SELECT * FROM users WHERE id = $1',
      [id]
    );
    return row ? User.fromRow(row) : null;
  }

  async findByEmail(email: string): Promise<User | null> {
    const row = await this.db.query(
      'SELECT * FROM users WHERE email = $1',
      [email.toLowerCase()]
    );
    return row ? User.fromRow(row) : null;
  }

  async create(data: CreateUserData): Promise<User> {
    const row = await this.db.query(
      `INSERT INTO users (id, email, name, password_hash, created_at)
       VALUES ($1, $2, $3, $4, NOW())
       RETURNING *`,
      [generateId(), data.email.toLowerCase(), data.name, data.passwordHash]
    );
    return User.fromRow(row);
  }

  // ... other methods
}
```

### Step 2.3: Update UserService

```typescript
// src/services/userService.ts
export class UserService {
  constructor(
    private userRepository: IUserRepository,
    private emailService: IEmailService
  ) {}

  async createUser(data: CreateUserDTO): Promise<User> {
    // Validate
    await this.validateNewUser(data);

    // Create via repository (not direct DB)
    const user = await this.userRepository.create({
      email: data.email,
      name: data.name,
      passwordHash: await this.hashPassword(data.password),
    });

    // Send welcome email
    await this.emailService.sendWelcome(user);

    return user;
  }

  // ... updated methods using repository
}
```

### Step 2.4: Verify

```
$ npm test
  47 tests passing ✅

$ npm run test:integration
  12 tests passing ✅

No behavior changes. Phase 2 complete.
```

Proceeding to Phase 3 (Extract Auth Service)?"
```

---

## 6. Hand-Holding Examples

### Example 1: Add a New API Endpoint

**Scenario:** Add a `GET /api/products/featured` endpoint that returns featured products.

```bash
# Step 1: Start the workflow
You: "I need to add a GET /api/products/featured endpoint"

Claude: "I'll help you add the featured products endpoint. Let me first understand your
codebase structure.

[Analyzes existing routes and controllers]

Found your API structure:
- Routes in: src/routes/
- Controllers in: src/controllers/
- Services in: src/services/

Questions:
1. How are products marked as featured? (database field? separate table?)
2. Should this endpoint support pagination?
3. Any caching requirements?"

# Step 2: Provide requirements
You: "Products have an 'isFeatured' boolean field. Return max 10, no pagination needed.
Cache for 5 minutes."

Claude: "Perfect. Here's my implementation plan:

1. Add route: GET /api/products/featured
2. Add controller method: getFeaturedProducts
3. Add service method with caching
4. Add tests
5. Update API documentation

Starting implementation...

## Step 1: Add the Route

```typescript
// src/routes/productRoutes.ts
import { Router } from 'express';
import { ProductController } from '../controllers/productController';

const router = Router();
const productController = new ProductController();

// Existing routes...
router.get('/', productController.getAll);
router.get('/:id', productController.getById);

// NEW: Featured products route
router.get('/featured', productController.getFeatured);

export default router;
```

## Step 2: Add Controller Method

```typescript
// src/controllers/productController.ts
export class ProductController {
  // ... existing methods

  async getFeatured(req: Request, res: Response, next: NextFunction) {
    try {
      const products = await this.productService.getFeaturedProducts();
      res.json({
        success: true,
        data: products,
        count: products.length,
      });
    } catch (error) {
      next(error);
    }
  }
}
```

## Step 3: Add Service Method with Caching

```typescript
// src/services/productService.ts
import { cacheManager } from '../utils/cache';

export class ProductService {
  private readonly FEATURED_CACHE_KEY = 'products:featured';
  private readonly FEATURED_CACHE_TTL = 300; // 5 minutes

  async getFeaturedProducts(): Promise<Product[]> {
    // Check cache first
    const cached = await cacheManager.get<Product[]>(this.FEATURED_CACHE_KEY);
    if (cached) {
      return cached;
    }

    // Query database
    const products = await this.productRepository.findFeatured(10);

    // Cache results
    await cacheManager.set(
      this.FEATURED_CACHE_KEY,
      products,
      this.FEATURED_CACHE_TTL
    );

    return products;
  }
}
```

## Step 4: Add Repository Method

```typescript
// src/repositories/productRepository.ts
async findFeatured(limit: number = 10): Promise<Product[]> {
  return this.db.query(`
    SELECT * FROM products
    WHERE is_featured = true AND active = true
    ORDER BY created_at DESC
    LIMIT $1
  `, [limit]);
}
```

## Step 5: Add Tests

```typescript
// tests/routes/products.test.ts
describe('GET /api/products/featured', () => {
  beforeEach(async () => {
    await cacheManager.clear();
    await createTestProducts([
      { name: 'Featured 1', isFeatured: true },
      { name: 'Featured 2', isFeatured: true },
      { name: 'Regular', isFeatured: false },
    ]);
  });

  it('returns only featured products', async () => {
    const response = await request(app)
      .get('/api/products/featured')
      .expect(200);

    expect(response.body.data).toHaveLength(2);
    expect(response.body.data.every(p => p.isFeatured)).toBe(true);
  });

  it('returns max 10 products', async () => {
    await createTestProducts(
      Array(15).fill({ name: 'Featured', isFeatured: true })
    );

    const response = await request(app)
      .get('/api/products/featured');

    expect(response.body.data).toHaveLength(10);
  });

  it('caches results for 5 minutes', async () => {
    // First request
    await request(app).get('/api/products/featured');

    // Verify cache was set
    const cached = await cacheManager.get('products:featured');
    expect(cached).toBeDefined();
  });
});
```

## Verification

```
$ npm test -- --grep 'featured'
  GET /api/products/featured
    ✓ returns only featured products (45ms)
    ✓ returns max 10 products (62ms)
    ✓ caches results for 5 minutes (23ms)

  3 passing

$ curl http://localhost:3000/api/products/featured
{
  \"success\": true,
  \"data\": [...],
  \"count\": 2
}
```

Endpoint is working. Would you like me to update the API documentation as well?"
```

### Example 2: Fix a Production Bug

**Scenario:** Users report receiving duplicate notification emails.

```bash
# Step 1: Report the bug
You: "Users are getting duplicate notification emails. It started yesterday."

Claude: "I'll investigate the duplicate email issue. Let me gather information:

**Initial Analysis:**
1. Checking recent deployments...
   - Found: Deploy yesterday at 14:30 - 'Add retry logic for email sending'

2. Searching for email sending code...
   - Found: src/services/notificationService.ts
   - Found: src/jobs/emailWorker.ts

3. Checking error logs...
   - Found: Multiple 'Email sent successfully' logs for same notification

**Likely Cause:** The new retry logic may be retrying even on success.

Let me examine the code...

```typescript
// src/jobs/emailWorker.ts (current - buggy)
async processEmail(job: EmailJob) {
  const maxRetries = 3;
  let attempt = 0;

  while (attempt < maxRetries) {
    try {
      await this.emailService.send(job.to, job.subject, job.body);
      attempt++; // BUG: Incrementing after success instead of breaking
    } catch (error) {
      attempt++;
      if (attempt >= maxRetries) {
        throw error;
      }
      await this.delay(1000 * attempt);
    }
  }
}
```

**Root Cause Found:** Missing `break` statement after successful send. The loop
continues and sends the email `maxRetries` times.

**Fix:**

```typescript
// src/jobs/emailWorker.ts (fixed)
async processEmail(job: EmailJob) {
  const maxRetries = 3;
  let attempt = 0;

  while (attempt < maxRetries) {
    try {
      await this.emailService.send(job.to, job.subject, job.body);
      return; // SUCCESS: Exit the function
    } catch (error) {
      attempt++;
      this.logger.warn(`Email send attempt ${attempt} failed`, { error, jobId: job.id });

      if (attempt >= maxRetries) {
        this.logger.error('Email send failed after all retries', { jobId: job.id });
        throw error;
      }

      await this.delay(1000 * attempt); // Exponential backoff
    }
  }
}
```

**Additional Improvements:**
1. Added logging for debugging
2. Made intent clearer with `return` instead of `break`

**Test to prevent regression:**

```typescript
describe('EmailWorker', () => {
  it('sends email only once on success', async () => {
    const sendSpy = jest.spyOn(emailService, 'send').mockResolvedValue(undefined);

    await emailWorker.processEmail(testJob);

    expect(sendSpy).toHaveBeenCalledTimes(1); // NOT 3
  });

  it('retries on failure', async () => {
    const sendSpy = jest.spyOn(emailService, 'send')
      .mockRejectedValueOnce(new Error('Temporary failure'))
      .mockResolvedValueOnce(undefined);

    await emailWorker.processEmail(testJob);

    expect(sendSpy).toHaveBeenCalledTimes(2);
  });
});
```

**Verification:**
```
$ npm test -- --grep 'EmailWorker'
  EmailWorker
    ✓ sends email only once on success
    ✓ retries on failure
    ✓ throws after max retries

  3 passing
```

Fix is ready. Shall I create a hotfix PR?"
```

### Example 3: Refactor a Module

**Scenario:** Refactor a tangled utility module into clean, focused functions.

```bash
# Step 1: Request refactoring
You: "Please refactor src/utils/helpers.ts - it's become a dumping ground"

Claude: "I'll analyze and refactor the helpers module.

**Current State Analysis:**

```typescript
// src/utils/helpers.ts (438 lines)
// Contains 34 functions covering:
// - String manipulation (8 functions)
// - Date formatting (6 functions)
// - Validation (7 functions)
// - Array utilities (5 functions)
// - Object utilities (4 functions)
// - Random mixed utilities (4 functions)
```

**Issues:**
1. No clear organization
2. Mixed responsibilities
3. Some functions duplicate lodash functionality
4. Inconsistent naming
5. No documentation

**Refactoring Plan:**

```
src/utils/
├── helpers.ts (current - 438 lines)
│
└── After refactoring:
    ├── index.ts           # Re-exports for backward compatibility
    ├── string.ts          # String utilities
    ├── date.ts            # Date formatting
    ├── validation.ts      # Validation helpers
    ├── array.ts           # Array utilities
    └── object.ts          # Object utilities
```

Starting refactoring...

## Step 1: Create Focused Modules

```typescript
// src/utils/string.ts
/**
 * String manipulation utilities
 */

/**
 * Capitalizes the first letter of a string
 */
export function capitalize(str: string): string {
  if (!str) return '';
  return str.charAt(0).toUpperCase() + str.slice(1);
}

/**
 * Converts string to URL-friendly slug
 */
export function slugify(str: string): string {
  return str
    .toLowerCase()
    .trim()
    .replace(/[^\w\s-]/g, '')
    .replace(/[\s_-]+/g, '-')
    .replace(/^-+|-+$/g, '');
}

/**
 * Truncates string to specified length with ellipsis
 */
export function truncate(str: string, length: number): string {
  if (str.length <= length) return str;
  return str.slice(0, length - 3) + '...';
}

// ... other string utilities
```

```typescript
// src/utils/date.ts
/**
 * Date formatting utilities
 */

/**
 * Formats date as relative time (e.g., '2 hours ago')
 */
export function formatRelative(date: Date): string {
  const now = new Date();
  const diffMs = now.getTime() - date.getTime();
  const diffSecs = Math.floor(diffMs / 1000);
  const diffMins = Math.floor(diffSecs / 60);
  const diffHours = Math.floor(diffMins / 60);
  const diffDays = Math.floor(diffHours / 24);

  if (diffSecs < 60) return 'just now';
  if (diffMins < 60) return `${diffMins} minute${diffMins > 1 ? 's' : ''} ago`;
  if (diffHours < 24) return `${diffHours} hour${diffHours > 1 ? 's' : ''} ago`;
  if (diffDays < 7) return `${diffDays} day${diffDays > 1 ? 's' : ''} ago`;

  return formatDate(date, 'MMM D, YYYY');
}

// ... other date utilities
```

```typescript
// src/utils/validation.ts
/**
 * Validation helper functions
 */

export function isValidEmail(email: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

export function isValidUrl(url: string): boolean {
  try {
    new URL(url);
    return true;
  } catch {
    return false;
  }
}

// ... other validation utilities
```

## Step 2: Create Backward-Compatible Index

```typescript
// src/utils/index.ts
/**
 * Utility functions - organized by category
 *
 * Import specific modules for tree-shaking:
 *   import { capitalize } from '@/utils/string';
 *
 * Or import all (backward compatible):
 *   import * as helpers from '@/utils';
 */

export * from './string';
export * from './date';
export * from './validation';
export * from './array';
export * from './object';
```

## Step 3: Update Imports Across Codebase

Found 47 files importing from helpers.ts.

For backward compatibility, keeping old import paths working:

```typescript
// src/utils/helpers.ts (now just re-exports)
/**
 * @deprecated Import from specific utility modules instead
 * @example import { capitalize } from '@/utils/string';
 */
export * from './index';
```

## Verification

```
$ npm test
  147 passing ✅ (no behavior changes)

$ npm run lint
  No errors

$ npm run build
  Build successful
  Bundle size: 124KB → 118KB (5% reduction from tree-shaking)
```

Refactoring complete. Old imports still work, but developers can now use
specific imports for better tree-shaking."
```

### Example 4: Set Up CI/CD

**Scenario:** Set up GitHub Actions CI/CD pipeline for a Node.js project.

```bash
# Step 1: Request CI/CD setup
You: "Set up CI/CD with GitHub Actions for this Node.js project"

Claude: "I'll set up a comprehensive CI/CD pipeline. Let me first understand your project:

**Project Analysis:**
- Runtime: Node.js 18
- Package manager: npm
- Test framework: Jest
- Build: TypeScript compilation
- Deployment target: Not specified

**Questions:**
1. Where do you deploy? (AWS, Vercel, Heroku, etc.)
2. Do you need staging and production environments?
3. Any required checks before merge? (coverage threshold, lint)

For now, I'll set up a complete CI pipeline. We can add CD later."

You: "We deploy to AWS ECS. Yes, staging and production."

Claude: "Perfect. Here's the complete CI/CD setup:

## GitHub Actions Workflow

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

env:
  NODE_VERSION: '18'

jobs:
  # Job 1: Code Quality
  lint:
    name: Lint & Type Check
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run ESLint
        run: npm run lint

      - name: Run TypeScript check
        run: npm run type-check

  # Job 2: Unit Tests
  test:
    name: Test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests with coverage
        run: npm test -- --coverage --coverageReporters=lcov

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          fail_ci_if_error: true

      - name: Check coverage threshold
        run: |
          COVERAGE=$(npm test -- --coverage --coverageReporters=json-summary | grep -Po '"pct":\K[0-9.]+' | head -1)
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "Coverage $COVERAGE% is below 80% threshold"
            exit 1
          fi

  # Job 3: Build
  build:
    name: Build
    runs-on: ubuntu-latest
    needs: [lint, test]
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build application
        run: npm run build

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build
          path: dist/
          retention-days: 7

  # Job 4: Security Scan
  security:
    name: Security Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run npm audit
        run: npm audit --audit-level=high

      - name: Run Snyk security scan
        uses: snyk/actions/node@master
        continue-on-error: true
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches:
      - main      # Deploy to production
      - develop   # Deploy to staging

env:
  AWS_REGION: us-east-1
  ECR_REPOSITORY: myapp
  ECS_CLUSTER_STAGING: myapp-staging
  ECS_CLUSTER_PRODUCTION: myapp-production
  ECS_SERVICE: myapp-service

jobs:
  deploy:
    name: Deploy to ${{ github.ref == 'refs/heads/main' && 'Production' || 'Staging' }}
    runs-on: ubuntu-latest
    environment: ${{ github.ref == 'refs/heads/main' && 'production' || 'staging' }}

    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build and push Docker image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG

      - name: Deploy to ECS
        env:
          CLUSTER: ${{ github.ref == 'refs/heads/main' && env.ECS_CLUSTER_PRODUCTION || env.ECS_CLUSTER_STAGING }}
        run: |
          aws ecs update-service \
            --cluster $CLUSTER \
            --service $ECS_SERVICE \
            --force-new-deployment

      - name: Wait for deployment
        env:
          CLUSTER: ${{ github.ref == 'refs/heads/main' && env.ECS_CLUSTER_PRODUCTION || env.ECS_CLUSTER_STAGING }}
        run: |
          aws ecs wait services-stable \
            --cluster $CLUSTER \
            --services $ECS_SERVICE
```

## Required Secrets

Add these to your GitHub repository settings:

```
AWS_ACCESS_KEY_ID      - AWS IAM access key
AWS_SECRET_ACCESS_KEY  - AWS IAM secret key
SNYK_TOKEN             - (Optional) Snyk API token
```

## Branch Protection Rules

Recommended settings for `main` branch:

```
✅ Require pull request reviews before merging
✅ Require status checks to pass before merging
   - lint
   - test
   - build
   - security
✅ Require branches to be up to date before merging
✅ Require conversation resolution before merging
```

## Verification

Created files:
- `.github/workflows/ci.yml`
- `.github/workflows/deploy.yml`

Test the pipeline locally with act:
```bash
# Install act (GitHub Actions local runner)
brew install act

# Test CI workflow
act pull_request
```

Pipeline is ready. Push to trigger the first run!"
```

---

## 7. Real-World Examples

### Example 1: Full-Stack Feature

**Task:** Add a commenting system to a blog application.

```bash
You: "Add a commenting system to our blog. Users should be able to comment on posts,
reply to comments, and authors should be able to moderate."

Claude: "I'll implement a complete commenting system. Here's my approach:

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                    Frontend                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │ CommentList │  │ CommentForm │  │ ModeratePanel│ │
│  └─────────────┘  └─────────────┘  └─────────────┘ │
└─────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────┐
│                    API Layer                         │
│  POST /api/posts/:id/comments                       │
│  GET  /api/posts/:id/comments                       │
│  POST /api/comments/:id/reply                       │
│  PATCH /api/comments/:id (moderate)                 │
│  DELETE /api/comments/:id                           │
└─────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────┐
│                   Database                           │
│  comments (id, post_id, user_id, parent_id,         │
│            content, status, created_at)             │
└─────────────────────────────────────────────────────┘
```

## Implementation

### 1. Database Migration

```typescript
// migrations/20240115_create_comments.ts
export async function up(knex: Knex): Promise<void> {
  await knex.schema.createTable('comments', (table) => {
    table.uuid('id').primary().defaultTo(knex.raw('gen_random_uuid()'));
    table.uuid('post_id').notNullable().references('id').inTable('posts').onDelete('CASCADE');
    table.uuid('user_id').notNullable().references('id').inTable('users');
    table.uuid('parent_id').nullable().references('id').inTable('comments').onDelete('CASCADE');
    table.text('content').notNullable();
    table.enum('status', ['pending', 'approved', 'rejected', 'spam']).defaultTo('pending');
    table.timestamps(true, true);

    // Indexes for common queries
    table.index(['post_id', 'status', 'created_at']);
    table.index(['user_id']);
    table.index(['parent_id']);
  });
}
```

### 2. Backend API

```typescript
// src/controllers/commentController.ts
export class CommentController {
  async getComments(req: Request, res: Response) {
    const { postId } = req.params;
    const { page = 1, limit = 20 } = req.query;

    const comments = await this.commentService.getCommentsForPost(postId, {
      page: Number(page),
      limit: Number(limit),
      includeReplies: true,
      status: 'approved', // Only approved for public
    });

    res.json(comments);
  }

  async createComment(req: Request, res: Response) {
    const { postId } = req.params;
    const { content } = req.body;
    const userId = req.user.id;

    // Validate content
    if (!content || content.trim().length < 2) {
      throw new ValidationError('Comment must be at least 2 characters');
    }

    if (content.length > 5000) {
      throw new ValidationError('Comment must be under 5000 characters');
    }

    const comment = await this.commentService.createComment({
      postId,
      userId,
      content: sanitizeHtml(content),
      status: await this.determineInitialStatus(userId),
    });

    // Notify post author
    await this.notificationService.notifyNewComment(comment);

    res.status(201).json(comment);
  }

  async replyToComment(req: Request, res: Response) {
    const { commentId } = req.params;
    const { content } = req.body;
    const userId = req.user.id;

    const parentComment = await this.commentService.getById(commentId);
    if (!parentComment) {
      throw new NotFoundError('Comment not found');
    }

    // Prevent deep nesting (max 2 levels)
    if (parentComment.parentId) {
      throw new ValidationError('Cannot reply to a reply');
    }

    const reply = await this.commentService.createComment({
      postId: parentComment.postId,
      userId,
      parentId: commentId,
      content: sanitizeHtml(content),
      status: await this.determineInitialStatus(userId),
    });

    // Notify parent comment author
    await this.notificationService.notifyReply(reply, parentComment);

    res.status(201).json(reply);
  }

  // Moderation endpoints
  async moderateComment(req: Request, res: Response) {
    const { commentId } = req.params;
    const { status } = req.body;

    // Check user is post author or admin
    const comment = await this.commentService.getById(commentId);
    const post = await this.postService.getById(comment.postId);

    if (post.authorId !== req.user.id && !req.user.isAdmin) {
      throw new ForbiddenError('Not authorized to moderate this comment');
    }

    const updated = await this.commentService.updateStatus(commentId, status);

    res.json(updated);
  }
}
```

### 3. Frontend Components

```tsx
// src/components/comments/CommentList.tsx
interface CommentListProps {
  postId: string;
}

export const CommentList: React.FC<CommentListProps> = ({ postId }) => {
  const { data, isLoading, fetchNextPage, hasNextPage } = useInfiniteQuery(
    ['comments', postId],
    ({ pageParam = 1 }) => commentApi.getComments(postId, pageParam),
    {
      getNextPageParam: (lastPage) => lastPage.nextPage,
    }
  );

  if (isLoading) return <CommentSkeleton count={3} />;

  const comments = data?.pages.flatMap(page => page.comments) ?? [];

  return (
    <div className="space-y-6">
      <CommentForm postId={postId} />

      {comments.length === 0 ? (
        <p className="text-gray-500">No comments yet. Be the first!</p>
      ) : (
        <>
          {comments.map(comment => (
            <Comment
              key={comment.id}
              comment={comment}
              onReply={() => setReplyingTo(comment.id)}
            />
          ))}

          {hasNextPage && (
            <button
              onClick={() => fetchNextPage()}
              className="text-blue-600 hover:underline"
            >
              Load more comments
            </button>
          )}
        </>
      )}
    </div>
  );
};
```

```tsx
// src/components/comments/Comment.tsx
interface CommentProps {
  comment: CommentType;
  onReply: () => void;
  isReply?: boolean;
}

export const Comment: React.FC<CommentProps> = ({ comment, onReply, isReply }) => {
  const { user } = useAuth();
  const [showReplyForm, setShowReplyForm] = useState(false);

  return (
    <div className={cn('border-l-2 pl-4', isReply ? 'ml-8 border-gray-200' : 'border-blue-200')}>
      <div className="flex items-start gap-3">
        <Avatar user={comment.author} size="sm" />

        <div className="flex-1">
          <div className="flex items-center gap-2">
            <span className="font-medium">{comment.author.name}</span>
            <span className="text-gray-500 text-sm">
              {formatRelative(comment.createdAt)}
            </span>
          </div>

          <p className="mt-1 text-gray-700">{comment.content}</p>

          <div className="mt-2 flex items-center gap-4 text-sm">
            {!isReply && user && (
              <button
                onClick={() => setShowReplyForm(true)}
                className="text-gray-500 hover:text-blue-600"
              >
                Reply
              </button>
            )}
          </div>

          {showReplyForm && (
            <div className="mt-3">
              <CommentForm
                postId={comment.postId}
                parentId={comment.id}
                onSuccess={() => setShowReplyForm(false)}
                onCancel={() => setShowReplyForm(false)}
              />
            </div>
          )}

          {/* Nested replies */}
          {comment.replies?.map(reply => (
            <Comment
              key={reply.id}
              comment={reply}
              onReply={() => {}}
              isReply
            />
          ))}
        </div>
      </div>
    </div>
  );
};
```

### 4. Tests

```typescript
// tests/comments.test.ts
describe('Comments Feature', () => {
  describe('API', () => {
    it('creates a comment on a post', async () => {
      const post = await createTestPost();
      const user = await createTestUser();

      const response = await request(app)
        .post(`/api/posts/${post.id}/comments`)
        .set('Authorization', `Bearer ${user.token}`)
        .send({ content: 'Great post!' });

      expect(response.status).toBe(201);
      expect(response.body.content).toBe('Great post!');
    });

    it('allows replies to comments', async () => {
      const comment = await createTestComment();
      const user = await createTestUser();

      const response = await request(app)
        .post(`/api/comments/${comment.id}/reply`)
        .set('Authorization', `Bearer ${user.token}`)
        .send({ content: 'I agree!' });

      expect(response.status).toBe(201);
      expect(response.body.parentId).toBe(comment.id);
    });

    it('prevents deep nesting', async () => {
      const reply = await createTestReply(); // Already has parentId
      const user = await createTestUser();

      const response = await request(app)
        .post(`/api/comments/${reply.id}/reply`)
        .set('Authorization', `Bearer ${user.token}`)
        .send({ content: 'Nested too deep' });

      expect(response.status).toBe(400);
    });
  });
});
```

Implementation complete. Total changes:
- 1 migration file
- 3 API endpoints
- 4 React components
- 1 service class
- 15 tests

Ready for review!"
```

### Example 2: Database Migration

**Task:** Migrate from single-tenant to multi-tenant database architecture.

```bash
You: "We need to add multi-tenancy. Each organization should have isolated data."

Claude: "I'll implement row-level multi-tenancy. Here's the approach:

## Multi-Tenancy Strategy

**Chosen approach:** Row-level isolation with `organization_id` column
- Single database, shared tables
- Each row tagged with organization
- Automatic filtering via middleware

```sql
-- Migration: Add organization support

-- Step 1: Create organizations table
CREATE TABLE organizations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(100) UNIQUE NOT NULL,
  settings JSONB DEFAULT '{}',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Step 2: Add organization_id to existing tables
ALTER TABLE users ADD COLUMN organization_id UUID REFERENCES organizations(id);
ALTER TABLE posts ADD COLUMN organization_id UUID REFERENCES organizations(id);
ALTER TABLE comments ADD COLUMN organization_id UUID REFERENCES organizations(id);
ALTER TABLE products ADD COLUMN organization_id UUID REFERENCES organizations(id);
-- ... repeat for all tenant-specific tables

-- Step 3: Create indexes for performance
CREATE INDEX idx_users_org ON users(organization_id);
CREATE INDEX idx_posts_org ON posts(organization_id);
CREATE INDEX idx_products_org ON products(organization_id);

-- Step 4: Row-Level Security Policies
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;
ALTER TABLE products ENABLE ROW LEVEL SECURITY;

-- Policy: Users can only see their organization's data
CREATE POLICY users_isolation ON users
  USING (organization_id = current_setting('app.current_org_id')::uuid);

CREATE POLICY posts_isolation ON posts
  USING (organization_id = current_setting('app.current_org_id')::uuid);
```

## Data Migration Script

```typescript
// scripts/migrate-to-multitenant.ts
async function migrateToMultiTenant() {
  const db = await getConnection();

  console.log('Starting multi-tenant migration...');

  // Step 1: Create default organization for existing data
  const [defaultOrg] = await db.query(`
    INSERT INTO organizations (name, slug)
    VALUES ('Default Organization', 'default')
    RETURNING id
  `);

  console.log(`Created default organization: ${defaultOrg.id}`);

  // Step 2: Migrate existing data
  const tables = ['users', 'posts', 'comments', 'products'];

  for (const table of tables) {
    const result = await db.query(`
      UPDATE ${table}
      SET organization_id = $1
      WHERE organization_id IS NULL
    `, [defaultOrg.id]);

    console.log(`Migrated ${result.rowCount} rows in ${table}`);
  }

  // Step 3: Make organization_id NOT NULL
  for (const table of tables) {
    await db.query(`
      ALTER TABLE ${table}
      ALTER COLUMN organization_id SET NOT NULL
    `);
  }

  console.log('Migration complete!');
}
```

## Application Layer Changes

```typescript
// src/middleware/tenantMiddleware.ts
export function tenantMiddleware(req: Request, res: Response, next: NextFunction) {
  // Extract tenant from subdomain or header
  const tenantSlug = extractTenant(req);

  if (!tenantSlug) {
    return res.status(400).json({ error: 'Tenant not specified' });
  }

  // Look up organization
  const org = await organizationService.findBySlug(tenantSlug);

  if (!org) {
    return res.status(404).json({ error: 'Organization not found' });
  }

  // Set tenant context
  req.organizationId = org.id;
  req.organization = org;

  // Set PostgreSQL session variable for RLS
  await db.query(`SET app.current_org_id = '${org.id}'`);

  next();
}

function extractTenant(req: Request): string | null {
  // Option 1: Subdomain (acme.app.com)
  const host = req.hostname;
  const subdomain = host.split('.')[0];
  if (subdomain && subdomain !== 'www' && subdomain !== 'app') {
    return subdomain;
  }

  // Option 2: Header (X-Tenant-ID)
  return req.headers['x-tenant-id'] as string || null;
}
```

```typescript
// src/repositories/baseRepository.ts
export abstract class BaseRepository<T> {
  constructor(
    protected db: Database,
    protected tableName: string
  ) {}

  // All queries automatically scoped to organization
  async findAll(organizationId: string): Promise<T[]> {
    return this.db.query(
      `SELECT * FROM ${this.tableName} WHERE organization_id = $1`,
      [organizationId]
    );
  }

  async create(organizationId: string, data: Partial<T>): Promise<T> {
    return this.db.query(
      `INSERT INTO ${this.tableName} (organization_id, ...) VALUES ($1, ...) RETURNING *`,
      [organizationId, ...Object.values(data)]
    );
  }
}
```

## Rollback Plan

```sql
-- rollback-multitenant.sql
-- WARNING: This will remove tenant isolation

ALTER TABLE users DROP COLUMN organization_id;
ALTER TABLE posts DROP COLUMN organization_id;
-- ... repeat for all tables

DROP TABLE organizations;
```

Migration ready. Test plan:
1. Run migration on staging
2. Verify existing data accessible
3. Create new organization
4. Verify data isolation
5. Run full test suite
6. Deploy to production"
```

### Example 3: API Versioning

**Task:** Implement API versioning for backward compatibility.

```bash
You: "We need to version our API. We're making breaking changes but can't break existing clients."

Claude: "I'll implement URL-based API versioning with backward compatibility.

## Versioning Strategy

```
/api/v1/users     ← Current (maintained)
/api/v2/users     ← New (breaking changes)
/api/users        ← Alias to latest stable (v1 for now)
```

## Implementation

### 1. Router Structure

```typescript
// src/routes/index.ts
import { Router } from 'express';
import v1Router from './v1';
import v2Router from './v2';

const router = Router();

// Versioned routes
router.use('/v1', v1Router);
router.use('/v2', v2Router);

// Default to v1 (update when v2 is stable)
router.use('/', v1Router);

export default router;
```

```typescript
// src/routes/v1/index.ts
import { Router } from 'express';
import { UserControllerV1 } from '../../controllers/v1/userController';

const router = Router();
const userController = new UserControllerV1();

router.get('/users', userController.list);
router.get('/users/:id', userController.getById);
router.post('/users', userController.create);

export default router;
```

```typescript
// src/routes/v2/index.ts
import { Router } from 'express';
import { UserControllerV2 } from '../../controllers/v2/userController';

const router = Router();
const userController = new UserControllerV2();

router.get('/users', userController.list);
router.get('/users/:id', userController.getById);
router.post('/users', userController.create);

export default router;
```

### 2. Version-Specific Controllers

```typescript
// src/controllers/v1/userController.ts
// Original API - maintained for backward compatibility
export class UserControllerV1 {
  async list(req: Request, res: Response) {
    const users = await userService.findAll();

    // V1 response format
    res.json({
      users: users.map(u => ({
        id: u.id,
        name: u.name,           // Deprecated: split in v2
        email: u.email,
        created: u.createdAt,   // Different key than v2
      })),
      total: users.length,
    });
  }
}
```

```typescript
// src/controllers/v2/userController.ts
// New API with breaking changes
export class UserControllerV2 {
  async list(req: Request, res: Response) {
    const { page = 1, limit = 20, sort } = req.query;
    const result = await userService.findPaginated({ page, limit, sort });

    // V2 response format (breaking changes)
    res.json({
      data: result.users.map(u => ({
        id: u.id,
        firstName: u.firstName,    // Split from 'name'
        lastName: u.lastName,      // Split from 'name'
        email: u.email,
        createdAt: u.createdAt,    // Consistent naming
        updatedAt: u.updatedAt,    // New field
      })),
      meta: {
        total: result.total,
        page: result.page,
        limit: result.limit,
        totalPages: result.totalPages,
      },
    });
  }
}
```

### 3. Version Detection Middleware

```typescript
// src/middleware/apiVersion.ts
export function apiVersionMiddleware(req: Request, res: Response, next: NextFunction) {
  // Extract version from URL
  const match = req.path.match(/^\/api\/v(\d+)/);
  const version = match ? parseInt(match[1]) : 1;

  req.apiVersion = version;

  // Add version to response headers
  res.setHeader('X-API-Version', version);

  // Deprecation warnings
  if (version === 1) {
    res.setHeader('X-API-Deprecated', 'true');
    res.setHeader('X-API-Sunset-Date', '2025-06-01');
  }

  next();
}
```

### 4. Shared Business Logic

```typescript
// src/services/userService.ts
// Shared service used by both versions
export class UserService {
  async findAll(): Promise<User[]> {
    return this.userRepository.findAll();
  }

  async findPaginated(options: PaginationOptions): Promise<PaginatedResult<User>> {
    return this.userRepository.findPaginated(options);
  }

  // Transform methods for different versions
  toV1Format(user: User): UserV1 {
    return {
      id: user.id,
      name: `${user.firstName} ${user.lastName}`,
      email: user.email,
      created: user.createdAt,
    };
  }

  toV2Format(user: User): UserV2 {
    return {
      id: user.id,
      firstName: user.firstName,
      lastName: user.lastName,
      email: user.email,
      createdAt: user.createdAt,
      updatedAt: user.updatedAt,
    };
  }
}
```

### 5. Migration Guide for Clients

```markdown
# API v2 Migration Guide

## Breaking Changes

### User Endpoints

| v1 | v2 | Description |
|----|----|-|
| `name` | `firstName`, `lastName` | Name field split |
| `created` | `createdAt` | Consistent timestamp naming |
| Array response | Paginated response | All list endpoints now paginated |

### Example Migration

**v1 Request:**
GET /api/v1/users

**v1 Response:**
{
  "users": [{ "id": "1", "name": "John Doe" }],
  "total": 1
}

**v2 Request:**
GET /api/v2/users?page=1&limit=20

**v2 Response:**
{
  "data": [{ "id": "1", "firstName": "John", "lastName": "Doe" }],
  "meta": { "total": 1, "page": 1, "limit": 20, "totalPages": 1 }
}

## Timeline

- v1 deprecated: January 2025
- v1 sunset: June 2025
- v2 becomes default: March 2025
```

API versioning implemented. Both versions are fully functional and tested."
```

### Example 4: Performance Optimization

**Task:** Optimize a slow dashboard that takes 8 seconds to load.

```bash
You: "Our admin dashboard takes 8 seconds to load. Users are complaining."

Claude: "I'll analyze and optimize the dashboard performance.

## Performance Analysis

### Step 1: Profiling

```typescript
// Added timing logs to identify bottlenecks

Dashboard Load Breakdown:
├── API: GET /api/dashboard/stats      - 3,200ms ← SLOW
├── API: GET /api/dashboard/users      - 2,100ms ← SLOW
├── API: GET /api/dashboard/revenue    - 1,800ms ← SLOW
├── API: GET /api/dashboard/activity   -   450ms
├── Frontend render                    -   350ms
└── Total                              - 7,900ms
```

### Step 2: Database Query Analysis

```sql
-- Stats query (3,200ms)
EXPLAIN ANALYZE
SELECT
  COUNT(*) as total_orders,
  SUM(amount) as revenue,
  COUNT(DISTINCT user_id) as customers
FROM orders
WHERE created_at > NOW() - INTERVAL '30 days';

-- Result: Seq Scan on orders (cost=0.00..45123.00)
-- Problem: No index on created_at, scanning 2M rows
```

## Optimizations Applied

### 1. Database Indexes

```sql
-- Add missing indexes
CREATE INDEX idx_orders_created_at ON orders(created_at DESC);
CREATE INDEX idx_orders_user_created ON orders(user_id, created_at DESC);
CREATE INDEX idx_users_created_at ON users(created_at DESC);

-- Result: Query time 3,200ms → 45ms
```

### 2. Query Optimization

```typescript
// Before: Multiple queries
const stats = await getOrderStats();      // Query 1
const revenue = await getRevenueStats();  // Query 2
const customers = await getCustomerCount(); // Query 3

// After: Single optimized query
const dashboardStats = await db.query(`
  SELECT
    COUNT(*) as total_orders,
    SUM(amount) as revenue,
    COUNT(DISTINCT user_id) as customers,
    (SELECT COUNT(*) FROM users WHERE created_at > NOW() - INTERVAL '30 days') as new_users
  FROM orders
  WHERE created_at > NOW() - INTERVAL '30 days'
`);

// Result: 3 queries → 1 query
```

### 3. Parallel API Calls

```typescript
// Before: Sequential fetches
const stats = await fetch('/api/dashboard/stats');
const users = await fetch('/api/dashboard/users');
const revenue = await fetch('/api/dashboard/revenue');

// After: Parallel fetches
const [stats, users, revenue] = await Promise.all([
  fetch('/api/dashboard/stats'),
  fetch('/api/dashboard/users'),
  fetch('/api/dashboard/revenue'),
]);

// Result: 7,100ms → 3,200ms (limited by slowest)
```

### 4. Caching Layer

```typescript
// src/services/dashboardService.ts
import { cacheManager } from '../utils/cache';

export class DashboardService {
  private CACHE_TTL = 60; // 1 minute

  async getStats(): Promise<DashboardStats> {
    const cacheKey = 'dashboard:stats';

    // Check cache
    const cached = await cacheManager.get<DashboardStats>(cacheKey);
    if (cached) return cached;

    // Fetch fresh data
    const stats = await this.fetchStats();

    // Cache for 1 minute
    await cacheManager.set(cacheKey, stats, this.CACHE_TTL);

    return stats;
  }
}

// Result: Subsequent loads 45ms → 5ms
```

### 5. Frontend Optimizations

```tsx
// Before: Render all at once
const Dashboard = () => {
  const { data, isLoading } = useDashboardData();

  if (isLoading) return <Loading />;

  return (
    <div>
      <StatsPanel data={data.stats} />
      <UserTable data={data.users} />
      <RevenueChart data={data.revenue} />
      <ActivityFeed data={data.activity} />
    </div>
  );
};

// After: Progressive loading
const Dashboard = () => {
  return (
    <div>
      <Suspense fallback={<StatsSkeleton />}>
        <StatsPanel />
      </Suspense>

      <Suspense fallback={<TableSkeleton />}>
        <UserTable />
      </Suspense>

      <Suspense fallback={<ChartSkeleton />}>
        <RevenueChart />
      </Suspense>

      <Suspense fallback={<FeedSkeleton />}>
        <ActivityFeed />
      </Suspense>
    </div>
  );
};

// Result: Perceived load time improved, content appears progressively
```

### 6. Data Aggregation Table

```sql
-- Create pre-aggregated stats table
CREATE TABLE dashboard_stats_daily (
  date DATE PRIMARY KEY,
  total_orders INTEGER,
  revenue DECIMAL(12,2),
  new_customers INTEGER,
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Scheduled job to update daily
CREATE OR REPLACE FUNCTION update_dashboard_stats()
RETURNS void AS $$
BEGIN
  INSERT INTO dashboard_stats_daily (date, total_orders, revenue, new_customers)
  SELECT
    CURRENT_DATE - 1,
    COUNT(*),
    SUM(amount),
    COUNT(DISTINCT user_id)
  FROM orders
  WHERE created_at::date = CURRENT_DATE - 1
  ON CONFLICT (date) DO UPDATE SET
    total_orders = EXCLUDED.total_orders,
    revenue = EXCLUDED.revenue,
    new_customers = EXCLUDED.new_customers,
    updated_at = NOW();
END;
$$ LANGUAGE plpgsql;

-- Result: 30-day stats query uses pre-aggregated data
-- Query time: 45ms → 3ms
```

## Results

```
Before Optimization:
├── Total load time: 7,900ms
├── Time to first content: 7,900ms
└── User experience: Poor

After Optimization:
├── Total load time: 890ms (89% improvement)
├── Time to first content: 120ms (skeleton)
├── Time to stats: 250ms
├── Time to full content: 890ms
└── User experience: Excellent
```

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| API Response | 7,100ms | 350ms | 95% |
| Frontend Render | 800ms | 120ms | 85% |
| Total Load | 7,900ms | 890ms | 89% |
| Cache Hit Load | N/A | 85ms | - |"
```

### Example 5: Security Audit

**Task:** Conduct a security audit and fix vulnerabilities.

```bash
You: "Run a security audit on our API"

Claude: "I'll conduct a comprehensive security audit.

## Security Audit Report

### 1. Authentication & Authorization

#### Finding: JWT Secret Hardcoded (CRITICAL)
```typescript
// src/auth/jwt.ts
const JWT_SECRET = 'super-secret-key-123'; // HARDCODED!
```

**Fix:**
```typescript
const JWT_SECRET = process.env.JWT_SECRET;
if (!JWT_SECRET) {
  throw new Error('JWT_SECRET environment variable required');
}
```

#### Finding: No Token Expiration Check (HIGH)
```typescript
// Before: Token never expires
const token = jwt.sign({ userId: user.id }, JWT_SECRET);

// After: Token expires in 1 hour
const token = jwt.sign(
  { userId: user.id },
  JWT_SECRET,
  { expiresIn: '1h' }
);
```

#### Finding: Missing Authorization Checks (HIGH)
```typescript
// Before: Any authenticated user can access any user's data
router.get('/users/:id', authenticate, userController.getById);

// After: Check ownership or admin role
router.get('/users/:id', authenticate, authorize('user:read'), userController.getById);

// Authorization middleware
function authorize(permission: string) {
  return (req: Request, res: Response, next: NextFunction) => {
    const { userId } = req.params;

    // Allow if accessing own data
    if (req.user.id === userId) return next();

    // Allow if admin
    if (req.user.role === 'admin') return next();

    // Deny otherwise
    return res.status(403).json({ error: 'Forbidden' });
  };
}
```

### 2. Input Validation

#### Finding: SQL Injection Vulnerability (CRITICAL)
```typescript
// Before: Direct string interpolation
const user = await db.query(`SELECT * FROM users WHERE email = '${email}'`);

// After: Parameterized query
const user = await db.query('SELECT * FROM users WHERE email = $1', [email]);
```

#### Finding: XSS Vulnerability (HIGH)
```typescript
// Before: Raw user input in response
res.send(`<h1>Welcome, ${user.name}</h1>`);

// After: Sanitize output
import { escape } from 'html-escaper';
res.send(`<h1>Welcome, ${escape(user.name)}</h1>`);
```

#### Finding: No Request Validation (MEDIUM)
```typescript
// Before: No validation
app.post('/users', (req, res) => {
  const user = await createUser(req.body); // Accepts anything!
});

// After: Schema validation
import { z } from 'zod';

const createUserSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8).max(100),
  name: z.string().min(1).max(100),
});

app.post('/users', validate(createUserSchema), (req, res) => {
  const user = await createUser(req.body); // Validated!
});
```

### 3. Rate Limiting

#### Finding: No Rate Limiting (HIGH)
```typescript
// Added rate limiting
import rateLimit from 'express-rate-limit';

// General API rate limit
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per window
  message: { error: 'Too many requests, please try again later' },
});

// Strict limit for auth endpoints
const authLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 5, // 5 attempts per hour
  message: { error: 'Too many login attempts' },
});

app.use('/api', apiLimiter);
app.use('/api/auth', authLimiter);
```

### 4. Sensitive Data Exposure

#### Finding: Password in API Response (CRITICAL)
```typescript
// Before: Returns full user object including password
res.json(user);

// After: Exclude sensitive fields
res.json({
  id: user.id,
  email: user.email,
  name: user.name,
  // password NOT included
});

// Better: Use DTOs
class UserResponseDTO {
  static fromUser(user: User) {
    return {
      id: user.id,
      email: user.email,
      name: user.name,
    };
  }
}
```

#### Finding: Verbose Error Messages (MEDIUM)
```typescript
// Before: Exposes internal details
catch (error) {
  res.status(500).json({ error: error.message, stack: error.stack });
}

// After: Generic error for production
catch (error) {
  logger.error('Internal error', { error });
  res.status(500).json({
    error: process.env.NODE_ENV === 'production'
      ? 'Internal server error'
      : error.message
  });
}
```

### 5. Security Headers

```typescript
// Added security headers with Helmet
import helmet from 'helmet';

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: [\"'self'\"],
      scriptSrc: [\"'self'\"],
      styleSrc: [\"'self'\", \"'unsafe-inline'\"],
      imgSrc: [\"'self'\", 'data:', 'https:'],
    },
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
  },
}));
```

### 6. Dependencies Audit

```bash
$ npm audit
found 12 vulnerabilities (3 critical, 5 high, 4 moderate)

$ npm audit fix
fixed 10 vulnerabilities

# Remaining 2 require manual update (breaking changes)
$ npm update lodash@latest  # CVE-2021-23337
$ npm update axios@latest   # CVE-2023-45857
```

## Audit Summary

| Category | Critical | High | Medium | Low |
|----------|----------|------|--------|-----|
| Auth | 1 | 2 | 0 | 0 |
| Input Validation | 1 | 1 | 1 | 0 |
| Rate Limiting | 0 | 1 | 0 | 0 |
| Data Exposure | 1 | 0 | 1 | 0 |
| Headers | 0 | 0 | 1 | 0 |
| Dependencies | 0 | 2 | 1 | 3 |
| **Total** | **3** | **6** | **4** | **3** |

All critical and high vulnerabilities have been fixed."
```

---

## 8. Workflow Automation

### Hooks for Automatic Workflows

```bash
# .claude/hooks/pre-commit.sh
#!/bin/bash
# Automatically run before commits

echo "Running pre-commit checks..."

# Lint staged files
npm run lint:staged

# Run affected tests
npm test -- --findRelatedTests $(git diff --cached --name-only)

# Check for secrets
git secrets --scan

echo "Pre-commit checks passed!"
```

### Skills for Complex Workflows

```markdown
# .claude/skills/feature-workflow.md

## Feature Development Workflow

When the user asks to implement a feature, follow this workflow:

### Phase 1: Requirements (ALWAYS START HERE)
1. Ask clarifying questions about:
   - Functional requirements
   - Non-functional requirements (performance, security)
   - User stories and acceptance criteria
   - Edge cases

### Phase 2: Design
1. Propose architecture/approach
2. Identify files to create/modify
3. Plan database changes if needed
4. Get user approval before proceeding

### Phase 3: Implementation
1. Create database migrations first
2. Implement backend (API, services)
3. Implement frontend (components, state)
4. Write tests alongside code

### Phase 4: Verification
1. Run all tests
2. Manual testing checklist
3. Performance check
4. Security review

### Phase 5: Documentation
1. Update API docs
2. Add code comments for complex logic
3. Update README if needed
```

### Agent Orchestration

```bash
# Using subagents for parallel tasks
You: "Implement user authentication with email verification"

Claude: "I'll orchestrate this using specialized agents:

**Spawning parallel agents for independent tasks:**

Agent 1: Database Setup
- Create users table migration
- Create email_verifications table migration
- Add indexes

Agent 2: Backend Services
- AuthService implementation
- EmailVerificationService implementation
- JWT token management

Agent 3: API Endpoints
- POST /auth/register
- POST /auth/login
- POST /auth/verify-email
- POST /auth/resend-verification

Agent 4: Email Templates
- Welcome email template
- Verification email template
- Password reset template

**Coordinating agents...**

[Agent 1 complete] Database migrations ready
[Agent 4 complete] Email templates ready
[Agent 2 complete] Services implemented
[Agent 3 complete] API endpoints created

**Integration phase:**
- Connecting services to endpoints
- Running integration tests
- Verifying email flow end-to-end

All agents complete. Authentication system ready for review."
```

### Automated Quality Gates

```typescript
// .claude/workflows/quality-gates.ts
export const qualityGates = {
  preCommit: {
    lint: { required: true, command: 'npm run lint' },
    typeCheck: { required: true, command: 'npm run type-check' },
    unitTests: { required: true, command: 'npm test' },
  },

  preMerge: {
    integrationTests: { required: true, command: 'npm run test:integration' },
    coverage: { required: true, threshold: 80 },
    securityScan: { required: true, command: 'npm audit' },
  },

  preRelease: {
    e2eTests: { required: true, command: 'npm run test:e2e' },
    performanceTest: { required: false, command: 'npm run test:perf' },
    manualReview: { required: true },
  },
};
```

---

## 9. Git Workflows

### Branching Strategy

```
main (production)
  │
  ├── develop (integration)
  │     │
  │     ├── feature/user-auth
  │     ├── feature/dashboard
  │     └── feature/notifications
  │
  ├── hotfix/critical-bug
  │
  └── release/v2.0.0
```

### Branch Naming Conventions

```bash
# Features
feature/user-authentication
feature/JIRA-123-add-search

# Bug fixes
bugfix/login-redirect
bugfix/JIRA-456-cart-total

# Hotfixes (production issues)
hotfix/security-patch
hotfix/payment-timeout

# Releases
release/v2.0.0
release/2024-q1
```

### Commit Message Format

```bash
# Format: type(scope): description

# Types
feat:     New feature
fix:      Bug fix
docs:     Documentation
style:    Formatting (no code change)
refactor: Code restructuring
test:     Adding tests
chore:    Maintenance

# Examples
feat(auth): add email verification flow
fix(cart): resolve race condition in checkout
docs(api): update authentication endpoints
refactor(users): extract validation to separate service
test(orders): add integration tests for order creation
```

### Pull Request Workflow

```bash
# 1. Create feature branch
git checkout -b feature/user-dashboard

# 2. Make changes and commit
git add .
git commit -m "feat(dashboard): add user statistics panel"

# 3. Push and create PR
git push -u origin feature/user-dashboard
gh pr create --title "feat(dashboard): add user statistics panel" --body "
## Summary
- Adds StatsPanel component showing user metrics
- Implements /api/dashboard/stats endpoint
- Adds caching for performance

## Testing
- [ ] Unit tests pass
- [ ] Manual testing on staging
- [ ] Performance verified (<2s load time)

## Screenshots
[Attach screenshots]
"

# 4. Address review feedback
git add .
git commit -m "fix(dashboard): address PR feedback"
git push

# 5. Merge after approval
gh pr merge --squash
```

### Handling Merge Conflicts

```bash
# Scenario: Conflict during PR merge

# 1. Fetch latest changes
git fetch origin

# 2. Rebase onto target branch
git rebase origin/main

# 3. Resolve conflicts
# Edit conflicted files, then:
git add <resolved-files>
git rebase --continue

# 4. Force push (only on feature branch!)
git push --force-with-lease

# The PR will automatically update
```

### Git Workflow with Claude Code

```bash
You: "Create a commit for the changes I've made"

Claude: "I'll create a commit for your changes.

Analyzing changes:
- Modified: src/services/userService.ts (added validation)
- Modified: src/controllers/userController.ts (error handling)
- Added: tests/services/userService.test.ts

Suggested commit message:
```
feat(users): add input validation and error handling

- Add email and password validation to UserService
- Improve error messages in UserController
- Add comprehensive unit tests for validation logic
```

Creating commit...
✅ Commit created: abc123f

Would you like me to push this to the remote?"
```

---

## 10. Testing Workflows

### Test-Driven Development (TDD)

```bash
# TDD Cycle: Red → Green → Refactor

You: "Add a function to calculate order totals with discounts using TDD"

Claude: "I'll implement this using TDD.

## Step 1: Write Failing Test (Red)

```typescript
// tests/services/orderService.test.ts
describe('calculateTotal', () => {
  it('calculates total without discount', () => {
    const items = [
      { price: 10, quantity: 2 },
      { price: 5, quantity: 3 },
    ];

    expect(calculateTotal(items)).toBe(35);
  });

  it('applies percentage discount', () => {
    const items = [{ price: 100, quantity: 1 }];
    const discount = { type: 'percentage', value: 10 };

    expect(calculateTotal(items, discount)).toBe(90);
  });

  it('applies fixed discount', () => {
    const items = [{ price: 100, quantity: 1 }];
    const discount = { type: 'fixed', value: 15 };

    expect(calculateTotal(items, discount)).toBe(85);
  });

  it('never returns negative total', () => {
    const items = [{ price: 10, quantity: 1 }];
    const discount = { type: 'fixed', value: 50 };

    expect(calculateTotal(items, discount)).toBe(0);
  });
});
```

Running tests...
❌ 4 failing (function doesn't exist)

## Step 2: Implement to Pass (Green)

```typescript
// src/services/orderService.ts
interface OrderItem {
  price: number;
  quantity: number;
}

interface Discount {
  type: 'percentage' | 'fixed';
  value: number;
}

export function calculateTotal(items: OrderItem[], discount?: Discount): number {
  // Calculate subtotal
  const subtotal = items.reduce(
    (sum, item) => sum + (item.price * item.quantity),
    0
  );

  // Apply discount
  let total = subtotal;
  if (discount) {
    if (discount.type === 'percentage') {
      total = subtotal * (1 - discount.value / 100);
    } else {
      total = subtotal - discount.value;
    }
  }

  // Never negative
  return Math.max(0, total);
}
```

Running tests...
✅ 4 passing

## Step 3: Refactor (if needed)

Code is clean and readable. No refactoring needed.

TDD cycle complete!"
```

### Integration Testing Workflow

```typescript
// tests/integration/checkout.test.ts
describe('Checkout Flow', () => {
  let app: Express;
  let db: Database;
  let testUser: User;

  beforeAll(async () => {
    // Set up test database
    db = await createTestDatabase();
    app = createApp(db);
  });

  beforeEach(async () => {
    // Clean state for each test
    await db.query('TRUNCATE users, orders, cart_items CASCADE');
    testUser = await createTestUser(db);
  });

  afterAll(async () => {
    await db.close();
  });

  it('completes full checkout flow', async () => {
    // 1. Add items to cart
    await request(app)
      .post('/api/cart/items')
      .set('Authorization', `Bearer ${testUser.token}`)
      .send({ productId: 'prod-1', quantity: 2 })
      .expect(201);

    // 2. Apply discount code
    await request(app)
      .post('/api/cart/discount')
      .set('Authorization', `Bearer ${testUser.token}`)
      .send({ code: 'SAVE10' })
      .expect(200);

    // 3. Create order
    const orderResponse = await request(app)
      .post('/api/orders')
      .set('Authorization', `Bearer ${testUser.token}`)
      .send({
        shippingAddress: testAddress,
        paymentMethod: 'card',
      })
      .expect(201);

    expect(orderResponse.body.status).toBe('pending');
    expect(orderResponse.body.discount).toBe(10);

    // 4. Verify cart is cleared
    const cartResponse = await request(app)
      .get('/api/cart')
      .set('Authorization', `Bearer ${testUser.token}`);

    expect(cartResponse.body.items).toHaveLength(0);
  });
});
```

### End-to-End Testing Workflow

```typescript
// tests/e2e/checkout.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Checkout', () => {
  test.beforeEach(async ({ page }) => {
    // Login
    await page.goto('/login');
    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'password123');
    await page.click('button[type="submit"]');
    await expect(page).toHaveURL('/dashboard');
  });

  test('user can complete checkout', async ({ page }) => {
    // Browse products
    await page.goto('/products');
    await page.click('[data-testid="product-1"] button');

    // Verify cart badge updates
    await expect(page.locator('[data-testid="cart-count"]')).toHaveText('1');

    // Go to cart
    await page.click('[data-testid="cart-icon"]');
    await expect(page).toHaveURL('/cart');

    // Proceed to checkout
    await page.click('text=Checkout');

    // Fill shipping info
    await page.fill('[name="address"]', '123 Test St');
    await page.fill('[name="city"]', 'Test City');
    await page.fill('[name="zip"]', '12345');

    // Fill payment info (test card)
    await page.fill('[name="cardNumber"]', '4242424242424242');
    await page.fill('[name="expiry"]', '12/25');
    await page.fill('[name="cvc"]', '123');

    // Complete order
    await page.click('text=Place Order');

    // Verify success
    await expect(page).toHaveURL(/\/orders\/\w+/);
    await expect(page.locator('h1')).toContainText('Order Confirmed');
  });
});
```

### Test Coverage Workflow

```bash
# Check coverage before committing
You: "Check test coverage for the auth module"

Claude: "Running coverage analysis for auth module:

```
$ npm test -- --coverage --collectCoverageFrom='src/auth/**/*.ts'

-----------------------|---------|----------|---------|---------|
File                   | % Stmts | % Branch | % Funcs | % Lines |
-----------------------|---------|----------|---------|---------|
src/auth/             |   87.5  |    78.9  |   90.0  |   87.5  |
  authService.ts      |   92.3  |    85.7  |  100.0  |   92.3  |
  jwtService.ts       |   85.0  |    75.0  |   80.0  |   85.0  |
  passwordService.ts  |   83.3  |    71.4  |   87.5  |   83.3  |
-----------------------|---------|----------|---------|---------|

Uncovered lines:
- jwtService.ts: 45-48 (token refresh edge case)
- passwordService.ts: 67-72 (bcrypt error handling)
```

Coverage is at 87.5%, above the 80% threshold.

To improve coverage, I recommend adding tests for:
1. JWT token refresh when original token is malformed
2. Bcrypt comparison failure scenarios

Want me to add these tests?"
```

---

## 11. Documentation Workflows

### Code Documentation

```typescript
/**
 * Calculates the total price for an order including discounts and tax.
 *
 * @param items - Array of order items with price and quantity
 * @param options - Optional calculation options
 * @param options.discount - Discount to apply (percentage or fixed)
 * @param options.taxRate - Tax rate as decimal (e.g., 0.08 for 8%)
 * @returns The calculated total, never negative
 *
 * @example
 * // Simple calculation
 * calculateTotal([{ price: 10, quantity: 2 }])
 * // Returns: 20
 *
 * @example
 * // With percentage discount
 * calculateTotal(
 *   [{ price: 100, quantity: 1 }],
 *   { discount: { type: 'percentage', value: 10 } }
 * )
 * // Returns: 90
 *
 * @example
 * // With tax
 * calculateTotal(
 *   [{ price: 100, quantity: 1 }],
 *   { taxRate: 0.08 }
 * )
 * // Returns: 108
 */
export function calculateTotal(
  items: OrderItem[],
  options?: CalculationOptions
): number {
  // Implementation
}
```

### API Documentation

```yaml
# openapi.yaml
openapi: 3.0.0
info:
  title: E-Commerce API
  version: 2.0.0
  description: API for managing orders, products, and users

paths:
  /api/orders:
    post:
      summary: Create a new order
      tags: [Orders]
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateOrderRequest'
            example:
              items:
                - productId: "prod-123"
                  quantity: 2
              shippingAddress:
                street: "123 Main St"
                city: "New York"
                zip: "10001"
              discountCode: "SAVE10"
      responses:
        '201':
          description: Order created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
        '400':
          description: Invalid request
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
        '401':
          description: Unauthorized
```

### README Documentation

```markdown
# Project Name

Brief description of what this project does.

## Quick Start

```bash
# Install dependencies
npm install

# Set up environment
cp .env.example .env

# Run database migrations
npm run db:migrate

# Start development server
npm run dev
```

## Features

- Feature 1: Description
- Feature 2: Description
- Feature 3: Description

## Architecture

```
src/
├── controllers/   # HTTP request handlers
├── services/      # Business logic
├── repositories/  # Data access
├── models/        # Data models
└── utils/         # Shared utilities
```

## API Reference

See [API Documentation](./docs/api.md) for detailed endpoint documentation.

## Testing

```bash
npm test              # Run unit tests
npm run test:int      # Run integration tests
npm run test:e2e      # Run end-to-end tests
```

## Deployment

See [Deployment Guide](./docs/deployment.md).
```

### Documentation Generation Workflow

```bash
You: "Generate API documentation from the codebase"

Claude: "I'll analyze the codebase and generate API documentation.

Scanning routes and controllers...

Found 24 endpoints across 6 controllers:
- AuthController: 5 endpoints
- UserController: 6 endpoints
- ProductController: 5 endpoints
- OrderController: 5 endpoints
- CartController: 3 endpoints

Generating OpenAPI specification...

```yaml
# Generated: docs/openapi.yaml
openapi: 3.0.0
info:
  title: Application API
  version: 1.0.0

paths:
  /api/auth/register:
    post:
      summary: Register new user
      # ... (auto-generated from code analysis)
```

Generating markdown documentation...

Created:
- docs/api/README.md (overview)
- docs/api/auth.md (authentication endpoints)
- docs/api/users.md (user endpoints)
- docs/api/products.md (product endpoints)
- docs/api/orders.md (order endpoints)
- docs/api/cart.md (cart endpoints)

Documentation generated successfully!"
```

---

## 12. Team Workflows

### Code Standards Enforcement

```javascript
// .eslintrc.js
module.exports = {
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended',
    'prettier',
  ],
  rules: {
    // Enforce consistent code style
    'no-console': 'warn',
    'no-unused-vars': 'error',
    '@typescript-eslint/explicit-function-return-type': 'error',
    '@typescript-eslint/no-explicit-any': 'error',

    // Naming conventions
    '@typescript-eslint/naming-convention': [
      'error',
      { selector: 'interface', format: ['PascalCase'] },
      { selector: 'class', format: ['PascalCase'] },
      { selector: 'function', format: ['camelCase'] },
      { selector: 'variable', format: ['camelCase', 'UPPER_CASE'] },
    ],
  },
};
```

### Code Review Checklist

```markdown
# Code Review Checklist

## Functionality
- [ ] Code does what it's supposed to do
- [ ] Edge cases are handled
- [ ] Error handling is appropriate

## Code Quality
- [ ] Code is readable and self-documenting
- [ ] No unnecessary complexity
- [ ] DRY principle followed
- [ ] SOLID principles followed

## Testing
- [ ] Unit tests added/updated
- [ ] Tests cover happy path and edge cases
- [ ] Tests are readable and maintainable

## Security
- [ ] No hardcoded secrets
- [ ] Input validation present
- [ ] SQL injection prevented
- [ ] XSS prevented

## Performance
- [ ] No N+1 queries
- [ ] Appropriate indexes exist
- [ ] No memory leaks

## Documentation
- [ ] Complex logic is commented
- [ ] API changes documented
- [ ] README updated if needed
```

### Onboarding Workflow

```markdown
# Developer Onboarding

## Day 1: Setup

### Environment Setup
```bash
# Clone repository
git clone https://github.com/company/project.git
cd project

# Install dependencies
npm install

# Copy environment template
cp .env.example .env.local
# Edit .env.local with provided credentials

# Set up database
docker-compose up -d postgres
npm run db:migrate
npm run db:seed

# Verify setup
npm test
npm run dev
```

### Required Tools
- Node.js 18+
- Docker Desktop
- VS Code with recommended extensions
- Claude Code CLI

## Day 2-3: Codebase Tour

### Architecture Overview
[Link to architecture docs]

### Key Files to Review
1. `src/app.ts` - Application entry point
2. `src/routes/` - API routing
3. `src/services/` - Business logic
4. `src/models/` - Data models

### First Task
Complete the "Hello World" ticket:
1. Add GET /api/hello endpoint
2. Write tests
3. Create PR
4. Get code review
```

### Team Communication Workflow

```bash
# Using Claude Code for team standards
You: "Review this PR against our team standards"

Claude: "I'll review PR #234 against your team standards.

## Team Standards Review

### Naming Conventions ✅
- Functions use camelCase
- Classes use PascalCase
- Constants use UPPER_SNAKE_CASE

### Code Organization ✅
- Files are in correct directories
- No circular dependencies
- Follows module structure

### Testing ⚠️
- Unit tests present
- Missing integration test for API endpoint
- Coverage: 76% (below 80% threshold)

### Documentation ✅
- JSDoc comments on public methods
- README updated

### Security ✅
- No hardcoded credentials
- Input validation present
- Using parameterized queries

## Summary
2 issues to address before merge:
1. Add integration test for new endpoint
2. Increase test coverage to 80%

Would you like me to help add the missing tests?"
```

---

## 13. Workflow Templates

### Feature Development Template

```markdown
# Feature: [Feature Name]

## Requirements
- [ ] Requirement 1
- [ ] Requirement 2
- [ ] Requirement 3

## Design
### Architecture
[Describe approach]

### Database Changes
[List migrations needed]

### API Changes
[List new/modified endpoints]

## Implementation Checklist
- [ ] Database migration
- [ ] Backend service
- [ ] API endpoints
- [ ] Frontend components
- [ ] Unit tests
- [ ] Integration tests
- [ ] Documentation

## Testing Plan
- [ ] Happy path scenarios
- [ ] Edge cases
- [ ] Error handling
- [ ] Performance testing

## Deployment
- [ ] Feature flag configured
- [ ] Rollback plan documented
- [ ] Monitoring alerts set up
```

### Bug Fix Template

```markdown
# Bug: [Bug Title]

## Report
- **Reporter:** [Name]
- **Date:** [Date]
- **Severity:** [Critical/High/Medium/Low]

## Description
[What's happening]

## Expected Behavior
[What should happen]

## Reproduction Steps
1. Step 1
2. Step 2
3. Bug occurs

## Root Cause
[Analysis of why this happens]

## Fix
[Description of the fix]

## Testing
- [ ] Reproduction test added
- [ ] Fix verified
- [ ] Regression tests pass

## Deployment
- [ ] Hotfix or regular release?
- [ ] Notify affected users?
```

### Refactoring Template

```markdown
# Refactoring: [Component/Module Name]

## Current State
- Lines of code: X
- Complexity: [High/Medium/Low]
- Test coverage: X%

## Issues
1. Issue 1
2. Issue 2
3. Issue 3

## Proposed Changes
1. Change 1
2. Change 2
3. Change 3

## Migration Plan
### Phase 1: [Description]
- [ ] Task 1
- [ ] Task 2

### Phase 2: [Description]
- [ ] Task 1
- [ ] Task 2

## Verification
- [ ] All existing tests pass
- [ ] No behavior changes
- [ ] Performance maintained
- [ ] Documentation updated
```

### Release Template

```markdown
# Release: v[X.Y.Z]

## Release Date
[Target date]

## Changes
### Features
- Feature 1 (#PR)
- Feature 2 (#PR)

### Bug Fixes
- Fix 1 (#PR)
- Fix 2 (#PR)

### Breaking Changes
- Change 1 (migration guide: [link])

## Pre-Release Checklist
- [ ] All PRs merged
- [ ] Version bumped
- [ ] Changelog updated
- [ ] E2E tests passing
- [ ] Security scan clean
- [ ] Performance benchmarks acceptable

## Release Steps
1. Create release branch
2. Run final tests
3. Build release artifacts
4. Deploy to staging
5. Smoke test
6. Deploy to production
7. Monitor for issues
8. Announce release

## Rollback Plan
[Steps to rollback if issues occur]

## Post-Release
- [ ] Announce to team
- [ ] Update documentation
- [ ] Close related tickets
- [ ] Schedule retrospective
```

---

## 14. Pros and Cons

### Benefits of Structured Workflows

| Benefit | Description |
|---------|-------------|
| **Consistency** | Same process every time reduces errors |
| **Quality** | Built-in checkpoints catch issues early |
| **Traceability** | Clear history of decisions and changes |
| **Onboarding** | New team members can follow established patterns |
| **Automation** | Workflows can be automated and enforced |
| **Communication** | Clear expectations for all parties |

### Workflow Overhead

| Overhead | Mitigation |
|----------|------------|
| **Time investment** | Start with lightweight workflows, add complexity as needed |
| **Learning curve** | Document workflows clearly, provide examples |
| **Rigidity** | Allow flexibility for simple tasks |
| **Documentation burden** | Automate documentation where possible |
| **Tool fatigue** | Choose tools that integrate well |

### When to Skip Formal Workflows

**Skip workflows for:**
- Typo fixes
- Config changes
- One-line fixes
- Documentation updates
- Dependency updates

**Use workflows for:**
- New features
- Bug fixes affecting users
- Refactoring
- Performance work
- Security changes
- Database migrations

### Workflow Maturity Model

```
Level 1: Ad Hoc
- No defined process
- Results vary by developer
- Difficult to predict outcomes

Level 2: Basic
- Simple checklist exists
- Manual verification
- Some consistency

Level 3: Defined
- Documented workflows
- Consistent process
- Peer review

Level 4: Managed
- Metrics tracked
- Automated gates
- Continuous improvement

Level 5: Optimized
- Data-driven decisions
- Automated workflows
- Predictable outcomes
```

---

## 15. Quick Reference

### Workflow Selection Decision Tree

```
Is this a simple change (typo, config)?
├── Yes → Direct fix, no formal workflow
└── No → Continue...

Is this a bug fix?
├── Yes → Bug Fix Workflow (Reproduce → Analyze → Fix → Test)
└── No → Continue...

Is this a new feature?
├── Yes → Feature Workflow (Plan → Build → Review → Deploy)
└── No → Continue...

Is this improving existing code?
├── Yes → Refactoring Workflow (Identify → Plan → Change → Verify)
└── No → Use general Plan → Build → Review → Fix cycle
```

### Command Quick Reference

```bash
# Git Workflow
git checkout -b feature/name    # Create feature branch
git add -p                      # Interactive staging
git commit -m "type: message"   # Commit with message
git push -u origin feature/name # Push and track
gh pr create                    # Create PR

# Testing
npm test                        # Run tests
npm test -- --coverage          # With coverage
npm test -- --watch             # Watch mode
npm run test:e2e                # E2E tests

# Code Quality
npm run lint                    # Run linter
npm run lint -- --fix           # Auto-fix issues
npm run type-check              # TypeScript check
npm audit                       # Security scan

# Claude Code
claude                          # Start interactive session
claude "task description"       # One-shot task
claude --continue               # Continue last session
```

### Workflow Phase Checklist

```markdown
## Planning Phase
- [ ] Requirements understood
- [ ] Questions answered
- [ ] Approach decided
- [ ] Risks identified

## Building Phase
- [ ] Code implemented
- [ ] Tests written
- [ ] Documentation updated
- [ ] Self-review done

## Review Phase
- [ ] All tests pass
- [ ] Code quality checks pass
- [ ] Security scan clean
- [ ] Peer review approved

## Deployment Phase
- [ ] Staging tested
- [ ] Production deployed
- [ ] Monitoring verified
- [ ] Stakeholders notified
```

### Common Workflow Patterns

| Pattern | When to Use | Key Steps |
|---------|-------------|-----------|
| **TDD** | New logic | Test → Code → Refactor |
| **Spike** | Uncertainty | Experiment → Learn → Discard |
| **Hotfix** | Production issue | Fix → Test → Deploy immediately |
| **Feature Flag** | Risky change | Deploy disabled → Enable gradually |
| **Blue-Green** | Zero downtime | Deploy parallel → Switch traffic |
| **Canary** | Gradual rollout | Deploy to % → Monitor → Expand |

### Troubleshooting Common Issues

| Issue | Solution |
|-------|----------|
| Merge conflicts | Rebase frequently, communicate with team |
| Flaky tests | Isolate test data, mock external services |
| Slow CI | Parallelize tests, cache dependencies |
| Review bottleneck | Set SLAs, pair programming |
| Scope creep | Stick to original requirements, create follow-up tickets |
| Integration issues | Test early, use feature flags |

---

## Conclusion

Effective development workflows are the foundation of consistent, high-quality software delivery. By following structured processes:

1. **Start with the basics** - Plan → Build → Review → Fix
2. **Choose the right workflow** for the task complexity
3. **Automate** repetitive steps with hooks and CI/CD
4. **Document** decisions and rationale
5. **Iterate** and improve workflows based on experience

Claude Code can assist at every stage, from planning and implementation to review and documentation. Use these workflows as starting points and adapt them to your team's needs.

Remember: The goal is not process for its own sake, but predictable, high-quality outcomes that make development enjoyable and sustainable.
