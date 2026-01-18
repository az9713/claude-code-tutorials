# Tutorial 67: Complete Configuration Reference

> **Comprehensive settings.json guide with all keys, values, and workflow templates**

## Table of Contents

1. [Overview](#overview)
2. [Configuration Files](#configuration-files)
3. [Complete Settings Reference](#complete-settings-reference)
4. [Workflow Templates](#workflow-templates)
5. [Key Combinations](#key-combinations)
6. [Quick Reference](#quick-reference)

---

## Overview

This tutorial provides a complete reference for Claude Code's `settings.json` configuration file, including every available key, its allowed values, and practical workflow templates.

### Configuration Hierarchy (Priority Order)

| Priority | Location | Purpose |
|----------|----------|---------|
| 1 (Highest) | CLI flags | Session overrides |
| 2 | Environment variables | Process-level config |
| 3 | `.claude/settings.local.json` | Local workspace (gitignored) |
| 4 | `.claude/settings.json` | Project settings (committed) |
| 5 | `~/.claude/settings.json` | User defaults |
| 6 | System settings | Enterprise managed |
| 7 (Lowest) | Built-in defaults | Fallback values |

---

## Configuration Files

### File Locations

**User Settings:**
```
~/.claude/settings.json                    # macOS/Linux
%USERPROFILE%\.claude\settings.json        # Windows
```

**Project Settings:**
```
<project-root>/.claude/settings.json       # Committed to version control
<project-root>/.claude/settings.local.json # Local only (gitignore)
```

**System/Enterprise Settings:**
```
/etc/claude/settings.json                  # macOS/Linux
C:\ProgramData\Claude\settings.json        # Windows
```

### Basic Structure

```json
{
  "model": "claude-sonnet-4-20250514",
  "permissions": {},
  "mcp": {},
  "hooks": {},
  "features": {}
}
```

---

## Complete Settings Reference

### Model Configuration

#### `model`

**Type:** `string`
**Default:** `"claude-sonnet-4-20250514"`
**Scope:** All levels

The Claude model to use for conversations.

```json
{
  "model": "claude-sonnet-4-20250514"
}
```

**Allowed Values:**

| Value | Description | Best For |
|-------|-------------|----------|
| `"claude-opus-4-5-20251101"` | Most capable | Complex reasoning, architecture |
| `"claude-sonnet-4-20250514"` | Balanced | General development |
| `"claude-haiku-3-5-20241022"` | Fastest | Quick tasks, iteration |
| `"opus"` | Alias for latest Opus | Convenience |
| `"sonnet"` | Alias for latest Sonnet | Convenience |
| `"haiku"` | Alias for latest Haiku | Convenience |

---

#### `modelAliases`

**Type:** `object`
**Default:** `{}`
**Scope:** User, Project

Custom shortcuts for model names.

```json
{
  "modelAliases": {
    "fast": "claude-haiku-3-5-20241022",
    "smart": "claude-opus-4-5-20251101",
    "default": "claude-sonnet-4-20250514",
    "thinking": "claude-opus-4-5-20251101"
  }
}
```

**Usage:**
```bash
claude --model fast
claude --model thinking
```

---

#### `maxTokens`

**Type:** `integer`
**Default:** `16384`
**Range:** `1` - `128000`
**Scope:** All levels

Maximum tokens in Claude's response.

```json
{
  "maxTokens": 8192
}
```

---

#### `maxThinkingTokens`

**Type:** `integer`
**Default:** `8192`
**Range:** `1024` - `65536`
**Scope:** All levels

Maximum tokens for extended thinking (chain-of-thought).

```json
{
  "maxThinkingTokens": 16384
}
```

---

#### `temperature`

**Type:** `number`
**Default:** `1.0`
**Range:** `0.0` - `1.0`
**Scope:** All levels

Controls response randomness/creativity.

```json
{
  "temperature": 0.7
}
```

| Value | Behavior | Use Case |
|-------|----------|----------|
| `0.0 - 0.3` | Deterministic | Code review, debugging |
| `0.4 - 0.6` | Balanced | General coding |
| `0.7 - 0.9` | Creative | Brainstorming, design |
| `1.0` | Maximum variety | Exploration |

---

### Permission Configuration

#### `permissions.defaultMode`

**Type:** `string`
**Default:** `"default"`
**Scope:** All levels

Global default for permission prompts.

```json
{
  "permissions": {
    "defaultMode": "ask"
  }
}
```

**Allowed Values:**

| Value | Behavior |
|-------|----------|
| `"default"` | Use tool-specific defaults |
| `"ask"` | Always prompt for confirmation |
| `"acceptEdits"` | Auto-approve file modifications |
| `"bypassPermissions"` | Skip all prompts (automation) |
| `"plan"` | Show plan without executing |

---

#### `permissions.tools`

**Type:** `object`
**Default:** `{}`
**Scope:** All levels

Per-tool permission configuration.

```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "defaultMode": "ask",
        "allow": ["git *", "npm *"],
        "deny": ["sudo *", "rm -rf /*"]
      },
      "Read": {
        "defaultMode": "allow",
        "deny": [".env*", "*.key"]
      },
      "Write": {
        "defaultMode": "ask"
      },
      "Edit": {
        "defaultMode": "ask"
      },
      "Glob": {
        "defaultMode": "allow"
      },
      "Grep": {
        "defaultMode": "allow"
      }
    }
  }
}
```

**Tool Names:**

| Tool | Description |
|------|-------------|
| `Bash` | Shell command execution |
| `Read` | File reading |
| `Write` | File creation |
| `Edit` | File modification |
| `Glob` | File pattern matching |
| `Grep` | Content searching |
| `WebFetch` | Web content retrieval |
| `WebSearch` | Web searching |
| `NotebookEdit` | Jupyter notebook editing |
| `TodoWrite` | Task list management |
| `Task` | Sub-agent invocation |
| `Skill` | Skill execution |
| `mcp__*` | MCP server tools |

---

#### Wildcard Permissions (v2.1.0+)

**Pattern Syntax:**

```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "allow": [
          "npm *",           // npm with any arguments
          "git commit *",    // git commit with any args
          "docker build *",  // docker build only
          "./scripts/*.sh"   // shell scripts in scripts/
        ],
        "deny": [
          "*sudo*",          // anything containing sudo
          "*rm -rf*",        // dangerous deletes
          "* | bash",        // piped execution
          "*production*"     // production commands
        ]
      },
      "mcp__github__*": {
        "defaultMode": "allow"
      },
      "Task(Explore)": {
        "defaultMode": "allow"
      }
    }
  }
}
```

**Wildcard Characters:**

| Pattern | Matches |
|---------|---------|
| `*` | Any characters |
| `npm *` | npm followed by anything |
| `*sudo*` | Anything containing sudo |
| `mcp__server__*` | All tools from server |
| `Task(AgentName)` | Specific agent type |

---

### MCP Server Configuration

#### `mcp.servers`

**Type:** `object`
**Default:** `{}`
**Scope:** User, Project

Configure Model Context Protocol servers.

```json
{
  "mcp": {
    "servers": {
      "filesystem": {
        "command": "npx",
        "args": ["-y", "@anthropic/mcp-server-filesystem", "/path/to/dir"],
        "env": {},
        "cwd": null,
        "timeout": 30000
      },
      "database": {
        "command": "node",
        "args": ["./tools/db-server.js"],
        "env": {
          "DB_HOST": "localhost",
          "DB_PORT": "5432"
        },
        "cwd": "./tools"
      },
      "github": {
        "command": "npx",
        "args": ["-y", "@anthropic/mcp-server-github"],
        "env": {
          "GITHUB_TOKEN": "${GITHUB_TOKEN}"
        }
      }
    }
  }
}
```

**Server Options:**

| Option | Type | Description |
|--------|------|-------------|
| `command` | `string` | Executable to run |
| `args` | `array<string>` | Command arguments |
| `env` | `object` | Environment variables |
| `cwd` | `string` | Working directory |
| `timeout` | `integer` | Startup timeout (ms) |

---

#### `mcpToolSearch` (v2.1.7+)

**Type:** `string`
**Default:** `"auto:10"`
**Scope:** User, Project

Controls MCP tool loading for context optimization.

```json
{
  "mcpToolSearch": "auto:10"
}
```

**Allowed Values:**

| Value | Behavior |
|-------|----------|
| `"enabled"` | Load all MCP tools upfront |
| `"disabled"` | Don't load MCP tools |
| `"auto:N"` | Defer if tools > N% of context |

**Examples:**
```json
{ "mcpToolSearch": "auto:10" }   // Defer if >10% context
{ "mcpToolSearch": "auto:5" }    // More aggressive deferral
{ "mcpToolSearch": "enabled" }   // Always load all tools
```

---

### Hooks Configuration

#### `hooks`

**Type:** `object`
**Default:** `{}`
**Scope:** Project

Event-driven scripts and commands.

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "./hooks/validate-bash.sh"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "./hooks/format-file.sh"
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "./hooks/setup-env.sh",
            "once": true
          }
        ]
      }
    ]
  }
}
```

**Hook Events:**

| Event | Trigger | Use Case |
|-------|---------|----------|
| `PreToolUse` | Before tool execution | Validation, blocking |
| `PostToolUse` | After tool execution | Logging, formatting |
| `SessionStart` | Session begins | Environment setup |
| `SessionStop` | Session ends | Cleanup |
| `Setup` | With `--init` flag | One-time initialization |

**Hook Options:**

| Option | Type | Description |
|--------|------|-------------|
| `type` | `string` | `"command"` only currently |
| `command` | `string` | Script/command to run |
| `once` | `boolean` | Run only once per session (v2.1.0+) |
| `timeout` | `integer` | Timeout in ms |

**Hook Return Values (PreToolUse):**

```json
// Block execution
{ "decision": "block", "reason": "Dangerous command" }

