# Tutorial 20: The Permission System in Claude Code

## A Comprehensive Guide to Security, Control, and Trust

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Core Concepts](#core-concepts)
4. [Understanding Permission Levels](#understanding-permission-levels)
5. [Permission Modes](#permission-modes)
6. [Configuring Permissions](#configuring-permissions)
7. [Tool-Specific Permissions](#tool-specific-permissions)
8. [Permission Precedence](#permission-precedence)
9. [Real-World Examples](#real-world-examples)
10. [Troubleshooting](#troubleshooting)
11. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)
12. [Best Practices](#best-practices)
13. [Summary](#summary)

---

## Overview

### What is the Permission System?

The Permission System in Claude Code is a sophisticated security framework that governs
what actions Claude can perform on your system. Think of it as a security guard that
stands between Claude's capabilities and your computer's resources, ensuring that
every action is authorized, appropriate, and safe.

### Why Permissions Matter

When you invite an AI assistant into your development environment, you're granting
access to powerful tools that can:

- Read and modify files across your filesystem
- Execute shell commands with your user privileges
- Make network requests to external services
- Interact with databases and other resources

Without proper controls, these capabilities could lead to unintended consequences:
accidental file deletions, exposure of sensitive credentials, or execution of
dangerous commands. The Permission System prevents these scenarios while still
allowing Claude to be genuinely helpful.

### The Safety Design Philosophy

Claude Code's permission system is built on three core principles:

1. **Defense in Depth**: Multiple layers of protection ensure that no single
   misconfiguration can lead to security breaches.

2. **Explicit Consent**: By default, potentially impactful actions require your
   explicit approval before execution.

3. **Transparency**: You can always see what Claude wants to do before it happens,
   and you maintain complete control over what's allowed.

### User Control is Paramount

The permission system puts you in the driver's seat. You decide:

- Which tools Claude can use freely
- Which actions require your approval
- Which operations are completely forbidden
- How these rules apply across different projects

This tutorial will teach you everything you need to know to configure permissions
that match your security requirements and workflow preferences.

---

## Prerequisites

Before diving into the permission system, ensure you have the following:

### Required Knowledge

1. **Claude Code Installation**: You should have Claude Code installed and working.
   If not, refer to Tutorial 01 for installation instructions.

2. **Basic Tool Understanding**: Familiarity with Claude Code's core tools:
   - `Bash`: Executes shell commands
   - `Read`: Reads file contents
   - `Write`: Creates or overwrites files
   - `Edit`: Modifies existing files
   - `Glob`: Searches for files by pattern
   - `Grep`: Searches file contents

3. **JSON Syntax**: Permission configurations use JSON format. You should understand:
   - Objects: `{ "key": "value" }`
   - Arrays: `["item1", "item2"]`
   - Nesting: `{ "outer": { "inner": "value" } }`

4. **File System Basics**: Understanding of:
   - Home directory (`~` or `%USERPROFILE%`)
   - Project directories
   - Configuration file locations

### Recommended Background

- Experience with at least one Claude Code session
- Understanding of glob patterns (`*.js`, `src/**/*.ts`)
- Familiarity with your operating system's security model

---

## Core Concepts

### The Three Permission States

Every tool and action in Claude Code exists in one of three permission states:

```
┌─────────────────────────────────────────────────────────────────┐
│                    PERMISSION STATES                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────┐      ┌─────────┐      ┌─────────┐                │
│   │  ALLOW  │      │   ASK   │      │  DENY   │                │
│   │    ✓    │      │    ?    │      │    ✗    │                │
│   └─────────┘      └─────────┘      └─────────┘                │
│                                                                 │
│   Automatic        Interactive       Blocked                   │
│   No prompt        User confirms     Never runs                │
│   Full trust       Case-by-case      Zero trust                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Permission Modes

Permission modes are global settings that affect how Claude operates:

| Mode | Description | Use Case |
|------|-------------|----------|
| `default` | Standard interactive operation | Daily development |
| `plan` | Planning only, no execution | Safe exploration |
| `acceptEdits` | Auto-accept file modifications | Trusted refactoring |
| `bypassPermissions` | Full trust, no prompts | Automation (dangerous) |

### Tool-Specific Rules

Permissions can be configured per-tool with pattern matching:

```json
{
  "tools": {
    "Bash": {
      "allow": ["git status", "npm test"],
      "deny": ["rm -rf /"]
    }
  }
}
```

### The Permission Resolution Flow

When Claude attempts an action, the permission system evaluates it through
multiple layers:

```
┌──────────────────────────────────────────────────────────────────┐
│                  PERMISSION RESOLUTION FLOW                      │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Action Requested                                                │
│        │                                                         │
│        ▼                                                         │
│  ┌─────────────┐    Yes    ┌─────────────┐                      │
│  │ CLI Override?│─────────►│ Apply CLI   │                      │
│  └─────────────┘           │ Settings    │                      │
│        │ No                └─────────────┘                      │
│        ▼                                                         │
│  ┌─────────────┐    Yes    ┌─────────────┐                      │
│  │ Project Rule?│─────────►│ Apply       │                      │
│  └─────────────┘           │ Project     │                      │
│        │ No                └─────────────┘                      │
│        ▼                                                         │
│  ┌─────────────┐    Yes    ┌─────────────┐                      │
│  │ User Rule?   │─────────►│ Apply User  │                      │
│  └─────────────┘           │ Settings    │                      │
│        │ No                └─────────────┘                      │
│        ▼                                                         │
│  ┌─────────────┐                                                │
│  │ Use Default │                                                │
│  └─────────────┘                                                │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Understanding Permission Levels

### Allow: Automatic Execution

The `allow` permission level grants Claude automatic permission to perform
specific actions without prompting you for confirmation.

#### When to Use Allow

- **Read-only operations**: Viewing files, searching code
- **Safe commands**: `git status`, `npm test`, `ls`
- **Trusted patterns**: Test files, generated code
- **Repetitive tasks**: Commands you approve frequently

#### How Allow Works

```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "allow": [
          "git status",
          "git diff",
          "git log"
        ]
      }
    }
  }
}
```

With this configuration:

```
You: What's the current git status?

Claude: I'll check the git status for you.
[Executing: git status - ALLOWED]

On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

Notice there's no confirmation prompt - the command runs immediately.

#### Allow Patterns

Allow rules support wildcards and patterns:

```json
{
  "allow": [
    "git *",           // Any git command
    "npm run *",       // Any npm script
    "./scripts/*.sh",  // Any script in scripts/
    "docker-compose *" // Any docker-compose command
  ]
}
```

#### Caution with Allow

Be careful what you allow automatically:

```json
// DANGEROUS - Don't do this!
{
  "allow": [
    "*",           // Allows EVERYTHING
    "rm *",        // Allows any deletion
    "sudo *"       // Allows privileged commands
  ]
}
```

### Ask: Interactive Confirmation

The `ask` permission level requires your explicit approval before Claude
performs an action. This is the default for most potentially impactful
operations.

#### When Ask is Used

- **File modifications**: Writing, editing, deleting files
- **Shell commands**: Most bash operations
- **Network requests**: API calls, downloads
- **Resource creation**: New files, directories

#### The Approval Flow

```
You: Create a new component file for the header.

Claude: I'll create a new Header component for you.

┌────────────────────────────────────────────────────────────────┐
│ Claude wants to write to: src/components/Header.tsx            │
│                                                                │
│ Preview:                                                       │
│ ┌────────────────────────────────────────────────────────────┐ │
│ │ import React from 'react';                                 │ │
│ │                                                            │ │
│ │ export const Header: React.FC = () => {                    │ │
│ │   return (                                                 │ │
│ │     <header className="header">                            │ │
│ │       <h1>My Application</h1>                              │ │
│ │     </header>                                              │ │
│ │   );                                                       │ │
│ │ };                                                         │ │
│ └────────────────────────────────────────────────────────────┘ │
│                                                                │
│ [y] Yes  [n] No  [e] Edit  [a] Always allow  [d] Always deny   │
└────────────────────────────────────────────────────────────────┘
```

#### Response Options

When prompted, you have several options:

| Key | Action | Effect |
|-----|--------|--------|
| `y` | Yes | Allow this specific action once |
| `n` | No | Deny this specific action once |
| `e` | Edit | Modify the proposed action before execution |
| `a` | Always allow | Add to permanent allow list |
| `d` | Always deny | Add to permanent deny list |

#### Configuring Ask Behavior

```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "defaultMode": "ask"  // Require confirmation for bash
      },
      "Write": {
        "defaultMode": "ask"  // Require confirmation for writes
      }
    }
  }
}
```

### Deny: Blocked Operations

The `deny` permission level completely blocks certain actions. Claude cannot
perform these operations even if explicitly asked.

#### When to Use Deny

- **Dangerous commands**: `rm -rf /`, `format`, system commands
- **Sensitive files**: Credentials, private keys, environment files
- **Privileged operations**: `sudo`, system configuration changes
- **Irreversible actions**: Database drops, force pushes

#### How Deny Works

```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "deny": [
          "rm -rf /",
          "rm -rf /*",
          "sudo *",
          ":(){:|:&};:",  // Fork bomb
          "chmod 777 *",
          "curl * | bash",
          "wget * | sh"
        ]
      }
    }
  }
}
```

When a denied action is attempted:

```
You: Delete all files in the root directory

Claude: I cannot execute that command. The operation "rm -rf /"
is blocked by your permission settings for safety reasons.

This command is on the deny list because it would delete all
files on your system, which is almost certainly not what you want.

If you need to delete specific files, please specify which files
or directories you'd like to remove, and I can help with that
in a safe way.
```

#### Deny Patterns

Deny rules also support patterns:

```json
{
  "deny": [
    "rm -rf *",        // Block recursive force delete
    "* > /dev/*",      // Block writing to devices
    "dd if=*",         // Block disk operations
    "mkfs.*",          // Block filesystem formatting
    "> /etc/*",        // Block writing to system config
    "chmod 777 *",     // Block world-writable permissions
    "curl * | *sh*",   // Block pipe-to-shell attacks
    "wget * | *sh*"    // Block pipe-to-shell attacks
  ]
}
```

#### Deny Takes Precedence

If an action matches both allow and deny rules, deny always wins:

```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "allow": ["git *"],       // Allow all git commands...
        "deny": ["git push -f *"] // ...except force push
      }
    }
  }
}
```

---

## Permission Modes

### Default Mode: Standard Interactive Operation

Default mode is the standard operating mode for Claude Code. It balances
productivity with security by requiring confirmation for impactful actions
while allowing read operations freely.

#### Characteristics

- Read operations: Typically allowed
- File modifications: Require confirmation
- Shell commands: Depend on specific rules
- Network access: Requires confirmation

#### When to Use

- Daily development work
- Exploring unfamiliar codebases
- Learning new projects
- General-purpose assistance

#### Configuration

Default mode is active unless you specify otherwise:

```bash
# Explicitly set default mode
claude --permission-mode default

