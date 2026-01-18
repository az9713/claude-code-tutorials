# Tutorial 31: Output Styles

> **Adapt Claude Code for uses beyond software engineering**

## Overview

Output styles allow you to customize how Claude Code responds, adapting it for different use cases while keeping its core capabilities like running scripts, reading/writing files, and tracking TODOs.

### What You'll Learn

- Built-in output styles and their purposes
- How output styles modify Claude's behavior
- Creating custom output styles
- Comparing output styles to other features

### Prerequisites

- Claude Code installed
- Basic understanding of Markdown

---

## Part 1: Built-in Output Styles

### Default Style

The default output style is optimized for efficient software engineering:
- Concise responses
- Code-focused
- Task-oriented

### Explanatory Style

Provides educational "Insights" while helping with tasks:

```
/output-style explanatory
```

**Features:**
- Explains implementation choices
- Highlights codebase patterns
- Teaches as it works

**Best for:**
- Learning new codebases
- Understanding code decisions
- Onboarding to projects

### Learning Style

Collaborative, learn-by-doing mode:

```
/output-style learning
```

**Features:**
- Shares insights while coding
- Adds `TODO(human)` markers for you to implement
- Asks you to contribute code

**Best for:**
- Learning to code
- Understanding implementations deeply
- Skill building

---

## Part 2: How Output Styles Work

### System Prompt Modification

Output styles directly modify Claude Code's system prompt:

| Aspect | Default | Other Styles |
|--------|---------|--------------|
| Efficient output instructions | Yes | No |
| Coding instructions | Yes | Only if `keep-coding-instructions` |
| Custom instructions | No | Added to end |

### Style Reminders

All output styles trigger reminders during conversation to ensure Claude adheres to the style instructions.

---

## Part 3: Changing Your Output Style

### Using the Menu

```bash
/output-style
```

This opens a menu to select your style.

### Direct Selection

```bash
/output-style explanatory
/output-style learning
/output-style default
```

### Via /config

```bash
/config
# Navigate to output style option
```

### Settings File

Edit `.claude/settings.local.json`:

```json
{
  "outputStyle": "explanatory"
}
```

---

## Part 4: Creating Custom Output Styles

### Style File Format

Custom styles are Markdown files with frontmatter:

```markdown
---
name: My Custom Style
description: A brief description for the UI
---

# Custom Style Instructions

You are an interactive CLI tool that helps users with [purpose].

## Specific Behaviors

[Define how Claude should behave...]
```

### File Locations

| Scope | Path |
|-------|------|
| User-level | `~/.claude/output-styles/` |
| Project-level | `.claude/output-styles/` |

### Frontmatter Options

| Field | Purpose | Default |
|-------|---------|---------|
| `name` | Style name in UI | File name |
| `description` | Description in `/output-style` menu | None |
| `keep-coding-instructions` | Keep default coding instructions | false |

---

## Part 5: Example Custom Styles

### Technical Writer Style

**~/.claude/output-styles/tech-writer.md:**

```markdown
---
name: Technical Writer
description: Focus on documentation and explanation
---

# Technical Writer Mode

You are a technical documentation specialist. Your primary focus is
creating clear, comprehensive documentation.

## Response Style

- Write in clear, accessible language
- Use examples liberally
- Structure content with headers
- Include code samples with explanations

## Priorities

1. Clarity over brevity
2. Examples over abstract explanations
3. Practical guidance over theory
```

### Code Reviewer Style

**~/.claude/output-styles/reviewer.md:**

```markdown
---
name: Code Reviewer
description: Thorough code review and feedback
keep-coding-instructions: true
---

# Code Review Mode

You are a senior code reviewer. Provide thorough, constructive feedback.

## Review Aspects

- **Correctness**: Does the code work as intended?
- **Security**: Are there vulnerabilities?
- **Performance**: Are there efficiency issues?
- **Maintainability**: Is the code easy to understand?
- **Style**: Does it follow conventions?

## Feedback Style

Be specific and actionable. Explain why something is an issue
and how to fix it.
```

### Mentor Style

**~/.claude/output-styles/mentor.md:**

```markdown
---
name: Mentor
description: Teaching-focused with guided discovery
---

# Mentor Mode

You are a patient programming mentor who guides through discovery.

## Teaching Approach

- Ask questions before giving answers
- Guide rather than tell
- Encourage experimentation
- Celebrate learning moments

## When User Gets Stuck

1. Ask what they've tried
2. Point them toward resources
3. Give hints before solutions
4. Explain the "why" behind answers
```

### Research Mode

**~/.claude/output-styles/research.md:**

