# Tutorial 48: Refactoring

> **Strategic refactoring with Claude Code**

## Overview

Systematic approaches to refactoring with Claude Code.

---

## Part 1: Planning Refactors

### Use Plan Mode

```bash
claude --permission-mode plan

Analyze @src/services/LegacyService.ts and propose
a refactoring plan.
```

---

## Part 2: Common Refactors

### Extract Function

```bash
Extract the validation logic from lines 45-80 into
a separate function.
@src/handlers/userHandler.ts
```

### Rename Symbol

```bash
Rename UserData to UserProfile across the codebase.
```

### Move Code

```bash
Move the helper functions from @src/utils.ts into
separate files by category.
```

---

## Part 3: Architecture Changes

### Dependency Injection

```bash
Refactor @src/services/OrderService.ts to use
dependency injection for the database connection.
```

### Design Patterns

```bash
Apply the repository pattern to @src/data/users.ts
```

---

## Part 4: Safe Refactoring

### With Checkpoints

```bash
# Refactor with safety net
1. Make change
2. Test
3. If broken, Esc+Esc to rewind
```

### Incremental Changes

```bash
Refactor in small steps:
1. Extract interface
2. Create new implementation
3. Swap implementations
4. Remove old code
```

---

## Part 5: Testing Refactors

### Ensure Tests Pass

```bash
Refactor @src/utils/parser.ts but ensure all tests
in @tests/parser.test.ts still pass.
```

---

## Quick Reference

### Refactoring Types

| Type | Example |
|------|---------|
| Extract | "Extract function" |
| Rename | "Rename across codebase" |
| Move | "Move to separate file" |
| Pattern | "Apply factory pattern" |

### Safety Checklist

- [ ] Start from green tests
- [ ] Small increments
- [ ] Test after each change
- [ ] Use checkpoints
- [ ] Review carefully

---

## Sources

- [Common Workflows](https://code.claude.com/docs/en/common-workflows.md)
- [Checkpointing](https://code.claude.com/docs/en/checkpointing.md)