# Or simply start without flags
claude
```

### Plan Mode: Safe Exploration

Plan mode restricts Claude to planning and analysis only. Claude can read,
search, and analyze code, but cannot make any modifications or execute
commands that change state.

#### Characteristics

- Read operations: Allowed
- File modifications: Blocked
- Shell commands: Read-only allowed (like `git status`)
- Execution: Disabled

#### When to Use

- Code review sessions
- Architecture analysis
- Learning new codebases
- Security audits
- Planning refactoring

#### Configuration

```bash
# Start in plan mode
claude --permission-mode plan
```

#### Example Session

```
You: Help me understand this codebase and plan improvements.

Claude: I'll analyze the codebase structure for you.

[Reading: package.json - ALLOWED]
[Reading: src/index.ts - ALLOWED]
[Searching: "export function" - ALLOWED]
[Executing: git log --oneline -10 - ALLOWED (read-only)]

Based on my analysis, here's the project structure:
...

I've identified several areas for improvement:
1. The authentication module could use dependency injection
2. Error handling is inconsistent across services
3. Test coverage is low in the API layer

Would you like me to create a detailed improvement plan?
Note: I'm in plan mode, so I can describe changes but not
implement them. Switch to default mode when you're ready
to make modifications.
```

### AcceptEdits Mode: Trusted File Operations

AcceptEdits mode automatically approves all file modification requests
(Write, Edit, NotebookEdit) while still requiring confirmation for
shell commands and other potentially impactful operations.

#### Characteristics

- Read operations: Allowed
- File Write/Edit: Auto-approved
- Shell commands: Still require confirmation
- Other tools: Default behavior

#### When to Use

- Large refactoring tasks
- Migrating codebases
- Generating boilerplate code
- Working in trusted projects
- Automated code generation

#### Configuration

```bash
# Start with auto-accept edits
claude --permission-mode acceptEdits
```

#### Example Session

```
You: Rename the "user" variable to "currentUser" across all files.

Claude: I'll update all occurrences of "user" to "currentUser".

[Editing: src/auth/login.ts - AUTO-ACCEPTED]
[Editing: src/auth/logout.ts - AUTO-ACCEPTED]
[Editing: src/services/userService.ts - AUTO-ACCEPTED]
[Editing: src/components/Profile.tsx - AUTO-ACCEPTED]
[Editing: src/hooks/useUser.ts - AUTO-ACCEPTED]

Done! I've updated 23 occurrences across 5 files.
```

#### Safety Considerations

Even in acceptEdits mode:

- Deny rules still apply
- Shell commands still prompt
- You can still review changes via git

### BypassPermissions Mode: Full Trust (Dangerous!)

BypassPermissions mode disables all permission checks. Claude can perform
any action without prompting. This is extremely powerful and potentially
dangerous.

#### Characteristics

- All operations: Auto-approved
- No confirmation prompts
- No safety checks
- Full system access with your privileges

#### When to Use (Rarely!)

- Fully automated CI/CD pipelines
- Trusted automation scripts
- Sandboxed environments
- When you've thoroughly vetted the task

#### Configuration

```bash
# DANGER: Bypass all permissions
claude --permission-mode bypassPermissions

# Usually combined with specific task
claude --permission-mode bypassPermissions "Run the full test suite and fix any failures"
```

#### Extreme Caution Required

```
┌─────────────────────────────────────────────────────────────────┐
│                        ⚠️  WARNING  ⚠️                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  bypassPermissions mode disables ALL safety checks.            │
│                                                                 │
│  Claude can:                                                    │
│  • Delete any file you have access to                          │
│  • Execute any command with your privileges                    │
│  • Modify system configurations                                │
│  • Make network requests to any destination                    │
│  • Install software                                            │
│                                                                 │
│  Only use this mode when:                                      │
│  • You're in a sandboxed/disposable environment                │
│  • You completely trust the task being performed               │
│  • You've reviewed the automation carefully                    │
│  • You have backups of important data                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Configuring Permissions

### Configuration File Hierarchy

Claude Code looks for permission settings in multiple locations, with
later sources overriding earlier ones:

