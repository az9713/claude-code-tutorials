# Tutorial 44: Testing Workflows

> **Writing and running tests with Claude Code**

## Overview

Effective workflows for test-driven development with Claude Code.

---

## Part 1: Writing Tests

### Generate Tests

```bash
Write unit tests for:
@src/services/UserService.ts

Cover:
- Happy path
- Edge cases
- Error handling
```

### Test Frameworks

```bash
Write tests using Jest for:
@src/utils/validator.ts

# Or specify framework
Write pytest tests for:
@src/api/routes.py
```

---

## Part 2: Test-Driven Development

### TDD Workflow

```bash
# Step 1: Write tests first
Write failing tests for a new EmailValidator class

# Step 2: Implement
Now implement EmailValidator to pass the tests

# Step 3: Refactor
Refactor the implementation for clarity
```

### Red-Green-Refactor

Claude follows TDD principles when asked.

---

## Part 3: Running Tests

### In Claude Code

```bash
# Run tests
!npm test

# Run specific test
!npm test -- UserService

# With coverage
!npm test -- --coverage
```

### Fix Failing Tests

```bash
These tests are failing:
[paste test output]

@src/services/UserService.ts
Fix the issues.
```

---

## Part 4: Coverage

### Check Coverage

```bash
!npm test -- --coverage
```

### Improve Coverage

```bash
Current coverage is 60%. Add tests to cover
uncovered branches in:
@src/utils/parser.ts
```

---

## Part 5: Best Practices

### Specific Test Requests

```bash
# Good
Write tests for UserService.createUser() covering:
- Valid email
- Invalid email
- Duplicate user
- Database error

# Less effective
"Write tests"
```

### Include Context

```bash
Here's the function and existing tests:
@src/services/auth.ts
@tests/auth.test.ts

Add tests for the new loginWithOTP method.
```

---

## Quick Reference

### Test Commands

| Command | Purpose |
|---------|---------|
| `!npm test` | Run all tests |
| `!npm test -- <pattern>` | Run matching |
| `!npm test -- --coverage` | With coverage |

### TDD Steps

1. Write failing test
2. Implement minimum code
3. Pass test
4. Refactor
5. Repeat

---

## Sources

- [Common Workflows](https://code.claude.com/docs/en/common-workflows.md)
