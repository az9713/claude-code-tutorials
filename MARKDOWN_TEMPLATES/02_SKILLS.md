# Claude Code Skills - Complete Guide

> **Skills** are reusable workflows, checklists, and process guides that can be invoked with `/skill-name`. They provide consistent, repeatable processes for common development tasks.

---

## Quick Reference

| Property | Value |
|----------|-------|
| **Location** | `.claude/skills/<skill-name>/SKILL.md` |
| **Frontmatter** | Required |
| **Invocation** | `/skill-name` or `/skill-name arguments` |
| **Arguments** | Available as `$ARGUMENTS` placeholder |
| **Context** | Inherits conversation context (or fork with `context: fork`) |

---

## Template Anatomy

```yaml
---
# REQUIRED FIELDS
name: skill-name               # Unique identifier (kebab-case)
description: One-line purpose  # Shown in /help output

# OPTIONAL FIELDS
allowed-tools:                 # Restrict available tools
  - Read
  - Grep
  - Bash
context: inherit               # inherit (default) | fork
---

# Skill Instructions (Markdown Body)

The markdown body contains the workflow, checklist,
or process guide. Use $ARGUMENTS to reference user input.

## Example with Arguments

Analyze: $ARGUMENTS
```

### Context Modes

| Mode | Behavior | Use Case |
|------|----------|----------|
| `inherit` | Sees full conversation | Contextual workflows |
| `fork` | Fresh context, just skill | Isolated operations |

---

## Templates

### Minimal Template

```markdown
---
name: quick-check
description: Quick code quality check
---

## Checklist

- [ ] Code compiles
- [ ] Tests pass
- [ ] No linting errors
```

### Standard Template

```markdown
---
name: code-review
description: Structured code review workflow
---

# Code Review

## Target
$ARGUMENTS

## Checklist

### Correctness
- [ ] Logic is correct
- [ ] Edge cases handled
- [ ] Error handling present

### Quality
- [ ] Follows conventions
- [ ] Well-named variables
- [ ] No duplication

### Security
- [ ] No hardcoded secrets
- [ ] Input validated
- [ ] No injection risks

## Output

Provide findings in this format:
- **Location**: file:line
- **Issue**: Description
- **Severity**: high/medium/low
- **Fix**: Suggestion
```

### Advanced Template

```markdown
---
name: comprehensive-review
description: Multi-phase review with severity ratings
allowed-tools:
  - Read
  - Glob
  - Grep
context: inherit
---

# Comprehensive Code Review

## Phase 1: Understanding

Before reviewing, understand:
1. What does this code do?
2. Why was it written this way?
3. What are the requirements?

## Phase 2: Analysis

### Correctness Dimension
| Check | Status | Notes |
|-------|--------|-------|
| Logic correct | | |
| Edge cases | | |
| Error handling | | |

### Security Dimension
| Check | Status | Notes |
|-------|--------|-------|
| Input validation | | |
| Authentication | | |
| Data exposure | | |

### Performance Dimension
| Check | Status | Notes |
|-------|--------|-------|
| Algorithmic complexity | | |
| Resource usage | | |
| Caching | | |

### Maintainability Dimension
| Check | Status | Notes |
|-------|--------|-------|
| Code clarity | | |
| Documentation | | |
| Test coverage | | |

## Phase 3: Reporting

### Finding Template

\`\`\`
## [SEVERITY] Issue Title

**File**: path/to/file.ts:42
**Dimension**: Security

### Problem
Description of the issue.

### Current Code
\\\`\\\`\\\`typescript
// problematic code
\\\`\\\`\\\`

### Suggested Fix
\\\`\\\`\\\`typescript
// fixed code
\\\`\\\`\\\`

### Impact
What could happen if not fixed.
\`\`\`

## Severity Definitions

| Level | Meaning | Example |
|-------|---------|---------|
| **CRITICAL** | Security vulnerability or data loss | SQL injection |
| **HIGH** | Significant bug or design flaw | Race condition |
| **MEDIUM** | Code quality issue | Missing error handling |
| **LOW** | Minor improvement | Naming convention |
| **INFO** | Suggestion only | Alternative approach |
```

