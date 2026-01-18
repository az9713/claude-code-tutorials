# Tutorial 17: Creating Claude Code Plugins

> **Create custom plugins to extend Claude Code with slash commands, agents, hooks, Skills, and MCP servers**

## Overview

Plugins let you extend Claude Code with custom functionality that can be shared across projects and teams. This tutorial covers creating your own plugins with all available components.

### What You'll Learn

- Understanding when to use plugins vs standalone configuration
- Creating plugins with slash commands
- Adding agents, Skills, and hooks to plugins
- Including MCP and LSP servers
- Testing, debugging, and sharing plugins

### Prerequisites

- Claude Code installed (version 1.0.33 or later)
- Basic understanding of Claude Code features
- Familiarity with JSON and Markdown

---

## Part 1: Plugins vs Standalone Configuration

### When to Use Each

| Approach | Slash Command Names | Best For |
|----------|---------------------|----------|
| **Standalone** (`.claude/` directory) | `/hello` | Personal workflows, project-specific, quick experiments |
| **Plugins** (`.claude-plugin/plugin.json`) | `/plugin-name:hello` | Team sharing, community distribution, versioned releases |

### Use Standalone Configuration When:

- Customizing for a single project
- Personal configuration that doesn't need sharing
- Experimenting before packaging
- Short command names like `/hello` or `/review`

### Use Plugins When:

- Sharing functionality with team or community
- Need same commands across multiple projects
- Want version control and easy updates
- Distributing through a marketplace
- Okay with namespaced commands like `/my-plugin:hello`

> **Tip:** Start with standalone `.claude/` for quick iteration, then convert to a plugin when ready to share.

---

## Part 2: Creating Your First Plugin

### Step 1: Create the Plugin Directory

```bash
mkdir my-first-plugin
```

### Step 2: Create the Plugin Manifest

Create `.claude-plugin/plugin.json`:

```bash
mkdir my-first-plugin/.claude-plugin
```

**my-first-plugin/.claude-plugin/plugin.json:**
```json
{
  "name": "my-first-plugin",
  "description": "A greeting plugin to learn the basics",
  "version": "1.0.0",
  "author": {
    "name": "Your Name"
  }
}
```

**Manifest Fields:**

| Field | Purpose |
|-------|---------|
| `name` | Unique identifier and slash command namespace |
| `description` | Shown in plugin manager when browsing |
| `version` | Track releases using semantic versioning |
| `author` | Optional; helpful for attribution |

**Additional optional fields:** `homepage`, `repository`, `license`, `keywords`

### Step 3: Add a Slash Command

Create the commands directory:

```bash
mkdir my-first-plugin/commands
```

**my-first-plugin/commands/hello.md:**
```markdown
---
description: Greet the user with a friendly message
---

# Hello Command

Greet the user warmly and ask how you can help them today.
```

### Step 4: Test Your Plugin

```bash
# Load plugin for testing
claude --plugin-dir ./my-first-plugin
```

In Claude Code:
```
/my-first-plugin:hello
```

Run `/help` to see your command listed.

### Step 5: Add Command Arguments

Update `hello.md` to accept user input:

```markdown
---
description: Greet the user with a personalized message
---

# Hello Command

Greet the user named "$ARGUMENTS" warmly and ask how you can help them today.
Make the greeting personal and encouraging.
```

Usage:
```
/my-first-plugin:hello Alex
```

**Argument Syntax:**

| Placeholder | Description |
|-------------|-------------|
| `$ARGUMENTS` | All arguments as a single string |
| `$1`, `$2`, etc. | Individual positional arguments |

---

## Part 3: Plugin Structure Overview

