# Tutorial 54: Claude Code CLI Complete Reference

> **The definitive command-line reference for Claude Code v2.1.x**

## Table of Contents

1. [Overview](#overview)
2. [Starting Claude Code](#starting-claude-code)
3. [Session Management](#session-management)
4. [Chrome Integration](#chrome-integration)
5. [Plugin Management](#plugin-management)
6. [System Prompts](#system-prompts)
7. [MCP Configuration](#mcp-configuration)
8. [Tool Control](#tool-control)
9. [Output Control](#output-control)
10. [Print Mode (Non-Interactive)](#print-mode-non-interactive)
11. [Agents & Subagents](#agents--subagents)
12. [Directory Management](#directory-management)
13. [Permission Control](#permission-control)
14. [Interactive Shortcuts](#interactive-shortcuts)
15. [Diagnostics & Maintenance](#diagnostics--maintenance)
16. [Environment Variables](#environment-variables)
17. [Slash Commands Reference](#slash-commands-reference)
18. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)

---

## Overview

Claude Code is Anthropic's official CLI for interacting with Claude. This tutorial provides a complete reference of all CLI commands, flags, and interactive features as of version 2.1.x.

### Installation Verification

```bash
# Check installation and version
claude --version
claude -v

# Run diagnostics
claude doctor
```

### Basic Usage Patterns

```bash
# Start interactive session
claude

# Start with initial prompt
claude "explain this codebase"

# Non-interactive (print mode)
claude -p "generate a README"

# Resume previous session
claude --resume

# Continue last session
claude --continue
```

---

## Starting Claude Code

### Basic Startup

```bash
# Interactive session in current directory
claude

# With initial prompt
claude "your prompt here"

# Multiple prompts (processed in sequence)
claude "first task" "second task"
```

### Startup Flags

| Flag | Short | Description |
|------|-------|-------------|
| `--version` | `-v` | Show version number |
| `--help` | `-h` | Show help message |
| `--debug` | | Enable debug logging |
| `--verbose` | | Enable verbose output |
| `--init` | | Run initialization hooks only |
| `--init-only` | | Run init then exit |
| `--maintenance` | | Run maintenance operations |

### Model Selection

```bash
# Specify model at startup
claude --model claude-sonnet-4-20250514
claude --model claude-opus-4-5-20251101
claude --model claude-haiku-3-5-20241022

# Use model alias
claude --model sonnet
claude --model opus
claude --model haiku
```

### Working Directory

```bash
# Start in specific directory
cd /path/to/project && claude

# Add additional directories to context
claude --add-dir /path/to/other/project
claude --add-dir ./lib --add-dir ./shared
```

---

## Session Management

### Session Resume & Continue

```bash
# Resume most recent session
claude --resume
claude -r

# Resume specific session by ID
claude --resume abc123
claude -r abc123

# Continue last session (alias for resume most recent)
claude --continue
claude -c

# Fork from existing session (creates new session with history)
claude --fork-session abc123
```

### Session ID Management

```bash
# Specify session ID for new session
claude --session-id my-custom-id

# Combine with resume (resume specific, or create if not exists)
claude --session-id my-feature-work --resume
```

### Session Examples

```bash
# Start a named session for a feature
claude --session-id feature-auth "implement user authentication"

# Later, resume that specific session
claude --resume feature-auth

# Fork session to try alternative approach
claude --fork-session feature-auth "try OAuth instead"
```

---

## Chrome Integration

> Added in v2.0.72

Chrome integration enables browser automation directly from Claude Code.

### Startup Flags

```bash
# Enable Chrome integration (default when extension installed)
claude --chrome

# Disable Chrome integration
claude --no-chrome

# Update Chrome extension
claude update
```

### Chrome Slash Command

```
/chrome          # Show Chrome extension status
/chrome status   # Detailed status info
/chrome connect  # Attempt to connect to extension
```

### Chrome Features

When Chrome integration is enabled:
- Browser automation via MCP tools (`mcp__claude-in-chrome__*`)
- Screenshot capture and analysis
- DOM interaction and form filling
- Page navigation and content extraction
- GIF recording of browser sessions

---

## Plugin Management

### Plugin Commands

```bash
# List installed plugins
claude plugin list

# Install plugin from registry
claude plugin install plugin-name

# Install from specific source
claude plugin install --scope @myorg plugin-name

# Install from local directory
claude plugin install --plugin-dir ./my-plugin

# Uninstall plugin
claude plugin uninstall plugin-name

# Enable/disable plugins
claude plugin enable plugin-name
claude plugin disable plugin-name

# Update plugins
claude plugin update              # Update all
claude plugin update plugin-name  # Update specific
```

### Plugin Startup Flags

```bash
# Load plugin from directory
claude --plugin-dir ./my-plugin

# Multiple plugins
claude --plugin-dir ./plugin1 --plugin-dir ./plugin2

# Specify plugin scope
claude --scope @myorg
```

### Plugin Environment Variables

```bash
# Force plugin auto-update
export FORCE_AUTOUPDATE_PLUGINS=true
claude
```

---

## System Prompts

### System Prompt Flags

```bash
# Set system prompt directly
claude --system-prompt "You are a Python expert"

# Load system prompt from file
claude --system-prompt-file ./prompts/python-expert.txt

# Append to default system prompt
claude --append-system-prompt "Always use type hints"

# Append from file
claude --append-system-prompt-file ./prompts/extra-instructions.txt
```

### Combining System Prompts

```bash
# Replace system prompt with file, then append extra
claude --system-prompt-file ./base-prompt.txt \
       --append-system-prompt "Focus on security"

# Multiple appends
claude --append-system-prompt "Use TypeScript" \
       --append-system-prompt "Follow ESLint rules"
```

### System Prompt Examples

```bash
# Code review specialist
claude --system-prompt "You are a senior code reviewer. Focus on:
1. Security vulnerabilities
2. Performance issues
3. Code maintainability
Always provide specific line numbers."

# Documentation writer
claude --system-prompt-file ./prompts/docs-writer.txt \
       "Document the API in src/api/"
```

---

## MCP Configuration

### MCP Startup Flags

```bash
# Load MCP configuration from file
claude --mcp-config ./mcp-config.json

# Strict MCP mode (fail if MCP config invalid)
claude --strict-mcp-config --mcp-config ./mcp-config.json
```

### MCP Commands

```bash
# List MCP servers
claude mcp list

# Add MCP server
claude mcp add server-name

# Remove MCP server
claude mcp remove server-name

# MCP server status
claude mcp status
```

### MCP Configuration File Format

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"]
    },
    "database": {
      "command": "python",
      "args": ["-m", "mcp_server_sqlite", "--db", "./data.db"]
    }
  }
}
```

### MCP Tool Search (v2.1.7+)

```bash
# Auto-defer tools when they exceed context threshold
# Configure in settings:
{
  "mcpToolSearch": "auto:10"  # Defer when tools > 10% of context
}
```

---

## Tool Control

### Tool Restriction Flags

```bash
# Allow only specific tools
claude --allowedTools "Read,Grep,Glob"

# Disallow specific tools
claude --disallowedTools "Bash,Write"

# Combined tool specification (JSON format)
claude --tools '{"allowed": ["Read", "Grep"], "disallowed": ["Bash"]}'
```

### Tool Patterns

```bash
# Read-only mode
claude --allowedTools "Read,Glob,Grep,WebFetch"

# File operations only
claude --allowedTools "Read,Write,Edit,Glob,Grep"

# Full access except bash
claude --disallowedTools "Bash"

# Restrict to specific MCP tools
claude --allowedTools "mcp__filesystem__*"
```

### Wildcard Permissions (v2.1.0+)

```bash
# In settings.json - allow all npm commands
{
  "permissions": {
    "allow": ["Bash(npm *)"]
  }
}

# Allow all git commands except force push
{
  "permissions": {
    "allow": ["Bash(git *)"],
    "deny": ["Bash(git push -f *)"]
  }
}

# MCP tool wildcards
{
  "permissions": {
    "allow": ["mcp__database__*"]
  }
}

# Task agent permissions
{
  "permissions": {
    "allow": ["Task(Explore)", "Task(Plan)"]
  }
}
```

---

## Output Control

### Output Format Flags

```bash
# JSON output (complete response)
claude -p "query" --output-format json

# Streaming JSON output
claude -p "query" --output-format stream-json

# Enable/disable streaming
claude -p "query" --stream
claude -p "query" --no-stream
```

### JSON Schema Validation

```bash
# Validate output against JSON schema
claude -p "generate user data" --json-schema '{"type": "object", "properties": {"name": {"type": "string"}}}'

# Schema from file
claude -p "generate config" --json-schema-file ./schema.json
```

### Output Examples

```bash
# Get structured JSON response
claude -p "list all functions in this file" \
  --output-format json \
  --json-schema '{"type": "array", "items": {"type": "object", "properties": {"name": {"type": "string"}, "line": {"type": "number"}}}}'

# Stream JSON for real-time processing
claude -p "analyze codebase" --output-format stream-json | jq -c '.content'
```

---

## Print Mode (Non-Interactive)

Print mode (`-p` or `--print`) runs Claude non-interactively, outputting results and exiting.

### Basic Print Mode

```bash
# Simple query
claude -p "what does this function do?"

# With file context
claude -p "explain this code" < main.py

# Pipe input
cat error.log | claude -p "explain this error"
```

### Print Mode Limits

```bash
# Limit API turns (round-trips)
claude -p "refactor this file" --max-turns 5

# Limit budget
claude -p "comprehensive review" --max-budget-usd 1.00

# Combined limits
claude -p "full analysis" --max-turns 10 --max-budget-usd 0.50
```

### Input Format

```bash
# Specify input format
claude -p --input-format text "plain text input"
claude -p --input-format json '{"query": "structured input"}'

# Stdin input format
echo '{"task": "analyze"}' | claude -p --input-format json
```

### Print Mode with Tools

```bash
# Print mode with specific tools
claude -p "run tests and report" --allowedTools "Bash,Read,Grep"

# Print mode with auto-accept (for automation)
claude -p "fix linting errors" --permission-mode acceptEdits
```

### Automation Examples

```bash
# CI/CD: Run tests and report
claude -p "run npm test and summarize failures" \
  --max-turns 5 \
  --allowedTools "Bash,Read" \
  --permission-mode bypassPermissions

# Generate documentation
claude -p "generate API documentation for src/api/" \
  --output-format json \
  --max-budget-usd 0.25

# Code review automation
cat diff.patch | claude -p "review this diff for issues" \
  --allowedTools "Read,Grep"
```

---

## Agents & Subagents

### Agent Selection

```bash
# Use specific agent type
claude --agent Explore "find authentication code"
claude --agent Plan "design user management feature"
claude --agent builder "implement the login form"

# Multiple agents (JSON format)
claude --agents '[{"type": "Explore", "task": "find tests"}, {"type": "builder", "task": "fix failing tests"}]'
```

### Available Agent Types

| Agent | Purpose | Tools |
|-------|---------|-------|
| `Explore` | Quick codebase exploration | All except Edit, Write |
| `Plan` | Design implementation plans | All except Edit, Write |
| `builder` | Write clean, tested code | All |
| `reviewer` | Code review and feedback | Read, Grep, Glob |
| `fixer` | Fix bugs and issues | All |
| `test-writer` | Create test suites | All |
| `doc-fetcher` | Fetch documentation | Read, Write, WebFetch |
| `security-auditor` | Security analysis | Read, Grep, Glob |
| `refactorer` | Code improvement | All |

### Custom Subagents

Define custom agents in settings:

```json
{
  "agents": {
    "my-reviewer": {
      "description": "Custom Python code reviewer",
      "tools": ["Read", "Grep", "Glob"],
      "systemPrompt": "You are a Python expert focused on PEP 8 compliance"
    }
  }
}
```

Use custom agent:

```bash
claude --agent my-reviewer "review the models directory"
```

---

## Directory Management

### Working Directory

```bash
# Claude uses current working directory by default
cd /path/to/project
claude

# Add additional directories
claude --add-dir ../shared-lib
claude --add-dir ./packages/core --add-dir ./packages/ui
```

### Directory Context

When multiple directories are added:
- All directories are available for file operations
- `@` mentions can reference files from any added directory
- Relative paths are resolved from each directory's root

### Example: Monorepo Setup

```bash
# Working in a monorepo
cd /projects/my-monorepo/apps/web
claude --add-dir ../../packages/shared --add-dir ../../packages/ui \
  "refactor to use shared components"
```

---

## Permission Control

### Permission Modes

```bash
# Default mode (interactive confirmations)
claude --permission-mode default

# Plan mode (read-only, no execution)
claude --permission-mode plan

# Auto-accept file edits
claude --permission-mode acceptEdits

# Bypass all permissions (DANGEROUS!)
claude --permission-mode bypassPermissions

# Deprecated alias
claude --dangerously-skip-permissions
```

### Permission Prompt Tool

```bash
# Custom permission prompt handler
claude --permission-prompt-tool ./my-permission-handler.sh
```

### Permission Mode Comparison

| Mode | File Read | File Write | Bash | Use Case |
|------|-----------|------------|------|----------|
| `default` | Ask/Allow | Ask | Ask | Daily development |
| `plan` | Allow | Deny | Limited | Code review |
| `acceptEdits` | Allow | Auto | Ask | Trusted refactoring |
| `bypassPermissions` | Auto | Auto | Auto | CI/CD (careful!) |

---

## Interactive Shortcuts

### Basic Controls

| Shortcut | Action |
|----------|--------|
| `Ctrl+C` | Cancel current operation |
| `Ctrl+D` | Exit session |
| `Ctrl+L` | Clear screen |
| `Ctrl+R` | Reverse history search |
| `Ctrl+V` | Paste (terminal dependent) |

### Model & Thinking Controls (v2.0.65+)

| Shortcut | Action |
|----------|--------|
| `Alt+T` | Toggle extended thinking display |
| `Alt+P` | Quick model switch |
| `Alt+Y` | Yank-pop (paste from history) |

### Navigation

| Shortcut | Action |
|----------|--------|
| `Shift+Tab` | Cycle through suggestions |
| `↑` / `↓` | Navigate command history |
| `Tab` | Autocomplete |
| Arrow keys | Dialog/menu navigation |

### Vim Mode (v2.1.0+)

Claude Code supports Vim-style editing when enabled:

| Motion | Action |
|--------|--------|
| `h`, `j`, `k`, `l` | Character/line navigation |
| `w`, `b`, `e` | Word navigation |
| `;`, `,` | Repeat `f`/`t` motions |
| `y` | Yank (copy) |
| `p`, `P` | Paste after/before |
| `>>`, `<<` | Indent/dedent line |
| `J` | Join lines |
| Text objects | `iw`, `aw`, `i"`, `a"`, etc. |

### Bash Mode

```bash
# Direct bash execution with ! prefix
!npm test
!git status
!ls -la

# Runs immediately without Claude interpretation
```

### Shift+Enter (v2.1.0+)

In supported terminals:
- `Shift+Enter` inserts a newline without submitting
- Allows multi-line input before sending

---

## Diagnostics & Maintenance

### Doctor Command

```bash
# Run full diagnostics
claude doctor

# Doctor checks:
# - Claude Code version
# - Node.js version
# - Network connectivity
# - API authentication
# - MCP server status
# - Permission configuration
# - Unreachable permission rules (v2.1.3+)
# - Available updates
```

### Update Commands

```bash
# Check for updates
claude update --check

# Install updates
claude update

# Update Chrome extension
claude update chrome
```

### Debug Mode

```bash
# Enable debug output
claude --debug

# Verbose mode for detailed logs
claude --verbose

# Both
claude --debug --verbose
```

### Maintenance Operations

```bash
# Run maintenance (cleanup, cache clear)
claude --maintenance

# Init-only mode (run hooks, then exit)
claude --init-only
```

---

## Environment Variables

### Authentication

| Variable | Purpose |
|----------|---------|
| `ANTHROPIC_API_KEY` | API authentication |
| `CLAUDE_API_KEY` | Alternative API key variable |

### Behavior Control

| Variable | Purpose | Example |
|----------|---------|---------|
| `CLAUDE_CODE_SHELL` | Override shell detection | `/bin/zsh` |
| `CLAUDE_CODE_TMPDIR` | Override temp directory | `/tmp/claude` |
| `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` | Disable backgrounding | `true` |
| `CLAUDE_CODE_FILE_READ_MAX_OUTPUT_TOKENS` | File read token limit | `10000` |

### Display & Demo

| Variable | Purpose |
|----------|---------|
| `IS_DEMO` | Hide email/org from UI |
| `FORCE_AUTOUPDATE_PLUGINS` | Force plugin auto-updates |

### Session Variables (Available in Skills)

| Variable | Purpose |
|----------|---------|
| `${CLAUDE_SESSION_ID}` | Current session ID |

---

## Slash Commands Reference

### Session Management

| Command | Purpose |
|---------|---------|
| `/help` | Show available commands |
| `/status` | Session status and info |
| `/clear` | Clear conversation context |
| `/compact` | Compress context (summarize) |
| `/rewind` | Undo recent changes |
| `/tasks` | List background tasks |

### Configuration

| Command | Purpose |
|---------|---------|
| `/config` | Open settings |
| `/config <search>` | Search settings (v2.1.6+) |
| `/model` | Change model |
| `/model <name>` | Set specific model |
| `/permissions` | View permission rules |
| `/permissions <search>` | Search permissions (v2.0.67+) |

### Planning & Workflow

| Command | Purpose |
|---------|---------|
| `/plan` | Enter plan mode (v2.1.0+) |
| `/plan <path>` | Create/edit plan file |

### Statistics & Info

| Command | Purpose |
|---------|---------|
| `/cost` | Show token usage and cost |
| `/stats` | Session statistics |
| `/stats <date>` | Filter stats by date (v2.1.6+) |
| `/doctor` | Run diagnostics |

### Chrome Integration

| Command | Purpose |
|---------|---------|
| `/chrome` | Chrome extension status |
| `/terminal-setup` | Terminal configuration help |

### Remote & Teleport

| Command | Purpose |
|---------|---------|
| `/remote-env` | Remote environment info (v2.1.0+) |

### File Operations

| Command | Purpose |
|---------|---------|
| `@<file>` | Reference file in context |
| `@<folder>/` | Reference folder |
| `@<url>` | Fetch and reference URL |

---

## Quick Reference Cheat Sheet

### Startup Quick Reference

```bash
# Interactive
claude                          # Start session
claude "prompt"                 # With initial prompt
claude -r                       # Resume last
claude -c                       # Continue last

# Non-interactive
claude -p "prompt"              # Print mode
claude -p "prompt" --max-turns 5

# Model selection
claude --model sonnet
claude --model opus
```

### Permission Quick Reference

```bash
claude --permission-mode default        # Interactive
claude --permission-mode plan           # Read-only
claude --permission-mode acceptEdits    # Auto-accept edits
claude --permission-mode bypassPermissions  # Full auto (careful!)
```

### Tool Quick Reference

```bash
claude --allowedTools "Read,Grep,Glob"      # Restrict tools
claude --disallowedTools "Bash"             # Exclude tools
```

### Output Quick Reference

```bash
claude -p "query" --output-format json      # JSON output
claude -p "query" --stream                  # Streaming
claude -p "query" --no-stream               # No streaming
```

### Session Quick Reference

```bash
claude --session-id my-session              # Named session
claude -r my-session                        # Resume named
claude --fork-session abc123                # Fork session
```

### Interactive Shortcuts Quick Reference

| Keys | Action |
|------|--------|
| `Ctrl+C` | Cancel |
| `Ctrl+D` | Exit |
| `Alt+T` | Toggle thinking |
| `Alt+P` | Switch model |
| `Esc Esc` | Rewind menu |
| `!cmd` | Direct bash |

### Slash Commands Quick Reference

```
/help     /status   /clear    /compact
/model    /config   /cost     /stats
/plan     /rewind   /tasks    /doctor
```

---

## Migration from TUTORIAL_40

This tutorial (54) supersedes TUTORIAL_40 (CLI Reference). Key additions:

### New in v2.1.x
- Chrome integration (`--chrome`, `/chrome`)
- Session ID management (`--session-id`, `--fork-session`)
- Full plugin management commands
- MCP tool search auto-mode
- Wildcard permissions (`Bash(npm *)`)
- Advanced Vim motions
- Alt+T/P/Y shortcuts
- `/config` and `/stats` search
- Environment variables deep dive

### Deprecated
- `--dangerously-skip-permissions` (use `--permission-mode bypassPermissions`)

---

## Sources

- [CLI Reference](https://code.claude.com/docs/en/cli-reference.md)
- [Interactive Mode](https://code.claude.com/docs/en/interactive-mode.md)
- [Slash Commands](https://code.claude.com/docs/en/slash-commands.md)
- [Settings](https://code.claude.com/docs/en/settings.md)
- [Changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

---

*This tutorial is part of the Claude Code documentation series.*
*Version 2.1.x | Last Updated: January 2026*
