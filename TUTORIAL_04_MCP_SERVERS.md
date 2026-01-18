# Tutorial 04: Claude Code MCP (Model Context Protocol) Servers

## Table of Contents

1. [Introduction](#1-introduction)
2. [Installation & Setup](#2-installation--setup)
3. [MCP Configuration Format](#3-mcp-configuration-format)
4. [Popular MCP Servers](#4-popular-mcp-servers)
5. [Hand-Holding User Journey](#5-hand-holding-user-journey)
6. [Real-World Examples](#6-real-world-examples)
7. [Creating Custom MCP Servers](#7-creating-custom-mcp-servers)
8. [Pros and Cons](#8-pros-and-cons)
9. [When to Use MCP](#9-when-to-use-mcp)
10. [Comparison with Similar Features](#10-comparison-with-similar-features)
11. [Security Best Practices](#11-security-best-practices)
12. [Troubleshooting](#12-troubleshooting)
13. [Quick Reference](#13-quick-reference)

---

## 1. Introduction

### What is MCP (Model Context Protocol)?

The **Model Context Protocol (MCP)** is an open standard developed by Anthropic that enables AI assistants like Claude to securely connect with external data sources, tools, and services. Think of MCP as a universal adapter that allows Claude to interact with the outside world through a standardized interface.

MCP creates a bridge between Claude's intelligence and external systems, enabling Claude to:
- Read and write files on your system
- Query databases
- Interact with APIs
- Control browsers
- Send messages via Slack, Discord, or email
- Access version control systems
- And much more

### Why MCP Matters for Claude Code

Claude Code is powerful on its own, but MCP transforms it into an extensible platform:

```
┌─────────────────────────────────────────────────────────────────┐
│                        Claude Code                               │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   Core Capabilities                      │   │
│  │  • Code Generation    • File Editing                    │   │
│  │  • Bash Commands      • Code Analysis                   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                              ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    MCP Layer                             │   │
│  │         (Model Context Protocol Interface)               │   │
│  └─────────────────────────────────────────────────────────┘   │
│              │           │           │           │              │
│              ▼           ▼           ▼           ▼              │
│         ┌───────┐   ┌───────┐   ┌───────┐   ┌───────┐         │
│         │GitHub │   │ Slack │   │  DB   │   │Browser│         │
│         │  MCP  │   │  MCP  │   │  MCP  │   │  MCP  │         │
│         └───────┘   └───────┘   └───────┘   └───────┘         │
└─────────────────────────────────────────────────────────────────┘
```

Without MCP, you would need to:
- Manually copy data between Claude and external services
- Write custom integration scripts for each service
- Handle authentication and API quirks yourself
- Lose context when switching between tools

With MCP, Claude can directly interact with these services as naturally as it reads files.

### How MCP Extends Claude's Capabilities

MCP servers expose **tools** and **resources** to Claude:

| Component | Description | Example |
|-----------|-------------|---------|
| **Tools** | Actions Claude can perform | `github_create_pr`, `slack_send_message` |
| **Resources** | Data Claude can access | Database schemas, file contents |
| **Prompts** | Pre-defined interaction patterns | Code review templates |

When you configure an MCP server, Claude gains new abilities:

```
Before MCP:
  User: "Create a GitHub PR for these changes"
  Claude: "I can't directly interact with GitHub. Please manually..."

After MCP (with GitHub MCP):
  User: "Create a GitHub PR for these changes"
  Claude: *Uses github_create_pull_request tool*
  Claude: "I've created PR #142: 'Add user authentication feature'"
```

### MCP Architecture Overview

MCP follows a client-server architecture:

```
┌──────────────────────────────────────────────────────────────────┐
│                         Your Machine                              │
│                                                                   │
│  ┌────────────────────┐      JSON-RPC       ┌─────────────────┐ │
│  │    Claude Code     │◄───────────────────►│   MCP Server    │ │
│  │    (MCP Client)    │      over stdio     │   (npm package) │ │
│  └────────────────────┘                     └─────────────────┘ │
│           │                                          │           │
│           │                                          │           │
│           ▼                                          ▼           │
│    ┌─────────────┐                          ┌─────────────────┐ │
│    │ mcp.json    │                          │ External Service│ │
│    │ (config)    │                          │ (GitHub, Slack) │ │
│    └─────────────┘                          └─────────────────┘ │
└──────────────────────────────────────────────────────────────────┘
```

**Key Components:**

1. **MCP Client** (Claude Code): Discovers and connects to MCP servers
2. **MCP Server**: A process that exposes tools/resources via the MCP protocol
3. **Transport**: Communication channel (usually stdio for local servers)
4. **Configuration**: `mcp.json` file defining which servers to load

**Communication Flow:**

```
1. Claude Code starts → Reads mcp.json
2. For each server in config → Spawns server process
3. Server sends capability list → Claude learns available tools
4. User makes request → Claude decides to use MCP tool
5. Claude sends tool call → Server executes and returns result
6. Claude processes result → Continues conversation
```

---

## 2. Installation & Setup

### MCP Configuration File Location

MCP servers are configured in JSON files at specific locations:

```
Configuration Hierarchy (in order of precedence):

1. Project-level (highest priority):
   ./.claude/mcp.json           # Current project directory
   ./.mcp.json                   # Alternative project location

2. User-level:
   ~/.claude/mcp.json           # User's home directory (Linux/Mac)
   %USERPROFILE%\.claude\mcp.json  # Windows

3. Enterprise-level (managed):
   /etc/claude/mcp.json         # System-wide (Linux)
```

**Creating the configuration directory:**

```bash
# Linux/Mac
mkdir -p ~/.claude
touch ~/.claude/mcp.json

# Windows (PowerShell)
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude"
New-Item -ItemType File -Force -Path "$env:USERPROFILE\.claude\mcp.json"

# Windows (Command Prompt)
mkdir "%USERPROFILE%\.claude"
echo {} > "%USERPROFILE%\.claude\mcp.json"
```

### Installing MCP Servers via npm

Most MCP servers are distributed as npm packages. Here's how to install them:

**Prerequisites:**
- Node.js 18+ installed
- npm or pnpm package manager

**Global Installation (Recommended for most servers):**

```bash
# Install a single MCP server
npm install -g @anthropic/mcp-server-filesystem

# Install multiple servers at once
npm install -g @anthropic/mcp-server-filesystem \
               @anthropic/mcp-server-github \
               @anthropic/mcp-server-memory

# Using pnpm (faster, more efficient)
pnpm add -g @anthropic/mcp-server-filesystem
```

**Local Installation (Project-specific):**

```bash
# In your project directory
npm install --save-dev @anthropic/mcp-server-filesystem

# The server will be at: ./node_modules/.bin/mcp-server-filesystem
```

**Verify Installation:**

```bash
# Check if the server is installed
which mcp-server-filesystem  # Linux/Mac
where mcp-server-filesystem  # Windows

# List globally installed MCP servers
npm list -g | grep mcp-server

# Test server directly (should output JSON-RPC messages)
mcp-server-filesystem --help
```

### Verifying MCP Server Installation

After configuring MCP servers, verify they work with Claude Code:

**Method 1: Claude Code Status Command**

```bash
# Check MCP server status
claude mcp status

# Expected output:
# MCP Servers:
#   ✓ filesystem - running (3 tools available)
#   ✓ github - running (12 tools available)
#   ✗ slack - failed to start (check configuration)
```

**Method 2: List Available MCP Tools**

```bash
# Inside Claude Code session
/mcp

# Or ask Claude directly:
"What MCP tools do you have available?"
```

**Method 3: Test a Simple Tool Call**

```bash
# Start Claude Code
claude

# Then ask:
"Use your filesystem MCP to list files in the current directory"
```

### Troubleshooting Installation Issues

**Issue 1: "Command not found" after installation**

```bash
# Problem: npm global bin not in PATH

# Solution 1: Find and add to PATH
npm config get prefix
# Add {prefix}/bin to your PATH

# Solution 2: Use npx instead
npx @anthropic/mcp-server-filesystem

# Solution 3: Link the package
npm link @anthropic/mcp-server-filesystem
```

**Issue 2: Permission errors on Linux/Mac**

```bash
# Problem: Global install requires sudo

# Solution: Configure npm to use local directory
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# Then reinstall without sudo
npm install -g @anthropic/mcp-server-filesystem
```

**Issue 3: Node.js version incompatibility**

```bash
# Problem: MCP servers require Node.js 18+

# Check your version
node --version

# Solution: Use nvm to install correct version
nvm install 20
nvm use 20
```

**Issue 4: Server crashes immediately**

```bash
# Debug by running server manually
mcp-server-filesystem 2>&1 | head -20

# Check for missing environment variables
echo $GITHUB_TOKEN  # Should not be empty for GitHub MCP

# Run with verbose logging
DEBUG=* mcp-server-filesystem
```

---

## 3. MCP Configuration Format

### JSON Structure Explained

The `mcp.json` file defines which MCP servers Claude Code should load and how to configure them:

```json
{
  "mcpServers": {
    "server-name": {
      "command": "executable-path",
      "args": ["arg1", "arg2"],
      "env": {
        "ENV_VAR": "value"
      },
      "cwd": "/working/directory",
      "disabled": false
    }
  }
}
```

**Root Structure:**

| Field | Type | Description |
|-------|------|-------------|
| `mcpServers` | object | Container for all server definitions |

### Server Definitions

Each server is defined by a unique name and configuration object:

```json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["/path/to/server.js"],
      "env": {},
      "cwd": null,
      "disabled": false
    }
  }
}
```

**Server Configuration Fields:**

| Field | Required | Type | Description |
|-------|----------|------|-------------|
| `command` | Yes | string | Executable to run (e.g., `node`, `python`, `npx`) |
| `args` | No | array | Command-line arguments passed to the executable |
| `env` | No | object | Environment variables for the server process |
| `cwd` | No | string | Working directory for the server |
| `disabled` | No | boolean | Set `true` to temporarily disable without removing |

### Environment Variables

Environment variables can be specified inline or reference system variables:

```json
{
  "mcpServers": {
    "github": {
      "command": "mcp-server-github",
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}",
        "GITHUB_API_URL": "https://api.github.com",
        "DEBUG": "true"
      }
    }
  }
}
```

**Variable Substitution Syntax:**

```json
{
  "env": {
    "TOKEN": "${MY_TOKEN}",
    "PATH_VAR": "${HOME}/custom/path",
    "COMBINED": "${USER}@${HOSTNAME}"
  }
}
```

**Note:** The `${VAR}` syntax reads from your system environment. Ensure these variables are set before starting Claude Code:

```bash
# In your shell profile (~/.bashrc or ~/.zshrc)
export GITHUB_TOKEN="ghp_xxxxxxxxxxxx"
export SLACK_TOKEN="xoxb-xxxxxxxxxxxx"
```

### Arguments and Options

Arguments are passed as an array of strings:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "mcp-server-filesystem",
      "args": [
        "--root", "/home/user/projects",
        "--allowed-extensions", ".js,.ts,.py",
        "--max-file-size", "10MB",
        "--verbose"
      ]
    }
  }
}
```

**Common Argument Patterns:**

```json
{
  "args": [
    "/path/to/script.js",
    "--flag",
    "--option", "value",
    "--option=value",
    "-c", "config.json"
  ]
}
```

### Complete Annotated Example

Here's a comprehensive `mcp.json` with detailed comments (note: actual JSON doesn't support comments):

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "mcp-server-filesystem",
      "args": [
        "/home/user/projects",
        "/home/user/documents"
      ],
      "env": {
        "MCP_LOG_LEVEL": "info"
      }
    },

    "github": {
      "command": "npx",
      "args": [
        "-y",
        "@anthropic/mcp-server-github"
      ],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}",
        "GITHUB_OWNER": "myorg",
        "GITHUB_REPO": "myrepo"
      }
    },

    "postgres": {
      "command": "mcp-server-postgres",
      "args": [
        "--connection-string",
        "${DATABASE_URL}"
      ],
      "env": {
        "PGPASSWORD": "${DB_PASSWORD}"
      }
    },

    "slack": {
      "command": "node",
      "args": [
        "/opt/mcp-servers/slack/index.js"
      ],
      "env": {
        "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}",
        "SLACK_CHANNEL": "dev-notifications"
      }
    },

    "memory": {
      "command": "mcp-server-memory",
      "args": [],
      "env": {
        "MEMORY_FILE": "${HOME}/.claude/memory.json"
      }
    },

    "disabled-server": {
      "command": "mcp-server-experimental",
      "args": [],
      "disabled": true
    }
  }
}
```

**Explanation of Each Server:**

| Server | Purpose | Key Configuration |
|--------|---------|-------------------|
| `filesystem` | File access | Paths to allowed directories |
| `github` | GitHub API | Token and default repo |
| `postgres` | Database queries | Connection string |
| `slack` | Messaging | Bot token and channel |
| `memory` | Persistent storage | Path to memory file |
| `disabled-server` | Temporarily disabled | `disabled: true` |

---

## 4. Popular MCP Servers

### Filesystem MCP - File Operations

The Filesystem MCP server provides secure file system access with configurable permissions.

**Installation:**

```bash
npm install -g @anthropic/mcp-server-filesystem
```

**Configuration:**

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "mcp-server-filesystem",
      "args": [
        "/home/user/projects",
        "/home/user/documents",
        "/tmp"
      ]
    }
  }
}
```

**Available Tools:**

| Tool | Description |
|------|-------------|
| `read_file` | Read contents of a file |
| `write_file` | Write content to a file |
| `list_directory` | List files in a directory |
| `create_directory` | Create a new directory |
| `move_file` | Move or rename a file |
| `delete_file` | Delete a file |
| `search_files` | Search for files by pattern |
| `get_file_info` | Get file metadata |

**Use Cases:**

1. **Project scaffolding**: Create directory structures for new projects
2. **File search**: Find files matching patterns across directories
3. **Batch operations**: Rename, move, or organize files
4. **Configuration management**: Read and update config files

**Example Usage:**

```
User: "Find all Python files in my projects folder modified in the last week"

Claude: *Uses filesystem.search_files and filesystem.get_file_info*
"I found 23 Python files modified recently. Here are the most recent..."
```

---

### GitHub MCP - Repository Operations

The GitHub MCP server enables full GitHub integration for repository management.

**Installation:**

```bash
npm install -g @anthropic/mcp-server-github
```

**Configuration:**

```json
{
  "mcpServers": {
    "github": {
      "command": "mcp-server-github",
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

**Getting a GitHub Token:**

1. Go to GitHub Settings > Developer Settings > Personal Access Tokens
2. Generate a new token with required scopes:
   - `repo` - Full repository access
   - `workflow` - GitHub Actions
   - `read:org` - Organization read access

**Available Tools:**

| Tool | Description |
|------|-------------|
| `create_repository` | Create a new repository |
| `list_repositories` | List user/org repositories |
| `create_pull_request` | Create a PR |
| `list_pull_requests` | List PRs for a repo |
| `merge_pull_request` | Merge a PR |
| `create_issue` | Create an issue |
| `list_issues` | List issues |
| `create_branch` | Create a new branch |
| `get_file_contents` | Read file from repo |
| `push_files` | Push changes to repo |
| `search_code` | Search code across repos |

**Use Cases:**

1. **PR Management**: Create, review, and merge pull requests
2. **Issue Tracking**: Create and manage issues
3. **Code Search**: Find code patterns across repositories
4. **Repository Setup**: Create repos with templates

---

### Database MCP - Database Queries

Database MCP servers allow Claude to query and interact with databases.

**PostgreSQL Installation:**

```bash
npm install -g @anthropic/mcp-server-postgres
```

**PostgreSQL Configuration:**

```json
{
  "mcpServers": {
    "postgres": {
      "command": "mcp-server-postgres",
      "args": [
        "postgresql://user:pass@localhost:5432/mydb"
      ]
    }
  }
}
```

**SQLite Installation:**

```bash
npm install -g @anthropic/mcp-server-sqlite
```

**SQLite Configuration:**

```json
{
  "mcpServers": {
    "sqlite": {
      "command": "mcp-server-sqlite",
      "args": [
        "--database", "/path/to/database.db"
      ]
    }
  }
}
```

**Available Tools:**

| Tool | Description |
|------|-------------|
| `query` | Execute SELECT queries |
| `execute` | Execute INSERT/UPDATE/DELETE |
| `list_tables` | List all tables |
| `describe_table` | Get table schema |
| `create_table` | Create a new table |

**Use Cases:**

1. **Data Analysis**: Query and analyze database contents
2. **Schema Exploration**: Understand database structure
3. **Data Migration**: Transform and move data
4. **Report Generation**: Create summaries from data

**Security Note:** Always use read-only connections when possible:

```json
{
  "args": [
    "postgresql://readonly_user:pass@localhost:5432/mydb"
  ]
}
```

---

### Slack MCP - Messaging

The Slack MCP server enables Claude to send and receive Slack messages.

**Installation:**

```bash
npm install -g @anthropic/mcp-server-slack
```

**Configuration:**

```json
{
  "mcpServers": {
    "slack": {
      "command": "mcp-server-slack",
      "env": {
        "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}",
        "SLACK_TEAM_ID": "${SLACK_TEAM_ID}"
      }
    }
  }
}
```

**Setting Up Slack Bot:**

1. Go to api.slack.com/apps
2. Create a new app
3. Add Bot Token Scopes:
   - `chat:write` - Send messages
   - `channels:read` - List channels
   - `channels:history` - Read messages
4. Install app to workspace
5. Copy the Bot User OAuth Token

**Available Tools:**

| Tool | Description |
|------|-------------|
| `send_message` | Send a message to a channel |
| `list_channels` | List available channels |
| `read_messages` | Read recent messages |
| `reply_to_thread` | Reply in a thread |
| `add_reaction` | Add emoji reaction |
| `upload_file` | Upload a file |

**Use Cases:**

1. **Notifications**: Alert team when tasks complete
2. **Standup Reports**: Post automated status updates
3. **Error Alerts**: Notify on build failures
4. **Collaboration**: Share code snippets

---

### Browser MCP - Web Automation

The Browser MCP server provides web automation capabilities using Playwright.

**Installation:**

```bash
npm install -g @anthropic/mcp-server-puppeteer
# or
npm install -g @anthropic/mcp-server-playwright
```

**Configuration:**

```json
{
  "mcpServers": {
    "browser": {
      "command": "mcp-server-puppeteer",
      "args": [],
      "env": {
        "PUPPETEER_HEADLESS": "true"
      }
    }
  }
}
```

**Available Tools:**

| Tool | Description |
|------|-------------|
| `navigate` | Go to a URL |
| `screenshot` | Take a screenshot |
| `click` | Click an element |
| `type` | Type text into input |
| `get_content` | Get page content |
| `evaluate` | Run JavaScript |
| `wait_for_selector` | Wait for element |

**Use Cases:**

1. **Web Scraping**: Extract data from websites
2. **Testing**: Automated UI testing
3. **Form Filling**: Automate form submissions
4. **Screenshots**: Capture web pages

---

### Memory MCP - Persistent Memory

The Memory MCP server provides persistent storage for Claude across sessions.

**Installation:**

```bash
npm install -g @anthropic/mcp-server-memory
```

**Configuration:**

```json
{
  "mcpServers": {
    "memory": {
      "command": "mcp-server-memory",
      "args": [],
      "env": {
        "MEMORY_FILE_PATH": "${HOME}/.claude/memory.json"
      }
    }
  }
}
```

**Available Tools:**

| Tool | Description |
|------|-------------|
| `store` | Store a key-value pair |
| `retrieve` | Retrieve a value by key |
| `delete` | Delete a stored value |
| `list_keys` | List all stored keys |
| `search` | Search stored content |

**Use Cases:**

1. **User Preferences**: Remember user settings
2. **Project Context**: Store project-specific information
3. **Learning**: Remember corrections and preferences
4. **Session Data**: Persist data across conversations

---

## 5. Hand-Holding User Journey

This section walks you through setting up MCP from scratch with exact commands and configurations.

### Step 1: Create mcp.json Configuration

**Create the configuration directory and file:**

```bash
# Linux/Mac
mkdir -p ~/.claude
cat > ~/.claude/mcp.json << 'EOF'
{
  "mcpServers": {}
}
EOF

# Windows (PowerShell)
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude"
@'
{
  "mcpServers": {}
}
'@ | Out-File -FilePath "$env:USERPROFILE\.claude\mcp.json" -Encoding UTF8
```

**Verify the file was created:**

```bash
# Linux/Mac
cat ~/.claude/mcp.json

# Windows
type "%USERPROFILE%\.claude\mcp.json"
```

### Step 2: Install Your First MCP Server

Let's start with the Filesystem MCP server - it's simple and immediately useful.

**Install the server:**

```bash
npm install -g @anthropic/mcp-server-filesystem
```

**Update your mcp.json:**

```bash
# Linux/Mac
cat > ~/.claude/mcp.json << 'EOF'
{
  "mcpServers": {
    "filesystem": {
      "command": "mcp-server-filesystem",
      "args": [
        "/home/user/projects",
        "/home/user/documents"
      ]
    }
  }
}
EOF
```

**For Windows users, update with your paths:**

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "mcp-server-filesystem",
      "args": [
        "C:\\Users\\YourName\\Projects",
        "C:\\Users\\YourName\\Documents"
      ]
    }
  }
}
```

### Step 3: Test the MCP Server

**Test the server independently:**

```bash
# Run the server directly to check for errors
mcp-server-filesystem /tmp 2>&1 | head -5

# You should see JSON-RPC initialization messages
```

**Test with Claude Code:**

```bash
# Start Claude Code
claude

# Check MCP status
/mcp

# You should see:
# Available MCP Servers:
#   filesystem: running
#     Tools: read_file, write_file, list_directory, ...
```

### Step 4: Use MCP Tools in Conversation

Now let's use the MCP tools in a real conversation:

```
You: "Using your filesystem MCP, list all files in /tmp"

Claude: I'll use the filesystem MCP to list the contents of /tmp.

*Uses list_directory tool*

Here are the files in /tmp:
- temp_file_1.txt (1.2 KB)
- cache_data/ (directory)
- session_abc123.json (4.5 KB)
...
```

**More examples to try:**

```
"Create a new directory called 'test-project' in my projects folder"

"Read the contents of package.json in my current project"

"Find all .md files in my documents folder"
```

### Step 5: Add More MCP Servers

Let's add GitHub MCP to your configuration.

**Set up your GitHub token:**

```bash
# Linux/Mac - add to ~/.bashrc or ~/.zshrc
export GITHUB_TOKEN="ghp_your_token_here"
source ~/.bashrc

# Windows - set environment variable
setx GITHUB_TOKEN "ghp_your_token_here"
# Restart terminal after setting
```

**Update mcp.json:**

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "mcp-server-filesystem",
      "args": [
        "/home/user/projects"
      ]
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
```

**Restart Claude Code and verify:**

```bash
claude
# Then type:
/mcp

# Should show both filesystem and github servers
```

### Step 6: Create a Custom MCP Server (Bonus)

Let's create a simple custom MCP server that provides a "fortune" tool.

**Create the server file:**

```bash
mkdir -p ~/mcp-servers
cat > ~/mcp-servers/fortune-server.js << 'EOF'
#!/usr/bin/env node

const fortunes = [
  "The best code is no code at all.",
  "Premature optimization is the root of all evil.",
  "First, solve the problem. Then, write the code.",
  "Code is like humor. When you have to explain it, it's bad.",
  "Any fool can write code that a computer can understand."
];

const readline = require('readline');

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
  terminal: false
});

function sendResponse(id, result) {
  const response = {
    jsonrpc: "2.0",
    id,
    result
  };
  console.log(JSON.stringify(response));
}

function handleRequest(request) {
  const { id, method, params } = request;

  switch (method) {
    case "initialize":
      sendResponse(id, {
        protocolVersion: "2024-11-05",
        capabilities: { tools: {} },
        serverInfo: { name: "fortune-server", version: "1.0.0" }
      });
      break;

    case "tools/list":
      sendResponse(id, {
        tools: [{
          name: "get_fortune",
          description: "Get a random programming fortune",
          inputSchema: { type: "object", properties: {} }
        }]
      });
      break;

    case "tools/call":
      if (params.name === "get_fortune") {
        const fortune = fortunes[Math.floor(Math.random() * fortunes.length)];
        sendResponse(id, {
          content: [{ type: "text", text: fortune }]
        });
      }
      break;

    default:
      sendResponse(id, {});
  }
}

rl.on('line', (line) => {
  try {
    const request = JSON.parse(line);
    handleRequest(request);
  } catch (e) {
    // Ignore parse errors
  }
});
EOF

chmod +x ~/mcp-servers/fortune-server.js
```

**Add to mcp.json:**

```json
{
  "mcpServers": {
    "fortune": {
      "command": "node",
      "args": ["/home/user/mcp-servers/fortune-server.js"]
    }
  }
}
```

**Test your custom server:**

```
You: "Give me a programming fortune"

Claude: *Uses get_fortune tool*

Here's a programming fortune for you:

"The best code is no code at all."

This is a reminder that sometimes the best solution is to avoid
writing unnecessary code. Simplicity is key!
```

---

## 6. Real-World Examples

### Example 1: GitHub Integration for PR Management

**Scenario:** You want Claude to help manage pull requests across your repositories.

**Configuration:**

```json
{
  "mcpServers": {
    "github": {
      "command": "mcp-server-github",
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

**Workflow:**

```
User: "List all open PRs in myorg/backend that need review"

Claude: *Uses github.list_pull_requests with filters*

I found 5 open PRs needing review:

1. PR #234 - "Add user authentication" by @alice
   - 3 files changed, +150/-20
   - No reviews yet
   - Last updated: 2 hours ago

2. PR #231 - "Fix database connection pooling" by @bob
   - 1 file changed, +25/-10
   - 1 approval, needs 1 more
   - Last updated: 1 day ago
...

Would you like me to:
- Review any of these PRs?
- Merge any that are ready?
- Add reviewers?
```

**Advanced PR Workflow:**

```
User: "Create a PR from feature/auth to main with a summary of all commits"

Claude:
*Uses github.get_commits to analyze changes*
*Uses github.create_pull_request*

I've created PR #250: "Feature: User Authentication System"

Summary I generated from the commits:
- Added JWT token generation and validation
- Created login/logout endpoints
- Added password hashing with bcrypt
- Created user session management
- Added rate limiting for auth endpoints

The PR is ready for review: https://github.com/myorg/backend/pull/250
```

---

### Example 2: Database Querying with SQL MCP

**Scenario:** You have a PostgreSQL database and want Claude to help with data analysis.

**Configuration:**

```json
{
  "mcpServers": {
    "postgres": {
      "command": "mcp-server-postgres",
      "args": [
        "postgresql://analyst:${DB_PASSWORD}@localhost:5432/analytics"
      ],
      "env": {
        "DB_PASSWORD": "${DB_PASSWORD}"
      }
    }
  }
}
```

**Workflow:**

```
User: "What are our top 10 customers by revenue this month?"

Claude: *Uses postgres.query*

Here are your top 10 customers by revenue for January 2025:

| Rank | Customer        | Revenue    | Orders |
|------|-----------------|------------|--------|
| 1    | Acme Corp       | $45,230.00 | 23     |
| 2    | TechStart Inc   | $38,150.00 | 15     |
| 3    | GlobalTrade Ltd | $31,890.00 | 42     |
...

Notable insights:
- Acme Corp increased 25% from last month
- TechStart Inc is a new customer this month
- Average order value is $1,523.40
```

**Schema Exploration:**

```
User: "Help me understand the orders table structure"

Claude: *Uses postgres.describe_table*

The `orders` table has the following structure:

| Column        | Type         | Nullable | Description              |
|---------------|--------------|----------|--------------------------|
| id            | uuid         | NO       | Primary key              |
| customer_id   | uuid         | NO       | FK to customers          |
| total_amount  | decimal(10,2)| NO       | Order total              |
| status        | varchar(20)  | NO       | pending/completed/cancelled |
| created_at    | timestamp    | NO       | Order creation time      |
| updated_at    | timestamp    | YES      | Last modification        |

Indexes:
- PRIMARY KEY on id
- INDEX on customer_id
- INDEX on created_at
- INDEX on status

This table has 1.2M rows and is related to:
- customers (customer_id → customers.id)
- order_items (orders.id → order_items.order_id)
```

---

### Example 3: Slack Notifications on Task Completion

**Scenario:** You want Claude to notify your team when long-running tasks complete.

**Configuration:**

```json
{
  "mcpServers": {
    "slack": {
      "command": "mcp-server-slack",
      "env": {
        "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}",
        "SLACK_TEAM_ID": "${SLACK_TEAM_ID}"
      }
    }
  }
}
```

**Workflow:**

```
User: "Run the full test suite and notify #dev-team when done"

Claude:
*Runs pytest with full coverage*
...
All 342 tests passed in 4m 23s

*Uses slack.send_message*

I've notified the team. Here's the message I sent to #dev-team:

────────────────────────────────────────
🧪 Test Suite Complete

✅ All 342 tests passed
📊 Coverage: 87.3%
⏱️ Duration: 4m 23s

No failures or warnings.
────────────────────────────────────────
```

**Build Failure Notification:**

```
User: "Deploy to staging and alert the team of any issues"

Claude:
*Runs deployment commands*
...
ERROR: Deployment failed - database migration error

*Uses slack.send_message to #dev-alerts*

I've sent an alert to #dev-alerts:

────────────────────────────────────────
🚨 Staging Deployment Failed

Error: Database migration failed
- Migration: 2025_01_15_add_user_preferences
- Error: Column 'settings' already exists

Action needed: Review migration script
Logs: /var/log/deploy/staging-2025-01-15.log
────────────────────────────────────────

Would you like me to investigate the migration issue?
```

---

### Example 4: Browser Automation for Testing

**Scenario:** You want Claude to perform visual regression testing on your web application.

**Configuration:**

```json
{
  "mcpServers": {
    "browser": {
      "command": "mcp-server-puppeteer",
      "env": {
        "PUPPETEER_HEADLESS": "true",
        "SCREENSHOT_DIR": "/tmp/screenshots"
      }
    }
  }
}
```

**Workflow:**

```
User: "Test the login flow on our staging site and screenshot each step"

Claude:
*Uses browser.navigate to staging.myapp.com/login*
*Uses browser.screenshot - saved as 01-login-page.png*

*Uses browser.type to enter test credentials*
*Uses browser.screenshot - saved as 02-credentials-entered.png*

*Uses browser.click on submit button*
*Uses browser.wait_for_selector for dashboard*
*Uses browser.screenshot - saved as 03-dashboard.png*

Login flow test completed successfully!

Screenshots captured:
1. 01-login-page.png - Initial login page
2. 02-credentials-entered.png - Form filled
3. 03-dashboard.png - Successful login, dashboard visible

All elements loaded correctly. Login time: 1.2 seconds.
No console errors detected.
```

**Form Testing:**

```
User: "Test our contact form with various input types"

Claude:
*Uses browser.navigate to myapp.com/contact*

Testing form validation...

Test 1: Empty submission
*Uses browser.click submit*
Result: ✅ Validation error shown for required fields

Test 2: Invalid email
*Uses browser.type "notanemail" into email field*
*Uses browser.click submit*
Result: ✅ Email validation error shown

Test 3: Valid submission
*Uses browser.type valid data into all fields*
*Uses browser.click submit*
Result: ✅ Success message displayed

All form validation tests passed!
```

---

### Example 5: Custom API Integration

**Scenario:** You want Claude to interact with your internal API for deployment management.

**Custom MCP Server (deploy-server.js):**

```javascript
#!/usr/bin/env node

const https = require('https');

const DEPLOY_API = process.env.DEPLOY_API_URL;
const DEPLOY_TOKEN = process.env.DEPLOY_API_TOKEN;

// MCP server implementation
const tools = [
  {
    name: "list_deployments",
    description: "List recent deployments",
    inputSchema: {
      type: "object",
      properties: {
        environment: { type: "string", enum: ["staging", "production"] },
        limit: { type: "number", default: 10 }
      }
    }
  },
  {
    name: "trigger_deployment",
    description: "Trigger a new deployment",
    inputSchema: {
      type: "object",
      properties: {
        environment: { type: "string", enum: ["staging", "production"] },
        version: { type: "string" },
        notify: { type: "boolean", default: true }
      },
      required: ["environment", "version"]
    }
  },
  {
    name: "rollback_deployment",
    description: "Rollback to a previous deployment",
    inputSchema: {
      type: "object",
      properties: {
        environment: { type: "string" },
        deployment_id: { type: "string" }
      },
      required: ["environment", "deployment_id"]
    }
  }
];

// ... MCP protocol handling code ...
```

**Configuration:**

```json
{
  "mcpServers": {
    "deploy": {
      "command": "node",
      "args": ["/opt/mcp-servers/deploy-server.js"],
      "env": {
        "DEPLOY_API_URL": "https://deploy.internal.mycompany.com",
        "DEPLOY_API_TOKEN": "${DEPLOY_API_TOKEN}"
      }
    }
  }
}
```

**Workflow:**

```
User: "What's the current deployment status for production?"

Claude: *Uses deploy.list_deployments for production*

Current Production Deployment Status:

| Version | Deployed At        | Status  | Deployed By |
|---------|-------------------|---------|-------------|
| v2.4.1  | 2025-01-15 14:30  | Active  | @sarah      |
| v2.4.0  | 2025-01-14 09:15  | Rolled back | @mike   |
| v2.3.9  | 2025-01-13 16:45  | Inactive | @sarah     |

Current active version: v2.4.1
Uptime: 18 hours
Health: All checks passing

User: "Roll back production to v2.3.9"

Claude:
*Uses deploy.rollback_deployment*

⚠️ Rollback initiated for production to v2.3.9

Progress:
1. ✅ Stopping current deployment (v2.4.1)
2. ✅ Restoring v2.3.9 containers
3. ✅ Health checks passing
4. ✅ Traffic switched

Rollback complete. Production is now running v2.3.9.
Previous version v2.4.1 has been marked as rolled back.
```

---

## 7. Creating Custom MCP Servers

### MCP SDK Overview

The MCP SDK provides tools for building custom MCP servers in multiple languages:

```
Available SDKs:
├── @anthropic/mcp-sdk (TypeScript/JavaScript)
├── mcp-python (Python)
├── mcp-rust (Rust)
└── mcp-go (Go) - Community maintained
```

**TypeScript SDK Installation:**

```bash
npm install @anthropic/mcp-sdk
```

**Python SDK Installation:**

```bash
pip install mcp
```

### Server Structure

A basic MCP server has this structure:

```
my-mcp-server/
├── package.json
├── tsconfig.json
├── src/
│   └── index.ts
└── README.md
```

**package.json:**

```json
{
  "name": "my-mcp-server",
  "version": "1.0.0",
  "type": "module",
  "bin": {
    "my-mcp-server": "./dist/index.js"
  },
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js"
  },
  "dependencies": {
    "@anthropic/mcp-sdk": "^1.0.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0"
  }
}
```

### Defining Tools

**TypeScript Example (src/index.ts):**

```typescript
import { Server } from '@anthropic/mcp-sdk/server';
import { StdioServerTransport } from '@anthropic/mcp-sdk/server/stdio';

// Create server instance
const server = new Server({
  name: 'my-mcp-server',
  version: '1.0.0'
});

// Define a tool
server.tool(
  'greet',
  'Greet a user by name',
  {
    type: 'object',
    properties: {
      name: {
        type: 'string',
        description: 'Name of the person to greet'
      },
      formal: {
        type: 'boolean',
        description: 'Use formal greeting',
        default: false
      }
    },
    required: ['name']
  },
  async ({ name, formal }) => {
    const greeting = formal
      ? `Good day, ${name}. How may I assist you?`
      : `Hey ${name}! What's up?`;

    return {
      content: [
        {
          type: 'text',
          text: greeting
        }
      ]
    };
  }
);

// Define another tool with external API call
server.tool(
  'get_weather',
  'Get current weather for a location',
  {
    type: 'object',
    properties: {
      city: { type: 'string' },
      units: { type: 'string', enum: ['celsius', 'fahrenheit'], default: 'celsius' }
    },
    required: ['city']
  },
  async ({ city, units }) => {
    // Call weather API
    const response = await fetch(
      `https://api.weather.com/v1/current?city=${city}&units=${units}`
    );
    const data = await response.json();

    return {
      content: [
        {
          type: 'text',
          text: `Weather in ${city}: ${data.temperature}° ${units}, ${data.conditions}`
        }
      ]
    };
  }
);

// Start the server
const transport = new StdioServerTransport();
server.connect(transport);
```

**Python Example:**

```python
#!/usr/bin/env python3

from mcp import Server, Tool
from mcp.transports import StdioTransport

server = Server("my-python-mcp-server", "1.0.0")

@server.tool(
    name="calculate",
    description="Perform mathematical calculations",
    input_schema={
        "type": "object",
        "properties": {
            "expression": {
                "type": "string",
                "description": "Mathematical expression to evaluate"
            }
        },
        "required": ["expression"]
    }
)
async def calculate(expression: str):
    try:
        # Safe evaluation (use a proper math parser in production)
        result = eval(expression, {"__builtins__": {}}, {})
        return {"content": [{"type": "text", "text": str(result)}]}
    except Exception as e:
        return {"content": [{"type": "text", "text": f"Error: {e}"}]}

if __name__ == "__main__":
    transport = StdioTransport()
    server.run(transport)
```

### Publishing to npm

**Prepare for publishing:**

```bash
# Build the project
npm run build

# Test locally
node dist/index.js

# Login to npm
npm login

# Publish
npm publish --access public
```

**Naming Convention:**

```
@your-org/mcp-server-{name}
# or
mcp-server-{name}
```

### Local Development

**Testing your server:**

```bash
# Run directly
node dist/index.js

# Test with sample input
echo '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | node dist/index.js

# Use with Claude Code (local path)
```

**Development Configuration:**

```json
{
  "mcpServers": {
    "dev-server": {
      "command": "npx",
      "args": ["tsx", "/path/to/my-mcp-server/src/index.ts"]
    }
  }
}
```

---

## 8. Pros and Cons

### Extensibility Benefits

| Benefit | Description |
|---------|-------------|
| **Unlimited Integrations** | Connect Claude to any API, database, or service |
| **Standardized Protocol** | One protocol to learn, works with all MCP servers |
| **Community Ecosystem** | Growing library of pre-built servers |
| **Language Agnostic** | Write servers in TypeScript, Python, Go, Rust |
| **Local Execution** | Servers run locally, data stays on your machine |
| **Hot Reload** | Restart servers without restarting Claude Code |

### Security Considerations

| Consideration | Description | Mitigation |
|---------------|-------------|------------|
| **Credential Exposure** | Servers need API tokens | Use environment variables, not hardcoded values |
| **Data Access** | Servers can access local files | Limit paths in filesystem MCP |
| **Network Access** | Servers can make HTTP requests | Run in sandboxed environment |
| **Code Execution** | Custom servers run arbitrary code | Review server source code |
| **Permission Scope** | Servers may have broad permissions | Use least-privilege tokens |

### Performance Impact

| Factor | Impact | Notes |
|--------|--------|-------|
| **Startup Time** | +1-5s per server | Servers spawn on Claude Code start |
| **Memory Usage** | +50-200MB per server | Node.js process overhead |
| **Latency** | +10-100ms per tool call | JSON-RPC communication |
| **Concurrent Calls** | Limited by server | Most servers are single-threaded |

### Complexity Trade-offs

| Complexity | Without MCP | With MCP |
|------------|-------------|----------|
| **Configuration** | None | Requires mcp.json setup |
| **Dependencies** | None | Requires npm packages |
| **Debugging** | Standard tools | Additional MCP debugging |
| **Updates** | N/A | Must update MCP servers |
| **Maintenance** | N/A | Monitor server health |

---

## 9. When to Use MCP

### Ideal Scenarios

**Use MCP when you need:**

1. **Persistent External Connections**
   - Database connections that stay open
   - WebSocket connections
   - Authenticated API sessions

2. **Complex API Interactions**
   - Multi-step API workflows
   - APIs with complex authentication
   - Rate-limited services needing smart handling

3. **Specialized Tools**
   - Browser automation
   - Image processing
   - File format conversions

4. **Cross-Session State**
   - Remember user preferences
   - Track project context
   - Maintain conversation history

5. **Team Collaboration**
   - Slack/Discord notifications
   - GitHub PR management
   - Jira ticket updates

### MCP vs Bash Commands

| Use Bash When | Use MCP When |
|---------------|--------------|
| One-off commands | Repeated operations |
| Simple file operations | Complex file workflows |
| Standard CLI tools | Custom integrations |
| No authentication needed | Authenticated services |
| Quick scripts | Persistent connections |

**Example - File Operations:**

```bash
# Bash: Simple file read
cat myfile.txt

# MCP: Complex file workflow with validation
# Use filesystem MCP when you need:
# - Sandboxed access
# - File monitoring
# - Batch operations with progress
```

**Example - API Calls:**

```bash
# Bash: One-time API call
curl -H "Authorization: Bearer $TOKEN" https://api.example.com/data

# MCP: Repeated API operations
# Use custom MCP when you need:
# - Token refresh handling
# - Rate limit management
# - Response caching
```

### MCP vs Custom Scripts

| Use Custom Scripts When | Use MCP When |
|------------------------|--------------|
| Running once | Running frequently |
| Self-contained logic | Needs Claude interaction |
| No Claude integration | Dynamic Claude decisions |
| Simple output | Structured responses |

---

## 10. Comparison with Similar Features

### MCP vs Hooks

| Aspect | MCP | Hooks |
|--------|-----|-------|
| **Purpose** | Extend Claude's capabilities | Automate Claude Code events |
| **Trigger** | Claude decides to use | Events trigger automatically |
| **Direction** | Claude → External | External → Claude workflow |
| **Interaction** | Bidirectional | One-way (hook runs, reports) |
| **Configuration** | mcp.json | .claude/hooks/ |

**When to Use Each:**

```
MCP: "I need Claude to be able to query our database"
     → Add database MCP server

Hooks: "I want tests to run automatically before every commit"
       → Create pre-commit hook
```

### MCP vs Skills

| Aspect | MCP | Skills |
|--------|-----|--------|
| **Purpose** | External tool access | Guided workflows |
| **Content** | API/service integrations | Instructions and patterns |
| **Location** | mcp.json + npm packages | .claude/skills/ |
| **Invocation** | Claude uses tools | /skill-name command |

**When to Use Each:**

```
MCP: "I need Claude to interact with GitHub API"
     → Add GitHub MCP server

Skills: "I want Claude to follow our PR review process"
        → Create code-review skill with steps
```

### MCP vs Subagents

| Aspect | MCP | Subagents |
|--------|-----|-----------|
| **Purpose** | Tool access | Parallel task execution |
| **Execution** | Synchronous tool calls | Async agent spawning |
| **Scope** | Single operation | Complex multi-step tasks |
| **Cost** | Tool call | Full agent session |

**When to Use Each:**

```
MCP: "Send a Slack message"
     → Use slack MCP tool

Subagents: "Research these 5 topics simultaneously"
           → Spawn 5 research subagents
```

### Decision Matrix

| Need | Solution |
|------|----------|
| Access external APIs | **MCP** |
| Automate on events | **Hooks** |
| Guide Claude's process | **Skills** |
| Parallel execution | **Subagents** |
| Query databases | **MCP** |
| Pre-commit checks | **Hooks** |
| Code review workflow | **Skills** |
| Research multiple topics | **Subagents** |
| Send notifications | **MCP** |
| Format code on save | **Hooks** |
| Deployment process | **Skills** |
| Analyze multiple files | **Subagents** |

---

## 11. Security Best Practices

### Credential Management

**Never hardcode credentials:**

```json
// BAD - credentials in config
{
  "mcpServers": {
    "github": {
      "env": {
        "GITHUB_TOKEN": "ghp_actualTokenHere123"  // DON'T DO THIS
      }
    }
  }
}

// GOOD - reference environment variables
{
  "mcpServers": {
    "github": {
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

**Use a secrets manager:**

```bash
# Option 1: dotenv file (add to .gitignore!)
echo "GITHUB_TOKEN=ghp_xxx" >> ~/.claude/.env

# Option 2: System keychain
# Mac
security add-generic-password -s "github-mcp" -a "token" -w "ghp_xxx"

# Option 3: HashiCorp Vault or similar
vault kv put secret/mcp/github token="ghp_xxx"
```

**Rotate credentials regularly:**

```bash
# Create a reminder
echo "0 0 1 * * /path/to/rotate-mcp-credentials.sh" | crontab -
```

### Permission Scoping

**GitHub Token Scopes:**

```
Minimal scopes for read-only:
- public_repo (for public repos only)
- read:org

Standard scopes for development:
- repo (full repository access)
- read:org
- workflow (if using GitHub Actions)

Avoid these unless necessary:
- delete_repo
- admin:org
- admin:repo_hook
```

**Database Permissions:**

```sql
-- Create a read-only user for MCP
CREATE USER mcp_reader WITH PASSWORD 'secure_password';
GRANT SELECT ON ALL TABLES IN SCHEMA public TO mcp_reader;

-- Or limited write access
CREATE USER mcp_writer WITH PASSWORD 'secure_password';
GRANT SELECT, INSERT, UPDATE ON specific_table TO mcp_writer;
-- No DELETE, no DROP
```

**Filesystem Restrictions:**

```json
{
  "mcpServers": {
    "filesystem": {
      "args": [
        "/home/user/projects",     // Only project files
        "--exclude", ".env",        // No env files
        "--exclude", "*.pem",       // No certificates
        "--exclude", "credentials*" // No credential files
      ]
    }
  }
}
```

### Audit Logging

**Enable MCP logging:**

```json
{
  "mcpServers": {
    "github": {
      "env": {
        "MCP_LOG_LEVEL": "info",
        "MCP_LOG_FILE": "/var/log/mcp/github.log"
      }
    }
  }
}
```

**Log format recommendation:**

```json
{
  "timestamp": "2025-01-15T14:30:00Z",
  "server": "github",
  "tool": "create_pull_request",
  "user": "local",
  "parameters": {
    "repo": "myorg/myrepo",
    "title": "Feature: Add auth"
  },
  "result": "success",
  "duration_ms": 234
}
```

**Monitoring script:**

```bash
#!/bin/bash
# monitor-mcp.sh - Alert on suspicious MCP activity

tail -f /var/log/mcp/*.log | while read line; do
  # Alert on sensitive operations
  if echo "$line" | grep -qE "(delete|drop|admin)"; then
    echo "ALERT: Sensitive MCP operation detected"
    echo "$line" | mail -s "MCP Security Alert" admin@company.com
  fi
done
```

---

## 12. Troubleshooting

### Common Errors

**Error: "MCP server failed to start"**

```
Symptoms:
- Server shows as "failed" in /mcp status
- Claude can't use tools from that server

Causes & Solutions:

1. Command not found
   $ which mcp-server-github
   # If empty, install the package:
   $ npm install -g @anthropic/mcp-server-github

2. Missing environment variable
   $ echo $GITHUB_TOKEN
   # If empty, set it:
   $ export GITHUB_TOKEN="ghp_xxx"

3. Permission denied
   $ ls -la $(which mcp-server-github)
   # If not executable:
   $ chmod +x $(which mcp-server-github)
```

**Error: "Tool call timed out"**

```
Symptoms:
- Claude says the tool call timed out
- Long-running operations fail

Causes & Solutions:

1. Slow network
   - Check API endpoint connectivity
   - Consider increasing timeout in server config

2. Rate limiting
   - Check if API is rate limiting
   - Implement backoff in custom servers

3. Server hanging
   - Check server logs for errors
   - Restart the MCP server
```

**Error: "Invalid response from MCP server"**

```
Symptoms:
- Malformed response errors
- Unexpected tool results

Causes & Solutions:

1. Server version mismatch
   $ npm update -g @anthropic/mcp-server-xxx

2. Corrupted server state
   $ claude mcp restart server-name

3. Bug in custom server
   - Check server output manually
   - Validate JSON-RPC responses
```

### Debug Techniques

**Enable verbose logging:**

```bash
# Set debug environment variable
export DEBUG=mcp:*

# Or in mcp.json
{
  "mcpServers": {
    "myserver": {
      "env": {
        "DEBUG": "*",
        "MCP_LOG_LEVEL": "debug"
      }
    }
  }
}
```

**Test server independently:**

```bash
# Run server manually
mcp-server-github 2>&1 | tee server.log

# Send test request
echo '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}' | \
  mcp-server-github | jq .

# Check for valid JSON-RPC response
```

**Inspect Claude Code MCP state:**

```bash
# Inside Claude Code
/mcp status
/mcp list-tools
/mcp restart myserver
```

### MCP Server Logs

**Finding logs:**

```bash
# Default log locations
~/.claude/logs/mcp-*.log
/tmp/mcp-server-*.log

# Check server-specific logs
cat ~/.claude/logs/mcp-github.log | tail -100

# Follow logs in real-time
tail -f ~/.claude/logs/mcp-*.log
```

**Log analysis commands:**

```bash
# Find errors
grep -i error ~/.claude/logs/mcp-*.log

# Count tool calls by type
grep "tools/call" ~/.claude/logs/mcp-*.log | \
  jq -r '.params.name' | sort | uniq -c

# Find slow operations (>1s)
grep "duration" ~/.claude/logs/mcp-*.log | \
  jq 'select(.duration_ms > 1000)'
```

**Creating debug reports:**

```bash
#!/bin/bash
# mcp-debug-report.sh

echo "=== MCP Debug Report ==="
echo "Date: $(date)"
echo ""

echo "=== Installed Servers ==="
npm list -g | grep mcp-server
echo ""

echo "=== Configuration ==="
cat ~/.claude/mcp.json
echo ""

echo "=== Recent Logs ==="
tail -50 ~/.claude/logs/mcp-*.log
echo ""

echo "=== Environment Variables ==="
env | grep -E "(MCP|GITHUB|SLACK|DATABASE)" | sed 's/=.*/=<hidden>/'
```

---

## 13. Quick Reference

### mcp.json Template

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "mcp-server-filesystem",
      "args": ["/path/to/allowed/directory"],
      "env": {}
    },

    "github": {
      "command": "mcp-server-github",
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },

    "database": {
      "command": "mcp-server-postgres",
      "args": ["postgresql://user:${DB_PASS}@localhost:5432/dbname"],
      "env": {
        "DB_PASS": "${DB_PASS}"
      }
    },

    "slack": {
      "command": "mcp-server-slack",
      "env": {
        "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}",
        "SLACK_TEAM_ID": "${SLACK_TEAM_ID}"
      }
    },

    "browser": {
      "command": "mcp-server-puppeteer",
      "env": {
        "PUPPETEER_HEADLESS": "true"
      }
    },

    "memory": {
      "command": "mcp-server-memory",
      "env": {
        "MEMORY_FILE_PATH": "${HOME}/.claude/memory.json"
      }
    },

    "custom": {
      "command": "node",
      "args": ["/path/to/custom-server.js"],
      "env": {
        "CUSTOM_API_KEY": "${CUSTOM_API_KEY}"
      }
    }
  }
}
```

### Popular Servers List

| Server | Package | Purpose |
|--------|---------|---------|
| Filesystem | `@anthropic/mcp-server-filesystem` | File operations |
| GitHub | `@anthropic/mcp-server-github` | Repository management |
| PostgreSQL | `@anthropic/mcp-server-postgres` | Database queries |
| SQLite | `@anthropic/mcp-server-sqlite` | Local database |
| Slack | `@anthropic/mcp-server-slack` | Team messaging |
| Puppeteer | `@anthropic/mcp-server-puppeteer` | Browser automation |
| Memory | `@anthropic/mcp-server-memory` | Persistent storage |
| Fetch | `@anthropic/mcp-server-fetch` | HTTP requests |
| Sequential Thinking | `@anthropic/mcp-server-sequential-thinking` | Reasoning |
| Google Drive | `@anthropic/mcp-server-gdrive` | Drive access |
| Google Maps | `@anthropic/mcp-server-google-maps` | Location services |
| Brave Search | `@anthropic/mcp-server-brave-search` | Web search |
| Exa | `@anthropic/mcp-server-exa` | AI-powered search |
| Sentry | `@anthropic/mcp-server-sentry` | Error tracking |
| Raygun | `@anthropic/mcp-server-raygun` | Application monitoring |
| Linear | `@anthropic/mcp-server-linear` | Project management |
| Notion | `@anthropic/mcp-server-notion` | Documentation |

### Command Cheat Sheet

```bash
# Installation
npm install -g @anthropic/mcp-server-{name}

# Verify installation
which mcp-server-{name}
mcp-server-{name} --help

# Claude Code MCP commands
/mcp                    # List all MCP tools
/mcp status             # Check server status
/mcp restart {name}     # Restart a server

# Configuration
~/.claude/mcp.json      # User config location
./.claude/mcp.json      # Project config location

# Environment variables
export VAR_NAME="value" # Set variable
${VAR_NAME}             # Reference in mcp.json

# Debugging
DEBUG=* mcp-server-{name}   # Verbose output
cat ~/.claude/logs/mcp-*.log # View logs

# Creating custom servers
npm install @anthropic/mcp-sdk  # Install SDK
npx tsx src/index.ts            # Run TypeScript directly

# Testing
echo '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | mcp-server-{name}
```

### Environment Variables Reference

```bash
# GitHub MCP
GITHUB_TOKEN=ghp_xxxxxxxxxxxxx
GITHUB_API_URL=https://api.github.com  # Optional, for Enterprise

# Slack MCP
SLACK_BOT_TOKEN=xoxb-xxxxxxxxxxxxx
SLACK_TEAM_ID=T0XXXXXXX

# Database MCP
DATABASE_URL=postgresql://user:pass@host:5432/db
DB_PASSWORD=xxxxxx

# Browser MCP
PUPPETEER_HEADLESS=true
PUPPETEER_EXECUTABLE_PATH=/path/to/chrome

# Memory MCP
MEMORY_FILE_PATH=~/.claude/memory.json

# General MCP
MCP_LOG_LEVEL=debug|info|warn|error
MCP_LOG_FILE=/path/to/logfile.log
DEBUG=mcp:*
```

---

## Summary

MCP (Model Context Protocol) transforms Claude Code from a powerful coding assistant into an extensible platform that can interact with virtually any external system. By understanding the configuration format, installing appropriate servers, and following security best practices, you can dramatically expand Claude's capabilities.

**Key Takeaways:**

1. **MCP is modular** - Add only the servers you need
2. **Security matters** - Use environment variables, scope permissions carefully
3. **Start simple** - Begin with filesystem MCP, then expand
4. **Debug systematically** - Use logs and manual testing
5. **Know alternatives** - MCP is powerful but not always the right tool

**Next Steps:**

1. Set up your first MCP server (filesystem is recommended)
2. Add GitHub MCP if you work with repositories
3. Explore the community ecosystem for specialized servers
4. Consider building custom servers for your unique needs

MCP represents a paradigm shift in how AI assistants interact with the world. By mastering it, you unlock Claude Code's full potential as a development partner.

---

*This tutorial is part of the Claude Code Advanced Features series.*
