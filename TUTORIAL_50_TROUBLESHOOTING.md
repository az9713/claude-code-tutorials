# Tutorial 50: Troubleshooting

> **Common issues and solutions for Claude Code**

## Overview

Solutions for common Claude Code issues.

---

## Part 1: Installation Issues

### Command Not Found

```bash
# Check if installed
npm list -g @anthropic-ai/claude-code

# Reinstall
npm install -g @anthropic-ai/claude-code
```

### Permission Errors

```bash
# Fix npm permissions
npm config set prefix ~/.npm-global
export PATH=~/.npm-global/bin:$PATH
```

---

## Part 2: Authentication Issues

### API Key Invalid

```bash
# Check key
echo $ANTHROPIC_API_KEY

# Set key
export ANTHROPIC_API_KEY=YOUR_KEY
```

### Login Issues

```bash
# Re-authenticate
claude logout
claude login
```

---

## Part 3: IDE Integration

### VS Code Icon Missing

1. Check VS Code version (1.98.0+)
2. Open a file (icon needs file open)
3. Restart VS Code
4. Disable conflicting extensions

### JetBrains Not Detected

1. Install plugin in remote host
2. Restart IDE completely
3. Run from integrated terminal

---

## Part 4: Performance Issues

### Slow Responses

```bash
# Compact context
/compact

# Clear history
/clear

# Use faster model
/model haiku
```

### High Token Usage

```bash
# Check costs
/cost

# Enable earlier compaction
export CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=50
```

---

## Part 5: Common Errors

### Rate Limiting

- Wait for quota reset
- Reduce parallel tasks
- Check team rate limits

### Context Too Long

```bash
/compact
# Or start fresh
/clear
```

### Tool Errors

Check permissions:
```bash
/config
# Review allowed tools
```

---

## Part 6: Diagnostics

### System Information

```bash
claude doctor
```

### Debug Mode

```bash
claude --debug
```

### Log Location

- macOS: `~/.claude/logs/`
- Windows: `%USERPROFILE%\.claude\logs\`
- Linux: `~/.claude/logs/`

---

## Quick Reference

### Common Fixes

| Issue | Solution |
|-------|----------|
| Not installed | `npm install -g @anthropic-ai/claude-code` |
| Auth failed | `claude logout && claude login` |
| Slow | `/compact` or `/clear` |
| Rate limited | Wait or reduce usage |

### Diagnostic Commands

| Command | Purpose |
|---------|---------|
| `claude doctor` | System info |
| `claude --version` | Version check |
| `claude --debug` | Debug mode |

---

## Sources

- [Troubleshooting](https://code.claude.com/docs/en/troubleshooting.md)
- [Quickstart](https://code.claude.com/docs/en/quickstart.md)