---

## Examples

### Example 1: Code Review Checklist

```markdown
---
name: code-review-checklist
description: Multi-dimensional code review with severity ratings
---

# Code Review Checklist

## Target
Review: $ARGUMENTS

## Pre-Review

Before starting:
- [ ] Understand the change purpose
- [ ] Read related tests
- [ ] Check PR description

## Review Dimensions

### 1. Correctness

- [ ] **Logic** - Algorithm is correct
- [ ] **Edge Cases** - Null, empty, boundary values handled
- [ ] **Error Handling** - Errors caught and handled appropriately
- [ ] **Type Safety** - Types are correct and strict

### 2. Security

- [ ] **Secrets** - No hardcoded credentials or API keys
- [ ] **Injection** - Input sanitized (SQL, XSS, command)
- [ ] **Auth** - Proper authentication/authorization checks
- [ ] **Data** - Sensitive data not exposed in logs/errors

### 3. Performance

- [ ] **Complexity** - No O(n²) when O(n) possible
- [ ] **Resources** - No memory leaks or excessive allocation
- [ ] **I/O** - Async operations used appropriately
- [ ] **Caching** - Repeated expensive operations cached

### 4. Maintainability

- [ ] **Naming** - Clear, descriptive names
- [ ] **Structure** - Single responsibility, small functions
- [ ] **Comments** - Complex logic explained
- [ ] **Tests** - Adequate test coverage

### 5. Project Standards

- [ ] **Style** - Follows project conventions
- [ ] **Patterns** - Uses established patterns
- [ ] **Dependencies** - No unnecessary new dependencies

## Output Format

For each finding:

```
### [file:line] SEVERITY - Category

**Issue**: What's wrong

**Why**: Why it matters

**Fix**:
\`\`\`diff
- bad code
+ good code
\`\`\`
```

## Summary Template

```
# Review Summary

## Verdict: APPROVE | REQUEST_CHANGES | COMMENT

## Statistics
- Critical: X
- High: X
- Medium: X
- Low: X

## Key Findings
1. ...
2. ...

## Required Changes (if any)
- ...
```
```

### Example 2: TDD Workflow

```markdown
---
name: tdd-workflow
description: Red-Green-Refactor cycle for test-driven development
---

# Test-Driven Development Workflow

## Target Feature
$ARGUMENTS

## The TDD Cycle

```
    ┌─────────────────────────────────────┐
    │                                     │
    │    ┌───────┐                        │
    │    │  RED  │ Write failing test     │
    │    └───┬───┘                        │
    │        │                            │
    │        ▼                            │
    │    ┌───────┐                        │
    │    │ GREEN │ Make it pass           │
    │    └───┬───┘                        │
    │        │                            │
    │        ▼                            │
    │  ┌──────────┐                       │
    │  │ REFACTOR │ Clean up              │
    │  └────┬─────┘                       │
    │       │                             │
    │       └─────────────────────────────┘
    │
```

## Phase 1: RED (Write Failing Test)

### Checklist
- [ ] Understand the requirement
- [ ] Write the test FIRST
- [ ] Test should fail (proves test works)
- [ ] Test name describes behavior