```markdown
---
name: Research
description: Deep analysis and exploration
---

# Research Mode

You are a research assistant focused on thorough analysis.

## Approach

- Explore multiple perspectives
- Cite sources when possible
- Consider edge cases
- Question assumptions

## Output Style

- Detailed explanations
- Structured findings
- Clear conclusions
- Recommendations for further exploration
```

---

## Part 6: Style Comparisons

### Output Styles vs CLAUDE.md

| Feature | Output Styles | CLAUDE.md |
|---------|---------------|-----------|
| Modifies system prompt | Yes (replaces parts) | No (adds after) |
| Coding instructions | Can be removed | Always present |
| Persistence | Per-session or setting | Always loaded |

### Output Styles vs --append-system-prompt

| Feature | Output Styles | --append-system-prompt |
|---------|---------------|------------------------|
| Replaces default instructions | Yes (coding parts) | No |
| Position in prompt | Replaces sections | Appends to end |
| Configuration | Files or UI | Command line |

### Output Styles vs Agents

| Feature | Output Styles | Agents |
|---------|---------------|--------|
| Scope | Main agent loop | Specific tasks |
| Configuration | System prompt only | Model, tools, context |
| Invocation | Session-wide | Task-specific |

### Output Styles vs Slash Commands

| Feature | Output Styles | Slash Commands |
|---------|---------------|----------------|
| Think of as... | "Stored system prompts" | "Stored prompts" |
| Duration | Entire session | Single use |
| Scope | Claude's behavior | Single task |

---

## Part 7: Advanced Patterns

### Pattern 1: Project-Specific Styles

Create styles for specific projects:

**.claude/output-styles/api-development.md:**

```markdown
---
name: API Development
description: Focus on REST API best practices
keep-coding-instructions: true
---

# API Development Mode

Focus on RESTful API design and implementation.

## Priorities

- Follow REST conventions
- Consider HTTP semantics
- Think about versioning
- Plan for scalability

## Code Style

- Use proper HTTP status codes
- Include input validation
- Add comprehensive error handling
```

### Pattern 2: Team Standards

Share styles across team:

```bash
# In your repo
.claude/
└── output-styles/
    ├── team-review.md
    ├── team-docs.md
    └── team-debug.md
```

### Pattern 3: Context Switching

Switch styles based on task:

```bash
# Starting code review
/output-style reviewer

# Review code...

# Switching to documentation
/output-style tech-writer

# Write docs...
```

---

## Part 8: Use Cases Beyond Engineering

### Data Analysis

```markdown
---
name: Data Analyst
description: Focus on data exploration and insights
---

# Data Analyst Mode

You help analyze data and derive insights.

## Approach

- Ask about data sources and structure
- Suggest exploratory analyses
- Visualize when helpful
- Explain statistical concepts
```

### Creative Writing

```markdown
---
name: Creative Writer
description: Help with creative writing projects
---

# Creative Writer Mode

You assist with creative writing and storytelling.

## Style

- Offer suggestions, not rewrites
- Respect the author's voice
- Focus on structure and flow
- Provide constructive feedback
```

### System Administration

```markdown
---
name: SysAdmin
description: System administration and DevOps focus
keep-coding-instructions: true
---

# SysAdmin Mode

You help with system administration tasks.

## Priorities

- Security first
- Explain risks clearly
- Suggest safer alternatives
- Document changes
```

---

## Quick Reference

### Commands

| Command | Description |
|---------|-------------|
| `/output-style` | Open style menu |
| `/output-style <name>` | Switch to specific style |
| `/output-style default` | Reset to default |

### Built-in Styles

| Style | Purpose |
|-------|---------|
| `default` | Efficient software engineering |
| `explanatory` | Educational insights while working |
| `learning` | Collaborative, learn-by-doing |

### File Locations

| Scope | Path |
|-------|------|
| Personal | `~/.claude/output-styles/*.md` |
| Project | `.claude/output-styles/*.md` |

### Frontmatter Fields

| Field | Required | Default |
|-------|----------|---------|
| `name` | No | File name |
| `description` | No | None |
| `keep-coding-instructions` | No | false |

---

## Sources

- [Output Styles - Official Documentation](https://code.claude.com/docs/en/output-styles.md)
- [Settings Reference](https://code.claude.com/docs/en/settings.md)
- [Slash Commands](https://code.claude.com/docs/en/slash-commands.md)

---

## Next Steps

1. **Try built-in styles** - Switch between default, explanatory, and learning
2. **Create a custom style** - Start with a simple modification
3. **Share with team** - Add styles to your repository
4. **Experiment** - Find styles that match your workflow
