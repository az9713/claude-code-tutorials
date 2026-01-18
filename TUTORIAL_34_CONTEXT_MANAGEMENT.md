# Tutorial 34: Context Window Management

> **Manage context effectively for long coding sessions**

## Overview

Claude Code has a limited context window. Learn to manage it effectively for productive long sessions.

---

## Part 1: Understanding Context

### What Consumes Context

- Conversation history
- File contents read
- Tool outputs
- System prompts
- Your prompts

### Context Limits

| Model | Context Window |
|-------|----------------|
| Standard | ~200k tokens |
| Extended [1m] | ~1M tokens |

---

## Part 2: Auto-Compaction

### How It Works

Claude auto-compacts at ~95% context capacity:
1. Summarizes conversation
2. Preserves key information
3. Reduces token count

### Customize Trigger

```bash
# Compact earlier (at 50%)
export CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=50
```

### Disable Auto-Compact

```bash
/config
# Navigate to "Auto-compact enabled"
# Toggle off
```

---

## Part 3: Manual Compaction

### Basic Compaction

```bash
/compact
```

### Focused Compaction

```bash
/compact Focus on test results and code changes

/compact Preserve API documentation discussion

/compact Keep error messages and fixes
```

### In CLAUDE.md

```markdown
# Summary instructions

When compacting, focus on:
- Code changes made
- Test results
- Error resolutions
```

---

## Part 4: Best Practices

### Clear Between Tasks

```bash
/clear
```

Start fresh for unrelated work.

### Be Specific

Avoid scanning entire codebases:

```bash
# Bad
"Look at my project"

# Good
"Review src/auth/login.ts:50-100"
```

### Use @-Mentions

```bash
@src/utils/helper.ts What does this do?
```

Targets specific files.

### Break Down Large Tasks

Instead of one massive request, use incremental steps:

```bash
Step 1: "Analyze the current architecture"
Step 2: "Propose changes"
Step 3: "Implement component A"
Step 4: "Implement component B"
```

---

## Part 5: Extended Context

### Enable 1M Context

```bash
/model sonnet[1m]
```

### When to Use

- Very large codebases
- Long analysis sessions
- Multi-file refactoring

### Considerations

- Different pricing
- May be slower
- Not always necessary

---

## Quick Reference

| Command | Description |
|---------|-------------|
| `/compact` | Compress context |
| `/compact <focus>` | Guided compression |
| `/clear` | Reset context |
| `/model sonnet[1m]` | Extended context |

---

## Sources

- [Common Workflows](https://code.claude.com/docs/en/common-workflows.md)
- [Model Configuration](https://code.claude.com/docs/en/model-config.md)
