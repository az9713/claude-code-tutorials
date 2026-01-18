# Tutorial 18: Plugin Marketplaces

> **Build and host plugin marketplaces to distribute Claude Code extensions across teams and communities**

## Overview

A plugin marketplace is a catalog that lets you distribute plugins to others. Marketplaces provide centralized discovery, version tracking, automatic updates, and support for multiple source types.

### What You'll Learn

- Creating marketplace.json files
- Defining plugin entries with different source types
- Hosting marketplaces on GitHub, GitLab, or other services
- Configuring team marketplaces
- Managing enterprise marketplace restrictions

### Prerequisites

- Understanding of Claude Code plugins (see [Tutorial 17](./TUTORIAL_17_CREATING_PLUGINS.md))
- Git repository for hosting
- At least one plugin to distribute

---

## Part 1: Marketplace Overview

### What Marketplaces Provide

| Feature | Description |
|---------|-------------|
| **Centralized discovery** | Browse all available plugins in one place |
| **Version tracking** | Track releases and updates |
| **Automatic updates** | Keep plugins current automatically |
| **Multiple sources** | Support git repos, local paths, remote URLs |
| **Team distribution** | Share across organization easily |

### Creation and Distribution Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│ 1. Create Plugins                                                    │
│    Build plugins with commands, agents, hooks, etc.                 │
├─────────────────────────────────────────────────────────────────────┤
│ 2. Create Marketplace File                                           │
│    Define marketplace.json listing your plugins                     │
├─────────────────────────────────────────────────────────────────────┤
│ 3. Host the Marketplace                                              │
│    Push to GitHub, GitLab, or another git host                      │
├─────────────────────────────────────────────────────────────────────┤
│ 4. Share with Users                                                  │
│    Users add with /plugin marketplace add                           │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Part 2: Quick Start - Local Marketplace

Create a marketplace with a `/review` command:

### Step 1: Create Directory Structure

```bash
mkdir -p my-marketplace/.claude-plugin
mkdir -p my-marketplace/plugins/review-plugin/.claude-plugin
mkdir -p my-marketplace/plugins/review-plugin/commands
```

### Step 2: Create Plugin Command

**my-marketplace/plugins/review-plugin/commands/review.md:**
```markdown
Review the code I've selected or the recent changes for:
- Potential bugs or edge cases
- Security concerns
- Performance issues
- Readability improvements

Be concise and actionable.
```

### Step 3: Create Plugin Manifest

**my-marketplace/plugins/review-plugin/.claude-plugin/plugin.json:**
```json
{
  "name": "review-plugin",
  "description": "Adds a /review command for quick code reviews",
  "version": "1.0.0"
}
```

### Step 4: Create Marketplace File

**my-marketplace/.claude-plugin/marketplace.json:**
```json
{
  "name": "my-plugins",
  "owner": {
    "name": "Your Name"
  },
  "plugins": [
    {
      "name": "review-plugin",
      "source": "./plugins/review-plugin",
      "description": "Adds a /review command for quick code reviews"
    }
  ]
}
```

### Step 5: Add and Install

```bash
/plugin marketplace add ./my-marketplace
/plugin install review-plugin@my-plugins
```

### Step 6: Try It Out

```bash
/review
```

---

## Part 3: Marketplace Schema

### Required Fields

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `name` | string | Marketplace identifier (kebab-case) | `"acme-tools"` |
| `owner` | object | Maintainer information | See below |
| `plugins` | array | List of available plugins | See below |

### Owner Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Maintainer or team name |
| `email` | string | No | Contact email |

### Optional Metadata

| Field | Type | Description |
|-------|------|-------------|
| `metadata.description` | string | Brief marketplace description |
| `metadata.version` | string | Marketplace version |
| `metadata.pluginRoot` | string | Base directory for relative paths |

### Complete Example

```json
{
  "name": "company-tools",
  "owner": {
    "name": "DevTools Team",
    "email": "devtools@example.com"
  },
  "metadata": {
    "description": "Internal development tools",
    "version": "2.0.0",
    "pluginRoot": "./plugins"
  },
  "plugins": [
    {
      "name": "code-formatter",
      "source": "formatter",
      "description": "Automatic code formatting on save",
      "version": "2.1.0"
    },
    {
      "name": "deployment-tools",
      "source": {
        "source": "github",
        "repo": "company/deploy-plugin"
      },
      "description": "Deployment automation tools"
    }
  ]
}
```

