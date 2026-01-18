# Tutorial 38: Extended Thinking

> **Use extended thinking for complex reasoning tasks**

## Overview

Extended thinking gives Claude more time to reason through complex problems.

---

## Part 1: What is Extended Thinking?

### How It Works

- Claude's internal reasoning before responding
- More thorough analysis
- Better for complex tasks

### When to Use

- Complex architecture decisions
- Difficult debugging
- Multi-step planning
- Code review

---

## Part 2: Enabling Thinking

### In Slash Commands

```markdown
---
description: Deep analysis command
thinking: true
---

Analyze this code thoroughly for bugs and improvements.
```

### Via Settings

```json
{
  "thinking": true
}
```

### At Runtime

```bash
claude --thinking
```

---

## Part 3: Thinking Budget

### Token Budget

Extended thinking uses additional tokens for reasoning.

### Configure Budget

```json
{
  "thinkingBudget": 10000
}
```

### Budget Tips

- Higher budget = deeper analysis
- Increases cost and latency
- Balance based on task complexity

---

## Part 4: Best Practices

### Use for Complex Tasks

```bash
# Good candidates for thinking
- "Design the database schema for this feature"
- "Debug this intermittent race condition"
- "Review this code for security vulnerabilities"
```

### Skip for Simple Tasks

```bash
# Don't need thinking
- "Fix this typo"
- "Add a comment here"
- "Rename this variable"
```

### Combine with Plan Mode

```bash
claude --permission-mode plan --thinking

# Deep analysis without making changes
```

---

## Part 5: Example Use Cases

### Architecture Review

```markdown
---
thinking: true
---

Review the current microservices architecture and propose
improvements for scalability and maintainability.
```

### Complex Debugging

```markdown
---
thinking: true
---

This test fails intermittently. Analyze for race conditions
or timing issues.
```

### Security Audit

```markdown
---
thinking: true
---

Perform a thorough security review of the authentication
system, checking for OWASP Top 10 vulnerabilities.
```

---

## Quick Reference

### Enable Thinking

| Method | Command |
|--------|---------|
| Command | `--thinking` |
| Settings | `"thinking": true` |
| Slash command | `thinking: true` frontmatter |

### When to Use

| Use Thinking | Skip Thinking |
|--------------|---------------|
| Architecture | Simple edits |
| Debugging | Typo fixes |
| Security review | Formatting |
| Complex planning | Routine tasks |

---

## Sources

- [Common Workflows](https://code.claude.com/docs/en/common-workflows.md)
- [Thinking Mode](https://docs.claude.com/en/docs/build-with-claude/extended-thinking)
