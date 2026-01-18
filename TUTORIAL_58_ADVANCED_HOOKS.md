# Tutorial 58: Advanced Hooks Patterns

> **Master hook events, return values, and sophisticated automation workflows**

## Table of Contents

1. [Overview](#overview)
2. [Hook Architecture Deep Dive](#hook-architecture-deep-dive)
3. [Hook Events Reference](#hook-events-reference)
4. [Return Values and Control Flow](#return-values-and-control-flow)
5. [PreToolUse Advanced Patterns](#pretooluse-advanced-patterns)
6. [PostToolUse Patterns](#posttooluse-patterns)
7. [Session Lifecycle Hooks](#session-lifecycle-hooks)
8. [Notification Hooks](#notification-hooks)
9. [Performance Optimization](#performance-optimization)
10. [Error Handling](#error-handling)
11. [Real-World Recipes](#real-world-recipes)
12. [Debugging Hooks](#debugging-hooks)
13. [Quick Reference](#quick-reference)

---

## Overview

This tutorial covers advanced hook patterns beyond the basics in Tutorial 01. You'll learn about sophisticated return values, conditional execution, performance optimization, and complex automation workflows.

### Prerequisites

- Familiarity with basic hooks (see Tutorial 01)
- Understanding of JSON configuration
- Shell scripting knowledge

### What You'll Learn

- All hook events and their use cases
- Return value patterns for controlling Claude's behavior
- The new `additionalContext` return value
- `once: true` for one-time hooks
- Frontmatter hooks in skills/commands
- Timeout configuration and performance tuning

---

## Hook Architecture Deep Dive

### How Hooks Execute

```
┌─────────────────────────────────────────────────────────────────┐
│                    Hook Execution Pipeline                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Event Triggered                                                 │
│       │                                                          │
│       ▼                                                          │
│  ┌─────────────────┐                                            │
│  │ Find Matching   │ Check event type, matcher patterns         │
│  │ Hooks           │                                            │
│  └────────┬────────┘                                            │
│           │                                                      │
│           ▼                                                      │
│  ┌─────────────────┐                                            │
│  │ Execute Hooks   │ Sequential, with timeout (10 min default)  │
│  │ (in order)      │                                            │
│  └────────┬────────┘                                            │
│           │                                                      │
│           ▼                                                      │
│  ┌─────────────────┐                                            │
│  │ Process Return  │ Parse stdout as JSON, apply actions        │
│  │ Values          │                                            │
│  └────────┬────────┘                                            │
│           │                                                      │
│           ▼                                                      │
│  Continue or Block Based on Return                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Hook Configuration Structure

```json
{
  "hooks": {
    "EVENT_TYPE": [
      {
        "matcher": "OPTIONAL_PATTERN",
        "command": "SHELL_COMMAND",
        "timeout": 30000,
        "once": false
      }
    ]
  }
}
```

### Environment Variables Available to Hooks

| Variable | Description | Events |
|----------|-------------|--------|
| `$CLAUDE_EVENT` | Event type name | All |
| `$CLAUDE_TOOL_NAME` | Tool being invoked | PreToolUse, PostToolUse |
| `$CLAUDE_TOOL_INPUT` | JSON tool parameters | PreToolUse, PostToolUse |
| `$CLAUDE_TOOL_OUTPUT` | Tool execution result | PostToolUse |
| `$CLAUDE_SESSION_ID` | Current session ID | All |
| `$CLAUDE_WORKING_DIR` | Working directory | All |
| `$CLAUDE_MODEL` | Current model | All |

---

## Hook Events Reference

### Complete Event List

| Event | Triggers | Matcher Target | Use Cases |
|-------|----------|----------------|-----------|
| `PreToolUse` | Before any tool executes | Tool name | Validation, logging, blocking |
| `PostToolUse` | After tool completes | Tool name | Logging, notifications, cleanup |
| `Notification` | Claude posts notification | Notification type | Alerts, integrations |
| `Stop` | Conversation ends | - | Cleanup, reports |
| `SessionStart` | Session begins | - | Setup, initialization |
| `SessionStop` | Session ends | - | Cleanup, logging |
| `Setup` | `--init` flag used | - | Project initialization |

### Event Timing

```
Session Timeline:
────────────────────────────────────────────────────────────────────►

│ SessionStart    │    Conversation Activity    │ SessionStop │
│                 │                              │             │
│  ┌──────────────┼──────────────────────────────┼─────────────┤
│  │              │                              │             │
│  │              │  PreToolUse ──► Tool ──►     │             │
│  │              │       PostToolUse            │             │
│  │              │                              │             │
│  │              │  Notification (async)        │             │
│  │              │                              │             │
│  │              │  Stop (end of turn)          │             │
│  └──────────────┼──────────────────────────────┼─────────────┤
│                 │                              │             │
│ Setup (--init)  │                              │             │
```

---

## Return Values and Control Flow

### PreToolUse Return Values

Hooks can return JSON to control Claude's behavior:

#### Block Execution

```json
{"decision": "block", "reason": "This operation is not allowed"}
```

Result: Tool is blocked, Claude sees the reason.

#### Allow with Feedback

```json
{"decision": "allow", "reason": "Validated successfully"}
```

Result: Tool proceeds, Claude sees validation message.

#### Provide Additional Context (v2.1.9)

```json
{
  "decision": "allow",
  "additionalContext": "Note: This file was last modified 2 hours ago by user@example.com"
}
```

Result: Tool proceeds, Claude receives the additional context to inform its next steps.

#### Modify Tool Input (v2.1.9)

```json
{
  "decision": "allow",
  "toolInput": {
    "command": "npm test -- --coverage"
  }
}
```

Result: Tool executes with modified parameters.

### PostToolUse Return Values

```json
{
  "additionalContext": "Performance note: This query took 2.3s"
}
```

Result: Claude receives the context after seeing tool output.

### Stop Hook Return Values

```json
{
  "additionalContext": "Session summary: 15 files modified, 3 tests added"
}
```

Result: Context available for any follow-up summaries.

---

## PreToolUse Advanced Patterns

### Pattern 1: Conditional Blocking

Block based on complex conditions:

```bash
#!/bin/bash
# ~/.claude/hooks/validate-bash.sh

TOOL_INPUT="$CLAUDE_TOOL_INPUT"
COMMAND=$(echo "$TOOL_INPUT" | jq -r '.command')

# Block destructive commands
if echo "$COMMAND" | grep -qE "rm -rf|drop table|truncate"; then
  echo '{"decision": "block", "reason": "Destructive command detected. Please confirm manually."}'
  exit 0
fi

# Block if outside working hours (for production)
HOUR=$(date +%H)
if [[ "$COMMAND" == *"production"* ]] && (( HOUR < 9 || HOUR > 17 )); then
  echo '{"decision": "block", "reason": "Production commands blocked outside business hours (9-17)"}'
  exit 0
fi

# Allow with context about what's happening
echo '{"decision": "allow"}'
```

Configuration:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "command": "~/.claude/hooks/validate-bash.sh"
      }
    ]
  }
}
```

### Pattern 2: Enriching Context with additionalContext

Provide Claude with relevant information before tool execution:

```bash
#!/bin/bash
# ~/.claude/hooks/enrich-file-context.sh

TOOL_INPUT="$CLAUDE_TOOL_INPUT"
FILE_PATH=$(echo "$TOOL_INPUT" | jq -r '.file_path // .path // empty')

if [[ -n "$FILE_PATH" ]] && [[ -f "$FILE_PATH" ]]; then
  # Get file metadata
  LAST_MODIFIED=$(stat -c %y "$FILE_PATH" 2>/dev/null || stat -f %Sm "$FILE_PATH" 2>/dev/null)
  FILE_SIZE=$(stat -c %s "$FILE_PATH" 2>/dev/null || stat -f %z "$FILE_PATH" 2>/dev/null)
  LINE_COUNT=$(wc -l < "$FILE_PATH")

  # Check git status
  GIT_STATUS=""
  if git rev-parse --is-inside-work-tree &>/dev/null; then
    if git diff --quiet "$FILE_PATH" 2>/dev/null; then
      GIT_STATUS="clean"
    else
      GIT_STATUS="has uncommitted changes"
    fi
  fi

  cat << EOF
{
  "decision": "allow",
  "additionalContext": "File info: ${LINE_COUNT} lines, ${FILE_SIZE} bytes, modified ${LAST_MODIFIED}. Git: ${GIT_STATUS}"
}
EOF
else
  echo '{"decision": "allow"}'
fi
```

### Pattern 3: Rate Limiting

Prevent too many operations in a time window:

```bash
#!/bin/bash
# ~/.claude/hooks/rate-limit.sh

TOOL_NAME="$CLAUDE_TOOL_NAME"
RATE_FILE="/tmp/claude-rate-${TOOL_NAME}"
MAX_PER_MINUTE=10

# Get current count
CURRENT_MINUTE=$(date +%Y%m%d%H%M)
if [[ -f "$RATE_FILE" ]]; then
  read STORED_MINUTE STORED_COUNT < "$RATE_FILE"
  if [[ "$STORED_MINUTE" == "$CURRENT_MINUTE" ]]; then
    COUNT=$STORED_COUNT
  else
    COUNT=0
  fi
else
  COUNT=0
fi

# Check rate limit
if (( COUNT >= MAX_PER_MINUTE )); then
  echo "{\"decision\": \"block\", \"reason\": \"Rate limit exceeded: max ${MAX_PER_MINUTE} ${TOOL_NAME} calls per minute\"}"
  exit 0
fi

# Update count
echo "$CURRENT_MINUTE $((COUNT + 1))" > "$RATE_FILE"
echo '{"decision": "allow"}'
```

### Pattern 4: Input Modification

Transform tool inputs before execution:

```bash
#!/bin/bash
# ~/.claude/hooks/transform-paths.sh

TOOL_INPUT="$CLAUDE_TOOL_INPUT"

# Expand ~ to absolute path
MODIFIED_INPUT=$(echo "$TOOL_INPUT" | jq '
  if .file_path then
    .file_path |= gsub("^~"; env.HOME)
  else
    .
  end
')

echo "{\"decision\": \"allow\", \"toolInput\": $MODIFIED_INPUT}"
```

---

## PostToolUse Patterns

### Pattern 1: Execution Logging

```bash
#!/bin/bash
# ~/.claude/hooks/log-execution.sh

LOG_FILE="$HOME/.claude/logs/tools-$(date +%Y%m%d).log"
mkdir -p "$(dirname "$LOG_FILE")"

cat >> "$LOG_FILE" << EOF
---
Timestamp: $(date -Iseconds)
Session: $CLAUDE_SESSION_ID
Tool: $CLAUDE_TOOL_NAME
Input: $CLAUDE_TOOL_INPUT
Output Length: $(echo "$CLAUDE_TOOL_OUTPUT" | wc -c) bytes
EOF

# No JSON output needed for logging
```

### Pattern 2: Output Analysis

```bash
#!/bin/bash
# ~/.claude/hooks/analyze-output.sh

TOOL_OUTPUT="$CLAUDE_TOOL_OUTPUT"

# Check for common issues in output
WARNINGS=""

if echo "$TOOL_OUTPUT" | grep -qi "deprecated"; then
  WARNINGS="${WARNINGS}Contains deprecation warnings. "
fi

if echo "$TOOL_OUTPUT" | grep -qi "error"; then
  WARNINGS="${WARNINGS}Contains error messages. "
fi

if echo "$TOOL_OUTPUT" | grep -qi "warning"; then
  WARNINGS="${WARNINGS}Contains warnings. "
fi

if [[ -n "$WARNINGS" ]]; then
  echo "{\"additionalContext\": \"⚠️ ${WARNINGS}\"}"
fi
```

### Pattern 3: Auto-Notification on Errors

```bash
#!/bin/bash
# ~/.claude/hooks/notify-errors.sh

TOOL_OUTPUT="$CLAUDE_TOOL_OUTPUT"
TOOL_NAME="$CLAUDE_TOOL_NAME"

# Check for failure indicators
if echo "$TOOL_OUTPUT" | grep -qiE "failed|error|exception|fatal"; then
  # Send notification (example: macOS)
  osascript -e "display notification \"$TOOL_NAME encountered an error\" with title \"Claude Code\""

  # Or send to Slack
  # curl -X POST -H 'Content-type: application/json' \
  #   --data "{\"text\":\"Claude tool $TOOL_NAME failed\"}" \
  #   "$SLACK_WEBHOOK_URL"
fi
```

---

## Session Lifecycle Hooks

### SessionStart Hook

Initialize environment when session begins:

```bash
#!/bin/bash
# ~/.claude/hooks/session-start.sh

# Create session workspace
SESSION_DIR="/tmp/claude-sessions/$CLAUDE_SESSION_ID"
mkdir -p "$SESSION_DIR"

# Initialize session log
echo "Session started: $(date -Iseconds)" > "$SESSION_DIR/session.log"

# Check prerequisites
WARNINGS=""
if ! command -v node &>/dev/null; then
  WARNINGS="${WARNINGS}Node.js not found. "
fi
if ! command -v git &>/dev/null; then
  WARNINGS="${WARNINGS}Git not found. "
fi

if [[ -n "$WARNINGS" ]]; then
  echo "{\"additionalContext\": \"⚠️ Missing tools: ${WARNINGS}\"}"
fi
```

Configuration:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "command": "~/.claude/hooks/session-start.sh"
      }
    ]
  }
}
```

### SessionStop Hook

Cleanup when session ends:

```bash
#!/bin/bash
# ~/.claude/hooks/session-stop.sh

SESSION_DIR="/tmp/claude-sessions/$CLAUDE_SESSION_ID"

if [[ -d "$SESSION_DIR" ]]; then
  # Archive session logs
  tar -czf "$HOME/.claude/archives/session-${CLAUDE_SESSION_ID}.tar.gz" \
    -C "/tmp/claude-sessions" "$CLAUDE_SESSION_ID"

  # Cleanup
  rm -rf "$SESSION_DIR"
fi

# Report session stats
echo "{\"additionalContext\": \"Session archived to ~/.claude/archives/\"}"
```

### Setup Hook (v2.1.10)

Run during `--init` for project setup:

```bash
#!/bin/bash
# ~/.claude/hooks/project-setup.sh

echo "Initializing project for Claude Code..."

# Create .claude directory structure
mkdir -p .claude/skills .claude/commands

# Copy templates
if [[ -d "$HOME/.claude/templates" ]]; then
  cp -r "$HOME/.claude/templates/"* .claude/
fi

# Install dependencies if package.json exists
if [[ -f "package.json" ]]; then
  npm install
fi

echo "{\"additionalContext\": \"Project initialized with Claude Code configuration\"}"
```

Configuration:

```json
{
  "hooks": {
    "Setup": [
      {
        "command": "~/.claude/hooks/project-setup.sh"
      }
    ]
  }
}
```

---

## Notification Hooks

### Notification Types

| Type | When | Content |
|------|------|---------|
| `user` | Claude wants attention | Message to user |
| `progress` | Long operation status | Progress update |
| `error` | Something went wrong | Error details |

### Desktop Notifications

```bash
#!/bin/bash
# ~/.claude/hooks/desktop-notify.sh

NOTIFICATION_TYPE=$(echo "$CLAUDE_TOOL_INPUT" | jq -r '.type // "info"')
MESSAGE=$(echo "$CLAUDE_TOOL_INPUT" | jq -r '.message // "Notification from Claude"')

case "$(uname)" in
  Darwin)
    osascript -e "display notification \"$MESSAGE\" with title \"Claude Code\" sound name \"Ping\""
    ;;
  Linux)
    notify-send "Claude Code" "$MESSAGE"
    ;;
  MINGW*|MSYS*|CYGWIN*)
    powershell -Command "[Windows.UI.Notifications.ToastNotificationManager, Windows.UI.Notifications, ContentType = WindowsRuntime] | Out-Null; \$xml = [Windows.UI.Notifications.ToastNotificationManager]::GetTemplateContent([Windows.UI.Notifications.ToastTemplateType]::ToastText02); \$xml.GetElementsByTagName('text')[0].AppendChild(\$xml.CreateTextNode('Claude Code')); \$xml.GetElementsByTagName('text')[1].AppendChild(\$xml.CreateTextNode('$MESSAGE')); [Windows.UI.Notifications.ToastNotificationManager]::CreateToastNotifier('Claude Code').Show(\$xml)"
    ;;
esac
```

Configuration:

```json
{
  "hooks": {
    "Notification": [
      {
        "command": "~/.claude/hooks/desktop-notify.sh"
      }
    ]
  }
}
```

---

## Performance Optimization

### The `once: true` Option (v2.1.0)

Run hook only once per session:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "command": "expensive-setup-script.sh",
        "once": true
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "command": "check-env-once.sh",
        "once": true
      }
    ]
  }
}
```

Use cases:
- One-time environment validation
- First-run setup scripts
- Initial dependency checks

### Timeout Configuration (v2.1.3)

Hooks have a 10-minute timeout by default. Override per-hook:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "command": "quick-validation.sh",
        "timeout": 5000
      },
      {
        "matcher": "Write",
        "command": "backup-before-write.sh",
        "timeout": 30000
      }
    ]
  }
}
```

**Timeout guidelines:**
| Hook Type | Recommended Timeout |
|-----------|-------------------|
| Validation | 5-10 seconds |
| Logging | 5 seconds |
| Network calls | 30 seconds |
| File operations | 15 seconds |
| Complex analysis | 60 seconds |

### Async Hook Pattern

For slow operations that shouldn't block:

```bash
#!/bin/bash
# Run slow operation in background, don't block Claude

# Quick validation that returns immediately
echo '{"decision": "allow"}'

# Background the slow work
{
  # Slow operations here
  sleep 5
  echo "Background work done" >> /tmp/hook-background.log
} &
```

---

## Error Handling

### Graceful Degradation

```bash
#!/bin/bash
# ~/.claude/hooks/safe-hook.sh

# Always succeed, even if our logic fails
trap 'echo "{\"decision\": \"allow\"}"' ERR

# Your logic here
RESULT=$(some-command-that-might-fail 2>&1) || true

if [[ -n "$RESULT" ]]; then
  echo "{\"decision\": \"allow\", \"additionalContext\": \"$RESULT\"}"
else
  echo '{"decision": "allow"}'
fi
```

### Logging Hook Errors

```bash
#!/bin/bash
# ~/.claude/hooks/logged-hook.sh

LOG_FILE="$HOME/.claude/logs/hook-errors.log"
mkdir -p "$(dirname "$LOG_FILE")"

{
  # Wrap in error logging
  set -e

  # Your hook logic here
  TOOL_INPUT="$CLAUDE_TOOL_INPUT"
  # ... processing ...

  echo '{"decision": "allow"}'
} 2>> "$LOG_FILE" || {
  # On any error, log it but allow the tool
  echo "[$(date -Iseconds)] Hook error in $0: $?" >> "$LOG_FILE"
  echo '{"decision": "allow"}'
}
```

---

## Real-World Recipes

### Recipe 1: Git Safety Hooks

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "command": "~/.claude/hooks/git-safety.sh"
      }
    ]
  }
}
```

```bash
#!/bin/bash
# ~/.claude/hooks/git-safety.sh

COMMAND=$(echo "$CLAUDE_TOOL_INPUT" | jq -r '.command')

# Block force pushes
if echo "$COMMAND" | grep -qE "git push.*(-f|--force)"; then
  echo '{"decision": "block", "reason": "Force push blocked. Use regular push or request manual override."}'
  exit 0
fi

# Warn about main branch changes
if echo "$COMMAND" | grep -qE "git (checkout|switch) (main|master)"; then
  echo '{"decision": "allow", "additionalContext": "⚠️ Switching to protected branch. Be careful with direct commits."}'
  exit 0
fi

# Block history rewrites
if echo "$COMMAND" | grep -qE "git (rebase|reset --hard|filter-branch)"; then
  echo '{"decision": "block", "reason": "History rewriting command blocked. This could lose work."}'
  exit 0
fi

echo '{"decision": "allow"}'
```

### Recipe 2: File Backup Before Edit

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit",
        "command": "~/.claude/hooks/backup-before-edit.sh"
      },
      {
        "matcher": "Write",
        "command": "~/.claude/hooks/backup-before-edit.sh"
      }
    ]
  }
}
```

```bash
#!/bin/bash
# ~/.claude/hooks/backup-before-edit.sh