### Directory Layout

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json       # Manifest (REQUIRED)
├── commands/              # Slash commands as Markdown files
│   └── hello.md
├── agents/                # Custom agent definitions
│   └── reviewer.md
├── skills/                # Agent Skills with SKILL.md files
│   └── code-review/
│       └── SKILL.md
├── hooks/                 # Event handlers
│   └── hooks.json
├── .mcp.json              # MCP server configurations
└── .lsp.json              # LSP server configurations
```

> **Warning:** Don't put `commands/`, `agents/`, `skills/`, or `hooks/` inside `.claude-plugin/`. Only `plugin.json` goes there.

---

## Part 4: Adding Slash Commands

### Basic Command

**commands/review.md:**
```markdown
---
description: Review code for best practices
---

Review the code I've selected or the recent changes for:
- Potential bugs or edge cases
- Security concerns
- Performance issues
- Readability improvements

Be concise and actionable.
```

### Command with Arguments

**commands/explain.md:**
```markdown
---
description: Explain code in detail
---

Explain the following code: $ARGUMENTS

Provide:
1. What the code does
2. How it works step by step
3. Any potential improvements
```

### Command with Thinking Mode

**commands/analyze.md:**
```markdown
---
description: Deep analysis of code architecture
thinking: true
---

Perform a thorough analysis of the codebase architecture.

Consider:
- Design patterns used
- Separation of concerns
- Dependency management
- Scalability considerations
```

### Command with Tool Restrictions

**commands/safe-review.md:**
```markdown
---
description: Read-only code review
allowed-tools:
  - Read
  - Search
  - Grep
---

Review the code without making any modifications.
Only read and analyze, do not edit files.
```

---

## Part 5: Adding Agents

Create custom agents that can be spawned as subagents.

### Basic Agent

**agents/security-reviewer.md:**
```markdown
---
description: Security-focused code reviewer
model: claude-sonnet-4-20250514
---

You are a security expert reviewing code for vulnerabilities.

Focus on:
- SQL injection
- XSS vulnerabilities
- Authentication issues
- Authorization flaws
- Sensitive data exposure
- Insecure dependencies

Provide specific recommendations with code examples.
```

### Agent with Tool Restrictions

**agents/doc-writer.md:**
```markdown
---
description: Documentation generator
allowed-tools:
  - Read
  - Write
  - Edit
---

You are a technical writer. Generate clear, comprehensive documentation.

Follow these guidelines:
- Use clear, concise language
- Include code examples
- Add diagrams where helpful
- Link to related documentation
```

---

## Part 6: Adding Skills

Skills are model-invoked: Claude automatically uses them based on task context.

### Basic Skill

**skills/code-review/SKILL.md:**
```markdown
---
name: code-review
description: Reviews code for best practices and potential issues. Use when reviewing code, checking PRs, or analyzing code quality.
---

When reviewing code, check for:
1. Code organization and structure
2. Error handling
3. Security concerns
4. Test coverage
5. Documentation completeness
```

### Skill with Supporting Files

```
skills/
└── api-design/
    ├── SKILL.md
    ├── patterns.md        # Reference patterns
    ├── examples/
    │   ├── rest-api.md
    │   └── graphql-api.md
    └── checklists/
        └── review-checklist.md
```

**skills/api-design/SKILL.md:**
```markdown
---
name: api-design
description: Helps design RESTful and GraphQL APIs following best practices.
---

Use this skill when designing APIs.

See @patterns.md for common patterns.
See @examples/ for reference implementations.
Use @checklists/review-checklist.md for final review.
```

### Skill with Tool Restrictions

**skills/security-audit/SKILL.md:**
```markdown
---
name: security-audit
description: Audits code for security vulnerabilities.
allowed-tools:
  - Read
  - Search
  - Grep
  - Bash(grep:*)
  - Bash(find:*)
---

Perform security auditing without modifying files.
Use read-only tools to analyze the codebase.
```

---

## Part 7: Adding Hooks

Hooks automate actions based on Claude Code events.

### Creating hooks.json

**hooks/hooks.json:**
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npm run lint:fix $FILE"
          }
        ]
      }
    ]
  }
}
```

### Available Hook Events