// Allow execution
{ "decision": "allow" }

// Add context (v2.1.9+)
{ "decision": "allow", "additionalContext": "Note: This file is generated" }

// Modify input
{ "decision": "allow", "updatedInput": { "command": "safe-command" } }
```

---

### Context Management

#### `autoCompact`

**Type:** `boolean`
**Default:** `true`
**Scope:** All levels

Automatically compress context when near limit.

```json
{
  "autoCompact": true
}
```

---

#### `compactThreshold`

**Type:** `integer`
**Default:** `80000`
**Range:** `10000` - `180000`
**Scope:** All levels

Token count that triggers auto-compaction.

```json
{
  "compactThreshold": 100000
}
```

---

#### `keepCodingInstructions`

**Type:** `boolean`
**Default:** `true`
**Scope:** All levels

Preserve coding context across compactions.

```json
{
  "keepCodingInstructions": true
}
```

---

### UI & Display Settings

#### `theme`

**Type:** `string`
**Default:** `"auto"`
**Scope:** User

Terminal color theme.

```json
{
  "theme": "dark"
}
```

**Allowed Values:** `"auto"`, `"dark"`, `"light"`

---

#### `outputStyle`

**Type:** `string`
**Default:** `"default"`
**Scope:** All levels

Response verbosity style.

```json
{
  "outputStyle": "concise"
}
```

**Allowed Values:** `"default"`, `"concise"`, `"verbose"`, `"technical"`

---

#### `showTurnDuration` (v2.1.7+)

**Type:** `boolean`
**Default:** `true`
**Scope:** User

Show timing information for each turn.

```json
{
  "showTurnDuration": false
}
```

---

#### `language` (v2.1.0+)

**Type:** `string`
**Default:** `"en"`
**Scope:** User

Response language preference.

```json
{
  "language": "ja"
}
```

**Supported Codes:** `"en"`, `"zh"`, `"ja"`, `"ko"`, `"es"`, `"fr"`, `"de"`, `"pt"`, `"it"`, `"ru"`, `"ar"`, `"hi"`

---

#### `statusLine`

**Type:** `object`
**Default:** `{ "show": true }`
**Scope:** User

Status bar configuration.

```json
{
  "statusLine": {
    "show": true,
    "items": ["model", "tokens", "cost", "time"]
  }
}
```

---

#### `notifications`

**Type:** `object`
**Default:** `{}`
**Scope:** User

Desktop and sound notifications.

```json
{
  "notifications": {
    "taskComplete": true,
    "errors": true,
    "warnings": false,
    "sound": false,
    "desktop": true
  }
}
```

---

### Terminal Settings

#### `terminal.input.enableIME`

**Type:** `boolean`
**Default:** `true`
**Scope:** User

Enable Input Method Editor support for CJK.

```json
{
  "terminal": {
    "input": {
      "enableIME": true,
      "compositionTimeout": 500
    }
  }
}
```

---

#### `terminal.unicodeWidth`

**Type:** `string`
**Default:** `"auto"`
**Scope:** User

Unicode character width handling.

```json
{
  "terminal": {
    "unicodeWidth": "ambiguous-wide"
  }
}
```

**Allowed Values:** `"auto"`, `"ambiguous-wide"`, `"ambiguous-narrow"`

---

### Plan Mode Settings

#### `plansDirectory` (v2.1.9+)

**Type:** `string`
**Default:** `"~/.claude/plans"`
**Scope:** User, Project

Storage location for plan files.

```json
{
  "plansDirectory": "~/.claude/plans"
}
```

---

#### `permissionMode`

**Type:** `string`
**Default:** `"default"`
**Scope:** All levels

Equivalent to `permissions.defaultMode`.

```json
{
  "permissionMode": "plan"
}
```

---

### Update & Release Settings

#### `releaseChannel` (v2.1.3+)

**Type:** `string`
**Default:** `"stable"`
**Scope:** User

Update channel for new versions.

```json
{
  "releaseChannel": "latest"
}
```

**Allowed Values:**

| Value | Description |
|-------|-------------|
| `"stable"` | Tested, stable releases |
| `"latest"` | Newest features |

---

### Network Settings

#### `proxy`

**Type:** `string | null`
**Default:** `null`
**Scope:** All levels

HTTP/HTTPS proxy URL.

```json
{
  "proxy": "http://proxy.company.com:8080"
}
```

**Formats:**
- `"http://host:port"`
- `"http://user:pass@host:port"`

---

#### `timeout`

**Type:** `integer`
**Default:** `30000`
**Scope:** All levels

API request timeout in milliseconds.

```json
{
  "timeout": 60000
}
```

---

#### `retries`

**Type:** `object`
**Default:** `{}`
**Scope:** All levels

Retry configuration for failed requests.

```json
{
  "retries": {
    "maxAttempts": 3,
    "initialDelay": 1000,
    "maxDelay": 10000,
    "backoffMultiplier": 2
  }
}
```

---

### Security Settings

#### `sandbox`

**Type:** `object`
**Default:** `{ "enabled": false }`
**Scope:** All levels

Command execution sandboxing.

```json
{
  "sandbox": {
    "enabled": true,
    "mode": "strict",
    "allowNetwork": false,
    "allowedPaths": [
      "/home/user/projects",
      "/tmp"
    ],
    "deniedPaths": [
      "/etc",
      "~/.ssh",
      "~/.aws"
    ]
  }
}
```

**Sandbox Modes:**

| Mode | Description |
|------|-------------|
| `"off"` | No sandboxing |
| `"permissive"` | Basic isolation |
| `"strict"` | Full isolation |

---

### Feature Flags

#### `features`

**Type:** `object`
**Default:** `{}`
**Scope:** System, Enterprise

Feature toggles (typically enterprise-managed).

```json
{
  "features": {
    "allowPlugins": true,
    "allowCustomMCP": true,
    "allowBypassPermissions": false,
    "allowWebFetch": true,
    "allowWebSearch": true
  }
}
```

---

### File Handling Settings

#### `respectGitignore` (v2.1.0+)

**Type:** `boolean`
**Default:** `true`
**Scope:** Project

Respect .gitignore in @-mentions.

```json
{
  "respectGitignore": true
}
```

---

#### `fileSuggestion` (v2.0.65+)

**Type:** `string`
**Default:** `"default"`
**Scope:** Project

Custom @ file search behavior.

```json
{
  "fileSuggestion": "default"
}
```

---

### API Settings

#### `api.baseUrl`

**Type:** `string`
**Default:** `"https://api.anthropic.com"`
**Scope:** System, Enterprise

API endpoint URL.

```json
{
  "api": {
    "baseUrl": "https://api.internal.company.com/claude"
  }
}
```

---

#### `api.organizationId`

**Type:** `string`
**Default:** `null`
**Scope:** Enterprise

Organization identifier for enterprise billing.

```json
{
  "api": {
    "organizationId": "org-xxx"
  }
}
```

---

## Workflow Templates

### Template 1: Personal Development

**Workflow:** Day-to-day coding with balanced security and convenience.

**Why these settings:**
- Sonnet model balances capability and cost
- Read-only operations don't prompt
- File modifications require approval
- Common dev commands pre-approved
- Dangerous operations blocked

```json
{
  "model": "sonnet",
  "modelAliases": {
    "fast": "haiku",
    "think": "opus"
  },
  "maxTokens": 8192,
  "temperature": 0.7,

  "permissions": {
    "defaultMode": "default",
    "tools": {
      "Read": { "defaultMode": "allow" },
      "Glob": { "defaultMode": "allow" },
      "Grep": { "defaultMode": "allow" },
      "Write": { "defaultMode": "ask" },
      "Edit": { "defaultMode": "ask" },
      "Bash": {
        "defaultMode": "ask",
        "allow": [
          "git *",
          "npm *",
          "yarn *",
          "pnpm *",
          "node *",
          "python *",
          "pip *",
          "cargo *",
          "go *",
          "make *"
        ],
        "deny": [
          "*sudo*",
          "*rm -rf /*",
          "*rm -rf ~/*",
          "*chmod 777*",
          "*curl * | bash*"
        ]
      }
    }
  },

  "autoCompact": true,
  "compactThreshold": 80000,
  "showTurnDuration": true,
  "theme": "auto"
}
```

---

### Template 2: Web Frontend Project

**Workflow:** React/Vue/Angular development with hot reload and testing.

**Why these settings:**
- Opus for complex component architecture
- All frontend tooling pre-approved
- Prettier/ESLint auto-run via hooks
- Dev server commands enabled

```json
{
  "model": "opus",
  "maxTokens": 16384,

  "permissions": {
    "tools": {
      "Read": { "defaultMode": "allow" },
      "Glob": { "defaultMode": "allow" },
      "Grep": { "defaultMode": "allow" },
      "Write": { "defaultMode": "ask" },
      "Edit": { "defaultMode": "ask" },
      "Bash": {
        "defaultMode": "ask",
        "allow": [
          "npm *",
          "yarn *",
          "pnpm *",
          "npx *",
          "vite *",
          "next *",
          "react-scripts *",
          "vue-cli-service *",
          "ng *",
          "jest *",
          "vitest *",
          "playwright *",
          "cypress *",
          "eslint *",
          "prettier *",
          "tsc *"
        ]
      }
    }
  },

  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write $CLAUDE_FILE_PATH"
          }
        ]
      },
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npx eslint --fix $CLAUDE_FILE_PATH"
          }
        ]
      }
    ]
  }
}
```

---

### Template 3: Backend API Development

**Workflow:** Node.js/Python API development with database access.

**Why these settings:**
- MCP server for database queries
- Docker commands for local services
- Test commands pre-approved
- Production commands blocked

```json
{
  "model": "sonnet",
  "maxTokens": 8192,

  "permissions": {
    "tools": {
      "Read": { "defaultMode": "allow" },
      "Glob": { "defaultMode": "allow" },
      "Grep": { "defaultMode": "allow" },
      "Write": { "defaultMode": "ask" },
      "Edit": { "defaultMode": "ask" },
      "Bash": {
        "defaultMode": "ask",
        "allow": [
          "npm *",
          "node *",
          "python *",
          "pip *",
          "pytest *",
          "jest *",
          "docker-compose up *",
          "docker-compose down *",
          "docker-compose logs *",
          "docker build *",
          "docker run *",
          "curl localhost:*",
          "make *"
        ],
        "deny": [
          "*production*",
          "*deploy*",
          "docker push *",
          "*rm -rf*"
        ]
      },
      "mcp__database__*": {
        "defaultMode": "ask"
      }
    }
  },

  "mcp": {
    "servers": {
      "database": {
        "command": "node",
        "args": ["./tools/db-mcp-server.js"],
        "env": {
          "DATABASE_URL": "${DATABASE_URL}"
        }
      }
    }
  },

  "mcpToolSearch": "auto:10"
}
```

---

### Template 4: Rapid Prototyping / Hackathon

**Workflow:** Maximum speed with minimal friction.

**Why these settings:**
- Haiku for fast responses
- Auto-approve most operations
- Minimal token usage
- Concise output

```json
{
  "model": "haiku",
  "modelAliases": {
    "think": "sonnet"
  },
  "maxTokens": 4096,
  "temperature": 0.8,

  "permissions": {
    "defaultMode": "acceptEdits",
    "tools": {
      "Bash": {
        "defaultMode": "acceptEdits",
        "allow": ["*"],
        "deny": [
          "*sudo*",
          "*rm -rf /*",
          "docker push *"
        ]
      },
      "Read": { "defaultMode": "allow" },
      "Write": { "defaultMode": "allow" },
      "Edit": { "defaultMode": "allow" },
      "Glob": { "defaultMode": "allow" },
      "Grep": { "defaultMode": "allow" }
    }
  },

  "autoCompact": true,
  "compactThreshold": 40000,
  "outputStyle": "concise",
  "showTurnDuration": false
}
```

---

### Template 5: Security-Conscious Development

**Workflow:** Maximum safety for sensitive codebases.

**Why these settings:**
- Every operation requires approval
- Sensitive files blocked from reading
- No shell execution
- Full audit logging via hooks

```json
{
  "model": "sonnet",
  "maxTokens": 8192,

  "permissions": {
    "defaultMode": "ask",
    "tools": {
      "Bash": {
        "defaultMode": "ask",
        "allow": [],
        "deny": ["*"]
      },
      "Read": {
        "defaultMode": "ask",
        "deny": [
          ".env*",
          "*.pem",
          "*.key",
          "*.p12",
          "**/secrets/**",
          "**/credentials/**",
          "**/.ssh/**",
          "**/.aws/**"
        ]
      },
      "Write": { "defaultMode": "ask" },
      "Edit": { "defaultMode": "ask" },
      "Glob": { "defaultMode": "ask" },
      "Grep": {
        "defaultMode": "ask",
        "deny": [
          ".env*",
          "**/secrets/**"
        ]
      },
      "WebFetch": { "defaultMode": "ask" },
      "WebSearch": { "defaultMode": "ask" }
    }
  },

  "sandbox": {
    "enabled": true,
    "mode": "strict",
    "allowNetwork": false,
    "deniedPaths": [
      "~/.ssh",
      "~/.aws",
      "~/.gnupg",
      "/etc"
    ]
  },

  "hooks": {
    "PostToolUse": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "./hooks/audit-log.sh"
          }
        ]
      }
    ]
  }
}
```

---

### Template 6: CI/CD Automation

**Workflow:** Headless execution in pipelines.

**Why these settings:**
- Bypass prompts for automation
- Limited tool access
- Fast model for cost efficiency
- No network access for isolation

```json
{
  "model": "haiku",
  "maxTokens": 4096,
  "temperature": 0.3,

  "permissions": {
    "defaultMode": "bypassPermissions",
    "tools": {
      "Bash": {
        "allow": [
          "npm *",
          "yarn *",
          "node *",
          "python *",
          "pytest *",
          "jest *",
          "cargo test *",
          "go test *",
          "make *",
          "git diff *",
          "git log *"
        ],
        "deny": [
          "*sudo*",
          "*rm -rf*",
          "docker push *",
          "npm publish *",
          "git push *",
          "*deploy*",
          "*production*"
        ]
      },
      "Read": { "defaultMode": "allow" },
      "Write": { "defaultMode": "allow" },
      "Edit": { "defaultMode": "allow" },
      "Glob": { "defaultMode": "allow" },
      "Grep": { "defaultMode": "allow" },
      "WebFetch": { "defaultMode": "deny" },
      "WebSearch": { "defaultMode": "deny" }
    }
  },

  "sandbox": {
    "enabled": true,
    "mode": "strict",
    "allowNetwork": false
  },

  "autoCompact": true,
  "compactThreshold": 40000,
  "outputStyle": "concise"
}
```

---

### Template 7: Code Review Mode

**Workflow:** Read-only analysis and review.

**Why these settings:**
- Plan mode prevents changes
- Read-only tools allowed
- Opus for deep analysis
- Extended thinking for complex review

```json
{
  "model": "opus",
  "maxTokens": 16384,
  "maxThinkingTokens": 16384,

  "permissions": {
    "defaultMode": "plan",
    "tools": {
      "Read": { "defaultMode": "allow" },
      "Glob": { "defaultMode": "allow" },
      "Grep": { "defaultMode": "allow" },
      "Write": { "defaultMode": "deny" },
      "Edit": { "defaultMode": "deny" },
      "Bash": {
        "defaultMode": "ask",
        "allow": [
          "git log *",
          "git diff *",
          "git show *",
          "git blame *"
        ],
        "deny": ["*"]
      }
    }
  },

  "autoCompact": true,
  "compactThreshold": 120000,
  "outputStyle": "verbose"
}
```

---

### Template 8: Monorepo / Large Codebase

**Workflow:** Optimized for large codebases with many MCP tools.

**Why these settings:**
- Aggressive MCP tool deferral
- High compaction threshold
- Subagent permissions
- Multiple MCP servers

```json
{
  "model": "sonnet",
  "maxTokens": 8192,

  "permissions": {
    "tools": {
      "Read": { "defaultMode": "allow" },
      "Glob": { "defaultMode": "allow" },
      "Grep": { "defaultMode": "allow" },
      "Write": { "defaultMode": "ask" },
      "Edit": { "defaultMode": "ask" },
      "Task(Explore)": { "defaultMode": "allow" },
      "Task(Plan)": { "defaultMode": "allow" },
      "Bash": {
        "allow": [
          "nx *",
          "turbo *",
          "pnpm *",
          "yarn workspace *",
          "lerna *"
        ]
      },
      "mcp__*": { "defaultMode": "ask" }
    }
  },

  "mcp": {
    "servers": {
      "workspace-a": {
        "command": "node",
        "args": ["./packages/a/tools/mcp-server.js"]
      },
      "workspace-b": {
        "command": "node",
        "args": ["./packages/b/tools/mcp-server.js"]
      }
    }
  },

  "mcpToolSearch": "auto:5",
  "autoCompact": true,
  "compactThreshold": 100000
}
```

---

### Template 9: Enterprise / Team Settings

**Workflow:** Consistent settings across a development team.

**Why these settings:**
- Enforced security policies
- Audit logging
- Standardized model usage
- Deployment command restrictions

```json
{
  "model": "claude-sonnet-4-20250514",
  "modelAliases": {
    "review": "claude-opus-4-5-20251101",
    "quick": "claude-haiku-3-5-20241022"
  },

  "permissions": {
    "defaultMode": "ask",
    "tools": {
      "Bash": {
        "defaultMode": "ask",
        "allow": [
          "git *",
          "npm *",
          "yarn *",
          "pnpm *",
          "docker-compose *",
          "make *",
          "./scripts/dev-*.sh",
          "./scripts/test-*.sh"
        ],
        "deny": [
          "*sudo*",
          "*rm -rf*",
          "./scripts/deploy-*.sh",
          "./scripts/prod-*.sh",
          "docker push *",
          "npm publish *",
          "kubectl apply *",
          "*production*"
        ]
      },
      "Read": {
        "defaultMode": "allow",
        "deny": [".env.production*"]
      },
      "Write": { "defaultMode": "ask" },
      "Edit": { "defaultMode": "ask" },
      "Glob": { "defaultMode": "allow" },
      "Grep": { "defaultMode": "allow" }
    }
  },

  "hooks": {
    "PostToolUse": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/hooks/team-audit-log.sh"
          }
        ]
      }
    ]
  },

  "autoCompact": true,
  "compactThreshold": 80000
}
```

---

### Template 10: Japanese Development Environment

**Workflow:** Full CJK support with proper IME handling.

**Why these settings:**
- Japanese language responses
- IME enabled for input
- Wide character handling
- Language-aware navigation

```json
{
  "model": "sonnet",
  "language": "ja",

  "terminal": {
    "input": {
      "enableIME": true,
      "compositionTimeout": 500
    },
    "unicodeWidth": "ambiguous-wide",
    "wordBoundaries": "language-aware"
  },

  "permissions": {
    "tools": {
      "Read": { "defaultMode": "allow" },
      "Glob": { "defaultMode": "allow" },
      "Grep": { "defaultMode": "allow" },
      "Write": { "defaultMode": "ask" },
      "Edit": { "defaultMode": "ask" },
      "Bash": {
        "allow": ["git *", "npm *", "node *"]
      }
    }
  }
}
```

---

## Key Combinations

### By Use Case

| Use Case | Key Settings |
|----------|--------------|
| Maximum safety | `defaultMode: "ask"`, `sandbox.enabled: true`, `sandbox.mode: "strict"` |
| Maximum speed | `model: "haiku"`, `defaultMode: "acceptEdits"`, `outputStyle: "concise"` |
| Deep analysis | `model: "opus"`, `maxThinkingTokens: 16384`, `outputStyle: "verbose"` |
| Automation | `defaultMode: "bypassPermissions"`, `outputStyle: "concise"` |
| Read-only | `defaultMode: "plan"`, `Write/Edit: deny: ["*"]` |

### Security Levels

| Level | Settings |
|-------|----------|
| Minimal | `defaultMode: "default"`, no sandbox |
| Standard | `defaultMode: "ask"` for Bash/Write/Edit |
| Strict | All tools `ask`, sandbox strict |
| Paranoid | All tools `ask`, Bash deny all, sandbox strict, no network |

---

## Quick Reference

### Essential Keys

```json
{
  "model": "sonnet",
  "maxTokens": 8192,
  "temperature": 0.7,
  "permissions": {
    "defaultMode": "default",
    "tools": {}
  },
  "autoCompact": true,
  "compactThreshold": 80000
}
```

### Permission Modes

| Mode | Prompts? | Use Case |
|------|----------|----------|
| `default` | Tool-specific | Normal use |
| `ask` | Always | Security |
| `acceptEdits` | Never for files | Trusted |
| `bypassPermissions` | Never | Automation |
| `plan` | N/A (no exec) | Review |

### v2.1.x Keys

| Key | Version | Description |
|-----|---------|-------------|
| `mcpToolSearch` | v2.1.7 | MCP tool loading |
| `plansDirectory` | v2.1.9 | Plan file storage |
| `showTurnDuration` | v2.1.7 | Turn timing display |
| `language` | v2.1.0 | Response language |
| `releaseChannel` | v2.1.3 | Update channel |
| `respectGitignore` | v2.1.0 | @-mention filtering |
| Hook `once` | v2.1.0 | Single execution |
| Hook `additionalContext` | v2.1.9 | Inject context |

---

## Sources

- [Claude Code Settings](https://code.claude.com/docs/en/settings.md)
- [Claude Code Permissions](https://code.claude.com/docs/en/permissions.md)
- [Claude Code Hooks](https://code.claude.com/docs/en/hooks.md)
- [Claude Code MCP](https://code.claude.com/docs/en/mcp.md)
- [Claude Code Changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

---

*This tutorial is part of the Claude Code documentation series.*
*Version 2.1.x | Last Updated: January 2026*
