# Tutorial 06: Claude Code Terminal Integration and Bash Commands

## Table of Contents

1. [Introduction](#1-introduction)
2. [Installation & Terminal Setup](#2-installation--terminal-setup)
3. [Basic Command Execution](#3-basic-command-execution)
4. [Permission System](#4-permission-system)
5. [Hand-Holding User Journey](#5-hand-holding-user-journey)
6. [Real-World Examples](#6-real-world-examples)
7. [Background Execution](#7-background-execution)
8. [Safety Features](#8-safety-features)
9. [Platform-Specific Guidance](#9-platform-specific-guidance)
10. [Pros and Cons](#10-pros-and-cons)
11. [When to Use Bash vs Other Tools](#11-when-to-use-bash-vs-other-tools)
12. [Common Patterns](#12-common-patterns)
13. [Troubleshooting](#13-troubleshooting)
14. [Quick Reference](#14-quick-reference)

---

## 1. Introduction

### Claude Code as a Terminal-Native Tool

Claude Code is designed from the ground up as a terminal-native AI assistant. Unlike browser-based AI tools, Claude Code lives in your terminal, making it a natural companion for developers who spend their day at the command line. This terminal-first approach means Claude can directly interact with your development environment, execute commands, and integrate seamlessly into your existing workflows.

```
┌─────────────────────────────────────────────────────────────┐
│                    Your Terminal                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  $ claude                                            │    │
│  │  > Help me set up a new Node.js project             │    │
│  │                                                      │    │
│  │  Claude: I'll create a new Node.js project for you. │    │
│  │  [Executing: mkdir my-project && cd my-project]     │    │
│  │  [Executing: npm init -y]                           │    │
│  │  [Executing: npm install express]                   │    │
│  │                                                      │    │
│  │  Done! Your project is ready with Express installed.│    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### How Claude Executes Commands

When you ask Claude to perform tasks that require shell commands, it uses an internal Bash tool to execute them. Here's the flow:

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Your        │     │  Claude      │     │  Bash        │     │  Your        │
│  Request     │────>│  Analyzes    │────>│  Tool        │────>│  System      │
│              │     │  Intent      │     │  Executes    │     │              │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
                                                │
                                                v
                     ┌──────────────────────────────────────┐
                     │  Output captured and returned        │
                     │  to Claude for interpretation        │
                     └──────────────────────────────────────┘
```

**Key characteristics:**

1. **Persistent shell session**: Commands run in a persistent shell, maintaining environment variables and working directory across commands
2. **Output capture**: Both stdout and stderr are captured and returned
3. **Timeout handling**: Commands have configurable timeouts (default 2 minutes, max 10 minutes)
4. **Error awareness**: Claude sees exit codes and error messages

### Safety and Sandboxing

Claude Code implements multiple layers of safety to protect your system:

```
┌─────────────────────────────────────────────────────────────┐
│                    Safety Layers                             │
├─────────────────────────────────────────────────────────────┤
│  Layer 1: Permission System                                  │
│  ├── Ask mode (default): Prompts for dangerous commands     │
│  ├── Auto mode: Allows pre-approved commands                │
│  └── Deny mode: Blocks specified commands                   │
├─────────────────────────────────────────────────────────────┤
│  Layer 2: Dangerous Command Detection                        │
│  ├── rm -rf detection                                       │
│  ├── Sudo command awareness                                 │
│  ├── System modification detection                          │
│  └── Network-sensitive operations                           │
├─────────────────────────────────────────────────────────────┤
│  Layer 3: Sandboxing (when enabled)                         │
│  ├── File system isolation                                  │
│  ├── Network restrictions                                   │
│  └── Process limitations                                    │
└─────────────────────────────────────────────────────────────┘
```

### Permission Model

The permission model follows a principle of least privilege:

```
┌────────────────────────────────────────────────────────────────┐
│                    Permission Flow                              │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Command Request                                               │
│         │                                                       │
│         v                                                       │
│   ┌─────────────┐                                              │
│   │ Is command  │──Yes──> Execute immediately                  │
│   │ allowlisted?│                                              │
│   └─────────────┘                                              │
│         │ No                                                    │
│         v                                                       │
│   ┌─────────────┐                                              │
│   │ Is command  │──Yes──> Block with explanation               │
│   │ blocklisted?│                                              │
│   └─────────────┘                                              │
│         │ No                                                    │
│         v                                                       │
│   ┌─────────────┐                                              │
│   │ Is command  │──Yes──> Prompt user for approval             │
│   │ dangerous?  │                                              │
│   └─────────────┘                                              │
│         │ No                                                    │
│         v                                                       │
│   Execute based on permission mode                              │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

---

## 2. Installation & Terminal Setup

### Installing Claude Code CLI

**Using npm (recommended):**

```bash
# Install globally
npm install -g @anthropic-ai/claude-code

# Verify installation
claude --version
```

**Using Homebrew (macOS):**

```bash
# Add tap and install
brew tap anthropic-ai/claude-code
brew install claude-code

# Verify
claude --version
```

**Using curl (Linux/macOS):**

```bash
# Download and install
curl -fsSL https://claude.ai/install.sh | sh

# Add to PATH if needed
export PATH="$HOME/.claude/bin:$PATH"
```

### Terminal Requirements

| Terminal | Support Level | Notes |
|----------|--------------|-------|
| **macOS Terminal.app** | Full | Native support |
| **iTerm2** | Full | Recommended for macOS |
| **Windows Terminal** | Full | Best Windows experience |
| **PowerShell 7+** | Full | Cross-platform |
| **Git Bash** | Full | Good Windows alternative |
| **WSL2** | Full | Linux experience on Windows |
| **VS Code Terminal** | Full | Integrated development |
| **Alacritty** | Full | GPU-accelerated |
| **Kitty** | Full | Feature-rich |
| **Hyper** | Partial | Some rendering issues |

**Minimum requirements:**
- Terminal with ANSI color support
- UTF-8 encoding
- 80+ column width recommended
- Node.js 18+ (for npm installation)

### Shell Configuration

#### Bash Configuration

Add to `~/.bashrc` or `~/.bash_profile`:

```bash
# Claude Code configuration
export ANTHROPIC_API_KEY="your-api-key-here"

# Optional: Add claude to PATH if not in standard location
export PATH="$HOME/.claude/bin:$PATH"

# Optional: Create aliases for common operations
alias cc="claude"
alias cchat="claude chat"
alias ccode="claude code"

# Optional: Enable command history integration
export CLAUDE_HISTORY_FILE="$HOME/.claude_history"
```

Reload configuration:

```bash
source ~/.bashrc
```

#### Zsh Configuration

Add to `~/.zshrc`:

```zsh
# Claude Code configuration
export ANTHROPIC_API_KEY="your-api-key-here"

# Add to PATH
export PATH="$HOME/.claude/bin:$PATH"

# Aliases
alias cc="claude"
alias cchat="claude chat"

# Optional: Custom prompt integration
claude_prompt() {
  if [[ -n "$CLAUDE_SESSION" ]]; then
    echo "[claude] "
  fi
}
PROMPT='$(claude_prompt)'$PROMPT
```

Reload:

```zsh
source ~/.zshrc
```

#### Fish Configuration

Add to `~/.config/fish/config.fish`:

```fish
# Claude Code configuration
set -gx ANTHROPIC_API_KEY "your-api-key-here"

# Add to PATH
fish_add_path $HOME/.claude/bin

# Aliases
alias cc="claude"
alias cchat="claude chat"

# Function for quick commands
function ask
    claude -p "$argv"
end
```

Reload:

```fish
source ~/.config/fish/config.fish
```

#### PowerShell Configuration

Add to `$PROFILE` (usually `~\Documents\PowerShell\Microsoft.PowerShell_profile.ps1`):

```powershell
# Claude Code configuration
$env:ANTHROPIC_API_KEY = "your-api-key-here"

# Add to PATH if needed
$env:PATH = "$env:USERPROFILE\.claude\bin;$env:PATH"

# Aliases
Set-Alias -Name cc -Value claude
Set-Alias -Name cchat -Value "claude chat"

# Function for quick queries
function Ask-Claude {
    param([string]$Question)
    claude -p $Question
}
Set-Alias -Name ask -Value Ask-Claude
```

Reload:

```powershell
. $PROFILE
```

### The /terminal-setup Command

Claude Code provides a built-in command to help configure your terminal:

```bash
claude /terminal-setup
```

This interactive command will:

1. **Detect your shell**: Identifies bash, zsh, fish, or PowerShell
2. **Check PATH configuration**: Ensures claude is accessible
3. **Verify API key**: Confirms ANTHROPIC_API_KEY is set
4. **Suggest optimizations**: Recommends terminal settings
5. **Create shell integrations**: Sets up aliases and functions

**Example session:**

```
$ claude /terminal-setup

Terminal Setup Wizard
=====================

Detected shell: zsh
Shell config file: /Users/you/.zshrc

Checking configuration...
[✓] Claude Code is in PATH
[✓] ANTHROPIC_API_KEY is set
[!] Shell integration not installed

Would you like to install shell integration? (y/n): y

Adding to ~/.zshrc:
  - Claude aliases (cc, cchat)
  - Path verification
  - Session management functions

Shell integration installed!
Run 'source ~/.zshrc' to activate, or restart your terminal.
```

### PATH Configuration

If Claude isn't found after installation, manually configure PATH:

**Find where claude is installed:**

```bash
# npm global bin location
npm bin -g

# Common locations
ls -la /usr/local/bin/claude
ls -la ~/.npm-global/bin/claude
ls -la ~/.local/bin/claude
```

**Add to PATH permanently:**

```bash
# For bash/zsh, add to shell config
echo 'export PATH="$HOME/.npm-global/bin:$PATH"' >> ~/.bashrc

# For fish
fish_add_path ~/.npm-global/bin

# For PowerShell
[Environment]::SetEnvironmentVariable("PATH", "$env:PATH;$env:USERPROFILE\.npm-global", "User")
```

---

## 3. Basic Command Execution

### How Claude Runs Commands

When you interact with Claude Code, commands are executed through a Bash tool interface:

```
┌─────────────────────────────────────────────────────────────┐
│                 Command Execution Flow                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  You: "Show me the git status of this project"              │
│                  │                                           │
│                  v                                           │
│  ┌──────────────────────────────────────┐                   │
│  │ Claude interprets request            │                   │
│  │ Determines: need to run `git status` │                   │
│  └──────────────────────────────────────┘                   │
│                  │                                           │
│                  v                                           │
│  ┌──────────────────────────────────────┐                   │
│  │ Bash Tool invoked with:              │                   │
│  │   command: "git status"              │                   │
│  │   description: "Show working tree    │                   │
│  │                 status"              │                   │
│  └──────────────────────────────────────┘                   │
│                  │                                           │
│                  v                                           │
│  ┌──────────────────────────────────────┐                   │
│  │ Command executes in shell            │                   │
│  │ Output captured (stdout + stderr)    │                   │
│  └──────────────────────────────────────┘                   │
│                  │                                           │
│                  v                                           │
│  ┌──────────────────────────────────────┐                   │
│  │ Claude receives output               │                   │
│  │ Interprets and explains to you       │                   │
│  └──────────────────────────────────────┘                   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Key aspects:**

1. **Automatic command selection**: Claude chooses appropriate commands based on your request
2. **Context awareness**: Commands use the current working directory
3. **Command chaining**: Multiple related commands can be executed in sequence
4. **Output interpretation**: Claude explains what the output means

### Output Handling

Claude captures and processes command output intelligently:

```
┌────────────────────────────────────────────────────────────┐
│                   Output Processing                         │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  Command Output                                             │
│       │                                                     │
│       v                                                     │
│  ┌─────────────────────────────────────┐                   │
│  │ Stdout captured                      │                   │
│  │ Stderr captured                      │                   │
│  │ Exit code recorded                   │                   │
│  └─────────────────────────────────────┘                   │
│       │                                                     │
│       v                                                     │
│  ┌─────────────────────────────────────┐                   │
│  │ Truncation if > 30,000 characters   │                   │
│  │ (prevents context overflow)          │                   │
│  └─────────────────────────────────────┘                   │
│       │                                                     │
│       v                                                     │
│  ┌─────────────────────────────────────┐                   │
│  │ Claude analyzes:                     │                   │
│  │ - Success/failure                    │                   │
│  │ - Key information                    │                   │
│  │ - Errors to address                  │                   │
│  │ - Next steps needed                  │                   │
│  └─────────────────────────────────────┘                   │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

**Output truncation:**
- Output exceeding 30,000 characters is truncated
- Claude is informed of truncation
- For large outputs, Claude may use targeted approaches (head, tail, grep)

### Error Handling

Claude handles errors gracefully:

```
┌────────────────────────────────────────────────────────────┐
│                    Error Handling                           │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  Error Type          │ Claude's Response                   │
│  ────────────────────┼───────────────────────────────────  │
│  Command not found   │ Suggests installation or            │
│                      │ alternative commands                 │
│  ────────────────────┼───────────────────────────────────  │
│  Permission denied   │ Explains issue, may suggest         │
│                      │ sudo (with caution)                  │
│  ────────────────────┼───────────────────────────────────  │
│  File not found      │ Verifies path, suggests             │
│                      │ corrections                          │
│  ────────────────────┼───────────────────────────────────  │
│  Syntax error        │ Identifies error, provides fix      │
│  ────────────────────┼───────────────────────────────────  │
│  Timeout             │ Explains timeout, suggests          │
│                      │ optimization or background run       │
│  ────────────────────┼───────────────────────────────────  │
│  Exit code != 0      │ Interprets error message,           │
│                      │ suggests resolution                  │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

**Example error handling:**

```
You: Run the tests

Claude: I'll run the tests for you.

[Executing: npm test]

The tests failed with 3 errors:

1. test/auth.test.js:45 - "should validate token"
   Expected: true, Received: false

2. test/auth.test.js:67 - "should reject expired token"
   TypeError: Cannot read property 'exp' of undefined

3. test/api.test.js:23 - "should return 200"
   Expected status 200, got 404

Would you like me to investigate these failures?
```

### Timeout Management

Commands have configurable timeouts to prevent hanging:

```
┌────────────────────────────────────────────────────────────┐
│                  Timeout Configuration                      │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  Default timeout:    120,000 ms (2 minutes)                │
│  Maximum timeout:    600,000 ms (10 minutes)               │
│  Background mode:    No timeout (runs in background)        │
│                                                             │
│  ─────────────────────────────────────────────────────────  │
│                                                             │
│  Timeout Scenarios:                                         │
│                                                             │
│  Quick commands (< 30s):                                    │
│    git status, ls, cat, npm --version                      │
│    → Default timeout is fine                                │
│                                                             │
│  Medium commands (30s - 2min):                              │
│    npm install, docker build (small), test suites          │
│    → Default timeout usually sufficient                     │
│                                                             │
│  Long commands (2min - 10min):                              │
│    Large builds, extensive test suites, deployments        │
│    → Extended timeout or background mode                    │
│                                                             │
│  Very long commands (> 10min):                              │
│    Full CI pipelines, large data processing                │
│    → Must use background mode                               │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

### Working Directory

Claude maintains awareness of the current working directory:

```
┌────────────────────────────────────────────────────────────┐
│               Working Directory Behavior                    │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  Session Start:                                             │
│    Working directory = where you launched Claude            │
│                                                             │
│  During Session:                                            │
│    Claude prefers absolute paths to avoid cd issues        │
│    If cd is used, state persists in shell session          │
│                                                             │
│  Best Practices:                                            │
│    ✓ Use absolute paths: /home/user/project/src/file.js   │
│    ✓ Let Claude manage directory context                   │
│    ✗ Avoid: cd project && do-something (fragile)          │
│    ✓ Prefer: do-something /full/path/to/project           │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

---

## 4. Permission System

### Permission Modes

Claude Code operates in different permission modes:

```
┌────────────────────────────────────────────────────────────┐
│                   Permission Modes                          │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  ASK MODE (Default)                                  │   │
│  │  ─────────────────                                   │   │
│  │  • Prompts before executing potentially dangerous   │   │
│  │    commands                                          │   │
│  │  • Safe commands execute automatically              │   │
│  │  • User maintains control over risky operations     │   │
│  │  • Recommended for most users                       │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  AUTO MODE                                           │   │
│  │  ─────────────                                       │   │
│  │  • Executes allowlisted commands without prompting  │   │
│  │  • Still blocks explicitly dangerous commands       │   │
│  │  • Good for trusted, repetitive workflows          │   │
│  │  • Requires explicit configuration                  │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  DENY MODE                                           │   │
│  │  ───────────                                         │   │
│  │  • Blocks all bash command execution                │   │
│  │  • Claude can only use read-only tools              │   │
│  │  • Maximum safety for code review scenarios         │   │
│  │  • Use when you want analysis only                  │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

**Configuring permission mode:**

```bash
# Set mode for current session
claude --permission-mode ask
claude --permission-mode auto
claude --permission-mode deny

# Set in configuration file (~/.claude/config.json)
{
  "permissions": {
    "mode": "ask",
    "allowlist": ["git *", "npm test", "npm run *"],
    "blocklist": ["rm -rf /", "sudo rm *"]
  }
}
```

### Dangerous Command Detection

Claude identifies and flags potentially dangerous commands:

```
┌────────────────────────────────────────────────────────────┐
│              Dangerous Command Categories                   │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  DESTRUCTIVE FILE OPERATIONS                                │
│  ──────────────────────────                                 │
│  • rm -rf (recursive forced delete)                        │
│  • rm -r / or rm -rf / (root deletion)                     │
│  • > important_file (file overwrite)                       │
│  • dd if=/dev/zero (disk overwrite)                        │
│                                                             │
│  SYSTEM MODIFICATIONS                                       │
│  ───────────────────                                        │
│  • sudo commands (elevated privileges)                      │
│  • chmod 777 (overly permissive)                           │
│  • chown root (ownership changes)                          │
│  • systemctl disable (service changes)                     │
│                                                             │
│  NETWORK OPERATIONS                                         │
│  ─────────────────                                          │
│  • curl | bash (remote code execution)                     │
│  • wget -O- | sh (remote code execution)                   │
│  • iptables (firewall changes)                             │
│  • nc -l (network listeners)                               │
│                                                             │
│  DATA EXPOSURE                                              │
│  ─────────────                                              │
│  • cat ~/.ssh/id_rsa (private key exposure)                │
│  • env (environment variable dump)                         │
│  • history (command history)                               │
│  • Commands with passwords in arguments                     │
│                                                             │
│  PACKAGE MANAGEMENT                                         │
│  ─────────────────                                          │
│  • npm install -g (global package install)                 │
│  • pip install --user (user-level install)                 │
│  • apt install / yum install (system packages)             │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

### Allowlists and Blocklists

Configure which commands are automatically allowed or blocked:

```json
// ~/.claude/config.json
{
  "permissions": {
    "allowlist": [
      "git status",
      "git diff",
      "git log *",
      "git branch *",
      "npm test",
      "npm run lint",
      "npm run build",
      "docker ps",
      "docker logs *",
      "ls *",
      "cat *",
      "head *",
      "tail *"
    ],
    "blocklist": [
      "rm -rf /",
      "rm -rf ~",
      "sudo rm *",
      ":(){ :|:& };:",
      "mkfs.*",
      "dd if=/dev/*",
      "> /dev/sda",
      "chmod -R 777 /",
      "curl * | sudo bash",
      "wget * | sudo sh"
    ]
  }
}
```

**Pattern matching in lists:**
- `*` matches any characters
- Patterns are checked against the full command
- Blocklist takes precedence over allowlist

### Per-Project Permissions

Configure permissions at the project level:

```json
// /your-project/.claude/config.json
{
  "permissions": {
    "mode": "auto",
    "allowlist": [
      "npm *",
      "yarn *",
      "pnpm *",
      "jest *",
      "vitest *",
      "docker-compose *",
      "make *"
    ],
    "projectSpecific": true
  }
}
```

**Permission precedence:**

```
┌────────────────────────────────────────────────────────────┐
│               Permission Precedence                         │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  1. Hardcoded blocklist (always blocked)                   │
│     └── rm -rf /, fork bombs, etc.                         │
│                                                             │
│  2. User blocklist (~/.claude/config.json)                 │
│     └── Your personal blocked commands                     │
│                                                             │
│  3. Project blocklist (.claude/config.json)                │
│     └── Project-specific blocked commands                  │
│                                                             │
│  4. Project allowlist (.claude/config.json)                │
│     └── Project-specific allowed commands                  │
│                                                             │
│  5. User allowlist (~/.claude/config.json)                 │
│     └── Your personal allowed commands                     │
│                                                             │
│  6. Permission mode (ask/auto/deny)                        │
│     └── Determines handling of unlisted commands           │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

---

## 5. Hand-Holding User Journey

Let's walk through using Claude Code's terminal integration step by step.

### Step 1: Run Your First Command Through Claude

**Starting Claude Code:**

```bash
$ claude
```

**Your first interaction:**

```
You: What's in this directory?

Claude: I'll list the contents of the current directory for you.

[Executing: ls -la]

Here's what's in the current directory:

total 48
drwxr-xr-x  8 user user  256 Jan 15 10:30 .
drwxr-xr-x 12 user user  384 Jan 14 09:15 ..
drwxr-xr-x  8 user user  256 Jan 15 10:30 .git
-rw-r--r--  1 user user  234 Jan 15 10:30 .gitignore
-rw-r--r--  1 user user 1205 Jan 15 10:30 package.json
-rw-r--r--  1 user user 8542 Jan 15 10:30 package-lock.json
drwxr-xr-x  4 user user  128 Jan 15 10:30 src
drwxr-xr-x  2 user user   64 Jan 15 10:30 tests
-rw-r--r--  1 user user  521 Jan 15 10:30 README.md

This appears to be a Node.js project with:
- Git version control (.git directory)
- Source code in the src/ directory
- Tests in the tests/ directory
- Standard npm configuration (package.json)

Would you like me to explore any of these further?
```

### Step 2: Handle Command Output

**Understanding output interpretation:**

```
You: Check if there are any git changes

Claude: I'll check the git status for you.

[Executing: git status]

On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
        modified:   src/index.js
        modified:   src/utils/helpers.js

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        src/components/NewFeature.js

no changes added to commit (use "git add" and/or "git commit -a")

Summary:
- You have 2 modified files that haven't been staged
- You have 1 new file that's not being tracked
- Your branch is up to date with origin/main

Would you like me to:
1. Show the diff of the changes?
2. Stage these files for commit?
3. Create a commit with these changes?
```

### Step 3: Chain Commands

**Executing multiple related commands:**

```
You: Yes, show me the diff and then stage everything

Claude: I'll show you the changes and then stage them.

[Executing: git diff]

diff --git a/src/index.js b/src/index.js
index 1234567..abcdefg 100644
--- a/src/index.js
+++ b/src/index.js
@@ -10,6 +10,9 @@ const app = express();
 app.use(cors());
 app.use(express.json());

+// New feature endpoint
+app.use('/api/feature', require('./components/NewFeature'));
+
 app.listen(3000, () => {
   console.log('Server running on port 3000');
 });

diff --git a/src/utils/helpers.js b/src/utils/helpers.js
index 7654321..fedcba9 100644
--- a/src/utils/helpers.js
+++ b/src/utils/helpers.js
@@ -5,3 +5,8 @@ export function formatDate(date) {
   return date.toISOString();
 }
+
+export function validateInput(input) {
+  return input && typeof input === 'string' && input.trim().length > 0;
+}

Changes summary:
1. src/index.js: Added new feature endpoint
2. src/utils/helpers.js: Added validateInput function

[Executing: git add -A]

All changes have been staged successfully:
- 2 modified files
- 1 new file

Ready to commit. Would you like me to create a commit?
```

### Step 4: Background Execution

**Running long processes in the background:**

```
You: Run the full test suite - it takes about 5 minutes

Claude: Since the test suite takes a while, I'll run it in the background
so you can continue working.

[Executing in background: npm test]

The test suite is now running in the background. You can:
- Continue working on other tasks
- I'll notify you when it completes
- Ask me to check the status anytime

You: Okay, while that's running, show me the package.json

Claude: I'll read the package.json file for you.

[Reading: package.json]

{
  "name": "my-project",
  "version": "1.0.0",
  "scripts": {
    "test": "jest --coverage",
    "start": "node src/index.js",
    "dev": "nodemon src/index.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "cors": "^2.8.5"
  }
}

---

[Background task completed]

The test suite has finished! Here are the results:

Test Suites: 12 passed, 12 total
Tests:       47 passed, 47 total
Snapshots:   0 total
Time:        4m 32s
Coverage:    87.3%

All tests passed successfully!
```

### Step 5: Configure Permissions

**Setting up your permission preferences:**

```
You: I want Claude to automatically run git commands without asking

Claude: I can help you configure automatic permissions for git commands.
Here's how to set this up:

Option 1: For this session only
----------------------------------
I can use auto mode for git commands during our current session.
Just tell me "use auto mode for git" and I'll proceed without
confirmation for git operations.

Option 2: Permanent configuration
----------------------------------
Add to your ~/.claude/config.json:

{
  "permissions": {
    "mode": "ask",
    "allowlist": [
      "git status",
      "git diff *",
      "git log *",
      "git branch *",
      "git add *",
      "git commit *",
      "git pull",
      "git push",
      "git fetch",
      "git checkout *",
      "git merge *",
      "git stash *"
    ]
  }
}

This keeps ask mode for other commands but auto-approves git commands.

Which option would you prefer?
```

---

## 6. Real-World Examples

### Example 1: Git Workflow

**Complete git workflow from status to push:**

```
You: I need to commit my changes and push to remote

Claude: I'll help you through the complete git workflow.

[Executing: git status]

On branch feature/user-auth
Changes not staged for commit:
        modified:   src/auth/login.js
        modified:   src/auth/logout.js
        modified:   tests/auth.test.js

[Executing: git diff --stat]

 src/auth/login.js    | 23 +++++++++++++++--------
 src/auth/logout.js   |  8 ++++++--
 tests/auth.test.js   | 45 +++++++++++++++++++++++++++++++++++++++++++
 3 files changed, 66 insertions(+), 10 deletions(-)

[Executing: git log --oneline -3]

a1b2c3d Add user registration endpoint
e4f5g6h Fix password validation
i7j8k9l Initial auth module

I see changes in the auth module. I'll stage and commit them.

[Executing: git add -A]
[Executing: git commit -m "$(cat <<'EOF'
Enhance authentication flow with improved login/logout

- Refactor login.js for better error handling
- Add session cleanup to logout.js
- Add comprehensive tests for auth module

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
EOF
)"]

[feature/user-auth abc1234] Enhance authentication flow...
 3 files changed, 66 insertions(+), 10 deletions(-)

[Executing: git push origin feature/user-auth]

Enumerating objects: 9, done.
Counting objects: 100% (9/9), done.
Delta compression using up to 8 threads
Compressing objects: 100% (5/5), done.
Writing objects: 100% (5/5), 1.23 KiB | 1.23 MiB/s, done.
Total 5 (delta 3), reused 0 (delta 0)
remote: Resolving deltas: 100% (3/3), completed with 3 local objects.
To github.com:user/repo.git
   a1b2c3d..abc1234  feature/user-auth -> feature/user-auth

Done! Your changes have been committed and pushed to origin/feature/user-auth.
```

### Example 2: npm/yarn Package Management

**Managing packages and dependencies:**

```
You: Add testing libraries and update all outdated packages

Claude: I'll help you set up testing and update packages.

[Executing: npm outdated]

Package      Current  Wanted  Latest  Location
express      4.17.1   4.17.1  4.18.2  my-project
cors         2.8.4    2.8.4   2.8.5   my-project
lodash       4.17.19  4.17.19 4.17.21 my-project

Found 3 outdated packages. I'll update them.

[Executing: npm update]

+ express@4.18.2
+ cors@2.8.5
+ lodash@4.17.21

Now I'll add testing libraries.

[Executing: npm install --save-dev jest @testing-library/react @testing-library/jest-dom]

added 245 packages in 12.5s

+ jest@29.7.0
+ @testing-library/react@14.1.2
+ @testing-library/jest-dom@6.2.0

[Executing: npm list --depth=0]

my-project@1.0.0
├── @testing-library/jest-dom@6.2.0
├── @testing-library/react@14.1.2
├── cors@2.8.5
├── express@4.18.2
├── jest@29.7.0
└── lodash@4.17.21

All packages updated and testing libraries installed. Would you like
me to create a basic jest.config.js file?
```

### Example 3: Docker Container Management

**Working with Docker containers:**

```
You: Show me running containers and help me debug the api container

Claude: I'll check your Docker containers and investigate the api container.

[Executing: docker ps]

CONTAINER ID   IMAGE          STATUS          PORTS                    NAMES
a1b2c3d4e5f6   my-api:latest  Up 2 hours      0.0.0.0:3000->3000/tcp   api
b2c3d4e5f6g7   postgres:15    Up 2 hours      0.0.0.0:5432->5432/tcp   db
c3d4e5f6g7h8   redis:alpine   Up 2 hours      0.0.0.0:6379->6379/tcp   cache

[Executing: docker logs api --tail 50]

[2024-01-15T10:30:15Z] Server starting...
[2024-01-15T10:30:16Z] Connected to database
[2024-01-15T10:30:16Z] Connected to Redis
[2024-01-15T10:30:45Z] GET /api/users 200 45ms
[2024-01-15T10:31:02Z] POST /api/auth/login 200 123ms
[2024-01-15T10:35:22Z] ERROR: Connection timeout to database
[2024-01-15T10:35:23Z] Retrying database connection...
[2024-01-15T10:35:25Z] Database reconnected
[2024-01-15T10:40:15Z] GET /api/users 500 5002ms
[2024-01-15T10:40:15Z] ERROR: Query timeout - users table

[Executing: docker stats api --no-stream]

CONTAINER   CPU %     MEM USAGE / LIMIT     MEM %     NET I/O
api         2.34%     256MiB / 512MiB       50.00%    1.2MB / 450KB

Analysis:
- The api container is running but experiencing database timeouts
- Memory usage is at 50% (256MB of 512MB limit)
- Recent errors show query timeouts on the users table

Recommendations:
1. Check database container health
2. Optimize the users table query
3. Consider increasing memory limit if needed

Would you like me to:
1. Check the database container logs?
2. Connect to the database and analyze the users table?
3. Restart the api container?
```

### Example 4: Database Operations

**Working with PostgreSQL:**

```
You: Connect to the postgres container and show me table sizes

Claude: I'll connect to PostgreSQL and analyze table sizes.

[Executing: docker exec db psql -U postgres -d myapp -c "
SELECT
    schemaname || '.' || tablename AS table,
    pg_size_pretty(pg_total_relation_size(schemaname || '.' || tablename)) AS total_size,
    pg_size_pretty(pg_relation_size(schemaname || '.' || tablename)) AS table_size,
    pg_size_pretty(pg_indexes_size(schemaname || '.' || tablename)) AS index_size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname || '.' || tablename) DESC
LIMIT 10;
"]

        table         | total_size | table_size | index_size
----------------------+------------+------------+------------
 public.events        | 2.3 GB     | 1.8 GB     | 512 MB
 public.users         | 456 MB     | 298 MB     | 158 MB
 public.sessions      | 234 MB     | 189 MB     | 45 MB
 public.logs          | 128 MB     | 98 MB      | 30 MB
 public.products      | 45 MB      | 32 MB      | 13 MB
 public.orders        | 23 MB      | 18 MB      | 5 MB
 public.categories    | 128 KB     | 64 KB      | 64 KB

[Executing: docker exec db psql -U postgres -d myapp -c "
SELECT count(*) as row_count FROM events;
"]

 row_count
-----------
  15234567

Analysis:
- The events table is your largest at 2.3 GB with 15M+ rows
- Users table is 456 MB - significant but manageable
- Consider archiving old events or implementing partitioning

Would you like me to:
1. Show the events table structure?
2. Analyze query performance on events?
3. Help set up table partitioning?
```

### Example 5: File System Operations

**Complex file system tasks:**

```
You: Find all JavaScript files larger than 100KB and show me a summary

Claude: I'll search for large JavaScript files and analyze them.

[Executing: find . -name "*.js" -size +100k -type f -exec ls -lh {} \; 2>/dev/null | sort -k5 -hr]

-rw-r--r-- 1 user user 2.1M ./node_modules/lodash/lodash.js
-rw-r--r-- 1 user user 1.8M ./dist/bundle.js
-rw-r--r-- 1 user user 456K ./src/generated/api-types.js
-rw-r--r-- 1 user user 234K ./src/data/countries.js
-rw-r--r-- 1 user user 189K ./src/components/DataGrid.js
-rw-r--r-- 1 user user 145K ./tests/fixtures/large-dataset.js
-rw-r--r-- 1 user user 112K ./src/utils/validators.js

Summary of large JavaScript files (>100KB):

| File                           | Size  | Category      |
|--------------------------------|-------|---------------|
| node_modules/lodash/lodash.js  | 2.1M  | Dependency    |
| dist/bundle.js                 | 1.8M  | Build output  |
| src/generated/api-types.js     | 456K  | Generated     |
| src/data/countries.js          | 234K  | Static data   |
| src/components/DataGrid.js     | 189K  | Component     |
| tests/fixtures/large-dataset.js| 145K  | Test fixture  |
| src/utils/validators.js        | 112K  | Utility       |

Observations:
- The largest files are expected (dependencies, bundles)
- DataGrid.js (189K) might benefit from code splitting
- validators.js (112K) could potentially be modularized

Would you like me to analyze any of these files for optimization?
```

### Example 6: Process Management

**Managing system processes:**

```
You: Show me what's using port 3000 and help me free it up

Claude: I'll identify what's using port 3000.

[Executing: lsof -i :3000]

COMMAND   PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
node    12345  user   23u  IPv4 123456      0t0  TCP *:3000 (LISTEN)

[Executing: ps aux | grep 12345 | grep -v grep]

user     12345  0.5  1.2 1234567 98765 ?  Sl   Jan14  12:34 node /home/user/old-project/server.js

Found it! There's a Node.js process from an old project:
- PID: 12345
- Command: node /home/user/old-project/server.js
- Started: Jan 14
- Memory: ~96MB

This appears to be a leftover development server from old-project.

[!] To free up port 3000, I would need to kill this process:
    kill 12345

Should I terminate this process? (This will stop the old-project server)

You: Yes, kill it

Claude: [Executing: kill 12345]

[Executing: lsof -i :3000]

(no output - port is now free)

Process terminated successfully. Port 3000 is now available.
```

### Example 7: Log Analysis

**Analyzing application logs:**

```
You: Analyze the last hour of error logs and summarize issues

Claude: I'll analyze recent error logs.

[Executing: find /var/log/myapp -name "*.log" -mmin -60 -exec grep -l "ERROR\|FATAL" {} \;]

/var/log/myapp/app.log
/var/log/myapp/worker.log

[Executing: grep -E "ERROR|FATAL" /var/log/myapp/app.log | tail -100 | sort | uniq -c | sort -rn]

     23 [ERROR] Database connection timeout
     15 [ERROR] Redis cache miss - fallback to DB
      8 [ERROR] JWT token validation failed
      5 [ERROR] Rate limit exceeded for IP
      3 [ERROR] File upload failed - size exceeded
      1 [FATAL] Out of memory - restarting worker

[Executing: grep -E "ERROR|FATAL" /var/log/myapp/worker.log | tail -50 | sort | uniq -c | sort -rn]

     12 [ERROR] Job processing timeout
      7 [ERROR] Failed to send email - SMTP error
      2 [ERROR] Queue overflow - dropping oldest jobs

Error Log Summary (Last Hour):
==============================

Application Server (app.log):
-----------------------------
| Count | Issue                              | Severity |
|-------|------------------------------------|----------|
| 23    | Database connection timeout        | High     |
| 15    | Redis cache miss                   | Medium   |
| 8     | JWT validation failed              | Medium   |
| 5     | Rate limit exceeded                | Low      |
| 3     | File upload size exceeded          | Low      |
| 1     | Out of memory (caused restart)     | Critical |

Worker (worker.log):
--------------------
| Count | Issue                              | Severity |
|-------|------------------------------------|----------|
| 12    | Job processing timeout             | High     |
| 7     | SMTP email sending failed          | Medium   |
| 2     | Queue overflow                     | High     |

Primary Issues to Address:
1. Database connectivity (23 timeouts) - check DB health/connections
2. Worker job timeouts (12 occurrences) - investigate slow jobs
3. Memory issue caused a restart - review memory allocation

Would you like me to investigate any of these in detail?
```

---

## 7. Background Execution

### Running Long Commands in Background

For commands that take longer than the timeout limit or when you want to continue working:

```
┌────────────────────────────────────────────────────────────┐
│              Background Execution Flow                      │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  You: "Run the full test suite in background"              │
│                   │                                         │
│                   v                                         │
│  ┌─────────────────────────────────────┐                   │
│  │ Claude starts command with          │                   │
│  │ run_in_background: true             │                   │
│  └─────────────────────────────────────┘                   │
│                   │                                         │
│                   v                                         │
│  ┌─────────────────────────────────────┐                   │
│  │ Command spawned in background       │                   │
│  │ Process ID tracked                  │                   │
│  │ Output accumulated                  │                   │
│  └─────────────────────────────────────┘                   │
│                   │                                         │
│                   v                                         │
│  ┌─────────────────────────────────────┐                   │
│  │ You continue with other tasks       │                   │
│  │ Claude can check status anytime     │                   │
│  └─────────────────────────────────────┘                   │
│                   │                                         │
│                   v                                         │
│  ┌─────────────────────────────────────┐                   │
│  │ When complete:                       │                   │
│  │ - Notification shown                │                   │
│  │ - Full output available             │                   │
│  └─────────────────────────────────────┘                   │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

**Example usage:**

```
You: Run npm install and npm run build in the background - it takes a while

Claude: I'll run the installation and build in the background.

[Starting background task: npm install && npm run build]

Background task started. I'll notify you when it completes.
You can continue with other tasks in the meantime.

Task ID: bg-12345
Started: 2024-01-15 10:30:00

---

[5 minutes later]

Background task completed (bg-12345)
Duration: 4m 32s
Exit code: 0

Output summary:
- npm install: 234 packages installed
- npm run build: Successfully compiled
  - Output: dist/bundle.js (1.2 MB)
  - Output: dist/bundle.css (45 KB)

Would you like to see the full output?
```

### Checking Command Status

```
You: What background tasks are running?

Claude: [Checking background tasks]

Active Background Tasks:
------------------------
| ID        | Command              | Duration | Status    |
|-----------|----------------------|----------|-----------|
| bg-12345  | npm test --coverage  | 2m 15s   | Running   |
| bg-12346  | docker build .       | 45s      | Running   |

Completed Tasks (this session):
-------------------------------
| ID        | Command              | Duration | Exit Code |
|-----------|----------------------|----------|-----------|
| bg-12340  | npm install          | 1m 23s   | 0         |
| bg-12341  | git fetch --all      | 5s       | 0         |

Would you like to check output from any of these?
```

### Retrieving Output

```
You: Show me the output from bg-12345

Claude: [Retrieving output from bg-12345]

Command: npm test --coverage
Status: Completed
Duration: 4m 32s
Exit Code: 0

Output:
-------
> my-project@1.0.0 test
> jest --coverage

PASS tests/auth.test.js (2.345s)
PASS tests/api.test.js (1.234s)
PASS tests/utils.test.js (0.567s)
...

Test Suites: 15 passed, 15 total
Tests:       89 passed, 89 total
Coverage:    87.5%

----------|---------|----------|---------|---------|
File      | % Stmts | % Branch | % Funcs | % Lines |
----------|---------|----------|---------|---------|
All files |   87.5  |    82.3  |   91.2  |   87.5  |
----------|---------|----------|---------|---------|
```

### Killing Background Processes

```
You: The docker build is taking too long, cancel it

Claude: [Terminating background task bg-12346]

Task bg-12346 (docker build .) has been terminated.

Duration before termination: 3m 45s

Partial output captured:
- Step 1/12: FROM node:18-alpine
- Step 2/12: WORKDIR /app
- Step 3/12: COPY package*.json ./
- Step 4/12: RUN npm ci
(terminated during npm ci)

The Docker build has been stopped. Would you like me to:
1. Restart with different options?
2. Check if there's a caching issue?
3. Try a different approach?
```

---

## 8. Safety Features

### Sandboxing Explanation

Claude Code can operate in a sandboxed environment for maximum security:

```
┌────────────────────────────────────────────────────────────┐
│                 Sandbox Architecture                        │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌───────────────────────────────────────────────────┐     │
│  │              Host System                           │     │
│  │  ┌─────────────────────────────────────────────┐  │     │
│  │  │           Sandbox Container                  │  │     │
│  │  │  ┌───────────────────────────────────────┐  │  │     │
│  │  │  │         Claude Code Process           │  │  │     │
│  │  │  │                                       │  │  │     │
│  │  │  │  • Limited file system access        │  │  │     │
│  │  │  │  • Network restrictions              │  │  │     │
│  │  │  │  • No privileged operations          │  │  │     │
│  │  │  │  • Resource limits enforced          │  │  │     │
│  │  │  │                                       │  │  │     │
│  │  │  └───────────────────────────────────────┘  │  │     │
│  │  │                                             │  │     │
│  │  │  Allowed:                                   │  │     │
│  │  │  • Read/write project directory            │  │     │
│  │  │  • Execute approved commands               │  │     │
│  │  │  • Limited network (npm, git)              │  │     │
│  │  │                                             │  │     │
│  │  │  Blocked:                                   │  │     │
│  │  │  • Access outside project                  │  │     │
│  │  │  • System modifications                    │  │     │
│  │  │  • Arbitrary network access                │  │     │
│  │  │                                             │  │     │
│  │  └─────────────────────────────────────────────┘  │     │
│  └───────────────────────────────────────────────────┘     │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

**Enabling sandboxing:**

```bash
# Run Claude in sandboxed mode
claude --sandbox

# Or configure in settings
claude config set sandbox.enabled true
```

### Dangerous Command Blocking

Claude automatically blocks or warns about dangerous commands:

```
┌────────────────────────────────────────────────────────────┐
│              Dangerous Command Handling                     │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  ALWAYS BLOCKED (cannot override):                         │
│  ─────────────────────────────────                         │
│  • rm -rf /                                                │
│  • rm -rf ~                                                │
│  • dd if=/dev/zero of=/dev/sda                            │
│  • mkfs.* /dev/sd*                                         │
│  • :(){ :|:& };:  (fork bomb)                             │
│  • > /dev/sda                                              │
│                                                             │
│  REQUIRES CONFIRMATION (ask mode):                         │
│  ──────────────────────────────────                        │
│  • sudo *                                                  │
│  • rm -rf * (in any directory)                            │
│  • chmod -R 777                                            │
│  • curl/wget | bash                                        │
│  • Database drop commands                                  │
│  • Service stop/disable commands                           │
│                                                             │
│  WARNING SHOWN (but allowed):                              │
│  ────────────────────────────                              │
│  • git push --force                                        │
│  • git reset --hard                                        │
│  • npm publish                                             │
│  • docker system prune                                     │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

**Example of blocked command:**

```
You: Delete everything in the root directory

Claude: I cannot execute that command. Deleting the root directory
(rm -rf /) is permanently blocked as it would destroy your system.

If you have a legitimate need to remove files, please specify:
1. The exact directory you want to clean
2. What files should be removed
3. Why this operation is needed

I can help with targeted cleanup operations that won't harm your system.
```

### Confirmation Prompts

When potentially dangerous commands are detected:

```
You: Clean up all docker resources

Claude: I can clean up Docker resources. This command will remove:
- All stopped containers
- All unused networks
- All dangling images
- All build cache

[Proposed command: docker system prune -af]

⚠️  Warning: This will permanently delete Docker resources.

Do you want to proceed? (yes/no)

You: yes

Claude: [Executing: docker system prune -af]

Deleted Containers:
a1b2c3d4e5f6
b2c3d4e5f6g7

Deleted Networks:
my-network
test-network

Deleted Images:
sha256:abc123... (1.2GB)
sha256:def456... (890MB)

Total reclaimed space: 4.5GB

Docker cleanup complete!
```

### Audit Logging

Claude Code maintains logs of all executed commands:

```
┌────────────────────────────────────────────────────────────┐
│                    Audit Log                                │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  Location: ~/.claude/logs/audit.log                        │
│                                                             │
│  Log Entry Format:                                          │
│  ─────────────────                                          │
│  {                                                          │
│    "timestamp": "2024-01-15T10:30:00Z",                    │
│    "session_id": "sess_abc123",                            │
│    "command": "git commit -m 'Update'",                    │
│    "working_dir": "/home/user/project",                    │
│    "exit_code": 0,                                         │
│    "duration_ms": 234,                                     │
│    "user_approved": true,                                  │
│    "permission_mode": "ask"                                │
│  }                                                          │
│                                                             │
│  Viewing Logs:                                              │
│  ─────────────                                              │
│  claude logs                    # View recent              │
│  claude logs --since 1h        # Last hour                 │
│  claude logs --filter "git"    # Git commands only         │
│  claude logs --json            # JSON output               │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

---

## 9. Platform-Specific Guidance

### macOS Specifics

```
┌────────────────────────────────────────────────────────────┐
│                    macOS Guide                              │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  Installation:                                              │
│  ─────────────                                              │
│  # Using Homebrew (recommended)                            │
│  brew install claude-code                                  │
│                                                             │
│  # Or npm                                                   │
│  npm install -g @anthropic-ai/claude-code                  │
│                                                             │
│  Shell:                                                     │
│  ──────                                                     │
│  Default shell: zsh (macOS Catalina+)                      │
│  Config file: ~/.zshrc                                     │
│                                                             │
│  Common Commands:                                           │
│  ────────────────                                           │
│  open .                    # Open Finder here              │
│  pbcopy < file            # Copy file to clipboard         │
│  pbpaste > file           # Paste clipboard to file        │
│  say "done"               # Text to speech                 │
│  caffeinate -t 3600       # Prevent sleep for 1 hour       │
│                                                             │
│  Permissions:                                               │
│  ───────────                                                │
│  May need to grant Terminal "Full Disk Access" in          │
│  System Preferences > Security & Privacy > Privacy         │
│                                                             │
│  Path Issues:                                               │
│  ────────────                                               │
│  Homebrew paths: /opt/homebrew/bin (Apple Silicon)         │
│                  /usr/local/bin (Intel)                    │
│  Add to ~/.zshrc: eval "$(/opt/homebrew/bin/brew shellenv)"│
│                                                             │
└────────────────────────────────────────────────────────────┘
```

**macOS-specific examples:**

```bash
# Open VS Code from terminal
code .

# Reveal file in Finder
open -R /path/to/file

# Quick Look preview
qlmanage -p /path/to/file

# Spotlight search
mdfind "kind:code filename:test"

# Check system info
system_profiler SPSoftwareDataType
```

### Linux Specifics

```
┌────────────────────────────────────────────────────────────┐
│                    Linux Guide                              │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  Installation:                                              │
│  ─────────────                                              │
│  # Debian/Ubuntu                                            │
│  curl -fsSL https://claude.ai/install.sh | sudo sh        │
│                                                             │
│  # Or npm                                                   │
│  npm install -g @anthropic-ai/claude-code                  │
│                                                             │
│  Shell:                                                     │
│  ──────                                                     │
│  Default: bash                                              │
│  Config: ~/.bashrc or ~/.bash_profile                      │
│                                                             │
│  Distribution-Specific:                                     │
│  ──────────────────────                                     │
│  Ubuntu/Debian:  apt, dpkg, systemctl                      │
│  RHEL/CentOS:    dnf/yum, rpm, systemctl                   │
│  Arch:           pacman, systemctl                          │
│  Alpine:         apk, rc-service                            │
│                                                             │
│  Common Commands:                                           │
│  ────────────────                                           │
│  xdg-open .              # Open file manager               │
│  xclip -selection c      # Copy to clipboard               │
│  systemctl status nginx  # Service status                  │
│  journalctl -f           # Follow system logs              │
│                                                             │
│  Permissions:                                               │
│  ───────────                                                │
│  Use sudo for system operations                            │
│  Add user to groups: sudo usermod -aG docker $USER         │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

**Linux-specific examples:**

```bash
# Check disk usage
df -h

# Monitor system resources
htop
top

# Service management
systemctl status nginx
journalctl -u nginx -f

# Package management (Ubuntu)
apt update && apt upgrade
apt install package-name

# Find files
locate filename
find / -name "*.conf" -type f 2>/dev/null
```

### Windows (PowerShell, Git Bash, WSL)

```
┌────────────────────────────────────────────────────────────┐
│                   Windows Guide                             │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  OPTION 1: PowerShell (Native Windows)                     │
│  ─────────────────────────────────────                     │
│  Installation:                                              │
│    npm install -g @anthropic-ai/claude-code                │
│                                                             │
│  Config: $PROFILE                                           │
│  (~\Documents\PowerShell\Microsoft.PowerShell_profile.ps1) │
│                                                             │
│  Common Commands:                                           │
│    Get-ChildItem (ls)     # List files                     │
│    Get-Content (cat)      # Read file                      │
│    Set-Location (cd)      # Change directory               │
│    Get-Process            # List processes                 │
│    Stop-Process -Name x   # Kill process                   │
│                                                             │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  OPTION 2: Git Bash (Unix-like on Windows)                 │
│  ─────────────────────────────────────────                 │
│  Installation:                                              │
│    1. Install Git for Windows                              │
│    2. npm install -g @anthropic-ai/claude-code             │
│                                                             │
│  Config: ~/.bashrc                                          │
│                                                             │
│  Notes:                                                     │
│    - Unix commands work (ls, cat, grep)                    │
│    - Windows paths: /c/Users/name/                         │
│    - Some tools need winpty prefix                         │
│                                                             │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  OPTION 3: WSL2 (Full Linux on Windows)                    │
│  ──────────────────────────────────────                    │
│  Installation:                                              │
│    1. wsl --install                                        │
│    2. Inside WSL: npm install -g @anthropic-ai/claude-code │
│                                                             │
│  Benefits:                                                  │
│    - Full Linux environment                                │
│    - Native Linux commands                                 │
│    - Docker support                                         │
│    - Access Windows files via /mnt/c/                      │
│                                                             │
│  Config: ~/.bashrc (in WSL)                                │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

**Windows-specific examples:**

```powershell
# PowerShell examples
Get-Process | Where-Object CPU -gt 10
Get-Service | Where-Object Status -eq "Running"
Get-EventLog -LogName Application -Newest 10
New-Item -ItemType Directory -Path ".\new-folder"
Compress-Archive -Path .\folder -DestinationPath .\archive.zip

# Open Explorer here
explorer .

# Open VS Code
code .
```

```bash
# Git Bash examples
# Use Unix commands with Windows paths
ls /c/Users/
cat /c/Users/name/file.txt
cd /c/Projects/my-app
```

```bash
# WSL examples
# Access Windows files
cd /mnt/c/Users/name/Projects

# Run Windows executables from WSL
explorer.exe .
code .

# Use Linux tools on Windows files
grep -r "pattern" /mnt/c/Projects/
```

### Cross-Platform Commands

Commands that work across all platforms:

```
┌────────────────────────────────────────────────────────────┐
│              Cross-Platform Command Reference               │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  Git (all platforms):                                       │
│  ────────────────────                                       │
│  git status, git add, git commit, git push, git pull       │
│  git branch, git checkout, git merge, git log              │
│                                                             │
│  Node.js/npm (all platforms):                              │
│  ────────────────────────────                              │
│  npm install, npm run, npm test, npm start                 │
│  node script.js, npx command                               │
│                                                             │
│  Python (all platforms):                                    │
│  ───────────────────────                                    │
│  python script.py, pip install, python -m venv             │
│                                                             │
│  Docker (all platforms):                                    │
│  ───────────────────────                                    │
│  docker ps, docker build, docker run, docker-compose       │
│                                                             │
│  Platform-Specific Alternatives:                            │
│  ───────────────────────────────                            │
│  │ Task          │ Unix/Mac      │ Windows PS    │         │
│  ├───────────────┼───────────────┼───────────────┤         │
│  │ List files    │ ls -la        │ Get-ChildItem │         │
│  │ Read file     │ cat file      │ Get-Content   │         │
│  │ Find text     │ grep pattern  │ Select-String │         │
│  │ Environment   │ export VAR=x  │ $env:VAR="x"  │         │
│  │ Process list  │ ps aux        │ Get-Process   │         │
│  │ Network       │ netstat -an   │ netstat -an   │         │
│  │ Disk space    │ df -h         │ Get-Volume    │         │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

---

## 10. Pros and Cons

### Advantages of Terminal Integration

```
┌────────────────────────────────────────────────────────────┐
│                       PROS                                  │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  DIRECT EXECUTION POWER                                     │
│  ─────────────────────                                      │
│  ✓ Full access to system commands                          │
│  ✓ No need to copy-paste between AI and terminal           │
│  ✓ Claude can chain complex operations                     │
│  ✓ Real-time feedback and error handling                   │
│  ✓ Context-aware command selection                         │
│                                                             │
│  WORKFLOW INTEGRATION                                       │
│  ────────────────────                                       │
│  ✓ Seamless git operations                                 │
│  ✓ Package management automation                           │
│  ✓ Build and test automation                               │
│  ✓ Container orchestration                                 │
│  ✓ Database interactions                                   │
│                                                             │
│  LEARNING & PRODUCTIVITY                                    │
│  ───────────────────────                                    │
│  ✓ Learn commands by seeing Claude use them                │
│  ✓ Reduces need to remember syntax                         │
│  ✓ Faster than manual typing for complex commands          │
│  ✓ Error messages explained automatically                  │
│                                                             │
│  INTELLIGENCE                                               │
│  ────────────                                               │
│  ✓ Interprets output and suggests next steps              │
│  ✓ Handles errors gracefully                               │
│  ✓ Chooses appropriate commands for your intent           │
│  ✓ Adapts to your project structure                        │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

### Disadvantages and Limitations

```
┌────────────────────────────────────────────────────────────┐
│                       CONS                                  │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  SAFETY OVERHEAD                                            │
│  ──────────────                                             │
│  ✗ Permission prompts can interrupt flow                   │
│  ✗ Some legitimate commands may be blocked                 │
│  ✗ Requires trust configuration for full automation        │
│  ✗ Sandboxing limits some advanced operations              │
│                                                             │
│  PERFORMANCE CONSIDERATIONS                                 │
│  ─────────────────────────                                  │
│  ✗ Latency for each command (API round trip)              │
│  ✗ Large outputs get truncated                             │
│  ✗ Very long commands need background mode                 │
│  ✗ Not suitable for interactive commands (vim, top)        │
│                                                             │
│  LIMITATIONS                                                │
│  ───────────                                                │
│  ✗ Cannot run interactive TUI applications                 │
│  ✗ No support for commands requiring user input            │
│  ✗ Limited to 10-minute maximum execution time            │
│  ✗ Some platform-specific tools may not work               │
│                                                             │
│  CONTEXT CONSTRAINTS                                        │
│  ───────────────────                                        │
│  ✗ Output truncation may lose important information        │
│  ✗ Very large repositories can overwhelm context          │
│  ✗ Shell state may not persist perfectly across calls     │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

### When Terminal Integration Excels

```
┌────────────────────────────────────────────────────────────┐
│               IDEAL USE CASES                               │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  ★ Git workflows (status, commit, push, branching)         │
│  ★ Package management (npm, pip, cargo)                    │
│  ★ Running test suites                                     │
│  ★ Build processes                                          │
│  ★ Docker container management                             │
│  ★ File system exploration                                 │
│  ★ Log analysis                                             │
│  ★ Database queries                                         │
│  ★ Process management                                       │
│  ★ System diagnostics                                       │
│                                                             │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│              POOR USE CASES                                 │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  ✗ Interactive editors (vim, nano, emacs)                  │
│  ✗ Real-time monitoring (htop, watch)                      │
│  ✗ Password prompts (ssh first connection)                 │
│  ✗ GUI applications                                        │
│  ✗ Very long-running processes (hours)                     │
│  ✗ Commands requiring stdin input                          │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

---

## 11. When to Use Bash vs Other Tools

### Bash vs Read Tool

```
┌────────────────────────────────────────────────────────────┐
│              Bash vs Read Tool                              │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  USE READ TOOL WHEN:                                        │
│  ───────────────────                                        │
│  ✓ Reading source code files                               │
│  ✓ Viewing configuration files                             │
│  ✓ Examining specific file contents                        │
│  ✓ Reading images (Read supports visual analysis)          │
│  ✓ Reading PDFs                                             │
│  ✓ Reading Jupyter notebooks                               │
│                                                             │
│  Read tool advantages:                                      │
│  • Line numbers included                                   │
│  • Handles binary files (images, PDFs)                     │
│  • Offset/limit support for large files                    │
│  • UTF-8 guaranteed                                         │
│  • No shell escaping issues                                │
│                                                             │
│  USE BASH (cat/head/tail) WHEN:                            │
│  ──────────────────────────────                            │
│  ✓ Reading logs with specific patterns (grep + cat)        │
│  ✓ Processing file content with pipes                      │
│  ✓ Reading from non-file sources (network, devices)        │
│  ✓ Complex text transformations needed                     │
│                                                             │
│  EXAMPLES:                                                  │
│  ─────────                                                  │
│  # Use Read:                                                │
│  Claude uses Read tool for: src/index.js                   │
│                                                             │
│  # Use Bash only for processing:                           │
│  cat /var/log/app.log | grep ERROR | tail -20              │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

### Bash vs Write Tool

```
┌────────────────────────────────────────────────────────────┐
│              Bash vs Write Tool                             │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  USE WRITE TOOL WHEN:                                       │
│  ────────────────────                                       │
│  ✓ Creating new source files                               │
│  ✓ Overwriting configuration files                         │
│  ✓ Writing any text content to files                       │
│                                                             │
│  Write tool advantages:                                     │
│  • Clean content handling                                  │
│  • No shell escaping needed                                │
│  • Atomic writes                                            │
│  • Proper encoding handling                                │
│  • Safety checks built-in                                  │
│                                                             │
│  USE BASH (echo/redirect) WHEN:                            │
│  ──────────────────────────────                            │
│  ✓ Appending to existing files                             │
│  ✓ Creating files with shell variable expansion            │
│  ✓ Writing command output to files                         │
│  ✓ Multi-file operations                                   │
│                                                             │
│  EXAMPLES:                                                  │
│  ─────────                                                  │
│  # Use Write tool:                                          │
│  Claude uses Write tool for: src/newComponent.js           │
│                                                             │
│  # Use Bash for appending:                                  │
│  echo "new line" >> existing-file.txt                      │
│                                                             │
│  # Use Bash for command output:                            │
│  date >> deployment.log                                     │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

### Bash vs Grep Tool

```
┌────────────────────────────────────────────────────────────┐
│              Bash vs Grep Tool                              │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  USE GREP TOOL WHEN:                                        │
│  ───────────────────                                        │
│  ✓ Searching code across a codebase                        │
│  ✓ Finding pattern occurrences                             │
│  ✓ Locating function definitions                           │
│  ✓ Searching with context lines                            │
│                                                             │
│  Grep tool advantages:                                      │
│  • Optimized for large codebases                           │
│  • Proper permission handling                              │
│  • Multiple output modes                                   │
│  • Built-in file type filtering                            │
│  • Multiline pattern support                               │
│                                                             │
│  USE BASH (grep/rg) WHEN:                                  │
│  ────────────────────────                                   │
│  ✓ Piping output from other commands                       │
│  ✓ Complex pipelines with multiple filters                 │
│  ✓ Counting/statistics operations                          │
│  ✓ Log file analysis with time filtering                   │
│                                                             │
│  EXAMPLES:                                                  │
│  ─────────                                                  │
│  # Use Grep tool:                                           │
│  Claude uses Grep tool for: "useEffect" in *.tsx           │
│                                                             │
│  # Use Bash for pipelines:                                  │
│  docker logs app | grep -E "ERROR|WARN" | sort | uniq -c  │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

### Decision Guide

```
┌────────────────────────────────────────────────────────────┐
│                 Tool Selection Flowchart                    │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  Need to READ a file?                                       │
│  ├── Yes → Is it a code/config file?                       │
│  │         ├── Yes → Use Read tool                         │
│  │         └── No  → Is it log analysis with grep?         │
│  │                   ├── Yes → Use Bash (grep|cat)         │
│  │                   └── No  → Use Read tool               │
│  └── No                                                     │
│                                                             │
│  Need to WRITE a file?                                      │
│  ├── Yes → Is it new content or full replacement?          │
│  │         ├── Yes → Use Write tool                        │
│  │         └── No  → Is it appending or modifying?         │
│  │                   ├── Appending → Use Bash (>>)         │
│  │                   └── Editing → Use Edit tool           │
│  └── No                                                     │
│                                                             │
│  Need to SEARCH for patterns?                               │
│  ├── Yes → Is it across multiple files?                    │
│  │         ├── Yes → Use Grep tool                         │
│  │         └── No  → Is it command output filtering?       │
│  │                   ├── Yes → Use Bash (|grep)            │
│  │                   └── No  → Use Grep tool               │
│  └── No                                                     │
│                                                             │
│  Need to EXECUTE something?                                 │
│  ├── Yes → Use Bash tool                                   │
│  └── No  → Determine actual need                           │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

---

## 12. Common Patterns

### Command Chaining

```
┌────────────────────────────────────────────────────────────┐
│              Command Chaining Operators                     │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  && (AND) - Run next command only if previous succeeds     │
│  ─────────────────────────────────────────────────────     │
│  npm install && npm test && npm build                      │
│  │              │            └── Runs if test passes       │
│  │              └── Runs if install succeeds               │
│  └── First command                                          │
│                                                             │
│  Example: Build only if tests pass                         │
│  npm test && npm run build && echo "Success!"              │
│                                                             │
│  || (OR) - Run next command only if previous fails         │
│  ──────────────────────────────────────────────────        │
│  npm test || echo "Tests failed!"                          │
│  │           └── Runs only if test fails                   │
│  └── First command                                          │
│                                                             │
│  Example: Fallback behavior                                 │
│  docker-compose up || docker compose up                    │
│                                                             │
│  ; (SEQUENCE) - Run regardless of previous result          │
│  ────────────────────────────────────────────────          │
│  echo "Starting"; npm test; echo "Done"                    │
│  │                │          └── Always runs               │
│  │                └── Always runs                          │
│  └── First command                                          │
│                                                             │
│  COMBINING PATTERNS                                         │
│  ──────────────────                                         │
│  # If test passes, build; otherwise show error             │
│  npm test && npm build || echo "Build failed"              │
│                                                             │
│  # Complex workflow                                         │
│  git fetch && git pull && npm install && npm test          │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

### Piping and Redirection

```
┌────────────────────────────────────────────────────────────┐
│              Pipes and Redirection                          │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  PIPES (|) - Send output to next command                   │
│  ───────────────────────────────────────                   │
│  command1 | command2 | command3                            │
│                                                             │
│  Examples:                                                  │
│  # Find large files                                        │
│  ls -la | sort -k5 -n | tail -10                          │
│                                                             │
│  # Count error lines                                       │
│  cat app.log | grep ERROR | wc -l                         │
│                                                             │
│  # Unique sorted list                                      │
│  cat names.txt | sort | uniq                               │
│                                                             │
│  REDIRECTION (>, >>, <)                                    │
│  ────────────────────────                                   │
│  >   Overwrite file                                        │
│  >>  Append to file                                        │
│  <   Read from file                                        │
│  2>  Redirect stderr                                       │
│  2>&1 Combine stderr and stdout                            │
│                                                             │
│  Examples:                                                  │
│  # Save output to file                                     │
│  npm test > test-results.txt                               │
│                                                             │
│  # Append to log                                           │
│  echo "$(date): Deployment" >> deploy.log                  │
│                                                             │
│  # Capture errors separately                               │
│  npm build 2> errors.txt                                   │
│                                                             │
│  # Capture both stdout and stderr                          │
│  npm test > output.txt 2>&1                                │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

### Environment Variables

```
┌────────────────────────────────────────────────────────────┐
│              Environment Variables                          │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  SETTING VARIABLES                                          │
│  ─────────────────                                          │
│  # For current command only                                │
│  NODE_ENV=production npm run build                         │
│                                                             │
│  # For session                                              │
│  export DATABASE_URL="postgres://localhost/mydb"           │
│                                                             │
│  # Multiple variables                                       │
│  export NODE_ENV=test DEBUG=true && npm test               │
│                                                             │
│  READING VARIABLES                                          │
│  ─────────────────                                          │
│  echo $HOME                                                 │
│  echo $PATH                                                 │
│  echo ${DATABASE_URL:-default_value}  # With default       │
│                                                             │
│  COMMON PATTERNS                                            │
│  ───────────────                                            │
│  # Run with test config                                    │
│  NODE_ENV=test npm test                                    │
│                                                             │
│  # Use .env file                                           │
│  source .env && npm start                                  │
│                                                             │
│  # Export from .env                                        │
│  export $(cat .env | xargs) && npm start                   │
│                                                             │
│  # Check if variable exists                                │
│  if [ -z "$API_KEY" ]; then echo "Missing API_KEY"; fi    │
│                                                             │
│  CROSS-PLATFORM (Node.js)                                  │
│  ────────────────────────                                   │
│  # Using cross-env package                                 │
│  npx cross-env NODE_ENV=production npm run build           │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

### Script Execution

```
┌────────────────────────────────────────────────────────────┐
│              Script Execution Patterns                      │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  RUNNING SCRIPTS                                            │
│  ───────────────                                            │
│  # Bash scripts                                             │
│  bash script.sh                                             │
│  ./script.sh  # If executable                              │
│  source script.sh  # In current shell                      │
│                                                             │
│  # Node.js scripts                                          │
│  node script.js                                             │
│  npm run script-name                                       │
│  npx script-name                                            │
│                                                             │
│  # Python scripts                                           │
│  python script.py                                           │
│  python3 script.py                                          │
│                                                             │
│  INLINE SCRIPTS                                             │
│  ──────────────                                             │
│  # Bash inline                                              │
│  bash -c 'for i in 1 2 3; do echo $i; done'               │
│                                                             │
│  # Node.js inline                                           │
│  node -e "console.log(process.version)"                    │
│                                                             │
│  # Python inline                                            │
│  python -c "import sys; print(sys.version)"                │
│                                                             │
│  MAKING SCRIPTS EXECUTABLE                                  │
│  ─────────────────────────                                  │
│  chmod +x script.sh                                         │
│  ./script.sh                                                │
│                                                             │
│  SCRIPT WITH ARGUMENTS                                      │
│  ─────────────────────                                      │
│  ./deploy.sh production                                     │
│  node migrate.js --direction=up                            │
│  python process.py --input=data.csv --output=result.json   │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

---

## 13. Troubleshooting

### Permission Denied Errors

```
┌────────────────────────────────────────────────────────────┐
│              Permission Denied Errors                       │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  SYMPTOMS:                                                  │
│  ──────────                                                 │
│  bash: ./script.sh: Permission denied                      │
│  Error: EACCES: permission denied, open '/path/file'       │
│                                                             │
│  COMMON CAUSES AND SOLUTIONS:                               │
│  ────────────────────────────                               │
│                                                             │
│  1. Script not executable                                   │
│     Problem: ./script.sh returns permission denied         │
│     Solution: chmod +x script.sh                           │
│                                                             │
│  2. Directory not writable                                  │
│     Problem: Cannot create file in directory               │
│     Solution: Check ownership and permissions              │
│     ls -la /path/to/directory                              │
│     chmod 755 /path/to/directory  # or chown               │
│                                                             │
│  3. File owned by different user                           │
│     Problem: Cannot modify file                            │
│     Solution: Change ownership or use sudo                 │
│     sudo chown $USER:$USER filename                        │
│                                                             │
│  4. npm global install fails                               │
│     Problem: EACCES when npm install -g                    │
│     Solutions:                                              │
│     a) Use nvm (recommended)                               │
│     b) Fix npm permissions:                                │
│        mkdir ~/.npm-global                                 │
│        npm config set prefix '~/.npm-global'               │
│        Add to PATH: ~/.npm-global/bin                      │
│                                                             │
│  5. Docker permission denied                               │
│     Problem: Cannot connect to Docker daemon               │
│     Solution: Add user to docker group                     │
│     sudo usermod -aG docker $USER                          │
│     # Then log out and back in                             │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

### Command Not Found

```
┌────────────────────────────────────────────────────────────┐
│              Command Not Found Errors                       │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  SYMPTOMS:                                                  │
│  ──────────                                                 │
│  bash: command: command not found                          │
│  zsh: command not found: command                           │
│                                                             │
│  DIAGNOSTIC STEPS:                                          │
│  ─────────────────                                          │
│                                                             │
│  1. Check if command exists                                │
│     which command                                          │
│     type command                                           │
│     command -v command                                     │
│                                                             │
│  2. Check PATH                                             │
│     echo $PATH                                              │
│     # Look for expected directories                        │
│                                                             │
│  3. Find command location                                  │
│     # macOS                                                 │
│     mdfind -name "command"                                 │
│     # Linux                                                 │
│     locate command                                         │
│     find /usr -name "command" 2>/dev/null                  │
│                                                             │
│  COMMON SOLUTIONS:                                          │
│  ─────────────────                                          │
│                                                             │
│  1. Install the missing tool                               │
│     # macOS                                                 │
│     brew install package-name                              │
│     # Ubuntu/Debian                                        │
│     sudo apt install package-name                          │
│     # Node.js tools                                        │
│     npm install -g package-name                            │
│                                                             │
│  2. Add to PATH                                            │
│     export PATH="/path/to/bin:$PATH"                       │
│     # Add to shell config for persistence                  │
│                                                             │
│  3. Use full path                                          │
│     /full/path/to/command arguments                        │
│                                                             │
│  4. Rehash shell (zsh)                                     │
│     rehash                                                  │
│                                                             │
│  5. Source shell config                                    │
│     source ~/.bashrc                                        │
│     source ~/.zshrc                                         │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

### Timeout Issues

```
┌────────────────────────────────────────────────────────────┐
│              Timeout Issues                                 │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  SYMPTOMS:                                                  │
│  ──────────                                                 │
│  Command timed out after 120000ms                          │
│  Operation cancelled due to timeout                        │
│                                                             │
│  CAUSES:                                                    │
│  ────────                                                   │
│  • Command takes longer than default 2-minute timeout      │
│  • Network operations with slow connections                │
│  • Large file operations                                   │
│  • CPU-intensive builds                                    │
│                                                             │
│  SOLUTIONS:                                                 │
│  ──────────                                                 │
│                                                             │
│  1. Use background execution                               │
│     Ask Claude to run command in background                │
│     "Run npm test in the background"                       │
│                                                             │
│  2. Break into smaller operations                          │
│     Instead of: npm install && npm run build && npm test   │
│     Run each separately or use background mode             │
│                                                             │
│  3. Optimize the command                                   │
│     # Use parallel execution                               │
│     npm test -- --parallel                                 │
│     # Limit scope                                          │
│     npm test -- --testPathPattern=specific                 │
│                                                             │
│  4. Check for hanging processes                            │
│     # Commands waiting for input will timeout              │
│     # Ensure non-interactive mode                          │
│     npm ci  # instead of npm install (no prompts)          │
│     git pull --no-edit                                     │
│                                                             │
│  5. Use streaming/partial output                           │
│     # For log monitoring, use tail with limit              │
│     tail -100 /var/log/app.log                            │
│     # Instead of watching entire file                      │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

### Platform-Specific Issues

```
┌────────────────────────────────────────────────────────────┐
│              Platform-Specific Troubleshooting              │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  WINDOWS ISSUES                                             │
│  ──────────────                                             │
│                                                             │
│  Line endings (CRLF vs LF):                                │
│  Problem: Scripts fail with ^M errors                      │
│  Solution: git config --global core.autocrlf true          │
│            Or use dos2unix tool                            │
│                                                             │
│  Path separators:                                           │
│  Problem: Paths with backslashes fail                      │
│  Solution: Use forward slashes or escape backslashes       │
│            "C:/Users/name/file" or "C:\\Users\\name\\file" │
│                                                             │
│  Case sensitivity:                                          │
│  Problem: File not found (case mismatch)                   │
│  Solution: Match exact case of files                       │
│                                                             │
│  PowerShell execution policy:                              │
│  Problem: Scripts cannot be run                            │
│  Solution: Set-ExecutionPolicy -ExecutionPolicy RemoteSigned│
│                                                             │
│  ─────────────────────────────────────────────────────────  │
│                                                             │
│  MACOS ISSUES                                               │
│  ────────────                                               │
│                                                             │
│  Apple Silicon (M1/M2) paths:                              │
│  Problem: Homebrew commands not found                      │
│  Solution: Add to PATH                                     │
│            export PATH="/opt/homebrew/bin:$PATH"           │
│                                                             │
│  Xcode command line tools:                                 │
│  Problem: git/make not found after macOS update           │
│  Solution: xcode-select --install                          │
│                                                             │
│  SIP restrictions:                                          │
│  Problem: Cannot modify system files                       │
│  Solution: Work in user directories or use Homebrew        │
│                                                             │
│  ─────────────────────────────────────────────────────────  │
│                                                             │
│  LINUX ISSUES                                               │
│  ────────────                                               │
│                                                             │
│  Missing build essentials:                                  │
│  Problem: npm install fails with node-gyp errors          │
│  Solution: sudo apt install build-essential                │
│                                                             │
│  Node version too old:                                     │
│  Problem: Syntax errors or unsupported features            │
│  Solution: Use nvm to install newer Node.js                │
│            nvm install 18 && nvm use 18                    │
│                                                             │
│  Docker socket permissions:                                │
│  Problem: Cannot connect to Docker daemon                  │
│  Solution: sudo usermod -aG docker $USER                   │
│            # Log out and back in                           │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

---

## 14. Quick Reference

### Common Commands Cheat Sheet

```
┌────────────────────────────────────────────────────────────┐
│              Common Commands Quick Reference                │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  GIT COMMANDS                                               │
│  ────────────                                               │
│  git status                  # Check status                │
│  git add .                   # Stage all changes           │
│  git commit -m "message"     # Commit                      │
│  git push                    # Push to remote              │
│  git pull                    # Pull from remote            │
│  git branch -a               # List all branches           │
│  git checkout -b name        # Create and switch branch    │
│  git log --oneline -10       # Recent commits              │
│  git diff                    # Show unstaged changes       │
│  git stash                   # Stash changes               │
│  git stash pop               # Apply stashed changes       │
│                                                             │
│  NPM/YARN COMMANDS                                          │
│  ─────────────────                                          │
│  npm install                 # Install dependencies        │
│  npm install package         # Add package                 │
│  npm install -D package      # Add dev dependency          │
│  npm uninstall package       # Remove package              │
│  npm run script              # Run npm script              │
│  npm test                    # Run tests                   │
│  npm run build               # Build project               │
│  npm outdated                # Check for updates           │
│  npm audit                   # Security check              │
│  npm ci                      # Clean install               │
│                                                             │
│  DOCKER COMMANDS                                            │
│  ───────────────                                            │
│  docker ps                   # List running containers     │
│  docker ps -a                # List all containers         │
│  docker images               # List images                 │
│  docker build -t name .      # Build image                 │
│  docker run -d -p 80:80 img  # Run container              │
│  docker logs container       # View logs                   │
│  docker exec -it cont bash   # Shell into container       │
│  docker stop container       # Stop container              │
│  docker rm container         # Remove container            │
│  docker-compose up -d        # Start compose services      │
│  docker-compose down         # Stop compose services       │
│                                                             │
│  FILE OPERATIONS                                            │
│  ───────────────                                            │
│  ls -la                      # List with details           │
│  pwd                         # Print working directory     │
│  mkdir -p dir/subdir         # Create directories          │
│  cp -r source dest           # Copy recursively            │
│  mv source dest              # Move/rename                 │
│  rm file                     # Delete file                 │
│  rm -r directory             # Delete directory            │
│  chmod +x script.sh          # Make executable             │
│  chown user:group file       # Change ownership            │
│                                                             │
│  PROCESS MANAGEMENT                                         │
│  ──────────────────                                         │
│  ps aux                      # List all processes          │
│  top                         # Interactive process view    │
│  kill PID                    # Terminate process           │
│  kill -9 PID                 # Force kill                  │
│  lsof -i :PORT               # Find process on port        │
│  pkill -f pattern            # Kill by pattern             │
│                                                             │
│  NETWORK                                                    │
│  ───────                                                    │
│  curl -X GET url             # HTTP GET                    │
│  curl -X POST -d data url    # HTTP POST                   │
│  wget url                    # Download file               │
│  netstat -tulpn              # List listening ports        │
│  ping host                   # Test connectivity           │
│  ssh user@host               # SSH connection              │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

### Permission Configuration Reference

```
┌────────────────────────────────────────────────────────────┐
│              Permission Configuration                       │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  CONFIG FILE LOCATIONS                                      │
│  ─────────────────────                                      │
│  Global:  ~/.claude/config.json                            │
│  Project: .claude/config.json (in project root)            │
│                                                             │
│  BASIC CONFIGURATION                                        │
│  ───────────────────                                        │
│  {                                                          │
│    "permissions": {                                         │
│      "mode": "ask",      // ask | auto | deny              │
│      "allowlist": [                                         │
│        "git *",                                            │
│        "npm *",                                            │
│        "docker ps"                                         │
│      ],                                                     │
│      "blocklist": [                                         │
│        "rm -rf *",                                         │
│        "sudo *"                                            │
│      ]                                                      │
│    }                                                        │
│  }                                                          │
│                                                             │
│  PERMISSION MODES                                           │
│  ────────────────                                           │
│  "ask"   - Prompt for dangerous commands (default)         │
│  "auto"  - Execute allowlisted without prompt              │
│  "deny"  - Block all bash execution                        │
│                                                             │
│  PATTERN SYNTAX                                             │
│  ──────────────                                             │
│  *         Matches anything                                │
│  "git *"   Matches all git commands                        │
│  "npm run *" Matches all npm run scripts                   │
│                                                             │
│  EXAMPLE CONFIGURATIONS                                     │
│  ──────────────────────                                     │
│                                                             │
│  # Safe mode (maximum restrictions)                        │
│  {                                                          │
│    "permissions": {                                         │
│      "mode": "ask",                                        │
│      "allowlist": ["git status", "ls", "pwd"],             │
│      "blocklist": ["rm *", "sudo *", "chmod *"]            │
│    }                                                        │
│  }                                                          │
│                                                             │
│  # Development mode (convenient workflow)                  │
│  {                                                          │
│    "permissions": {                                         │
│      "mode": "auto",                                       │
│      "allowlist": [                                         │
│        "git *", "npm *", "yarn *",                         │
│        "docker *", "docker-compose *",                     │
│        "make *", "cargo *", "go *"                         │
│      ],                                                     │
│      "blocklist": [                                         │
│        "rm -rf /", "sudo rm *",                            │
│        "curl * | bash", "wget * | sh"                      │
│      ]                                                      │
│    }                                                        │
│  }                                                          │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

### Background Command Syntax

```
┌────────────────────────────────────────────────────────────┐
│              Background Commands Reference                  │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  STARTING BACKGROUND TASKS                                  │
│  ─────────────────────────                                  │
│  Ask Claude:                                                │
│  • "Run npm test in the background"                        │
│  • "Start the build process in background"                 │
│  • "Run this command in the background: [command]"         │
│                                                             │
│  Claude will:                                               │
│  1. Start command with run_in_background: true             │
│  2. Return a task ID (e.g., bg-12345)                      │
│  3. Continue the conversation                              │
│  4. Notify when complete                                   │
│                                                             │
│  CHECKING STATUS                                            │
│  ───────────────                                            │
│  Ask Claude:                                                │
│  • "What background tasks are running?"                    │
│  • "Check status of bg-12345"                              │
│  • "Is the build done yet?"                                │
│                                                             │
│  RETRIEVING OUTPUT                                          │
│  ─────────────────                                          │
│  Ask Claude:                                                │
│  • "Show me the output from bg-12345"                      │
│  • "What was the result of the background test?"           │
│  • "Did the build succeed?"                                │
│                                                             │
│  STOPPING BACKGROUND TASKS                                  │
│  ─────────────────────────                                  │
│  Ask Claude:                                                │
│  • "Stop the background build"                             │
│  • "Cancel bg-12345"                                       │
│  • "Kill the running test"                                 │
│                                                             │
│  BEST PRACTICES                                             │
│  ──────────────                                             │
│  ✓ Use for commands > 2 minutes                            │
│  ✓ Use for builds, tests, deployments                      │
│  ✓ Continue with other tasks while waiting                 │
│  ✗ Don't use for quick commands (adds overhead)            │
│  ✗ Don't use for interactive commands                      │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

### Emergency Commands

```
┌────────────────────────────────────────────────────────────┐
│              Emergency & Recovery Commands                  │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  KILL RUNAWAY PROCESSES                                     │
│  ──────────────────────                                     │
│  # Find process by name                                    │
│  pgrep -f "process-name"                                   │
│                                                             │
│  # Kill by name                                            │
│  pkill -f "process-name"                                   │
│                                                             │
│  # Force kill by PID                                       │
│  kill -9 PID                                               │
│                                                             │
│  # Kill process on specific port                           │
│  lsof -ti:3000 | xargs kill -9                            │
│                                                             │
│  GIT RECOVERY                                               │
│  ────────────                                               │
│  # Undo last commit (keep changes)                         │
│  git reset --soft HEAD~1                                   │
│                                                             │
│  # Discard all local changes                               │
│  git checkout -- .                                         │
│                                                             │
│  # Recover deleted branch                                  │
│  git reflog                                                │
│  git checkout -b branch-name SHA                           │
│                                                             │
│  # Abort merge/rebase                                      │
│  git merge --abort                                         │
│  git rebase --abort                                        │
│                                                             │
│  DOCKER CLEANUP                                             │
│  ──────────────                                             │
│  # Stop all containers                                     │
│  docker stop $(docker ps -q)                               │
│                                                             │
│  # Remove all stopped containers                           │
│  docker container prune -f                                 │
│                                                             │
│  # Remove unused images                                    │
│  docker image prune -a -f                                  │
│                                                             │
│  NPM RECOVERY                                               │
│  ────────────                                               │
│  # Clear npm cache                                         │
│  npm cache clean --force                                   │
│                                                             │
│  # Remove node_modules and reinstall                       │
│  rm -rf node_modules package-lock.json && npm install      │
│                                                             │
│  # Fix npm permissions (Linux/macOS)                       │
│  sudo chown -R $(whoami) ~/.npm                            │
│                                                             │
│  DISK SPACE                                                │
│  ──────────                                                 │
│  # Check disk usage                                        │
│  df -h                                                      │
│                                                             │
│  # Find large files                                        │
│  du -sh * | sort -hr | head -20                           │
│                                                             │
│  # Find large directories                                  │
│  du -h --max-depth=1 | sort -hr                           │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

---

## Summary

Claude Code's terminal integration transforms how developers interact with their development environment. By combining the intelligence of Claude with direct command execution capabilities, you get:

1. **Seamless Workflows**: No more copy-pasting between AI and terminal
2. **Intelligent Execution**: Commands chosen based on your intent
3. **Safety First**: Multiple layers of protection against dangerous operations
4. **Cross-Platform**: Works on macOS, Linux, and Windows
5. **Flexible Permissions**: Configure trust levels for your workflow

Remember:
- Start with ask mode until you're comfortable
- Use the appropriate tool (Read, Write, Grep) when possible
- Leverage background execution for long-running tasks
- Configure allowlists for frequently used commands
- Check the audit log to review command history

The terminal is where developers live, and Claude Code makes it smarter.

---

*This tutorial is part of the Claude Code documentation series. For more information, visit the official Claude Code documentation.*
