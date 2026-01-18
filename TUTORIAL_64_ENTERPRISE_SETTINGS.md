# Tutorial 64: Enterprise Managed Settings

> **Centralized configuration, team policies, and managed deployments for organizations**

## Table of Contents

1. [Overview](#overview)
2. [Configuration Hierarchy](#configuration-hierarchy)
3. [Managed Settings Distribution](#managed-settings-distribution)
4. [Security Policies](#security-policies)
5. [Team Configuration Patterns](#team-configuration-patterns)
6. [Compliance Settings](#compliance-settings)
7. [Audit and Logging](#audit-and-logging)
8. [Deployment Strategies](#deployment-strategies)
9. [Onboarding Automation](#onboarding-automation)
10. [Monitoring and Enforcement](#monitoring-and-enforcement)
11. [Troubleshooting](#troubleshooting)
12. [Quick Reference](#quick-reference)

---

## Overview

Enterprise deployments of Claude Code require centralized control over settings, permissions, and policies. This tutorial covers strategies for managing Claude Code across teams and organizations.

### Enterprise Requirements

| Requirement | Solution |
|-------------|----------|
| Consistent settings | Managed configuration files |
| Security policies | Permission templates |
| Compliance | Audit logging, deny rules |
| Scalability | Automated deployment |
| Governance | Centralized control |

### Key Concepts

- **Managed settings**: Centrally distributed configuration
- **Policy enforcement**: Mandatory rules that can't be overridden
- **Audit trails**: Logging for compliance
- **Template configurations**: Standardized setups for teams

---

## Configuration Hierarchy

### Priority Order (Highest to Lowest)

1. **Command-line arguments** - Runtime overrides
2. **Environment variables** - Process-level config
3. **Workspace settings** - `.claude/settings.local.json` (not committed)
4. **Project settings** - `.claude/settings.json` (committed)
5. **User settings** - `~/.claude/settings.json`
6. **Managed settings** - Enterprise distribution
7. **Default settings** - Built-in values

### Enterprise Control Points

```
┌─────────────────────────────────────────────────────────────┐
│                    Enterprise Control                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Managed Settings (Highest Enterprise Control)              │
│  ├── Security policies (mandatory denies)                   │
│  ├── Audit requirements (mandatory hooks)                   │
│  └── API configuration (endpoints, keys)                    │
│                                                              │
│  User Settings (User Customization)                         │
│  ├── Personal preferences                                   │
│  └── Allowed customizations                                 │
│                                                              │
│  Project Settings (Team Standards)                          │
│  ├── Project-specific rules                                 │
│  └── Shared team configuration                              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Managed Settings Distribution

### Method 1: System-Wide Configuration

Place managed settings in system location:

**macOS/Linux:**
```bash
/etc/claude/settings.json
```

**Windows:**
```
C:\ProgramData\Claude\settings.json
```

### Method 2: Environment Variable

Point to managed config file:

```bash
export CLAUDE_MANAGED_SETTINGS="/path/to/managed/settings.json"
```

### Method 3: Network Distribution

Fetch settings from internal endpoint:

```bash
export CLAUDE_SETTINGS_URL="https://internal.company.com/claude/settings.json"
```

### Method 4: Package Management

Include in company npm package:

```json
// @company/claude-config/package.json
{
  "name": "@company/claude-config",
  "version": "1.0.0",
  "files": ["settings.json", "hooks/"]
}
```

```bash
npm install -g @company/claude-config
export CLAUDE_MANAGED_SETTINGS="$(npm root -g)/@company/claude-config/settings.json"
```

### Managed Settings Template

```json
{
  "$schema": "https://code.claude.com/schemas/settings.json",
  "version": "2.1",
  "managedBy": "IT Security Team",
  "lastUpdated": "2026-01-15",

  "api": {
    "baseUrl": "https://api.internal.company.com/claude",
    "organizationId": "org-xxx"
  },

  "model": "claude-sonnet-4-20250514",

  "permissions": {
    "defaultMode": "ask",
    "tools": {
      "Bash": {
        "defaultMode": "ask",
        "deny": [
          "*sudo*",
          "*rm -rf*",
          "*curl * | bash*",
          "*wget * | sh*",
          "*> /etc/*",
          "*production*"
        ]
      },
      "Read": {
        "deny": [
          ".env*",
          "*.pem",
          "*.key",
          "**/secrets/**",
          "**/credentials/**"
        ]
      },
      "Write": {
        "deny": [
          ".env*",
          "*.pem",
          "*.key"
        ]
      }
    }
  },

  "hooks": {
    "PostToolUse": [
      {
        "command": "/opt/claude/hooks/audit-log.sh",
        "managed": true
      }
    ]
  },

  "features": {
    "allowPlugins": false,
    "allowCustomMCP": false,
    "allowBypassPermissions": false
  }
}
```

---

## Security Policies

### Mandatory Deny Rules

Rules that cannot be overridden by users:

```json
{
  "permissions": {
    "mandatoryDeny": {
      "Bash": [
        "*sudo*",
        "*rm -rf /*",
        "*chmod 777*",
        "*> /etc/*",
        "*curl * | bash*"
      ],
      "Read": [
        "/etc/passwd",
        "/etc/shadow",
        "**/.ssh/**",
        "**/.aws/credentials"
      ],
      "Write": [
        "/etc/**",
        "/usr/**",
        "/bin/**"
      ]
    }
  }
}
```

### Network Restrictions

```json
{
  "network": {
    "allowedDomains": [
      "api.anthropic.com",
      "*.internal.company.com"
    ],
    "blockedDomains": [
      "*.pastebin.com",
      "*.hastebin.com"
    ],
    "proxy": {
      "http": "http://proxy.company.com:8080",
      "https": "http://proxy.company.com:8080",
      "noProxy": ["localhost", "127.0.0.1", ".internal.company.com"]
    }
  }
}
```

### API Key Management

Never store API keys in settings files. Use:

```json
{
  "api": {
    "keySource": "environment",
    "keyVariable": "ANTHROPIC_API_KEY"
  }
}
```

Or integrate with secrets management:

```json
{
  "api": {
    "keySource": "vault",
    "vaultPath": "secret/claude/api-key"
  }
}
```

---

## Team Configuration Patterns

### Pattern 1: Role-Based Configurations

Create configurations for different roles:

**developers.json:**
```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "allow": ["npm *", "git *", "make *"],
        "deny": ["*production*", "*deploy*"]
      },
      "Task(*)": { "defaultMode": "allow" }
    }
  }
}
```

**reviewers.json:**
```json
{
  "permissions": {
    "defaultMode": "deny",
    "tools": {
      "Read": { "defaultMode": "allow" },
      "Glob": { "defaultMode": "allow" },
      "Grep": { "defaultMode": "allow" },
      "Task(reviewer)": { "defaultMode": "allow" },
      "Task(Explore)": { "defaultMode": "allow" }
    }
  }
}
```

**security-auditors.json:**
```json
{
  "permissions": {
    "tools": {
      "Read": { "defaultMode": "allow" },
      "Grep": { "defaultMode": "allow" },
      "Task(security-auditor)": { "defaultMode": "allow" }
    }
  }
}
```

### Pattern 2: Environment-Based

**development.json:**
```json
{
  "model": "claude-haiku-3-5-20241022",
  "permissions": {
    "defaultMode": "allow"
  }
}
```

**staging.json:**
```json
{
  "model": "claude-sonnet-4-20250514",
  "permissions": {
    "defaultMode": "ask"
  }
}
```

**production.json:**
```json
{
  "model": "claude-sonnet-4-20250514",
  "permissions": {
    "defaultMode": "deny",
    "tools": {
      "Read": { "defaultMode": "allow" },
      "Glob": { "defaultMode": "allow" }
    }
  }
}
```

### Pattern 3: Project Templates

```bash
# Company project template
company-project/
├── .claude/
│   ├── settings.json       # Project defaults
│   ├── settings.local.json # User overrides (gitignored)
│   ├── skills/
│   │   └── company-standards.md
│   └── hooks/
│       └── pre-commit.sh
└── ...
```

---

## Compliance Settings

### SOC 2 Compliance

```json
{
  "compliance": {
    "soc2": true
  },
  "audit": {
    "enabled": true,
    "logLevel": "detailed",
    "logDestination": "syslog"
  },
  "permissions": {
    "mandatoryDeny": {
      "Read": ["**/pii/**", "**/customer-data/**"],
      "Write": ["**/audit-logs/**"]
    }
  }
}
```

### HIPAA Compliance

```json
{
  "compliance": {
    "hipaa": true
  },
  "permissions": {
    "mandatoryDeny": {
      "Read": ["**/phi/**", "**/patient-records/**"],
      "Bash": ["*curl*", "*wget*"]
    }
  },
  "network": {
    "allowedDomains": ["api.internal.company.com"]
  }
}
```

### PCI-DSS Compliance

```json
{
  "compliance": {
    "pciDss": true
  },
  "permissions": {
    "mandatoryDeny": {
      "Read": ["**/cardholder/**", "*.pan"],
      "Bash": ["*echo*card*", "*print*"]
    }
  }
}
```

---

## Audit and Logging

### Audit Hook

```bash
#!/bin/bash
# /opt/claude/hooks/audit-log.sh

LOG_FILE="/var/log/claude/audit.log"
TIMESTAMP=$(date -Iseconds)

cat >> "$LOG_FILE" << EOF
{
  "timestamp": "$TIMESTAMP",
  "session": "$CLAUDE_SESSION_ID",
  "user": "$(whoami)",
  "tool": "$CLAUDE_TOOL_NAME",
  "input": $CLAUDE_TOOL_INPUT
}
EOF
```

### Centralized Logging

Send to SIEM:

```bash
#!/bin/bash
# /opt/claude/hooks/siem-log.sh

curl -X POST "$SIEM_ENDPOINT" \
  -H "Content-Type: application/json" \
  -d "{
    \"timestamp\": \"$(date -Iseconds)\",
    \"source\": \"claude-code\",
    \"session\": \"$CLAUDE_SESSION_ID\",
    \"user\": \"$(whoami)\",
    \"hostname\": \"$(hostname)\",
    \"tool\": \"$CLAUDE_TOOL_NAME\",
    \"input\": $CLAUDE_TOOL_INPUT
  }"
```

### Log Rotation

```bash
# /etc/logrotate.d/claude
/var/log/claude/*.log {
    daily
    rotate 90
    compress
    delaycompress
    missingok
    notifempty
    create 0640 root adm
}
```

---

## Deployment Strategies

### Strategy 1: Configuration Management

**Ansible:**
```yaml
# claude-config.yml
- name: Deploy Claude Code settings
  hosts: developer_workstations
  tasks:
    - name: Create Claude config directory
      file:
        path: /etc/claude
        state: directory
        mode: '0755'

    - name: Deploy managed settings
      copy:
        src: files/claude-settings.json
        dest: /etc/claude/settings.json
        mode: '0644'

    - name: Deploy hooks
      copy:
        src: files/hooks/
        dest: /etc/claude/hooks/
        mode: '0755'

    - name: Set environment variable
      lineinfile:
        path: /etc/environment
        line: 'CLAUDE_MANAGED_SETTINGS=/etc/claude/settings.json'
```

**Puppet:**
```puppet
# claude/manifests/init.pp
class claude {
  file { '/etc/claude':
    ensure => directory,
  }

  file { '/etc/claude/settings.json':
    ensure  => file,
    source  => 'puppet:///modules/claude/settings.json',
    require => File['/etc/claude'],
  }
}
```

### Strategy 2: Container Images

```dockerfile
# Dockerfile
FROM node:20-slim

# Install Claude Code
RUN npm install -g @anthropic-ai/claude-code@2.1.12

# Copy managed settings
COPY settings.json /etc/claude/settings.json
COPY hooks/ /etc/claude/hooks/

# Set environment
ENV CLAUDE_MANAGED_SETTINGS=/etc/claude/settings.json

USER node
ENTRYPOINT ["claude"]
```

### Strategy 3: MDM Distribution

For macOS with Jamf/Kandji:

```xml
<!-- com.anthropic.claude.plist -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>CLAUDE_MANAGED_SETTINGS</key>
    <string>/Library/Application Support/Claude/settings.json</string>
</dict>
</plist>
```

---

## Onboarding Automation

### Setup Script

```bash
#!/bin/bash
# setup-claude.sh

set -e

echo "Setting up Claude Code for enterprise..."

# Check prerequisites
command -v node >/dev/null || { echo "Node.js required"; exit 1; }
command -v npm >/dev/null || { echo "npm required"; exit 1; }

# Install Claude Code
npm install -g @anthropic-ai/claude-code@2.1.12

# Create directories
mkdir -p ~/.claude/skills ~/.claude/commands

# Copy company templates
cp /opt/company/claude/skills/* ~/.claude/skills/
cp /opt/company/claude/commands/* ~/.claude/commands/

# Set up environment
if ! grep -q CLAUDE_MANAGED_SETTINGS ~/.bashrc; then
  echo 'export CLAUDE_MANAGED_SETTINGS=/etc/claude/settings.json' >> ~/.bashrc
fi

# Verify setup
claude doctor

echo "Setup complete! Restart your terminal or run: source ~/.bashrc"
```

### Interactive Setup

```bash
#!/bin/bash
# interactive-setup.sh

echo "Claude Code Enterprise Setup"
echo "============================"

# Role selection
echo "Select your role:"
select role in "Developer" "Reviewer" "Security Auditor" "Admin"; do
  case $role in
    Developer) CONFIG="developer.json"; break;;
    Reviewer) CONFIG="reviewer.json"; break;;
    "Security Auditor") CONFIG="security.json"; break;;
    Admin) CONFIG="admin.json"; break;;
  esac
done

# Copy role config
cp "/opt/company/claude/configs/$CONFIG" ~/.claude/settings.json

# Team selection
echo "Select your team:"
select team in "Frontend" "Backend" "Platform" "Security"; do
  TEAM_SKILLS="/opt/company/claude/teams/$team/skills/"
  if [[ -d "$TEAM_SKILLS" ]]; then
    cp "$TEAM_SKILLS"* ~/.claude/skills/
  fi
  break
done

echo "Setup complete for $role in $team team!"
```

---

## Monitoring and Enforcement

### Health Check Script

```bash
#!/bin/bash
# health-check.sh

ERRORS=0

# Check managed settings are applied
if [[ ! -f "$CLAUDE_MANAGED_SETTINGS" ]]; then
  echo "ERROR: Managed settings not found"
  ((ERRORS++))
fi

# Check version
REQUIRED_VERSION="2.1.12"
CURRENT_VERSION=$(claude --version 2>/dev/null | grep -oE '[0-9]+\.[0-9]+\.[0-9]+')
if [[ "$CURRENT_VERSION" != "$REQUIRED_VERSION" ]]; then
  echo "WARNING: Version mismatch. Required: $REQUIRED_VERSION, Found: $CURRENT_VERSION"
fi

# Check mandatory hooks
if ! grep -q "audit-log.sh" ~/.claude/settings.json 2>/dev/null; then
  echo "WARNING: Audit hook may not be configured"
fi

exit $ERRORS
```

### Compliance Report

```bash
#!/bin/bash
# compliance-report.sh

echo "Claude Code Compliance Report"
echo "============================="
echo "Date: $(date)"
echo "Hostname: $(hostname)"
echo "User: $(whoami)"
echo ""

# Version check
echo "Version: $(claude --version 2>/dev/null || echo 'Not installed')"

# Settings check
echo ""
echo "Configuration:"
echo "  Managed settings: ${CLAUDE_MANAGED_SETTINGS:-'Not set'}"
echo "  Settings file exists: $(test -f "${CLAUDE_MANAGED_SETTINGS}" && echo 'Yes' || echo 'No')"

# Permission check
echo ""
echo "Permissions:"
if [[ -f ~/.claude/settings.json ]]; then
  jq '.permissions.tools | keys[]' ~/.claude/settings.json 2>/dev/null
fi

# Audit log check
echo ""
echo "Audit Logging:"
if [[ -f /var/log/claude/audit.log ]]; then
  echo "  Last 5 entries:"
  tail -5 /var/log/claude/audit.log
else
  echo "  No audit log found"
fi
```

---

## Troubleshooting

### Settings Not Applied

**Check order of precedence:**
```bash
# Show effective settings
claude config show

# Check managed settings path
echo $CLAUDE_MANAGED_SETTINGS
cat $CLAUDE_MANAGED_SETTINGS
```

**Verify JSON validity:**
```bash
cat ~/.claude/settings.json | jq .
```

### User Overriding Managed Settings

**Problem:** Users creating local overrides

**Solution:** Use mandatory deny rules that can't be overridden:
```json
{
  "permissions": {
    "mandatoryDeny": {
      "Bash": ["*sudo*"]
    }
  }
}
```

### Audit Logs Not Writing

**Check permissions:**
```bash
ls -la /var/log/claude/
touch /var/log/claude/test.log
```

**Check hook execution:**
```bash
CLAUDE_TOOL_NAME="test" /opt/claude/hooks/audit-log.sh
```

---

## Quick Reference

### Configuration Files

| Type | Location | Purpose |
|------|----------|---------|
| Managed | `/etc/claude/settings.json` | Enterprise policies |
| User | `~/.claude/settings.json` | User preferences |
| Project | `.claude/settings.json` | Team standards |
| Local | `.claude/settings.local.json` | Personal overrides |

### Environment Variables

| Variable | Purpose |
|----------|---------|
| `CLAUDE_MANAGED_SETTINGS` | Path to managed settings |
| `CLAUDE_SETTINGS_URL` | URL for settings fetch |
| `ANTHROPIC_API_KEY` | API authentication |

### Key Settings

```json
{
  "permissions": {
    "mandatoryDeny": {},   // Can't be overridden
    "defaultMode": "ask",  // Default for unlisted
    "tools": {}            // Tool-specific rules
  },
  "features": {
    "allowPlugins": false,
    "allowCustomMCP": false,
    "allowBypassPermissions": false
  },
  "audit": {
    "enabled": true,
    "logDestination": "syslog"
  }
}
```

### Deployment Checklist

- [ ] Create managed settings file
- [ ] Configure mandatory deny rules
- [ ] Set up audit logging hook
- [ ] Distribute via configuration management
- [ ] Set up monitoring/health checks
- [ ] Document onboarding process
- [ ] Create role-based configurations
- [ ] Test compliance reporting

---

## Sources

- [Claude Code Settings Documentation](https://code.claude.com/docs/en/settings.md)
- [Claude Code Security](https://code.claude.com/docs/en/security.md)
- [Claude Code Administration](https://code.claude.com/docs/en/iam.md)

---

*This tutorial is part of the Claude Code documentation series.*
*Version 2.1.x | Last Updated: January 2026*
