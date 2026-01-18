# Claude Code Hooks: A Comprehensive Tutorial

## Table of Contents

1. [Introduction](#introduction)
2. [Installation & Setup](#installation--setup)
3. [Types of Hooks](#types-of-hooks)
4. [Hand-Holding User Journey](#hand-holding-user-journey)
5. [Real-World Examples](#real-world-examples)
6. [Pros and Cons](#pros-and-cons)
7. [When to Use Hooks](#when-to-use-hooks)
8. [Comparison with Similar Features](#comparison-with-similar-features)
9. [Troubleshooting](#troubleshooting)
10. [Quick Reference](#quick-reference)

---

## Introduction

### What Are Hooks in Claude Code?

Hooks are custom scripts or commands that Claude Code executes automatically at specific points during its operation. Think of them as event listeners that trigger your code when Claude performs certain actions—like before running a bash command, after editing a file, or when a session starts.

Hooks allow you to extend Claude Code's functionality without modifying its core behavior. They're the bridge between Claude's AI capabilities and your custom automation workflows.

```
┌─────────────────────────────────────────────────────────────┐
│                     Claude Code Operation                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   User Request                                              │
│        │                                                    │
│        ▼                                                    │
│   ┌─────────────┐                                          │
│   │ PreToolUse  │ ◄── Your hook runs BEFORE the tool       │
│   │   Hook      │                                          │
│   └──────┬──────┘                                          │
│          │                                                  │
│          ▼                                                  │
│   ┌─────────────┐                                          │
│   │  Tool Runs  │     (Bash, Edit, Write, etc.)            │
│   └──────┬──────┘                                          │
│          │                                                  │
│          ▼                                                  │
│   ┌─────────────┐                                          │
│   │ PostToolUse │ ◄── Your hook runs AFTER the tool        │
│   │   Hook      │                                          │
│   └─────────────┘                                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Why Hooks Matter

**1. Automation**
- Automatically format code after Claude writes it
- Run tests after file modifications
- Sync changes to external systems
- Generate documentation on the fly

**2. Validation**
- Block dangerous commands before they execute
- Validate SQL queries against your schema
- Ensure code style compliance
- Check for security vulnerabilities

**3. Security**
- Prevent accidental file deletions
- Block access to sensitive directories
- Audit all tool invocations
- Enforce command allowlists

**4. Integration**
- Send Slack/Discord notifications
- Update project management tools
- Trigger CI/CD pipelines
- Log activities to external systems

### Prerequisites

Before using hooks, ensure you have:

- **Claude Code CLI**: Version 1.0.0 or later
  ```bash
  claude --version
  ```

- **Node.js** (optional, for JavaScript hooks): Version 18.0.0 or later
  ```bash
  node --version
  ```

- **Python** (optional, for Python hooks): Version 3.8 or later
  ```bash
  python --version
  ```

- **Basic command line knowledge**: Understanding of shell scripts, file paths, and JSON

---

## Installation & Setup

### Where Hooks Are Configured

Claude Code looks for hook configurations in settings files at multiple levels:

#### 1. User-Level Settings (Global)

**Location:**
- **Windows**: `%APPDATA%\Claude\settings.json`
  - Typically: `C:\Users\YourUsername\AppData\Roaming\Claude\settings.json`
- **macOS**: `~/.config/claude/settings.json`
  - Typically: `/Users/YourUsername/.config/claude/settings.json`
- **Linux**: `~/.config/claude/settings.json`
  - Typically: `/home/YourUsername/.config/claude/settings.json`

#### 2. Project-Level Settings (Local)

**Location**: `.claude/settings.json` in your project root

Example project structure:
```
my-project/
├── .claude/
│   └── settings.json    ◄── Project-specific hooks go here
├── src/
├── package.json
└── README.md
```

#### 3. Settings Precedence

Project-level settings override user-level settings for hooks with the same name. This allows you to have default hooks globally while customizing them per project.

```
Priority (highest to lowest):
1. .claude/settings.json (project)
2. ~/.config/claude/settings.json (user)
3. System defaults
```

### Hook Configuration File Structure

The `settings.json` file uses this structure for hooks:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "name": "my-pre-hook",
        "command": "/path/to/script.sh",
        "tools": ["Bash", "Write"],
        "enabled": true
      }
    ],
    "PostToolUse": [
      {
        "name": "my-post-hook",
        "command": "node /path/to/script.js",
        "tools": ["Edit"],
        "enabled": true
      }
    ],
    "Notification": [
      {
        "name": "notify-hook",
        "command": "python /path/to/notify.py",
        "enabled": true
      }
    ],
    "SessionStart": [
      {
        "name": "session-init",
        "command": "/path/to/init.sh",
        "enabled": true
      }
    ],
    "SessionStop": [
      {
        "name": "session-cleanup",
        "command": "/path/to/cleanup.sh",
        "enabled": true
      }
    ]
  }
}
```

### How to Enable/Disable Hooks

#### Method 1: Toggle the `enabled` Field

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "name": "format-code",
        "command": "prettier --write",
        "enabled": false  // Hook is disabled
      }
    ]
  }
}
```

#### Method 2: Remove the Hook Entry

Simply delete the hook object from the array to completely remove it.

#### Method 3: Use Claude Code CLI (if available)

```bash
# List all hooks
claude hooks list

# Disable a specific hook
claude hooks disable format-code

# Enable a specific hook
claude hooks enable format-code
```

#### Method 4: Environment Variable Override

```bash
# Disable all hooks for a session
CLAUDE_HOOKS_DISABLED=true claude

# Disable specific hook types
CLAUDE_DISABLE_PRE_HOOKS=true claude
```

---

## Types of Hooks

### PreToolUse Hooks

**When they run**: Before a tool (Bash, Edit, Write, Read, etc.) executes.

**Purpose**: Validate, modify, or block tool invocations before they happen.

**Input received** (via stdin as JSON):
```json
{
  "hook_type": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": {
    "command": "rm -rf /tmp/test"
  },
  "session_id": "abc123",
  "timestamp": "2025-01-15T10:30:00Z"
}
```

**Expected output** (via stdout as JSON):
```json
{
  "action": "allow",  // or "block" or "modify"
  "message": "Optional message to display",
  "modified_input": {
    // Only if action is "modify"
    "command": "rm -rf /tmp/test --interactive"
  }
}
```

**Action types**:
- `allow`: Let the tool proceed as-is
- `block`: Stop the tool from running (with optional message)
- `modify`: Change the tool's input before execution

**Configuration example**:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "name": "block-dangerous-commands",
        "command": "node C:/scripts/check-command.js",
        "tools": ["Bash"],
        "enabled": true
      }
    ]
  }
}
```

### PostToolUse Hooks

**When they run**: After a tool completes execution.

**Purpose**: React to tool results, trigger follow-up actions, or log outcomes.

**Input received** (via stdin as JSON):
```json
{
  "hook_type": "PostToolUse",
  "tool_name": "Edit",
  "tool_input": {
    "file_path": "/path/to/file.js",
    "old_string": "const x = 1",
    "new_string": "const x = 2"
  },
  "tool_output": {
    "success": true,
    "message": "File edited successfully"
  },
  "session_id": "abc123",
  "timestamp": "2025-01-15T10:30:05Z"
}
```

**Expected output** (optional, via stdout as JSON):
```json
{
  "action": "continue",  // or "notify"
  "message": "Optional message to display"
}
```

**Configuration example**:
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "name": "auto-format-on-edit",
        "command": "python C:/scripts/format-file.py",
        "tools": ["Edit", "Write"],
        "enabled": true
      }
    ]
  }
}
```

### Notification Hooks

**When they run**: When Claude Code wants to notify the user about something (task completion, errors, etc.).

**Purpose**: Route notifications to external systems (Slack, Discord, email, desktop notifications).

**Input received** (via stdin as JSON):
```json
{
  "hook_type": "Notification",
  "notification_type": "task_complete",
  "title": "Task Completed",
  "message": "Successfully refactored the authentication module",
  "session_id": "abc123",
  "timestamp": "2025-01-15T10:35:00Z"
}
```

**Configuration example**:
```json
{
  "hooks": {
    "Notification": [
      {
        "name": "slack-notify",
        "command": "bash C:/scripts/slack-notify.sh",
        "enabled": true
      }
    ]
  }
}
```

### Session Hooks

#### SessionStart Hook

**When it runs**: When a Claude Code session begins.

**Purpose**: Initialize environment, load context, set up resources.

**Input received** (via stdin as JSON):
```json
{
  "hook_type": "SessionStart",
  "session_id": "abc123",
  "working_directory": "C:/Users/simon/my-project",
  "timestamp": "2025-01-15T10:00:00Z"
}
```

**Configuration example**:
```json
{
  "hooks": {
    "SessionStart": [
      {
        "name": "load-env",
        "command": "bash C:/scripts/load-project-env.sh",
        "enabled": true
      }
    ]
  }
}
```

#### SessionStop Hook

**When it runs**: When a Claude Code session ends.

**Purpose**: Clean up resources, save state, generate reports.

**Input received** (via stdin as JSON):
```json
{
  "hook_type": "SessionStop",
  "session_id": "abc123",
  "working_directory": "C:/Users/simon/my-project",
  "duration_seconds": 1800,
  "timestamp": "2025-01-15T10:30:00Z"
}
```

**Configuration example**:
```json
{
  "hooks": {
    "SessionStop": [
      {
        "name": "generate-session-report",
        "command": "python C:/scripts/session-report.py",
        "enabled": true
      }
    ]
  }
}
```

### Hook Properties Reference

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | string | Yes | Unique identifier for the hook |
| `command` | string | Yes | Command or script to execute |
| `tools` | array | No | List of tools to trigger on (PreToolUse/PostToolUse only) |
| `enabled` | boolean | No | Whether the hook is active (default: true) |
| `timeout` | number | No | Maximum execution time in milliseconds (default: 30000) |
| `env` | object | No | Additional environment variables to pass |

---

## Hand-Holding User Journey

### Step 1: Create Your First Hook

Let's create a simple hook that logs every Bash command Claude tries to run.

**Goal**: Every time Claude runs a Bash command, log it to a file.

#### 1.1 Create the Hook Script

First, create a directory for your hooks:

**Windows (PowerShell)**:
```powershell
mkdir C:\claude-hooks
```

**macOS/Linux**:
```bash
mkdir -p ~/claude-hooks
```

Now create the logging script:

**Windows**: Create `C:\claude-hooks\log-commands.js`

```javascript
#!/usr/bin/env node

const fs = require('fs');
const path = require('path');

// Read input from stdin
let input = '';
process.stdin.setEncoding('utf8');

process.stdin.on('data', (chunk) => {
  input += chunk;
});

process.stdin.on('end', () => {
  try {
    const data = JSON.parse(input);

    // Create log entry
    const logEntry = {
      timestamp: new Date().toISOString(),
      tool: data.tool_name,
      command: data.tool_input?.command || 'N/A',
      session: data.session_id
    };

    // Append to log file
    const logPath = 'C:\\claude-hooks\\command-log.json';
    let logs = [];

    if (fs.existsSync(logPath)) {
      const existing = fs.readFileSync(logPath, 'utf8');
      logs = JSON.parse(existing || '[]');
    }

    logs.push(logEntry);
    fs.writeFileSync(logPath, JSON.stringify(logs, null, 2));

    // Allow the command to proceed
    console.log(JSON.stringify({ action: 'allow' }));

  } catch (error) {
    // On error, allow the command anyway
    console.log(JSON.stringify({ action: 'allow' }));
  }
});
```

**macOS/Linux**: Create `~/claude-hooks/log-commands.js`

```javascript
#!/usr/bin/env node

const fs = require('fs');
const path = require('path');
const os = require('os');

let input = '';
process.stdin.setEncoding('utf8');

process.stdin.on('data', (chunk) => {
  input += chunk;
});

process.stdin.on('end', () => {
  try {
    const data = JSON.parse(input);

    const logEntry = {
      timestamp: new Date().toISOString(),
      tool: data.tool_name,
      command: data.tool_input?.command || 'N/A',
      session: data.session_id
    };

    const logPath = path.join(os.homedir(), 'claude-hooks', 'command-log.json');
    let logs = [];

    if (fs.existsSync(logPath)) {
      const existing = fs.readFileSync(logPath, 'utf8');
      logs = JSON.parse(existing || '[]');
    }

    logs.push(logEntry);
    fs.writeFileSync(logPath, JSON.stringify(logs, null, 2));

    console.log(JSON.stringify({ action: 'allow' }));

  } catch (error) {
    console.log(JSON.stringify({ action: 'allow' }));
  }
});
```

Make it executable (macOS/Linux only):
```bash
chmod +x ~/claude-hooks/log-commands.js
```

#### 1.2 Configure the Hook

Create or edit your settings file:

**Windows**: Create/edit `%APPDATA%\Claude\settings.json`

To find the exact path:
```powershell
echo $env:APPDATA\Claude\settings.json
# Example output: C:\Users\simon\AppData\Roaming\Claude\settings.json
```

Add this content:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "name": "log-all-commands",
        "command": "node C:\\claude-hooks\\log-commands.js",
        "tools": ["Bash"],
        "enabled": true
      }
    ]
  }
}
```

**macOS/Linux**: Create/edit `~/.config/claude/settings.json`

```bash
mkdir -p ~/.config/claude
```

Add this content:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "name": "log-all-commands",
        "command": "node ~/claude-hooks/log-commands.js",
        "tools": ["Bash"],
        "enabled": true
      }
    ]
  }
}
```

#### 1.3 Verify the Configuration

Restart Claude Code and run a simple command. Then check the log:

**Windows**:
```powershell
type C:\claude-hooks\command-log.json
```

**macOS/Linux**:
```bash
cat ~/claude-hooks/command-log.json
```

You should see entries like:
```json
[
  {
    "timestamp": "2025-01-15T10:30:00.000Z",
    "tool": "Bash",
    "command": "ls -la",
    "session": "abc123"
  }
]
```

### Step 2: Test the Hook

#### 2.1 Manual Testing

Test your hook script independently:

**Windows**:
```powershell
echo '{"hook_type":"PreToolUse","tool_name":"Bash","tool_input":{"command":"echo test"},"session_id":"test123"}' | node C:\claude-hooks\log-commands.js
```

Expected output:
```json
{"action":"allow"}
```

**macOS/Linux**:
```bash
echo '{"hook_type":"PreToolUse","tool_name":"Bash","tool_input":{"command":"echo test"},"session_id":"test123"}' | node ~/claude-hooks/log-commands.js
```

#### 2.2 Integration Testing

1. Start a fresh Claude Code session
2. Ask Claude to run a simple command: "Run `echo hello world`"
3. Check your log file for the entry
4. Ask Claude to run another command and verify it's logged

#### 2.3 Test Edge Cases

Test what happens when:
- The hook script has a syntax error
- The hook times out
- The hook returns invalid JSON

### Step 3: Debug Hook Issues

#### 3.1 Enable Verbose Logging

Set environment variable before running Claude:

**Windows**:
```powershell
$env:CLAUDE_HOOK_DEBUG = "true"
claude
```

**macOS/Linux**:
```bash
CLAUDE_HOOK_DEBUG=true claude
```

#### 3.2 Check Hook Execution Logs

Look for hook-related messages in:

**Windows**: `%APPDATA%\Claude\logs\`
**macOS/Linux**: `~/.config/claude/logs/`

#### 3.3 Common Debug Techniques

**Add console.error for debugging** (stderr doesn't affect hook output):

```javascript
process.stdin.on('end', () => {
  console.error('DEBUG: Received input:', input);  // Goes to stderr, visible in logs

  // ... rest of your code

  console.log(JSON.stringify({ action: 'allow' }));  // Goes to stdout, parsed by Claude
});
```

**Test with mock input**:

Create a test file `test-input.json`:
```json
{
  "hook_type": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": {
    "command": "rm -rf /"
  },
  "session_id": "test"
}
```

Run:
```bash
cat test-input.json | node your-hook-script.js
```

#### 3.4 Validate JSON Output

Ensure your hook outputs valid JSON:

```javascript
// Good - outputs clean JSON
console.log(JSON.stringify({ action: 'allow' }));