### Test Naming Convention
\`\`\`
test_[unit]_[scenario]_[expected_result]
it('should [behavior] when [condition]')
\`\`\`

### Example
\`\`\`typescript
describe('calculateTotal', () => {
  it('should return sum of item prices', () => {
    const items = [{ price: 10 }, { price: 20 }];
    expect(calculateTotal(items)).toBe(30);
  });
});
\`\`\`

### Verify RED
\`\`\`bash
npm test -- --grep "calculateTotal"
# Should FAIL - function doesn't exist yet
\`\`\`

## Phase 2: GREEN (Make It Pass)

### Checklist
- [ ] Write MINIMAL code to pass
- [ ] Don't over-engineer
- [ ] Don't add features not tested
- [ ] Test passes

### Implementation
\`\`\`typescript
function calculateTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}
\`\`\`

### Verify GREEN
\`\`\`bash
npm test -- --grep "calculateTotal"
# Should PASS
\`\`\`

## Phase 3: REFACTOR (Clean Up)

### Checklist
- [ ] Remove duplication
- [ ] Improve naming
- [ ] Simplify logic
- [ ] Tests still pass

### Questions to Ask
1. Is there duplication?
2. Are names clear?
3. Is the code simple?
4. Does it follow project patterns?

### Verify REFACTOR
\`\`\`bash
npm test
# ALL tests should still PASS
\`\`\`

## Repeat

Continue the cycle for each new requirement:
1. New test (RED)
2. Implementation (GREEN)
3. Clean up (REFACTOR)

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Solution |
|--------------|---------|----------|
| Test after | Tests confirm bias | Write test first |
| Too much GREEN | Over-engineering | Minimal code only |
| Skip REFACTOR | Technical debt | Always clean up |
| Multiple features | Confusion | One test at a time |
```

### Example 3: API Design Standards

```markdown
---
name: api-design-standards
description: RESTful API design patterns and conventions
---

# API Design Standards

## Designing API For
$ARGUMENTS

## URL Structure

### Resource Naming
\`\`\`
# GOOD - Nouns, plural
GET /users
GET /users/{id}
GET /users/{id}/posts

# BAD - Verbs, actions in URL
GET /getUsers
GET /user/get/{id}
POST /createUser
\`\`\`

### Hierarchy
\`\`\`
# Collection
GET /organizations

# Item in collection
GET /organizations/{orgId}

# Sub-collection
GET /organizations/{orgId}/members

# Item in sub-collection
GET /organizations/{orgId}/members/{memberId}
\`\`\`

## HTTP Methods

| Method | Usage | Idempotent | Safe |
|--------|-------|------------|------|
| GET | Read resource | Yes | Yes |
| POST | Create resource | No | No |
| PUT | Replace resource | Yes | No |
| PATCH | Partial update | No | No |
| DELETE | Remove resource | Yes | No |

## Request/Response Standards

### Request Body
\`\`\`json
{
  "name": "string (required)",
  "email": "string (required)",
  "role": "string (optional, default: 'user')"
}
\`\`\`

### Success Response
\`\`\`json
{
  "data": { ... },
  "meta": {
    "requestId": "uuid",
    "timestamp": "ISO8601"
  }
}
\`\`\`

### Error Response
\`\`\`json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Human readable message",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ]
  },
  "meta": {
    "requestId": "uuid",
    "timestamp": "ISO8601"
  }
}
\`\`\`

## Status Codes

### Success
| Code | Meaning | Use Case |
|------|---------|----------|
| 200 | OK | GET success, PUT/PATCH success |
| 201 | Created | POST created new resource |
| 204 | No Content | DELETE success |

### Client Errors
| Code | Meaning | Use Case |
|------|---------|----------|
| 400 | Bad Request | Validation error |
| 401 | Unauthorized | Auth required |
| 403 | Forbidden | No permission |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate/version conflict |
| 422 | Unprocessable | Semantic error |

### Server Errors
| Code | Meaning | Use Case |
|------|---------|----------|
| 500 | Server Error | Unexpected error |
| 502 | Bad Gateway | Upstream error |
| 503 | Unavailable | Maintenance |

## Pagination

\`\`\`
GET /users?page=2&limit=20

Response:
{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 150,
    "totalPages": 8,
    "hasNext": true,
    "hasPrev": true
  }
}
\`\`\`

## Filtering & Sorting

\`\`\`
# Filtering
GET /users?status=active&role=admin

# Sorting
GET /users?sort=createdAt:desc,name:asc

# Searching
GET /users?q=john
\`\`\`

## Versioning

\`\`\`
# URL versioning (recommended)
GET /v1/users
GET /v2/users

# Header versioning
GET /users
Accept: application/vnd.api+json;version=2
\`\`\`

## Checklist

- [ ] Resources are nouns, plural
- [ ] HTTP methods used correctly
- [ ] Consistent response format
- [ ] Proper status codes
- [ ] Pagination for lists
- [ ] Filtering/sorting supported
- [ ] Versioning strategy defined
- [ ] Error format standardized
```