```
┌─────────────────────────────────────────────────────────────────┐
│              CONFIGURATION HIERARCHY                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Priority        Location                                       │
│  ────────        ────────                                       │
│                                                                 │
│  HIGHEST         Command-line flags                             │
│     │            claude --permission-mode bypassPermissions     │
│     │                                                           │
│     │            Enterprise policy                              │
│     │            /etc/claude/settings.json (Linux/Mac)          │
│     │            %PROGRAMDATA%\claude\settings.json (Windows)   │
│     │                                                           │
│     │            Project settings                               │
│     │            .claude/settings.json                          │
│     │                                                           │
│     ▼            User settings                                  │
│  LOWEST          ~/.claude/settings.json                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Global Permissions: User Settings

User settings apply to all your Claude Code sessions across all projects.

#### Location

| Operating System | Path |
|------------------|------|
| Linux/macOS | `~/.claude/settings.json` |
| Windows | `%USERPROFILE%\.claude\settings.json` |

#### Creating User Settings

```bash
# Create the settings directory
mkdir -p ~/.claude

# Create or edit settings file
nano ~/.claude/settings.json
```

#### Template: Complete User Settings

```json
{
  "permissions": {
    "defaultMode": "default",
    "tools": {
      "Bash": {
        "defaultMode": "ask",
        "allow": [
          "git status",
          "git diff",
          "git log *",
          "git branch *",
          "git show *",
          "npm test",
          "npm run build",
          "npm run lint",
          "npm run format",
          "yarn test",
          "yarn build",
          "pnpm test",
          "pnpm build",
          "ls *",
          "pwd",
          "which *",
          "echo *",
          "cat *",
          "head *",
          "tail *",
          "wc *",
          "date",
          "whoami"
        ],
        "deny": [
          "rm -rf /",
          "rm -rf /*",
          "rm -rf ~",
          "rm -rf ~/*",
          "sudo *",
          "su *",
          "chmod 777 *",
          "chmod -R 777 *",
          "chown -R * /",
          ":(){:|:&};:",
          "dd if=/dev/*",
          "mkfs.*",
          "> /dev/sda*",
          "curl * | bash",
          "curl * | sh",
          "wget * | bash",
          "wget * | sh",
          "eval \"$(curl *\"",
          "python -c \"import os; os.system*\"",
          "nc -l *",
          "ncat *"
        ]
      },
      "Write": {
        "defaultMode": "ask",
        "allow": [
          "*.test.js",
          "*.test.ts",
          "*.test.tsx",
          "*.spec.js",
          "*.spec.ts",
          "*.spec.tsx",
          "__tests__/*",
          "__mocks__/*"
        ],
        "deny": [
          ".env",
          ".env.*",
          "*.pem",
          "*.key",
          "*.crt",
          "**/credentials*",
          "**/secrets*",
          "**/*password*",
          "**/id_rsa*",
          "**/id_ed25519*",
          "**/.ssh/*",
          "**/token*"
        ]
      },
      "Edit": {
        "defaultMode": "ask",
        "deny": [
          ".env",
          ".env.*",
          "*.lock",
          "package-lock.json",
          "yarn.lock",
          "pnpm-lock.yaml"
        ]
      },
      "Read": {
        "defaultMode": "allow",
        "deny": [
          ".env",
          ".env.*",
          "*.pem",
          "*.key",
          "**/credentials*",
          "**/*password*",
          "**/id_rsa*"
        ]
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

### Project Permissions

Project settings apply only to a specific project and can be shared
with team members via version control.

#### Location

```
your-project/
├── .claude/
│   └── settings.json    <- Project permissions
├── src/
├── package.json
└── ...
```

#### Creating Project Settings

```bash
# Navigate to your project root
cd /path/to/your/project

# Create the .claude directory
mkdir -p .claude

# Create project settings
nano .claude/settings.json
```

#### Template: Project Settings

```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "allow": [
          "docker-compose up",
          "docker-compose up -d",
          "docker-compose down",
          "docker-compose logs *",
          "docker-compose ps",
          "docker build *",
          "./scripts/*.sh",
          "make *",
          "cargo build",
          "cargo test",
          "cargo run",
          "go build",
          "go test ./...",
          "go run *.go"
        ],
        "deny": [
          "docker system prune -a",
          "docker volume rm *"
        ]
      },
      "Edit": {
        "allow": [
          "src/**/*.ts",
          "src/**/*.tsx",
          "src/**/*.js",
          "src/**/*.jsx",
          "lib/**/*.ts",
          "tests/**/*.ts"
        ],
        "deny": [
          "*.lock",
          "package-lock.json",
          "dist/**/*",
          "build/**/*",
          "node_modules/**/*"
        ]
      },
      "Write": {
        "allow": [
          "src/**/*.ts",
          "src/**/*.tsx",
          "tests/**/*.ts"
        ],
        "deny": [
          ".env*",
          "config/secrets/*"
        ]
      }
    }
  },
  "projectContext": {
    "description": "E-commerce platform with TypeScript backend and React frontend",
    "conventions": [
      "Use functional components with hooks",
      "Follow the repository pattern for data access",
      "All new code must have unit tests"
    ]
  }
}
```

#### Version Control Considerations

Decide what to include in your repository:

```gitignore
# .gitignore options

# Option 1: Share all project settings
# (nothing to add)

# Option 2: Share structure, ignore local overrides
.claude/settings.local.json

# Option 3: Ignore all Claude settings
.claude/
```

**Recommendation**: Share `.claude/settings.json` with your team for
consistent experiences, but use `.claude/settings.local.json` for
personal overrides.

### Command-Line Overrides

Command-line flags provide the highest priority overrides for individual
sessions.

#### Permission Mode Flag

```bash
# Set permission mode for this session
claude --permission-mode default
claude --permission-mode plan
claude --permission-mode acceptEdits
claude --permission-mode bypassPermissions
```

#### Allowed Tools Flag

```bash
# Restrict Claude to specific tools only
claude --allowedTools "Read,Glob,Grep"

# Multiple tools as comma-separated list
claude --allowedTools "Read,Glob,Grep,Edit,Write"

# Read-only mode (practical alternative to plan mode)
claude --allowedTools "Read,Glob,Grep,WebFetch"
```

#### Combining Flags

```bash
# Auto-accept edits but restrict to specific tools
claude --permission-mode acceptEdits --allowedTools "Read,Write,Edit,Glob,Grep"

# Non-interactive mode with specific task
claude --permission-mode bypassPermissions "Run all tests and generate a coverage report"
```

#### Template: Common CLI Patterns

```bash
# Development: Standard with auto-accept edits
claude --permission-mode acceptEdits

# Code Review: Read-only analysis
claude --permission-mode plan

# CI/CD: Full automation (use with caution)
claude --permission-mode bypassPermissions

# Security Audit: Read and search only
claude --allowedTools "Read,Glob,Grep"

# Documentation: Read source, write docs only
claude --allowedTools "Read,Glob,Grep,Write" \
       --permission-mode acceptEdits

# Quick fix: Specific task with auto-accept
claude --permission-mode acceptEdits "Fix the TypeScript errors in src/"
```

---

## Tool-Specific Permissions

### Bash Command Patterns

The Bash tool is the most security-sensitive tool and supports detailed
pattern matching for allow/deny rules.

#### Pattern Syntax

| Pattern | Matches | Example |
|---------|---------|---------|
| `exact` | Exact command | `git status` |
| `cmd *` | Command with any args | `git *` |
| `*pattern*` | Anywhere in command | `*test*` |
| `./path/*` | Scripts in directory | `./scripts/*` |
| `prefix*` | Commands starting with | `npm run *` |

#### Safe Bash Configuration

```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "defaultMode": "ask",
        "allow": [
          "git status",
          "git diff",
          "git diff --staged",
          "git diff HEAD~*",
          "git log *",
          "git log --oneline *",
          "git branch",
          "git branch -a",
          "git show *",
          "git blame *",
          "git stash list",
          "npm test",
          "npm test -- *",
          "npm run test*",
          "npm run build",
          "npm run lint",
          "npm run lint -- --fix",
          "npm run format",
          "npm run typecheck",
          "npx tsc --noEmit",
          "npx prettier --check *",
          "npx eslint *",
          "node --version",
          "npm --version",
          "ls",
          "ls -la",
          "ls -la *",
          "pwd",
          "which *",
          "head -n * *",
          "tail -n * *",
          "wc -l *",
          "find . -name *",
          "find . -type f -name *"
        ],
        "deny": [
          "rm -rf /",
          "rm -rf /*",
          "rm -rf ~",
          "rm -rf ~/*",
          "rm -rf .",
          "rm -rf ./*",
          "sudo *",
          "su *",
          "su - *",
          "doas *",
          "pkexec *",
          "chmod 777 *",
          "chmod -R 777 *",
          "chown * /*",
          "chgrp * /*",
          "curl * | bash",
          "curl * | sh",
          "wget * -O - | bash",
          "wget * -O - | sh",
          "bash -c \"$(curl *\"",
          "sh -c \"$(curl *\"",
          "eval \"$(curl *\"",
          "python* -c *os.system*",
          "python* -c *subprocess*",
          "perl -e *system*",
          "ruby -e *system*",
          "nc -l*",
          "ncat -l*",
          "netcat *",
          "> /dev/sda*",
          "dd if=*",
          "mkfs.*",
          "fdisk *",
          "parted *",
          "shred *",
          ":(){ :|:& };:",
          "fork()",
          "history -c",
          "export HISTSIZE=0"
        ]
      }
    }
  }
}
```

#### Dangerous Patterns to Always Deny

```json
{
  "deny": [
    "rm -rf *",
    "sudo *",
    "curl * | *sh*",
    "wget * | *sh*",
    "> /etc/*",
    "> /dev/*",
    "chmod 777 *",
    ":()*{*",
    "dd if=/dev/*"
  ]
}
```

### File Operation Rules

Configure permissions for Read, Write, and Edit operations based on
file paths and patterns.

#### Read Tool Configuration

```json
{
  "permissions": {
    "tools": {
      "Read": {
        "defaultMode": "allow",
        "deny": [
          ".env",
          ".env.*",
          ".env.local",
          ".env.production",
          "*.pem",
          "*.key",
          "*.p12",
          "*.pfx",
          "**/secrets/*",
          "**/credentials/*",
          "**/.ssh/*",
          "**/id_rsa*",
          "**/id_ed25519*",
          "**/id_ecdsa*",
          "**/.gnupg/*",
          "**/.aws/credentials",
          "**/.kube/config",
          "**/token*",
          "**/*secret*",
          "**/*password*.txt"
        ]
      }
    }
  }
}
```

#### Write Tool Configuration

```json
{
  "permissions": {
    "tools": {
      "Write": {
        "defaultMode": "ask",
        "allow": [
          "src/**/*.ts",
          "src/**/*.tsx",
          "src/**/*.js",
          "src/**/*.jsx",
          "src/**/*.css",
          "src/**/*.scss",
          "lib/**/*",
          "tests/**/*",
          "__tests__/**/*",
          "*.test.ts",
          "*.test.tsx",
          "*.spec.ts",
          "*.spec.tsx",
          "docs/**/*.md",
          "*.md"
        ],
        "deny": [
          ".env*",
          "*.pem",
          "*.key",
          "*.crt",
          "**/secrets/*",
          "**/credentials/*",
          ".git/**/*",
          "node_modules/**/*",
          "dist/**/*",
          "build/**/*",
          "*.lock",
          "package-lock.json",
          "yarn.lock",
          "pnpm-lock.yaml"
        ]
      }
    }
  }
}
```

#### Edit Tool Configuration

```json
{
  "permissions": {
    "tools": {
      "Edit": {
        "defaultMode": "ask",
        "allow": [
          "src/**/*",
          "lib/**/*",
          "tests/**/*",
          "scripts/**/*"
        ],
        "deny": [
          ".env*",
          "*.lock",
          "package-lock.json",
          "yarn.lock",
          "pnpm-lock.yaml",
          ".git/**/*",
          "node_modules/**/*",
          "*.min.js",
          "*.min.css",
          "dist/**/*",
          "build/**/*"
        ]
      }
    }
  }
}
```

### Network Access Rules

Control how Claude interacts with external resources.

#### WebFetch Configuration

```json
{
  "permissions": {
    "tools": {
      "WebFetch": {
        "defaultMode": "ask",
        "allow": [
          "https://docs.*",
          "https://*.github.com/*",
          "https://stackoverflow.com/*",
          "https://developer.mozilla.org/*",
          "https://www.typescriptlang.org/*",
          "https://reactjs.org/*",
          "https://nodejs.org/*"
        ],
        "deny": [
          "http://*",
          "*.local/*",
          "*.internal/*",
          "localhost*",
          "127.0.0.1*",
          "192.168.*",
          "10.*",
          "172.16.*"
        ]
      }
    }
  }
}
```

### MCP Tool Permissions

If you're using Model Context Protocol (MCP) tools, configure their
permissions similarly.

#### MCP Configuration

```json
{
  "permissions": {
    "tools": {
      "mcp__database__query": {
        "defaultMode": "ask",
        "allow": [
          "SELECT *"
        ],
        "deny": [
          "DROP *",
          "DELETE *",
          "TRUNCATE *",
          "ALTER *",
          "CREATE *"
        ]
      },
      "mcp__slack__send_message": {
        "defaultMode": "ask"
      },
      "mcp__github__create_issue": {
        "defaultMode": "ask"
      }
    }
  }
}
```

---

## Permission Precedence

### The Precedence Hierarchy

When multiple configuration sources define permissions for the same
action, Claude Code resolves conflicts using a strict precedence order:

| Level | Priority | Location | Scope |
|-------|----------|----------|-------|
| CLI flags | Highest | Command line | Session only |
| Enterprise | Very High | System config | All users |
| Project | High | .claude/settings.json | This project |
| User | Medium | ~/.claude/settings.json | All projects |
| Built-in | Lowest | Internal defaults | Fallback |

### Resolution Rules

#### Rule 1: Higher Priority Wins

```
CLI: --permission-mode acceptEdits
User: defaultMode: "ask"
Result: acceptEdits (CLI wins)
```

#### Rule 2: Deny Always Trumps Allow

```
Project: allow: ["git *"]
User: deny: ["git push -f *"]
Result: "git push -f main" is DENIED
```

#### Rule 3: Specific Rules Override General

```
User: Bash.defaultMode: "ask"
Project: Bash.allow: ["npm test"]
Result: "npm test" is ALLOWED (specific rule)
```

### Example: Complex Resolution

Consider this configuration:

**User settings (~/.claude/settings.json):**
```json
{
  "permissions": {
    "defaultMode": "default",
    "tools": {
      "Bash": {
        "defaultMode": "ask",
        "allow": ["git *", "npm *"],
        "deny": ["rm -rf *"]
      }
    }
  }
}
```

**Project settings (.claude/settings.json):**
```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "allow": ["./scripts/deploy.sh"],
        "deny": ["git push -f *"]
      }
    }
  }
}
```

**Resolution for various commands:**

| Command | User | Project | Result | Reason |
|---------|------|---------|--------|--------|
| `git status` | allow | - | ALLOW | Matches user allow |
| `npm test` | allow | - | ALLOW | Matches user allow |
| `git push -f main` | allow | deny | DENY | Deny trumps allow |
| `./scripts/deploy.sh` | ask | allow | ALLOW | Project overrides |
| `rm -rf /tmp/test` | deny | - | DENY | Matches user deny |
| `docker build` | ask | - | ASK | Default mode |

### Debugging Precedence

When permissions don't behave as expected, trace through the hierarchy:

```
1. Check CLI flags first
   → Was --permission-mode or --allowedTools used?

2. Check enterprise policy (if applicable)
   → /etc/claude/settings.json or equivalent

3. Check project settings
   → .claude/settings.json in project root

4. Check user settings
   → ~/.claude/settings.json

5. Fall back to defaults
   → Built-in Claude Code defaults
```

---

## Real-World Examples

### Example 1: Developer Workstation Setup (Balanced Security)

A configuration that balances productivity with security for daily
development work.

**~/.claude/settings.json:**
```json
{
  "permissions": {
    "defaultMode": "default",
    "tools": {
      "Bash": {
        "defaultMode": "ask",
        "allow": [
          "git status",
          "git diff",
          "git diff --staged",
          "git log *",
          "git branch *",
          "git show *",
          "git stash *",
          "npm test",
          "npm test -- *",
          "npm run *",
          "yarn test",
          "yarn build",
          "pnpm test",
          "pnpm build",
          "node --version",
          "npm --version",
          "npx tsc --noEmit",
          "npx jest *",
          "npx prettier --check *",
          "npx eslint *",
          "ls *",
          "pwd",
          "which *",
          "cat *",
          "head *",
          "tail *"
        ],
        "deny": [
          "rm -rf *",
          "sudo *",
          "su *",
          "chmod 777 *",
          "curl * | bash",
          "wget * | sh",
          "> /etc/*",
          "git push -f *",
          "git reset --hard HEAD~*"
        ]
      },
      "Read": {
        "defaultMode": "allow",
        "deny": [
          ".env*",
          "*.pem",
          "*.key"
        ]
      },
      "Write": {
        "defaultMode": "ask",
        "allow": [
          "*.test.ts",
          "*.test.tsx",
          "*.spec.ts"
        ],
        "deny": [
          ".env*",
          "*.lock"
        ]
      },
      "Edit": {
        "defaultMode": "ask",
        "deny": [
          ".env*",
          "*.lock"
        ]
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

**Use Case**: Everyday development where you want Claude to help with
testing, linting, and code exploration without constant prompts, but
still require confirmation for file changes.

### Example 2: CI/CD Pipeline Permissions (Automated, Restricted)

A configuration for automated environments where Claude runs
unattended but with strict limitations.

**CI environment configuration:**
```json
{
  "permissions": {
    "defaultMode": "default",
    "tools": {
      "Bash": {
        "defaultMode": "deny",
        "allow": [
          "npm ci",
          "npm test",
          "npm run test:*",
          "npm run build",
          "npm run lint",
          "npm run typecheck",
          "git status",
          "git diff HEAD",
          "npx jest --coverage",
          "npx tsc --noEmit"
        ]
      },
      "Read": {
        "defaultMode": "allow",
        "deny": [
          ".env*",
          "**/secrets/*"
        ]
      },
      "Write": {
        "defaultMode": "deny",
        "allow": [
          "coverage/**/*",
          "test-results/**/*",
          ".claude/reports/*"
        ]
      },
      "Edit": {
        "defaultMode": "deny"
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

**CLI invocation:**
```bash
claude --permission-mode bypassPermissions \
       --allowedTools "Bash,Read,Glob,Grep" \
       "Run the test suite and generate a coverage report"
```

**Use Case**: Automated testing and analysis in CI pipelines where
Claude should only run tests and generate reports, not modify code.

### Example 3: Code Review Mode (Read-Only)

A configuration for reviewing code without making any changes.

**CLI invocation:**
```bash
claude --permission-mode plan
```

**Alternative with explicit tool restrictions:**
```bash
claude --allowedTools "Read,Glob,Grep"
```

**Project configuration for review-only:**
```json
{
  "permissions": {
    "defaultMode": "plan",
    "tools": {
      "Bash": {
        "defaultMode": "deny",
        "allow": [
          "git log *",
          "git show *",
          "git blame *",
          "git diff *"
        ]
      },
      "Read": {
        "defaultMode": "allow"
      },
      "Write": {
        "defaultMode": "deny"
      },
      "Edit": {
        "defaultMode": "deny"
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

**Use Case**: Security audits, code reviews, or exploring unfamiliar
codebases where you want zero risk of accidental modifications.

### Example 4: Trusted Project with Auto-Accept

A configuration for projects where you fully trust Claude's edits.

**.claude/settings.json:**
```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "defaultMode": "ask",
        "allow": [
          "npm *",
          "yarn *",
          "pnpm *",
          "git add *",
          "git commit -m *",
          "git status",
          "git diff *",
          "docker-compose *",
          "./scripts/*"
        ],
        "deny": [
          "rm -rf *",
          "git push *",
          "git reset --hard *"
        ]
      },
      "Write": {
        "defaultMode": "allow",
        "deny": [
          ".env*",
          "*.lock"
        ]
      },
      "Edit": {
        "defaultMode": "allow",
        "deny": [
          ".env*",
          "*.lock"
        ]
      },
      "Read": {
        "defaultMode": "allow"
      }
    }
  }
}
```

**CLI invocation:**
```bash
claude --permission-mode acceptEdits
```

**Use Case**: Personal projects, prototypes, or heavily-tested
refactoring tasks where you want Claude to work quickly.

### Example 5: Security Audit Configuration

A locked-down configuration for security-sensitive analysis.

```json
{
  "permissions": {
    "defaultMode": "ask",
    "tools": {
      "Bash": {
        "defaultMode": "deny",
        "allow": [
          "git log --oneline *",
          "git show *",
          "git blame *",
          "find . -name *",
          "wc -l *"
        ],
        "deny": [
          "*"
        ]
      },
      "Read": {
        "defaultMode": "allow",
        "deny": [
          ".env*",
          "*.pem",
          "*.key",
          "*.crt",
          "**/secrets/*",
          "**/credentials/*",
          "**/.aws/*",
          "**/.ssh/*"
        ]
      },
      "Write": {
        "defaultMode": "deny"
      },
      "Edit": {
        "defaultMode": "deny"
      },
      "Glob": {
        "defaultMode": "allow"
      },
      "Grep": {
        "defaultMode": "allow"
      },
      "WebFetch": {
        "defaultMode": "deny"
      }
    }
  }
}
```

**Use Case**: Auditing codebases for security vulnerabilities where
you want Claude to analyze but not modify or execute anything.

### Example 6: Team Shared Permissions

A project configuration designed for team consistency.

**.claude/settings.json (committed to repo):**
```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "defaultMode": "ask",
        "allow": [
          "npm test",
          "npm run test:*",
          "npm run build",
          "npm run lint",
          "npm run lint:fix",
          "npm run format",
          "npm run typecheck",
          "docker-compose up -d",
          "docker-compose down",
          "docker-compose logs *",
          "./scripts/setup.sh",
          "./scripts/dev.sh",
          "make build",
          "make test"
        ],
        "deny": [
          "npm publish",
          "docker push *",
          "git push *",
          "rm -rf *",
          "./scripts/deploy*"
        ]
      },
      "Edit": {
        "allow": [
          "src/**/*",
          "tests/**/*",
          "docs/**/*"
        ],
        "deny": [
          ".env*",
          "*.lock",
          "config/production/*"
        ]
      },
      "Write": {
        "allow": [
          "src/**/*",
          "tests/**/*"
        ],
        "deny": [
          ".env*",
          "*.lock",
          "config/production/*"
        ]
      }
    }
  },
  "projectContext": {
    "description": "Shared team project - use consistent Claude settings",
    "conventions": [
      "Run tests before committing",
      "Use conventional commit messages",
      "Don't modify lock files directly"
    ]
  }
}
```

**.claude/settings.local.json (gitignored, personal overrides):**
```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "allow": [
          "git push origin feature/*"
        ]
      }
    }
  }
}
```

**Use Case**: Teams that want consistent Claude behavior across all
members while allowing personal customization.

### Example 7: Onboarding New Developers

A restrictive configuration for developers new to a codebase.

```json
{
  "permissions": {
    "defaultMode": "ask",
    "tools": {
      "Bash": {
        "defaultMode": "ask",
        "allow": [
          "git status",
          "git diff",
          "git log *",
          "npm test",
          "npm run build"
        ],
        "deny": [
          "git push *",
          "git commit *",
          "npm publish",
          "rm *",
          "mv *",
          "cp *"
        ]
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
      }
    }
  },
  "projectContext": {
    "description": "Onboarding mode - all changes require review",
    "onboarding": {
      "readFirst": [
        "README.md",
        "CONTRIBUTING.md",
        "docs/ARCHITECTURE.md"
      ],
      "mentorContact": "@senior-dev on Slack"
    }
  }
}
```

**Use Case**: New team members learning the codebase who should
explore freely but get confirmation before making changes.

---

## Troubleshooting

### Problem: "Permission Denied" for Expected Commands

**Symptom**: Claude reports that a command is blocked, but you expected
it to be allowed.

**Diagnostic Steps**:

1. **Check the exact command format**
   ```
   Your allow rule: "git status"
   Attempted command: "git status --short"
   → Mismatch! Use "git status *" to allow flags
   ```

2. **Check for deny rules at higher priority**
   ```
   User settings: deny: ["git *"]
   Project settings: allow: ["git status"]
   → Deny at user level blocks project allow
   ```

3. **Verify pattern syntax**
   ```
   Wrong: "npm run-*"
   Right: "npm run *" or "npm run-script *"
   ```

**Solution Examples**:

```json
// Before: Too specific
{
  "allow": ["git status"]
}

