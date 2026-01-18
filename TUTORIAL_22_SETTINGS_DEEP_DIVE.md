# Tutorial 22: Settings Deep Dive

## Complete Guide to Claude Code Configuration

**Tutorial Level**: Beginner to Intermediate
**Estimated Time**: 90-120 minutes
**Last Updated**: January 2025

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Core Concepts](#core-concepts)
4. [Settings Hierarchy](#settings-hierarchy)
5. [All Settings Reference](#all-settings-reference)
6. [Complete Templates](#complete-templates)
7. [Real-World Examples](#real-world-examples)
8. [Settings Precedence](#settings-precedence)
9. [Environment Variables](#environment-variables)
10. [Troubleshooting](#troubleshooting)
11. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)
12. [Best Practices](#best-practices)

---

## Overview

### What is the Settings System?

Claude Code's settings system provides a comprehensive way to configure every aspect
of your AI-assisted development experience. From controlling which model responds to
your queries, to fine-tuning permissions and security policies, settings give you
complete control over Claude Code's behavior.

### Why Settings Matter

Settings are the foundation of a productive Claude Code workflow:

- **Personalization**: Tailor Claude Code to match your coding style
- **Security**: Control what operations Claude can perform
- **Consistency**: Share configurations across teams and projects
- **Efficiency**: Optimize performance for different use cases
- **Compliance**: Meet enterprise and organizational requirements

### Settings Scopes

Claude Code supports multiple configuration scopes, each serving different purposes:

```
┌─────────────────────────────────────────────────────────────────┐
│                      CLI FLAGS (Highest)                        │
│  Session-specific overrides, immediate effect                   │
├─────────────────────────────────────────────────────────────────┤
│                    PROJECT SETTINGS                              │
│  .claude/settings.json - Per-project configuration              │
├─────────────────────────────────────────────────────────────────┤
│                     USER SETTINGS                                │
│  ~/.claude/settings.json - Personal defaults                    │
├─────────────────────────────────────────────────────────────────┤
│                    SYSTEM SETTINGS                               │
│  /etc/claude/settings.json - Enterprise/admin managed           │
├─────────────────────────────────────────────────────────────────┤
│                   BUILT-IN DEFAULTS (Lowest)                     │
│  Hard-coded defaults in Claude Code                             │
└─────────────────────────────────────────────────────────────────┘
```

### How Precedence Works

When the same setting exists at multiple levels, higher-priority sources override
lower ones. This allows:

- System admins to enforce policies that users cannot override
- Users to set personal preferences across all projects
- Projects to customize behavior for their specific needs
- CLI flags to make temporary session-specific changes

---

## Prerequisites

Before diving into settings configuration, ensure you have:

### Required

1. **Claude Code Installed**
   ```bash
   # Verify installation
   claude --version

   # Expected output: claude-code v1.x.x or similar
   ```

2. **Authentication Configured**
   ```bash
   # Check authentication status
   claude auth status

   # If needed, authenticate
   claude auth login
   ```

3. **Basic Command Line Skills**
   - Navigating directories
   - Creating and editing files
   - Understanding JSON format

### Recommended

4. **A Text Editor with JSON Support**
   - VS Code, Sublime Text, Vim, etc.
   - JSON syntax highlighting helps catch errors

5. **Understanding of Your Project Structure**
   - Know where your project root is
   - Understand team conventions

### Knowledge Check

Before proceeding, you should be comfortable with:

- [ ] Running Claude Code from the terminal
- [ ] Basic JSON syntax (objects, arrays, strings, numbers)
- [ ] File paths (absolute vs. relative)
- [ ] Your operating system's configuration directories

---

## Core Concepts

### Settings Files: JSON Format

All Claude Code settings files use JSON (JavaScript Object Notation) format.
Here's a quick refresher:

```json
{
  "stringValue": "hello",
  "numberValue": 42,
  "booleanValue": true,
  "nullValue": null,
  "arrayValue": ["one", "two", "three"],
  "nestedObject": {
    "innerKey": "innerValue"
  }
}
```

**JSON Rules to Remember:**
- Keys must be quoted with double quotes
- No trailing commas allowed
- Use `true`/`false` (not `True`/`False`)
- Comments are NOT supported in standard JSON

### Settings File Locations

#### System Settings (Enterprise/Admin)

**Linux/macOS:**
```
/etc/claude/settings.json
```

**Windows:**
```
C:\ProgramData\claude\settings.json
```

These settings are typically managed by system administrators and apply to all
users on the machine. Regular users usually cannot modify these files.

#### User Settings (Personal)

**Linux/macOS:**
```
~/.claude/settings.json
```

**Windows:**
```
%USERPROFILE%\.claude\settings.json
C:\Users\YourUsername\.claude\settings.json
```

These are your personal defaults that apply to all projects unless overridden.

#### Project Settings (Per-Project)

```
<project-root>/.claude/settings.json
```

These settings apply only when working within a specific project directory.
They should be version-controlled and shared with your team.

### Creating Your First Settings File

Let's create a basic user settings file:

**Step 1: Create the directory (if it doesn't exist)**

```bash
# Linux/macOS
mkdir -p ~/.claude

# Windows (PowerShell)
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude"
```

**Step 2: Create the settings file**

```bash
# Linux/macOS
touch ~/.claude/settings.json

# Windows (PowerShell)
New-Item -ItemType File -Path "$env:USERPROFILE\.claude\settings.json"
```

**Step 3: Add minimal configuration**

```json
{
  "model": "sonnet"
}
```

**Step 4: Verify settings are loaded**

```bash
claude config show
```

### Settings Inheritance

Settings inherit from lower-priority sources and can be selectively overridden:

```
System Settings:     { "model": "haiku", "maxTokens": 2048 }
                              ↓
User Settings:       { "model": "sonnet" }
                              ↓
Effective for User:  { "model": "sonnet", "maxTokens": 2048 }
                              ↓
Project Settings:    { "maxTokens": 4096 }
                              ↓
Effective in Project:{ "model": "sonnet", "maxTokens": 4096 }
```

### Deep Merging vs. Replacement

For simple values (strings, numbers, booleans), higher-priority settings replace
lower ones. For objects and arrays, behavior varies:

**Objects are deep-merged:**
```json
// User settings
{
  "permissions": {
    "defaultMode": "ask",
    "tools": {
      "Bash": { "defaultMode": "ask" }
    }
  }
}

// Project settings
{
  "permissions": {
    "tools": {
      "Read": { "defaultMode": "allow" }
    }
  }
}

// Effective settings (merged)
{
  "permissions": {
    "defaultMode": "ask",
    "tools": {
      "Bash": { "defaultMode": "ask" },
      "Read": { "defaultMode": "allow" }
    }
  }
}
```

**Arrays are typically replaced:**
```json
// User settings
{
  "permissions": {
    "allow": ["git *", "npm *"]
  }
}

// Project settings
{
  "permissions": {
    "allow": ["docker *", "make *"]
  }
}

// Effective settings (project replaces user)
{
  "permissions": {
    "allow": ["docker *", "make *"]
  }
}
```

---

## Settings Hierarchy

### Level 1: System Settings

**Location:**
- Linux/macOS: `/etc/claude/settings.json`
- Windows: `C:\ProgramData\claude\settings.json`

**Purpose:** Enterprise-wide policies and defaults

**Typical Uses:**
- Enforcing security policies
- Setting approved model lists
- Configuring enterprise proxies
- Blocking dangerous operations

**Who Manages:** System administrators, IT departments

**Example System Settings:**

```json
{
  "model": "sonnet",
  "permissions": {
    "defaultMode": "ask",
    "deny": [
      "sudo *",
      "rm -rf /*",
      "chmod 777 *",
      "curl * | sh",
      "wget * | sh"
    ]
  },
  "sandbox": {
    "enabled": true,
    "mode": "strict"
  },
  "proxy": "http://corporate-proxy.example.com:8080",
  "marketplace": "https://internal-marketplace.example.com"
}
```

**Key Characteristics:**
- Requires admin/root privileges to modify
- Cannot be overridden by user or project settings (for certain locked keys)
- Applies to all users on the system

### Level 2: User Settings

**Location:**
- Linux/macOS: `~/.claude/settings.json`
- Windows: `%USERPROFILE%\.claude\settings.json`

**Purpose:** Personal preferences and defaults

**Typical Uses:**
- Preferred model selection
- Personal permission preferences
- UI customization
- Default behaviors

**Who Manages:** Individual developers

**Example User Settings:**

```json
{
  "model": "sonnet",
  "modelAliases": {
    "fast": "haiku",
    "smart": "opus",
    "default": "sonnet"
  },
  "maxTokens": 4096,
  "temperature": 0.7,

  "permissions": {
    "defaultMode": "default",
    "tools": {
      "Read": { "defaultMode": "allow" },
      "Glob": { "defaultMode": "allow" },
      "Grep": { "defaultMode": "allow" },
      "Bash": {
        "defaultMode": "ask",
        "allow": [
          "git *",
          "npm *",
          "yarn *",
          "pnpm *",
          "node *"
        ]
      },
      "Write": { "defaultMode": "ask" },
      "Edit": { "defaultMode": "ask" }
    }
  },

  "autoCompact": true,
  "compactThreshold": 80000,

  "notifications": {
    "taskComplete": true,
    "errors": true,
    "sound": false
  },

  "theme": "auto"
}
```

**Key Characteristics:**
- User has full control
- Applies to all projects by default
- Can be overridden by project settings and CLI flags

### Level 3: Project Settings

**Location:** `<project-root>/.claude/settings.json`

**Purpose:** Project-specific configuration

**Typical Uses:**
- Project-appropriate model selection
- Project-specific permissions
- MCP server configuration
- Hook definitions
- Team-shared settings

**Who Manages:** Project maintainers, team leads

**Example Project Settings:**

```json
{
  "model": "opus",

  "permissions": {
    "tools": {
      "Bash": {
        "allow": [
          "docker-compose *",
          "docker build *",
          "docker run *",
          "./scripts/*",
          "make *",
          "pytest *",
          "cargo *"
        ],
        "deny": [
          "docker push *"
        ]
      }
    }
  },

  "mcp": {
    "servers": {
      "database": {
        "command": "node",
        "args": ["./tools/db-server.js"],
        "env": {
          "DB_HOST": "localhost"
        }
      },
      "documentation": {
        "command": "python",
        "args": ["-m", "doc_server"],
        "cwd": "./docs"
      }
    }
  },

  "hooks": {
    "postToolUse": {
      "Bash": "./hooks/log-commands.sh",
      "Write": "./hooks/format-on-write.sh"
    },
    "preSession": "./hooks/setup-env.sh"
  }
}
```

**Key Characteristics:**
- Scoped to the specific project
- Should be version-controlled (add to git)
- Shared with team members
- Overrides user settings within the project

### Level 4: CLI Flags

**Purpose:** Session-specific overrides

**Typical Uses:**
- Temporary model changes
- One-time permission overrides
- Debugging and testing
- Scripted automation

**Common CLI Flags:**

```bash
# Model selection
claude --model opus
claude --model haiku
claude --model sonnet

# Permission modes
claude --permission-mode ask
claude --permission-mode acceptEdits
claude --permission-mode bypassPermissions

# Tool restrictions
claude --allowedTools "Read,Glob,Grep"
claude --disallowedTools "Bash,Write"

# Custom settings file
claude --settings ./custom-settings.json

# Debug mode
claude --debug

# Disable specific features
claude --no-sandbox
claude --no-hooks
```

**Key Characteristics:**
- Highest priority (overrides everything)
- Only affects the current session
- Not persisted anywhere
- Ideal for testing and one-off operations

---

## All Settings Reference

### Model Settings

#### `model`

**Type:** `string`
**Default:** `"sonnet"`
**Scope:** All levels

Specifies the default Claude model to use for conversations.

```json
{
  "model": "opus"
}
```

**Valid Values:**
- `"opus"` - Most capable, best for complex tasks
- `"sonnet"` - Balanced performance and cost
- `"haiku"` - Fastest, most economical

**Use Cases:**
- `"opus"` - Architecture decisions, complex debugging, code review
- `"sonnet"` - General development, code generation, refactoring
- `"haiku"` - Quick questions, simple edits, rapid iteration

---

#### `modelAliases`

**Type:** `object`
**Default:** `{}`
**Scope:** User, Project

Create shortcuts for model names or specific model versions.

```json
{
  "modelAliases": {
    "fast": "haiku",
    "smart": "opus",
    "default": "sonnet",
    "latest-opus": "claude-opus-4-5-20251101",
    "stable": "claude-3-5-sonnet-20241022"
  }
}
```

**Usage:**
```bash
claude --model fast    # Uses haiku
claude --model smart   # Uses opus
```

---

#### `maxTokens`

**Type:** `number`
**Default:** `4096`
**Scope:** All levels

Maximum number of tokens in Claude's response.

```json
{
  "maxTokens": 8192
}
```

**Considerations:**
- Higher values allow longer responses
- May increase latency and cost
- Some models have maximum limits
- Typical range: 1024 - 16384

---

#### `maxThinkingTokens`

**Type:** `number`
**Default:** `8192`
**Scope:** All levels

Maximum tokens for extended thinking (chain-of-thought reasoning).

```json
{
  "maxThinkingTokens": 16384
}
```

**When to Increase:**
- Complex multi-step problems
- Debugging difficult issues
- Architecture planning
- Code review with many files

---

#### `temperature`

**Type:** `number`
**Default:** `0.7`
**Scope:** All levels
**Range:** `0.0` to `1.0`

Controls randomness/creativity in responses.

```json
{
  "temperature": 0.5
}
```

**Guidelines:**
- `0.0 - 0.3` - Deterministic, precise (debugging, code review)
- `0.4 - 0.6` - Balanced (general development)
- `0.7 - 0.9` - Creative (brainstorming, design)
- `1.0` - Maximum creativity (may be less coherent)

---

### Permission Settings

#### `permissions.defaultMode`

**Type:** `string`
**Default:** `"default"`
**Scope:** All levels

Sets the default permission behavior for all tools.

```json
{
  "permissions": {
    "defaultMode": "ask"
  }
}
```

**Valid Values:**

| Mode | Description | Use Case |
|------|-------------|----------|
| `"default"` | Use tool-specific defaults | Normal development |
| `"ask"` | Always prompt for confirmation | Security-conscious |
| `"acceptEdits"` | Auto-approve file modifications | Trusted projects |
| `"bypassPermissions"` | Skip all confirmations | Automation, CI/CD |
| `"plan"` | Show plan without executing | Review mode |

---

#### `permissions.tools`

**Type:** `object`
**Default:** `{}`
**Scope:** All levels

Configure permissions for individual tools.

```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "defaultMode": "ask",
        "allow": ["git *", "npm *"],
        "deny": ["sudo *", "rm -rf *"]
      },
      "Read": {
        "defaultMode": "allow"
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
      },
      "WebFetch": {
        "defaultMode": "ask"
      }
    }
  }
}
```

**Tool Names:**
- `Bash` - Shell command execution
- `Read` - File reading
- `Write` - File creation
- `Edit` - File modification
- `Glob` - File pattern matching
- `Grep` - Content searching
- `WebFetch` - Web content retrieval
- `WebSearch` - Web searching
- `NotebookEdit` - Jupyter notebook editing
- `TodoWrite` - Task list management
- `Skill` - Skill invocation
- `Task` - Sub-agent tasks

---

#### `permissions.allow`

**Type:** `array<string>`
**Default:** `[]`
**Scope:** All levels

Global list of allowed command patterns.

```json
{
  "permissions": {
    "allow": [
      "git *",
      "npm *",
      "yarn *",
      "pnpm *",
      "node *",
      "python *",
      "make *"
    ]
  }
}
```

**Pattern Syntax:**
- `*` - Matches any characters
- Exact strings match literally
- Patterns are case-sensitive

---

#### `permissions.deny`

**Type:** `array<string>`
**Default:** `[]`
**Scope:** All levels

Global list of denied command patterns. Takes precedence over allow.

```json
{
  "permissions": {
    "deny": [
      "sudo *",
      "rm -rf /*",
      "rm -rf ~/*",
      "chmod 777 *",
      "> /dev/*",
      "dd if=*",
      "mkfs.*",
      "curl * | sh",
      "curl * | bash",
      "wget * | sh",
      "wget * | bash"
    ]
  }
}
```

---

### Behavior Settings

#### `autoCompact`

**Type:** `boolean`
**Default:** `true`
**Scope:** All levels

Automatically compress conversation context when it gets large.

```json
{
  "autoCompact": true
}
```

**Benefits:**
- Prevents context window exhaustion
- Maintains conversation continuity
- Reduces costs for long sessions

---

#### `compactThreshold`

**Type:** `number`
**Default:** `80000`
**Scope:** All levels

Token count threshold that triggers auto-compaction.

```json
{
  "compactThreshold": 50000
}
```

**Guidelines:**
- Lower values = more frequent compaction, less context
- Higher values = less compaction, more memory usage
- Typical range: 40000 - 120000

---

#### `outputStyle`

**Type:** `string`
**Default:** `"default"`
**Scope:** All levels

Controls the formatting style of Claude's responses.

```json
{
  "outputStyle": "concise"
}
```

**Valid Values:**
- `"default"` - Balanced explanations
- `"concise"` - Minimal explanations
- `"verbose"` - Detailed explanations
- `"technical"` - Assume technical knowledge

---

#### `keepCodingInstructions`

**Type:** `boolean`
**Default:** `true`
**Scope:** All levels

Preserve coding context and instructions across compactions.

```json
{
  "keepCodingInstructions": true
}
```

---

### UI/UX Settings

#### `theme`

**Type:** `string`
**Default:** `"auto"`
**Scope:** User

Color theme for the terminal interface.

```json
{
  "theme": "dark"
}
```

**Valid Values:**
- `"auto"` - Follow system preference
- `"dark"` - Dark background theme
- `"light"` - Light background theme

---

#### `notifications`

**Type:** `object`
**Default:** `{}`
**Scope:** User

Configure notification behavior.

```json
{
  "notifications": {
    "taskComplete": true,
    "errors": true,
    "warnings": true,
    "sound": false,
    "desktop": true
  }
}
```

**Options:**
- `taskComplete` - Notify when long tasks finish
- `errors` - Notify on errors
- `warnings` - Notify on warnings
- `sound` - Play audio notifications
- `desktop` - Show desktop notifications

---

#### `statusLine`

**Type:** `object`
**Default:** `{}`
**Scope:** User

Configure the status bar display.

```json
{
  "statusLine": {
    "show": true,
    "position": "bottom",
    "items": ["model", "tokens", "cost", "time"]
  }
}
```

---

### Integration Settings

#### `sandbox`

**Type:** `object`
**Default:** `{}`
**Scope:** All levels

Configure sandboxing for command execution.

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
      "/var",
      "/root"
    ]
  }
}
```

**Modes:**
- `"off"` - No sandboxing
- `"permissive"` - Basic isolation
- `"strict"` - Full isolation
- `"custom"` - User-defined rules

---

#### `mcp`

**Type:** `object`
**Default:** `{}`
**Scope:** Project, User

Model Context Protocol server configuration.

```json
{
  "mcp": {
    "servers": {
      "database": {
        "command": "node",
        "args": ["./tools/db-server.js"],
        "env": {
          "DB_HOST": "localhost",
          "DB_PORT": "5432"
        },
        "cwd": "./tools"
      },
      "filesystem": {
        "command": "npx",
        "args": ["-y", "@anthropic/mcp-server-filesystem", "/path/to/allowed"]
      },
      "github": {
        "command": "npx",
        "args": ["-y", "@anthropic/mcp-server-github"],
        "env": {
          "GITHUB_TOKEN": "${GITHUB_TOKEN}"
        }
      }
    },
    "timeout": 30000,
    "retries": 3
  }
}
```

**Server Configuration Options:**
- `command` - Executable to run
- `args` - Command-line arguments
- `env` - Environment variables
- `cwd` - Working directory

---

#### `hooks`

**Type:** `object`
**Default:** `{}`
**Scope:** Project

Define custom scripts to run at specific points.

```json
{
  "hooks": {
    "preSession": "./hooks/setup.sh",
    "postSession": "./hooks/cleanup.sh",
    "preToolUse": {
      "Bash": "./hooks/validate-command.sh",
      "Write": "./hooks/check-path.sh"
    },
    "postToolUse": {
      "Bash": "./hooks/log-command.sh",
      "Write": "./hooks/format-file.sh",
      "Edit": "./hooks/lint-file.sh"
    },
    "onError": "./hooks/notify-error.sh"
  }
}
```

**Hook Types:**
- `preSession` - Before conversation starts
- `postSession` - After conversation ends
- `preToolUse` - Before a tool executes
- `postToolUse` - After a tool executes
- `onError` - When an error occurs

---

#### `plugins`

**Type:** `object`
**Default:** `{}`
**Scope:** User, Project

Configure plugins and extensions.

```json
{
  "plugins": {
    "enabled": [
      "code-formatter",
      "git-integration",
      "test-runner"
    ],
    "disabled": [
      "experimental-feature"
    ],
    "config": {
      "code-formatter": {
        "style": "prettier",
        "autoFormat": true
      }
    }
  }
}
```

---

#### `marketplace`

**Type:** `string`
**Default:** `"https://marketplace.anthropic.com"`
**Scope:** System, User

URL of the plugin/skill marketplace.

```json
{
  "marketplace": "https://internal-marketplace.company.com"
}
```

---

### Network Settings

#### `proxy`

**Type:** `string | null`
**Default:** `null`
**Scope:** All levels

HTTP/HTTPS proxy for API requests.

```json
{
  "proxy": "http://proxy.company.com:8080"
}
```

**Formats:**
- `"http://proxy:port"`
- `"https://proxy:port"`
- `"http://user:pass@proxy:port"`
- `null` - No proxy

---

#### `timeout`

**Type:** `number`
**Default:** `30000`
**Scope:** All levels

Request timeout in milliseconds.

```json
{
  "timeout": 60000
}
```

**Guidelines:**
- Default 30000ms (30 seconds)
- Increase for slow networks
- Increase for complex operations

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

## Complete Templates

### Template 1: Complete User Settings

```json
// ~/.claude/settings.json
// Comprehensive user configuration with all common options
{
  "model": "sonnet",
  "modelAliases": {
    "fast": "haiku",
    "smart": "opus",
    "default": "sonnet",
    "thinking": "opus"
  },
  "maxTokens": 4096,
  "maxThinkingTokens": 8192,
  "temperature": 0.7,

  "permissions": {
    "defaultMode": "default",
    "tools": {
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
          "make *",
          "cmake *"
        ],
        "deny": [
          "sudo *",
          "rm -rf /*",
          "rm -rf ~/*",
          "chmod 777 *",
          "> /dev/*",
          "curl * | sh",
          "wget * | bash"
        ]
      },
      "Read": { "defaultMode": "allow" },
      "Write": { "defaultMode": "ask" },
      "Edit": { "defaultMode": "ask" },
      "Glob": { "defaultMode": "allow" },
      "Grep": { "defaultMode": "allow" },
      "WebFetch": { "defaultMode": "ask" },
      "WebSearch": { "defaultMode": "allow" }
    }
  },

  "autoCompact": true,
  "compactThreshold": 80000,
  "outputStyle": "default",
  "keepCodingInstructions": true,

  "sandbox": {
    "enabled": true,
    "mode": "permissive"
  },

  "notifications": {
    "taskComplete": true,
    "errors": true,
    "warnings": false,
    "sound": false,
    "desktop": true
  },

  "theme": "auto",

  "statusLine": {
    "show": true,
    "items": ["model", "tokens", "cost"]
  },

  "proxy": null,
  "timeout": 30000,
  "retries": {
    "maxAttempts": 3,
    "initialDelay": 1000,
    "maxDelay": 10000
  }
}
```

---

### Template 2: Project Settings

```json
// .claude/settings.json
// Project-specific configuration for a web application
{
  "model": "opus",

  "permissions": {
    "tools": {
      "Bash": {
        "allow": [
          "docker-compose *",
          "docker build *",
          "docker run *",
          "docker exec *",
          "docker logs *",
          "./scripts/*",
          "make *",
          "npm run *",
          "yarn *",
          "pnpm *",
          "jest *",
          "vitest *",
          "playwright *",
          "eslint *",
          "prettier *"
        ],
        "deny": [
          "docker push *",
          "docker login *"
        ]
      }
    }
  },

  "mcp": {
    "servers": {
      "database": {
        "command": "node",
        "args": ["./tools/db-server.js"],
        "env": {
          "DB_HOST": "localhost",
          "DB_PORT": "5432",
          "DB_NAME": "devdb"
        }
      },
      "documentation": {
        "command": "python",
        "args": ["-m", "doc_server"],
        "cwd": "./docs"
      },
      "api-spec": {
        "command": "npx",
        "args": ["openapi-mcp-server", "./api/openapi.yaml"]
      }
    }
  },

  "hooks": {
    "preSession": "./hooks/setup-env.sh",
    "postToolUse": {
      "Bash": "./hooks/log-commands.sh",
      "Write": "./hooks/format-on-write.sh",
      "Edit": "./hooks/lint-on-edit.sh"
    }
  }
}
```

---

### Template 3: Minimal Secure Settings

```json
// Minimal configuration with security focus
{
  "permissions": {
    "defaultMode": "ask",
    "tools": {
      "Bash": {
        "defaultMode": "ask",
        "allow": [],
        "deny": [
          "sudo *",
          "rm -rf *",
          "chmod *",
          "chown *",
          "curl * | *",
          "wget * | *",
          "> /dev/*",
          "dd *",
          "mkfs *",
          "mount *",
          "umount *"
        ]
      },
      "Write": { "defaultMode": "ask" },
      "Edit": { "defaultMode": "ask" },
      "Read": { "defaultMode": "ask" },
      "Glob": { "defaultMode": "ask" },
      "Grep": { "defaultMode": "ask" },
      "WebFetch": { "defaultMode": "ask" },
      "WebSearch": { "defaultMode": "ask" }
    }
  },

  "sandbox": {
    "enabled": true,
    "mode": "strict",
    "allowNetwork": false,
    "allowedPaths": [],
    "deniedPaths": [
      "/etc",
      "/var",
      "/root",
      "/home/*/.*"
    ]
  },

  "autoCompact": true,
  "compactThreshold": 50000
}
```

---

### Template 4: Performance-Optimized Settings

```json
// Configuration optimized for speed and efficiency
{
  "model": "haiku",
  "maxTokens": 2048,
  "maxThinkingTokens": 4096,
  "temperature": 0.5,

  "permissions": {
    "defaultMode": "acceptEdits",
    "tools": {
      "Bash": {
        "defaultMode": "acceptEdits",
        "allow": ["*"],
        "deny": ["sudo *", "rm -rf /*"]
      },
      "Read": { "defaultMode": "allow" },
      "Write": { "defaultMode": "allow" },
      "Edit": { "defaultMode": "allow" },
      "Glob": { "defaultMode": "allow" },
      "Grep": { "defaultMode": "allow" }
    }
  },

  "autoCompact": true,
  "compactThreshold": 50000,
  "outputStyle": "concise",

  "notifications": {
    "taskComplete": false,
    "errors": true
  },

  "timeout": 15000
}
```

---

### Template 5: CLI Flag Overrides

```bash
#!/bin/bash
# Common CLI flag patterns for different use cases

# === Model Selection ===
# Use Opus for complex tasks
claude --model opus

# Use Haiku for quick operations
claude --model haiku

# Use a specific model version
claude --model claude-opus-4-5-20251101

# Use an alias (requires alias in settings)
claude --model smart

# === Permission Modes ===
# Maximum security - ask for everything
claude --permission-mode ask

# Balanced - auto-approve file edits only
claude --permission-mode acceptEdits

# Automation mode - skip all prompts (use carefully!)
claude --permission-mode bypassPermissions

# Review mode - show plan without executing
claude --permission-mode plan

# === Tool Restrictions ===
# Read-only mode
claude --allowedTools "Read,Glob,Grep"

# No shell access
claude --disallowedTools "Bash"

# Specific tools only
claude --allowedTools "Read,Write,Edit,Glob,Grep"

# === Custom Configuration ===
# Use a specific settings file
claude --settings ./configs/production.json

# Override specific settings
claude --settings ./configs/base.json --model opus

# === Debugging ===
# Enable debug output
claude --debug

# Verbose logging
claude --debug --verbose

# === Sandbox Control ===
# Disable sandbox (use carefully!)
claude --no-sandbox

# Disable hooks
claude --no-hooks

# === Combined Examples ===
# Fast iteration mode
claude --model haiku --permission-mode acceptEdits

# Secure review mode
claude --model opus --permission-mode ask --allowedTools "Read,Glob,Grep"

# CI/CD automation
claude --model sonnet --permission-mode bypassPermissions --no-hooks

# Custom project config with model override
claude --settings ./.claude/ci-settings.json --model haiku
```

---

## Real-World Examples

### Example 1: Developer Workstation Configuration

**Scenario:** A full-stack developer working on multiple projects with different
tech stacks needs a flexible configuration.

**User Settings (`~/.claude/settings.json`):**

```json
{
  "model": "sonnet",
  "modelAliases": {
    "quick": "haiku",
    "deep": "opus",
    "review": "opus"
  },
  "maxTokens": 4096,
  "temperature": 0.7,

  "permissions": {
    "defaultMode": "default",
    "tools": {
      "Bash": {
        "defaultMode": "ask",
        "allow": [
          "git *",
          "npm *", "yarn *", "pnpm *",
          "node *", "npx *",
          "python *", "pip *", "poetry *",
          "cargo *", "rustc *",
          "go *",
          "docker *",
          "kubectl *",
          "aws *", "gcloud *", "az *",
          "terraform *",
          "make *", "cmake *"
        ],
        "deny": [
          "sudo *",
          "rm -rf /*",
          "rm -rf ~/*",
          "chmod 777 *"
        ]
      },
      "Read": { "defaultMode": "allow" },
      "Glob": { "defaultMode": "allow" },
      "Grep": { "defaultMode": "allow" },
      "Write": { "defaultMode": "ask" },
      "Edit": { "defaultMode": "ask" }
    }
  },

  "autoCompact": true,
  "compactThreshold": 80000,

  "notifications": {
    "taskComplete": true,
    "errors": true,
    "desktop": true
  },

  "theme": "dark"
}
```

---

### Example 2: CI/CD Pipeline Settings

**Scenario:** Automated builds and deployments need unattended operation.

**CI Settings (`.claude/ci-settings.json`):**

```json
{
  "model": "sonnet",
  "maxTokens": 4096,
  "temperature": 0.3,

  "permissions": {
    "defaultMode": "bypassPermissions",
    "tools": {
      "Bash": {
        "defaultMode": "allow",
        "allow": [
          "npm *", "yarn *", "pnpm *",
          "node *",
          "docker build *",
          "docker-compose *",
          "make *",
          "pytest *",
          "jest *",
          "cargo test *",
          "go test *",
          "git *"
        ],
        "deny": [
          "sudo *",
          "rm -rf /*",
          "docker push *",
          "docker login *",
          "aws *",
          "gcloud *",
          "kubectl apply *"
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
  "compactThreshold": 50000,
  "outputStyle": "concise",

  "notifications": {
    "taskComplete": false,
    "errors": false
  },

  "timeout": 60000
}
```

**CI Pipeline Usage:**

```yaml
# .github/workflows/claude-review.yml
name: Claude Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Claude Code Review
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          claude --settings ./.claude/ci-settings.json \
                 --model sonnet \
                 --permission-mode bypassPermissions \
                 "Review the changes in this PR and provide feedback"
```

---

### Example 3: Enterprise Managed Settings

**Scenario:** IT department needs to enforce security policies across all users.

**System Settings (`/etc/claude/settings.json`):**

```json
{
  "model": "sonnet",

  "permissions": {
    "defaultMode": "ask",
    "deny": [
      "sudo *",
      "su *",
      "rm -rf /*",
      "rm -rf /home/*",
      "chmod 777 *",
      "chmod -R 777 *",
      "chown -R *:* /*",
      "> /dev/*",
      "dd if=*",
      "mkfs.*",
      "fdisk *",
      "parted *",
      "curl * | sh",
      "curl * | bash",
      "wget * -O - | sh",
      "wget * -O - | bash",
      "eval $(*)",
      "bash -c \"$(curl *)\""
    ]
  },

  "sandbox": {
    "enabled": true,
    "mode": "strict",
    "allowNetwork": true,
    "deniedPaths": [
      "/etc/passwd",
      "/etc/shadow",
      "/etc/sudoers",
      "/root",
      "/home/*/.ssh",
      "/home/*/.aws",
      "/home/*/.config/gcloud"
    ]
  },

  "proxy": "http://corporate-proxy.example.com:8080",
  "marketplace": "https://internal-marketplace.example.com",

  "timeout": 60000,
  "retries": {
    "maxAttempts": 5,
    "initialDelay": 1000,
    "maxDelay": 30000
  }
}
```

---

### Example 4: Open Source Project Settings

**Scenario:** An open source project needs contributor-friendly settings.

**Project Settings (`.claude/settings.json`):**

```json
{
  "model": "sonnet",

  "permissions": {
    "defaultMode": "default",
    "tools": {
      "Bash": {
        "defaultMode": "ask",
        "allow": [
          "git *",
          "npm *", "yarn *", "pnpm *",
          "node *",
          "make *",
          "cargo *",
          "pytest *", "python -m pytest *",
          "jest *",
          "./scripts/*.sh",
          "pre-commit *"
        ]
      },
      "Read": { "defaultMode": "allow" },
      "Glob": { "defaultMode": "allow" },
      "Grep": { "defaultMode": "allow" }
    }
  },

  "hooks": {
    "postToolUse": {
      "Write": "./scripts/hooks/format-check.sh",
      "Edit": "./scripts/hooks/lint-check.sh"
    }
  },

  "autoCompact": true,
  "compactThreshold": 60000
}
```

**Contributing Guide Addition:**

```markdown
## Using Claude Code

This project includes Claude Code settings in `.claude/settings.json`.
When you use Claude Code in this repository, these settings will be
automatically applied.

### Quick Start

```bash
# Clone the repository
git clone https://github.com/example/project.git
cd project

# Use Claude Code with project settings
claude "Help me understand the codebase structure"
```

The project settings enable common development commands while maintaining
security through permission prompts for file modifications.
```

---

### Example 5: Security-Conscious Configuration

**Scenario:** A security researcher needs maximum protection.

**User Settings:**

```json
{
  "model": "sonnet",
  "maxTokens": 4096,
  "temperature": 0.5,

  "permissions": {
    "defaultMode": "ask",
    "tools": {
      "Bash": {
        "defaultMode": "ask",
        "allow": [],
        "deny": [
          "*"
        ]
      },
      "Read": {
        "defaultMode": "ask",
        "deny": [
          "*/.env*",
          "*/.ssh/*",
          "*/.aws/*",
          "*/.config/gcloud/*",
          "*/credentials*",
          "*/secrets*",
          "*/*.pem",
          "*/*.key"
        ]
      },
      "Write": { "defaultMode": "ask" },
      "Edit": { "defaultMode": "ask" },
      "Glob": { "defaultMode": "ask" },
      "Grep": {
        "defaultMode": "ask",
        "deny": [
          "*/.env*",
          "*/credentials*",
          "*/secrets*"
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
    "allowedPaths": [],
    "deniedPaths": [
      "~/.ssh",
      "~/.aws",
      "~/.config/gcloud",
      "~/.gnupg",
      "/etc",
      "/var"
    ]
  },

  "autoCompact": true,
  "compactThreshold": 40000,

  "notifications": {
    "taskComplete": true,
    "errors": true
  }
}
```

---

### Example 6: Performance-Optimized Team Settings

**Scenario:** A team needs fast iteration for a hackathon or rapid prototyping.

**Project Settings:**

```json
{
  "model": "haiku",
  "modelAliases": {
    "think": "opus",
    "review": "sonnet"
  },
  "maxTokens": 2048,
  "maxThinkingTokens": 4096,
  "temperature": 0.6,

  "permissions": {
    "defaultMode": "acceptEdits",
    "tools": {
      "Bash": {
        "defaultMode": "acceptEdits",
        "allow": [
          "git *",
          "npm *", "yarn *",
          "node *",
          "python *",
          "make *",
          "docker-compose up *",
          "docker-compose down *",
          "docker-compose logs *",
          "curl localhost:*",
          "./scripts/*"
        ],
        "deny": [
          "sudo *",
          "rm -rf /*",
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

  "notifications": {
    "taskComplete": false,
    "errors": true
  },

  "timeout": 15000
}
```

---

### Example 7: Team-Shared Project Settings

**Scenario:** A development team needs consistent settings across all members.

**Project Settings with Documentation:**

```json
{
  "_comment": "Team settings - see CONTRIBUTING.md for details",

  "model": "sonnet",
  "modelAliases": {
    "fast": "haiku",
    "smart": "opus"
  },
  "maxTokens": 4096,
  "temperature": 0.7,

  "permissions": {
    "defaultMode": "default",
    "tools": {
      "Bash": {
        "defaultMode": "ask",
        "allow": [
          "git *",
          "npm *", "yarn *", "pnpm *",
          "nx *",
          "docker-compose *",
          "make *",
          "jest *", "vitest *",
          "eslint *", "prettier *",
          "tsc *",
          "./scripts/dev-*.sh",
          "./scripts/test-*.sh"
        ],
        "deny": [
          "sudo *",
          "rm -rf /*",
          "./scripts/deploy-*.sh",
          "./scripts/prod-*.sh",
          "docker push *",
          "npm publish *"
        ]
      },
      "Read": { "defaultMode": "allow" },
      "Glob": { "defaultMode": "allow" },
      "Grep": { "defaultMode": "allow" },
      "Write": { "defaultMode": "ask" },
      "Edit": { "defaultMode": "ask" }
    }
  },

  "mcp": {
    "servers": {
      "project-docs": {
        "command": "node",
        "args": ["./tools/docs-server.js"]
      },
      "api-schema": {
        "command": "npx",
        "args": ["openapi-mcp", "./api/schema.yaml"]
      }
    }
  },

  "hooks": {
    "preSession": "./scripts/hooks/check-env.sh",
    "postToolUse": {
      "Write": "./scripts/hooks/auto-format.sh",
      "Edit": "./scripts/hooks/auto-lint.sh"
    }
  },

  "autoCompact": true,
  "compactThreshold": 80000
}
```

---

## Settings Precedence

### Precedence Table

| Priority | Source | Scope | Managed By | Override Lower? |
|----------|--------|-------|------------|-----------------|
| 1 (Highest) | CLI flags | Session only | Developer | Yes |
| 2 | Project settings | This project | Team | Yes |
| 3 | User settings | All projects | Developer | Yes |
| 4 | System settings | All users | Admin | Yes |
| 5 (Lowest) | Built-in defaults | Universal | Anthropic | N/A |

### Precedence Examples

**Example 1: Model Selection**

```
System:  { "model": "haiku" }      <- Admin wants cost savings
User:    { "model": "sonnet" }     <- Developer prefers balance
Project: { "model": "opus" }       <- Project needs power
CLI:     --model haiku             <- Quick test

Result when using CLI flag: haiku
Result without CLI flag: opus
Result in different project: sonnet
Result for new user: haiku
```

**Example 2: Permission Merging**

```
System:  { "permissions": { "deny": ["sudo *"] } }
User:    { "permissions": { "deny": ["rm -rf *"] } }
Project: { "permissions": { "allow": ["docker *"] } }

Effective: {
  "permissions": {
    "deny": ["sudo *", "rm -rf *"],  <- Merged from system + user
    "allow": ["docker *"]             <- Added by project
  }
}
```

**Example 3: Locked Settings**

Some system settings can be "locked" to prevent user override:

```json
// System settings
{
  "sandbox": {
    "enabled": true,
    "mode": "strict",
    "_locked": true
  }
}
```

When a setting is locked, lower-priority sources cannot override it.

### Viewing Effective Settings

Check what settings are actually in effect:

```bash
# Show all effective settings
claude config show

# Show specific setting
claude config get model

# Show settings with sources
claude config show --verbose

# Show raw settings files
claude config show --raw
```

---

## Environment Variables

### Authentication Variables

| Variable | Purpose | Example |
|----------|---------|---------|
| `ANTHROPIC_API_KEY` | Primary API key | `YOUR_API_KEY_HERE` |
| `ANTHROPIC_API_KEY_FILE` | Path to key file | `/secrets/api-key` |

```bash
# Set API key directly
export ANTHROPIC_API_KEY="YOUR_API_KEY_HERE"

# Use key from file
export ANTHROPIC_API_KEY_FILE="/etc/claude/api-key"

# Windows PowerShell
$env:ANTHROPIC_API_KEY = "YOUR_API_KEY_HERE"
```

### Configuration Variables

| Variable | Purpose | Example |
|----------|---------|---------|
| `CLAUDE_CONFIG` | Custom config path | `./custom-config.json` |
| `CLAUDE_MODEL` | Default model | `opus` |
| `CLAUDE_SETTINGS` | Settings file path | `~/.claude/work.json` |
| `CLAUDE_HOME` | Claude home directory | `~/.claude-work` |

```bash
# Use custom configuration
export CLAUDE_CONFIG="./configs/development.json"

# Override default model
export CLAUDE_MODEL="opus"

# Custom settings location
export CLAUDE_SETTINGS="~/.claude/secure-settings.json"
```

### Debugging Variables

| Variable | Purpose | Values |
|----------|---------|--------|
| `CLAUDE_DEBUG` | Enable debug logging | `1`, `true`, `verbose` |
| `CLAUDE_LOG_LEVEL` | Log verbosity | `error`, `warn`, `info`, `debug` |
| `CLAUDE_LOG_FILE` | Log output file | `/var/log/claude.log` |

```bash
# Enable debug mode
export CLAUDE_DEBUG=1

# Set log level
export CLAUDE_LOG_LEVEL=debug

# Log to file
export CLAUDE_LOG_FILE="/tmp/claude-debug.log"
```

### Network Variables

| Variable | Purpose | Example |
|----------|---------|---------|
| `HTTP_PROXY` | HTTP proxy | `http://proxy:8080` |
| `HTTPS_PROXY` | HTTPS proxy | `https://proxy:8443` |
| `NO_PROXY` | Bypass proxy | `localhost,127.0.0.1` |
| `CLAUDE_TIMEOUT` | Request timeout (ms) | `60000` |

```bash
# Configure proxy
export HTTP_PROXY="http://proxy.company.com:8080"
export HTTPS_PROXY="http://proxy.company.com:8080"
export NO_PROXY="localhost,127.0.0.1,.company.com"

# Set timeout
export CLAUDE_TIMEOUT=60000
```

### Variable Precedence

Environment variables typically have medium precedence:

```
CLI flags > Environment variables > Settings files > Defaults
```

However, this varies by setting. Some settings can only be configured
via settings files, while others (like API keys) are environment-first.

---

## Troubleshooting

### Issue 1: Settings Not Applying

**Symptoms:**
- Changes to settings files have no effect
- Claude uses different settings than expected

**Diagnostic Steps:**

```bash
# 1. Verify settings file exists and is readable
ls -la ~/.claude/settings.json

# 2. Validate JSON syntax
python -m json.tool ~/.claude/settings.json

# 3. Check effective settings
claude config show

# 4. Check for syntax errors
claude config validate
```

**Common Causes:**

1. **Invalid JSON syntax**
   ```json
   // WRONG: Trailing comma
   { "model": "opus", }

   // CORRECT: No trailing comma
   { "model": "opus" }
   ```

2. **Wrong file location**
   ```bash
   # Check you're editing the right file
   echo $HOME  # Should match ~ in path
   ```

3. **File permissions**
   ```bash
   # Fix permissions
   chmod 644 ~/.claude/settings.json
   ```

4. **Higher-priority override**
   ```bash
   # Check for project settings
   ls -la .claude/settings.json

   # Check for CLI flags in aliases
   alias claude
   ```

---

### Issue 2: Precedence Confusion

**Symptoms:**
- Project settings override user settings unexpectedly
- System settings take effect when they shouldn't

**Diagnostic Steps:**

```bash
# 1. Show settings with sources
claude config show --verbose

# 2. Check all settings files
cat /etc/claude/settings.json 2>/dev/null
cat ~/.claude/settings.json
cat .claude/settings.json 2>/dev/null
```

**Understanding Merging:**

```json
// User: ~/.claude/settings.json
{
  "model": "sonnet",
  "permissions": {
    "tools": {
      "Bash": { "allow": ["git *"] }
    }
  }
}

// Project: .claude/settings.json
{
  "model": "opus",
  "permissions": {
    "tools": {
      "Bash": { "allow": ["docker *"] }
    }
  }
}

// Effective (in project):
// - model: "opus" (project overrides user)
// - Bash allow: ["docker *"] (project REPLACES user array)
```

**Note:** Arrays are replaced, not merged. To add to an array, you must
include all desired values in the higher-priority settings.

---

### Issue 3: Invalid JSON

**Symptoms:**
- Error message about JSON parsing
- Settings file ignored entirely

**Diagnostic Steps:**

```bash
# Validate with Python
python -m json.tool ~/.claude/settings.json

# Validate with jq
jq . ~/.claude/settings.json

# Validate with node
node -e "console.log(JSON.parse(require('fs').readFileSync('$HOME/.claude/settings.json')))"
```

**Common JSON Errors:**

```json
// ERROR: Single quotes (use double quotes)
{ 'model': 'opus' }

// ERROR: Unquoted keys
{ model: "opus" }

// ERROR: Trailing commas
{ "model": "opus", "maxTokens": 4096, }

// ERROR: Comments (not allowed in standard JSON)
{
  // This is a comment
  "model": "opus"
}

// ERROR: Missing comma between properties
{ "model": "opus" "maxTokens": 4096 }

// CORRECT:
{
  "model": "opus",
  "maxTokens": 4096
}
```

**JSON Validation Tips:**

1. Use a JSON-aware editor with syntax highlighting
2. Install a JSON linter extension
3. Use online JSON validators for quick checks
4. Run `claude config validate` after changes

---

### Issue 4: Unknown Settings

**Symptoms:**
- Warning about unrecognized settings
- Settings appear to have no effect

**Diagnostic Steps:**

```bash
# Check for typos
claude config validate

# List valid settings
claude config list
```

**Common Typos:**

```json
// WRONG                    // CORRECT
"maxtoken"                  "maxTokens"
"permisions"                "permissions"
"defaultmode"               "defaultMode"
"autocompact"               "autoCompact"
"compactthreshold"          "compactThreshold"
```

**Version Compatibility:**

Some settings may not be available in older versions:

```bash
# Check Claude Code version
claude --version

# Update to latest
npm update -g @anthropic/claude-code
```

---

### Issue 5: Settings Reset

**Symptoms:**
- Settings return to defaults after restart
- Custom settings disappear

**Diagnostic Steps:**

```bash
# 1. Check file still exists
ls -la ~/.claude/settings.json

# 2. Check file wasn't overwritten
stat ~/.claude/settings.json  # Check modification time

# 3. Check for backup
ls -la ~/.claude/settings.json.bak

# 4. Check git status (for project settings)
git status .claude/settings.json
```

**Prevention:**

1. **Version control project settings**
   ```bash
   git add .claude/settings.json
   git commit -m "Add Claude Code project settings"
   ```

2. **Backup user settings**
   ```bash
   cp ~/.claude/settings.json ~/.claude/settings.json.bak
   ```

3. **Use dotfiles management**
   ```bash
   # Example with GNU Stow
   mkdir -p ~/dotfiles/claude/.claude
   mv ~/.claude/settings.json ~/dotfiles/claude/.claude/
   cd ~/dotfiles && stow claude
   ```

---

### Issue 6: Permission Denied Errors

**Symptoms:**
- Cannot read or write settings files
- "Permission denied" errors

**Solutions:**

```bash
# Fix user settings directory permissions
chmod 755 ~/.claude
chmod 644 ~/.claude/settings.json

# Fix project settings permissions
chmod 755 .claude
chmod 644 .claude/settings.json

# Check ownership
ls -la ~/.claude/
chown $USER:$USER ~/.claude/settings.json
```

---

## Quick Reference Cheat Sheet

### Settings Files Locations

```
System:  /etc/claude/settings.json (Linux/macOS)
         C:\ProgramData\claude\settings.json (Windows)

User:    ~/.claude/settings.json (Linux/macOS)
         %USERPROFILE%\.claude\settings.json (Windows)

Project: .claude/settings.json (relative to project root)
```

### Essential Settings

```json
{
  "model": "sonnet|opus|haiku",
  "maxTokens": 4096,
  "temperature": 0.7,
  "permissions": {
    "defaultMode": "default|ask|acceptEdits|bypassPermissions",
    "tools": { ... }
  },
  "autoCompact": true,
  "compactThreshold": 80000
}
```

### CLI Quick Reference

```bash
# Models
claude --model opus
claude --model haiku

# Permissions
claude --permission-mode ask
claude --permission-mode acceptEdits
claude --permission-mode bypassPermissions

# Tools
claude --allowedTools "Read,Glob,Grep"
claude --disallowedTools "Bash"

# Config
claude --settings ./custom.json
claude config show
claude config validate
```

### Environment Variables

```bash
ANTHROPIC_API_KEY=...      # API authentication
CLAUDE_MODEL=opus          # Default model
CLAUDE_CONFIG=./config     # Config path
CLAUDE_DEBUG=1             # Debug logging
HTTP_PROXY=http://...      # Proxy settings
```

### Permission Modes

| Mode | Behavior | Use Case |
|------|----------|----------|
| `default` | Tool-specific defaults | Normal use |
| `ask` | Always prompt | Security |
| `acceptEdits` | Auto-approve edits | Trusted work |
| `bypassPermissions` | Skip all prompts | Automation |
| `plan` | Show plan only | Review |

### Common Tool Patterns

```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "allow": ["git *", "npm *", "make *"],
        "deny": ["sudo *", "rm -rf *"]
      },
      "Read": { "defaultMode": "allow" },
      "Write": { "defaultMode": "ask" }
    }
  }
}
```

---

## Best Practices

### 1. Start with Secure Defaults

Always begin with restrictive settings and relax them as needed:

```json
{
  "permissions": {
    "defaultMode": "ask",
    "tools": {
      "Bash": {
        "defaultMode": "ask",
        "deny": ["sudo *", "rm -rf *"]
      }
    }
  }
}
```

**Rationale:**
- Prevents accidental destructive operations
- Gives you visibility into what Claude is doing
- Establishes trust before granting more access

### 2. Document Project Settings

Include comments (in a separate file) explaining your choices:

**`.claude/README.md`:**
```markdown
# Claude Code Settings

This project uses Claude Code with custom settings in `.claude/settings.json`.

## Why These Settings?

- **Model: Opus** - This codebase requires deep understanding of complex patterns
- **Docker allowed** - We use Docker for local development
- **Deploy scripts denied** - Deployment should go through CI/CD only

## For New Team Members

The settings will apply automatically when you use Claude Code in this repository.
```

### 3. Review Team Settings Regularly

Schedule periodic reviews of shared settings:

```markdown
## Quarterly Settings Review Checklist

- [ ] Are permission patterns still appropriate?
- [ ] Have new tools been added that need rules?
- [ ] Are there commands we should allow/deny?
- [ ] Is the model choice still optimal?
- [ ] Are MCP servers still needed and configured correctly?
- [ ] Are hooks working as expected?
```

### 4. Version Control Project Settings

Always track project settings in version control:

```bash
# Add to repository
git add .claude/settings.json
git commit -m "Add Claude Code project configuration"

# Include in .gitignore exceptions if needed
echo "!.claude/" >> .gitignore
echo "!.claude/settings.json" >> .gitignore
```

### 5. Use Model Aliases for Consistency

Define aliases to standardize team usage:

```json
{
  "modelAliases": {
    "review": "opus",
    "code": "sonnet",
    "quick": "haiku"
  }
}
```

Then team members use:
```bash
claude --model review   # Everyone gets Opus for reviews
claude --model quick    # Everyone gets Haiku for quick tasks
```

### 6. Separate Settings by Environment

Create environment-specific settings files:

```
.claude/
  settings.json         # Base settings (committed)
  settings.local.json   # Local overrides (gitignored)
  settings.ci.json      # CI/CD settings (committed)
```

**`.gitignore`:**
```
.claude/settings.local.json
```

**Usage:**
```bash
# Local development (uses base + local)
claude

# CI/CD (uses base + ci)
claude --settings .claude/settings.ci.json
```

### 7. Audit Permission Changes

Log when permission settings change:

```bash
# Git hook: .git/hooks/pre-commit
#!/bin/bash

if git diff --cached --name-only | grep -q ".claude/settings.json"; then
    echo "Claude Code settings modified:"
    git diff --cached .claude/settings.json
    read -p "Review changes above. Continue? [y/N] " -n 1 -r
    echo
    if [[ ! $REPLY =~ ^[Yy]$ ]]; then
        exit 1
    fi
fi
```

### 8. Test Settings Before Deployment

Validate settings before committing:

```bash
# Validate JSON syntax
python -m json.tool .claude/settings.json > /dev/null

# Validate Claude Code accepts it
claude config validate --settings .claude/settings.json

# Test with restrictive mode first
claude --settings .claude/settings.json --permission-mode plan \
    "Review the project structure"
```

### 9. Use Hooks for Enforcement

Add hooks to enforce standards:

```json
{
  "hooks": {
    "postToolUse": {
      "Write": "./scripts/enforce-format.sh",
      "Edit": "./scripts/enforce-lint.sh"
    }
  }
}
```

**`scripts/enforce-format.sh`:**
```bash
#!/bin/bash
# Auto-format written files
prettier --write "$CLAUDE_TOOL_OUTPUT_PATH" 2>/dev/null || true
```

### 10. Monitor and Iterate

Track what works and what doesn't:

```json
{
  "hooks": {
    "postToolUse": {
      "Bash": "./scripts/log-commands.sh"
    }
  }
}
```

**`scripts/log-commands.sh`:**
```bash
#!/bin/bash
echo "$(date -Iseconds) $CLAUDE_TOOL_COMMAND" >> ~/.claude/command-history.log
```

Review logs periodically to refine allow/deny patterns:

```bash
# Find most common commands
sort ~/.claude/command-history.log | uniq -c | sort -rn | head -20

# Find blocked commands to consider allowing
grep -l "permission denied" ~/.claude/command-history.log
```

---

## Summary

Claude Code's settings system provides comprehensive control over your AI-assisted
development experience. Key takeaways:

1. **Hierarchy matters** - Understand the four levels (System, User, Project, CLI)
2. **Start secure** - Begin with restrictive settings and relax as needed
3. **Document choices** - Explain why settings are configured a certain way
4. **Version control** - Track project settings with your code
5. **Review regularly** - Settings should evolve with your needs
6. **Test changes** - Validate settings before deploying to the team

With proper configuration, Claude Code becomes a powerful, safe, and consistent
tool for your development workflow.

---

## Next Steps

Now that you understand the settings system:

1. **Create your user settings** - Set up `~/.claude/settings.json`
2. **Configure a project** - Add `.claude/settings.json` to a project
3. **Experiment with models** - Try different models for different tasks
4. **Set up MCP servers** - Extend Claude's capabilities with custom tools
5. **Define hooks** - Automate formatting and validation

For more information, explore:
- [Tutorial 20: MCP Integration](./TUTORIAL_20_MCP_INTEGRATION.md)
- [Tutorial 21: Hooks and Automation](./TUTORIAL_21_HOOKS_AUTOMATION.md)
- [Claude Code Documentation](https://docs.anthropic.com/claude-code)

---

*This tutorial is part of the Claude Code Subagents series. For questions or
feedback, please open an issue in the repository.*
