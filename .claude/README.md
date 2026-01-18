# Reference `.claude` Directory

A comprehensive Claude Code configuration for experienced full-stack developers. Copy this directory to your project root to get started immediately.

## Quick Start

```bash
# Copy to your project
cp -r .claude /path/to/your/project/

# Customize settings
cd /path/to/your/project/.claude
cp settings.local.json.example settings.local.json
# Edit settings.local.json for your local environment
```

## Directory Structure

```
.claude/
├── settings.json              # Shared team settings (commit this)
├── settings.local.json.example # Template for local overrides
├── mcp.json                   # MCP server configurations
├── .claudeignore              # Files to exclude from context
├── commands/                  # Slash commands (/command-name)
│   ├── commit.md              # /commit - Smart git commits
│   ├── pr.md                  # /pr - Create pull requests
│   ├── review.md              # /review - Code review
│   ├── test.md                # /test - Run tests intelligently
│   ├── deploy/
│   │   ├── staging.md         # /deploy:staging
│   │   └── production.md      # /deploy:production
│   └── db/
│       ├── migrate.md         # /db:migrate
│       └── seed.md            # /db:seed
└── skills/                    # Domain knowledge & patterns
    ├── workflow/              # Development workflows
    │   ├── git-workflow.md    # Branching, commits, PRs
    │   ├── tdd.md             # Test-Driven Development
    │   ├── debugging.md       # Systematic debugging
    │   └── refactoring.md     # Safe refactoring patterns
    ├── frontend/              # Frontend development
    │   ├── react-patterns.md  # React hooks & patterns
    │   ├── accessibility.md   # a11y best practices
    │   └── performance.md     # Web performance
    ├── backend/               # Backend development
    │   ├── api-design.md      # REST API design
    │   ├── database.md        # Database patterns
    │   └── security.md        # Security best practices
    ├── devops/                # DevOps & infrastructure
    │   ├── docker.md          # Docker best practices
    │   ├── ci-cd.md           # CI/CD pipelines
    │   └── monitoring.md      # Observability
    └── quality/               # Code quality
        ├── code-review.md     # Review checklist
        ├── testing-strategy.md # Testing pyramid
        └── documentation.md   # Documentation patterns
```

## Using Commands

Commands are invoked with a slash prefix:

```
/commit              # Create a smart commit
/pr                  # Create a pull request
/review              # Review current changes
/review 123          # Review PR #123
/test                # Run tests
/deploy:staging      # Deploy to staging
/db:migrate run      # Run migrations
```

## Using Skills

Skills provide contextual knowledge. Claude will use them automatically when relevant, or you can reference them:

```
"Help me with the git workflow for this feature"
"Review this code using the code-review checklist"
"Apply the TDD approach to implement this function"
```

## Configuration Files

### settings.json

Shared team configuration. **Commit this file.**

Key sections:
- `permissions.allow` - Pre-approved commands
- `permissions.deny` - Blocked dangerous commands
- `context.excludePatterns` - Files to ignore

### settings.local.json

Your local overrides. **Do NOT commit this file.**

```json
{
  "model": { "default": "claude-opus-4-20250514" },
  "env": { "DATABASE_URL": "postgresql://localhost/mydb" }
}
```

### mcp.json

MCP server integrations. All servers are disabled by default. Enable what you need:

```json
{
  "mcpServers": {
    "github": {
      "disabled": false  // Enable GitHub integration
    }
  }
}
```

**Available servers:**
- `filesystem` - Extended file operations
- `github` - GitHub API integration
- `postgres` - PostgreSQL database access
- `sqlite` - SQLite database access
- `memory` - Persistent knowledge storage
- `puppeteer` - Browser automation
- `slack` - Slack integration

## Customization

### Adding a Command

Create a markdown file in `commands/`:

```markdown
# .claude/commands/my-command.md

# My Custom Command

Description of what this command does.

## Instructions

1. Step one
2. Step two

## Arguments

$ARGUMENTS - What arguments this accepts
```

Invoke with `/my-command`.

### Adding a Skill

Create a markdown file in `skills/`:

```markdown
# .claude/skills/my-domain/my-skill.md

---
name: my-skill
description: What this skill provides
---

# Skill Content

Knowledge and patterns for Claude to use...
```

### Modifying Permissions

In `settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(your-custom-command *)"
    ],
    "deny": [
      "Bash(dangerous-command *)"
    ]
  }
}
```

## What to Commit

| File | Commit? | Notes |
|------|---------|-------|
| `settings.json` | ✅ Yes | Shared team config |
| `settings.local.json` | ❌ No | Local overrides |
| `mcp.json` | ✅ Yes | No secrets, servers disabled |
| `commands/**` | ✅ Yes | Shared commands |
| `skills/**` | ✅ Yes | Shared knowledge |
| `.claudeignore` | ✅ Yes | Context exclusions |

Add to `.gitignore`:
```
.claude/settings.local.json
```

## Removing Features

This configuration is intentionally comprehensive. Remove what you don't need:

```bash
# Don't need frontend skills?
rm -rf .claude/skills/frontend/

# Don't need deployment commands?
rm -rf .claude/commands/deploy/

# Don't need MCP servers?
rm .claude/mcp.json
```

## Environment Variables

Some features require environment variables:

```bash
# For GitHub MCP server
export GITHUB_TOKEN=ghp_xxx

# For database MCP servers
export DATABASE_URL=postgresql://user:pass@host/db

# For Slack MCP server
export SLACK_BOT_TOKEN=xoxb_xxx
```

## Further Reading

See the tutorials in this repository:
- `TUTORIAL_05_SKILLS.md` - Skills in depth
- `TUTORIAL_03_SUBAGENTS.md` - Sub-agents and Task tool
- `TUTORIAL_04_MCP_SERVERS.md` - MCP server setup
- `TUTORIAL_08_WORKFLOWS.md` - Development workflows