| Event | Trigger |
|-------|---------|
| `PreToolUse` | Before Claude uses a tool |
| `PostToolUse` | After Claude uses a tool |
| `SessionStart` | When a Claude Code session begins |
| `SessionEnd` | When a session ends |
| `Notification` | When Claude wants to notify user |
| `Stop` | When Claude completes a task |

### Example: Format on Save

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/format.sh $FILE"
          }
        ]
      }
    ]
  }
}
```

### Example: Pre-commit Validation

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash(git commit:*)",
        "hooks": [
          {
            "type": "command",
            "command": "npm run test && npm run lint"
          }
        ]
      }
    ]
  }
}
```

---

## Part 8: Adding MCP Servers

Include MCP (Model Context Protocol) servers in your plugin.

### Basic MCP Configuration

**.mcp.json:**
```json
{
  "my-database": {
    "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
    "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"]
  }
}
```

### MCP Server with Environment Variables

```json
{
  "github-integration": {
    "command": "node",
    "args": ["${CLAUDE_PLUGIN_ROOT}/servers/github/index.js"],
    "env": {
      "GITHUB_TOKEN": "${env:GITHUB_TOKEN}"
    }
  }
}
```

> **Note:** Use `${CLAUDE_PLUGIN_ROOT}` to reference files within the plugin's installation directory.

---

## Part 9: Adding LSP Servers

Include Language Server Protocol servers for code intelligence.

### Basic LSP Configuration

**.lsp.json:**
```json
{
  "go": {
    "command": "gopls",
    "args": ["serve"],
    "extensionToLanguage": {
      ".go": "go"
    }
  }
}
```

### Multiple Language Servers

```json
{
  "typescript": {
    "command": "typescript-language-server",
    "args": ["--stdio"],
    "extensionToLanguage": {
      ".ts": "typescript",
      ".tsx": "typescriptreact"
    }
  },
  "python": {
    "command": "pylsp",
    "args": [],
    "extensionToLanguage": {
      ".py": "python"
    }
  }
}
```

> **Tip:** For common languages (TypeScript, Python, Rust), install pre-built LSP plugins from the official marketplace. Create custom LSP plugins only for unsupported languages.

---

## Part 10: Testing and Debugging

### Test During Development

```bash
# Load single plugin
claude --plugin-dir ./my-plugin

# Load multiple plugins
claude --plugin-dir ./plugin-one --plugin-dir ./plugin-two
```

### What to Test

- [ ] Commands work with `/command-name`
- [ ] Agents appear in `/agents`
- [ ] Skills trigger appropriately
- [ ] Hooks fire on correct events
- [ ] MCP servers connect successfully
- [ ] LSP provides code intelligence

### Debugging Issues

**Check structure:**
```bash
# Verify plugin structure
claude plugin validate ./my-plugin
```

**Common issues:**

| Issue | Solution |
|-------|----------|
| Directories in wrong place | Move to plugin root, not inside `.claude-plugin/` |
| Commands not appearing | Check markdown frontmatter syntax |
| Hooks not firing | Verify matcher pattern |
| MCP server fails | Check command and path variables |

### Validation Commands

```bash
# Validate plugin structure
claude plugin validate ./my-plugin

# From within Claude Code
/plugin validate ./my-plugin
```

---

## Part 11: Converting Standalone to Plugin

### Migration Steps

1. **Create plugin structure:**
```bash
mkdir -p my-plugin/.claude-plugin
```

2. **Create manifest:**
```json
{
  "name": "my-plugin",
  "description": "Migrated from standalone configuration",
  "version": "1.0.0"
}
```

3. **Copy existing files:**
```bash
cp -r .claude/commands my-plugin/
cp -r .claude/agents my-plugin/
cp -r .claude/skills my-plugin/
```

4. **Migrate hooks:**