### Example 4: Database Migration

```markdown
---
name: database-migration
description: Safe database migration workflow with rollback plan
---

# Database Migration Workflow

## Migration Purpose
$ARGUMENTS

## Pre-Migration Checklist

### Planning
- [ ] Migration script written
- [ ] Rollback script written
- [ ] Data backup verified
- [ ] Downtime window scheduled (if needed)
- [ ] Team notified

### Testing
- [ ] Tested on local
- [ ] Tested on staging
- [ ] Performance tested (large datasets)
- [ ] Rollback tested

## Migration Types

### 1. Additive (Safe)
\`\`\`sql
-- Add column (nullable or with default)
ALTER TABLE users ADD COLUMN nickname VARCHAR(50);

-- Add index
CREATE INDEX idx_users_email ON users(email);

-- Add table
CREATE TABLE user_preferences (...);
\`\`\`
**Rollback**: Usually safe, just remove added elements

### 2. Destructive (Dangerous)
\`\`\`sql
-- Drop column
ALTER TABLE users DROP COLUMN old_field;

-- Drop table
DROP TABLE deprecated_table;

-- Remove index
DROP INDEX idx_old_index;
\`\`\`
**Rollback**: Requires data restore from backup

### 3. Transformative (Complex)
\`\`\`sql
-- Rename column
ALTER TABLE users RENAME COLUMN name TO full_name;

-- Change type
ALTER TABLE users ALTER COLUMN age TYPE INTEGER;

-- Split/merge tables
-- (requires data migration script)
\`\`\`
**Rollback**: Requires inverse transformation

## Safe Migration Pattern

### Step 1: Add New (Don't Remove Old)
\`\`\`sql
-- Add new column
ALTER TABLE users ADD COLUMN email_new VARCHAR(255);
\`\`\`

### Step 2: Dual Write
\`\`\`typescript
// Write to both columns
user.email = value;
user.email_new = value;
\`\`\`

### Step 3: Backfill
\`\`\`sql
UPDATE users SET email_new = email WHERE email_new IS NULL;
\`\`\`

### Step 4: Switch Reads
\`\`\`typescript
// Read from new column
const email = user.email_new;
\`\`\`

### Step 5: Stop Dual Write
\`\`\`typescript
// Write only to new
user.email_new = value;
\`\`\`

### Step 6: Remove Old (After Verification)
\`\`\`sql
ALTER TABLE users DROP COLUMN email;
ALTER TABLE users RENAME COLUMN email_new TO email;
\`\`\`

## Migration Script Template

\`\`\`sql
-- Migration: 001_add_user_preferences
-- Description: Add user preferences table
-- Date: 2024-01-15
-- Author: developer@example.com

-- Up
BEGIN;

CREATE TABLE user_preferences (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  key VARCHAR(100) NOT NULL,
  value JSONB,
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(user_id, key)
);

CREATE INDEX idx_user_preferences_user_id ON user_preferences(user_id);

COMMIT;

-- Down
BEGIN;

DROP INDEX IF EXISTS idx_user_preferences_user_id;
DROP TABLE IF EXISTS user_preferences;

COMMIT;
\`\`\`

## Execution Checklist

### Before
- [ ] Backup database
- [ ] Verify backup integrity
- [ ] Test rollback script
- [ ] Monitor database metrics

### During
- [ ] Run migration in transaction
- [ ] Monitor for locks
- [ ] Watch for errors

### After
- [ ] Verify data integrity
- [ ] Check application functionality
- [ ] Monitor performance
- [ ] Keep backup for 7 days

## Rollback Triggers

Rollback immediately if:
- Migration fails
- Application errors spike
- Performance degrades significantly
- Data corruption detected

## Emergency Rollback

\`\`\`bash
# 1. Stop application
kubectl scale deployment app --replicas=0

# 2. Run rollback
psql -f rollback.sql

# 3. Verify data
psql -c "SELECT COUNT(*) FROM users;"

# 4. Restart application
kubectl scale deployment app --replicas=3
\`\`\`
```