// Bad - extra text breaks parsing
console.log('Processing...');
console.log(JSON.stringify({ action: 'allow' }));
```

### Step 4: Advanced Hook Patterns

#### 4.1 Chaining Multiple Hooks

You can have multiple hooks for the same event:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "name": "log-command",
        "command": "node C:\\hooks\\log.js",
        "tools": ["Bash"],
        "enabled": true
      },
      {
        "name": "validate-command",
        "command": "node C:\\hooks\\validate.js",
        "tools": ["Bash"],
        "enabled": true
      },
      {
        "name": "check-permissions",
        "command": "python C:\\hooks\\permissions.py",
        "tools": ["Bash", "Write", "Edit"],
        "enabled": true
      }
    ]
  }
}
```

Hooks execute in order. If any hook returns `"action": "block"`, subsequent hooks don't run.

#### 4.2 Conditional Hooks Based on Project

Use project-level settings to override global hooks:

**Global** (`~/.config/claude/settings.json`):
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "name": "strict-validation",
        "command": "node ~/hooks/strict.js",
        "enabled": true
      }
    ]
  }
}
```

**Project** (`./my-experimental-project/.claude/settings.json`):
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "name": "strict-validation",
        "command": "echo '{\"action\":\"allow\"}'",
        "enabled": true
      }
    ]
  }
}
```