Create `my-plugin/hooks/hooks.json` with your hooks configuration:
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [{ "type": "command", "command": "npm run lint:fix $FILE" }]
      }
    ]
  }
}
```

5. **Test:**
```bash
claude --plugin-dir ./my-plugin
```

### What Changes

| Standalone (`.claude/`) | Plugin |
|-------------------------|--------|
| Only available in one project | Shareable via marketplaces |
| Files in `.claude/commands/` | Files in `plugin-name/commands/` |
| Hooks in `settings.json` | Hooks in `hooks/hooks.json` |
| Must manually copy to share | Install with `/plugin install` |

---

## Part 12: Sharing Your Plugin

### Add Documentation

Create a `README.md` with:
- Installation instructions
- Usage examples
- Configuration options
- Changelog

### Version Your Plugin

Use semantic versioning in `plugin.json`:
- `1.0.0` → Initial release
- `1.1.0` → New features, backward compatible
- `2.0.0` → Breaking changes

### Distribution Options

1. **Marketplace**: Package and share through plugin marketplaces
2. **Direct path**: Share plugin directory via `--plugin-dir`
3. **Git repository**: Others can clone and use directly

See [Tutorial 18: Plugin Marketplaces](./TUTORIAL_18_PLUGIN_MARKETPLACES.md) for marketplace distribution.

---

## Complete Plugin Example

### Directory Structure

```
enterprise-tools/
├── .claude-plugin/
│   └── plugin.json
├── commands/
│   ├── review.md
│   ├── deploy.md
│   └── rollback.md
├── agents/
│   └── security-reviewer.md
├── skills/
│   └── compliance-check/
│       └── SKILL.md
├── hooks/
│   └── hooks.json
├── .mcp.json
├── scripts/
│   └── validate.sh
└── README.md
```

### plugin.json

```json
{
  "name": "enterprise-tools",
  "description": "Enterprise workflow automation tools",
  "version": "2.0.0",
  "author": {
    "name": "Enterprise Team",
    "email": "team@example.com"
  },
  "homepage": "https://docs.example.com/plugins",
  "repository": "https://github.com/company/enterprise-tools",
  "license": "MIT",
  "keywords": ["enterprise", "workflow", "automation"]
}
```

### commands/review.md

```markdown
---
description: Enterprise code review with compliance checks
allowed-tools:
  - Read
  - Search
  - Grep
---

# Enterprise Code Review

Perform a comprehensive code review following enterprise standards.

Check for:
1. Security vulnerabilities (OWASP Top 10)
2. PII/PHI data handling
3. Logging compliance
4. Error handling standards
5. Test coverage requirements

Use the compliance-check skill for detailed analysis.
```

### hooks/hooks.json

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
          }
        ]
      }
    ]
  }
}
```

---

## Quick Reference

### Plugin Manifest Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Plugin identifier (kebab-case) |
| `description` | No | Brief description |
| `version` | No | Semantic version |
| `author` | No | Author info (name, email) |
| `homepage` | No | Documentation URL |
| `repository` | No | Source code URL |
| `license` | No | SPDX license identifier |
| `keywords` | No | Discovery tags |

### Command Frontmatter

| Field | Description |
|-------|-------------|
| `description` | Command description |
| `allowed-tools` | Restrict available tools |
| `thinking` | Enable extended thinking |

### Environment Variables

| Variable | Description |
|----------|-------------|
| `${CLAUDE_PLUGIN_ROOT}` | Plugin installation directory |
| `$FILE` | Current file path (in hooks) |
| `$ARGUMENTS` | All command arguments |
| `$1`, `$2`, etc. | Positional arguments |

---

## Sources

- [Create Plugins - Official Documentation](https://code.claude.com/docs/en/plugins.md)
- [Plugins Reference](https://code.claude.com/docs/en/plugins-reference.md)
- [Slash Commands](https://code.claude.com/docs/en/slash-commands.md)
- [Hooks Reference](https://code.claude.com/docs/en/hooks.md)

---

## Next Steps

1. **Create a simple plugin** - Start with one command
2. **Add more components** - Include agents or skills
3. **Test thoroughly** - Use `--plugin-dir` for development
4. **Share with your team** - See [Tutorial 18: Plugin Marketplaces](./TUTORIAL_18_PLUGIN_MARKETPLACES.md)