> **Note:** With `pluginRoot: "./plugins"`, you can write `"source": "formatter"` instead of `"source": "./plugins/formatter"`.

### Reserved Names

The following marketplace names are reserved and cannot be used:
- `claude-code-marketplace`
- `claude-code-plugins`
- `claude-plugins-official`
- `anthropic-marketplace`
- `anthropic-plugins`
- `agent-skills`
- `life-sciences`

Names that impersonate official marketplaces are also blocked.

---

## Part 4: Plugin Entries

### Required Plugin Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Plugin identifier (kebab-case) |
| `source` | string/object | Where to fetch the plugin |

### Optional Plugin Fields

**Standard Metadata:**

| Field | Type | Description |
|-------|------|-------------|
| `description` | string | Brief plugin description |
| `version` | string | Plugin version |
| `author` | object | Author info (name, email) |
| `homepage` | string | Documentation URL |
| `repository` | string | Source code URL |
| `license` | string | SPDX license identifier |
| `keywords` | array | Discovery tags |
| `category` | string | Plugin category |
| `tags` | array | Searchability tags |

**Component Configuration:**

| Field | Type | Description |
|-------|------|-------------|
| `commands` | string/array | Paths to command files |
| `agents` | string/array | Paths to agent files |
| `hooks` | string/object | Hooks configuration |
| `mcpServers` | string/object | MCP server configurations |
| `lspServers` | string/object | LSP server configurations |

**Special Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `strict` | boolean | If `false`, plugin doesn't need its own `plugin.json` |

---

## Part 5: Plugin Sources

### Relative Paths

For plugins in the same repository:

```json
{
  "name": "my-plugin",
  "source": "./plugins/my-plugin"
}
```

> **Note:** Relative paths only work when users add your marketplace via Git. For URL-based distribution, use GitHub or git URL sources.

### GitHub Repositories

```json
{
  "name": "github-plugin",
  "source": {
    "source": "github",
    "repo": "owner/plugin-repo"
  }
}
```

With specific ref:
```json
{
  "name": "github-plugin",
  "source": {
    "source": "github",
    "repo": "owner/plugin-repo",
    "ref": "v2.0"
  }
}
```

### Git Repositories (Any Host)

```json
{
  "name": "git-plugin",
  "source": {
    "source": "url",
    "url": "https://gitlab.com/team/plugin.git"
  }
}
```

### Advanced Plugin Entry

Complete example with all features:

```json
{
  "name": "enterprise-tools",
  "source": {
    "source": "github",
    "repo": "company/enterprise-plugin"
  },
  "description": "Enterprise workflow automation tools",
  "version": "2.1.0",
  "author": {
    "name": "Enterprise Team",
    "email": "enterprise@example.com"
  },
  "homepage": "https://docs.example.com/plugins/enterprise-tools",
  "repository": "https://github.com/company/enterprise-plugin",
  "license": "MIT",
  "keywords": ["enterprise", "workflow", "automation"],
  "category": "productivity",
  "commands": [
    "./commands/core/",
    "./commands/enterprise/",
    "./commands/experimental/preview.md"
  ],
  "agents": [
    "./agents/security-reviewer.md",
    "./agents/compliance-checker.md"
  ],
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
          }
        ]
      }
    ]
  },
  "mcpServers": {
    "enterprise-db": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"]
    }
  },
  "strict": false
}
```

**Key points:**
- `${CLAUDE_PLUGIN_ROOT}` references the plugin's installation directory
- `strict: false` means the plugin doesn't need its own `plugin.json`
- Multiple command/agent paths can be specified as arrays

---

## Part 6: Hosting Marketplaces

### Host on GitHub (Recommended)

1. Create a new repository
2. Add `.claude-plugin/marketplace.json`
3. Add your plugins
4. Share with: `/plugin marketplace add owner/repo`

**Benefits:** Version control, issue tracking, team collaboration

### Host on GitLab or Other Git Services

