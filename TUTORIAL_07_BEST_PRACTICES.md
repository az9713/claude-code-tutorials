# Tutorial 07: Claude Code Best Practices

A comprehensive guide to maximizing productivity, maintaining code quality, and working effectively with Claude Code.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Project Setup Best Practices](#2-project-setup-best-practices)
3. [Prompting Best Practices](#3-prompting-best-practices)
4. [Context Management](#4-context-management)
5. [Code Quality Practices](#5-code-quality-practices)
6. [Hand-Holding User Journey](#6-hand-holding-user-journey)
7. [Real-World Scenarios](#7-real-world-scenarios)
8. [Cost Optimization](#8-cost-optimization)
9. [Security Best Practices](#9-security-best-practices)
10. [Team Collaboration](#10-team-collaboration)
11. [Performance Optimization](#11-performance-optimization)
12. [Common Anti-Patterns](#12-common-anti-patterns)
13. [Checklist: Before Starting Work](#13-checklist-before-starting-work)
14. [Quick Reference](#14-quick-reference)

---

## 1. Introduction

### Why Best Practices Matter

Claude Code is a powerful AI-assisted development tool that can dramatically increase your productivity—but only when used correctly. Following best practices ensures:

- **Consistent Results**: Predictable, high-quality outputs every time
- **Time Savings**: Less iteration and rework
- **Cost Efficiency**: Reduced token usage and API costs
- **Code Quality**: Maintainable, secure, well-tested code
- **Team Alignment**: Shared understanding across your organization

### Common Mistakes to Avoid

Before diving into best practices, here are the most common mistakes developers make:

| Mistake | Consequence | Better Approach |
|---------|-------------|-----------------|
| Vague prompts | Irrelevant or incomplete code | Be specific about requirements |
| No CLAUDE.md file | Claude lacks project context | Document project conventions |
| Accepting code blindly | Bugs and security issues | Always review generated code |
| Ignoring context limits | Degraded performance | Use /compact and /clear strategically |
| One massive prompt | Confusion and errors | Break into smaller tasks |
| Not testing changes | Broken functionality | Test incrementally |

### Maximizing Productivity with Claude Code

The developers who get the most from Claude Code follow a simple pattern:

```
Prepare → Prompt → Review → Test → Iterate
```

1. **Prepare**: Set up project context (CLAUDE.md, .claudeignore)
2. **Prompt**: Write clear, specific requests
3. **Review**: Examine every change before accepting
4. **Test**: Verify changes work as expected
5. **Iterate**: Refine based on results

This tutorial will teach you to excel at each step.

---

## 2. Project Setup Best Practices

### CLAUDE.md File Importance

The `CLAUDE.md` file is your project's "instruction manual" for Claude. It's automatically read at the start of every session.

**Why it matters:**
- Provides consistent context across all sessions
- Reduces repetitive explanations
- Ensures Claude follows your conventions
- Saves tokens by pre-loading essential information

**Create a comprehensive CLAUDE.md:**

```markdown
# Project: E-Commerce Platform

## Overview
A Next.js e-commerce platform with TypeScript, PostgreSQL, and Stripe integration.

## Tech Stack
- Frontend: Next.js 14, React 18, TailwindCSS
- Backend: Next.js API Routes, Prisma ORM
- Database: PostgreSQL 15
- Payments: Stripe
- Testing: Jest, React Testing Library, Playwright

## Project Structure
```
src/
├── app/           # Next.js App Router pages
├── components/    # React components (PascalCase)
├── lib/           # Utilities and helpers
├── hooks/         # Custom React hooks
├── types/         # TypeScript definitions
├── services/      # Business logic
└── tests/         # Test files mirror src/ structure
```

## Conventions
- Use TypeScript strict mode
- Components: PascalCase (UserProfile.tsx)
- Utilities: camelCase (formatCurrency.ts)
- Tests: [filename].test.ts
- Use named exports, not default exports

## Commands
- `npm run dev` - Start development server
- `npm run test` - Run Jest tests
- `npm run test:e2e` - Run Playwright tests
- `npm run lint` - ESLint check
- `npm run build` - Production build

## Important Notes
- Never commit .env files
- All API routes require authentication middleware
- Use Prisma migrations for database changes
- Follow the existing error handling patterns in lib/errors.ts
```

### Project Structure Documentation

Document your project structure so Claude understands where things belong:

**Good Practice:**
```markdown
## Directory Purposes

### /src/components
Reusable UI components. Each component has:
- ComponentName.tsx - The component
- ComponentName.test.tsx - Unit tests
- ComponentName.stories.tsx - Storybook stories
- index.ts - Named export

### /src/services
Business logic separated from UI. Services:
- Are pure functions when possible
- Handle errors consistently
- Are fully unit tested
- Never import from components/
```

**Bad Practice:**
```markdown
## Structure
- src/ has code
- tests/ has tests
```

### .claudeignore Configuration

The `.claudeignore` file prevents Claude from accessing sensitive or irrelevant files:

```gitignore
# Secrets - NEVER let Claude access these
.env
.env.*
*.pem
*.key
credentials.json
secrets/

# Large generated files (waste context)
node_modules/
dist/
build/
.next/
coverage/

# Binary files (Claude can't read these)
*.png
*.jpg
*.pdf
*.zip

# Irrelevant files
*.log
.DS_Store
Thumbs.db

# Lock files (usually not needed)
package-lock.json
yarn.lock
pnpm-lock.yaml
```

### Initial Context Setup

When starting a new project with Claude Code:

**Step 1: Create CLAUDE.md**
```bash
claude "Create a CLAUDE.md file for this project based on the existing
codebase. Include tech stack, conventions, commands, and project structure."
```

**Step 2: Create .claudeignore**
```bash
claude "Create a .claudeignore file. Exclude secrets, node_modules,
build artifacts, and binary files."
```

**Step 3: Verify setup**
```bash
claude "Summarize what you understand about this project from CLAUDE.md"
```

---

## 3. Prompting Best Practices

### Be Specific and Clear

The quality of Claude's output directly correlates with the quality of your input.

**Bad Prompt:**
```
Add a login feature
```

**Good Prompt:**
```
Add a login feature with these requirements:
1. Email/password authentication
2. Use our existing AuthService in src/services/auth.ts
3. Create a LoginForm component in src/components/auth/
4. Include form validation (email format, password min 8 chars)
5. Show loading state during authentication
6. Redirect to /dashboard on success
7. Display error messages on failure
8. Follow our existing form patterns in src/components/forms/
```

### Provide Context

Always provide relevant context, even if it seems obvious:

**Bad Prompt:**
```
Fix the bug
```

**Good Prompt:**
```
Fix the null pointer bug in UserService.getProfile():
- Error: "Cannot read property 'email' of undefined"
- Occurs when user.profile is null
- See error log: src/logs/error-2024-01-15.log line 234
- The user object comes from our Prisma query in src/services/user.ts
- Expected behavior: Return default profile if none exists
```

### Break Down Complex Tasks

Large tasks should be decomposed into smaller, manageable steps:

**Bad Approach:**
```
Build an entire e-commerce checkout system
```

**Good Approach:**
```
Let's build the checkout system in phases:

Phase 1: Cart Management
- Create CartContext for state management
- Build AddToCart button component
- Build CartSummary component

Phase 2: Checkout Form
- Create CheckoutForm with shipping address
- Add form validation
- Integrate with our address validation API

Phase 3: Payment Integration
- Integrate Stripe Elements
- Handle payment processing
- Create order confirmation

Let's start with Phase 1. Create the CartContext following our
existing context pattern in src/contexts/AuthContext.tsx
```

### Use Examples

Show Claude what you want through examples:

**Good Prompt with Example:**
```
Create a new API endpoint for user preferences. Follow the pattern
of our existing endpoints.

Example from src/app/api/users/route.ts:
```typescript
export async function GET(request: NextRequest) {
  try {
    const session = await getServerSession(authOptions);
    if (!session) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }
    const users = await prisma.user.findMany();
    return NextResponse.json(users);
  } catch (error) {
    return handleApiError(error);
  }
}
```

Create src/app/api/preferences/route.ts with GET and PUT methods
following this same pattern.
```

### Avoid Ambiguity

Remove any room for misinterpretation:

**Ambiguous:**
```
Make it faster
```

**Clear:**
```
Optimize the ProductList component render performance:
1. Implement React.memo for ProductCard components
2. Use useMemo for the filtered products calculation
3. Add virtualization for lists > 100 items using react-window
4. Target: Render 1000 products in < 100ms
```

### Good vs Bad Prompt Examples

| Scenario | Bad Prompt | Good Prompt |
|----------|------------|-------------|
| Bug fix | "Fix the error" | "Fix the TypeError in UserService.ts line 45 where user.profile can be undefined. Add null check and return empty profile object as fallback." |
| New feature | "Add search" | "Add search functionality to ProductList: 1) Search input with debounce (300ms), 2) Filter by name and description, 3) Show 'No results' message, 4) Preserve existing filters" |
| Refactor | "Clean up the code" | "Refactor AuthService.ts: 1) Extract token validation to separate function, 2) Replace callbacks with async/await, 3) Add TypeScript types for all parameters" |
| Testing | "Add tests" | "Add unit tests for CartService.calculateTotal(): 1) Empty cart returns 0, 2) Single item calculates correctly, 3) Multiple items sum correctly, 4) Discounts apply correctly" |

---

## 4. Context Management

### When to Use /clear

Use `/clear` to completely reset the conversation when:

- Starting a completely new, unrelated task
- The conversation has become confused or off-track
- You've finished a major feature and are starting fresh
- Claude seems to be hallucinating or making repeated errors

```bash
# After completing user authentication feature
/clear

# Starting fresh on a new feature
"Now let's work on the payment integration..."
```

### When to Use /compact

Use `/compact` to summarize and compress context when:

- The conversation is getting long but you need continuity
- You're seeing "context limit" warnings
- The task requires remembering previous decisions
- You want to preserve important context while freeing space

```bash
# Long debugging session, need to continue but running low on context
/compact

# Claude will summarize the conversation and continue
"Based on our debugging, we identified the issue is in the
caching layer. Let's now implement the fix..."
```

### Context Window Awareness

Claude has a limited context window. Be aware of what consumes tokens:

**High Token Consumers:**
- Large file contents
- Long conversation histories
- Verbose error logs
- Entire codebases

**Strategies to Manage Context:**

1. **Be selective about file reads:**
   ```
   # Bad: "Read the entire src/ directory"
   # Good: "Read src/services/auth.ts - I need to understand the login flow"
   ```

2. **Summarize when sharing logs:**
   ```
   # Bad: Paste entire 1000-line log file
   # Good: "Error on line 234: TypeError: Cannot read 'email' of undefined
           Stack trace points to UserService.getProfile()"
   ```

3. **Reference instead of repeat:**
   ```
   # Bad: Re-paste the entire component code
   # Good: "In the UserProfile component we discussed earlier, add..."
   ```

### Avoiding Context Pollution

Keep your context focused and relevant:

**Do:**
- Focus on one task at a time
- Clear context between unrelated tasks
- Provide only necessary file contents
- Use specific file paths

**Don't:**
- Ask Claude to read "all files" in a directory
- Mix multiple unrelated requests
- Keep old, irrelevant conversation history
- Paste entire log files

### Strategic Use of Subagents

Use the Task tool to spawn subagents for isolated work:

```
Use a subagent to:
1. Research the best approach for implementing WebSocket support
2. Don't modify any files, just provide a recommendation

Then we'll implement the chosen approach together.
```

Subagents are ideal for:
- Research tasks
- Code analysis
- Generating documentation
- Exploring multiple approaches
- Isolated experiments

---

## 5. Code Quality Practices

### Review Before Accepting Changes

**Never blindly accept generated code.** Always review:

**Checklist for Code Review:**
- [ ] Does it match the requirements?
- [ ] Are there any obvious bugs?
- [ ] Does it follow project conventions?
- [ ] Are there security concerns?
- [ ] Is error handling appropriate?
- [ ] Are edge cases covered?
- [ ] Is the code readable and maintainable?

**In Claude Code:**
```bash
# After Claude generates code, before accepting:
"Before I accept this change, explain:
1. What this code does step by step
2. Any edge cases it handles
3. Any potential issues I should be aware of"
```

### Incremental Changes

Make small, focused changes rather than large rewrites:

**Bad Approach:**
```
Refactor the entire authentication system
```

**Good Approach:**
```
Let's refactor authentication incrementally:

Step 1: Extract token validation to its own function
Step 2: Add TypeScript types to the auth functions
Step 3: Replace callback-based code with async/await
Step 4: Add unit tests for each function
Step 5: Update dependent components

Let's start with Step 1.
```

Benefits of incremental changes:
- Easier to review
- Easier to test
- Easier to revert if something breaks
- Clearer git history
- Less risk

### Testing Alongside Implementation

Request tests with every implementation:

```
Create a new UserService with these methods:
- getUser(id): Fetch user by ID
- updateUser(id, data): Update user data
- deleteUser(id): Soft delete user

For each method:
1. Implement the method
2. Write unit tests covering:
   - Success case
   - User not found
   - Invalid input
   - Database errors
```

**Test-First Approach:**
```
I want to implement a shopping cart. Let's use TDD:

1. First, write failing tests for CartService.addItem()
2. Then implement just enough code to pass
3. Refactor if needed
4. Move to the next method

Start with the tests for addItem().
```

### Security Awareness

Always ask Claude to consider security:

```
Implement user password reset with these security requirements:
1. Tokens expire after 1 hour
2. Tokens are single-use
3. Use cryptographically secure random tokens
4. Rate limit reset requests (max 3 per hour)
5. Don't reveal if email exists in system
6. Log all reset attempts for audit

Review the implementation for OWASP Top 10 vulnerabilities.
```

### Avoiding Over-Engineering

Keep solutions appropriate to the problem:

**Over-engineered:**
```
"Create an abstract factory pattern with dependency injection
for a simple form validation utility"
```

**Right-sized:**
```
"Create a simple form validation utility:
- validateEmail(email): boolean
- validatePassword(password): string[] (returns error messages)
- validateRequired(value): boolean

Keep it simple - just pure functions, no classes needed."
```

---

## 6. Hand-Holding User Journey

Follow this step-by-step journey for every project.

### Step 1: Set Up a New Project Properly

**1.1 Create CLAUDE.md**

```bash
claude
```

```
I'm starting work on this project. Please:
1. Analyze the codebase structure
2. Identify the tech stack and conventions
3. Create a comprehensive CLAUDE.md file

Include:
- Project overview
- Tech stack details
- Directory structure with purposes
- Naming conventions
- Common commands
- Important patterns to follow
```

**1.2 Create .claudeignore**

```
Create a .claudeignore file that excludes:
- All secret/credential files
- Build artifacts and dependencies
- Binary files
- Log files
- Any files that would waste context
```

**1.3 Verify Understanding**

```
Summarize your understanding of this project:
- What does it do?
- What are the main technologies?
- What conventions should you follow?
- What files should you never modify?
```

### Step 2: Write Effective Prompts

**2.1 Start with Context**

```
I'm working on the user authentication feature.
Current state: Basic login exists but no password reset.
Goal: Add password reset functionality.
Constraints: Must use existing EmailService, follow current patterns.
```

**2.2 Be Specific**

```
Add password reset with these exact requirements:
1. POST /api/auth/forgot-password - accepts email, sends reset link
2. POST /api/auth/reset-password - accepts token and new password
3. Tokens expire in 1 hour, stored in Redis
4. Email template at src/emails/password-reset.tsx
5. Follow error handling pattern in src/lib/errors.ts
```

**2.3 Request Step-by-Step**

```
Let's implement this in steps. After each step, wait for my approval:
Step 1: Create the database migration for reset tokens
Step 2: Create the token service
Step 3: Create the API endpoints
Step 4: Create the email template
Step 5: Write tests

Start with Step 1.
```

### Step 3: Manage Context Efficiently

**3.1 Check Context Status**

```
/cost
```

This shows your current token usage and helps you decide when to compact.

**3.2 Compact When Needed**

```
/compact
```

Do this when:
- You've been working for a while
- Context is over 50% used
- Before starting a complex task

**3.3 Clear for Fresh Start**

```
/clear
```

Do this when:
- Starting completely new work
- The conversation is confused
- You've finished a major feature

### Step 4: Review and Validate Changes

**4.1 Review Generated Code**

```
Before I accept this code:
1. Walk me through what it does
2. Explain any complex parts
3. List potential edge cases
4. Identify any security concerns
```

**4.2 Check for Issues**

```
Review the code you just generated for:
1. TypeScript type safety
2. Error handling completeness
3. Security vulnerabilities
4. Performance issues
5. Missing edge cases
```

**4.3 Run Tests**

```
Run the tests for the code you just created and show me the results.
If any fail, explain why and fix them.
```

### Step 5: Maintain Code Quality

**5.1 Request Linting**

```
Run the linter on the files you modified and fix any issues.
```

**5.2 Request Type Checking**

```
Run TypeScript type checking and resolve any errors.
```

**5.3 Final Verification**

```
Before we finish, verify:
1. All tests pass
2. No linting errors
3. No type errors
4. The feature works as specified
5. Documentation is updated if needed
```

---

## 7. Real-World Scenarios

### Scenario 1: Starting a New Feature

**Context:** You need to add a product review system to an e-commerce site.

**Step 1: Plan**
```
I need to add a product review system. Help me plan the implementation:

Requirements:
- Users can leave 1-5 star ratings with text reviews
- Reviews require purchase verification
- Reviews can be marked helpful by other users
- Products show average rating and review count

What components, services, and database changes do we need?
```

**Step 2: Database First**
```
Create the Prisma schema changes for the review system:
- Review model with rating, text, userId, productId
- Helpful votes tracking
- Indexes for common queries

Generate the migration but don't run it yet.
```

**Step 3: Service Layer**
```
Create ReviewService in src/services/review.ts:
- createReview(userId, productId, rating, text)
- getProductReviews(productId, pagination)
- markHelpful(reviewId, userId)
- getAverageRating(productId)

Include validation and error handling.
```

**Step 4: API Routes**
```
Create API routes in src/app/api/reviews/:
- POST /api/reviews - Create review
- GET /api/reviews?productId=X - Get reviews
- POST /api/reviews/[id]/helpful - Mark helpful

Follow our existing API patterns.
```

**Step 5: Components**
```
Create React components:
- ReviewForm - Star rating input + text area
- ReviewList - Paginated review display
- ReviewCard - Individual review display
- StarRating - Reusable star display

Follow existing component patterns in src/components.
```

**Step 6: Integration**
```
Integrate reviews into the ProductDetail page:
- Add ReviewList below product info
- Add ReviewForm for authenticated users who purchased
- Show average rating in product header
```

**Step 7: Testing**
```
Write tests for:
- ReviewService unit tests (all methods)
- API route integration tests
- ReviewForm component tests
- End-to-end test for full review flow
```

### Scenario 2: Debugging a Complex Bug

**Context:** Users report intermittent 500 errors on checkout.

**Step 1: Gather Information**
```
I'm debugging intermittent 500 errors on checkout. Here's what I know:
- Occurs ~5% of the time
- Error log shows: "Transaction timeout in PaymentService"
- Happens more during peak hours
- Started after last week's deployment

Help me investigate. First, read src/services/payment.ts
```

**Step 2: Analyze Code**
```
I see the payment processing code. Look for:
1. Race conditions
2. Missing error handling
3. Timeout configurations
4. Connection pool issues

Read any related files you need.
```

**Step 3: Form Hypothesis**
```
Based on your analysis, what are the most likely causes?
Rank them by probability and explain your reasoning.
```

**Step 4: Create Targeted Fix**
```
Let's address the most likely cause: connection pool exhaustion.

Create a fix that:
1. Increases pool size appropriately
2. Adds connection timeout handling
3. Implements retry with exponential backoff
4. Adds monitoring/logging for pool status

Make minimal changes to fix the issue.
```

**Step 5: Add Tests**
```
Write tests that would have caught this bug:
1. Test behavior under connection pool pressure
2. Test timeout handling
3. Test retry behavior
```

**Step 6: Verify**
```
Run all payment-related tests and verify the fix doesn't break anything.
```

### Scenario 3: Refactoring Legacy Code

**Context:** Old callback-based code needs modernization.

**Step 1: Understand Current State**
```
I need to refactor src/services/legacy/userSync.js to modern standards.
Current issues:
- Callback hell (5+ levels deep)
- No TypeScript
- No error handling
- No tests

First, read and summarize what this code does.
```

**Step 2: Plan Refactoring**
```
Create a refactoring plan that:
1. Preserves all existing functionality
2. Converts to TypeScript with full types
3. Converts callbacks to async/await
4. Adds proper error handling
5. Maintains backwards compatibility

Break into small, safe steps.
```

**Step 3: Create Tests First**
```
Before changing anything, write tests that capture current behavior:
- Test all public functions
- Test edge cases we can identify
- These tests must pass before AND after refactoring
```

**Step 4: Incremental Refactoring**
```
Now refactor step by step. After each step:
1. Make the change
2. Run the tests
3. Verify everything passes
4. Show me the change

Start with converting the first callback to async/await.
```

**Step 5: Final Review**
```
The refactoring is complete. Review the final code:
1. All tests still pass?
2. TypeScript types are complete?
3. Error handling is comprehensive?
4. Code is cleaner and more maintainable?
```

### Scenario 4: Code Review Workflow

**Context:** You want Claude to review a PR.

**Step 1: Load the Changes**
```
Review my PR for the new notification system.
The changes are in these files:
- src/services/notification.ts
- src/components/NotificationBell.tsx
- src/hooks/useNotifications.ts
- src/app/api/notifications/route.ts

Focus on:
1. Code quality and best practices
2. Security vulnerabilities
3. Performance issues
4. Missing edge cases
5. Test coverage
```

**Step 2: Get Detailed Feedback**
```
For each issue you found:
1. Severity (Critical/Major/Minor/Nit)
2. Location (file and line)
3. Problem description
4. Suggested fix with code

Start with critical issues.
```

**Step 3: Address Feedback**
```
Let's address the critical issues you found:
1. SQL injection vulnerability in notification query
2. Missing authentication on delete endpoint

Fix these issues now.
```

**Step 4: Final Check**
```
Review the fixes and confirm:
1. All critical issues are resolved
2. No new issues introduced
3. The code is ready to merge
```

### Scenario 5: Documentation Generation

**Context:** You need API documentation for your REST API.

**Step 1: Analyze Endpoints**
```
Generate OpenAPI/Swagger documentation for our API.
The API routes are in src/app/api/.

For each endpoint, document:
- Method and path
- Description
- Request parameters/body
- Response format
- Error responses
- Authentication requirements
- Example requests/responses
```

**Step 2: Generate OpenAPI Spec**
```
Create an openapi.yaml file with complete documentation.
Follow OpenAPI 3.0 specification.
Include schemas for all request/response types.
```

**Step 3: Create README**
```
Create API documentation in docs/API.md that includes:
- Authentication overview
- Rate limiting details
- Common error formats
- Quick start guide
- All endpoints with examples
```

**Step 4: Verify Accuracy**
```
Cross-reference the documentation with actual code:
1. Are all endpoints documented?
2. Are the types accurate?
3. Are the examples valid?

Fix any discrepancies.
```

### Scenario 6: CI/CD Pipeline Setup

**Context:** Setting up GitHub Actions for a Node.js project.

**Step 1: Requirements**
```
Set up GitHub Actions CI/CD with these requirements:
- Run on push to main and all PRs
- Node.js 18 and 20 matrix
- Steps: Install, Lint, Type Check, Test, Build
- Cache node_modules
- Upload test coverage
- Deploy to staging on main push
- Require approval for production deploy
```

**Step 2: Create Workflow**
```
Create .github/workflows/ci.yml with:
- The job matrix for Node versions
- Proper caching configuration
- All required steps
- Clear job dependencies
```

**Step 3: Create Deploy Workflow**
```
Create .github/workflows/deploy.yml with:
- Staging auto-deploy from main
- Production manual trigger with approval
- Environment secrets handling
- Rollback capability
```

**Step 4: Add Status Badges**
```
Update README.md with:
- CI status badge
- Coverage badge
- Link to workflow runs
```

**Step 5: Verify**
```
Review the workflows for:
- Security (no exposed secrets)
- Efficiency (proper caching)
- Reliability (failure handling)
- Best practices compliance
```

---

## 8. Cost Optimization

### Understanding Token Usage

Tokens are the "currency" of Claude API usage:
- **Input tokens**: Your prompts + context (files read, conversation history)
- **Output tokens**: Claude's responses

**Token Estimation:**
- ~4 characters = 1 token (English text)
- ~100 tokens = 75 words
- A typical source file: 500-2000 tokens

### Efficient Prompting

**Reduce input tokens:**

```
# Inefficient (high tokens)
"I have this really long problem that I need help with and
basically what's happening is that when users try to log in
they sometimes get an error and I'm not sure why this is
happening but I think it might be related to the session
handling code that we wrote last month..."

# Efficient (low tokens)
"Debug login error. Issue: Intermittent session timeout.
Suspected cause: Session handling in auth.ts.
Error: 'Session expired unexpectedly'"
```

**Be specific about file needs:**

```
# Inefficient
"Read all the files in src/services"

# Efficient
"Read src/services/auth.ts - I need to understand the
login function"
```

### Model Selection

Choose the right model for the task:

| Model | Best For | Cost | Speed |
|-------|----------|------|-------|
| **Opus** | Complex architecture, nuanced code review, difficult debugging | Highest | Slowest |
| **Sonnet** | General development, feature implementation, most tasks | Medium | Medium |
| **Haiku** | Simple queries, quick lookups, formatting, documentation | Lowest | Fastest |

**Switching Models:**
```bash
# For complex architectural decisions
claude --model opus "Design the microservices architecture..."

# For standard development
claude --model sonnet "Implement the user service..."

# For quick questions
claude --model haiku "What's the syntax for TypeScript generics?"
```

### /cost Monitoring

Regularly check your usage:

```bash
/cost
```

Output shows:
- Tokens used this session
- Estimated cost
- Context window usage

**Set Budgets:**
- Monitor /cost at the start and end of sessions
- Set mental limits for exploratory work
- Use cheaper models for experimentation

### Reducing Unnecessary Context

**Strategies:**

1. **Use .claudeignore** - Prevent large files from loading
2. **Be selective with file reads** - Only request files you need
3. **Compact regularly** - Keep context focused
4. **Clear between tasks** - Don't carry irrelevant history
5. **Summarize instead of pasting** - For logs and large outputs

```
# Instead of pasting 500 lines of logs:
"The error log shows a NullPointerException on line 234
in UserService.java. The stack trace indicates it originates
from getProfile() when user.address is null."
```

---

## 9. Security Best Practices

### Never Commit Secrets

**Critical files to protect:**

```gitignore
# In .claudeignore AND .gitignore
.env
.env.*
*.pem
*.key
*.p12
credentials.json
service-account.json
secrets/
config/local.json
```

**If Claude generates code with secrets:**
```
Review the code you generated. Replace any hardcoded secrets
with environment variable references:
- Database passwords → process.env.DB_PASSWORD
- API keys → process.env.API_KEY
- JWT secrets → process.env.JWT_SECRET
```

### Review Generated Code

**Security Review Checklist:**

```
Review this code for security vulnerabilities:
1. SQL Injection - Are queries parameterized?
2. XSS - Is user input sanitized/escaped?
3. CSRF - Are tokens validated?
4. Authentication - Are endpoints protected?
5. Authorization - Are permissions checked?
6. Secrets - Any hardcoded credentials?
7. Injection - Any eval() or dynamic code?
8. Dependencies - Any known vulnerable packages?
```

### Permission Management

**Use minimal permissions:**

```
When creating this database user:
1. Only grant permissions actually needed
2. Use read-only access where possible
3. Scope to specific tables/schemas
4. Never use root/admin in application code
```

**API Permission Scoping:**
```typescript
// Bad - overly permissive
app.use(authMiddleware); // All routes require auth

// Good - explicit permissions
router.get('/public-data', publicEndpoint);
router.get('/user/:id', requireAuth, requireOwnership, getUserEndpoint);
router.delete('/user/:id', requireAuth, requireAdmin, deleteUserEndpoint);
```

### Sensitive File Handling

```
When working with user data:
1. Never log sensitive fields (password, SSN, etc.)
2. Encrypt at rest and in transit
3. Mask in error messages
4. Implement data retention policies
5. Add audit logging for access
```

### .claudeignore for Secrets

Create a comprehensive .claudeignore:

```gitignore
# Authentication & Secrets
.env*
*.pem
*.key
*.p12
*.pfx
credentials*
secrets*
*secret*
*password*
token*
auth*config*

# AWS
.aws/
aws-credentials*

# GCP
service-account*.json
gcp-key*.json

# Azure
azure-credentials*

# Certificates
*.crt
*.cer
*.ca-bundle

# Private keys
id_rsa*
id_ed25519*
*.ppk
```

---

## 10. Team Collaboration

### Shared CLAUDE.md

Create a team-standard CLAUDE.md and commit it:

```markdown
# Team Project: Platform X

## Team Conventions
These are agreed-upon standards for our team.

### Code Style
- ESLint config: .eslintrc.js (enforced in CI)
- Prettier config: .prettierrc (auto-format on save)
- TypeScript: strict mode, no any

### Git Workflow
- Branch naming: feature/TICKET-123-description
- Commit format: "TICKET-123: Short description"
- PR required for all changes
- Minimum 1 approval required

### Architecture Decisions
- State management: Zustand (not Redux)
- Data fetching: React Query
- Forms: React Hook Form + Zod
- See docs/architecture-decisions/ for ADRs

### Communication
- Questions: #platform-x-dev Slack
- Code review: Tag @platform-x-reviewers
- Urgent issues: Page on-call
```

### Team Skills and Agents

Create shared skill configurations:

```markdown
# .claude/skills/team-review.md

# Team Code Review Standard

When reviewing code, check:
1. Follows our ESLint configuration
2. Has appropriate test coverage (>80%)
3. Uses approved libraries only
4. Follows our API design guidelines
5. Has proper error handling
6. Includes documentation for public APIs
```

### Consistent Conventions

Document and enforce conventions:

```markdown
## Naming Conventions (ENFORCED)

### Files
- Components: PascalCase.tsx (UserProfile.tsx)
- Hooks: camelCase.ts starting with 'use' (useAuth.ts)
- Services: camelCase.ts (userService.ts)
- Types: PascalCase.ts (UserTypes.ts)
- Tests: *.test.ts or *.spec.ts

### Code
- Variables/functions: camelCase
- Classes/Components: PascalCase
- Constants: SCREAMING_SNAKE_CASE
- Private methods: _prefixUnderscore

### Database
- Tables: snake_case plural (user_profiles)
- Columns: snake_case (created_at)
```

### Knowledge Sharing

Use Claude to maintain team knowledge:

```
Update our CLAUDE.md to document the new authentication
pattern we implemented. Include:
1. How it works
2. Where the code lives
3. How to extend it
4. Common pitfalls
```

---

## 11. Performance Optimization

### Parallel Agent Usage

Use multiple agents for independent tasks:

```
I need to update 5 independent microservices with the new
logging format. Create separate tasks for each:

1. User Service - src/services/user/
2. Order Service - src/services/order/
3. Payment Service - src/services/payment/
4. Inventory Service - src/services/inventory/
5. Notification Service - src/services/notification/

These can run in parallel since they don't depend on each other.
```

### Background Tasks

Use background execution for long-running tasks:

```bash
# Run tests in background while continuing to code
/task "Run the full test suite and report failures" --background

# Continue working on other things
"While tests run, let's implement the next feature..."
```

### Efficient Tool Usage

**Minimize file reads:**
```
# Inefficient
"Read all files in src/components"

# Efficient
"Read src/components/UserProfile/index.tsx - I need to modify the props"
```

**Batch related operations:**
```
# Inefficient - multiple separate requests
"Add import to file A"
"Add import to file B"
"Add import to file C"

# Efficient - single request
"Add the new UserService import to these files:
- src/pages/Profile.tsx
- src/pages/Settings.tsx
- src/pages/Dashboard.tsx"
```

### Avoiding Redundant Operations

```
# Inefficient - reading same file multiple times
"Read auth.ts to understand login"
[later]
"Read auth.ts to see the logout function"

# Efficient - read once, reference
"Read auth.ts - I need to understand all auth functions"
[later]
"In the auth.ts we read earlier, modify the logout function to..."
```

---

## 12. Common Anti-Patterns

### Anti-Pattern 1: Vague Prompts

**Bad:**
```
Make it better
Fix the code
Add some tests
Improve performance
```

**Good:**
```
Improve the UserList component performance by:
1. Adding React.memo to prevent unnecessary re-renders
2. Using useMemo for filtered list calculation
3. Implementing virtualization for lists > 100 items
Target: 60fps scrolling with 1000 items
```

### Anti-Pattern 2: Ignoring Context Limits

**Bad:**
- Continuing long conversations without /compact
- Reading entire directories
- Pasting massive log files
- Never clearing context

**Good:**
- Monitor with /cost
- Use /compact when context grows large
- Use /clear between unrelated tasks
- Be selective about what you share

### Anti-Pattern 3: Accepting Code Blindly

**Bad:**
```
[Claude generates code]
"Looks good, commit it"
```

**Good:**
```
[Claude generates code]
"Walk me through this code step by step"
"What edge cases does this handle?"
"Are there any security concerns?"
"Let's run the tests before committing"
```

### Anti-Pattern 4: Over-Relying on Claude

**Bad:**
- Asking Claude to make every tiny change
- Not learning from Claude's solutions
- Never writing code yourself

**Good:**
- Use Claude for complex tasks, write simple code yourself
- Study Claude's patterns and learn from them
- Build your understanding, not just your codebase

### Anti-Pattern 5: Not Testing Changes

**Bad:**
```
[Claude generates code]
"Ship it"
```

**Good:**
```
[Claude generates code]
"Write tests for this code"
"Run the tests"
"Run the linter"
"Run type checking"
"Now we can commit"
```

### Anti-Pattern 6: Mixing Multiple Tasks

**Bad:**
```
Fix the login bug, add the new feature, refactor the
database layer, and update the documentation.
```

**Good:**
```
Let's fix the login bug first. The issue is...
[Complete and verify]
/clear
Now let's add the new feature...
```

---

## 13. Checklist: Before Starting Work

### Pre-Work Checklist

```markdown
## Before Starting a Task

- [ ] CLAUDE.md exists and is up-to-date
- [ ] .claudeignore excludes secrets and large files
- [ ] Git branch is created for this work
- [ ] Requirements are clearly understood
- [ ] Context is clean (/clear if needed)
- [ ] Relevant files are identified (not "all files")
```

### During Work Checklist

```markdown
## While Working

- [ ] Prompts are specific and clear
- [ ] Changes are incremental, not massive
- [ ] Each change is reviewed before accepting
- [ ] Tests are written alongside code
- [ ] Context is compacted when needed (/compact)
- [ ] Errors are fixed before moving on
- [ ] Security is considered for each change
```

### Post-Work Checklist

```markdown
## Before Finishing

- [ ] All tests pass
- [ ] Linter shows no errors
- [ ] Type checking passes
- [ ] Code is reviewed (manually or by Claude)
- [ ] Changes are committed with clear message
- [ ] Documentation is updated if needed
- [ ] No secrets in committed code
- [ ] Feature works as expected
```

---

## 14. Quick Reference

### Do's and Don'ts

| Do | Don't |
|----|-------|
| Be specific in prompts | Use vague language |
| Provide context | Assume Claude knows everything |
| Review generated code | Accept code blindly |
| Test incrementally | Skip testing |
| Use /compact when needed | Let context overflow |
| Use /clear between tasks | Mix unrelated work |
| Document in CLAUDE.md | Repeat explanations |
| Use .claudeignore | Expose secrets |
| Make incremental changes | Make massive rewrites |
| Ask for explanations | Assume correctness |

### Quick Tips Card

```
┌─────────────────────────────────────────────────────────────┐
│                 CLAUDE CODE QUICK TIPS                       │
├─────────────────────────────────────────────────────────────┤
│ CONTEXT COMMANDS                                             │
│   /clear    - Fresh start (new task)                        │
│   /compact  - Compress context (same task)                  │
│   /cost     - Check token usage                             │
│                                                             │
│ GOOD PROMPT FORMULA                                         │
│   Context + Specific Task + Constraints + Expected Output   │
│                                                             │
│ BEFORE ACCEPTING CODE                                       │
│   1. Understand what it does                                │
│   2. Check for security issues                              │
│   3. Verify edge cases                                      │
│   4. Run tests                                              │
│                                                             │
│ COST SAVING                                                 │
│   • Use Haiku for simple queries                            │
│   • Be selective about file reads                           │
│   • Compact context regularly                               │
│   • Don't paste entire files unnecessarily                  │
└─────────────────────────────────────────────────────────────┘
```

### Emergency Commands

When things go wrong:

```bash
# Claude is confused or giving bad output
/clear

# Context is too large
/compact

# Need to stop current operation
Ctrl+C

# Start fresh session
Exit and restart Claude Code

# Claude made unwanted changes
git checkout -- .  # Discard all changes
git checkout -- path/to/file  # Discard specific file

# Undo last commit (if committed unwanted changes)
git reset --soft HEAD~1
```

### Keyboard Shortcuts Reference

```
Ctrl+C     - Interrupt current operation
Ctrl+D     - Exit Claude Code
↑/↓        - Navigate command history
Tab        - Autocomplete
```

---

## Summary

Following these best practices will help you:

1. **Work more efficiently** - Clear prompts = better results
2. **Maintain code quality** - Review and test everything
3. **Control costs** - Smart context management
4. **Stay secure** - Never expose secrets
5. **Collaborate better** - Shared conventions and documentation

Remember the core loop:

```
Prepare → Prompt → Review → Test → Iterate
```

Master this loop, and Claude Code becomes an incredibly powerful extension of your development capabilities.

---

## Additional Resources

- [Claude Code Official Documentation](https://docs.anthropic.com/claude-code)
- [Prompt Engineering Guide](https://docs.anthropic.com/prompt-engineering)
- [API Pricing](https://anthropic.com/pricing)

---

*This tutorial is part of the Claude Code Tutorial Series. For questions or contributions, please refer to the main repository documentation.*