The project hook overrides the global one, effectively disabling strict validation for that project.

#### 4.3 Environment-Aware Hooks

Pass environment info to your hooks:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "name": "env-aware-hook",
        "command": "node C:\\hooks\\smart-hook.js",
        "env": {
          "HOOK_MODE": "production",
          "LOG_LEVEL": "info",
          "PROJECT_TYPE": "nodejs"
        },
        "enabled": true
      }
    ]
  }
}
```

Access in your script:
```javascript
const mode = process.env.HOOK_MODE;  // "production"
const logLevel = process.env.LOG_LEVEL;  // "info"
```

---

## Real-World Examples

### Example 1: Auto-Format Code Before Commits

**Goal**: Automatically run Prettier on any file Claude edits, before the edit is committed.

**File**: `C:\claude-hooks\auto-format.js`

```javascript
#!/usr/bin/env node

const { execSync } = require('child_process');
const fs = require('fs');
const path = require('path');

let input = '';
process.stdin.setEncoding('utf8');

process.stdin.on('data', chunk => input += chunk);

process.stdin.on('end', () => {
  try {
    const data = JSON.parse(input);

    // Only process after Edit or Write operations
    if (data.hook_type !== 'PostToolUse') {
      console.log(JSON.stringify({ action: 'continue' }));
      return;
    }

    const filePath = data.tool_input?.file_path;

    if (!filePath || !fs.existsSync(filePath)) {
      console.log(JSON.stringify({ action: 'continue' }));
      return;
    }

    // Only format JavaScript, TypeScript, JSON, CSS files
    const formattableExtensions = ['.js', '.jsx', '.ts', '.tsx', '.json', '.css', '.scss', '.md'];
    const ext = path.extname(filePath).toLowerCase();

    if (!formattableExtensions.includes(ext)) {
      console.log(JSON.stringify({ action: 'continue' }));
      return;
    }

    // Run Prettier
    try {
      execSync(`npx prettier --write "${filePath}"`, {
        cwd: path.dirname(filePath),
        stdio: 'pipe'
      });

      console.error(`[auto-format] Formatted: ${filePath}`);
      console.log(JSON.stringify({
        action: 'continue',
        message: `Auto-formatted ${path.basename(filePath)} with Prettier`
      }));
    } catch (formatError) {
      console.error(`[auto-format] Prettier failed: ${formatError.message}`);
      console.log(JSON.stringify({ action: 'continue' }));
    }

  } catch (error) {
    console.error(`[auto-format] Error: ${error.message}`);
    console.log(JSON.stringify({ action: 'continue' }));
  }
});
```

**Configuration** (`.claude/settings.json`):

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "name": "auto-format-code",
        "command": "node C:\\claude-hooks\\auto-format.js",
        "tools": ["Edit", "Write"],
        "enabled": true,
        "timeout": 10000
      }
    ]
  }
}
```

