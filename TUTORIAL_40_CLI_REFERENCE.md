# Tutorial 40: CLI Reference

> **Complete command-line reference for Claude Code**

## Overview

Comprehensive reference of all Claude Code CLI commands and options.

---

## Part 1: Basic Commands

### Starting Claude

```bash
claude                    # Start interactive session
claude "prompt"           # Start with initial prompt
claude -p "prompt"        # Non-interactive (print mode)
```

### Common Flags

| Flag | Description |
|------|-------------|
| `-p`, `--print` | Non-interactive mode |
| `--model <name>` | Set model |
| `--resume` | Resume previous session |
| `--sandbox` | Enable sandboxing |
| `--verbose` | Verbose output |

---

## Part 2: Session Management

```bash
# Resume any session
claude --resume

# Resume specific session
claude --resume <session-id>

# Resume by name
claude --resume "my-session"

# Continue last session
claude --continue
```

---

## Part 3: Permissions

```bash
# Permission modes
claude --permission-mode default
claude --permission-mode acceptEdits
claude --permission-mode plan
claude --permission-mode bypassPermissions

# Allowed tools
claude --allowedTools "Read(*) Write(*)"
```

---

## Part 4: Output Control

```bash
# JSON output
claude -p "..." --output-format json

# Stream vs complete
claude -p "..." --stream
claude -p "..." --no-stream
```

---

## Part 5: System Prompt

```bash
# Append to system prompt
claude --append-system-prompt "Custom instructions"

# From file
claude --append-system-prompt "$(cat custom.txt)"
```

---

## Part 6: Plugin Management

```bash
# Load plugin
claude --plugin-dir ./my-plugin

# Multiple plugins
claude --plugin-dir ./plugin1 --plugin-dir ./plugin2
```

---

## Part 7: Background Execution

```bash
# Send to web for background execution
claude --remote "Your prompt"

# Teleport session to terminal
claude --teleport
```

---

## Part 8: Diagnostics

```bash
# System info
claude doctor

# Version check
claude --version

# Debug mode
claude --debug
```

---

## Quick Reference

### Essential Commands

| Command | Purpose |
|---------|---------|
| `claude` | Start session |
| `claude -p "..."` | Non-interactive |
| `claude --resume` | Resume session |
| `claude doctor` | Diagnostics |
| `claude --version` | Version info |

### Slash Commands (Interactive)

| Command | Purpose |
|---------|---------|
| `/help` | Show commands |
| `/cost` | Token usage |
| `/model` | Change model |
| `/clear` | Clear context |
| `/compact` | Compress context |
| `/config` | Settings |
| `/status` | Session info |
| `/tasks` | Background tasks |
| `/rewind` | Undo changes |

---

## Sources

- [CLI Reference](https://code.claude.com/docs/en/cli-reference.md)
- [Interactive Mode](https://code.claude.com/docs/en/interactive-mode.md)
