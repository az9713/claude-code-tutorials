# Tutorial 37: Privacy and Data Handling

> **Understand how Claude Code handles your data**

## Overview

Important information about data privacy and handling with Claude Code.

---

## Part 1: Data Flow

### What Claude Sees

- Files you reference or select
- Terminal output
- Diagnostic information
- Conversation history

### What Claude Doesn't Automatically Access

- Unreferenced files
- System files outside workspace
- Credential files (when properly protected)

---

## Part 2: Local vs Cloud Processing

### Interactive Sessions

- Prompts sent to Anthropic API
- Code context included in requests
- No persistent storage of code

### Background Tasks

- Run on Anthropic VMs
- Cloned repository
- Temporary execution environment
- Cleaned after completion

---

## Part 3: Enterprise Options

### AWS Bedrock

- Data stays in your AWS account
- Regional processing
- Your security controls

### Google Vertex AI

- Data in your GCP project
- Workload Identity Federation
- Compliance controls

### On-Premises

Contact Anthropic for enterprise options.

---

## Part 4: Protecting Sensitive Data

### Exclude Files

In CLAUDE.md:
```markdown
# Privacy

Never read or reference:
- .env files
- *secret* files
- credentials/*
```

### Permission Denials

```json
{
  "permissions": {
    "deny": [
      "Read(*.env)",
      "Read(.env*)",
      "Read(*secret*)",
      "Read(*credential*)"
    ]
  }
}
```

---

## Part 5: Session Data

### Conversation Storage

- Stored locally in SQLite
- 30-day default retention
- Configurable cleanup

### Clear Session Data

```bash
/clear  # Current session

# Full cleanup
rm -rf ~/.claude/sessions/
```

---

## Part 6: Team Considerations

### Shared Workspaces

- API usage tracked per workspace
- Cost reporting available
- Admin controls for limits

### Audit Logging

For enterprise, consider:
- LiteLLM for detailed logging
- Cloud provider audit logs
- Custom proxy logging

---

## Quick Reference

### Protection Checklist

- [ ] Deny access to credential files
- [ ] Use enterprise providers if required
- [ ] Configure session retention
- [ ] Review data in prompts
- [ ] Use sandbox for sensitive analysis

### Data Locations

| Data | Location |
|------|----------|
| Sessions | `~/.claude/sessions/` |
| Settings | `~/.claude/settings.json` |
| Logs | `~/.claude/logs/` |

---

## Sources

- [Security](https://code.claude.com/docs/en/security.md)
- [Settings](https://code.claude.com/docs/en/settings.md)
