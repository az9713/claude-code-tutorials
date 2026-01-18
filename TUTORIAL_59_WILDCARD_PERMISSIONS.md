# Tutorial 59: Wildcard Permissions Mastery

> **Security-first patterns for flexible permission rules in Claude Code**

## Table of Contents

1. [Overview](#overview)
2. [Wildcard Fundamentals](#wildcard-fundamentals)
3. [Bash Command Patterns](#bash-command-patterns)
4. [MCP Tool Patterns](#mcp-tool-patterns)
5. [Task Agent Patterns](#task-agent-patterns)
6. [File Path Patterns](#file-path-patterns)
7. [Security Considerations](#security-considerations)
8. [Anti-Patterns to Avoid](#anti-patterns-to-avoid)
9. [Testing Permissions](#testing-permissions)
10. [Common Configurations](#common-configurations)
11. [Troubleshooting](#troubleshooting)
12. [Quick Reference](#quick-reference)

---

## Overview

Wildcard permissions allow you to create flexible rules that match multiple commands or tools with a single pattern. This tutorial covers security-focused usage of wildcards to balance productivity with safety.

### Prerequisites

- Understanding of basic permissions (Tutorial 20)
- Familiarity with JSON configuration
- Knowledge of your team's security requirements

### Key Concepts

| Term | Description |
|------|-------------|
| Wildcard | `*` character matching any sequence |
| Pattern | Rule string with wildcards |
| Matcher | Pattern compared against tool calls |
| Precedence | Deny > Allow > Default mode |

---

## Wildcard Fundamentals

### Basic Syntax

```
*     Matches any sequence of characters (including empty)
```

**Examples:**

| Pattern | Matches | Doesn't Match |
|---------|---------|---------------|
| `npm *` | `npm install`, `npm test`, `npm run build` | `npx`, `yarn` |
| `git status *` | `git status`, `git status --short` | `git commit` |
| `* --help` | `npm --help`, `git --help` | `npm install` |
| `docker * ubuntu` | `docker run ubuntu`, `docker pull ubuntu` | `docker run alpine` |

### Pattern Matching Rules

1. **Wildcards match greedily** - `*` consumes as many characters as possible
2. **Literal characters match exactly** - `npm` only matches `npm`
3. **Spaces are significant** - `npm *` ≠ `npm*`
4. **Case sensitive** - `Git *` ≠ `git *`

### Configuration Location

```json
// ~/.claude/settings.json (user settings)
// .claude/settings.json (project settings)
{
  "permissions": {
    "tools": {
      "Bash": {
        "allow": ["pattern1", "pattern2"],
        "deny": ["pattern3"]
      }
    }
  }
}
```

---

## Bash Command Patterns

### Package Manager Patterns

```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "allow": [
          "npm install *",
          "npm test *",
          "npm run *",
          "npm ci",
          "yarn add *",
          "yarn install *",
          "yarn test *",
          "pnpm install *",
          "pnpm test *"
        ],
        "deny": [
          "npm publish *",
          "npm unpublish *",
          "npm deprecate *",
          "yarn publish *"
        ]
      }
    }
  }
}
```

### Git Patterns

```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "allow": [
          "git status *",
          "git diff *",
          "git log *",
          "git branch *",
          "git checkout *",
          "git switch *",
          "git add *",
          "git commit *",
          "git pull *",
          "git fetch *",
          "git stash *"
        ],
        "deny": [
          "git push --force*",
          "git push -f*",
          "git reset --hard*",
          "git clean -fd*",
          "git rebase*",
          "git filter-branch*"
        ]
      }
    }
  }
}
```

### Docker Patterns

```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "allow": [
          "docker build *",
          "docker run *",
          "docker ps *",
          "docker logs *",
          "docker images *",
          "docker-compose up *",
          "docker-compose down *"
        ],
        "deny": [
          "docker rm -f *",
          "docker system prune *",
          "docker * --privileged*",
          "docker * -v /:/host*"
        ]
      }
    }
  }
}
```

### Build Tool Patterns

```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "allow": [
          "make *",
          "cargo build *",
          "cargo test *",
          "cargo run *",
          "go build *",
          "go test *",
          "python -m pytest *",
          "python -m pip install *"
        ]
      }
    }
  }
}
```

### Compound Command Patterns (v2.1.7)

Block command chaining to prevent permission bypass:

```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "deny": [
          "*&&*",
          "*||*",
          "*;*",
          "*|*"
        ]
      }
    }
  }
}
```

**Why this matters:**

Without compound restrictions:
```bash
# Allowed individually:
npm test          # ✓
# But chained:
npm test && rm -rf /  # Could execute destructive command!
```

**More selective pipe blocking:**

```json
{
  "permissions": {
    "tools": {
      "Bash": {
        "allow": [
          "* | grep *",
          "* | head *",
          "* | tail *",
          "* | wc *",
          "* | sort *"
        ],
        "deny": [
          "*;*",
          "*&&*"
        ]
      }
    }
  }
}
```

---

## MCP Tool Patterns

### Server-Level Wildcards (v2.0.71)

Allow or deny entire MCP servers:

```json
{
  "permissions": {
    "tools": {
      "mcp__filesystem__*": {
        "defaultMode": "allow"
      },
      "mcp__database__*": {
        "defaultMode": "ask"
      },
      "mcp__dangerous__*": {
        "defaultMode": "deny"
      }
    }
  }
}
```

### Tool-Name Wildcards

Match specific tools across servers:

```json
{
  "permissions": {
    "tools": {
      "mcp__*__read*": {
        "defaultMode": "allow"
      },
      "mcp__*__list*": {
        "defaultMode": "allow"
      },
      "mcp__*__delete*": {
        "defaultMode": "deny"
      },
      "mcp__*__drop*": {
        "defaultMode": "deny"
      }
    }
  }
}
```

### Combined Patterns

```json
{
  "permissions": {
    "tools": {
      "mcp__github__*": {
        "defaultMode": "allow"
      },
      "mcp__github__delete*": {
        "defaultMode": "deny"
      },
      "mcp__*__*_production*": {
        "defaultMode": "deny"
      }
    }
  }
}
```

### MCP Permission Hierarchy

```
Most Specific     mcp__github__create_pr       → Matches exactly
        ↓         mcp__github__create*         → Matches create_*
        ↓         mcp__github__*               → Matches any github tool
        ↓         mcp__*__create*              → Matches create_* from any server
Least Specific    mcp__*__*                    → Matches all MCP tools
```

---

## Task Agent Patterns

### Agent Permission Syntax (v2.1.0)

Control which sub-agents can be spawned:

```json
{
  "permissions": {
    "tools": {
      "Task(Explore)": { "defaultMode": "allow" },
      "Task(Plan)": { "defaultMode": "allow" },
      "Task(builder)": { "defaultMode": "ask" },
      "Task(reviewer)": { "defaultMode": "allow" },
      "Task(fixer)": { "defaultMode": "ask" },
      "Task(test-writer)": { "defaultMode": "allow" },
      "Task(security-auditor)": { "defaultMode": "allow" },
      "Task(refactorer)": { "defaultMode": "ask" },
      "Task(*)": { "defaultMode": "deny" }
    }
  }
}
```

### Read-Only Agent Configuration

```json
{
  "permissions": {
    "tools": {
      "Task(Explore)": { "defaultMode": "allow" },
      "Task(Plan)": { "defaultMode": "allow" },
      "Task(reviewer)": { "defaultMode": "allow" },
      "Task(*)": { "defaultMode": "deny" }
    }
  }
}
```

### Full Agent Access

```json
{
  "permissions": {
    "tools": {
      "Task(*)": { "defaultMode": "allow" }
    }
  }
}
```

---

## File Path Patterns

### Read Tool Patterns

```json
{
  "permissions": {
    "tools": {
      "Read": {
        "deny": [
          ".env*",
          "*.pem",
          "*.key",
          "*.crt",
          "**/secrets/*",
          "**/credentials/*",
          "**/.aws/*",
          "**/.ssh/*",
          "**/node_modules/*"
        ]
      }
    }
  }
}
```

### Write Tool Patterns

```json
{
  "permissions": {
    "tools": {
      "Write": {
        "allow": [
          "src/**",
          "test/**",
          "docs/**"
        ],
        "deny": [
          ".env*",
          "*.lock",
          "package-lock.json",
          "yarn.lock",
          "**/dist/*",
          "**/build/*"
        ]
      }
    }
  }
}
```

### Glob Pattern Reference

| Pattern | Matches |
|---------|---------|
| `*` | Any characters in single path segment |
| `**` | Any path depth |
| `*.ext` | Files with extension |
| `**/dir/*` | Files in named directory at any depth |

---

## Security Considerations

### The Principle of Least Privilege

Start with deny-by-default, then add specific allows:

```json
{
  "permissions": {
    "defaultMode": "deny",
    "tools": {
      "Bash": {
        "defaultMode": "deny",
        "allow": [
          "npm test",
          "npm run build",
          "git status"
        ]
      }
    }
  }
}
```

### Overly Permissive Patterns to Avoid

```json
// ❌ DANGEROUS - Never use these
{
  "permissions": {
    "tools": {
      "Bash": {
        "allow": [
          "*",           // Allows ANY command
          "rm *",        // Allows deletion anywhere
          "sudo *"       // Allows privilege escalation
        ]
      },
      "mcp__*__*": {
        "defaultMode": "allow"  // Allows all MCP tools
      }
    }
  }
}
```

### Recommended Security Patterns

```json
// ✓ SAFE - Specific, limited patterns
{
  "permissions": {
    "tools": {
      "Bash": {
        "defaultMode": "ask",
        "allow": [
          "npm test",
          "npm run lint",
          "npm run build",
          "git status",
          "git diff"
        ],
        "deny": [
          "*sudo*",
          "*rm -rf*",
          "*> /dev/*",
          "* | sh",
          "* | bash",
          "*curl * | *",
          "*wget * | *"
        ]
      }
    }
  }
}
```

### Pattern Precedence

1. **Deny always wins** - A matching deny blocks even if allow matches
2. **More specific doesn't override** - `npm test` allowed + `npm *` denied = denied
3. **Order doesn't matter** - Rules processed as sets, not sequences

```json
// Example: This configuration denies npm test!
{
  "permissions": {
    "tools": {
      "Bash": {
        "allow": ["npm test"],  // Specific allow
        "deny": ["npm *"]       // Broader deny WINS
      }
    }
  }
}
```

---

## Anti-Patterns to Avoid

### Anti-Pattern 1: Too Broad Wildcards

```json
// ❌ BAD
{
  "Bash": {
    "allow": ["git *"]  // Includes git push --force!
  }
}

// ✓ GOOD
{
  "Bash": {
    "allow": [
      "git status *",
      "git diff *",
      "git log *",
      "git add *",
      "git commit *"
    ],
    "deny": [
      "git push --force*",
      "git reset --hard*"
    ]
  }
}
```

### Anti-Pattern 2: Deny Shadowing Allow

```json
// ❌ BAD - allow is unreachable
{
  "Bash": {
    "allow": ["git push origin main"],
    "deny": ["git push *"]  // Blocks everything including the allow!
  }
}

// ✓ GOOD - specific deny
{
  "Bash": {
    "allow": ["git push *"],
    "deny": ["git push --force*", "git push -f*"]
  }
}
```

### Anti-Pattern 3: Forgetting Compound Commands

```json
// ❌ BAD - can be bypassed
{
  "Bash": {
    "allow": ["npm test"]
    // Missing: deny compound commands
  }
}

// ✓ GOOD - block chaining
{
  "Bash": {
    "allow": ["npm test"],
    "deny": ["*&&*", "*;*"]
  }
}
```

### Anti-Pattern 4: Case Sensitivity Oversight

```json
// ❌ BAD - only matches lowercase
{
  "Bash": {
    "deny": ["rm *"]
  }
}

// ✓ GOOD - handle variations
{
  "Bash": {
    "deny": ["rm *", "RM *", "Rm *"]  // Or use hooks for complex validation
  }
}
```

---

## Testing Permissions

### Using /permissions Command (v2.0.67)

```bash
# Search for specific patterns
/permissions npm

# Search for MCP tools
/permissions mcp__github

# View all rules
/permissions *
```

### Using /doctor Command (v2.1.3)

Detect unreachable rules:

```bash
/doctor

# Output:
# ⚠️ Unreachable permission rules detected:
#   - Bash.allow: "git push" is shadowed by Bash.deny: "git *"
```

### Manual Testing

Test specific commands:

```bash
# Ask Claude to run a command
You: Run `npm test`

# Claude will either:
# - Execute (if allowed)
# - Ask permission (if in ask mode)
# - Refuse (if denied)
```

### Permission Test Matrix

Create a test document:

```markdown
## Permission Test Cases

| Command | Expected | Actual | Notes |
|---------|----------|--------|-------|
| `npm test` | allow | | |
| `npm publish` | deny | | |
| `git status` | allow | | |
| `git push --force` | deny | | |
| `rm -rf /` | deny | | |
| `npm test && echo "done"` | deny | | compound |
```

---

## Common Configurations

### Configuration 1: Safe Developer Setup

```json
{
  "permissions": {
    "defaultMode": "ask",
    "tools": {
      "Read": { "defaultMode": "allow" },
      "Glob": { "defaultMode": "allow" },
      "Grep": { "defaultMode": "allow" },
      "Edit": { "defaultMode": "ask" },
      "Write": { "defaultMode": "ask" },
      "Bash": {
        "defaultMode": "ask",
        "allow": [
          "npm test *",
          "npm run *",
          "npm install *",
          "npm ci",
          "git status *",
          "git diff *",
          "git log *",
          "git add *",
          "git commit *",
          "ls *",
          "pwd",
          "which *",
          "cat *",
          "head *",
          "tail *"
        ],
        "deny": [
          "*&&*",
          "*;*",
          "*sudo*",
          "*rm -rf*",
          "git push --force*",
          "git reset --hard*"
        ]
      },
      "Task(Explore)": { "defaultMode": "allow" },
      "Task(Plan)": { "defaultMode": "allow" },
      "Task(*)": { "defaultMode": "ask" }
    }
  }
}
```

### Configuration 2: CI/CD Pipeline

```json
{
  "permissions": {
    "defaultMode": "allow",
    "tools": {
      "Bash": {
        "defaultMode": "allow",
        "deny": [
          "*sudo*",
          "*rm -rf /*",
          "*> /etc/*",
          "git push --force*",
          "*curl * | bash*",
          "*wget * | bash*"
        ]
      }
    }
  }
}
```

### Configuration 3: Code Review Mode

```json
{
  "permissions": {
    "defaultMode": "deny",
    "tools": {
      "Read": { "defaultMode": "allow" },
      "Glob": { "defaultMode": "allow" },
      "Grep": { "defaultMode": "allow" },
      "Edit": { "defaultMode": "deny" },
      "Write": { "defaultMode": "deny" },
      "Bash": {
        "defaultMode": "deny",
        "allow": [
          "git diff *",
          "git log *",
          "git show *",
          "git blame *"
        ]
      },
      "Task(Explore)": { "defaultMode": "allow" },
      "Task(reviewer)": { "defaultMode": "allow" },
      "Task(*)": { "defaultMode": "deny" }
    }
  }
}
```

### Configuration 4: MCP-Heavy Workflow

```json
{
  "permissions": {
    "tools": {
      "mcp__github__*": { "defaultMode": "allow" },
      "mcp__github__delete*": { "defaultMode": "deny" },
      "mcp__database__query*": { "defaultMode": "allow" },
      "mcp__database__*": { "defaultMode": "ask" },
      "mcp__*__read*": { "defaultMode": "allow" },
      "mcp__*__list*": { "defaultMode": "allow" },
      "mcp__*__delete*": { "defaultMode": "deny" },
      "mcp__*__drop*": { "defaultMode": "deny" }
    }
  }
}
```

---

## Troubleshooting

### Permission Not Working as Expected

**Symptom:** Command is denied but you expected it to be allowed

**Check:**
1. Run `/permissions <command>` to see matching rules
2. Run `/doctor` to detect unreachable rules
3. Verify no broader deny pattern shadows your allow

**Fix:**
```json
// Remove or narrow the deny pattern
{
  "Bash": {
    "allow": ["npm test"],
    "deny": ["npm publish"]  // Specific, not "npm *"
  }
}
```

### Wildcard Too Greedy

**Symptom:** Pattern matches more than intended

**Check:**
```
Pattern: "npm *"
Matches: npm install, npm test, npm i malicious-package
```

**Fix:**
```json
// Use more specific patterns
{
  "Bash": {
    "allow": [
      "npm test *",
      "npm run *",
      "npm install --save-dev *"
    ]
  }
}
```

### MCP Tools Not Matching

**Symptom:** MCP wildcard doesn't apply

**Check:**
- Tool name format: `mcp__servername__toolname`
- Server name matches exactly
- No typos in pattern

**Fix:**
```json
{
  "mcp__github__*": { "defaultMode": "allow" }  // Not "mcp__GitHub__*"
}
```

---

## Quick Reference

### Wildcard Syntax

| Pattern | Meaning | Example Match |
|---------|---------|---------------|
| `npm *` | npm + space + anything | `npm test`, `npm install foo` |
| `git * main` | git + anything + main | `git checkout main` |
| `mcp__*__*` | All MCP tools | `mcp__github__create_pr` |
| `Task(*)` | Any agent | `Task(builder)` |
| `*.js` | Files ending .js | `index.js` |
| `**/test/*` | Files in test/ at any depth | `src/test/foo.ts` |

### Precedence Rules

```
1. Deny always beats Allow
2. More specific patterns don't override less specific
3. Rule order in arrays doesn't matter
4. Tool-specific beats default mode
5. Project settings beat user settings (for same key)
```

### Security Patterns

```json
{
  "deny": [
    "*sudo*",         // Block privilege escalation
    "*rm -rf*",       // Block recursive force delete
    "*&&*",           // Block command chaining
    "*;*",            // Block command sequence
    "*| sh",          // Block shell piping
    "*| bash",        // Block bash piping
    "*> /dev/*",      // Block device writes
    "*curl * | *",    // Block download-and-execute
    "*wget * | *"     // Block download-and-execute
  ]
}
```

### Common Tool Patterns

| Tool | Safe Allow Pattern | Common Deny Pattern |
|------|-------------------|---------------------|
| Bash | `npm test *` | `*sudo*` |
| Read | `src/**` | `.env*`, `*.key` |
| Write | `src/**`, `test/**` | `*.lock`, `.env*` |
| MCP | `mcp__server__read*` | `mcp__*__delete*` |
| Task | `Task(Explore)` | `Task(*)` (deny unknown) |

---

## Sources

- [Claude Code Permissions Documentation](https://code.claude.com/docs/en/settings.md)
- [Claude Code Changelog v2.0.71, v2.1.0, v2.1.7](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

---

*This tutorial is part of the Claude Code documentation series.*
*Version 2.1.x | Last Updated: January 2026*