### Example 5: Debugging Methodology

```markdown
---
name: debugging-methodology
description: Systematic debugging process for any issue
---

# Systematic Debugging

## Issue
$ARGUMENTS

## Phase 1: Reproduce

### Checklist
- [ ] Can reproduce consistently
- [ ] Identified exact steps
- [ ] Noted environment details
- [ ] Captured error messages

### Environment Details
```
- OS:
- Version:
- Browser (if applicable):
- Node/Python/etc version:
- Related dependencies:
```

### Reproduction Steps
1.
2.
3.

### Error Message
\`\`\`
[Paste exact error here]
\`\`\`

## Phase 2: Isolate

### Binary Search
\`\`\`
Working state ─────────────────── Bug introduced
     │                                  │
     ├─── Commit A (works)              │
     │                                  │
     ├─── Commit B (works)              │
     │                                  │
     ├─── Commit C (BROKEN) ◄───────────┘
     │
     └─── Commit D (broken)

git bisect start
git bisect bad HEAD
git bisect good <last-known-good>
\`\`\`

### Component Isolation
\`\`\`
Full System
    │
    ├── Frontend ───── Works?
    │
    ├── API ────────── Works?
    │
    ├── Database ───── Works?
    │
    └── External ───── Works?
\`\`\`

### Minimal Reproduction
- [ ] Removed unrelated code
- [ ] Simplified to smallest failing case
- [ ] Can share reproduction

## Phase 3: Hypothesize

### Potential Causes
| Hypothesis | Likelihood | Evidence |
|------------|------------|----------|
| | High/Med/Low | |
| | | |
| | | |

### Questions to Answer
1. What changed recently?
2. Does it fail in all environments?
3. Is it data-dependent?
4. Is it timing-dependent?
5. Does it affect all users?

## Phase 4: Test

### Testing Hypotheses

For each hypothesis:
\`\`\`
Hypothesis: [Description]

Test: [What I'll do to verify]

Expected if true: [What I expect to see]

Actual result: [What I observed]

Conclusion: [Confirmed/Rejected]
\`\`\`

### Logging Points
\`\`\`typescript
// Add strategic logging
console.log('[DEBUG] functionName input:', input);
console.log('[DEBUG] functionName state:', state);
console.log('[DEBUG] functionName output:', output);
\`\`\`

### Debugger Breakpoints
\`\`\`
- [ ] Before suspected code
- [ ] After suspected code
- [ ] In error handler
- [ ] At data transformation
\`\`\`

## Phase 5: Fix

### Fix Requirements
- [ ] Addresses root cause (not symptom)
- [ ] Doesn't break other functionality
- [ ] Has regression test
- [ ] Is minimal/focused

### Fix Template
\`\`\`diff
- problematic code
+ fixed code
\`\`\`

### Verification
- [ ] Original issue resolved
- [ ] No new errors introduced
- [ ] All tests pass
- [ ] Performance not degraded

## Phase 6: Document

### Bug Report
\`\`\`markdown
## Bug: [Title]

### Description
[What was wrong]

### Root Cause
[Why it happened]

### Fix
[What changed]

### Prevention
[How to avoid in future]

### Related
- Issue: #123
- PR: #456
\`\`\`

## Debugging Tools Cheatsheet

### JavaScript/Node
\`\`\`bash
node --inspect app.js          # Chrome DevTools
DEBUG=* node app.js            # Debug logging
node --trace-warnings app.js   # Warning traces
\`\`\`

### Python
\`\`\`python
import pdb; pdb.set_trace()    # Breakpoint
python -m pdb script.py        # Debug mode
\`\`\`

### General
\`\`\`bash
git bisect                     # Find bad commit
strace / ltrace                # System calls
tcpdump / wireshark            # Network
\`\`\`
```

