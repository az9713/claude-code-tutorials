# Tutorial 36: Security Best Practices

> **Secure Claude Code usage in development and production**

## Overview

Comprehensive security guidelines for using Claude Code safely.

---

## Part 1: Permission System

### Permission Modes

| Mode | Behavior |
|------|----------|
| `default` | Ask before actions |
| `acceptEdits` | Auto-accept file changes |
| `plan` | Read-only, no changes |
| `bypassPermissions` | Skip all prompts (dangerous) |

### Configure Permissions

```json
// ~/.claude/settings.json
{
  "permissions": {
    "allow": [
      "Read(~/projects/*)",
      "Search"
    ],
    "deny": [
      "Write(~/)",
      "Bash(rm *)",
      "Bash(sudo *)"
    ]
  }
}
```

---

## Part 2: Sandbox Mode

### Enable Sandboxing

```bash
claude --sandbox
```

### Sandbox Features

- Filesystem isolation
- Network restrictions
- Process isolation

### When to Use

- Untrusted repositories
- Experimental code
- Security-sensitive analysis

---

## Part 3: API Key Security

### Never Commit Keys

```bash
# Bad
ANTHROPIC_API_KEY=YOUR_KEY in code

# Good
export ANTHROPIC_API_KEY=${{ secrets.KEY }}
```

### Use OIDC

For CI/CD, use OIDC instead of long-lived keys:

- AWS Bedrock: IAM role assumption
- Google Vertex: Workload Identity Federation

---

## Part 4: Tool Restrictions

### Limit Bash Commands

```json
{
  "permissions": {
    "deny": [
      "Bash(rm -rf *)",
      "Bash(sudo *)",
      "Bash(curl *)",
      "Bash(wget *)"
    ]
  }
}
```

### Read-Only Mode

```bash
claude --permission-mode plan
```

Only analysis, no modifications.

---

## Part 5: File Access Control

### Restrict Paths

```json
{
  "permissions": {
    "allow": [
      "Read(./src/*)",
      "Write(./src/*)"
    ],
    "deny": [
      "Read(~/.ssh/*)",
      "Read(~/.aws/*)",
      "Write(~/)"
    ]
  }
}
```

### Protect Sensitive Files

```json
{
  "permissions": {
    "deny": [
      "Read(*.env)",
      "Read(*secret*)",
      "Read(*credential*)"
    ]
  }
}
```

---

## Part 6: IDE Security

### VS Code

- Use Restricted Mode for untrusted repos
- Disable auto-accept for sensitive code
- Review config file changes

### JetBrains

- Use manual approval mode
- Be cautious with trusted prompts
- Monitor file access

---

## Part 7: CI/CD Security

### Environment Variables

```yaml
env:
  ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

### Job Isolation

```yaml
jobs:
  claude:
    runs-on: ubuntu-latest
    container:
      image: node:20
      options: --network none  # No network access
```

### Review All PRs

Never auto-merge Claude-generated code.

---

## Part 8: Network Security

### Restrict Network Access

```bash
claude --sandbox  # Limited network
```

### Proxy Configuration

For enterprise:
```json
{
  "proxy": "http://proxy.company.com:8080"
}
```

---

## Quick Reference

### Security Checklist

- [ ] Use least-privilege permissions
- [ ] Enable sandbox for untrusted code
- [ ] Never commit API keys
- [ ] Use OIDC in CI/CD
- [ ] Restrict bash commands
- [ ] Protect sensitive files
- [ ] Review all generated changes

### Dangerous Patterns

| Avoid | Risk |
|-------|------|
| `bypassPermissions` | All prompts skipped |
| `Bash(*)` allow | Any command runs |
| Auto-merge PRs | Unreviewed code |
| Keys in code | Exposed credentials |

---

## Sources

- [Security](https://code.claude.com/docs/en/security.md)
- [Permissions](https://code.claude.com/docs/en/permissions.md)
- [Sandboxing](https://code.claude.com/docs/en/sandboxing.md)
