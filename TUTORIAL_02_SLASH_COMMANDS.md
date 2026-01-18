# Tutorial 02: Claude Code Slash Commands

A comprehensive guide to mastering slash commands in Claude Code - from built-in essentials to creating your own custom workflows.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Installation & Setup](#2-installation--setup)
3. [Complete List of Built-in Commands](#3-complete-list-of-built-in-commands)
4. [Hand-Holding User Journey](#4-hand-holding-user-journey)
5. [Real-World Examples](#5-real-world-examples)
6. [Creating Custom Slash Commands](#6-creating-custom-slash-commands)
7. [Pros and Cons](#7-pros-and-cons)
8. [When to Use Slash Commands](#8-when-to-use-slash-commands)
9. [Comparison with Similar Features](#9-comparison-with-similar-features)
10. [Quick Reference](#10-quick-reference)

---

## 1. Introduction

### What Are Slash Commands in Claude Code?

Slash commands are special instructions that begin with a forward slash (`/`) and provide shortcuts for common operations in Claude Code. Instead of typing lengthy requests or navigating menus, you can simply type a command like `/help` or `/clear` to perform specific actions instantly.

Think of slash commands as keyboard shortcuts for conversations - they let you quickly execute predefined actions without explaining what you want in natural language.

**Example:**
```
# Instead of typing:
"Please clear the current conversation context and start fresh"

# You simply type:
/clear
```

### Built-in vs Custom Slash Commands

Claude Code provides two categories of slash commands:

| Type | Description | Examples |
|------|-------------|----------|
| **Built-in Commands** | Pre-installed commands that come with Claude Code | `/help`, `/clear`, `/init`, `/doctor` |
| **Custom Commands** | User-defined commands stored in markdown files | `/commit`, `/review-pr`, `/deploy` |

**Built-in commands** handle core functionality like configuration, authentication, and session management. They work immediately without any setup.

**Custom commands** let you create reusable workflows specific to your projects or team. They can include prompts, instructions, and even call other tools.

### How They Differ from Regular Prompts

| Aspect | Slash Commands | Regular Prompts |
|--------|---------------|-----------------|
| **Structure** | Predefined, consistent behavior | Free-form natural language |
| **Speed** | Instant execution | Requires Claude to interpret |
| **Repeatability** | Always produces same action | May vary in interpretation |
| **Customization** | Fixed behavior (built-in) or template-based (custom) | Fully flexible |
| **Use Case** | Frequent, standardized operations | Unique or complex requests |

**Regular prompt:**
```
Can you help me understand what commands are available and how to use them?
```

**Slash command:**
```
/help
```

Both achieve similar goals, but slash commands are faster and more predictable for operations you perform frequently.

---

## 2. Installation & Setup

### No Installation Needed for Built-in Commands

Built-in slash commands work immediately when you install Claude Code. There is no additional setup required.

```bash
# After installing Claude Code, all built-in commands are available
claude

# Inside Claude Code, just type:
/help
```

### How to Create Custom Slash Commands

Custom slash commands are defined as markdown files in specific directories. Here is how to create one:

**Step 1: Create the commands directory**

```bash
# For project-specific commands
mkdir -p .claude/commands

# For personal commands (available in all projects)
# On macOS/Linux:
mkdir -p ~/.claude/commands

# On Windows:
mkdir %USERPROFILE%\.claude\commands
```

**Step 2: Create a command file**

Each command is a markdown file. The filename (without `.md`) becomes the command name.

```bash
# Create a commit command
# File: .claude/commands/commit.md
```

**Step 3: Write the command content**

```markdown
# Commit Changes

Review the staged changes and create a conventional commit message.

1. Run `git diff --staged` to see what will be committed
2. Analyze the changes and determine the commit type (feat, fix, docs, etc.)
3. Generate a commit message following conventional commits format
4. Execute the commit with the generated message
```

**Step 4: Use the command**

```
/commit
```

### Configuration Locations

Custom commands can be stored in three locations, listed in order of precedence:

| Location | Scope | Path |
|----------|-------|------|
| **Project Commands** | Current project only | `.claude/commands/` |
| **User Commands** | All projects for current user | `~/.claude/commands/` |
| **Team Commands** | Shared via version control | `.claude/commands/` (committed to repo) |

**Directory structure example:**

```
your-project/
├── .claude/
│   └── commands/
│       ├── commit.md          # /commit
│       ├── review-pr.md       # /review-pr
│       └── deploy/
│           └── staging.md     # /deploy:staging
└── src/
    └── ...

~/.claude/
└── commands/
    ├── daily-standup.md       # /daily-standup (personal)
    └── timetrack.md           # /timetrack (personal)
```

---

## 3. Complete List of Built-in Commands

### /help - Getting Help

**Purpose:** Display available commands and usage information.

**Syntax:**
```
/help
/help [command-name]
```

**Examples:**
```
# Show all available commands
/help

# Get help for a specific command
/help init
/help config
```

**Use Cases:**
- Discovering available commands when starting with Claude Code
- Learning the syntax for unfamiliar commands
- Quick reference during coding sessions

**Expected Output:**
```
Available commands:
  /clear          Clear conversation context
  /compact        Compact conversation to save tokens
  /config         View or modify configuration
  /cost           Show token usage and costs
  /doctor         Run diagnostics
  /help           Show this help message
  /init           Initialize a new project
  /login          Authenticate with Anthropic
  /logout         Sign out of current session
  /memory         Manage conversation memory
  /model          Select or view current model
  /permissions    Manage tool permissions
  /review         Review code changes
  /terminal-setup Configure terminal integration

Type /help <command> for detailed information about a specific command.
```

---

### /clear - Clearing Context

**Purpose:** Clear the current conversation context and start fresh.

**Syntax:**
```
/clear
```

**Examples:**
```
# Clear all context
/clear
```

**Use Cases:**
- Starting a new task unrelated to previous conversation
- Resetting after confusion or errors
- Freeing up context window for large files
- Switching between different areas of a codebase

**What It Does:**
- Removes all previous messages from the conversation
- Clears any files or code Claude was tracking
- Does NOT affect project configuration or memory files
- Does NOT clear custom instructions in CLAUDE.md

**Expected Output:**
```
Context cleared. Starting fresh conversation.
```

---

### /compact - Compacting Conversation

**Purpose:** Compress the current conversation to use fewer tokens while preserving essential context.

**Syntax:**
```
/compact
/compact [focus-area]
```

**Examples:**
```
# Compact entire conversation
/compact

# Compact but preserve focus on specific topic
/compact authentication-flow
```

**Use Cases:**
- Long coding sessions approaching context limits
- Preserving important decisions while reducing noise
- Before working on a complex feature that needs context space
- When Claude mentions context is getting full

**What It Does:**
- Summarizes previous conversation turns
- Preserves key decisions, file changes, and code context
- Reduces token usage significantly (often 50-80% reduction)
- Maintains understanding of project state

**Expected Output:**
```
Conversation compacted.
Previous: 45,230 tokens
Current: 12,450 tokens
Preserved: Key decisions, file changes, and current task context
```

---

### /config - Configuration

**Purpose:** View or modify Claude Code configuration settings.

**Syntax:**
```
/config
/config [setting] [value]
/config --list
```

**Examples:**
```
# View all current settings
/config

# View specific setting
/config model

# Change a setting
/config model claude-sonnet-4-20250514

# List all available settings
/config --list
```

**Use Cases:**
- Checking current configuration
- Switching models for different tasks
- Adjusting behavior settings
- Troubleshooting unexpected behavior

**Common Settings:**

| Setting | Description | Values |
|---------|-------------|--------|
| `model` | Active Claude model | `claude-sonnet-4-20250514`, `claude-opus-4-5-20250501` |
| `theme` | Color theme | `dark`, `light`, `auto` |
| `verbose` | Output verbosity | `true`, `false` |

---

### /cost - Checking Costs

**Purpose:** Display token usage and estimated costs for the current session.

**Syntax:**
```
/cost
/cost --session
/cost --today
/cost --month
```

**Examples:**
```
# Show current session costs
/cost

# Show today's usage
/cost --today

# Show monthly usage
/cost --month
```

**Use Cases:**
- Monitoring API usage during development
- Budget tracking for teams
- Identifying expensive operations
- Optimizing workflows for cost efficiency

**Expected Output:**
```
Current Session Usage:
  Input tokens:  23,456
  Output tokens: 8,234
  Total tokens:  31,690

Estimated Cost: $0.47

Today's Total:
  Sessions: 5
  Total tokens: 156,789
  Estimated cost: $2.34
```

---

### /doctor - Diagnostics

**Purpose:** Run diagnostics to identify and troubleshoot issues with Claude Code.

**Syntax:**
```
/doctor
/doctor --verbose
/doctor --fix
```

**Examples:**
```
# Run basic diagnostics
/doctor

# Run detailed diagnostics
/doctor --verbose

# Attempt automatic fixes
/doctor --fix
```

**Use Cases:**
- Troubleshooting when Claude Code behaves unexpectedly
- Verifying installation after updates
- Checking authentication status
- Diagnosing permission issues

**What It Checks:**
- Authentication status and token validity
- Configuration file syntax
- Permission settings
- Network connectivity
- Tool availability
- Memory file integrity

**Expected Output:**
```
Running Claude Code diagnostics...

[OK] Authentication: Valid token found
[OK] Configuration: All settings valid
[OK] Permissions: Standard permissions active
[OK] Network: API endpoint reachable
[OK] Tools: All built-in tools available
[WARN] Memory: Large memory file detected (consider cleanup)

Diagnostics complete: 5 passed, 1 warning, 0 errors
```

---

### /init - Project Initialization

**Purpose:** Initialize Claude Code configuration for a new project.

**Syntax:**
```
/init
/init --minimal
/init --template [template-name]
```

**Examples:**
```
# Interactive initialization
/init

# Minimal setup (just essential files)
/init --minimal

# Use a specific template
/init --template typescript-node
```

**Use Cases:**
- Setting up Claude Code for a new project
- Creating CLAUDE.md with project context
- Establishing coding conventions
- Configuring project-specific commands

**What It Creates:**
```
your-project/
├── CLAUDE.md              # Project context and instructions
├── .claude/
│   ├── settings.json      # Project-specific settings
│   └── commands/          # Custom slash commands directory
└── .claudeignore          # Files to exclude from context
```

**Expected Output:**
```
Initializing Claude Code for your project...

Created:
  - CLAUDE.md (project context file)
  - .claude/settings.json (configuration)
  - .claude/commands/ (custom commands directory)
  - .claudeignore (exclusion patterns)

Next steps:
1. Edit CLAUDE.md to describe your project
2. Add custom commands to .claude/commands/
3. Configure .claudeignore for large/sensitive files

Initialization complete!
```

---

### /login, /logout - Authentication

**Purpose:** Manage authentication with Anthropic services.

**Syntax:**
```
/login
/login --api-key
/logout
/logout --all
```

**Examples:**
```
# Interactive login
/login

# Login with API key
/login --api-key

# Logout current session
/logout

# Logout all sessions/devices
/logout --all
```

**Use Cases:**
- Initial setup after installation
- Switching between accounts
- Refreshing expired tokens
- Securing shared machines

**Login Flow:**
```
/login

Opening browser for authentication...
Waiting for authorization...

Successfully authenticated!
Account: developer@example.com
Organization: Acme Corp
```

---

### /memory - Memory Management

**Purpose:** View and manage Claude Code's persistent memory for your project.

**Syntax:**
```
/memory
/memory show
/memory add [content]
/memory clear
/memory edit
```

**Examples:**
```
# Show current memory
/memory

# Add information to memory
/memory add "The API uses JWT tokens for authentication"

# Clear all memory
/memory clear

# Open memory for editing
/memory edit
```

**Use Cases:**
- Storing project-specific context that persists across sessions
- Recording architectural decisions
- Saving frequently needed information
- Maintaining project conventions

**What Memory Contains:**
- Project conventions you have mentioned
- Important decisions made during conversations
- Custom instructions for Claude
- Frequently referenced information

**Expected Output:**
```
Current Memory (CLAUDE.md):

# Project: E-commerce API

## Architecture
- Microservices with Node.js
- PostgreSQL database
- Redis for caching

## Conventions
- Use camelCase for variables
- Prefix interfaces with I
- All API responses use standard envelope format

## Important Notes
- JWT tokens expire after 24 hours
- Rate limiting: 100 requests/minute
```

---

### /model - Model Selection

**Purpose:** View or change the active Claude model.

**Syntax:**
```
/model
/model [model-name]
/model --list
```

**Examples:**
```
# Show current model
/model

# Switch to a different model
/model claude-opus-4-5-20250501

# List available models
/model --list
```

**Use Cases:**
- Switching to a more powerful model for complex tasks
- Using a faster model for simple operations
- Checking which model is currently active

**Available Models:**

| Model | Best For |
|-------|----------|
| `claude-sonnet-4-20250514` | Balanced speed and capability |
| `claude-opus-4-5-20250501` | Complex reasoning, large codebases |

**Expected Output:**
```
Current model: claude-sonnet-4-20250514

Available models:
  claude-sonnet-4-20250514    (current)
  claude-opus-4-5-20250501

Use /model <name> to switch models.
```

---

### /permissions - Permission Management

**Purpose:** View and manage tool permissions for Claude Code.

**Syntax:**
```
/permissions
/permissions list
/permissions grant [tool]
/permissions revoke [tool]
/permissions reset
```

**Examples:**
```
# View current permissions
/permissions

# Grant file write permission
/permissions grant file-write

# Revoke bash execution
/permissions revoke bash

# Reset to defaults
/permissions reset
```

**Use Cases:**
- Controlling what actions Claude can perform
- Increasing security for sensitive projects
- Enabling additional capabilities when needed
- Auditing current permission state

**Permission Categories:**

| Permission | Description | Default |
|------------|-------------|---------|
| `file-read` | Read files from disk | Granted |
| `file-write` | Write/modify files | Ask |
| `bash` | Execute shell commands | Ask |
| `web-fetch` | Access web URLs | Granted |
| `mcp-tools` | Use MCP server tools | Ask |

**Expected Output:**
```
Current Permissions:

Tool Permissions:
  [GRANTED] file-read     - Read files from disk
  [ASK]     file-write    - Write/modify files
  [ASK]     bash          - Execute shell commands
  [GRANTED] web-fetch     - Access web URLs
  [ASK]     mcp-tools     - Use MCP server tools

Session Grants:
  bash: Granted for this session

Use /permissions grant/revoke <tool> to modify.
```

---

### /review - Code Review

**Purpose:** Initiate an AI-powered code review of changes.

**Syntax:**
```
/review
/review [file-or-directory]
/review --staged
/review --branch [branch-name]
```

**Examples:**
```
# Review all uncommitted changes
/review

# Review specific file
/review src/auth/login.ts

# Review staged changes only
/review --staged

# Review changes compared to branch
/review --branch main
```

**Use Cases:**
- Pre-commit code review
- Learning from suggested improvements
- Catching bugs before they reach production
- Enforcing code quality standards

**What It Reviews:**
- Code correctness and logic errors
- Security vulnerabilities
- Performance issues
- Style and convention adherence
- Documentation completeness

**Expected Output:**
```
Reviewing changes...

Files analyzed: 3
  - src/auth/login.ts
  - src/auth/validate.ts
  - src/utils/token.ts

Findings:

[HIGH] src/auth/login.ts:45
  Potential SQL injection vulnerability
  Consider using parameterized queries

[MEDIUM] src/auth/validate.ts:23
  Missing null check before accessing user.email

[LOW] src/utils/token.ts:12
  Magic number 3600 should be a named constant

Summary: 1 high, 1 medium, 1 low severity issues found
```

---

### /terminal-setup - Terminal Configuration

**Purpose:** Configure terminal integration and shell settings.

**Syntax:**
```
/terminal-setup
/terminal-setup --shell [shell-name]
/terminal-setup --reset
```

**Examples:**
```
# Interactive terminal setup
/terminal-setup

# Configure for specific shell
/terminal-setup --shell zsh

# Reset terminal configuration
/terminal-setup --reset
```

**Use Cases:**
- Initial setup after installation
- Fixing terminal display issues
- Configuring shell integration
- Enabling special features

**What It Configures:**
- Shell integration scripts
- Color and formatting support
- Keyboard shortcuts
- Output handling

**Expected Output:**
```
Terminal Setup

Detected shell: zsh
Terminal: iTerm2

Configuration:
  [OK] Shell integration installed
  [OK] Color support enabled
  [OK] Unicode support verified
  [OK] Keyboard shortcuts configured

Your terminal is configured for Claude Code.
```

---

## 4. Hand-Holding User Journey

Let us walk through your first experience with slash commands, step by step.

### Step 1: Discover Available Commands (/help)

**What you will do:** Learn what commands are available.

**Keystrokes:**
```
1. Open your terminal
2. Type: claude
3. Press Enter to start Claude Code
4. Type: /help
5. Press Enter
```

**What you will see:**
```
Welcome to Claude Code!

> /help

Available commands:
  /clear          Clear conversation context
  /compact        Compact conversation to save tokens
  /config         View or modify configuration
  /cost           Show token usage and costs
  /doctor         Run diagnostics
  /help           Show this help message
  /init           Initialize a new project
  ...

Type /help <command> for more details.
```

**What just happened:** You discovered all available built-in commands. Each one has a specific purpose explained in the short description.

---

### Step 2: Use Your First Command

**What you will do:** Check your current model and session costs.

**Keystrokes:**
```
1. Type: /model
2. Press Enter
3. Type: /cost
4. Press Enter
```

**What you will see:**
```
> /model
Current model: claude-sonnet-4-20250514

> /cost
Current Session Usage:
  Input tokens:  1,234
  Output tokens: 567
  Total tokens:  1,801
  Estimated Cost: $0.03
```

**What just happened:** You used two different commands to check your configuration and usage. Notice how quick and direct the responses are compared to asking "what model am I using?" in natural language.

---

### Step 3: Create a Custom Command

**What you will do:** Create a simple custom command for generating commit messages.

**Keystrokes:**
```
1. In your terminal (outside Claude Code), navigate to your project
2. Create the commands directory:
   mkdir -p .claude/commands

3. Create the command file:
   # On macOS/Linux:
   touch .claude/commands/commit.md

   # On Windows:
   type nul > .claude\commands\commit.md

4. Open the file in your editor and add this content:
```

**File content (.claude/commands/commit.md):**
```markdown
# Generate Commit Message

Analyze the staged changes and create a conventional commit message.

## Instructions

1. Run `git diff --staged` to see what changes are staged
2. Analyze the changes to understand what was modified
3. Determine the appropriate commit type:
   - feat: New feature
   - fix: Bug fix
   - docs: Documentation only
   - style: Formatting, no code change
   - refactor: Code restructuring
   - test: Adding tests
   - chore: Maintenance tasks
4. Generate a commit message in this format:
   ```
   type(scope): brief description

   - Bullet points explaining changes
   - Keep each line under 72 characters
   ```
5. Show me the suggested commit message before committing
```

**Now use it:**
```
1. Start Claude Code: claude
2. Type: /commit
3. Press Enter
```

**What you will see:**
```
> /commit

Running git diff --staged...

Analyzing changes in:
  - src/auth/login.ts (modified)
  - src/auth/types.ts (new file)

Suggested commit message:

feat(auth): add type definitions for authentication module

- Create AuthUser and AuthToken interfaces
- Add type safety to login function parameters
- Export types for use in other modules

Would you like me to commit with this message?
```

---

### Step 4: Share Commands with Your Team

**What you will do:** Commit your custom commands to version control.

**Keystrokes:**
```
1. Stage the commands directory:
   git add .claude/commands/

2. Commit:
   git commit -m "chore: add custom Claude Code commands"

3. Push to your repository:
   git push origin main
```

**Team usage:**
```
# When teammates pull the repository, they automatically get the commands:
git pull origin main

# Now they can use your custom command:
claude
> /commit
```

**What just happened:** Your custom commands are now shared with everyone who clones or pulls your repository. The `.claude/commands/` directory is version controlled, so commands stay synchronized across the team.

---

## 5. Real-World Examples

### Example 1: Quick Project Setup with /init

**Scenario:** You are starting a new Node.js API project and want to set up Claude Code properly.

```bash
# Create and enter your project
mkdir my-api && cd my-api
npm init -y

# Start Claude Code
claude
```

```
> /init

Initializing Claude Code for your project...

I'll analyze your project structure to create appropriate configuration.

Detected:
  - Node.js project (package.json found)
  - No existing Claude configuration

Creating configuration files...

Created:
  - CLAUDE.md
  - .claude/settings.json
  - .claude/commands/
  - .claudeignore

I've pre-populated CLAUDE.md with Node.js conventions.
Would you like me to open it for customization?

> yes

Opening CLAUDE.md...
```

**Resulting CLAUDE.md:**
```markdown
# My API Project

## Overview
Node.js API project.

## Tech Stack
- Node.js
- npm for package management

## Conventions
- Use ES modules (import/export)
- Follow StandardJS style
- Use async/await for asynchronous code

## Important Directories
- src/ - Source code
- tests/ - Test files
- node_modules/ - Dependencies (excluded from context)

## Commands
- npm start - Start the application
- npm test - Run tests
```

---

### Example 2: Cost Monitoring Workflow with /cost

**Scenario:** You are working on a large refactoring task and want to monitor your API usage.

```
> /cost

Current Session:
  Input tokens:  0
  Output tokens: 0
  Estimated Cost: $0.00

Starting fresh! Let me know what you'd like to work on.

> Please refactor the authentication module to use JWT tokens instead of sessions.
  Include proper error handling and logging.

[Claude performs the refactoring work...]

> /cost

Current Session:
  Input tokens:  45,678
  Output tokens: 12,345
  Total tokens:  58,023
  Estimated Cost: $0.87

This session's breakdown:
  - File reading: 23,456 tokens
  - Code analysis: 12,222 tokens
  - Code generation: 22,345 tokens

> /compact

Compacting conversation to reduce token usage...

Preserved:
  - Refactoring decisions made
  - Files modified (auth.ts, middleware.ts, types.ts)
  - Current task state

Before: 58,023 tokens
After: 15,234 tokens
Saved: 42,789 tokens (73% reduction)

> /cost

Current Session (after compact):
  Input tokens:  15,234
  Output tokens: 0
  Estimated Cost: $0.23

Ready to continue. Context preserved for auth refactoring.
```

---

### Example 3: Debugging with /doctor

**Scenario:** Claude Code is not responding correctly to your requests, and you need to diagnose the issue.

```
> /doctor

Running Claude Code diagnostics...

Checking authentication...
[OK] API key valid
[OK] Token not expired

Checking configuration...
[OK] Global config valid
[WARN] Project config has deprecated setting: "legacy_mode"
       Recommendation: Remove this setting from .claude/settings.json

Checking permissions...
[OK] File read: granted
[OK] File write: ask mode
[ERROR] Bash execution: blocked
        This may prevent running tests and build commands

Checking network...
[OK] API endpoint reachable (latency: 45ms)
[OK] No proxy issues detected

Checking tools...
[OK] All built-in tools available
[WARN] MCP server "database" not responding
       Last successful connection: 2 hours ago

Summary:
  Passed: 7
  Warnings: 2
  Errors: 1

Would you like me to attempt automatic fixes? (/doctor --fix)

> /doctor --fix

Attempting fixes...

[FIXED] Removed deprecated "legacy_mode" setting
[FIXED] Reset bash permission to "ask" mode
[SKIPPED] MCP server issue requires manual intervention

Fixes applied: 2
Manual action needed: 1

For the MCP server issue, try:
  1. Check if the database MCP server is running
  2. Restart with: claude mcp restart database
```

---

### Example 4: Context Management with /clear and /compact

**Scenario:** You have been working on multiple unrelated tasks and need to manage your context effectively.

```
> [After a long session working on database migrations...]

The migration is complete. All tables have been updated.

> /cost

Current Session:
  Total tokens: 127,456
  Estimated Cost: $1.91

Warning: Approaching context limit (128,000 tokens)

> /compact

Compacting conversation...

Current context summary:
  - Completed: Database migration (users, orders, products tables)
  - Files modified: 12 migration files
  - Current state: All migrations applied successfully

Before: 127,456 tokens
After: 8,234 tokens
Saved: 119,222 tokens (93% reduction)

> Now I need to work on the frontend components. This is unrelated to the database work.

> /clear

Context cleared.

Note: If you need to reference the previous database work,
it's been logged to .claude/session-logs/2024-01-15-migrations.md

> /cost

Current Session:
  Total tokens: 0
  Estimated Cost: $0.00

Fresh start! What would you like to work on?

> Let's build a React component for user authentication.

Starting fresh with the authentication component work...
```

---

### Example 5: Custom Commit Workflow Command

**Scenario:** Your team has specific commit message conventions and you want to enforce them automatically.

**First, create the command (.claude/commands/commit.md):**

```markdown
# Team Commit Workflow

Generate a commit following our team's conventions and run pre-commit checks.

## Our Commit Format

```
[TICKET-XXX] type: description

Body explaining the why, not the what.
Maximum 72 characters per line.

Refs: #issue-number
```

## Types We Use
- feat: New functionality
- fix: Bug repair
- refactor: Code improvement without behavior change
- docs: Documentation updates
- test: Test additions or fixes
- chore: Build, CI, or tooling changes

## Workflow

1. First, run `git status` to see what's changed
2. Run `git diff --staged` to review staged changes
3. Ask me for the JIRA ticket number if not obvious from branch name
4. Generate commit message following our format
5. Run `npm run lint` to check for issues
6. Run `npm test -- --related` to run related tests
7. If all passes, show the commit command but wait for my approval
8. After approval, execute the commit

## Important
- Never commit directly to main
- Always include ticket reference
- Run tests before committing
```

**Using the command:**

```
> /commit

Running git status...
  Modified: src/auth/login.ts
  Modified: src/auth/validate.ts
  New file: src/auth/two-factor.ts

Running git diff --staged...
[Shows the diff]

I see you're working on two-factor authentication.
What's the JIRA ticket number for this work?

> PROJ-1234

Generating commit message...

Proposed commit:
```
[PROJ-1234] feat: add two-factor authentication support

Implement TOTP-based 2FA for user accounts:
- Add generateTOTP and verifyTOTP functions
- Create 2FA validation middleware
- Update login flow to check 2FA status

Refs: #89
```

Running pre-commit checks...
  [OK] npm run lint - No issues
  [OK] npm test --related - 3 tests passed

Ready to commit. Proceed? (yes/no)

> yes

Committed: a1b2c3d [PROJ-1234] feat: add two-factor authentication support
```

---

## 6. Creating Custom Slash Commands

### File Location and Format

Custom commands are markdown files stored in specific directories:

```
Project commands:     .claude/commands/*.md
User commands:        ~/.claude/commands/*.md
```

**Naming conventions:**
- Filename becomes command name: `commit.md` creates `/commit`
- Use hyphens for multi-word: `review-pr.md` creates `/review-pr`
- Subdirectories use colons: `deploy/staging.md` creates `/deploy:staging`

### Command Structure

A well-structured command file includes:

```markdown
# Command Title

Brief description of what this command does.

## Context
Background information Claude needs to understand the task.

## Instructions
Step-by-step instructions for Claude to follow:
1. First step
2. Second step
3. Third step

## Examples
Show examples of expected input/output if helpful.

## Constraints
Any limitations or rules to follow.
```

**Complete example (.claude/commands/api-endpoint.md):**

```markdown
# Create API Endpoint

Generate a new REST API endpoint following our project conventions.

## Context
We use Express.js with TypeScript. All endpoints follow RESTful conventions
and include proper error handling, validation, and documentation.

## Arguments
When invoked, the user should provide:
- Resource name (e.g., "users", "products")
- HTTP method (GET, POST, PUT, DELETE)
- Any special requirements

## Instructions

1. Ask for the resource name and HTTP method if not provided
2. Create the route file in `src/routes/[resource].ts`
3. Create the controller in `src/controllers/[resource]Controller.ts`
4. Create request/response types in `src/types/[resource].ts`
5. Add validation schemas in `src/validation/[resource].ts`
6. Register the route in `src/routes/index.ts`
7. Create a basic test file in `tests/routes/[resource].test.ts`

## File Templates

### Route Template
```typescript
import { Router } from 'express';
import { ResourceController } from '../controllers/resourceController';
import { validateRequest } from '../middleware/validation';
import { resourceSchema } from '../validation/resource';

const router = Router();
const controller = new ResourceController();

router.[method]('/', validateRequest(resourceSchema), controller.[method]);

export default router;
```

## Constraints
- Always include JSDoc comments
- Use async/await, never callbacks
- Include proper HTTP status codes
- Add request validation
```

### Arguments and Options

Commands can accept arguments in several ways:

**Positional arguments (in the command invocation):**
```
/api-endpoint users POST
```

**Interactive prompting (in the command file):**
```markdown
## Arguments Required

If the user doesn't provide these, ask for them:
1. Resource name - The name of the API resource
2. HTTP method - The HTTP verb to use
```

**Using $ARGUMENTS placeholder:**
```markdown
# Deploy Command

Deploy the application to the specified environment.

## Instructions
Deploy to: $ARGUMENTS

If $ARGUMENTS is empty, ask which environment:
- staging
- production
```

Usage:
```
/deploy staging
```

### Best Practices

**1. Be specific about steps:**
```markdown
# Good
1. Run `git status` to check for uncommitted changes
2. If there are changes, ask whether to stash them
3. Run `git fetch origin` to get latest changes

# Bad
1. Check git status and handle any issues
```

**2. Include error handling:**
```markdown
## Error Handling
- If tests fail, show the failures and ask how to proceed
- If the file already exists, ask before overwriting
- If required dependencies are missing, list them
```

**3. Provide context:**
```markdown
## Context
This project uses:
- React 18 with TypeScript
- Jest for testing
- ESLint with our custom config

Keep these in mind when generating code.
```

**4. Show examples:**
```markdown
## Example Output

A successful run should produce:
```
src/
  components/
    Button/
      Button.tsx        # Component file
      Button.test.tsx   # Test file
      Button.stories.tsx # Storybook file
      index.ts          # Barrel export
```
```

**5. Keep commands focused:**
```markdown
# Good: Single-purpose command
# create-component.md - Creates a React component

# Bad: Multi-purpose command
# do-stuff.md - Creates components, runs tests, deploys...
```

---

## 7. Pros and Cons

### Advantages of Slash Commands

| Advantage | Description |
|-----------|-------------|
| **Speed** | Instant execution without interpreting natural language |
| **Consistency** | Same command always produces same behavior |
| **Discoverability** | `/help` shows all available commands |
| **Shareability** | Custom commands can be version controlled |
| **Reduced Tokens** | Shorter than equivalent natural language requests |
| **Standardization** | Team-wide consistency in workflows |
| **Error Reduction** | Predefined steps reduce mistakes |

**Speed comparison:**

```
# Slash command approach
/commit
Time: Immediate execution

# Natural language approach
"Please look at my staged changes, analyze them, and create a
conventional commit message following our team's format with
the ticket number from the branch name"
Time: Claude must parse and interpret request first
```

### Limitations

| Limitation | Description |
|------------|-------------|
| **Rigidity** | Built-in commands have fixed behavior |
| **Learning Curve** | Must memorize command names and syntax |
| **Limited Parameters** | Some commands have minimal customization |
| **Markdown Knowledge** | Creating custom commands requires markdown |
| **No Conditionals** | Custom commands cannot have if/else logic |
| **Maintenance** | Custom commands need updating as projects evolve |

### Memory Considerations

**Token usage patterns:**

| Scenario | Token Impact |
|----------|--------------|
| Using `/clear` | Resets to zero tokens |
| Using `/compact` | Typically reduces 50-80% |
| Custom command text | Loaded into context each use |
| Long custom commands | Can use significant tokens |

**Best practices for memory efficiency:**
- Keep custom commands under 500 words
- Use `/compact` before complex tasks
- Use `/clear` when switching contexts
- Monitor with `/cost` regularly

---

## 8. When to Use Slash Commands

### Ideal Scenarios

**Use slash commands when:**

1. **Performing standard operations**
   ```
   /init     # Starting a new project
   /clear    # Resetting context
   /cost     # Checking usage
   ```

2. **Following repeatable workflows**
   ```
   /commit   # Team's commit process
   /deploy   # Deployment checklist
   /review   # Code review process
   ```

3. **Quick configuration checks**
   ```
   /model    # What model am I using?
   /config   # What are my settings?
   /permissions  # What can Claude do?
   ```

4. **Troubleshooting**
   ```
   /doctor   # Something isn't working
   ```

### Slash Commands vs Typing Full Requests

| Situation | Better Choice | Why |
|-----------|---------------|-----|
| Check current model | `/model` | Faster, direct answer |
| Complex refactoring task | Natural language | Needs nuanced instructions |
| Start new project | `/init` | Standardized setup |
| Unique one-time task | Natural language | Not worth creating command |
| Team's PR review process | `/review-pr` (custom) | Consistency across team |
| Exploratory question | Natural language | Open-ended discussion |

### Decision Guide

**Ask yourself:**

```
Will I do this task more than 3 times?
  YES → Consider creating a custom command
  NO  → Use natural language

Is this a standard operation?
  YES → Check if a built-in command exists (/help)
  NO  → Use natural language

Does my team need consistency?
  YES → Create a shared custom command
  NO  → Individual preference

Am I troubleshooting Claude Code itself?
  YES → Use /doctor, /config, /permissions
  NO  → Describe the issue in natural language
```

---

## 9. Comparison with Similar Features

### Slash Commands vs Skills

| Aspect | Slash Commands | Skills |
|--------|----------------|--------|
| **Definition** | Markdown files with instructions | More complex registered capabilities |
| **Location** | `.claude/commands/` | Skill registry system |
| **Complexity** | Simple text-based prompts | Can include tools and integrations |
| **Creation** | Create markdown file | Use Skill tool to invoke |
| **Invocation** | `/command-name` | Via Skill tool or reference |
| **Best For** | Quick, text-based workflows | Complex multi-tool operations |

**Example comparison:**

```markdown
# Slash command (/commit)
Simple prompt that guides Claude through commit process

# Skill (commit)
May include:
- Git integration
- Ticket system lookups
- CI/CD triggers
- Team notification
```

### Slash Commands vs Hooks

| Aspect | Slash Commands | Hooks |
|--------|----------------|-------|
| **Trigger** | Manual (user types command) | Automatic (event-based) |
| **Control** | User initiates | System initiates |
| **Use Case** | On-demand actions | Pre/post operation tasks |
| **Definition** | Markdown instructions | Configuration + scripts |

**Example comparison:**

```
Slash command: /lint
  User types command to run linting

Hook: pre-commit
  Linting runs automatically before every commit
```

### Slash Commands vs Direct Prompts

| Aspect | Slash Commands | Direct Prompts |
|--------|----------------|----------------|
| **Format** | Structured, predefined | Free-form natural language |
| **Flexibility** | Fixed behavior | Unlimited flexibility |
| **Learning** | Must know command exists | Intuitive |
| **Repeatability** | Identical each time | May vary |
| **Speed** | Immediate | Requires interpretation |

**When to use each:**

```
Slash commands:
- "I do this exact thing every day" → /daily-standup
- "My team has a specific process" → /review-pr
- "I want consistent results" → /test-component

Direct prompts:
- "I've never done this before"
- "This is a unique situation"
- "I need to explain context"
- "I want to have a discussion"
```

---

## 10. Quick Reference

### Complete Command Cheat Sheet

| Command | Purpose | Common Usage |
|---------|---------|--------------|
| `/help` | Show available commands | `/help`, `/help init` |
| `/clear` | Clear conversation context | `/clear` |
| `/compact` | Compress conversation | `/compact` |
| `/config` | View/modify settings | `/config`, `/config model` |
| `/cost` | Show token usage | `/cost`, `/cost --today` |
| `/doctor` | Run diagnostics | `/doctor`, `/doctor --fix` |
| `/init` | Initialize project | `/init`, `/init --minimal` |
| `/login` | Authenticate | `/login` |
| `/logout` | Sign out | `/logout` |
| `/memory` | Manage memory | `/memory`, `/memory add` |
| `/model` | Select model | `/model`, `/model opus` |
| `/permissions` | Manage permissions | `/permissions` |
| `/review` | Code review | `/review`, `/review --staged` |
| `/terminal-setup` | Configure terminal | `/terminal-setup` |

### Common Workflows

**Starting a new project:**
```
claude
/init
[Configure CLAUDE.md]
/help
```

**Daily development session:**
```
/cost                    # Check starting point
/model                   # Verify model
[Do work]
/cost                    # Monitor usage
/compact                 # If approaching limits
```

**End of session:**
```
/cost --today            # Review daily usage
/commit                  # Custom: commit work
/logout                  # If on shared machine
```

**Troubleshooting:**
```
/doctor                  # Check for issues
/config                  # Review settings
/permissions             # Check access
```

**Context management:**
```
/compact                 # Reduce tokens, keep context
/clear                   # Fresh start, no context
/memory                  # View persistent memory
```

### Custom Command Quick Start

**1. Create directory:**
```bash
mkdir -p .claude/commands
```

**2. Create command file:**
```bash
# .claude/commands/quick-test.md
```

**3. Add content:**
```markdown
# Quick Test

Run tests for the current file or directory.

1. Identify the current file or ask which file to test
2. Find the corresponding test file
3. Run the test with `npm test -- [test-file]`
4. Report results clearly
```

**4. Use it:**
```
/quick-test
```

---

## Summary

Slash commands in Claude Code provide a powerful way to streamline your development workflow. Key takeaways:

1. **Built-in commands** handle essential operations like configuration, context management, and diagnostics
2. **Custom commands** let you create reusable workflows specific to your projects
3. **Use slash commands** for repetitive, standardized tasks where consistency matters
4. **Use natural language** for unique, complex, or exploratory requests
5. **Share commands** with your team via version control for consistent workflows
6. **Monitor usage** with `/cost` and manage context with `/clear` and `/compact`

Start with the built-in commands, create your first custom command, and gradually build a library of workflows that make your development faster and more consistent.

---

*Tutorial 02 of the Claude Code series. For more tutorials, check the other guides in this series.*