---

### Example 2: Block Dangerous Commands

**Goal**: Prevent Claude from running destructive commands like `rm -rf`, `DROP TABLE`, etc.

**File**: `C:\claude-hooks\block-dangerous.js`

```javascript
#!/usr/bin/env node

let input = '';
process.stdin.setEncoding('utf8');

process.stdin.on('data', chunk => input += chunk);

process.stdin.on('end', () => {
  try {
    const data = JSON.parse(input);

    if (data.hook_type !== 'PreToolUse' || data.tool_name !== 'Bash') {
      console.log(JSON.stringify({ action: 'allow' }));
      return;
    }

    const command = (data.tool_input?.command || '').toLowerCase();

    // Define dangerous patterns
    const dangerousPatterns = [
      // File system destruction
      { pattern: /rm\s+(-[rf]+\s+)*[\/~]/, message: 'Blocking rm command on root or home directory' },
      { pattern: /rm\s+-rf?\s+\*/, message: 'Blocking recursive delete with wildcard' },
      { pattern: /rmdir\s+\//, message: 'Blocking rmdir on root paths' },
      { pattern: /del\s+\/s\s+\/q/, message: 'Blocking Windows recursive delete' },
      { pattern: /format\s+[a-z]:/, message: 'Blocking disk format command' },

      // Database destruction
      { pattern: /drop\s+(database|table|schema)/i, message: 'Blocking DROP statement' },
      { pattern: /truncate\s+table/i, message: 'Blocking TRUNCATE statement' },
      { pattern: /delete\s+from\s+\w+\s*;?\s*$/i, message: 'Blocking DELETE without WHERE clause' },

      // System commands
      { pattern: /:(){ :|:& };:/, message: 'Blocking fork bomb' },
      { pattern: /mkfs\./, message: 'Blocking filesystem format' },
      { pattern: /dd\s+if=.*of=\/dev\/[sh]d/, message: 'Blocking dd to disk device' },

      // Credential exposure
      { pattern: /curl.*\|\s*(ba)?sh/, message: 'Blocking piped curl to shell' },
      { pattern: /wget.*\|\s*(ba)?sh/, message: 'Blocking piped wget to shell' },

      // Git destruction
      { pattern: /git\s+push.*--force.*main/, message: 'Blocking force push to main' },
      { pattern: /git\s+push.*--force.*master/, message: 'Blocking force push to master' },
      { pattern: /git\s+reset\s+--hard\s+origin/, message: 'Blocking hard reset to origin' }
    ];

    // Check each pattern
    for (const { pattern, message } of dangerousPatterns) {
      if (pattern.test(command)) {
        console.error(`[BLOCKED] ${message}: ${command}`);
        console.log(JSON.stringify({
          action: 'block',
          message: `🛑 BLOCKED: ${message}\n\nCommand: ${command}\n\nIf you need to run this command, please do so manually.`
        }));
        return;
      }
    }

    // Allow safe commands
    console.log(JSON.stringify({ action: 'allow' }));

  } catch (error) {
    // On error, block for safety
    console.error(`[block-dangerous] Error: ${error.message}`);
    console.log(JSON.stringify({
      action: 'block',
      message: 'Hook error - blocking command for safety'
    }));
  }
});
```

**Configuration**:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "name": "block-dangerous-commands",
        "command": "node C:\\claude-hooks\\block-dangerous.js",
        "tools": ["Bash"],
        "enabled": true
      }
    ]
  }
}
```

---

### Example 3: Notify Slack on Task Completion

**Goal**: Send a Slack message when Claude completes a task or encounters an error.

**File**: `C:\claude-hooks\slack-notify.js`

```javascript
#!/usr/bin/env node

const https = require('https');

// Replace with your Slack webhook URL
const SLACK_WEBHOOK_URL = process.env.SLACK_WEBHOOK_URL || 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL';

let input = '';
process.stdin.setEncoding('utf8');

process.stdin.on('data', chunk => input += chunk);

function sendSlackMessage(text, color = '#36a64f') {
  return new Promise((resolve, reject) => {
    const url = new URL(SLACK_WEBHOOK_URL);

    const payload = JSON.stringify({
      attachments: [{
        color: color,
        text: text,
        footer: 'Claude Code Hook',
        ts: Math.floor(Date.now() / 1000)
      }]
    });

    const options = {
      hostname: url.hostname,
      path: url.pathname,
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Content-Length': Buffer.byteLength(payload)
      }
    };

    const req = https.request(options, (res) => {
      resolve(res.statusCode === 200);
    });

    req.on('error', reject);
    req.write(payload);
    req.end();
  });
}

