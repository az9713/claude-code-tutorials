# Tutorial 05: Claude Code Skills - A Comprehensive Guide

## Table of Contents

1. [Introduction](#1-introduction)
2. [Installation & Setup](#2-installation--setup)
3. [Skill File Format](#3-skill-file-format)
4. [Built-in Skills Overview](#4-built-in-skills-overview)
5. [Hand-Holding User Journey](#5-hand-holding-user-journey)
6. [Real-World Examples](#6-real-world-examples)
7. [Creating Custom Skills](#7-creating-custom-skills)
8. [Skill Triggers](#8-skill-triggers)
9. [Pros and Cons](#9-pros-and-cons)
10. [When to Use Skills](#10-when-to-use-skills)
11. [Comparison Table](#11-comparison-table)
12. [Advanced Patterns](#12-advanced-patterns)
13. [Quick Reference](#13-quick-reference)

---

## 1. Introduction

### What are Skills in Claude Code?

Skills are reusable instruction sets that extend Claude Code's capabilities for specific tasks. Think of them as **specialized knowledge modules** that Claude can activate when needed. Unlike subagents that spawn separate processes, skills inject expertise directly into the current conversation context.

A skill is essentially a markdown file containing:
- Detailed instructions for a specific task type
- Best practices and patterns to follow
- Checklists and workflows
- Tool restrictions (optional)

When activated, skills provide Claude with domain-specific guidance that improves output quality and consistency.

### Skills vs Subagents: Key Differences

| Aspect | Skills | Subagents |
|--------|--------|-----------|
| **Execution** | Same conversation context | Separate spawned process |
| **Memory** | Full context access | Isolated, limited context |
| **Activation** | Instant (no spawn time) | Requires process creation |
| **Cost** | No additional tokens for spawning | Token overhead for task description |
| **Parallelism** | Sequential only | Can run in parallel |
| **Scope** | Guidance and patterns | Independent task execution |
| **Best for** | Expertise injection | Delegated work |

**Analogy**: Skills are like consulting a reference manual while working. Subagents are like delegating a task to a colleague.

### How Skills Enhance Claude's Capabilities

1. **Consistent Quality**: Skills encode best practices ensuring reproducible results
2. **Domain Expertise**: Specialized knowledge for code review, testing, documentation
3. **Workflow Automation**: Multi-step processes with checkpoints
4. **Team Standards**: Shared conventions across team members
5. **Reduced Errors**: Checklists prevent common mistakes
6. **Faster Onboarding**: New team members get immediate access to team knowledge

### Built-in vs Custom Skills

**Built-in Skills** come pre-packaged with Claude Code:
- `code-review` - Systematic code review checklist
- `git-workflow` - Git branching and commit patterns
- `testing` - Test writing strategies
- `project-workflow` - Development lifecycle patterns
- `debug-chrome-extension` - Browser extension debugging

**Custom Skills** are created by users:
- Project-specific workflows
- Company coding standards
- Domain-specific patterns
- Team conventions

---

## 2. Installation & Setup

### Skill File Locations

Skills can live in three locations, loaded in priority order:

```
1. Project Skills (highest priority)
   .claude/skills/

2. User Skills (personal)
   ~/.claude/skills/

3. Plugin Skills (from extensions)
   Managed by plugin system
```

### Project Skills (.claude/skills/)

Project skills are stored in your repository and shared with the team:

```
your-project/
├── .claude/
│   └── skills/
│       ├── code-review.md
│       ├── api-design.md
│       └── testing-standards.md
├── src/
└── package.json
```

**Creating project skills directory:**

```bash
# From project root
mkdir -p .claude/skills

# Create a skill file
touch .claude/skills/my-skill.md
```

**Benefits of project skills:**
- Version controlled with your code
- Automatically available to team members
- Can reference project-specific patterns
- Evolve with your codebase

### User Skills (~/.claude/skills/)

User skills are personal and available across all projects:

```bash
# Create user skills directory
mkdir -p ~/.claude/skills

# Create personal skill
touch ~/.claude/skills/my-personal-workflow.md
```

**Windows path:**
```
C:\Users\<username>\.claude\skills\
```

**Use cases for user skills:**
- Personal productivity workflows
- Cross-project standards you prefer
- Experimental skills before team adoption

### Plugin Skills

Skills can also come from Claude Code plugins:

```json
// In a plugin's package.json
{
  "claude": {
    "skills": [
      "skills/specialized-review.md"
    ]
  }
}
```

### Enabling/Disabling Skills

Skills are automatically discovered and enabled. To manage them:

**List available skills:**
```bash
# In Claude Code
/skills list
```

**Invoke a specific skill:**
```bash
# Using slash command
/code-review

# Or mention in conversation
"Use the code-review skill to analyze this PR"
```

**Disable a skill temporarily:**
Skills activate based on context. To prevent activation, be explicit:
```
"Without using the code-review skill, give me quick feedback on this code"
```

---

## 3. Skill File Format

### Markdown Structure

Skills are markdown files with optional YAML frontmatter:

```markdown
---
name: skill-name
description: Brief description of what this skill does
allowed_tools:
  - Read
  - Grep
  - Glob
---

# Skill Title

## Overview
What this skill does and when to use it.

## Instructions
Detailed guidance for Claude when this skill is active.

## Checklist
- [ ] Step 1
- [ ] Step 2

## Examples
Show expected inputs and outputs.
```

### YAML Frontmatter

The frontmatter configures skill behavior:

```yaml
---
# Required: Unique identifier (used for /skill-name invocation)
name: api-design-review

# Required: Shown in skill listings
description: Review API designs for RESTful best practices

# Optional: Restrict available tools
allowed_tools:
  - Read
  - Grep
  - Glob
  - Edit

# Optional: Override the model for this skill
model: claude-sonnet-4-20250514

# Optional: Fork context (skill runs with snapshot of current context)
context:
  fork: true
---
```

### Name and Description

**Name requirements:**
- Lowercase with hyphens (kebab-case)
- No spaces or special characters
- Unique across all skill locations
- Used for `/skill-name` invocation

**Good names:**
- `code-review`
- `api-design-patterns`
- `testing-standards`

**Bad names:**
- `Code Review` (spaces)
- `mySkill` (camelCase)
- `skill.review` (dots)

**Description best practices:**
- One sentence explaining the skill's purpose
- Include key use cases
- Be specific about what makes this skill unique

```yaml
# Good description
description: Review Python code for PEP 8 compliance, type hints, and docstring quality

# Too vague
description: Reviews code
```

### Allowed Tools

Restrict which tools the skill can use:

```yaml
allowed_tools:
  # File operations
  - Read           # Read file contents
  - Write          # Create/overwrite files
  - Edit           # Modify existing files

  # Search
  - Grep           # Search file contents
  - Glob           # Find files by pattern

  # Execution
  - Bash           # Run shell commands

  # Other
  - WebFetch       # Fetch web content
  - WebSearch      # Search the web
  - Task           # Spawn subagents
  - TodoWrite      # Manage todo lists
```

**Why restrict tools?**
- Security: Prevent unintended file modifications
- Focus: Keep skill on track
- Safety: Review skills shouldn't edit code

### Model Override

Force a specific model for the skill:

```yaml
# Use faster model for simple skills
model: claude-sonnet-4-20250514

# Use most capable model for complex skills
model: claude-opus-4-5-20251101
```

**Use cases:**
- Cost optimization for simple skills
- Quality assurance for critical skills
- Specific model capabilities needed

### Context Options

Control how context is handled:

```yaml
context:
  # Fork creates a snapshot - changes don't persist
  fork: true

  # Include specific files in context
  include:
    - "README.md"
    - "ARCHITECTURE.md"
```

**Forked context** is useful for:
- Experimental analysis
- What-if scenarios
- Skills that might make mistakes

---

## 4. Built-in Skills Overview

### code-review Skill

**Purpose:** Systematic code review with comprehensive checklist

**Invocation:**
```bash
/code-review
# or
"Review this code using the code-review skill"
```

**What it does:**
- Analyzes code for bugs, security issues, performance
- Checks naming conventions and code style
- Evaluates error handling and edge cases
- Suggests improvements with examples

**Example usage:**
```
User: /code-review

Claude: I'll review the code systematically.

## Code Review Checklist

### Correctness
- [x] Logic handles all edge cases
- [ ] Missing null check on line 45
- [x] Algorithm complexity is appropriate

### Security
- [ ] SQL query on line 23 vulnerable to injection
- [x] Input validation present
- [x] No hardcoded credentials

### Maintainability
- [x] Functions are single-purpose
- [ ] Magic number on line 67 should be constant
- [x] Code is well-documented

## Recommendations
1. **Critical**: Parameterize SQL query on line 23
2. **Important**: Add null check for `user` object
3. **Minor**: Extract magic number to named constant
```

### git-workflow Skill

**Purpose:** Git branching, commits, and PR best practices

**Invocation:**
```bash
/git-workflow
```

**Capabilities:**
- Branch naming conventions
- Commit message formatting
- PR description templates
- Merge strategies
- Conflict resolution guidance

**Example usage:**
```
User: /git-workflow - help me prepare a PR for this feature

Claude: I'll help you prepare a proper PR following git workflow best practices.

## Branch Check
Current branch: `feature/user-authentication`
✓ Branch name follows convention: feature/<description>

## Commit Analysis
3 commits on this branch:
1. "Add user model" - ✓ Good
2. "wip" - ⚠️ Should be squashed or reworded
3. "Add authentication endpoints" - ✓ Good

## Recommended Actions
1. Squash commit 2 into commit 3:
   ```bash
   git rebase -i HEAD~3
   ```

2. PR Title: "Add user authentication system"

3. PR Description:
   ## Summary
   - Implements user authentication with JWT tokens
   - Adds login/logout endpoints
   - Includes password hashing with bcrypt

   ## Test Plan
   - [ ] Run auth test suite
   - [ ] Manual login/logout test
   - [ ] Verify token expiration
```

### testing Skill

**Purpose:** Test writing strategies and patterns

**Invocation:**
```bash
/testing
```

**Capabilities:**
- Test case identification
- Coverage analysis
- Test structure recommendations
- Mocking strategies
- Edge case enumeration

**Example usage:**
```
User: /testing - what tests should I write for this function?

Claude: I'll analyze this function and recommend comprehensive tests.

## Function Analysis
`calculateDiscount(price, userTier, promoCode)`

## Recommended Test Cases

### Happy Path Tests
1. Regular user with valid promo code
2. Premium user without promo code
3. VIP user with percentage discount

### Edge Cases
1. Price is zero
2. Price is negative (should throw)
3. Invalid user tier
4. Expired promo code
5. Case-insensitive promo code matching

### Boundary Tests
1. Maximum discount cap
2. Minimum price threshold
3. Discount rounding behavior

### Integration Tests
1. Database lookup for promo codes
2. User tier verification
3. Discount audit logging

## Test Template
```javascript
describe('calculateDiscount', () => {
  describe('happy path', () => {
    it('should apply 10% discount for regular users', () => {
      expect(calculateDiscount(100, 'regular', null)).toBe(90);
    });
  });

  describe('edge cases', () => {
    it('should throw for negative prices', () => {
      expect(() => calculateDiscount(-10, 'regular', null))
        .toThrow('Price must be positive');
    });
  });
});
```

### project-workflow Skill

**Purpose:** Development lifecycle and project patterns

**Invocation:**
```bash
/project-workflow
```

**Capabilities:**
- Plan-Build-Review-Fix cycle
- Feature development process
- Release preparation
- Documentation requirements

**Example usage:**
```
User: /project-workflow - I need to add a payment feature

Claude: I'll guide you through the feature development workflow.

## Feature Development Workflow

### Phase 1: Planning
- [ ] Write feature specification
- [ ] Identify affected components
- [ ] Estimate complexity (this appears Medium)
- [ ] Create branch: `feature/payment-integration`

### Phase 2: Design
- [ ] API contract definition
- [ ] Database schema changes
- [ ] Security review checklist
- [ ] Payment provider selection

### Phase 3: Implementation
- [ ] Core payment service
- [ ] API endpoints
- [ ] Frontend integration
- [ ] Error handling

### Phase 4: Testing
- [ ] Unit tests (>80% coverage)
- [ ] Integration tests with mock provider
- [ ] E2E tests for checkout flow
- [ ] Security penetration testing

### Phase 5: Review
- [ ] Code review
- [ ] Security review
- [ ] Performance review

### Phase 6: Release
- [ ] Feature flag configuration
- [ ] Staged rollout plan
- [ ] Monitoring dashboards
- [ ] Rollback procedure

Shall I help with any specific phase?
```

### Debugging Skills

**Purpose:** Systematic debugging approaches

**Available debugging skills:**
- `debug-chrome-extension` - Browser extension issues
- `systematic-debugging` - General debugging methodology

**Invocation:**
```bash
/systematic-debugging
```

**Example usage:**
```
User: /systematic-debugging - users report intermittent login failures

Claude: I'll apply systematic debugging methodology.

## Problem Statement
Intermittent login failures reported by users

## Information Gathering
1. When did this start? (recent deployment?)
2. Frequency: How often does it fail?
3. Pattern: Specific users, times, or random?
4. Error messages: What do users see?

## Hypothesis Generation
Based on "intermittent":
1. Race condition in auth flow
2. Session store connectivity issues
3. Load balancer session affinity problems
4. Token expiration edge case
5. Database connection pool exhaustion

## Investigation Plan

### Step 1: Log Analysis
```bash
# Search for auth failures in last 24h
grep -i "auth.*fail" /var/log/app/*.log | tail -100
```

### Step 2: Metrics Check
- Auth service error rate
- Database connection pool usage
- Redis connection status

### Step 3: Reproduce
- Load test auth endpoint
- Test with specific failing user accounts

## Next Steps
Let me start with log analysis. Can you share access to your logging system?
```

---

## 5. Hand-Holding User Journey

### Step 1: Discover Available Skills

**Goal:** See what skills are available in your environment

**Action:**
```bash
# In Claude Code conversation
"What skills are available?"

# Or use the skill listing
/skills
```

**Expected output:**
```
Available Skills:

Built-in Skills:
  - code-review: Code review checklist and best practices
  - git-workflow: Git workflow patterns for branching and commits
  - testing: Testing patterns and best practices
  - project-workflow: Development workflow patterns

Project Skills (.claude/skills/):
  - (none found)

User Skills (~/.claude/skills/):
  - (none found)

To use a skill, type /skill-name or mention it in your request.
```

### Step 2: Use a Built-in Skill

**Goal:** Experience how skills enhance Claude's responses

**Action:**
```bash
# Create a simple function to review
"Here's a function I wrote. /code-review

function divide(a, b) {
  return a / b;
}"
```

**Expected output:**
```
## Code Review: divide function

### Issues Found

#### Critical
1. **Division by zero**: No check for `b === 0`
   ```javascript
   // Current (dangerous)
   return a / b;

   // Recommended
   if (b === 0) throw new Error('Division by zero');
   return a / b;
   ```

#### Moderate
2. **Type safety**: No validation of input types
   - What if `a` or `b` is `undefined`?
   - What if strings are passed?

#### Minor
3. **Documentation**: Missing JSDoc comments

### Suggested Improvement
```javascript
/**
 * Divides two numbers safely.
 * @param {number} a - Dividend
 * @param {number} b - Divisor
 * @returns {number} Result of division
 * @throws {Error} If divisor is zero
 */
function divide(a, b) {
  if (typeof a !== 'number' || typeof b !== 'number') {
    throw new TypeError('Arguments must be numbers');
  }
  if (b === 0) {
    throw new Error('Division by zero');
  }
  return a / b;
}
```
```

### Step 3: Create Your First Custom Skill

**Goal:** Create a project-specific skill

**Action 1: Create the skills directory**
```bash
# In your terminal
mkdir -p .claude/skills
```

**Action 2: Create the skill file**

Create `.claude/skills/python-style.md`:

```markdown
---
name: python-style
description: Enforce team Python coding standards
allowed_tools:
  - Read
  - Grep
  - Glob
---

# Python Style Guide Checker

## Overview
Check Python code against our team's style guidelines.

## Our Standards

### Naming
- Classes: PascalCase
- Functions/variables: snake_case
- Constants: UPPER_SNAKE_CASE
- Private: prefix with underscore

### Docstrings
- All public functions must have docstrings
- Use Google style format:
  ```python
  def function(arg1: str, arg2: int) -> bool:
      """Short description.

      Longer description if needed.

      Args:
          arg1: Description of arg1
          arg2: Description of arg2

      Returns:
          Description of return value

      Raises:
          ValueError: When something is wrong
      """
  ```

### Imports
- Standard library first
- Third-party second (blank line)
- Local imports third (blank line)
- Alphabetize within groups

### Type Hints
- Required for all function signatures
- Use `Optional[]` for nullable types
- Use `list[]` not `List[]` (Python 3.9+)

## Review Checklist
When reviewing Python code, check:
- [ ] All names follow conventions
- [ ] Public functions have docstrings
- [ ] Imports are organized
- [ ] Type hints are present
- [ ] No unused imports
- [ ] No magic numbers (use constants)

## Example Review Output
```
## Style Check: user_service.py

### Violations Found
1. Line 15: `getUserById` should be `get_user_by_id`
2. Line 23: Missing docstring for `process_payment`
3. Line 5-10: Imports not sorted correctly

### Suggestions
- Add type hints to `calculate_total` on line 45
- Extract magic number `86400` to `SECONDS_PER_DAY`
```
```

**Action 3: Verify the skill is discovered**
```bash
# In Claude Code
"List available skills"
```

**Expected:** Your `python-style` skill should appear under Project Skills.

### Step 4: Test and Refine Your Skill

**Goal:** Use your skill and improve it based on results

**Action:**
```bash
# In Claude Code
"/python-style - review this code:

def GetUserData(userId):
    data = db.query(userId)
    return data"
```

**Expected output:**
```
## Style Check Results

### Violations Found

1. **Function name** (Line 1)
   - Current: `GetUserData` (PascalCase)
   - Should be: `get_user_data` (snake_case)

2. **Parameter name** (Line 1)
   - Current: `userId` (camelCase)
   - Should be: `user_id` (snake_case)

3. **Missing docstring** (Line 1)
   - Public functions require docstrings

4. **Missing type hints** (Line 1)
   - Add parameter and return types

### Corrected Version
```python
def get_user_data(user_id: int) -> dict:
    """Retrieve user data from database.

    Args:
        user_id: The unique identifier of the user

    Returns:
        Dictionary containing user data
    """
    data = db.query(user_id)
    return data
```
```

**Refinement:** If the output isn't what you expected, edit the skill file to add more specific instructions.

### Step 5: Share Skills with Team

**Goal:** Make your skill available to teammates

**Action 1: Commit the skill to version control**
```bash
git add .claude/skills/python-style.md
git commit -m "Add Python style guide skill for code reviews"
git push
```

**Action 2: Document in your README**
```markdown
## Claude Code Skills

This project includes custom Claude Code skills:

### python-style
Enforces our team's Python coding standards. Use with:
```
/python-style
```

### Adding New Skills
Place skill files in `.claude/skills/` following the format in existing skills.
```

**Action 3: Team members pull and use**
```bash
# Teammate pulls the repo
git pull

# Skill is automatically available
# In Claude Code:
/python-style
```

---

## 6. Real-World Examples

### Example 1: Code Review Checklist Skill

**File:** `.claude/skills/thorough-review.md`

```markdown
---
name: thorough-review
description: Comprehensive code review with security, performance, and maintainability checks
allowed_tools:
  - Read
  - Grep
  - Glob
---

# Thorough Code Review

## Overview
Perform an exhaustive code review covering all quality dimensions.

## Review Dimensions

### 1. Correctness
- [ ] Logic handles all expected inputs
- [ ] Edge cases are covered
- [ ] Error conditions are handled
- [ ] Return values are correct types
- [ ] Side effects are documented

### 2. Security
- [ ] Input validation present
- [ ] No SQL injection vulnerabilities
- [ ] No XSS vulnerabilities
- [ ] Secrets are not hardcoded
- [ ] Authentication checks in place
- [ ] Authorization verified
- [ ] Sensitive data encrypted
- [ ] Audit logging for sensitive operations

### 3. Performance
- [ ] No N+1 query patterns
- [ ] Appropriate data structures used
- [ ] No unnecessary iterations
- [ ] Caching considered where appropriate
- [ ] Database indexes would be used
- [ ] No memory leaks (closures, listeners)

### 4. Maintainability
- [ ] Functions are single-purpose
- [ ] Names are descriptive
- [ ] Comments explain "why" not "what"
- [ ] No magic numbers
- [ ] DRY principle followed
- [ ] Complexity is manageable

### 5. Testing
- [ ] Unit tests exist or are straightforward to write
- [ ] Edge cases are testable
- [ ] Mocking requirements are reasonable
- [ ] Integration points are clear

### 6. Documentation
- [ ] Public APIs are documented
- [ ] Complex logic is explained
- [ ] Examples provided where helpful
- [ ] Changelog updated if applicable

## Output Format

```
## Code Review: [filename]

### Summary
[1-2 sentence overview]

### Critical Issues
[Must fix before merge]

### Important Issues
[Should fix, may block merge]

### Minor Issues
[Nice to fix, won't block]

### Positive Notes
[What's done well]

### Recommendation
[ ] Approve
[ ] Approve with minor changes
[ ] Request changes
[ ] Needs discussion
```

## Severity Guidelines

**Critical:**
- Security vulnerabilities
- Data loss potential
- Crashes in production
- Incorrect business logic

**Important:**
- Performance issues at scale
- Missing error handling
- Poor testability
- Violation of team standards

**Minor:**
- Style inconsistencies
- Missing optional docs
- Refactoring opportunities
- Naming improvements
```

### Example 2: PR Template Generator Skill

**File:** `.claude/skills/pr-template.md`

```markdown
---
name: pr-template
description: Generate comprehensive PR descriptions from code changes
allowed_tools:
  - Read
  - Grep
  - Glob
  - Bash
---

# PR Template Generator

## Overview
Analyze code changes and generate a well-structured PR description.

## Process

### Step 1: Gather Information
Run these commands to understand the changes:
```bash
git diff main...HEAD --stat
git log main..HEAD --oneline
```

### Step 2: Analyze Changes
For each modified file:
1. What type of change? (feature/fix/refactor/docs/test)
2. What's the impact?
3. Are there breaking changes?

### Step 3: Generate PR Description

Use this template:

```markdown
## Summary
<!-- 2-3 sentences describing what this PR does -->

## Type of Change
- [ ] New feature
- [ ] Bug fix
- [ ] Refactoring
- [ ] Documentation
- [ ] Tests
- [ ] Configuration

## Changes Made
<!-- Bullet list of specific changes -->
-
-
-

## Breaking Changes
<!-- List any breaking changes, or "None" -->

## Testing Done
<!-- How was this tested? -->
- [ ] Unit tests added/updated
- [ ] Integration tests pass
- [ ] Manual testing performed

## Screenshots
<!-- If UI changes, add before/after screenshots -->

## Checklist
- [ ] Code follows project style guidelines
- [ ] Self-review completed
- [ ] Comments added for complex logic
- [ ] Documentation updated
- [ ] No console.log or debug statements

## Related Issues
<!-- Link related issues: Fixes #123, Related to #456 -->

## Deployment Notes
<!-- Any special deployment considerations -->
```

## Examples

### Feature PR
```markdown
## Summary
Adds user profile picture upload functionality with image resizing and S3 storage.

## Type of Change
- [x] New feature

## Changes Made
- Added `ProfilePictureUpload` component
- Implemented image resizing service using Sharp
- Added S3 upload utility
- Created `/api/users/:id/avatar` endpoint
- Added profile picture to user settings page

## Breaking Changes
None

## Testing Done
- [x] Unit tests for image resizing
- [x] Integration tests for upload endpoint
- [x] Manual testing with various image sizes

## Related Issues
Fixes #234
```

### Bug Fix PR
```markdown
## Summary
Fixes race condition in checkout flow that caused duplicate orders.

## Type of Change
- [x] Bug fix

## Changes Made
- Added database transaction around order creation
- Implemented optimistic locking on cart
- Added idempotency key to prevent duplicates

## Breaking Changes
None - backwards compatible

## Testing Done
- [x] Unit tests for locking mechanism
- [x] Load testing to verify fix
- [x] Manual testing of checkout flow
```
```

### Example 3: Documentation Style Guide Skill

**File:** `.claude/skills/docs-style.md`

```markdown
---
name: docs-style
description: Ensure documentation follows team conventions and is comprehensive
allowed_tools:
  - Read
  - Write
  - Edit
  - Glob
---

# Documentation Style Guide

## Overview
Generate and review documentation following our team standards.

## Documentation Types

### 1. Code Comments
```javascript
// BAD: What the code does (obvious)
// Increment counter
counter++;

// GOOD: Why the code does it
// Skip first element as it's the header row
for (let i = 1; i < rows.length; i++)
```

### 2. Function Documentation

**JavaScript/TypeScript:**
```typescript
/**
 * Calculates the total price including discounts and tax.
 *
 * @param items - Array of cart items with price and quantity
 * @param discountCode - Optional promotional code
 * @returns Object containing subtotal, discount, tax, and total
 * @throws {InvalidDiscountError} If discount code is invalid
 *
 * @example
 * const result = calculateTotal([
 *   { price: 10, quantity: 2 }
 * ], 'SAVE10');
 * // result: { subtotal: 20, discount: 2, tax: 1.62, total: 19.62 }
 */
```

**Python:**
```python
def calculate_total(items: list[CartItem], discount_code: str | None = None) -> PriceBreakdown:
    """Calculate total price including discounts and tax.

    Takes a list of cart items and applies any applicable discounts
    before calculating tax.

    Args:
        items: List of cart items with price and quantity
        discount_code: Optional promotional code for discounts

    Returns:
        PriceBreakdown with subtotal, discount, tax, and total

    Raises:
        InvalidDiscountError: If the discount code doesn't exist or is expired

    Example:
        >>> result = calculate_total([CartItem(price=10, qty=2)], 'SAVE10')
        >>> result.total
        19.62
    """
```

### 3. README Structure
```markdown
# Project Name

Brief description (1-2 sentences)

## Features
- Key feature 1
- Key feature 2

## Quick Start
```bash
npm install
npm run dev
```

## Installation
Detailed installation steps...

## Usage
How to use with examples...

## API Reference
Link to detailed docs or inline reference...

## Configuration
Environment variables and options...

## Contributing
How to contribute...

## License
License information
```

### 4. API Documentation
```markdown
## Endpoint Name

Brief description of what this endpoint does.

**URL:** `POST /api/v1/resource`

**Authentication:** Required (Bearer token)

**Request Body:**
```json
{
  "field1": "string (required) - Description",
  "field2": "number (optional) - Description"
}
```

**Response:**
```json
{
  "id": "string - Unique identifier",
  "createdAt": "ISO 8601 timestamp"
}
```

**Errors:**
| Code | Description |
|------|-------------|
| 400  | Invalid request body |
| 401  | Not authenticated |
| 403  | Not authorized |
| 404  | Resource not found |

**Example:**
```bash
curl -X POST https://api.example.com/v1/resource \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"field1": "value"}'
```
```

## Review Checklist
- [ ] All public APIs documented
- [ ] Examples provided
- [ ] Error cases documented
- [ ] Types/parameters described
- [ ] Return values explained
- [ ] No outdated documentation
```

### Example 4: API Design Patterns Skill

**File:** `.claude/skills/api-design.md`

```markdown
---
name: api-design
description: Design RESTful APIs following best practices and team conventions
allowed_tools:
  - Read
  - Write
  - Edit
  - Grep
---

# API Design Patterns

## Overview
Guide API design decisions following REST best practices and team conventions.

## URL Structure

### Resource Naming
```
# Good - nouns, plural, lowercase
GET /api/v1/users
GET /api/v1/users/123
GET /api/v1/users/123/orders

# Bad
GET /api/v1/getUser        # verb in URL
GET /api/v1/User           # singular, capitalized
GET /api/v1/user_orders    # underscore
```

### Nested Resources
```
# Good - clear hierarchy
GET /api/v1/users/123/orders
GET /api/v1/orders/456/items

# Avoid deep nesting (max 2 levels)
# Bad
GET /api/v1/users/123/orders/456/items/789/details

# Better - flatten with filters
GET /api/v1/order-items/789
GET /api/v1/order-items?orderId=456
```

## HTTP Methods

| Method | Usage | Idempotent | Body |
|--------|-------|------------|------|
| GET | Retrieve resource(s) | Yes | No |
| POST | Create resource | No | Yes |
| PUT | Replace resource | Yes | Yes |
| PATCH | Partial update | Yes | Yes |
| DELETE | Remove resource | Yes | No |

## Response Formats

### Single Resource
```json
{
  "id": "123",
  "type": "user",
  "attributes": {
    "name": "John Doe",
    "email": "john@example.com"
  },
  "relationships": {
    "organization": {
      "id": "456",
      "type": "organization"
    }
  },
  "meta": {
    "createdAt": "2024-01-15T10:30:00Z",
    "updatedAt": "2024-01-16T14:20:00Z"
  }
}
```

### Collection
```json
{
  "data": [
    { "id": "1", ... },
    { "id": "2", ... }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalPages": 5,
    "totalItems": 100
  },
  "links": {
    "self": "/api/v1/users?page=1",
    "next": "/api/v1/users?page=2",
    "prev": null
  }
}
```

### Error Response
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "email",
        "message": "Must be a valid email address"
      }
    ],
    "requestId": "req_abc123"
  }
}
```

## Status Codes

### Success
- `200 OK` - Successful GET, PUT, PATCH
- `201 Created` - Successful POST
- `204 No Content` - Successful DELETE

### Client Errors
- `400 Bad Request` - Invalid request body/params
- `401 Unauthorized` - Missing/invalid authentication
- `403 Forbidden` - Authenticated but not authorized
- `404 Not Found` - Resource doesn't exist
- `409 Conflict` - Resource conflict (duplicate)
- `422 Unprocessable Entity` - Validation failed

### Server Errors
- `500 Internal Server Error` - Unexpected server error
- `503 Service Unavailable` - Temporary unavailability

## Query Parameters

### Filtering
```
GET /api/v1/users?status=active&role=admin
GET /api/v1/orders?createdAfter=2024-01-01
```

### Sorting
```
GET /api/v1/users?sort=createdAt:desc
GET /api/v1/users?sort=lastName:asc,firstName:asc
```

### Pagination
```
GET /api/v1/users?page=2&pageSize=20
GET /api/v1/users?cursor=abc123&limit=20
```

### Field Selection
```
GET /api/v1/users?fields=id,name,email
GET /api/v1/users?include=organization,orders
```

## Versioning
```
# URL versioning (preferred)
/api/v1/users
/api/v2/users

# Header versioning (alternative)
Accept: application/vnd.api+json; version=1
```

## Design Checklist
- [ ] URLs use nouns, not verbs
- [ ] Resources are plural
- [ ] HTTP methods used correctly
- [ ] Consistent response format
- [ ] Proper status codes
- [ ] Pagination for collections
- [ ] Version in URL
- [ ] Error responses informative
```

### Example 5: Testing Standards Skill

**File:** `.claude/skills/testing-standards.md`

```markdown
---
name: testing-standards
description: Enforce team testing standards including coverage, naming, and structure
allowed_tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
---

# Testing Standards

## Overview
Ensure all code meets our team's testing requirements.

## Coverage Requirements

| Type | Minimum Coverage |
|------|------------------|
| Unit Tests | 80% line coverage |
| Integration Tests | Critical paths |
| E2E Tests | Happy paths |

## Test File Organization

```
src/
├── services/
│   └── userService.ts
└── __tests__/
    └── services/
        └── userService.test.ts

# Or colocated:
src/
├── services/
│   ├── userService.ts
│   └── userService.test.ts
```

## Naming Conventions

### Test Files
```
✓ userService.test.ts
✓ userService.spec.ts
✗ userServiceTests.ts
✗ test_userService.ts
```

### Test Descriptions
```typescript
// Good - describes behavior
describe('UserService', () => {
  describe('createUser', () => {
    it('should create a user with valid data', () => {});
    it('should throw ValidationError for invalid email', () => {});
    it('should hash password before storing', () => {});
  });
});

// Bad - describes implementation
describe('UserService', () => {
  it('test createUser', () => {});
  it('createUser works', () => {});
});
```

## Test Structure: AAA Pattern

```typescript
it('should calculate total with discount', () => {
  // Arrange - Set up test data
  const items = [
    { price: 100, quantity: 2 },
    { price: 50, quantity: 1 }
  ];
  const discount = 0.1;

  // Act - Execute the code
  const result = calculateTotal(items, discount);

  // Assert - Verify the result
  expect(result).toBe(225); // (200 + 50) * 0.9
});
```

## Mocking Guidelines

### When to Mock
- External services (APIs, databases)
- Time-dependent code
- Random number generation
- File system operations
- Environment-specific code

### When NOT to Mock
- Pure functions
- Data transformations
- Internal utilities
- Simple value objects

### Mock Examples
```typescript
// Good - mock external dependency
jest.mock('../services/emailService');
const mockSendEmail = emailService.send as jest.Mock;
mockSendEmail.mockResolvedValue({ sent: true });

// Bad - over-mocking
jest.mock('../utils/formatDate'); // Don't mock pure functions
```

## Test Categories

### Unit Tests
```typescript
// Test single function/method in isolation
describe('formatCurrency', () => {
  it('should format USD correctly', () => {
    expect(formatCurrency(1234.56, 'USD')).toBe('$1,234.56');
  });
});
```

### Integration Tests
```typescript
// Test multiple components together
describe('Order Creation Flow', () => {
  it('should create order and update inventory', async () => {
    const order = await orderService.create(orderData);
    const inventory = await inventoryService.get(productId);

    expect(order.status).toBe('created');
    expect(inventory.quantity).toBe(initialQuantity - orderData.quantity);
  });
});
```

### E2E Tests
```typescript
// Test full user flows
describe('Checkout Flow', () => {
  it('should complete purchase successfully', async () => {
    await page.goto('/products');
    await page.click('[data-testid="add-to-cart"]');
    await page.click('[data-testid="checkout"]');
    await page.fill('[name="cardNumber"]', '4242424242424242');
    await page.click('[data-testid="pay"]');

    await expect(page.locator('.success-message')).toBeVisible();
  });
});
```

## Edge Cases Checklist
- [ ] Empty inputs (null, undefined, [], '')
- [ ] Boundary values (0, -1, MAX_INT)
- [ ] Invalid types
- [ ] Network failures
- [ ] Timeout scenarios
- [ ] Concurrent operations
- [ ] Unicode/special characters

## Review Checklist
- [ ] Tests exist for new code
- [ ] Tests follow AAA pattern
- [ ] Descriptive test names
- [ ] No test interdependencies
- [ ] Mocks are appropriate
- [ ] Edge cases covered
- [ ] Tests run in isolation
- [ ] No flaky tests
```

### Example 6: Commit Message Formatter Skill

**File:** `.claude/skills/commit-format.md`

```markdown
---
name: commit-format
description: Format commit messages following Conventional Commits standard
allowed_tools:
  - Read
  - Bash
  - Grep
---

# Commit Message Formatter

## Overview
Generate properly formatted commit messages following Conventional Commits.

## Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

## Types

| Type | Description | Example |
|------|-------------|---------|
| `feat` | New feature | feat: add user registration |
| `fix` | Bug fix | fix: resolve login timeout |
| `docs` | Documentation | docs: update API reference |
| `style` | Formatting | style: fix indentation |
| `refactor` | Code restructuring | refactor: extract validation |
| `perf` | Performance | perf: optimize query |
| `test` | Tests | test: add unit tests |
| `chore` | Maintenance | chore: update dependencies |
| `ci` | CI/CD changes | ci: add GitHub Actions |
| `build` | Build system | build: configure webpack |

## Scope (Optional)
Indicates the affected component:
- `auth`, `api`, `ui`, `db`, `config`

## Subject Rules
1. Imperative mood ("add" not "added")
2. No capitalization
3. No period at end
4. Max 50 characters

## Body Rules
1. Explain "what" and "why"
2. Wrap at 72 characters
3. Blank line before body

## Footer Rules
1. Reference issues: `Fixes #123`
2. Breaking changes: `BREAKING CHANGE: description`
3. Co-authors: `Co-authored-by: Name <email>`

## Examples

### Simple Feature
```
feat(auth): add password reset functionality
```

### Bug Fix with Body
```
fix(api): resolve race condition in order processing

Multiple concurrent requests could create duplicate orders.
Added database transaction and optimistic locking to prevent
this issue.

Fixes #456
```

### Breaking Change
```
feat(api)!: change authentication to JWT

Migrate from session-based auth to JWT tokens for better
scalability and stateless operation.

BREAKING CHANGE: All API clients must include JWT token
in Authorization header. Session cookies no longer work.

Migration guide: docs/jwt-migration.md
```

### Multiple Scopes
```
refactor(auth,api): extract validation middleware

Moved validation logic from individual routes to shared
middleware for consistency and reduced duplication.
```

## Generation Process

1. Analyze `git diff --staged`
2. Identify primary change type
3. Determine scope from affected files
4. Write imperative subject
5. Add body if changes are complex
6. Include footer references

## Anti-patterns

```
# Bad - vague
git commit -m "fix stuff"
git commit -m "updates"
git commit -m "WIP"

# Bad - past tense
git commit -m "fixed the bug"
git commit -m "added feature"

# Bad - too long
git commit -m "add new user registration feature with email verification and password requirements"
```

## Quick Reference

```bash
# Feature
git commit -m "feat(scope): add something"

# Fix
git commit -m "fix(scope): resolve issue"

# Breaking change (note the !)
git commit -m "feat(scope)!: breaking change"

# With body (use heredoc)
git commit -m "$(cat <<'EOF'
feat(auth): add OAuth2 support

Implement OAuth2 authorization code flow for Google and
GitHub providers. Users can now link social accounts.

Fixes #789
EOF
)"
```
```

---

## 7. Creating Custom Skills

### Step-by-Step Guide

#### Step 1: Identify the Need

Ask yourself:
- Is this task performed repeatedly?
- Does it require specific knowledge or patterns?
- Would the team benefit from consistency?
- Is there a checklist or process to follow?

**Good skill candidates:**
- Code review for specific frameworks
- Company-specific documentation standards
- Domain-specific design patterns
- Release processes

#### Step 2: Create the Skill File

```bash
# Create skills directory
mkdir -p .claude/skills

# Create skill file
touch .claude/skills/my-skill.md
```

#### Step 3: Write the Frontmatter

```yaml
---
name: my-skill
description: Clear, specific description of what this skill does
allowed_tools:
  - Read
  - Grep
  - Glob
---
```

#### Step 4: Structure the Content

```markdown
# Skill Title

## Overview
One paragraph explaining:
- What this skill does
- When to use it
- Expected outcomes

## Process
Step-by-step instructions Claude should follow.

## Checklist
- [ ] Item 1
- [ ] Item 2

## Examples
Show expected inputs and outputs.

## Anti-patterns
What NOT to do.
```

#### Step 5: Test the Skill

```bash
# In Claude Code
/my-skill

# Or
"Use the my-skill skill to review this code"
```

#### Step 6: Iterate and Improve

- Note where Claude deviates from expectations
- Add more specific instructions
- Include more examples
- Add anti-patterns to avoid

### Best Practices for Descriptions

**DO:**
```yaml
# Specific about scope
description: Review React components for accessibility, hooks usage, and performance patterns

# Clear about when to use
description: Generate database migration files for PostgreSQL schema changes

# Mention key capabilities
description: Analyze Python code for type hints, docstrings, and PEP 8 compliance
```

**DON'T:**
```yaml
# Too vague
description: Reviews code

# Too long
description: This skill will review your code and look for many different types of issues including bugs, security problems, performance issues, style violations, and more

# Redundant
description: A skill that helps with code review
```

### Tool Restrictions

**Read-only skill (safest):**
```yaml
allowed_tools:
  - Read
  - Grep
  - Glob
```

**Analysis + suggestions:**
```yaml
allowed_tools:
  - Read
  - Grep
  - Glob
  - Edit  # Can suggest edits
```

**Full automation:**
```yaml
allowed_tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
```

### Supporting Files

Skills can reference supporting files:

```markdown
# My Skill

## Reference Materials
See the following project files for standards:
- `docs/CODING_STANDARDS.md` - Code style guide
- `docs/API_CONVENTIONS.md` - API patterns
- `.eslintrc.json` - Linting rules

When reviewing, ensure code follows these documents.
```

### Progressive Disclosure

Structure skills from simple to detailed:

```markdown
# API Review Skill

## Quick Check (always do)
- [ ] URLs use nouns
- [ ] Proper HTTP methods
- [ ] Consistent naming

## Standard Review (default)
- [ ] All quick checks
- [ ] Error handling
- [ ] Authentication
- [ ] Pagination

## Deep Review (on request)
- [ ] All standard checks
- [ ] Performance implications
- [ ] Caching strategy
- [ ] Rate limiting
- [ ] Documentation completeness
```

---

## 8. Skill Triggers

### Automatic Detection

Skills can activate automatically based on context:

**File type detection:**
```
User: "Review this Python file"
Claude: (Detects .py file, may activate python-related skills)
```

**Keyword detection:**
```
User: "I need to do a code review"
Claude: (Detects "code review", may activate code-review skill)
```

**Task type detection:**
```
User: "Help me write tests for this function"
Claude: (Detects testing intent, may activate testing skill)
```

### Manual Invocation (/skill-name)

Explicitly invoke a skill:

```bash
# Direct invocation
/code-review

# With context
/code-review Review the changes in the last commit

# Combined with other text
"I'm ready to submit this PR. /pr-template"
```

### When Skills Activate

Skills activate when:

1. **Explicit invocation:** User types `/skill-name`
2. **Direct mention:** "Use the code-review skill"
3. **Context matching:** Task matches skill description
4. **File type matching:** Working with files the skill handles

Skills do NOT activate when:
1. User explicitly excludes: "Without using skills..."
2. Task doesn't match any skill
3. Skill is not available in current context

### Controlling Activation

**Force activation:**
```
"Use the testing-standards skill for this review"
/testing-standards
```

**Prevent activation:**
```
"Give me quick feedback without using the full code-review process"
"Just a simple review, no need for the detailed checklist"
```

---

## 9. Pros and Cons

### Pros

#### 1. Context Sharing Benefits
- Full access to conversation history
- No need to re-explain context
- Can reference earlier discussions
- Maintains continuity

#### 2. Instant Activation
- No process spawning
- No token overhead for task description
- Immediate response
- No cold start delay

#### 3. Consistent Quality
- Enforces standards across team
- Reproducible results
- Encoded best practices
- Reduces human error

#### 4. Easy Maintenance
- Plain markdown files
- Version controlled
- Easy to update
- Team can contribute

#### 5. Cost Effective
- No additional API calls
- No subagent overhead
- Efficient token usage
- Scales well

### Cons

#### 1. Sequential Only
- Cannot run skills in parallel
- One skill at a time
- Blocking operation
- No concurrency

#### 2. Context Pollution
- Skill instructions consume context
- Long skills reduce available space
- May affect other tasks
- Potential overflow

#### 3. Limited Isolation
- Skills see entire conversation
- No sandboxing
- Mistakes affect main context
- No rollback

#### 4. Maintenance Overhead
- Skills need updates
- Can become stale
- Require documentation
- Team training needed

#### 5. Discovery Challenge
- Users may not know skills exist
- No automatic suggestions
- Manual invocation required
- Learning curve

### When Skills Add Overhead

Skills may be counterproductive when:

1. **Simple tasks:** One-off tasks don't need structured approach
2. **Exploratory work:** Uncertain requirements don't fit checklists
3. **Time-sensitive:** Detailed review adds time
4. **Unique situations:** Non-standard cases don't match skill patterns

---

## 10. When to Use Skills

### Ideal Scenarios

| Scenario | Why Skills Work |
|----------|-----------------|
| Code reviews | Consistent checklist, no isolation needed |
| Documentation | Style guides benefit from standards |
| PR preparation | Template-based, context-aware |
| Design reviews | Patterns and principles |
| Debugging | Systematic methodology |
| Testing strategy | Coverage requirements |

### Skills vs Subagents Decision

Use **Skills** when:
- Task requires conversation context
- Task is advisory (review, analyze)
- Instant response needed
- Single focused task
- Team standards apply

Use **Subagents** when:
- Tasks are independent
- Parallel execution beneficial
- Task needs isolation
- Multiple file operations
- Long-running operations

### Skills vs Hooks

Use **Skills** when:
- Interactive guidance needed
- Context-dependent decisions
- User involvement required
- Advisory output

Use **Hooks** when:
- Automatic execution required
- No user interaction
- Pre/post command processing
- Validation before commit

### Decision Flowchart

```
                    ┌─────────────────────────┐
                    │   What type of task?    │
                    └───────────┬─────────────┘
                                │
            ┌───────────────────┼───────────────────┐
            ▼                   ▼                   ▼
    ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
    │   Advisory   │   │  Execution   │   │  Automation  │
    │   (review,   │   │  (implement, │   │  (validate,  │
    │   analyze)   │   │   build)     │   │   format)    │
    └──────┬───────┘   └──────┬───────┘   └──────┬───────┘
           │                  │                   │
           ▼                  ▼                   ▼
    ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
    │    SKILL     │   │Need parallel │   │    HOOK      │
    │              │   │ execution?   │   │              │
    └──────────────┘   └──────┬───────┘   └──────────────┘
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
             ┌──────────┐        ┌──────────┐
             │   Yes    │        │    No    │
             └────┬─────┘        └────┬─────┘
                  │                   │
                  ▼                   ▼
           ┌──────────────┐   ┌──────────────┐
           │  SUBAGENT    │   │    SKILL     │
           │              │   │  (if pattern │
           │              │   │   exists)    │
           └──────────────┘   └──────────────┘
```

---

## 11. Comparison Table

### Skills vs Subagents

| Feature | Skills | Subagents |
|---------|--------|-----------|
| **Execution Model** | Same context | Separate process |
| **Context Access** | Full conversation | Task description only |
| **Activation Time** | Instant | Process spawn overhead |
| **Parallelism** | No | Yes |
| **Token Cost** | Lower | Higher (task + context) |
| **Isolation** | None | Full |
| **Error Impact** | Affects conversation | Contained |
| **Best For** | Advisory, review | Execution, parallel work |
| **Output** | Inline response | Task completion |
| **User Interaction** | Continuous | At completion |
| **File Location** | .claude/skills/ | N/A (code-based) |
| **Complexity** | Low | Medium |

### Skills vs Slash Commands

| Feature | Skills | Slash Commands |
|---------|--------|----------------|
| **Definition** | Markdown files | Built into Claude Code |
| **Customizable** | Yes | No |
| **Purpose** | Domain expertise | System operations |
| **Examples** | /code-review | /help, /clear, /status |
| **Extensible** | Yes | No |
| **Invocation** | /skill-name | /command |
| **Context Aware** | Yes | Limited |

### Skills vs Hooks

| Feature | Skills | Hooks |
|---------|--------|-------|
| **Activation** | Manual/detected | Automatic on events |
| **User Interaction** | Yes | No (background) |
| **Blocking** | Optional | Configurable |
| **Use Case** | Advisory, guidance | Validation, automation |
| **Definition** | Markdown | TOML + scripts |
| **Output** | Conversation | Logs/side effects |
| **Examples** | Code review | Pre-commit validation |
| **Error Handling** | Interactive | Fail/continue |

### Detailed Feature Comparison

| Capability | Skills | Subagents | Hooks |
|------------|--------|-----------|-------|
| Read files | ✓ | ✓ | ✓ |
| Write files | ✓ | ✓ | ✓ |
| Run commands | ✓ | ✓ | ✓ |
| Parallel execution | ✗ | ✓ | ✗ |
| Context isolation | ✗ | ✓ | ✓ |
| User prompts | ✓ | ✗ | ✗ |
| Automatic trigger | Partial | ✗ | ✓ |
| Custom logic | ✗ | ✓ | ✓ |
| Conversation access | ✓ | ✗ | ✗ |
| Team sharing | ✓ | ✓ | ✓ |
| Version control | ✓ | ✓ | ✓ |

---

## 12. Advanced Patterns

### Skills with Forked Context

Forked context creates a snapshot, allowing experimental analysis:

```yaml
---
name: what-if-analysis
description: Analyze code changes without affecting main context
context:
  fork: true
allowed_tools:
  - Read
  - Grep
  - Glob
  - Edit  # Edits won't persist due to fork
---

# What-If Analysis

## Overview
Explore code changes in isolation. Changes made during this
skill's execution will not persist to the main conversation.

## Use Cases
- Exploring refactoring options
- Testing different approaches
- Impact analysis

## Process
1. Make experimental changes
2. Analyze results
3. Report findings
4. User decides whether to apply in main context
```

**Usage:**
```
/what-if-analysis

"What would happen if we changed this function to async?"
```

### Skills Calling Subagents

Skills can spawn subagents for parallel work:

```markdown
---
name: parallel-review
description: Comprehensive review using parallel subagents
allowed_tools:
  - Read
  - Grep
  - Glob
  - Task  # Allows spawning subagents
---

# Parallel Review Skill

## Overview
Perform comprehensive review by delegating to specialized subagents.

## Process

### Step 1: Identify Files
Use Glob to find all files to review.

### Step 2: Spawn Subagents
For large reviews, spawn subagents using the Task tool:

```
Spawn subagent with task:
"Review file X for security issues. Report:
- Critical issues
- Warnings
- Suggestions"
```

### Step 3: Aggregate Results
Collect subagent reports and summarize:
- Overall security posture
- Critical findings
- Prioritized recommendations

## Example Subagent Tasks

### Security Review Agent
```
Task: Review authentication code in src/auth/
Focus on:
- Credential handling
- Session management
- Input validation
Return: JSON with findings
```

### Performance Review Agent
```
Task: Analyze database queries in src/services/
Focus on:
- N+1 queries
- Missing indexes
- Unbounded queries
Return: JSON with findings
```
```

### Skill Chains

Skills can reference other skills:

```markdown
---
name: full-pr-review
description: Complete PR review combining multiple specialized skills
allowed_tools:
  - Read
  - Grep
  - Glob
---

# Full PR Review

## Overview
Comprehensive PR review combining multiple review perspectives.

## Process

### Phase 1: Code Quality
Invoke the code-review skill patterns:
- Check correctness
- Check maintainability
- Check documentation

### Phase 2: Testing
Apply testing-standards skill patterns:
- Verify test coverage
- Check test quality
- Identify missing tests

### Phase 3: Security
Apply security-review patterns:
- Check for vulnerabilities
- Verify authentication
- Check authorization

### Phase 4: API Design (if applicable)
Apply api-design patterns:
- Check RESTful conventions
- Verify response formats
- Check error handling

## Final Report

```markdown
## PR Review Summary

### Code Quality: [PASS/FAIL]
- Findings...

### Testing: [PASS/FAIL]
- Findings...

### Security: [PASS/FAIL]
- Findings...

### API Design: [PASS/FAIL/N/A]
- Findings...

## Recommendation
[ ] Approve
[ ] Request Changes
[ ] Needs Discussion
```
```

### Skills with External Services

Skills can guide integration with external services:

```markdown
---
name: jira-integration
description: Create and update Jira tickets from code analysis
allowed_tools:
  - Read
  - Grep
  - Bash  # For API calls
---

# Jira Integration Skill

## Overview
Analyze code and create/update Jira tickets for issues found.

## Prerequisites
- JIRA_API_TOKEN environment variable set
- JIRA_BASE_URL environment variable set

## Process

### Finding Issues
1. Analyze code for TODO comments
2. Identify FIXME markers
3. Find deprecated usage

### Creating Tickets
For each issue, format a Jira API call:

```bash
curl -X POST "$JIRA_BASE_URL/rest/api/3/issue" \
  -H "Authorization: Bearer $JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "fields": {
      "project": {"key": "PROJ"},
      "summary": "Issue title",
      "description": "Details from code analysis",
      "issuetype": {"name": "Task"}
    }
  }'
```

## Output Format
- List of created tickets with URLs
- Summary of issues found
- Recommendations for prioritization
```

---

## 13. Quick Reference

### Skill File Template

```markdown
---
name: skill-name
description: One-line description of what this skill does
allowed_tools:
  - Read
  - Grep
  - Glob
---

# Skill Title

## Overview
Brief description of the skill's purpose and when to use it.

## When to Use
- Scenario 1
- Scenario 2

## Process
1. Step one
2. Step two
3. Step three

## Checklist
- [ ] Check item 1
- [ ] Check item 2
- [ ] Check item 3

## Examples

### Input
```
Example input or scenario
```

### Output
```
Expected output or response
```

## Anti-patterns
- What NOT to do
- Common mistakes

## Related Skills
- other-skill-name
```

### Common Patterns

**Read-only analysis:**
```yaml
allowed_tools:
  - Read
  - Grep
  - Glob
```

**Code modification:**
```yaml
allowed_tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
```

**Full automation:**
```yaml
allowed_tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
  - Task
```

**Checklist format:**
```markdown
## Review Checklist
- [ ] Item 1
- [ ] Item 2
- [ ] Item 3
```

**Progressive depth:**
```markdown
## Quick Review
- [ ] Essential check 1
- [ ] Essential check 2

## Standard Review
- [ ] All quick checks plus...
- [ ] Standard check 1

## Deep Review
- [ ] All standard checks plus...
- [ ] Deep check 1
```

### Troubleshooting

#### Skill Not Found

**Problem:** `/my-skill` returns "skill not found"

**Solutions:**
1. Check file location:
   ```bash
   ls .claude/skills/my-skill.md
   ```
2. Verify frontmatter syntax:
   ```yaml
   ---
   name: my-skill  # Must match invocation
   description: Description here
   ---
   ```
3. Check for YAML errors (proper spacing, no tabs)

#### Skill Not Activating Automatically

**Problem:** Skill doesn't activate when expected

**Solutions:**
1. Use explicit invocation: `/my-skill`
2. Mention skill directly: "Use the my-skill skill"
3. Check description matches your use case

#### Skill Behaving Unexpectedly

**Problem:** Skill output doesn't match expectations

**Solutions:**
1. Add more specific instructions
2. Include examples of expected output
3. Add anti-patterns section
4. Test with explicit invocation

#### Tool Restrictions Not Working

**Problem:** Skill uses tools not in `allowed_tools`

**Solutions:**
1. Verify YAML syntax:
   ```yaml
   allowed_tools:
     - Read  # Note: proper list format
     - Grep
   ```
2. Check tool name spelling
3. Restart Claude Code session

#### Context Overflow

**Problem:** Long skills cause context issues

**Solutions:**
1. Split into multiple focused skills
2. Use progressive disclosure (quick/standard/deep)
3. Reference external files instead of inline content
4. Reduce examples to essential cases

### Skill File Locations Summary

| Location | Priority | Shared | Use Case |
|----------|----------|--------|----------|
| `.claude/skills/` | Highest | Team | Project standards |
| `~/.claude/skills/` | Medium | Personal | Cross-project |
| Plugin skills | Lowest | Varies | Extensions |

### Quick Commands

```bash
# List skills
"What skills are available?"

# Use skill
/skill-name

# Use with context
/skill-name analyze this code

# Prevent skill use
"Without skills, just tell me..."
```

---

## Conclusion

Skills are a powerful way to extend Claude Code with domain-specific expertise. They provide:

- **Consistency**: Standardized approaches across your team
- **Efficiency**: No spawn overhead like subagents
- **Flexibility**: Easy to create, modify, and share
- **Integration**: Full access to conversation context

Start with built-in skills, then create custom ones as you identify repeated patterns in your workflow. Remember: skills are for guidance and expertise, while subagents are for delegated execution.

### Next Steps

1. Try the built-in `/code-review` skill
2. Create a simple custom skill for your project
3. Share useful skills with your team
4. Iterate based on feedback

### Related Tutorials

- Tutorial 01: Introduction to Claude Code
- Tutorial 02: Hooks and Automation
- Tutorial 03: Subagents for Parallel Work
- Tutorial 04: MCP Server Integration
- **Tutorial 05: Skills (this document)**
- Tutorial 06: Advanced Patterns and Workflows
