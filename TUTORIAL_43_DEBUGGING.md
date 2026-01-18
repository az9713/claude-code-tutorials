# Tutorial 43: Debugging with Claude Code

> **Effective debugging strategies with Claude Code**

## Overview

Strategies for effective debugging with Claude Code.

---

## Part 1: Basic Debugging

### Describe the Problem

```bash
This function returns undefined sometimes:
@src/utils/parser.ts

Input: [1, 2, 3]
Expected: 6
Actual: undefined
```

### Share Error Messages

```bash
I'm getting this error:
TypeError: Cannot read property 'map' of undefined
at processData (utils.js:45)

Here's the code:
@src/utils.js
```

---

## Part 2: Diagnostic Sharing

### IDE Integration

VS Code and JetBrains automatically share:
- Lint errors
- Type errors
- Syntax issues

### Manual Sharing

```bash
npm test
# Share failing test output with Claude

npm run lint
# Share lint errors
```

---

## Part 3: Debugging Strategies

### Binary Search

```bash
The app crashes on startup. Help me narrow down:
1. Comment out half the imports
2. Test if crash persists
3. Repeat to find culprit
```

### Add Logging

```bash
Add debug logging to trace the data flow in:
@src/services/orderProcessor.ts
```

### Test Isolation

```bash
Write a minimal test case that reproduces:
- Input: [array of nulls]
- Expected: filtered array
- Actual: crash
```

---

## Part 4: Common Issues

### Null/Undefined

```bash
Find all places in @src/api/ where we access 
properties without null checks.
```

### Async Issues

```bash
This code has a race condition:
@src/services/cache.ts
How do I fix it?
```

### Type Errors

```bash
TypeScript says this is wrong but it worked before:
@src/types/user.ts
What changed?
```

---

## Part 5: Using Checkpoints

### Safe Debugging

```bash
# Make a change
# If it breaks things, rewind
Esc + Esc
```

### Experiment Freely

Try different fixes knowing you can revert.

---

## Quick Reference

### Debugging Workflow

1. Describe problem clearly
2. Share relevant code
3. Include error messages
4. Try suggested fix
5. Rewind if needed

### Useful Commands

| Command | Purpose |
|---------|---------|
| `/rewind` | Undo changes |
| `/compact` | Clear old context |
| `/clear` | Fresh start |

---

## Sources

- [Common Workflows](https://code.claude.com/docs/en/common-workflows.md)
- [Checkpointing](https://code.claude.com/docs/en/checkpointing.md)