```bash
/plugin marketplace add https://gitlab.com/company/plugins.git
```

### Private Repositories

Set authentication tokens in your environment:

| Provider | Environment Variables |
|----------|----------------------|
| GitHub | `GITHUB_TOKEN` or `GH_TOKEN` |
| GitLab | `GITLAB_TOKEN` or `GL_TOKEN` |
| Bitbucket | `BITBUCKET_TOKEN` |

```bash
export GITHUB_TOKEN=ghp_xxxxxxxxxxxxxxxxxxxx
```

Tokens are only used when authentication is required.

### Test Locally Before Distribution

```bash
/plugin marketplace add ./my-local-marketplace
/plugin install test-plugin@my-local-marketplace
```

---

## Part 7: Team Marketplace Configuration

### Automatic Marketplace Prompts

Configure your repository so team members are prompted to install your marketplace when they trust the project.

**.claude/settings.json:**
```json
{
  "extraKnownMarketplaces": {
    "company-tools": {
      "source": {
        "source": "github",
        "repo": "your-org/claude-plugins"
      }
    }
  }
}
```

### Enable Plugins by Default

```json
{
  "enabledPlugins": {
    "code-formatter@company-tools": true,
    "deployment-tools@company-tools": true
  }
}
```

### Multiple Team Marketplaces

```json
{
  "extraKnownMarketplaces": {
    "frontend-tools": {
      "source": {
        "source": "github",
        "repo": "your-org/frontend-plugins"
      }
    },
    "backend-tools": {
      "source": {
        "source": "github",
        "repo": "your-org/backend-plugins"
      }
    },
    "security-tools": {
      "source": {
        "source": "github",
        "repo": "your-org/security-plugins",
        "ref": "v2.0"
      }
    }
  }
}
```

---

## Part 8: Enterprise Marketplace Restrictions

### Using strictKnownMarketplaces

Administrators can restrict which marketplaces users can add using managed settings.

| Value | Behavior |
|-------|----------|
| Undefined (default) | No restrictions; users can add any marketplace |
| Empty array `[]` | Complete lockdown; no new marketplaces allowed |
| List of sources | Only specified marketplaces allowed |

### Disable All Marketplace Additions

```json
{
  "strictKnownMarketplaces": []
}
```

### Allow Specific Marketplaces Only

```json
{
  "strictKnownMarketplaces": [
    {
      "source": "github",
      "repo": "acme-corp/approved-plugins"
    },
    {
      "source": "github",
      "repo": "acme-corp/security-tools",
      "ref": "v2.0"
    },
    {
      "source": "url",
      "url": "https://plugins.example.com/marketplace.json"
    }
  ]
}
```

### How Restrictions Work

- Validated early, before network requests
- Exact matching required (repo, ref, path must all match)
- Set in managed settings, cannot be overridden by users

---

## Part 9: Validation and Testing

### Validate Marketplace

```bash
# From command line
claude plugin validate .

# From within Claude Code
/plugin validate .
```

### Add for Testing

```bash
/plugin marketplace add ./path/to/marketplace
```

### Install Test Plugin

```bash
/plugin install test-plugin@marketplace-name
```

### Common Validation Errors

| Error | Solution |
|-------|----------|
| `File not found: .claude-plugin/marketplace.json` | Create the manifest file |
| `Invalid JSON syntax` | Check for syntax errors |
| `Duplicate plugin name` | Give each plugin unique name |
| `Path traversal not allowed` | Don't use `..` in source paths |

### Validation Warnings

| Warning | Solution |
|---------|----------|
| `Marketplace has no plugins` | Add plugins to the array |
| `No marketplace description` | Add `metadata.description` |
| `npm source not implemented` | Use `github` or local paths |

---

## Part 10: Troubleshooting

### Marketplace Not Loading

- Verify URL is accessible
- Check `.claude-plugin/marketplace.json` exists
- Validate JSON syntax
- Confirm access permissions for private repos

### Plugin Installation Failures

- Verify plugin source URLs are accessible
- Check plugin directories contain required files
- For GitHub sources, ensure repos are public or you have access

### Private Repository Authentication Fails