process.stdin.on('end', async () => {
  try {
    const data = JSON.parse(input);

    if (data.hook_type !== 'Notification') {
      console.log(JSON.stringify({ action: 'continue' }));
      return;
    }

    const notificationType = data.notification_type || 'info';
    const title = data.title || 'Claude Code Notification';
    const message = data.message || 'No message provided';

    // Choose color based on notification type
    const colors = {
      task_complete: '#36a64f',  // Green
      error: '#ff0000',          // Red
      warning: '#ffcc00',        // Yellow
      info: '#0066cc'            // Blue
    };

    const color = colors[notificationType] || colors.info;
    const emoji = {
      task_complete: '✅',
      error: '❌',
      warning: '⚠️',
      info: 'ℹ️'
    }[notificationType] || 'ℹ️';

    const slackMessage = `${emoji} *${title}*\n${message}`;

    await sendSlackMessage(slackMessage, color);
    console.error(`[slack-notify] Sent notification: ${title}`);

    console.log(JSON.stringify({ action: 'continue' }));

  } catch (error) {
    console.error(`[slack-notify] Error: ${error.message}`);
    console.log(JSON.stringify({ action: 'continue' }));
  }
});
```

**Configuration**:

```json
{
  "hooks": {
    "Notification": [
      {
        "name": "slack-notify",
        "command": "node C:\\claude-hooks\\slack-notify.js",
        "env": {
          "SLACK_WEBHOOK_URL": "https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX"
        },
        "enabled": true
      }
    ]
  }
}
```

---

### Example 4: Validate SQL Queries Before Execution

**Goal**: Check SQL queries for common issues before Claude runs them.

**File**: `C:\claude-hooks\validate-sql.js`

```javascript
#!/usr/bin/env node

let input = '';
process.stdin.setEncoding('utf8');

process.stdin.on('data', chunk => input += chunk);

process.stdin.on('end', () => {
  try {
    const data = JSON.parse(input);

    if (data.hook_type !== 'PreToolUse' || data.tool_name !== 'Bash') {
      console.log(JSON.stringify({ action: 'allow' }));
      return;
    }

    const command = data.tool_input?.command || '';

    // Check if this is a SQL command (mysql, psql, sqlite3, etc.)
    const sqlTools = ['mysql', 'psql', 'sqlite3', 'sqlcmd', 'sql'];
    const isSqlCommand = sqlTools.some(tool => command.includes(tool));

    if (!isSqlCommand) {
      console.log(JSON.stringify({ action: 'allow' }));
      return;
    }

    const issues = [];
    const warnings = [];
    const upperCommand = command.toUpperCase();

    // Critical issues (will block)
    if (/DROP\s+(DATABASE|TABLE|SCHEMA)/i.test(command)) {
      issues.push('DROP statement detected - this will permanently delete data');
    }

    if (/TRUNCATE\s+TABLE/i.test(command)) {
      issues.push('TRUNCATE statement detected - this will delete all rows');
    }

    if (/DELETE\s+FROM\s+\w+\s*(;|\s*$)/i.test(command) && !/WHERE/i.test(command)) {
      issues.push('DELETE without WHERE clause - this will delete ALL rows');
    }

    if (/UPDATE\s+\w+\s+SET/i.test(command) && !/WHERE/i.test(command)) {
      issues.push('UPDATE without WHERE clause - this will update ALL rows');
    }

    // Warnings (will allow but notify)
    if (/SELECT\s+\*/i.test(command) && !/LIMIT/i.test(command)) {
      warnings.push('SELECT * without LIMIT may return too many rows');
    }

    if (/--.*password|--.*pwd|--.*pass=/i.test(command)) {
      warnings.push('Password appears to be in command line - consider using environment variables');
    }

    if (/ALTER\s+TABLE/i.test(command)) {
      warnings.push('ALTER TABLE detected - ensure you have a backup');
    }

    // Block if there are critical issues
    if (issues.length > 0) {
      console.log(JSON.stringify({
        action: 'block',
        message: `🛑 SQL VALIDATION FAILED\n\nIssues found:\n${issues.map(i => `• ${i}`).join('\n')}\n\nCommand: ${command}\n\nPlease modify the query or run it manually after review.`
      }));
      return;
    }

    // Allow with warnings
    if (warnings.length > 0) {
      console.error(`[sql-validate] Warnings: ${warnings.join('; ')}`);
    }

    console.log(JSON.stringify({ action: 'allow' }));

  } catch (error) {
    console.error(`[sql-validate] Error: ${error.message}`);
    console.log(JSON.stringify({ action: 'allow' }));
  }
});
```

**Configuration**:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "name": "validate-sql",
        "command": "node C:\\claude-hooks\\validate-sql.js",
        "tools": ["Bash"],
        "enabled": true
      }
    ]
  }
}
```

---

### Example 5: Auto-Run Tests After File Changes

**Goal**: Automatically run relevant tests when Claude modifies source files.

**File**: `C:\claude-hooks\auto-test.js`

