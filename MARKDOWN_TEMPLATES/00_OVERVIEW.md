# Claude Code Markdown Templates - Overview

> **Quick Navigation Hub** for all Claude Code markdown file types, templates, and examples.

## What This Documentation Covers

Claude Code uses markdown files for configuration in five key areas. This documentation provides templates, examples, patterns, and anti-patterns for each.

| Document | Feature | Location | Purpose |
|----------|---------|----------|---------|
| [01_SUBAGENTS.md](./01_SUBAGENTS.md) | Subagents | `.claude/agents/*.md` | Specialized AI agents for specific tasks |
| [02_SKILLS.md](./02_SKILLS.md) | Skills | `.claude/skills/*/SKILL.md` | Reusable workflows and checklists |
| [03_CLAUDE_MD.md](./03_CLAUDE_MD.md) | CLAUDE.md | Project root | Project context and conventions |
| [04_MODULAR_RULES.md](./04_MODULAR_RULES.md) | Modular Rules | `.claude/rules/*.md` | Path-specific coding standards |
| [05_SLASH_COMMANDS.md](./05_SLASH_COMMANDS.md) | Slash Commands | `.claude/commands/*.md` | Custom `/command` shortcuts |

---

## Quick Reference Matrix

### Feature Comparison

| Feature | Frontmatter | Invocation | Primary Use Case |
|---------|-------------|------------|------------------|
| **Subagents** | Required | `Task` tool / auto | Parallel specialized work |
| **Skills** | Required | `/skill-name` | Repeatable workflows |
| **CLAUDE.md** | None | Automatic | Project context |
| **Modular Rules** | Optional | Automatic (path-based) | Coding standards |
| **Slash Commands** | Optional | `/command` | Quick actions |

### Directory Structure

```
your-project/
├── CLAUDE.md                          # Project context (auto-loaded)
├── .claude/
│   ├── settings.json                  # Project settings
│   ├── settings.local.json            # Local overrides (gitignored)
│   ├── agents/                        # Subagent definitions
│   │   ├── security-auditor.md
│   │   ├── code-reviewer.md
│   │   └── test-runner.md
│   ├── skills/                        # Skill definitions
│   │   ├── code-review/
│   │   │   └── SKILL.md
│   │   ├── tdd-workflow/
│   │   │   └── SKILL.md
│   │   └── debugging/
│   │       └── SKILL.md
│   ├── rules/                         # Modular rules (path-specific)
│   │   ├── typescript.md
│   │   ├── react.md
│   │   └── security.md
│   └── commands/                      # Custom slash commands
│       ├── commit.md
│       ├── review-pr.md
│       └── deploy.md
```

---

## When to Use Each Feature

### Use Subagents When...
- Task can run **in parallel** with other work
- Task requires **specialized tools** (e.g., security audit = read-only)
- Task benefits from **focused context** (isolated from main conversation)
- Output needs **structured format** for further processing

### Use Skills When...
- Workflow is **repeatable** across projects
- Process has **multiple steps** or a checklist
- You want **user-invocable** shortcuts (`/skill-name`)
- Guidance needs **conditional logic** based on context

### Use CLAUDE.md When...
- Information is **project-specific** context
- Content should be **auto-loaded** for every conversation
- Rules apply **project-wide** (not path-specific)
- Documentation is **essential for understanding** the codebase

### Use Modular Rules When...
- Rules are **path-specific** (e.g., React rules for `src/components/`)
- You want **separation of concerns** (TypeScript vs Python rules)
- Rules should **conditionally load** based on active files
- Team has **domain-specific standards** for different areas

### Use Slash Commands When...
- Action is **simple and quick** (1-3 steps)
- You want a **memorable shortcut** (`/deploy`, `/commit`)
- Action takes **arguments** (`/create-component Button`)
- No complex workflow logic needed

---

## Frontmatter Field Reference

### Subagent Fields

```yaml
---
name: agent-name               # Required - identifier
description: What it does      # Required - shown in agent list
tools:                         # Required - allowed tools
  - Read
  - Glob
  - Grep
model: sonnet                  # Optional - sonnet|opus|haiku
permissionMode: default        # Optional - default|allowAll
bypassPermissions: false       # Optional - skip permission prompts
systemPrompt: |                # Optional - custom instructions
  You are a specialized agent...
---
```

### Skill Fields

```yaml
---
name: skill-name               # Required - identifier
description: What it does      # Required - shown in /help
allowed-tools:                 # Optional - restrict tools
  - Read
  - Grep
context: fork                  # Optional - fork|inherit
---
```

### Modular Rule Fields