### Example 6: Release Checklist

```markdown
---
name: release-checklist
description: Pre-release verification workflow
---

# Release Checklist

## Release
$ARGUMENTS

## Pre-Release Verification

### Code Quality
- [ ] All PRs merged to release branch
- [ ] No pending critical bugs
- [ ] Code review completed for all changes
- [ ] No TODO/FIXME in release code

### Testing
- [ ] All unit tests pass
- [ ] Integration tests pass
- [ ] E2E tests pass
- [ ] Performance tests pass
- [ ] Security scan completed
- [ ] Manual QA sign-off

### Documentation
- [ ] CHANGELOG updated
- [ ] API docs updated
- [ ] README current
- [ ] Migration guide (if breaking changes)
- [ ] Release notes drafted

### Dependencies
- [ ] Dependencies up to date
- [ ] No known vulnerabilities
- [ ] License compliance verified
- [ ] Lock file committed

## Build Verification

### Build Checks
\`\`\`bash
# Clean build
rm -rf node_modules dist
npm ci
npm run build

# Verify output
ls -la dist/

# Test production build
npm run start:prod
\`\`\`

### Artifact Checklist
- [ ] Build succeeds
- [ ] Artifacts correct size
- [ ] No dev dependencies in production
- [ ] Environment variables documented

## Deployment Preparation

### Infrastructure
- [ ] Staging environment ready
- [ ] Production environment ready
- [ ] Database migrations ready
- [ ] Rollback plan documented

### Monitoring
- [ ] Alerts configured
- [ ] Dashboards updated
- [ ] On-call scheduled
- [ ] Runbooks current

## Release Execution

### Version Bump
\`\`\`bash
# Semantic versioning
npm version [major|minor|patch]

# Or manual
# Update package.json version
# Update CHANGELOG.md
# Create git tag
\`\`\`

### Deployment Steps
1. [ ] Deploy to staging
2. [ ] Smoke test staging
3. [ ] Deploy to production (canary)
4. [ ] Monitor metrics
5. [ ] Full production rollout
6. [ ] Verify production

### Communication
- [ ] Team notified of deployment
- [ ] Status page updated
- [ ] Users notified (if needed)
- [ ] Social media (if major release)

## Post-Release

### Verification
- [ ] All services healthy
- [ ] No error spikes
- [ ] Performance nominal
- [ ] User feedback monitored

### Documentation
- [ ] GitHub release created
- [ ] Blog post (if major)
- [ ] Internal retrospective scheduled

## Rollback Procedure

If issues detected:

1. **Assess** - Critical or can wait?
2. **Decide** - Rollback or hotfix?
3. **Execute** - Run rollback playbook
4. **Communicate** - Notify stakeholders
5. **Postmortem** - Document learnings

\`\`\`bash
# Quick rollback
kubectl rollout undo deployment/app

# Or redeploy previous version
./deploy.sh v1.2.3
\`\`\`
```

### Example 7: Onboarding Guide