// After: Allows variations
{
  "allow": ["git status", "git status *"]
}
```

```json
// Before: Accidental wildcard conflict
{
  "deny": ["rm *"],     // Denies "rm -rf /" but also "rm temp.txt"
  "allow": ["rm *.log"]  // Won't work - deny trumps
}

// After: More specific deny
{
  "deny": ["rm -rf *", "rm -r *"],  // Specific dangerous patterns
  "allow": ["rm *.log", "rm *.tmp"]  // Safe deletions work
}
```

### Problem: Permission Prompt Fatigue

**Symptom**: You're constantly confirming the same actions and it's
slowing down your work.

**Solutions**:

1. **Expand your allow list**
   ```json
   {
     "allow": [
       "git *",        // All git commands
       "npm *",        // All npm commands
       "ls *",         // All ls variations
       "cat *",        // All cat commands
       "head *",       // All head commands
       "tail *"        // All tail commands
     ]
   }
   ```

2. **Use "Always Allow" at prompts**
   When prompted, press `a` to permanently allow that pattern.

3. **Consider acceptEdits mode for trusted sessions**
   ```bash
   claude --permission-mode acceptEdits
   ```

4. **Create project-specific allows**
   ```json
   // .claude/settings.json
   {
     "permissions": {
       "tools": {
         "Bash": {
           "allow": ["./scripts/*", "make *"]
         }
       }
     }
   }
   ```

### Problem: Unexpected Auto-Allow

**Symptom**: Commands are running without confirmation when you expected
to be asked.

**Diagnostic Steps**:

1. **Check permission mode**
   ```bash
   # Were you in bypassPermissions or acceptEdits mode?
   # Check your shell history for flags used
   ```

2. **Check for overly broad allow patterns**
   ```json
   // This allows EVERYTHING starting with "rm"
   { "allow": ["rm *"] }  // Dangerous!

   // This is safer
   { "allow": ["rm *.log", "rm *.tmp"] }
   ```

3. **Check pattern precedence**
   ```json
   // User level: allow all npm
   { "allow": ["npm *"] }

   // Project level can't override with deny
   // (unless you restructure)
   ```

**Solutions**:

```json
// Add explicit deny rules for dangerous variations
{
  "allow": ["npm *"],
  "deny": ["npm unpublish *", "npm deprecate *"]
}
```

### Problem: Permission Conflicts

**Symptom**: Permissions behave differently than your configuration
suggests.

**Diagnostic Steps**:

1. **Map the full hierarchy**
   ```
   CLI flags: (check command used)
   Enterprise: /etc/claude/settings.json (if exists)
   Project: ./.claude/settings.json
   User: ~/.claude/settings.json
   ```

2. **Check for JSON syntax errors**
   ```bash
   # Validate your JSON files
   cat ~/.claude/settings.json | python -m json.tool
   cat .claude/settings.json | python -m json.tool
   ```

3. **Look for duplicate keys**
   ```json
   // Invalid: duplicate "allow" keys
   {
     "tools": {
       "Bash": {
         "allow": ["git *"],
         "allow": ["npm *"]  // This replaces the first!
       }
     }
   }

   // Correct: single array
   {
     "tools": {
       "Bash": {
         "allow": ["git *", "npm *"]
       }
     }
   }
   ```

### Problem: Debugging Permission Resolution

**Technique**: Create a test configuration to understand precedence.

```json
// ~/.claude/settings.json - Add debug marker
{
  "permissions": {
    "debug": "user-level-config",
    "tools": {
      "Bash": {
        "allow": ["echo USER_CONFIG_ACTIVE"]
      }
    }
  }
}
```

```json
// .claude/settings.json - Add debug marker
{
  "permissions": {
    "debug": "project-level-config",
    "tools": {
      "Bash": {
        "allow": ["echo PROJECT_CONFIG_ACTIVE"]
      }
    }
  }
}
```

Then test:
```
You: Run "echo USER_CONFIG_ACTIVE"
You: Run "echo PROJECT_CONFIG_ACTIVE"
```

Observe which commands run without prompting to confirm which
configuration is taking effect.

### Common Error Messages

| Error | Likely Cause | Solution |
|-------|--------------|----------|
| "Command blocked by permissions" | Matches deny rule | Check deny patterns |
| "Tool not available in current mode" | Plan mode or restricted tools | Change mode or add tool |
| "Permission configuration invalid" | JSON syntax error | Validate JSON |
| "Cannot override enterprise policy" | Admin restrictions | Contact admin |

---

## Quick Reference Cheat Sheet

### Permission Levels

```
ALLOW  → Automatic, no prompt
ASK    → Requires confirmation (default)
DENY   → Always blocked
```

### Permission Modes

```bash
claude --permission-mode default         # Standard (default)
claude --permission-mode plan            # Read-only, no execution
claude --permission-mode acceptEdits     # Auto-approve file changes
claude --permission-mode bypassPermissions # Full trust (DANGEROUS)
```

### Tool Restriction

```bash
claude --allowedTools "Read,Glob,Grep"   # Read-only tools
claude --allowedTools "Read,Write,Edit"  # File tools only
```

### Configuration Locations

```
User:      ~/.claude/settings.json
Project:   ./.claude/settings.json
Enterprise: /etc/claude/settings.json (Linux/Mac)
           %PROGRAMDATA%\claude\settings.json (Windows)