FILE_PATH=$(echo "$CLAUDE_TOOL_INPUT" | jq -r '.file_path')
BACKUP_DIR="$HOME/.claude/backups/$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"

if [[ -f "$FILE_PATH" ]]; then
  BACKUP_NAME="$(basename "$FILE_PATH").$(date +%H%M%S).bak"
  cp "$FILE_PATH" "$BACKUP_DIR/$BACKUP_NAME"
  echo "{\"decision\": \"allow\", \"additionalContext\": \"Backup created: $BACKUP_DIR/$BACKUP_NAME\"}"
else
  echo '{"decision": "allow"}'
fi
```

### Recipe 3: Cost Tracking

```json
{
  "hooks": {
    "Stop": [
      {
        "command": "~/.claude/hooks/track-cost.sh"
      }
    ]
  }
}
```

```bash
#!/bin/bash
# ~/.claude/hooks/track-cost.sh

COST_FILE="$HOME/.claude/costs/$(date +%Y%m).csv"
mkdir -p "$(dirname "$COST_FILE")"

# Initialize CSV if needed
if [[ ! -f "$COST_FILE" ]]; then
  echo "date,session_id,model" > "$COST_FILE"
fi

# Log session
echo "$(date -Iseconds),$CLAUDE_SESSION_ID,$CLAUDE_MODEL" >> "$COST_FILE"
```

### Recipe 4: Frontmatter Hooks in Skills (v2.1.0)

Define hooks directly in skill files:

```yaml
---
name: database-query
description: Execute database queries safely
hooks:
  PreToolUse:
    - matcher: Bash
      command: "~/.claude/hooks/validate-sql.sh"
