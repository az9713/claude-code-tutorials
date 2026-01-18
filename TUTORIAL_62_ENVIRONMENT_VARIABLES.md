# Tutorial 62: Environment Variables Deep Dive

> **Complete reference for all Claude Code environment variables and configuration**

## Table of Contents

1. [Overview](#overview)
2. [Authentication Variables](#authentication-variables)
3. [Runtime Configuration](#runtime-configuration)
4. [Shell & Terminal Variables](#shell--terminal-variables)
5. [Resource Limits](#resource-limits)
6. [Feature Flags](#feature-flags)
7. [Debug & Development](#debug--development)
8. [Enterprise & Deployment](#enterprise--deployment)
9. [Hook Environment Variables](#hook-environment-variables)
10. [Platform-Specific Variables](#platform-specific-variables)
11. [Precedence & Conflicts](#precedence--conflicts)
12. [Quick Reference](#quick-reference)

---

## Overview

Claude Code uses environment variables for configuration that needs to be:
- Set before startup
- Different across environments
- Kept out of version-controlled settings files
- Injected by CI/CD systems

### Variable Categories

| Category | Purpose | Examples |
|----------|---------|----------|
| Authentication | API keys, providers | `ANTHROPIC_API_KEY` |
| Runtime | Behavior during execution | `CLAUDE_CODE_SHELL` |
| Resource | Limits and quotas | `CLAUDE_CODE_FILE_READ_MAX_OUTPUT_TOKENS` |
| Feature | Enable/disable features | `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` |
| Debug | Development and troubleshooting | `CLAUDE_DEBUG` |

### Setting Environment Variables

**Bash/Zsh:**
```bash
export ANTHROPIC_API_KEY="sk-ant-..."
```

**Fish:**
```fish
set -gx ANTHROPIC_API_KEY "sk-ant-..."
```

**PowerShell:**
```powershell
$env:ANTHROPIC_API_KEY = "sk-ant-..."
```

**Permanent (add to shell profile):**
```bash
# ~/.bashrc or ~/.zshrc
export ANTHROPIC_API_KEY="sk-ant-..."
export CLAUDE_CODE_SHELL="/bin/zsh"
```

---

## Authentication Variables

### ANTHROPIC_API_KEY

**Purpose:** Primary API key for Claude API access

```bash
export ANTHROPIC_API_KEY="sk-ant-api03-..."
```

**Notes:**
- Required for direct API access
- Not needed when using third-party providers (Bedrock, Vertex AI)
- Store securely, never commit to version control

### ANTHROPIC_MODEL

**Purpose:** Override default model selection

```bash
export ANTHROPIC_MODEL="claude-sonnet-4-20250514"
```

**Common values:**
- `claude-opus-4-5-20251101`
- `claude-sonnet-4-20250514`
- `claude-haiku-3-5-20241022`

### ANTHROPIC_BASE_URL

**Purpose:** Custom API endpoint (for proxies or enterprise setups)

```bash
export ANTHROPIC_BASE_URL="https://api.myproxy.com/v1"
```

### AWS Credentials (Bedrock)

For Amazon Bedrock deployment:

```bash
export AWS_ACCESS_KEY_ID="AKIA..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_REGION="us-east-1"
export AWS_SESSION_TOKEN="..."  # If using temporary credentials
```

### Google Cloud (Vertex AI)

For Google Cloud Vertex AI:

```bash
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/service-account.json"
export CLOUDSDK_CORE_PROJECT="my-project-id"
export CLOUDSDK_COMPUTE_REGION="us-central1"
```

---

## Runtime Configuration

### CLAUDE_CODE_SHELL (v2.0.65)

**Purpose:** Override shell detection

```bash
export CLAUDE_CODE_SHELL="/bin/zsh"
export CLAUDE_CODE_SHELL="/usr/local/bin/fish"
export CLAUDE_CODE_SHELL="C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe"
```

**When to use:**
- Shell detection fails
- Want consistent behavior across environments
- Running in containers with minimal shells

### CLAUDE_CODE_TMPDIR (v2.1.5)

**Purpose:** Override temporary directory location

```bash
export CLAUDE_CODE_TMPDIR="/custom/tmp/claude"
export CLAUDE_CODE_TMPDIR="D:\\Temp\\Claude"
```

**Use cases:**
- Insufficient space on default tmp
- Security requirements for temp files
- RAM disk for performance

### CLAUDE_SESSION_ID

**Purpose:** Available within skills for session tracking (v2.1.9)

```bash
# Set automatically, but can be referenced in skills
${CLAUDE_SESSION_ID}
```

### HOME / USERPROFILE

**Purpose:** User home directory (standard OS variable)

Claude Code stores configuration in:
- Linux/macOS: `$HOME/.claude/`
- Windows: `%USERPROFILE%\.claude\`

---

## Shell & Terminal Variables

### TERM

**Purpose:** Terminal type identification

```bash
export TERM="xterm-256color"
```

**Affects:**
- Color support detection
- Cursor behavior
- Key sequence interpretation

### TERM_PROGRAM

**Purpose:** Terminal application name

```bash
# Set automatically by terminals
TERM_PROGRAM="iTerm.app"
TERM_PROGRAM="WezTerm"
TERM_PROGRAM="vscode"
```

### COLORTERM

**Purpose:** True color (24-bit) support indication

```bash
export COLORTERM="truecolor"
# or
export COLORTERM="24bit"
```

### LANG / LC_ALL

**Purpose:** Locale and encoding settings

```bash
export LANG="en_US.UTF-8"
export LC_ALL="en_US.UTF-8"
```

**Important for:**
- Unicode/emoji display
- CJK input handling
- Filename encoding

### EDITOR / VISUAL

**Purpose:** Default text editor

```bash
export EDITOR="vim"
export VISUAL="code --wait"
```

**Used when:**
- Opening files for editing
- Git commit messages
- Interactive editing prompts

---

## Resource Limits

### CLAUDE_CODE_FILE_READ_MAX_OUTPUT_TOKENS (v2.1.0)

**Purpose:** Maximum tokens when reading files

```bash
export CLAUDE_CODE_FILE_READ_MAX_OUTPUT_TOKENS="50000"
```

**Default:** Usually around 32,000 tokens

**Use cases:**
- Reading very large files
- Reducing token usage
- Preventing context overflow

### MAX_OUTPUT_TOKENS

**Purpose:** Maximum response length

```bash
export MAX_OUTPUT_TOKENS="8192"
```

### API Rate Limiting Variables

Some setups support:

```bash
export ANTHROPIC_RATE_LIMIT_REQUESTS_PER_MINUTE="60"
export ANTHROPIC_RATE_LIMIT_TOKENS_PER_MINUTE="100000"
```

---

## Feature Flags

### CLAUDE_CODE_DISABLE_BACKGROUND_TASKS (v2.1.4)

**Purpose:** Disable background task execution

```bash
export CLAUDE_CODE_DISABLE_BACKGROUND_TASKS="1"
```

**When to use:**
- Debugging timing issues
- Resource-constrained environments
- When background tasks cause problems

### FORCE_AUTOUPDATE_PLUGINS (v2.1.2)

**Purpose:** Control plugin auto-update behavior

```bash
export FORCE_AUTOUPDATE_PLUGINS="1"   # Force updates
export FORCE_AUTOUPDATE_PLUGINS="0"   # Disable updates
```

### IS_DEMO (v2.1.0)

**Purpose:** Hide email/organization from UI

```bash
export IS_DEMO="1"
```

**Use cases:**
- Screencasts and demos
- Training videos
- Public presentations

### NO_COLOR

**Purpose:** Disable all color output (standard convention)

```bash
export NO_COLOR="1"
```

**Effects:**
- Removes ANSI color codes
- Better for logging
- Accessibility

### FORCE_COLOR

**Purpose:** Force color output even when not a TTY

```bash
export FORCE_COLOR="1"
```

**Use cases:**
- CI/CD with color-capable output
- Piping to tools that support ANSI

---

## Debug & Development

### CLAUDE_DEBUG

**Purpose:** Enable debug output

```bash
export CLAUDE_DEBUG="1"
```

**Shows:**
- API request/response details
- Internal state changes
- Timing information

### CLAUDE_VERBOSE

**Purpose:** Verbose logging

```bash
export CLAUDE_VERBOSE="1"
```

### DEBUG

**Purpose:** General debug flag (Node.js convention)

```bash
export DEBUG="claude:*"
export DEBUG="claude:api,claude:tools"
```

### NODE_ENV

**Purpose:** Node.js environment mode

```bash
export NODE_ENV="development"  # More detailed errors
export NODE_ENV="production"   # Optimized for production
```

### LOG_LEVEL

**Purpose:** Logging verbosity

```bash
export LOG_LEVEL="debug"   # Most verbose
export LOG_LEVEL="info"    # Normal
export LOG_LEVEL="warn"    # Warnings and errors
export LOG_LEVEL="error"   # Errors only
```

---

## Enterprise & Deployment

### HTTP_PROXY / HTTPS_PROXY

**Purpose:** HTTP proxy configuration

```bash
export HTTP_PROXY="http://proxy.company.com:8080"
export HTTPS_PROXY="http://proxy.company.com:8080"
export NO_PROXY="localhost,127.0.0.1,.internal.company.com"
```

### SSL/TLS Configuration

```bash
export NODE_TLS_REJECT_UNAUTHORIZED="0"  # Disable certificate verification (NOT recommended)
export NODE_EXTRA_CA_CERTS="/path/to/company-ca.pem"  # Add custom CA
```

### CI/CD Variables

Common CI environment variables that Claude Code may detect:

```bash
# GitHub Actions
export GITHUB_ACTIONS="true"
export GITHUB_REPOSITORY="owner/repo"
export GITHUB_SHA="abc123..."

# GitLab CI
export GITLAB_CI="true"
export CI_PROJECT_PATH="group/project"

# Jenkins
export JENKINS_URL="http://jenkins.company.com"
export BUILD_NUMBER="123"

# Generic
export CI="true"
export CONTINUOUS_INTEGRATION="true"
```

### Organization Settings

```bash
export ANTHROPIC_ORGANIZATION_ID="org-..."
export ANTHROPIC_PROJECT_ID="proj-..."
```

---

## Hook Environment Variables

Variables available to hook scripts:

### All Hooks

| Variable | Description |
|----------|-------------|
| `CLAUDE_EVENT` | Event type (PreToolUse, PostToolUse, etc.) |
| `CLAUDE_SESSION_ID` | Current session identifier |
| `CLAUDE_WORKING_DIR` | Working directory |
| `CLAUDE_MODEL` | Current model name |

### Tool Hooks (PreToolUse, PostToolUse)

| Variable | Description |
|----------|-------------|
| `CLAUDE_TOOL_NAME` | Tool being invoked |
| `CLAUDE_TOOL_INPUT` | JSON string of tool parameters |
| `CLAUDE_TOOL_OUTPUT` | Tool result (PostToolUse only) |

### Example Hook Script

```bash
#!/bin/bash
echo "Event: $CLAUDE_EVENT"
echo "Session: $CLAUDE_SESSION_ID"
echo "Tool: $CLAUDE_TOOL_NAME"
echo "Input: $CLAUDE_TOOL_INPUT"

# Parse tool input
COMMAND=$(echo "$CLAUDE_TOOL_INPUT" | jq -r '.command // empty')
FILE_PATH=$(echo "$CLAUDE_TOOL_INPUT" | jq -r '.file_path // .path // empty')
```

---

## Platform-Specific Variables

### macOS

```bash
# Terminal identification
TERM_PROGRAM="Apple_Terminal"
TERM_PROGRAM="iTerm.app"

# Home directory
HOME="/Users/username"

# Path separator
PATH="/usr/local/bin:/usr/bin:/bin"
```

### Linux

```bash
# Common distributions set
XDG_CONFIG_HOME="$HOME/.config"
XDG_DATA_HOME="$HOME/.local/share"
XDG_CACHE_HOME="$HOME/.cache"

# Desktop environment
XDG_CURRENT_DESKTOP="GNOME"
DESKTOP_SESSION="ubuntu"

# Display (for GUI terminals)
DISPLAY=":0"
WAYLAND_DISPLAY="wayland-0"
```

### Windows

```powershell
# User profile
$env:USERPROFILE = "C:\Users\username"
$env:APPDATA = "C:\Users\username\AppData\Roaming"
$env:LOCALAPPDATA = "C:\Users\username\AppData\Local"

# System paths
$env:SystemRoot = "C:\Windows"
$env:TEMP = "C:\Users\username\AppData\Local\Temp"
```

### WSL (Windows Subsystem for Linux)

```bash
# WSL-specific
WSL_DISTRO_NAME="Ubuntu-22.04"
WSL_INTEROP="/run/WSL/..."

# Access Windows paths
WSLENV="PATH/l"
```

---

## Precedence & Conflicts

### Configuration Priority

From highest to lowest priority:

1. **Command-line arguments** (`--model`, `--shell`)
2. **Environment variables** (`ANTHROPIC_MODEL`, `CLAUDE_CODE_SHELL`)
3. **Workspace settings** (`.claude/settings.local.json`)
4. **Project settings** (`.claude/settings.json`)
5. **User settings** (`~/.claude/settings.json`)
6. **Default values**

### Conflict Resolution Examples

```bash
# Environment: ANTHROPIC_MODEL="claude-sonnet-4-20250514"
# settings.json: "model": "claude-haiku-3-5-20241022"

# Result: claude-sonnet-4-20250514 (env var wins)
```

```bash
# Environment: CLAUDE_CODE_SHELL="/bin/bash"
# Command line: claude --shell /bin/zsh

# Result: /bin/zsh (CLI wins)
```

### Checking Effective Configuration

```bash
# View current environment
env | grep -i claude
env | grep -i anthropic

# Check which settings are active
claude config show
```

---

## Quick Reference

### Essential Variables

```bash
# Authentication
export ANTHROPIC_API_KEY="sk-ant-..."

# Shell
export CLAUDE_CODE_SHELL="/bin/zsh"

# Locale
export LANG="en_US.UTF-8"

# Terminal
export TERM="xterm-256color"
export COLORTERM="truecolor"
```

### All CLAUDE_CODE_* Variables

| Variable | Purpose | Version |
|----------|---------|---------|
| `CLAUDE_CODE_SHELL` | Override shell | v2.0.65 |
| `CLAUDE_CODE_TMPDIR` | Override temp directory | v2.1.5 |
| `CLAUDE_CODE_FILE_READ_MAX_OUTPUT_TOKENS` | File read limit | v2.1.0 |
| `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` | Disable backgrounding | v2.1.4 |

### All ANTHROPIC_* Variables

| Variable | Purpose |
|----------|---------|
| `ANTHROPIC_API_KEY` | API authentication |
| `ANTHROPIC_MODEL` | Model override |
| `ANTHROPIC_BASE_URL` | Custom API endpoint |
| `ANTHROPIC_ORGANIZATION_ID` | Organization ID |
| `ANTHROPIC_PROJECT_ID` | Project ID |

### Feature Flag Summary

| Variable | Value | Effect |
|----------|-------|--------|
| `IS_DEMO` | `1` | Hide email/org |
| `NO_COLOR` | `1` | Disable colors |
| `FORCE_COLOR` | `1` | Force colors |
| `FORCE_AUTOUPDATE_PLUGINS` | `1`/`0` | Control plugin updates |
| `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` | `1` | No background tasks |

### Debug Variables

| Variable | Value | Effect |
|----------|-------|--------|
| `CLAUDE_DEBUG` | `1` | Debug mode |
| `CLAUDE_VERBOSE` | `1` | Verbose output |
| `DEBUG` | `claude:*` | Node.js debug |
| `LOG_LEVEL` | `debug` | Log verbosity |

### Shell Profile Template

```bash
# ~/.bashrc or ~/.zshrc

# Claude Code Authentication
export ANTHROPIC_API_KEY="sk-ant-..."

# Claude Code Configuration
export CLAUDE_CODE_SHELL="$SHELL"
export CLAUDE_CODE_TMPDIR="/tmp/claude"

# Terminal
export COLORTERM="truecolor"
export LANG="en_US.UTF-8"

# Optional: Debug (comment out for normal use)
# export CLAUDE_DEBUG="1"
# export CLAUDE_VERBOSE="1"
```

### Windows Profile Template

```powershell
# PowerShell Profile ($PROFILE)

# Claude Code Authentication
$env:ANTHROPIC_API_KEY = "sk-ant-..."

# Claude Code Configuration
$env:CLAUDE_CODE_SHELL = "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe"
$env:CLAUDE_CODE_TMPDIR = "D:\Temp\Claude"

# Optional: Debug
# $env:CLAUDE_DEBUG = "1"
```

---

## Sources

- [Claude Code Settings Documentation](https://code.claude.com/docs/en/settings.md)
- [Claude Code Changelog v2.0.65-v2.1.9](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Anthropic API Documentation](https://docs.anthropic.com/)

---

*This tutorial is part of the Claude Code documentation series.*
*Version 2.1.x | Last Updated: January 2026*
