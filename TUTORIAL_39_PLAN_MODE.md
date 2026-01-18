# Tutorial 39: Plan Mode

> **Collaborate safely on approaches before making changes**

## Overview

Plan mode lets you discuss and plan with Claude without making code changes.

---

## Part 1: What is Plan Mode?

### Features

- Read-only analysis
- No file modifications
- Discussion and planning
- Safe for exploration

### Use Cases

- Architecture discussions
- Understanding code
- Planning refactors
- Security review

---

## Part 2: Enabling Plan Mode

### At Startup

```bash
claude --permission-mode plan
```

### Per Command

```markdown
---
permission-mode: plan
---

Analyze this code for improvements.
```

### As Default

```json
{
  "permissionMode": "plan"
}
```

---

## Part 3: Working in Plan Mode

### Allowed Actions

- Reading files
- Searching code
- Analyzing patterns
- Discussing approaches

### Blocked Actions

- Writing files
- Editing code
- Running destructive commands

### Exit Plan Mode

Switch to execution when ready:

```bash
/config
# Change permission mode

# Or restart without --permission-mode plan
```

---

## Part 4: Plan-Execute Workflow

### Step 1: Plan

```bash
claude --permission-mode plan

> Let's discuss how to refactor the authentication system
```

### Step 2: Collaborate

- Discuss approaches
- Review options
- Agree on plan

### Step 3: Execute

```bash
# Exit plan mode
claude

> Implement the authentication refactor we discussed
```

### Using opusplan

Automates this with model switching:

```bash
/model opusplan
# Uses Opus for planning, Sonnet for execution
```

---

## Part 5: Best Practices

### Start with Plan Mode for Complex Tasks

```bash
claude --permission-mode plan

> I want to add caching to the API layer. 
> What are the options and trade-offs?
```

### Document the Plan

```bash
> Write this plan to docs/caching-plan.md
```

Then execution can reference it.

### Review Before Execution

Use plan mode to:
1. Understand current state
2. Identify risks
3. Plan rollback strategy
4. Then switch to execution

---

## Quick Reference

### Commands

| Command | Description |
|---------|-------------|
| `claude --permission-mode plan` | Start in plan mode |
| `/config` | Change permission mode |

### Permission Modes

| Mode | Can Read | Can Write |
|------|----------|-----------|
| `plan` | ✅ | ❌ |
| `default` | ✅ | Ask |
| `acceptEdits` | ✅ | ✅ |

---

## Sources

- [Common Workflows](https://code.claude.com/docs/en/common-workflows.md)
- [Settings](https://code.claude.com/docs/en/settings.md)