---

# Database Query Skill

When executing database queries:
1. Validate the query syntax
2. Check for dangerous operations
3. Execute with timeout
```

---

## Debugging Hooks

### Enable Verbose Mode

```bash
claude --verbose
```

Shows hook execution details:
- Which hooks are matched
- Hook command and arguments
- Hook return values
- Timing information

### Test Hook Manually

```bash
# Set environment variables
export CLAUDE_TOOL_NAME="Bash"
export CLAUDE_TOOL_INPUT='{"command": "npm test"}'
export CLAUDE_SESSION_ID="test-123"

# Run hook
~/.claude/hooks/your-hook.sh
```

### Debug Output in Hook

```bash
#!/bin/bash
# Add debug output to stderr (doesn't affect JSON return)

echo "DEBUG: Tool=$CLAUDE_TOOL_NAME" >&2
echo "DEBUG: Input=$CLAUDE_TOOL_INPUT" >&2

# Your logic...

echo '{"decision": "allow"}'
```

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Hook not running | Matcher doesn't match | Check tool name exactly |
| Invalid JSON | Shell expansion issues | Use `cat << 'EOF'` or escape properly |
| Timeout | Hook too slow | Add timeout or use `once: true` |
| Permission denied | Script not executable | `chmod +x hook.sh` |
| Path not found | ~ not expanded | Use `$HOME` or full path |

---

## Quick Reference

### Return Value Cheat Sheet

```json
// Block tool execution
{"decision": "block", "reason": "Explanation for Claude"}

