# Tutorial 65: Diagnostics & Health Checks

> **Self-service troubleshooting with /doctor, debugging tools, and health monitoring**

## Table of Contents

1. [Overview](#overview)
2. [The /doctor Command](#the-doctor-command)
3. [Configuration Diagnostics](#configuration-diagnostics)
4. [Permission Diagnostics](#permission-diagnostics)
5. [MCP Server Diagnostics](#mcp-server-diagnostics)
6. [Network Diagnostics](#network-diagnostics)
7. [Performance Diagnostics](#performance-diagnostics)
8. [Common Issues and Solutions](#common-issues-and-solutions)
9. [Debug Mode](#debug-mode)
10. [Health Monitoring Scripts](#health-monitoring-scripts)
11. [Quick Reference](#quick-reference)

---

## Overview

Claude Code provides built-in diagnostic tools to help identify and resolve issues. This tutorial covers comprehensive troubleshooting workflows.

### When to Use Diagnostics

- Installation or upgrade problems
- Configuration not working as expected
- Permission issues
- MCP server connection failures
- Slow performance
- Unexpected behavior

### Diagnostic Tools

| Tool | Purpose |
|------|---------|
| `/doctor` | Comprehensive health check |
| `/config` | View current configuration |
| `/permissions` | Search permission rules |
| `/stats` | Usage statistics |
| `/cost` | Token and cost tracking |
| `--verbose` | Detailed logging |
| `--debug` | Debug output |

---

## The /doctor Command

### Basic Usage

```bash
/doctor
```

### Output Sections

```
Claude Code Health Check
========================

✓ Version: 2.1.12 (up to date)
✓ Node.js: v20.10.0
✓ Platform: darwin arm64

Configuration:
✓ User settings: ~/.claude/settings.json (valid)
✓ Project settings: .claude/settings.json (valid)
⚠ Managed settings: Not configured

Authentication:
✓ API key: Configured (ANTHROPIC_API_KEY)
✓ API endpoint: api.anthropic.com (reachable)

MCP Servers:
✓ filesystem: Connected (3 tools)
⚠ database: Not running
✗ custom-server: Connection failed

Permissions:
✓ Configuration valid
⚠ Unreachable rules detected (2)

Hooks:
✓ PreToolUse: 2 hooks configured
✓ PostToolUse: 1 hook configured

Updates:
⚠ New version available: 2.1.13
```

### Understanding Status Icons

| Icon | Meaning |
|------|---------|
| ✓ | Healthy / OK |
| ⚠ | Warning (non-critical) |
| ✗ | Error (needs attention) |
| ? | Unknown / Unable to check |

### /doctor Sub-Commands (v2.1.6)

```bash
/doctor updates      # Check for updates only
/doctor permissions  # Permission analysis
/doctor mcp          # MCP server status
/doctor config       # Configuration validation
```

---

## Configuration Diagnostics

### View Current Configuration

```bash
/config
```

Shows merged configuration from all sources.

### Search Configuration (v2.1.6)

```bash
/config search model
/config search permissions
/config search mcp
```

### Validate Configuration

```bash
# Check settings file syntax
cat ~/.claude/settings.json | jq .

# Check project settings
cat .claude/settings.json | jq .
```

### Configuration Source Tracing

Identify where a setting comes from:

```bash
/config show model

# Output:
# model: claude-sonnet-4-20250514
# Source: User settings (~/.claude/settings.json)
```

### Common Configuration Issues

#### Invalid JSON

```
Error: Failed to parse settings.json

Solution: Validate JSON syntax
$ cat ~/.claude/settings.json | jq .
```

#### Missing Required Fields

```
Warning: API key not configured

Solution: Set environment variable
$ export ANTHROPIC_API_KEY="sk-ant-..."
```

#### Conflicting Settings

```
Warning: Project settings override user settings for "model"

Info: This is expected behavior (project takes precedence)
```

---

## Permission Diagnostics

### Search Permissions (v2.0.67)

```bash
/permissions npm
/permissions git push
/permissions mcp__github
```

### Unreachable Rule Detection (v2.1.3)

```bash
/doctor permissions

# Output:
# Unreachable Rules Detected:
#
# 1. Bash.allow: "git push" is shadowed by Bash.deny: "git *"
#    Location: ~/.claude/settings.json
#    Recommendation: Remove allow rule or narrow deny pattern
#
# 2. Read.allow: ".env.example" blocked by Read.deny: ".env*"
#    Location: .claude/settings.json
#    Recommendation: Add explicit exception or rename file
```

### Testing Permission Rules

```bash
# Check what happens for a specific command
/permissions "npm test"

# Output:
# Command: npm test
# Decision: ALLOW
# Matched rule: Bash.allow: "npm *"
# Source: User settings
```

### Permission Precedence Visualization

```bash
/permissions --verbose "git push"

# Output:
# Evaluating: git push
#
# Checking deny rules:
#   - "git push --force*" (no match)
#   - "*sudo*" (no match)
#
# Checking allow rules:
#   - "git *" (MATCH)
#
# Decision: ALLOW (matched allow rule)
```

---

## MCP Server Diagnostics

### List MCP Servers

```bash
claude mcp list
```

### Check Server Status

```bash
claude mcp status

# Output:
# MCP Server Status
# =================
#
# filesystem:
#   Status: Running
#   Tools: read_file, write_file, list_directory
#   Last ping: 2ms
#
# database:
#   Status: Stopped
#   Error: Connection refused
#
# github:
#   Status: Running
#   Tools: 15 tools available
#   Last ping: 45ms
```

### Test Server Connection

```bash
claude mcp test filesystem

# Output:
# Testing mcp__filesystem...
# ✓ Server process started
# ✓ Initialization successful
# ✓ Tool discovery: 3 tools
# ✓ Test call: list_directory (passed)
```

### MCP Server Logs

```bash
# View server logs
claude mcp logs filesystem

# Tail logs in real-time
claude mcp logs -f filesystem
```

### Common MCP Issues

#### Server Not Starting

```
Error: mcp__custom__tool: Server failed to start

Checks:
1. Command exists: which mcp-custom-server
2. Permissions: ls -la $(which mcp-custom-server)
3. Dependencies: mcp-custom-server --version
4. Config: cat ~/.claude/mcp.json | jq '.servers.custom'
```

#### Tool Discovery Failed

```
Warning: Server started but no tools discovered

Checks:
1. Server implements tools/list method
2. No initialization errors in logs
3. Tool definitions are valid JSON
```

---

## Network Diagnostics

### API Connectivity

```bash
# Test API endpoint
curl -I https://api.anthropic.com/v1/health

# Check with proxy
curl -I --proxy $HTTP_PROXY https://api.anthropic.com/v1/health
```

### DNS Resolution

```bash
# Check DNS
nslookup api.anthropic.com
dig api.anthropic.com
```

### SSL/TLS Issues

```bash
# Check certificate
openssl s_client -connect api.anthropic.com:443 -servername api.anthropic.com

# Test with curl
curl -v https://api.anthropic.com/v1/health 2>&1 | grep -E "SSL|certificate"
```

### Proxy Configuration

```bash
# Check proxy variables
echo "HTTP_PROXY: $HTTP_PROXY"
echo "HTTPS_PROXY: $HTTPS_PROXY"
echo "NO_PROXY: $NO_PROXY"

# Test through proxy
curl --proxy $HTTP_PROXY https://api.anthropic.com/v1/health
```

---

## Performance Diagnostics

### Token Usage

```bash
/cost

# Output:
# Session Token Usage
# ===================
# Input tokens: 45,230
# Output tokens: 12,450
# Total: 57,680
#
# Tool definitions: 8,450 (14.6%)
# Conversation: 28,000 (48.5%)
# File contents: 21,230 (36.8%)
```

### Statistics (v2.1.6)

```bash
/stats

# Output:
# Session Statistics
# ==================
# Duration: 1h 23m
# Turns: 45
# Tools used: 156
# Files read: 34
# Files modified: 12
```

### Date Filtering for Stats (v2.1.6)

```bash
/stats today
/stats week
/stats month
/stats 2026-01-15
```

### Context Window Analysis

```bash
/doctor context

# Output:
# Context Window Analysis
# =======================
# Total capacity: 200,000 tokens
# Current usage: 156,000 tokens (78%)
#
# Breakdown:
# - System prompt: 5,200 (2.6%)
# - Tool definitions: 12,400 (6.2%)
# - Conversation history: 98,000 (49%)
# - File contents: 40,400 (20.2%)
#
# Recommendation: Consider using /compact to reduce context
```

---

## Common Issues and Solutions

### Issue: "Command not found: claude"

```bash
# Check PATH
echo $PATH | tr ':' '\n' | grep -E "npm|node"

# Find installation
npm list -g @anthropic-ai/claude-code

# Add to PATH if needed
export PATH="$PATH:$(npm root -g)/../bin"
```

### Issue: "API key invalid"

```bash
# Check environment variable
echo $ANTHROPIC_API_KEY | head -c 20

# Verify key format
# Should start with: sk-ant-

# Test key
curl https://api.anthropic.com/v1/messages \
  -H "anthropic-version: 2024-01-01" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -d '{"model":"claude-sonnet-4-20250514","max_tokens":10,"messages":[{"role":"user","content":"Hi"}]}'
```

### Issue: "Permission denied"

```bash
# Check hook permissions
ls -la ~/.claude/hooks/

# Make executable
chmod +x ~/.claude/hooks/*.sh

# Check script shebang
head -1 ~/.claude/hooks/pre-tool.sh
# Should be: #!/bin/bash or #!/usr/bin/env bash
```

### Issue: "Settings not loading"

```bash
# Validate JSON
cat ~/.claude/settings.json | jq .

# Check file permissions
ls -la ~/.claude/settings.json

# Verify location
echo $HOME/.claude/settings.json
```

### Issue: "MCP server timeout"

```bash
# Check server process
ps aux | grep mcp

# Increase timeout in settings
{
  "mcp": {
    "servers": {
      "slow-server": {
        "timeout": 60000
      }
    }
  }
}
```

### Issue: "Context limit reached"

```bash
# Compact context
/compact

# Or start fresh
/clear

# Check what's using context
/cost
```

---

## Debug Mode

### Enable Debug Output

```bash
# Environment variable
export CLAUDE_DEBUG=1
claude

# Or command line
claude --debug
```

### Verbose Mode

```bash
claude --verbose
```

Shows:
- API requests/responses
- Tool executions
- Hook invocations
- File operations

### Debug Log File

```bash
# Enable log file
export CLAUDE_LOG_FILE=~/.claude/debug.log
claude

# Tail logs
tail -f ~/.claude/debug.log
```

### Debug Specific Components

```bash
# Debug hooks only
export DEBUG=claude:hooks

# Debug MCP only
export DEBUG=claude:mcp

# Debug all
export DEBUG=claude:*
```

---

## Health Monitoring Scripts

### Basic Health Check

```bash
#!/bin/bash
# health-check.sh

ERRORS=0

# Check installation
if ! command -v claude &>/dev/null; then
  echo "ERROR: Claude Code not installed"
  ((ERRORS++))
fi

# Check version
VERSION=$(claude --version 2>/dev/null | grep -oE '[0-9]+\.[0-9]+\.[0-9]+')
if [[ -z "$VERSION" ]]; then
  echo "ERROR: Cannot determine version"
  ((ERRORS++))
fi

# Check API key
if [[ -z "$ANTHROPIC_API_KEY" ]]; then
  echo "ERROR: API key not configured"
  ((ERRORS++))
fi

# Check settings file
if [[ ! -f ~/.claude/settings.json ]]; then
  echo "WARNING: No user settings file"
fi

# Check connectivity
if ! curl -s --max-time 5 https://api.anthropic.com/v1/health &>/dev/null; then
  echo "ERROR: Cannot reach API endpoint"
  ((ERRORS++))
fi

if [[ $ERRORS -eq 0 ]]; then
  echo "OK: All health checks passed"
  exit 0
else
  echo "FAILED: $ERRORS error(s) found"
  exit 1
fi
```

### Monitoring Script

```bash
#!/bin/bash
# monitor-claude.sh

LOG_FILE="/var/log/claude-monitor.log"

while true; do
  TIMESTAMP=$(date -Iseconds)

  # Run health check
  if ./health-check.sh &>/dev/null; then
    STATUS="OK"
  else
    STATUS="FAILED"
  fi

  # Log result
  echo "$TIMESTAMP,$STATUS" >> "$LOG_FILE"

  # Alert on failure
  if [[ "$STATUS" == "FAILED" ]]; then
    # Send alert (customize for your alerting system)
    echo "Claude Code health check failed at $TIMESTAMP" | mail -s "Alert" admin@example.com
  fi

  sleep 300  # Check every 5 minutes
done
```

### Comprehensive Diagnostic Script

```bash
#!/bin/bash
# full-diagnostic.sh

echo "Claude Code Full Diagnostic Report"
echo "==================================="
echo "Generated: $(date)"
echo ""

echo "## System Information"
echo "OS: $(uname -s) $(uname -r)"
echo "Architecture: $(uname -m)"
echo "Shell: $SHELL"
echo ""

echo "## Claude Code"
echo "Version: $(claude --version 2>/dev/null || echo 'Not installed')"
echo "Location: $(which claude 2>/dev/null || echo 'Not found')"
echo ""

echo "## Node.js"
echo "Version: $(node --version 2>/dev/null || echo 'Not installed')"
echo "npm Version: $(npm --version 2>/dev/null || echo 'Not installed')"
echo ""

echo "## Configuration"
echo "User settings: $(test -f ~/.claude/settings.json && echo 'Present' || echo 'Missing')"
echo "Project settings: $(test -f .claude/settings.json && echo 'Present' || echo 'Missing')"
echo ""

echo "## Environment"
echo "API Key: $(test -n "$ANTHROPIC_API_KEY" && echo 'Configured' || echo 'Missing')"
echo "Proxy: ${HTTP_PROXY:-'Not set'}"
echo ""

echo "## Connectivity"
if curl -s --max-time 5 https://api.anthropic.com/v1/health &>/dev/null; then
  echo "API: Reachable"
else
  echo "API: Unreachable"
fi
echo ""

echo "## Recent Errors"
if [[ -f ~/.claude/logs/error.log ]]; then
  tail -20 ~/.claude/logs/error.log
else
  echo "No error log found"
fi
```

---

## Quick Reference

### Diagnostic Commands

| Command | Purpose |
|---------|---------|
| `/doctor` | Comprehensive health check |
| `/doctor updates` | Check for updates |
| `/doctor permissions` | Permission analysis |
| `/doctor mcp` | MCP server status |
| `/config` | View configuration |
| `/config search X` | Search config for X |
| `/permissions X` | Search permissions |
| `/stats` | Usage statistics |
| `/cost` | Token usage |

### CLI Flags

| Flag | Purpose |
|------|---------|
| `--version` | Show version |
| `--verbose` | Verbose output |
| `--debug` | Debug mode |

### Environment Variables

| Variable | Purpose |
|----------|---------|
| `CLAUDE_DEBUG` | Enable debug |
| `CLAUDE_VERBOSE` | Verbose logging |
| `CLAUDE_LOG_FILE` | Log to file |
| `DEBUG` | Node.js debug namespaces |

### Quick Fixes

```bash
# Settings not loading
cat ~/.claude/settings.json | jq .

# Permission issues
chmod +x ~/.claude/hooks/*.sh

# Clear context
/compact

# Reset session
/clear

# Check MCP
claude mcp status

# Full diagnostic
/doctor
```

### Status Meanings

| Icon | Meaning | Action |
|------|---------|--------|
| ✓ | OK | None needed |
| ⚠ | Warning | Review but not critical |
| ✗ | Error | Needs fixing |
| ? | Unknown | Manual check needed |

---

## Sources

- [Claude Code Troubleshooting](https://code.claude.com/docs/en/troubleshooting.md)
- [Claude Code Changelog v2.1.3, v2.1.6](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

---

*This tutorial is part of the Claude Code documentation series.*
*Version 2.1.x | Last Updated: January 2026*