```

### Priority Order

```
CLI flags      → Highest
Enterprise     → Very High
Project        → High
User           → Medium
Built-in       → Lowest
```

### Pattern Matching

```
"exact command"     → Matches exactly
"prefix *"          → Matches prefix + anything
"* suffix"          → Matches anything + suffix
"*pattern*"         → Matches anywhere
"path/**/*"         → Recursive glob
"*.extension"       → File extension
```

### Essential Deny Patterns

```json
{
  "deny": [
    "rm -rf /",
    "rm -rf /*",
    "sudo *",
    "chmod 777 *",
    "curl * | bash",
    "wget * | sh",
    "> /dev/*",
    "dd if=/dev/*"
  ]
}
```

### Safe Allow Patterns

```json
{
  "allow": [
    "git status",
    "git diff *",
    "git log *",
    "npm test",
    "npm run build",
    "ls *",
    "pwd",
    "which *"
  ]
}
```

### File Protection

```json
{
  "deny": [
    ".env*",
    "*.pem",
    "*.key",
    "*.lock",
    "**/secrets/*",
    "**/credentials/*"
  ]
}
```

### Minimal Configuration Template

```json
{
  "permissions": {
    "defaultMode": "default",
    "tools": {
      "Bash": { "defaultMode": "ask" },
      "Read": { "defaultMode": "allow" },
      "Write": { "defaultMode": "ask" },
      "Edit": { "defaultMode": "ask" },
      "Glob": { "defaultMode": "allow" },
      "Grep": { "defaultMode": "allow" }
    }
  }
}
```

---

## Best Practices

### Principle of Least Privilege

Grant only the permissions necessary for the task at hand.

**Do:**
```json
{
  "allow": [
    "npm test",
    "npm run build",
    "npm run lint"
  ]
}
```

**Don't:**
```json
{
  "allow": ["*"]  // Too broad!
}
```

**Why**: Overly permissive configurations increase risk without
proportional benefit. Start restrictive and expand as needed.

### Layer Your Defenses

Use multiple permission layers to catch mistakes.

```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "defaultMode": "ask",        // Layer 1: Require confirmation
        "allow": ["git status"],     // Layer 2: Selective auto-allow
        "deny": ["rm -rf *"]         // Layer 3: Hard blocks
      },
      "Write": {
        "defaultMode": "ask",        // Layer 1
        "deny": [".env*", "*.pem"]   // Layer 2: Protect secrets
      }
    }
  }
}
```

### Separate Development and Production

Never use `bypassPermissions` mode with production systems.

```bash
# Development (okay)
claude --permission-mode acceptEdits

