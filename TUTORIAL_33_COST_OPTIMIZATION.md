# Tutorial 33: Cost Optimization

> **Track and optimize token usage and costs**

## Overview

Claude Code consumes tokens for each interaction. Learn to track, manage, and optimize costs effectively.

### Average Costs

- **Individual**: ~$6/developer/day (90% under $12)
- **Team**: ~$100-200/developer/month with Sonnet 4.5

---

## Part 1: Tracking Costs

### /cost Command

```bash
/cost
```

Shows:
```
Total cost:            $0.55
Total duration (API):  6m 19.7s
Total duration (wall): 6h 33m 10.2s
Total code changes:    0 lines added, 0 lines removed
```

> **Note:** `/cost` is for API users, not Max/Pro subscribers.

### Additional Tracking

- **Claude Console**: Historical usage (Admin/Billing role)
- **Workspace limits**: Set spending limits (Admin role)

---

## Part 2: Reducing Token Usage

### Compact Conversations

```bash
# Manual compaction
/compact

# With focus instructions
/compact Focus on code samples and API usage

# Auto-compact triggers at 95% context
# Override with environment variable:
export CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=50
```

### Configure in CLAUDE.md

```markdown
# Summary instructions

When you are using compact, please focus on test output and code changes
```

### Write Specific Queries

| Avoid | Better |
|-------|--------|
| "Look at the code" | "Review auth.ts lines 50-75" |
| "Find bugs" | "Check for null pointer in UserService" |

### Break Down Tasks

Split large tasks into focused interactions rather than one massive prompt.

### Clear History

```bash
/clear
```

Reset context between unrelated tasks.

---

## Part 3: Team Cost Management

### Rate Limit Recommendations

| Team Size | TPM/User | RPM/User |
|-----------|----------|----------|
| 1-5 | 200k-300k | 5-7 |
| 5-20 | 100k-150k | 2.5-3.5 |
| 20-50 | 50k-75k | 1.25-1.75 |
| 50-100 | 25k-35k | 0.62-0.87 |
| 100-500 | 15k-20k | 0.37-0.47 |
| 500+ | 10k-15k | 0.25-0.35 |

### Workspace Spending Limits

Set limits in Claude Console for the "Claude Code" workspace.

### Third-Party Tracking

For Bedrock/Vertex deployments, consider tools like LiteLLM for cost tracking.

---

## Part 4: Cost Factors

Costs vary based on:

| Factor | Impact |
|--------|--------|
| Codebase size | More scanning = more tokens |
| Query complexity | Vague requests cost more |
| Files modified | More files = more context |
| Conversation length | Longer history = more tokens |
| Compaction frequency | Regular compaction reduces costs |

### Background Token Usage

Small background consumption (~$0.04/session):
- Conversation summarization
- Status checks

---

## Part 5: Optimization Strategies

### Strategy 1: Model Selection

| Task | Model | Cost |
|------|-------|------|
| Simple edits | Haiku | $ |
| Daily coding | Sonnet | $$ |
| Complex work | Opus | $$$ |

### Strategy 2: Session Management

```bash
# Start fresh for new tasks
/clear

# Compact regularly
/compact

# End sessions cleanly
/exit
```

### Strategy 3: Focused Prompts

```bash
# Bad: vague, causes scanning
"What's wrong with the code?"

# Good: specific, minimal scanning
"Fix the TypeError on line 45 of UserService.ts"
```

---

## Quick Reference

### Commands

| Command | Description |
|---------|-------------|
| `/cost` | View session costs |
| `/compact` | Compress context |
| `/clear` | Reset context |

### Environment Variables

| Variable | Description |
|----------|-------------|
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | Trigger compaction earlier |

---

## Sources

- [Manage Costs](https://code.claude.com/docs/en/costs.md)
