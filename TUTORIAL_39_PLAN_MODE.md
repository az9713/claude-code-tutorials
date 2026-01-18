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

## New in Claude Code v2.1.x

### /plan Command Shortcut (v2.1.0)

Enter plan mode directly with a slash command:

```
/plan
```

This switches to plan mode without restarting the session.

### Shift+Tab Quick Toggle (v2.1.2)

Toggle between plan and execute modes instantly:

```
Shift+Tab  →  Toggle plan/execute mode
```

This provides rapid switching during iterative development.

### plansDirectory Setting (v2.1.9)

Configure where plan files are stored:

```json
{
  "plansDirectory": "~/.claude/plans"
}
```

**Default locations:**
- User plans: `~/.claude/plans/`
- Project plans: `.claude/plans/`

Plan files are named with descriptive slugs (e.g., `staged-wibbling-lark.md`).

### Auto-Accept Edits in Plan Mode (v2.0.62)

When reviewing a plan, you can auto-accept edits:

```json
{
  "permissionMode": "acceptEdits"
}
```

Or use the CLI flag:

```bash
claude --permission-mode acceptEdits
```

This allows Claude to implement the plan without prompting for each file change.

### Permission Prompt Reduction (v2.1.0)

Plan mode now has streamlined permission handling:
- Tool permission prompts removed during read-only operations
- Faster analysis without interruptions
- Cleaner UX for exploration workflows

### Plan Mode with Tools

Specify which tools work in plan mode:

```json
{
  "permissions": {
    "tools": {
      "Read": { "allow": ["*"] },
      "Grep": { "allow": ["*"] },
      "Glob": { "allow": ["*"] },
      "Write": { "deny": ["*"] },
      "Edit": { "deny": ["*"] },
      "Bash": { "deny": ["*"] }
    }
  }
}
```

### Plan Mode Workflow Example

```bash
# 1. Start in plan mode
claude --permission-mode plan

# 2. Analyze and plan
> Analyze the authentication system and create a refactoring plan

# 3. Toggle to execute mode (Shift+Tab)

# 4. Or use /plan to return to planning
/plan

# 5. Execute when ready
> Implement the plan we created
```

---

## Quick Reference

### Commands

| Command | Description |
|---------|-------------|
| `claude --permission-mode plan` | Start in plan mode |
| `/plan` | Enter plan mode (v2.1.0) |
| `/config` | Change permission mode |
| `Shift+Tab` | Toggle plan/execute (v2.1.2) |

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
- [Claude Code Changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

---

*This tutorial is part of the Claude Code documentation series.*
*Version 2.1.x | Last Updated: January 2026*