```javascript
#!/usr/bin/env node

const { execSync } = require('child_process');
const path = require('path');
const fs = require('fs');

let input = '';
process.stdin.setEncoding('utf8');

process.stdin.on('data', chunk => input += chunk);

process.stdin.on('end', () => {
  try {
    const data = JSON.parse(input);

    // Only run after Edit or Write operations
    if (data.hook_type !== 'PostToolUse') {
      console.log(JSON.stringify({ action: 'continue' }));
      return;
    }

    const filePath = data.tool_input?.file_path;

    if (!filePath) {
      console.log(JSON.stringify({ action: 'continue' }));
      return;
    }

    const ext = path.extname(filePath);
    const dir = path.dirname(filePath);
    const basename = path.basename(filePath, ext);

    // Skip if the file itself is a test file
    if (basename.includes('.test') || basename.includes('.spec') || basename.includes('_test')) {
      console.log(JSON.stringify({ action: 'continue' }));
      return;
    }

    // Only process source files
    const sourceExtensions = ['.js', '.jsx', '.ts', '.tsx', '.py', '.go', '.rs'];
    if (!sourceExtensions.includes(ext)) {
      console.log(JSON.stringify({ action: 'continue' }));
      return;
    }

    // Find corresponding test file
    const possibleTestFiles = [
      path.join(dir, `${basename}.test${ext}`),
      path.join(dir, `${basename}.spec${ext}`),
      path.join(dir, '__tests__', `${basename}.test${ext}`),
      path.join(dir, '__tests__', `${basename}.spec${ext}`),
      path.join(dir, 'tests', `${basename}_test${ext}`),
      path.join(dir, '..', '__tests__', `${basename}.test${ext}`)
    ];

    const testFile = possibleTestFiles.find(f => fs.existsSync(f));

    if (!testFile) {
      console.error(`[auto-test] No test file found for: ${filePath}`);
      console.log(JSON.stringify({ action: 'continue' }));
      return;
    }

    // Determine test runner based on project
    let testCommand;
    const projectRoot = findProjectRoot(dir);

    if (fs.existsSync(path.join(projectRoot, 'package.json'))) {
      // Node.js project
      const packageJson = JSON.parse(fs.readFileSync(path.join(projectRoot, 'package.json'), 'utf8'));

      if (packageJson.devDependencies?.jest || packageJson.dependencies?.jest) {
        testCommand = `npx jest "${testFile}" --passWithNoTests`;
      } else if (packageJson.devDependencies?.vitest || packageJson.dependencies?.vitest) {
        testCommand = `npx vitest run "${testFile}"`;
      } else if (packageJson.devDependencies?.mocha || packageJson.dependencies?.mocha) {
        testCommand = `npx mocha "${testFile}"`;
      } else {
        testCommand = `npm test -- --testPathPattern="${testFile}"`;
      }
    } else if (fs.existsSync(path.join(projectRoot, 'pytest.ini')) ||
               fs.existsSync(path.join(projectRoot, 'setup.py'))) {
      // Python project
      testCommand = `python -m pytest "${testFile}" -v`;
    } else if (fs.existsSync(path.join(projectRoot, 'go.mod'))) {
      // Go project
      testCommand = `go test -v -run "${basename}"`;
    } else {
      console.error(`[auto-test] Unknown project type for: ${filePath}`);
      console.log(JSON.stringify({ action: 'continue' }));
      return;
    }

    // Run the tests
    console.error(`[auto-test] Running: ${testCommand}`);

    try {
      const output = execSync(testCommand, {
        cwd: projectRoot,
        encoding: 'utf8',
        timeout: 60000,
        stdio: 'pipe'
      });

      console.error(`[auto-test] Tests passed for: ${testFile}`);
      console.log(JSON.stringify({
        action: 'continue',
        message: `✅ Tests passed: ${path.basename(testFile)}`
      }));

    } catch (testError) {
      console.error(`[auto-test] Tests failed: ${testError.message}`);
      console.log(JSON.stringify({
        action: 'continue',
        message: `❌ Tests failed for ${path.basename(testFile)}. Check output for details.`
      }));
    }

  } catch (error) {
    console.error(`[auto-test] Error: ${error.message}`);
    console.log(JSON.stringify({ action: 'continue' }));
  }
});

function findProjectRoot(startDir) {
  let current = startDir;

  while (current !== path.parse(current).root) {
    if (fs.existsSync(path.join(current, 'package.json')) ||
        fs.existsSync(path.join(current, 'go.mod')) ||
        fs.existsSync(path.join(current, 'Cargo.toml')) ||
        fs.existsSync(path.join(current, 'setup.py')) ||
        fs.existsSync(path.join(current, '.git'))) {
      return current;
    }
    current = path.dirname(current);
  }

  return startDir;
}
```

**Configuration**:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "name": "auto-run-tests",
        "command": "node C:\\claude-hooks\\auto-test.js",
        "tools": ["Edit", "Write"],
        "enabled": true,
        "timeout": 120000
      }
    ]
  }
}
```

---

## Pros and Cons

### Benefits of Using Hooks

| Benefit | Description |
|---------|-------------|
| **Automation** | Eliminate repetitive manual tasks like formatting, testing, linting |
| **Safety** | Block dangerous operations before they cause damage |
| **Consistency** | Ensure all Claude operations follow your standards |
| **Auditability** | Log all operations for security and debugging |
| **Integration** | Connect Claude to your existing tools and workflows |
| **Customization** | Tailor Claude's behavior to your specific needs |
| **Real-time** | React immediately to operations as they happen |

### Limitations and Gotchas

| Limitation | Mitigation |
|------------|------------|
| **Performance overhead** | Keep hooks fast; use timeouts; cache results |
| **Complexity** | Start simple; test thoroughly; document well |
| **Debugging difficulty** | Use console.error for logging; test hooks independently |
| **Silent failures** | Always output valid JSON; use try-catch everywhere |
| **Platform differences** | Use Node.js/Python for cross-platform compatibility |
| **State management** | Hooks are stateless by design; use files/databases for state |
| **Order dependency** | Hooks run in defined order; plan dependencies carefully |

### Performance Considerations

```
Hook Performance Guidelines
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Target execution time:
  • PreToolUse hooks: < 100ms (blocks tool execution)
  • PostToolUse hooks: < 500ms (runs after completion)
  • Notification hooks: < 1000ms (async, less critical)

Optimization techniques:
  1. Avoid network calls in PreToolUse when possible
  2. Use asynchronous operations where supported
  3. Cache expensive computations
  4. Skip processing early for irrelevant tools
  5. Set appropriate timeouts

Memory usage:
  • Hook processes start fresh each time
  • Don't load large files unless necessary
  • Stream large inputs instead of buffering

Timeout settings:
  • Default: 30000ms (30 seconds)
  • Recommended for PreToolUse: 5000-10000ms
  • Maximum: 300000ms (5 minutes)
```

---

## When to Use Hooks

### Ideal Scenarios

**Use hooks when you need to:**

1. **Validate before execution**
   - Block SQL DELETE without WHERE
   - Prevent commits with secrets
   - Validate file paths

2. **Transform operations**
   - Add required flags to commands
   - Modify file paths
   - Inject environment variables

3. **React after completion**
   - Format code after edits
   - Run tests after changes
   - Update documentation

4. **Integrate with external systems**
   - Send notifications
   - Update dashboards
   - Trigger pipelines

5. **Enforce policies**
   - Coding standards
   - Security rules
   - Workflow requirements

### When NOT to Use Hooks

**Avoid hooks when:**

| Scenario | Better Alternative |
|----------|-------------------|
| Complex multi-step logic | Use Skills instead |
| Providing context to Claude | Use MCP servers or custom instructions |
| Adding new capabilities | Use MCP tools |
| One-time operations | Just ask Claude directly |
| Heavy computation | Run separately, reference results |

### Decision Flowchart

```
Should you use a hook?
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

START
  │
  ▼
Do you need to intercept or react to Claude's tool usage?
  │
  ├─ NO ──▶ Don't use a hook
  │         Consider: Skills, MCP, Custom Instructions
  │
  ▼ YES
  │
Is the operation simple and fast (< 1 second)?
  │
  ├─ NO ──▶ Consider async processing
  │         or timeout extensions
  │
  ▼ YES
  │