# Staging (careful)
claude --permission-mode default

# Production (never bypass!)
claude --permission-mode plan --allowedTools "Read,Glob,Grep"
```

### Regular Permission Audits

Review your permission configurations periodically:

1. **Monthly review**: Check allow lists for stale entries
2. **After incidents**: Tighten rules if something went wrong
3. **Team changes**: Update shared configurations
4. **Project milestones**: Adjust for changing needs

**Audit Checklist**:
```
[ ] Review all allow patterns - still needed?
[ ] Check deny patterns - comprehensive enough?
[ ] Verify no sensitive files are readable
[ ] Confirm production systems are protected
[ ] Validate team members have appropriate access
```

### Team Permission Standards

Establish and document team conventions.

**PERMISSIONS.md** (for your team):
```markdown
# Team Permission Standards

## Required Deny Rules
All developers must include these deny rules:
- `rm -rf *`
- `sudo *`
- `.env*` files
- Production deployment commands

## Project Settings
Each project should have `.claude/settings.json` with:
- Project-specific build commands allowed
- Team-approved deployment scripts
- Protected configuration files

## Review Process
- Permission changes require code review
- Test permission changes in development first
- Document any exceptions
```

### Balance Security and Productivity

Find the right balance for your situation:

```
More Security ←――――――――――――――――――→ More Productivity

