# Tutorial 63: Migration & Upgrade Guide

> **Safely upgrade Claude Code versions and handle breaking changes**

## Table of Contents

1. [Overview](#overview)
2. [Version Numbering](#version-numbering)
3. [Checking Current Version](#checking-current-version)
4. [Upgrade Process](#upgrade-process)
5. [Breaking Changes by Version](#breaking-changes-by-version)
6. [Migration Checklists](#migration-checklists)
7. [Rollback Procedures](#rollback-procedures)
8. [Windows Path Migration](#windows-path-migration)
9. [SDK Migration](#sdk-migration)
10. [Configuration Migration](#configuration-migration)
11. [Troubleshooting Upgrades](#troubleshooting-upgrades)
12. [Quick Reference](#quick-reference)

---

## Overview

This guide helps you upgrade Claude Code versions safely, understand breaking changes, and handle migration tasks. Following these procedures ensures stable operation in production environments.

### Why Careful Upgrades Matter

- **Settings format changes** may require migration
- **Permission syntax** evolves between versions
- **Hook behavior** may change
- **CLI flags** can be deprecated or renamed
- **MCP protocols** may update

---

## Version Numbering

### Semantic Versioning

Claude Code uses semantic versioning: `MAJOR.MINOR.PATCH`

```
v2.1.12
│ │  └── Patch: Bug fixes, no breaking changes
│ └───── Minor: New features, backward compatible
└─────── Major: Breaking changes
```

### Release Channels (v2.1.3)

Configure release channel:

```json
{
  "releaseChannel": "stable"  // or "latest"
}
```

| Channel | Description | Recommended For |
|---------|-------------|-----------------|
| `stable` | Tested releases | Production, teams |
| `latest` | Latest features | Development, testing |

---

## Checking Current Version

### Version Commands

```bash
# Check installed version
claude --version
claude -v

# Detailed version info
claude version

# Check for updates
claude doctor
```

### Version Output

```
Claude Code v2.1.12
Node.js v20.10.0
Platform: darwin arm64
```

---

## Upgrade Process

### Step 1: Check Current Version

```bash
claude --version
```

### Step 2: Backup Configuration

```bash
# Backup user settings
cp -r ~/.claude ~/.claude.backup.$(date +%Y%m%d)

# Backup project settings (if applicable)
cp -r .claude .claude.backup.$(date +%Y%m%d)
```

### Step 3: Review Release Notes

Before upgrading, review:
- [Changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- Breaking changes for your version jump
- New required dependencies

### Step 4: Upgrade

**npm (global install):**
```bash
npm update -g @anthropic-ai/claude-code
```

**Specific version:**
```bash
npm install -g @anthropic-ai/claude-code@2.1.12
```

### Step 5: Verify Upgrade

```bash
# Check version
claude --version

# Run diagnostics
claude doctor

# Test basic functionality
claude "hello world"
```

### Step 6: Migrate Configuration (if needed)

Follow version-specific migration steps below.

---

## Breaking Changes by Version

### v2.1.0 Breaking Changes

**Hooks:**
- `once: true` is now supported (no migration needed)
- Frontmatter hooks in skills now supported

**Permissions:**
- Wildcard syntax introduced for Bash
- `Task(AgentName)` syntax for agent permissions

**Migration actions:**
```json
// Old (still works but deprecated patterns):
{
  "Bash": {
    "allow": ["npm test"]
  }
}

// New (recommended):
{
  "Bash": {
    "allow": ["npm *"]  // If you want wildcard
  }
}
```

### v2.0.x → v2.1.x Migration

**New settings to consider:**

```json
{
  "mcpToolSearch": "auto:10",          // v2.1.7+
  "plansDirectory": "~/.claude/plans", // v2.1.9+
  "showTurnDuration": true,            // v2.1.7+
  "language": "en"                     // v2.1.0+
}
```

### v1.x → v2.x Migration

Major breaking changes:

1. **Configuration file format changed**
   - Old: Various JSON formats
   - New: Standardized `~/.claude/settings.json`

2. **MCP tool naming changed**
   - Old: `mcp_server_tool`
   - New: `mcp__server__tool`

3. **Permission syntax changed**
   - Old: Various formats
   - New: Standardized `tools` object

**Migration script:**

```bash
#!/bin/bash
# migrate-v1-to-v2.sh

OLD_CONFIG="$HOME/.claude-code/config.json"
NEW_CONFIG="$HOME/.claude/settings.json"

if [[ -f "$OLD_CONFIG" ]]; then
  echo "Found v1 config, migrating..."
  mkdir -p "$HOME/.claude"

  # Basic migration (customize as needed)
  jq '{
    model: .model,
    permissions: {
      tools: (.permissions // {})
    }
  }' "$OLD_CONFIG" > "$NEW_CONFIG"

  echo "Migration complete. Review $NEW_CONFIG"
fi
```

---

## Migration Checklists

### Pre-Upgrade Checklist

- [ ] Document current version: `claude --version`
- [ ] Backup `~/.claude/` directory
- [ ] Backup project `.claude/` directories
- [ ] Review changelog for breaking changes
- [ ] Check plugin compatibility
- [ ] Schedule upgrade during low-activity period
- [ ] Notify team members (if shared config)

### Post-Upgrade Checklist

- [ ] Verify version: `claude --version`
- [ ] Run diagnostics: `claude doctor`
- [ ] Test basic conversation
- [ ] Test tool usage (Bash, Read, Write)
- [ ] Test MCP tools if used
- [ ] Test hooks if configured
- [ ] Verify permissions work correctly
- [ ] Check plugin functionality

### Rollback Decision Points

Rollback if:
- [ ] Core functionality broken
- [ ] Critical hooks failing
- [ ] Permission system behaving unexpectedly
- [ ] MCP tools not connecting

---

## Rollback Procedures

### Quick Rollback

```bash
# Install specific previous version
npm install -g @anthropic-ai/claude-code@2.1.11

# Restore configuration
rm -rf ~/.claude
mv ~/.claude.backup.YYYYMMDD ~/.claude
```

### Partial Rollback (Config Only)

```bash
# Keep new binary, restore old config
cp ~/.claude.backup.YYYYMMDD/settings.json ~/.claude/
```

### Version Pinning

Prevent accidental upgrades:

```json
// package.json (if project-local)
{
  "dependencies": {
    "@anthropic-ai/claude-code": "2.1.12"
  }
}
```

---

## Windows Path Migration

### Path Format Changes

Windows paths migrated from backslash to forward slash compatibility:

**Old format (still supported):**
```json
{
  "permissions": {
    "tools": {
      "Read": {
        "deny": ["C:\\Users\\*\\AppData\\*"]
      }
    }
  }
}
```

**New format (recommended):**
```json
{
  "permissions": {
    "tools": {
      "Read": {
        "deny": ["C:/Users/*/AppData/*"]
      }
    }
  }
}
```

### Automatic Migration

Claude Code v2.1.x auto-converts paths. No manual action required.

### Manual Path Migration Script

```powershell
# migrate-paths.ps1

$configPath = "$env:USERPROFILE\.claude\settings.json"

if (Test-Path $configPath) {
    $content = Get-Content $configPath -Raw
    # Convert backslashes to forward slashes in paths
    $content = $content -replace '\\\\', '/'
    Set-Content $configPath $content
    Write-Host "Paths migrated successfully"
}
```

---

## SDK Migration

### Claude Agent SDK Changes

If using the Agent SDK for custom integrations:

**v1.x SDK:**
```javascript
const { Claude } = require('@anthropic-ai/claude-agent-sdk');
const claude = new Claude({ apiKey: '...' });
```

**v2.x SDK:**
```javascript
import { ClaudeAgent } from '@anthropic-ai/claude-agent-sdk';
const agent = new ClaudeAgent({ apiKey: '...' });
```

### API Changes

| v1.x | v2.x |
|------|------|
| `claude.complete()` | `agent.chat()` |
| `claude.tools` | `agent.tools` |
| `onProgress` callback | `stream` option |

### Migration Example

```javascript
// Old v1.x
const response = await claude.complete({
  prompt: userInput,
  onProgress: (chunk) => console.log(chunk)
});

// New v2.x
const stream = await agent.chat({
  messages: [{ role: 'user', content: userInput }],
  stream: true
});

for await (const event of stream) {
  console.log(event);
}
```

---

## Configuration Migration

### Settings File Locations

| Version | Location |
|---------|----------|
| v1.x | `~/.claude-code/config.json` |
| v2.0.x | `~/.claude/settings.json` |
| v2.1.x | `~/.claude/settings.json` (same) |

### Key Renames

| Old Key | New Key | Version |
|---------|---------|---------|
| `apiKey` | Use `ANTHROPIC_API_KEY` env | v2.0+ |
| `defaultModel` | `model` | v2.0+ |
| `permissionRules` | `permissions.tools` | v2.0+ |

### Full Migration Script

```bash
#!/bin/bash
# Full migration script v1.x to v2.1.x

set -e

OLD_DIR="$HOME/.claude-code"
NEW_DIR="$HOME/.claude"
BACKUP_DIR="$HOME/.claude-migration-backup-$(date +%Y%m%d)"

echo "Claude Code Migration Script"
echo "============================"

# Backup everything
if [[ -d "$NEW_DIR" ]]; then
  echo "Backing up existing config to $BACKUP_DIR"
  cp -r "$NEW_DIR" "$BACKUP_DIR"
fi

# Create new structure
mkdir -p "$NEW_DIR/skills"
mkdir -p "$NEW_DIR/commands"
mkdir -p "$NEW_DIR/logs"

# Migrate settings
if [[ -f "$OLD_DIR/config.json" ]]; then
  echo "Migrating settings..."

  jq '{
    model: (.defaultModel // "claude-sonnet-4-20250514"),
    permissions: {
      defaultMode: "default",
      tools: (.permissionRules // {})
    }
  }' "$OLD_DIR/config.json" > "$NEW_DIR/settings.json"
fi

# Migrate skills/commands if they exist
if [[ -d "$OLD_DIR/skills" ]]; then
  cp -r "$OLD_DIR/skills/"* "$NEW_DIR/skills/" 2>/dev/null || true
fi

if [[ -d "$OLD_DIR/commands" ]]; then
  cp -r "$OLD_DIR/commands/"* "$NEW_DIR/commands/" 2>/dev/null || true
fi

echo "Migration complete!"
echo "Please verify: cat $NEW_DIR/settings.json"
```

---

## Troubleshooting Upgrades

### Common Issues

#### "Command not found" After Upgrade

```bash
# Rehash shell commands
hash -r  # bash/zsh
rehash   # zsh

# Verify installation path
which claude
npm list -g @anthropic-ai/claude-code
```

#### Settings Not Loading

```bash
# Check file validity
cat ~/.claude/settings.json | jq .

# If invalid JSON, restore backup
cp ~/.claude.backup.*/settings.json ~/.claude/
```

#### Permissions Changed Unexpectedly

```bash
# Check for unreachable rules
claude doctor

# View current permissions
/permissions *
```

#### Hooks Not Running

```bash
# Verify hook file permissions
ls -la ~/.claude/hooks/

# Make executable
chmod +x ~/.claude/hooks/*.sh

# Test hook manually
CLAUDE_TOOL_NAME="Bash" ~/.claude/hooks/pre-tool.sh
```

#### MCP Tools Missing

```bash
# Check MCP configuration
claude mcp list

# Verify server is running
claude mcp status
```

### Debug Mode for Upgrades

```bash
# Run with verbose logging
claude --verbose

# Full debug mode
CLAUDE_DEBUG=1 claude
```

---

## Quick Reference

### Version Commands

```bash
claude --version          # Show version
claude doctor             # Diagnostics including update check
```

### Upgrade Commands

```bash
# Upgrade to latest
npm update -g @anthropic-ai/claude-code

# Specific version
npm install -g @anthropic-ai/claude-code@X.Y.Z

# Check available versions
npm view @anthropic-ai/claude-code versions
```

### Backup Commands

```bash
# Full backup
cp -r ~/.claude ~/.claude.backup.$(date +%Y%m%d)

# Settings only
cp ~/.claude/settings.json ~/.claude/settings.json.backup
```

### Rollback Commands

```bash
# Downgrade version
npm install -g @anthropic-ai/claude-code@PREVIOUS_VERSION

# Restore config
rm -rf ~/.claude && mv ~/.claude.backup.YYYYMMDD ~/.claude
```

### Key Migration Points

| From | To | Key Changes |
|------|----|----|
| v1.x | v2.0 | Config location, permission format |
| v2.0.x | v2.1.x | MCP tool search, wildcard permissions |

### Version Feature Matrix

| Feature | v2.0.x | v2.1.0+ |
|---------|--------|---------|
| Wildcard permissions | ❌ | ✓ |
| MCP tool search | ❌ | ✓ |
| LSP integration | ❌ | ✓ |
| Chrome integration | Partial | ✓ |
| Hook additionalContext | ❌ | ✓ |
| once:true hooks | ❌ | ✓ |
| Skill hot-reload | ❌ | ✓ |
| context:fork | ❌ | ✓ |

---

## Sources

- [Claude Code Changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Claude Code Documentation](https://code.claude.com/docs/)
- [Migration Notes](https://code.claude.com/docs/en/migration.md)

---

*This tutorial is part of the Claude Code documentation series.*
*Version 2.1.x | Last Updated: January 2026*