```yaml
---
paths:                         # Optional - glob patterns
  - "src/components/**/*.tsx"
  - "**/*.test.ts"
---
```

### Slash Command Fields

```yaml
---
description: What it does      # Optional - shown in /help
allowed-tools:                 # Optional - restrict tools
  - Bash
  - Read
argument-hint: <component>     # Optional - shown in /help
---
```

---

## Quick Templates

### Minimal Subagent

```markdown
---
name: helper
description: General helper agent
tools:
  - Read
  - Glob
  - Grep
---

Help with the assigned task.
```

### Minimal Skill

```markdown
---
name: checklist
description: Run a checklist
---

## Checklist

- [ ] Item 1
- [ ] Item 2
- [ ] Item 3
```

### Minimal CLAUDE.md

```markdown
# Project Name

## Tech Stack
- TypeScript, React, Node.js

## Commands
- `npm run dev` - Start development
- `npm test` - Run tests
```

### Minimal Modular Rule

```markdown
---
paths:
  - "**/*.ts"
---

Use strict TypeScript. No `any` types.
```

### Minimal Slash Command

```markdown
---
description: Say hello
---

Greet the user warmly.
```

---

## Common Patterns Across Features

### 1. Structured Output
All features benefit from specifying expected output format:

```markdown
## Output Format

Respond with:
1. **Summary**: One-line overview
2. **Details**: Bullet points
3. **Next Steps**: Recommended actions
```

### 2. Conditional Logic
Use conditional sections for context-aware behavior:

```markdown
## If TypeScript Project
- Check for type errors
- Verify strict mode

## If JavaScript Project
- Check for ESLint errors
- Verify JSDoc comments
```

### 3. @import for Modularity
CLAUDE.md supports imports for organization:

```markdown
@import ./docs/architecture.md
@import ./docs/conventions.md
```

### 4. $ARGUMENTS Placeholder
Skills and commands support argument substitution:

```markdown
Analyze the component: $ARGUMENTS
```

---

## Anti-Patterns to Avoid

| Anti-Pattern | Why It's Bad | Better Approach |
|--------------|--------------|-----------------|
| Over-tooling subagents | Security risk, confusion | Minimal required tools only |
| Vague instructions | Inconsistent output | Specific, structured prompts |
| Duplicate functionality | Maintenance burden | Extend existing features |
| Hardcoded paths | Not portable | Use $ARGUMENTS or patterns |
| Missing descriptions | Won't show in /help | Always include description |
| Circular @imports | Infinite loops | Linear dependency chain |
| Overly broad rules | Performance impact | Specific path patterns |

---

## Troubleshooting Quick Reference

| Issue | Likely Cause | Solution |
|-------|--------------|----------|
| Subagent not found | Wrong path or name | Check `.claude/agents/*.md` |
| Skill not in /help | Missing description | Add `description:` field |
| Rules not loading | Path pattern mismatch | Test glob with actual paths |
| Command not working | Syntax error in frontmatter | Validate YAML format |
| CLAUDE.md ignored | Wrong location | Must be in project root |
| @import failing | Path doesn't exist | Use relative paths from root |

---

## Learning Path

### Beginner
1. Start with [CLAUDE.md](./03_CLAUDE_MD.md) - essential for every project
2. Add a [Slash Command](./05_SLASH_COMMANDS.md) - quick wins
3. Create your first [Skill](./02_SKILLS.md) - reusable workflows

### Intermediate
4. Define [Modular Rules](./04_MODULAR_RULES.md) - path-specific standards
5. Build a [Subagent](./01_SUBAGENTS.md) - parallel specialized work

### Advanced
6. Combine features - Skills that spawn Subagents
7. Create plugin ecosystems - Shareable skill/agent collections
8. Enterprise patterns - Team-wide standardization

---

## Official Documentation

- **Documentation Hub**: https://code.claude.com/docs
- **Subagents**: https://code.claude.com/docs/sub-agents
- **Skills**: https://code.claude.com/docs/skills
- **Hooks**: https://code.claude.com/docs/hooks
- **Memory & CLAUDE.md**: https://code.claude.com/docs/memory

---

## Next Steps

Choose the feature most relevant to your needs:

1. **[Subagents](./01_SUBAGENTS.md)** - For parallel, specialized work
2. **[Skills](./02_SKILLS.md)** - For repeatable workflows
3. **[CLAUDE.md](./03_CLAUDE_MD.md)** - For project context
4. **[Modular Rules](./04_MODULAR_RULES.md)** - For coding standards
5. **[Slash Commands](./05_SLASH_COMMANDS.md)** - For quick actions