Does it need to run EVERY time for a specific tool?
  │
  ├─ NO ──▶ Consider conditional logic
  │         or project-specific settings
  │
  ▼ YES
  │
┌─────────────────────────────────────────────┐
│             USE A HOOK                       │
│                                             │
│  PreToolUse: validation, blocking, logging  │
│  PostToolUse: formatting, testing, syncing  │
│  Notification: alerts, integrations         │
│  Session: setup, cleanup                    │
└─────────────────────────────────────────────┘
```

---

## Comparison with Similar Features

### Hooks vs Skills

| Aspect | Hooks | Skills |
|--------|-------|--------|
| **Purpose** | Intercept/react to tool operations | Provide new capabilities to Claude |
| **Trigger** | Automatic on events | Manual invocation (/skill-name) |
| **Complexity** | Simple input/output | Can be multi-step, conversational |
| **State** | Stateless | Can maintain conversation context |
| **Scope** | Per-operation | Per-task or workflow |
| **Use case** | Validation, logging, formatting | Commit workflows, PR reviews, debugging |

**Example comparison:**

```
Hook: Block 'rm -rf' commands automatically
Skill: /commit - Interactive commit workflow with message generation
```

### Hooks vs MCP Servers

| Aspect | Hooks | MCP Servers |
|--------|-------|-------------|
| **Purpose** | React to Claude's actions | Provide new tools to Claude |
| **Direction** | Observes Claude | Extends Claude |
| **Persistence** | Per-operation | Long-running process |
| **Protocol** | stdin/stdout JSON | MCP protocol over stdio |
| **Capabilities** | Limited to reactions | Full tool/resource provision |
| **Setup** | Settings.json | MCP configuration |

**Example comparison:**

```
Hook: Log every database query Claude runs
MCP Server: Provide a "query_database" tool for Claude to use
```

### Feature Selection Guide

```
What do you want to achieve?
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────────────────────────────────┐
│ "I want to validate/block/log  │
│  Claude's existing operations" │
└───────────────┬────────────────┘
                │
                ▼
           USE HOOKS

┌─────────────────────────────────┐
│ "I want to add new tools or    │
│  data sources for Claude"      │
└───────────────┬────────────────┘
                │
                ▼
        USE MCP SERVERS

┌─────────────────────────────────┐
│ "I want to teach Claude new    │
│  workflows or patterns"        │
└───────────────┬────────────────┘
                │
                ▼
           USE SKILLS

┌─────────────────────────────────┐
│ "I want Claude to remember     │
│  project-specific context"     │
└───────────────┬────────────────┘
                │
                ▼
   USE CLAUDE.md / CUSTOM INSTRUCTIONS
```

---

## Troubleshooting

### Common Errors and Solutions

#### Error 1: Hook Not Triggering

**Symptoms**: Hook never runs, no logs, no effect

**Solutions**:

```bash
# 1. Verify settings file location
# Windows
dir %APPDATA%\Claude\settings.json

# macOS/Linux
ls -la ~/.config/claude/settings.json

# 2. Validate JSON syntax
# Use a JSON validator or:
node -e "console.log(JSON.parse(require('fs').readFileSync('settings.json')))"

# 3. Check 'enabled' is true
# Look for: "enabled": true (not false, not missing)

# 4. Verify tool name matches exactly
# Must be: "Bash", "Edit", "Write", "Read" (case-sensitive)

# 5. Check script path exists
# Windows
dir C:\claude-hooks\your-script.js

# macOS/Linux
ls -la ~/claude-hooks/your-script.js
```

#### Error 2: Hook Runs But Doesn't Block

**Symptoms**: Hook executes but dangerous command still runs

**Solutions**:

```javascript
// 1. Ensure you output to stdout, not stderr
console.log(JSON.stringify({ action: 'block' }));  // ✓ Correct
console.error(JSON.stringify({ action: 'block' })); // ✗ Wrong

// 2. Output must be valid JSON
console.log('{"action":"block"}');  // ✓ Works
console.log('{action:"block"}');    // ✗ Invalid JSON

// 3. No extra output before/after JSON
console.log('Processing...');                       // ✗ Don't do this
console.log(JSON.stringify({ action: 'block' }));
console.log('Done');                                // ✗ Don't do this
```

#### Error 3: Hook Times Out

**Symptoms**: Operations are slow, timeout errors in logs

**Solutions**:

```json
// 1. Increase timeout in configuration
{
  "hooks": {
    "PreToolUse": [
      {
        "name": "slow-hook",
        "command": "node slow-script.js",
        "timeout": 60000  // 60 seconds instead of default 30
      }
    ]
  }
}
```

```javascript
// 2. Optimize your hook
// Avoid synchronous network calls
// Use caching
// Exit early when possible

process.stdin.on('end', () => {
  const data = JSON.parse(input);

  // Exit early if not relevant
  if (data.tool_name !== 'Bash') {
    console.log(JSON.stringify({ action: 'allow' }));
    return;
  }

  // ... rest of logic
});
```

#### Error 4: JSON Parse Errors

**Symptoms**: `SyntaxError: Unexpected token` in hook

**Solutions**:

```javascript
// 1. Always wrap in try-catch
try {
  const data = JSON.parse(input);
  // ... process
} catch (error) {
  console.error(`Parse error: ${error.message}`);
  console.log(JSON.stringify({ action: 'allow' }));
}

// 2. Handle empty input
if (!input || input.trim() === '') {
  console.log(JSON.stringify({ action: 'allow' }));
  return;
}
```

#### Error 5: Permission Denied

**Symptoms**: `EACCES` or permission errors

**Solutions**:

```bash
# macOS/Linux: Make script executable
chmod +x ~/claude-hooks/your-script.js

# Check script shebang
# First line should be:
#!/usr/bin/env node
# or
#!/usr/bin/env python3

# Windows: Check execution policy
powershell Get-ExecutionPolicy
powershell Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

### Debug Techniques

#### Technique 1: Add Debug Logging

```javascript
#!/usr/bin/env node

const fs = require('fs');

let input = '';
process.stdin.on('data', chunk => input += chunk);

process.stdin.on('end', () => {
  // Log to a debug file
  const debugLog = (msg) => {
    fs.appendFileSync('C:\\claude-hooks\\debug.log',
      `[${new Date().toISOString()}] ${msg}\n`
    );
  };

  debugLog(`Received input: ${input}`);

  try {
    const data = JSON.parse(input);
    debugLog(`Parsed data: ${JSON.stringify(data)}`);

    // Your logic here
    const result = { action: 'allow' };

    debugLog(`Returning: ${JSON.stringify(result)}`);
    console.log(JSON.stringify(result));

  } catch (error) {
    debugLog(`ERROR: ${error.message}`);
    console.log(JSON.stringify({ action: 'allow' }));
  }
});
```