// Allow with message
{"decision": "allow", "reason": "Validation passed"}

// Allow with context for Claude
{"decision": "allow", "additionalContext": "Extra info for Claude"}

// Modify tool input
{"decision": "allow", "toolInput": {"modified": "parameters"}}

// PostToolUse context
{"additionalContext": "Info about the output"}
```

### Configuration Options

```json
{
  "hooks": {
    "EVENT_NAME": [
      {
        "matcher": "ToolName",    // Optional: filter by tool
        "command": "script.sh",   // Required: command to run
        "timeout": 30000,         // Optional: ms (default: 600000)
        "once": true              // Optional: run once per session
      }
    ]
  }
}
```

### Event Quick Reference

| Event | Matcher | Return Values |
|-------|---------|---------------|
| PreToolUse | Tool name | decision, reason, additionalContext, toolInput |
| PostToolUse | Tool name | additionalContext |
| Notification | Type | additionalContext |
| Stop | - | additionalContext |
| SessionStart | - | additionalContext |
| SessionStop | - | additionalContext |
| Setup | - | additionalContext |

### Common Patterns

```bash
# Block with reason
echo '{"decision": "block", "reason": "Not allowed"}'

# Allow silently
echo '{"decision": "allow"}'

# Add context
echo '{"additionalContext": "FYI: something relevant"}'

# Parse tool input
COMMAND=$(echo "$CLAUDE_TOOL_INPUT" | jq -r '.command')

# Parse file path
FILE=$(echo "$CLAUDE_TOOL_INPUT" | jq -r '.file_path // .path')
```

---

## Sources

- [Claude Code Hooks Documentation](https://code.claude.com/docs/en/hooks.md)
- [Claude Code Changelog v2.1.0-v2.1.10](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

---

*This tutorial is part of the Claude Code documentation series.*
*Version 2.1.x | Last Updated: January 2026*