- Verify token is set: `echo $GITHUB_TOKEN`
- Check token has required permissions
- Ensure token hasn't expired
- Verify correct provider token is set

### Relative Paths Fail in URL-Based Marketplaces

**Cause:** URL-based marketplaces only download `marketplace.json`, not plugin files.

**Solutions:**
1. Use external sources (GitHub, git URL)
2. Use Git-based marketplace that clones entire repository

### Files Not Found After Installation

**Cause:** Plugins are copied to cache; paths outside plugin directory don't work.

**Solutions:**
1. Use symlinks inside plugin directory
2. Restructure to keep all files inside plugin source path

---

## Part 11: Complete Marketplace Example

### Directory Structure

```
company-marketplace/
├── .claude-plugin/
│   └── marketplace.json
├── plugins/
│   ├── code-formatter/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json
│   │   └── commands/
│   │       └── format.md
│   ├── security-scanner/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json
│   │   ├── commands/
│   │   │   └── scan.md
│   │   └── agents/
│   │       └── security-expert.md
│   └── deployment/
│       ├── .claude-plugin/
│       │   └── plugin.json
│       └── commands/
│           ├── deploy.md
│           └── rollback.md
└── README.md
```

### marketplace.json

```json
{
  "name": "company-tools",
  "owner": {
    "name": "DevOps Team",
    "email": "devops@example.com"
  },
  "metadata": {
    "description": "Company-wide development tools",
    "version": "1.0.0",
    "pluginRoot": "./plugins"
  },
  "plugins": [
    {
      "name": "code-formatter",
      "source": "code-formatter",
      "description": "Automatic code formatting and linting",
      "version": "1.2.0",
      "category": "code-quality",
      "keywords": ["formatting", "linting", "eslint", "prettier"]
    },
    {
      "name": "security-scanner",
      "source": "security-scanner",
      "description": "Security vulnerability scanning",
      "version": "2.0.0",
      "category": "security",
      "keywords": ["security", "vulnerabilities", "OWASP"]
    },
    {
      "name": "deployment",
      "source": "deployment",
      "description": "Deployment automation tools",
      "version": "1.5.0",
      "category": "devops",
      "keywords": ["deploy", "rollback", "kubernetes"]
    }
  ]
}
```

---

## Quick Reference

### Commands

| Command | Description |
|---------|-------------|
| `/plugin marketplace add <source>` | Add a marketplace |
| `/plugin marketplace update` | Update all marketplaces |
| `/plugin marketplace remove <name>` | Remove a marketplace |
| `/plugin install <plugin>@<marketplace>` | Install a plugin |
| `/plugin validate .` | Validate marketplace |

### Source Types

| Type | Example |
|------|---------|
| Relative path | `"./plugins/my-plugin"` |
| GitHub | `{"source": "github", "repo": "owner/repo"}` |
| Git URL | `{"source": "url", "url": "https://..."}` |

### Environment Variables

| Variable | Description |
|----------|-------------|
| `GITHUB_TOKEN` | GitHub authentication |
| `GITLAB_TOKEN` | GitLab authentication |
| `BITBUCKET_TOKEN` | Bitbucket authentication |
| `${CLAUDE_PLUGIN_ROOT}` | Plugin installation directory |

### Settings for Teams

| Setting | Description |
|---------|-------------|
| `extraKnownMarketplaces` | Pre-configure marketplaces |
| `enabledPlugins` | Auto-enable specific plugins |
| `strictKnownMarketplaces` | Restrict allowed marketplaces |

---

## Sources

- [Plugin Marketplaces - Official Documentation](https://code.claude.com/docs/en/plugin-marketplaces.md)
- [Discover and Install Plugins](https://code.claude.com/docs/en/discover-plugins.md)
- [Plugins Reference](https://code.claude.com/docs/en/plugins-reference.md)
- [Settings Reference](https://code.claude.com/docs/en/settings.md)

---

## Next Steps

1. **Create a local marketplace** - Start with one plugin
2. **Host on GitHub** - Push to a repository
3. **Share with team** - Configure in `.claude/settings.json`
4. **Set up auto-updates** - Let users stay current automatically
5. **Add enterprise controls** - Use `strictKnownMarketplaces` if needed