#### Technique 2: Test Independently

```bash
# Create test input
echo '{"hook_type":"PreToolUse","tool_name":"Bash","tool_input":{"command":"rm -rf /"}}' > test-input.json

# Run your hook with test input
cat test-input.json | node C:\claude-hooks\block-dangerous.js

# Check output
# Expected: {"action":"block","message":"..."}
```

#### Technique 3: Validate Configuration

```javascript
// validate-config.js - Run this to check your setup
const fs = require('fs');
const path = require('path');
const os = require('os');

const configPaths = [
  path.join(os.homedir(), '.config', 'claude', 'settings.json'),
  path.join(process.env.APPDATA || '', 'Claude', 'settings.json'),
  '.claude/settings.json'
];

console.log('Checking Claude Code hook configuration...\n');

for (const configPath of configPaths) {
  if (fs.existsSync(configPath)) {
    console.log(`✓ Found: ${configPath}`);

    try {
      const config = JSON.parse(fs.readFileSync(configPath, 'utf8'));

      if (config.hooks) {
        for (const [hookType, hooks] of Object.entries(config.hooks)) {
          console.log(`  ${hookType}:`);
          for (const hook of hooks) {
            const status = hook.enabled !== false ? '✓' : '✗';
            console.log(`    ${status} ${hook.name}: ${hook.command}`);

            // Check if command exists
            const cmd = hook.command.split(' ')[1] || hook.command.split(' ')[0];
            if (!fs.existsSync(cmd.replace(/"/g, ''))) {
              console.log(`      ⚠ Warning: Script may not exist at ${cmd}`);
            }
          }
        }
      } else {
        console.log('  No hooks configured');
      }
    } catch (error) {
      console.log(`  ✗ Invalid JSON: ${error.message}`);
    }
  }
}
```

---

## Quick Reference

### Cheat Sheet of Hook Types

```
┌────────────────────────────────────────────────────────────────────────┐
│                        HOOK TYPES CHEAT SHEET                          │
├────────────────────────────────────────────────────────────────────────┤
│                                                                        │
│  PreToolUse                                                            │
│  ───────────                                                           │
│  • Runs: BEFORE a tool executes                                        │
│  • Can: allow, block, or modify                                        │
│  • Use for: validation, security, logging                              │
│  • Tools: Bash, Edit, Write, Read, Glob, Grep                         │
│                                                                        │
│  PostToolUse                                                           │
│  ────────────                                                          │
│  • Runs: AFTER a tool completes                                        │
│  • Can: continue, notify                                               │
│  • Use for: formatting, testing, syncing                               │
│  • Receives: tool output in addition to input                          │
│                                                                        │
│  Notification                                                          │
│  ────────────                                                          │
│  • Runs: When Claude sends a notification                              │
│  • Types: task_complete, error, warning, info                          │
│  • Use for: Slack, email, desktop notifications                        │
│                                                                        │
│  SessionStart                                                          │
│  ─────────────                                                         │
│  • Runs: When Claude Code session begins                               │
│  • Use for: environment setup, context loading                         │
│                                                                        │
│  SessionStop                                                           │
│  ────────────                                                          │
│  • Runs: When Claude Code session ends                                 │
│  • Use for: cleanup, reporting, state saving                           │
│                                                                        │
└────────────────────────────────────────────────────────────────────────┘
```

### Configuration Template

Copy and customize this template for your `settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "name": "security-validator",
        "command": "node C:\\claude-hooks\\security.js",
        "tools": ["Bash"],
        "enabled": true,
        "timeout": 5000
      },
      {
        "name": "command-logger",
        "command": "node C:\\claude-hooks\\log-commands.js",
        "tools": ["Bash", "Edit", "Write"],
        "enabled": true
      }
    ],
    "PostToolUse": [
      {
        "name": "auto-formatter",
        "command": "node C:\\claude-hooks\\format.js",
        "tools": ["Edit", "Write"],
        "enabled": true
      },
      {
        "name": "test-runner",
        "command": "node C:\\claude-hooks\\run-tests.js",
        "tools": ["Edit"],
        "enabled": false,
        "timeout": 120000
      }
    ],
    "Notification": [
      {
        "name": "slack-alerts",
        "command": "node C:\\claude-hooks\\slack.js",
        "enabled": true,
        "env": {
          "SLACK_WEBHOOK": "https://hooks.slack.com/..."
        }
      }
    ],
    "SessionStart": [
      {
        "name": "session-init",
        "command": "node C:\\claude-hooks\\init.js",
        "enabled": true
      }
    ],
    "SessionStop": [
      {
        "name": "session-report",
        "command": "node C:\\claude-hooks\\report.js",
        "enabled": true
      }
    ]
  }
}
```

### Minimal Hook Script Template

```javascript
#!/usr/bin/env node
// Minimal hook template - customize as needed

let input = '';
process.stdin.setEncoding('utf8');
process.stdin.on('data', chunk => input += chunk);

process.stdin.on('end', () => {
  try {
    const data = JSON.parse(input);

    // Your logic here
    // Access: data.hook_type, data.tool_name, data.tool_input, etc.

    // Return appropriate action
    console.log(JSON.stringify({
      action: 'allow'  // or 'block', 'modify', 'continue'
      // message: 'Optional message'
      // modified_input: {} // Only for 'modify' action
    }));

  } catch (error) {
    console.error(`Hook error: ${error.message}`);
    console.log(JSON.stringify({ action: 'allow' }));
  }
});
```

---

## Summary

Claude Code Hooks provide a powerful way to customize and extend Claude's behavior:

- **PreToolUse** hooks validate and potentially block operations before they run
- **PostToolUse** hooks react to completed operations for formatting, testing, or syncing
- **Notification** hooks integrate with external alerting systems
- **Session** hooks handle setup and cleanup at session boundaries

Start with simple logging hooks, test thoroughly, and gradually add more sophisticated automation. Remember that hooks should be fast, reliable, and fail gracefully.

For more information, consult the Claude Code documentation or explore the examples in this tutorial.

---

*This tutorial was generated for Claude Code users. Last updated: January 2025.*