```markdown
---
name: onboarding-guide
description: New developer orientation workflow
---

# Developer Onboarding

## New Developer
$ARGUMENTS

## Day 1: Setup

### Accounts & Access
- [ ] GitHub organization access
- [ ] Slack workspace joined
- [ ] Email configured
- [ ] Calendar access
- [ ] 1Password/secrets access
- [ ] Cloud provider access (AWS/GCP/Azure)

### Development Environment
- [ ] Repository cloned
- [ ] Dependencies installed
- [ ] Environment variables set
- [ ] Local development running
- [ ] Tests passing locally

### Tools Installation
\`\`\`bash
# Required tools
brew install node git docker

# Project setup
git clone <repo>
cd <repo>
npm install
cp .env.example .env
npm run dev
\`\`\`

## Day 2-3: Codebase

### Architecture Overview
- [ ] Read architecture docs
- [ ] Understand module structure
- [ ] Review data flow
- [ ] Identify key components

### Key Files to Review
\`\`\`
├── README.md              # Start here
├── CLAUDE.md              # AI assistant context
├── package.json           # Dependencies & scripts
├── src/
│   ├── index.ts           # Entry point
│   ├── config/            # Configuration
│   ├── services/          # Business logic
│   └── api/               # API routes
└── tests/                 # Test structure
\`\`\`

### Run Key Commands
\`\`\`bash
npm run dev      # Development server
npm test         # Run tests
npm run build    # Production build
npm run lint     # Check code style
\`\`\`

## Week 1: First Contribution

### Starter Tasks
Good first issues:
- Documentation updates
- Small bug fixes
- Test coverage improvements
- Minor UI tweaks

### PR Process
1. Create branch: `feature/description` or `fix/description`
2. Make changes
3. Write/update tests
4. Submit PR with description
5. Address review feedback
6. Merge after approval

### Code Standards
- [ ] Read style guide
- [ ] Understand commit conventions
- [ ] Review PR template
- [ ] Learn testing patterns

## Week 2-4: Deeper Dive

### Domain Knowledge
- [ ] Understand business domain
- [ ] Review product requirements
- [ ] Shadow senior developer
- [ ] Attend team meetings

### Technical Depth
- [ ] Understand deployment process
- [ ] Learn monitoring/alerting
- [ ] Review incident procedures
- [ ] Explore CI/CD pipeline

### Relationships
- [ ] 1:1 with manager
- [ ] Meet team members
- [ ] Identify mentor/buddy
- [ ] Join relevant channels

## Resources

### Documentation
- [ ] Internal wiki
- [ ] API documentation
- [ ] Runbooks
- [ ] Architecture decision records

### Contacts
| Role | Person | For |
|------|--------|-----|
| Manager | | Career, blockers |
| Tech Lead | | Architecture, code |
| Buddy | | Day-to-day questions |
| DevOps | | Infrastructure |

### Helpful Links
- Repository:
- Documentation:
- Slack: #team-channel
- CI/CD:

## 30-60-90 Day Goals

### 30 Days
- [ ] Complete all setup
- [ ] Ship first PR
- [ ] Understand core codebase
- [ ] Know team processes

### 60 Days
- [ ] Own a feature
- [ ] Participate in code reviews
- [ ] Understand full system
- [ ] Contribute to on-call

### 90 Days
- [ ] Lead a small project
- [ ] Mentor newer members
- [ ] Propose improvements
- [ ] Full on-call participation
```

---

## Patterns

### Pattern 1: Checklist-Driven Skills

```markdown
## Checklist

- [ ] Item 1 - Description
- [ ] Item 2 - Description
- [ ] Item 3 - Description

Work through each item systematically.
```

Best for: Reviews, verifications, processes with discrete steps.

### Pattern 2: Phase-Based Skills

```markdown
## Phase 1: Preparation
...

## Phase 2: Execution
...

## Phase 3: Verification
...
```

Best for: Multi-stage processes, workflows with dependencies.

### Pattern 3: Template-Providing Skills

```markdown
## Output Template

\`\`\`markdown
# [Title]

## Summary
[One paragraph]

## Details
- Point 1
- Point 2
\`\`\`
```

Best for: Consistent output format, documentation generation.

### Pattern 4: Decision Tree Skills

```markdown
## If TypeScript Project
- Check type errors
- Verify strict mode

## If JavaScript Project
- Check ESLint errors
- Verify JSDoc

## If Python Project
- Check type hints
- Run mypy
```

Best for: Multi-language, multi-framework workflows.

### Pattern 5: Interactive Skills

```markdown
## Questions to Answer

Before proceeding, determine:
1. What is the scope? (file/module/project)
2. What is the priority? (critical/normal/low)
3. Who is the audience? (developer/user/ops)

Based on answers, adjust the approach.
```

Best for: Context-dependent workflows.

---

## Anti-Patterns

### Anti-Pattern 1: No Actionable Items

```markdown
# BAD
This skill helps with code review.
Review the code carefully.
Make sure it's good.
```