plan mode     ask everything    selective allow    acceptEdits    bypassPermissions
     ↓              ↓                  ↓                ↓                ↓
  Slowest       Careful          Balanced          Faster         Fastest
  Safest        Safe             Reasonable        Some risk      Risky
```

**Recommendations by Context**:

| Context | Recommended Approach |
|---------|---------------------|
| Learning new codebase | `plan` mode |
| Unfamiliar project | `default` with minimal allows |
| Daily development | `default` with common allows |
| Trusted refactoring | `acceptEdits` |
| Automation/CI | Careful `bypassPermissions` with tool limits |

### Document Your Configurations

Add comments explaining why rules exist:

```json
{
  "permissions": {
    "_comment": "Last updated: 2024-01-15 by @devname",
    "tools": {
      "Bash": {
        "_comment": "Allow common dev commands, block dangerous ops",
        "allow": [
          "npm test",
          "npm run build"
        ],
        "deny": [
          "rm -rf *",
          "git push -f *"
        ]
      },
      "Write": {
        "_comment": "Protect credentials and lock files",
        "deny": [
          ".env*",
          "*.lock"
        ]
      }
    }
  }
}
```

### Recovery Procedures

Know how to recover if something goes wrong:

1. **Immediate stop**: Press Ctrl+C to interrupt Claude
2. **Check git status**: See what changed
3. **Review before committing**: `git diff` everything
4. **Rollback if needed**: `git checkout .` or `git stash`
5. **Tighten permissions**: Add deny rules for problem patterns

**Emergency Lockdown**:
```bash
# Restart Claude in plan mode
claude --permission-mode plan

# Or with minimal tools
claude --allowedTools "Read,Glob,Grep"
```

---

## New in Claude Code v2.1.x

This section covers recent improvements to the permission system in Claude Code v2.1.x.

### Wildcard Permissions (v2.0.71 / v2.1.0)

Permissions now support wildcard patterns for more flexible command matching:

#### Basic Wildcards

```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "allow": [
          "npm *",        // Any npm command
          "git status *", // git status with any flags
          "yarn *"        // Any yarn command
        ]
      }
    }
  }
}
```

#### Pattern Syntax

| Pattern | Matches | Example |
|---------|---------|---------|
| `*` | Any sequence of characters | `npm *` → `npm install`, `npm run build` |
| `git * main` | Git commands targeting main | `git push origin main`, `git checkout main` |
| `docker * -it` | Docker with interactive flag | `docker run -it ubuntu` |

#### MCP Tool Wildcards (v2.0.71)

Allow entire MCP servers or specific tool patterns:

```json
{
  "permissions": {
    "tools": {
      "mcp__database__*": {
        "defaultMode": "allow"
      },
      "mcp__github__*": {
        "defaultMode": "ask"
      },
      "mcp__*__read*": {
        "defaultMode": "allow"
      }
    }
  }
}
```

**MCP wildcard patterns:**
- `mcp__server__*` - All tools from a specific server
- `mcp__*__toolname` - Specific tool from any server
- `mcp__*__*` - All MCP tools (use with caution)

#### Task Agent Wildcards (v2.1.0)

Control which sub-agents can be spawned:

```json
{
  "permissions": {
    "tools": {
      "Task(Explore)": { "defaultMode": "allow" },
      "Task(Plan)": { "defaultMode": "allow" },
      "Task(builder)": { "defaultMode": "ask" },
      "Task(*)": { "defaultMode": "deny" }
    }
  }
}
```

### Compound Command Restrictions (v2.1.7)

Enhanced security for chained commands prevents permission bypass:

```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "allow": ["npm test"],
        "deny": [
          "*&&*",      // Deny command chaining
          "*||*",      // Deny OR chaining
          "*;*",       // Deny semicolon chaining
          "*|*"        // Deny piping (be careful with this)
        ]
      }
    }
  }
}
```

**Why this matters:**
Without compound restrictions, a malicious pattern could exploit:
```bash
# Allowed: npm test
# Blocked: npm test && rm -rf /  (if compound denied)
```

**Recommended approach:**
```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "allow": [
          "npm test",
          "npm run *",
          "git status"
        ],
        "deny": [
          "*&&*",
          "*;*"
        ]
      }
    }
  }
}
```

### `/permissions` Search Command (v2.0.67)

Search your permission configuration interactively:

```bash
/permissions npm

