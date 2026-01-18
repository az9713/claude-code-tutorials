# Tutorial 46: Code Review

> **Perform thorough code reviews with Claude Code**

## Overview

Use Claude Code for comprehensive code review.

---

## Part 1: Review Commands

### Basic Review

```bash
Review @src/services/AuthService.ts for issues.
```

### Focused Review

```bash
Review @src/api/routes.ts for:
- Security vulnerabilities
- Error handling
- Performance issues
```

---

## Part 2: Security Review

### OWASP Check

```bash
Review @src/auth/ for OWASP Top 10 vulnerabilities.
```

### Specific Vulnerabilities

```bash
Check for:
- SQL injection
- XSS
- CSRF
- Authentication bypass
```

---

## Part 3: Performance Review

```bash
Analyze @src/services/DataProcessor.ts for:
- Algorithm efficiency
- Memory usage
- N+1 queries
- Caching opportunities
```

---

## Part 4: Code Quality

```bash
Review @src/components/ for:
- Code duplication
- Naming conventions
- Complexity
- Test coverage
```

---

## Part 5: Review Workflows

### PR Review Mode

Use plan mode for read-only review:

```bash
claude --permission-mode plan

Review the changes in this PR branch.
```

### Extended Thinking

For thorough analysis:

```bash
claude --thinking

Perform a comprehensive security review.
```

---

## Quick Reference

### Review Types

| Type | Focus Areas |
|------|-------------|
| Security | OWASP, auth, injection |
| Performance | Complexity, caching |
| Quality | DRY, naming, tests |
| Style | Conventions, formatting |

### Review Command Template

```bash
Review @path/to/file.ts for:
- [Focus area 1]
- [Focus area 2]
- [Focus area 3]
```

---

## Sources

- [Common Workflows](https://code.claude.com/docs/en/common-workflows.md)