```markdown
# GOOD
## Code Review Checklist

- [ ] Logic is correct
- [ ] Edge cases handled
- [ ] Tests included
- [ ] No security issues
```

**Why**: Vague instructions produce vague results.

### Anti-Pattern 2: Overly Broad Skills

```markdown
# BAD
---
name: everything
description: Does everything
---

Review code, write tests, fix bugs, deploy, monitor...
```

```markdown
# GOOD
---
name: code-review
description: Focused code review
---

[Specific code review workflow]
```

**Why**: Focused skills are more useful and maintainable.

### Anti-Pattern 3: Missing $ARGUMENTS

```markdown
# BAD
---
name: analyze
description: Analyze code
---

Analyze the code.
```

```markdown
# GOOD
---
name: analyze
description: Analyze specified code
---

Analyze: $ARGUMENTS

If no target specified, ask for one.
```

**Why**: Users expect to provide context with skill invocation.

### Anti-Pattern 4: Duplicating Built-in Functionality

```markdown
# BAD - Claude already does this
---
name: read-file
description: Read a file
---

Read the file the user specifies.
```

**Why**: Skills should add value, not wrap existing capabilities.

### Anti-Pattern 5: No Tool Restrictions for Sensitive Skills

```markdown
# BAD - Too permissive for security review
---
name: security-audit
description: Security audit
---
```

```markdown
# GOOD - Read-only for safety
---
name: security-audit
description: Security audit
allowed-tools:
  - Read
  - Glob
  - Grep
---
```

**Why**: Security reviews shouldn't modify code.

### Anti-Pattern 6: Assuming Context

```markdown
# BAD
Check the React components in src/components.
```

```markdown
# GOOD
## Detect Framework

First, identify the project framework:
- Check package.json for React/Vue/Angular
- Check for framework-specific files
- Adapt review based on framework found
```

**Why**: Skills should work across different projects.

---

## Troubleshooting

### Skill Not Showing in /help

**Symptoms**: Skill exists but doesn't appear in `/help` output

**Causes**:
1. Missing `description` field
2. Invalid YAML frontmatter
3. Wrong file location

**Solution**:
```yaml
---
name: my-skill
description: This field is required for /help  # <- Must have this
---
```

### $ARGUMENTS Not Replaced

**Symptoms**: Literal `$ARGUMENTS` appears in output

**Causes**:
1. Skill invoked without arguments
2. Typo in placeholder

**Solution**:
```markdown
## Target
$ARGUMENTS

## If No Target Provided
Ask the user what to analyze.
```

### Skill Has Wrong Tools

**Symptoms**: Skill can't access expected tools

**Causes**:
1. `allowed-tools` restricts tools
2. Tool name misspelled

**Solution**:
```yaml
allowed-tools:
  - Read     # Correct
  - read     # Wrong (case-sensitive)
```

### Context Not Available

**Symptoms**: Skill doesn't see conversation context

**Causes**:
1. `context: fork` isolates skill
2. Skill expects different context

**Solution**:
```yaml
# To see conversation:
context: inherit  # or omit (inherit is default)

# To have fresh context:
context: fork
```

---

## Cheat Sheet

### File Location
```
.claude/skills/<skill-name>/SKILL.md
```

### Minimal Frontmatter
```yaml
---
name: skill-name
description: What it does
---
```

### All Frontmatter Fields
```yaml
---
name: string           # Required
description: string    # Required
allowed-tools:         # Optional
  - Tool1
  - Tool2
context: inherit|fork  # Optional (default: inherit)
---
```

### Placeholder
```markdown
$ARGUMENTS - Replaced with user's input after /skill-name
```

### Invocation
```
/skill-name
/skill-name these are the arguments
```

---

## Related Documentation

- [Subagents](./01_SUBAGENTS.md) - For parallel specialized work
- [CLAUDE.md](./03_CLAUDE_MD.md) - For project context
- [Slash Commands](./05_SLASH_COMMANDS.md) - For quick actions
- [Official Docs](https://code.claude.com/docs/skills)