# Output:
# Permission rules matching "npm":
#
# User settings (~/.claude/settings.json):
#   Bash.allow: "npm test", "npm run *", "npm install"
#   Bash.deny: "npm publish"
#
# Project settings (.claude/settings.json):
#   Bash.allow: "npm run build"
```

**Search capabilities:**
- Search by command name: `/permissions git`
- Search by tool name: `/permissions Bash`
- Search by MCP server: `/permissions mcp__github`
- View all rules: `/permissions *`

### Unreachable Rule Detection with /doctor (v2.1.3)

The `/doctor` command now detects unreachable permission rules:

```bash
/doctor

# Output includes:
# ⚠️ Unreachable permission rules detected:
#
# In ~/.claude/settings.json:
#   - Bash.allow: "git push" is shadowed by Bash.deny: "git *"
#   - Read.allow: ".env.example" is blocked by Read.deny: ".env*"
#
# Recommendation: Remove unreachable rules or adjust deny patterns
```

**Common unreachable rule patterns:**

| Problem | Allow Rule | Deny Rule | Fix |
|---------|------------|-----------|-----|
| Broad deny shadows specific allow | `git push origin main` | `git *` | Use `git push *` in deny, specific in allow |
| Wildcards conflict | `.env.example` | `.env*` | Add explicit exception or rename file |
| Order confusion | `npm test` | `npm *` | Deny rules always take precedence |

**How to fix unreachable rules:**

1. **Remove the unreachable allow rule** if the deny is intentional
2. **Narrow the deny pattern** to allow specific exceptions
3. **Use more specific patterns** in both allow and deny

```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "allow": [
          "git status",
          "git diff",
          "git log *"
        ],
        "deny": [
          "git push --force*",  // Specific deny
          "git reset --hard*"   // Instead of "git *"
        ]
      }
    }
  }
}
```

### Permission Mode Flags (v2.1.0)

Enhanced CLI flags for permission control:

```bash
# Start with specific permission mode
claude --permission-mode plan        # Read-only
claude --permission-mode default     # Normal prompts
claude --permission-mode acceptEdits # Auto-accept file changes

# Bypass all permissions (dangerous!)
claude --dangerously-skip-permissions

# Custom permission prompt tool (for automation)
claude --permission-prompt-tool custom-handler
```

### Quick Reference: v2.1.x Permission Features

| Feature | Version | Usage |
|---------|---------|-------|
| Bash wildcards | v2.1.0 | `Bash(npm *)` |
| MCP wildcards | v2.0.71 | `mcp__server__*` |
| Task agent permissions | v2.1.0 | `Task(AgentName)` |
| Compound command deny | v2.1.7 | `"*&&*"` patterns |
| `/permissions` search | v2.0.67 | `/permissions <query>` |
| Unreachable detection | v2.1.3 | `/doctor` shows warnings |
| Permission mode flag | v2.1.0 | `--permission-mode` |

---

## Summary

### Key Takeaways

1. **Three Permission Levels**: `allow`, `ask`, and `deny` give you
   granular control over what Claude can do.

2. **Four Permission Modes**: From `plan` (read-only) to
   `bypassPermissions` (full trust), choose the mode that matches
   your current needs.

3. **Configuration Hierarchy**: CLI flags override project settings,
   which override user settings. Deny always trumps allow.

4. **Pattern Matching**: Use wildcards to create flexible rules that
   cover command variations.

5. **Safety First**: Start restrictive and expand permissions as you
   build trust. Always protect sensitive files.

### The Permission Mindset

Think of permissions as a conversation between you and Claude:

- **Allow**: "I trust you with this. Go ahead."
- **Ask**: "Check with me first. Let's decide together."
- **Deny**: "Never do this, even if I accidentally ask."

This framework lets Claude be genuinely helpful while keeping you in
control. The goal isn't to prevent Claude from doing anything - it's
to ensure Claude does exactly what you want, safely.

### Next Steps

1. **Create your user settings**: Start with the balanced developer
   configuration from Example 1.

2. **Customize for your workflow**: Add allows for commands you use
   frequently, denies for operations you never want.

3. **Add project settings**: Share team configurations via version
   control.

4. **Iterate**: Adjust based on experience. Too many prompts? Add
   allows. Unexpected behavior? Add denies.

### Additional Resources

- Tutorial 01: Getting Started with Claude Code
- Tutorial 05: The Tool System
- Tutorial 15: MCP Integration
- Claude Code documentation: Security best practices

---

## Appendix: Complete Configuration Templates

### Template A: Conservative Developer

```json
{
  "permissions": {
    "defaultMode": "default",
    "tools": {
      "Bash": {
        "defaultMode": "ask",
        "allow": [
          "git status",
          "git diff",
          "git log --oneline -20",
          "npm test",
          "npm run build",
          "ls",
          "pwd"
        ],
        "deny": [
          "rm -rf *",
          "sudo *",
          "chmod 777 *",
          "curl * | bash",
          "git push -f *"
        ]
      },
      "Read": {
        "defaultMode": "allow",
        "deny": [".env*", "*.pem", "*.key"]
      },
      "Write": {
        "defaultMode": "ask",
        "deny": [".env*", "*.lock"]
      },
      "Edit": {
        "defaultMode": "ask",
        "deny": [".env*", "*.lock"]
      },
      "Glob": {"defaultMode": "allow"},
      "Grep": {"defaultMode": "allow"}
    }
  }
}
```

### Template B: Productive Developer

```json
{
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
          "npx *",
          "ls *",
          "cat *",
          "head *",
          "tail *",
          "grep *",
          "find *",
          "wc *",
          "pwd",
          "which *",
          "docker ps",
          "docker logs *",
          "docker-compose *"
        ],
        "deny": [
          "rm -rf /",
          "rm -rf /*",
          "sudo *",
          "chmod 777 *",
          "git push -f *",
          "npm unpublish *",
          "docker system prune -a"
        ]
      },
      "Read": {"defaultMode": "allow", "deny": [".env*"]},
      "Write": {"defaultMode": "ask", "deny": [".env*"]},
      "Edit": {"defaultMode": "ask", "deny": [".env*"]},
      "Glob": {"defaultMode": "allow"},
      "Grep": {"defaultMode": "allow"}
    }
  }
}
```

### Template C: Strict Security

```json
{
  "permissions": {
    "defaultMode": "ask",
    "tools": {
      "Bash": {
        "defaultMode": "deny",
        "allow": [
          "git status",
          "git log --oneline *",
          "ls",
          "pwd"
        ]
      },
      "Read": {
        "defaultMode": "ask",
        "deny": [
          ".env*",
          "*.pem",
          "*.key",
          "*.crt",
          "**/secrets/*",
          "**/credentials/*",
          "**/.aws/*",
          "**/.ssh/*"
        ]
      },
      "Write": {"defaultMode": "deny"},
      "Edit": {"defaultMode": "deny"},
      "Glob": {"defaultMode": "allow"},
      "Grep": {"defaultMode": "allow"},
      "WebFetch": {"defaultMode": "deny"}
    }
  }
}
```

---

*This tutorial is part of the Claude Code documentation series.*
*Version 2.1.x | Last Updated: January 2026*
