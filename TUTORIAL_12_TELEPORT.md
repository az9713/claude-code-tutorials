# Tutorial 12: Teleport - Seamless CLI and Web Integration

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Core Concepts](#core-concepts)
4. [Teleporting FROM CLI to Web](#teleporting-from-cli-to-web)
5. [Teleporting TO CLI from Web](#teleporting-to-cli-from-web)
6. [Bidirectional Workflows](#bidirectional-workflows)
7. [Background Cloud Execution](#background-cloud-execution)
8. [Templates and Commands](#templates-and-commands)
9. [Real-World Examples](#real-world-examples)
10. [CLI vs Web Comparison](#cli-vs-web-comparison)
11. [When to Use Each Environment](#when-to-use-each-environment)
12. [Troubleshooting](#troubleshooting)
13. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)
14. [Best Practices](#best-practices)
15. [Advanced Patterns](#advanced-patterns)
16. [Summary](#summary)

---

## Overview

### What is Teleport?

Teleport is a revolutionary feature in Claude Code that enables seamless transfer of your
working session between the command-line interface (CLI) and the web interface at
claude.ai/code. Think of it as a "save state" for your AI-assisted development workflow
that you can resume on any device, anywhere.

### Why Teleport is Revolutionary

Traditional development workflows force you to choose: work locally with full file access
but be tethered to your machine, or work on the web with mobility but lose your local
context. Teleport eliminates this compromise entirely.

**Before Teleport:**
```
Developer starts complex refactor on laptop
    |
    v
Needs to leave for meeting
    |
    v
Options:
  - Abandon work (lose context)
  - Copy/paste conversation (tedious, loses state)
  - Stay at desk (not always possible)
```

**With Teleport:**
```
Developer starts complex refactor on laptop
    |
    v
Needs to leave for meeting
    |
    v
/teleport
    |
    v
Continue on mobile/tablet during commute
    |
    v
Return to desk, teleport back to CLI
    |
    v
Finish work with full local file access
```

### Key Benefits

1. **Continuity** - Never lose your conversation context or work progress
2. **Flexibility** - Work from anywhere on any device
3. **Efficiency** - No copy-pasting, no re-explaining, no lost context
4. **Collaboration** - Share sessions with team members instantly
5. **Cloud Power** - Offload long-running tasks to cloud execution

---

## Prerequisites

Before using Teleport, ensure you have the following set up:

### 1. Claude Code CLI Installation

```bash
# Install Claude Code CLI globally
npm install -g @anthropic-ai/claude-code

# Verify installation
claude --version
```

Expected output:
```
Claude Code CLI v1.x.x
```

### 2. Anthropic Account with claude.ai Access

You need an active Anthropic account with access to claude.ai/code:

1. Visit https://claude.ai
2. Sign up or log in with your existing account
3. Ensure you have access to the Claude Code feature (claude.ai/code)

### 3. CLI Authentication

The CLI must be authenticated with your Anthropic account:

```bash
# Authenticate CLI with your Anthropic account
claude auth login

# Follow the browser-based authentication flow
# A browser window will open for OAuth authentication
```

Verification:
```bash
# Check authentication status
claude auth status
```

Expected output:
```
Authenticated as: your.email@example.com
Account type: Pro/Team/Enterprise
Teleport: Enabled
```

### 4. Network Requirements

Teleport requires internet connectivity for:
- Initial session transfer
- Syncing conversation state
- Downloading/uploading context files

Ensure your firewall allows connections to:
- api.anthropic.com
- claude.ai
- cdn.anthropic.com

### 5. Browser Configuration

For automatic browser opening:

```bash
# Set your preferred browser (optional)
claude config set browser chrome  # or firefox, safari, edge

# Test browser integration
claude config test-browser
```

---

## Core Concepts

### Understanding the Teleport Architecture

Teleport works by creating a serialized snapshot of your current Claude Code session
and transferring it to Anthropic's cloud infrastructure. This snapshot includes:

```
┌─────────────────────────────────────────────────────────────┐
│                    Teleport Session State                    │
├─────────────────────────────────────────────────────────────┤
│  Conversation History                                        │
│  ├── All messages (user and assistant)                      │
│  ├── Tool calls and results                                 │
│  └── Reasoning and context                                  │
├─────────────────────────────────────────────────────────────┤
│  File Context                                                │
│  ├── Files read during session                              │
│  ├── Files modified (with diffs)                            │
│  └── Project structure snapshot                             │
├─────────────────────────────────────────────────────────────┤
│  Environment State                                           │
│  ├── Working directory                                      │
│  ├── Git branch and status                                  │
│  └── Active CLAUDE.md rules                                 │
├─────────────────────────────────────────────────────────────┤
│  Session Metadata                                            │
│  ├── Timestamp                                              │
│  ├── Session ID                                             │
│  └── Transfer direction (CLI→Web or Web→CLI)                │
└─────────────────────────────────────────────────────────────┘
```

### CLI-to-Web Transfer

When you teleport from CLI to web:

1. **Session Serialization** - Current conversation and context is packaged
2. **Secure Upload** - Encrypted transfer to Anthropic cloud
3. **Web Session Creation** - New session initialized at claude.ai/code
4. **Browser Launch** - Automatic opening (or URL provided)
5. **Seamless Continuation** - Pick up exactly where you left off

```
┌──────────────┐    Teleport    ┌──────────────┐
│   CLI        │ ────────────►  │   Web        │
│   Session    │                │   Session    │
├──────────────┤                ├──────────────┤
│ Local files  │   Encrypted    │ Cloud sandbox│
│ Local shell  │   Transfer     │ GVisor env   │
│ Full access  │                │ Web UI       │
└──────────────┘                └──────────────┘
```

### Session Continuity

Teleport maintains perfect session continuity:

- **Conversation Memory** - Claude remembers everything discussed
- **Task Progress** - Incomplete tasks are preserved
- **File State** - Modified files are tracked
- **Reasoning Context** - Claude's understanding of your goals persists

### Cloud Execution Environment

When working on claude.ai/code after teleport:

```
┌─────────────────────────────────────────┐
│         Cloud Execution Sandbox          │
├─────────────────────────────────────────┤
│  Runtime: GVisor (secure container)     │
│  File System: Isolated workspace        │
│  Network: Controlled egress             │
│  Resources: Managed compute allocation  │
├─────────────────────────────────────────┤
│  Capabilities:                          │
│  ✓ Run code in multiple languages       │
│  ✓ Install packages                     │
│  ✓ Create and modify files              │
│  ✓ Execute shell commands               │
│  ✓ Visual output (graphs, images)       │
└─────────────────────────────────────────┘
```

---

## Teleporting FROM CLI to Web

### The `/teleport` Command

The primary command for transferring your session to the web:

```bash
# Inside an active Claude Code session
> /teleport
```

Or use the shorthand:
```bash
> /tp
```

### Basic Teleport Workflow

**Step 1: Start a CLI session**
```bash
claude

# Begin working on your task
> Help me refactor this authentication module to use JWT tokens
```

**Step 2: Work progresses normally**
```
Claude: I'll help you refactor the authentication module. Let me first examine
the current implementation...

[Claude reads files, makes suggestions, you iterate]
```

**Step 3: Initiate teleport when needed**
```bash
> /teleport

Preparing teleport...
✓ Serializing conversation (47 messages)
✓ Packaging file context (12 files)
✓ Capturing environment state
✓ Encrypting session data

Teleport ready!
Opening browser to: https://claude.ai/code/session/abc123xyz

Press Enter to open browser, or copy the URL above.
```

**Step 4: Continue on the web**

Your browser opens to claude.ai/code with your full session loaded.
Continue working exactly where you left off.

### What Gets Transferred

| Component | Transferred | Notes |
|-----------|-------------|-------|
| Conversation history | Yes | All messages and context |
| Files read | Yes | Content snapshot |
| Files modified | Yes | Original + changes |
| Pending edits | Yes | Uncommitted changes |
| Git state | Yes | Branch, status, diffs |
| CLAUDE.md rules | Yes | Project-specific rules |
| Environment variables | No | Security consideration |
| Local secrets | No | Never transferred |
| Installed tools | Partial | Cloud has own toolset |

### Teleport with a Message

Add context for why you're teleporting:

```bash
> /teleport "Continuing on my tablet - need to review the API design"
```

This message appears at the top of your web session, helping you (or a colleague)
understand the context of the transfer.

### Teleport Options

```bash
# Teleport with specific options
> /teleport --no-browser    # Get URL only, don't open browser
> /teleport --minimal       # Transfer conversation only, no files
> /teleport --full          # Include all project files (larger transfer)
> /teleport --share         # Create shareable link for team member
```

### Automatic Browser Opening

By default, teleport opens your system's default browser. Configure this:

```bash
# In your shell, before starting claude
export CLAUDE_BROWSER=firefox

# Or configure permanently
claude config set browser firefox
```

Supported browsers:
- `chrome` / `google-chrome`
- `firefox`
- `safari` (macOS only)
- `edge`
- `brave`
- `default` (system default)

### Continuing Work on claude.ai/code

Once on the web interface:

1. **Review the Session Banner**
   ```
   ┌─────────────────────────────────────────────────────────┐
   │ Session teleported from CLI                             │
   │ Original working directory: /home/user/projects/myapp   │
   │ Message: "Continuing on my tablet..."                   │
   └─────────────────────────────────────────────────────────┘
   ```

2. **Access Teleported Files**
   - Files appear in the web workspace
   - Full read/write access in cloud sandbox
   - Changes can be teleported back to CLI

3. **Continue the Conversation**
   ```
   You: Okay, now that I can see the visualization better,
        let's update the API response format

   Claude: I can see from our earlier discussion that we were
           working on the JWT authentication. Let me show you
           the updated response format...
   ```

### Teleport Session Lifetime

Teleported sessions have different lifetimes:

| Session Type | Lifetime | Extension |
|--------------|----------|-----------|
| Active session | 24 hours | Auto-extends with activity |
| Idle session | 2 hours | Manual refresh available |
| Shared session | 1 hour | Recipient can extend |
| Background task | Until completion | +1 hour after completion |

---

## Teleporting TO CLI from Web

### Starting Work on claude.ai/code

Begin your session on the web interface:

1. **Navigate to claude.ai/code**
2. **Start a new session or continue existing**
3. **Work on your task** - write code, debug, design

```
[On claude.ai/code]

You: Help me design a REST API for a todo application

Claude: I'll help you design a REST API for a todo application.
        Let me create a comprehensive specification...

[Claude creates files, runs code, shows visualizations]
```

### Initiating Teleport to CLI

From the web interface, use the teleport option:

1. **Click the session menu (three dots)**
2. **Select "Teleport to CLI"**
3. **Copy the provided command**

Or type in the web chat:
```
> /teleport-to-cli
```

You'll receive:
```
Teleport package created!

Run this command in your terminal:
claude --teleport abc123xyz789

Or start Claude and enter:
/teleport-receive abc123xyz789
```

### Using `claude --teleport`

In your terminal:

```bash
# Start Claude with teleport session
claude --teleport abc123xyz789
```

Output:
```
Receiving teleport session...
✓ Downloading conversation (23 messages)
✓ Extracting file context (8 files)
✓ Applying environment state
✓ Loading session metadata

Session teleported from: claude.ai/code
Original start time: 2024-01-15 14:30 UTC
Files to sync: 8

Would you like to:
[1] Sync all files to current directory
[2] Sync to a new directory
[3] Review files before syncing
[4] Continue without syncing files

Choice: 1

Syncing files...
✓ api/routes.py (created)
✓ api/models.py (created)
✓ tests/test_routes.py (created)
...

Session loaded. Continuing conversation...

Claude: Welcome back to the CLI! I see we were designing the REST API
        for your todo application. Your local files are now synced with
        the work we did on the web. What would you like to do next?
```

### Syncing Remote Changes Locally

When teleporting from web to CLI, you have options for file handling:

**Option 1: Full Sync**
```bash
claude --teleport abc123 --sync-all
```
All files from the web session are created/updated locally.

**Option 2: Selective Sync**
```bash
claude --teleport abc123 --sync-interactive
```
Review each file before syncing:
```
File: api/routes.py (new file)
Preview:
  1 | from flask import Flask, jsonify
  2 | from models import Todo
  3 | ...
Sync this file? [y/n/d(diff)/v(view all)]: y
```

**Option 3: Directory Mapping**
```bash
claude --teleport abc123 --sync-to ./my-project
```
Sync files to a specific local directory.

### Teleport Receive Command

If you're already in a Claude session:

```bash
# Inside active Claude session
> /teleport-receive abc123xyz789

# Or short form
> /tpr abc123xyz789
```

This merges the incoming session with your current session.

---

## Bidirectional Workflows

### Workflow 1: Start Locally, Continue on Mobile

**Scenario**: You're refactoring code at your desk but need to leave for a meeting.

```bash
# At your desk - CLI session
claude
> Help me refactor the user service to use dependency injection

[Work progresses for 30 minutes]

> /teleport "Heading to standup, will continue on phone"

# Teleport complete - URL copied to clipboard
```

**On your mobile device:**
1. Open claude.ai/code in mobile browser
2. Session loads with full context
3. Continue reviewing, planning, or even making changes

```
You: While I'm mobile, let's plan out the remaining changes

Claude: Based on our refactoring session, here are the remaining tasks:
1. UserService constructor injection
2. Update UserController dependencies
3. Modify test fixtures
...
```

### Workflow 2: Start on Web, Finish in IDE

**Scenario**: You design an algorithm on the web with visualizations, then implement it locally.

**On claude.ai/code:**
```
You: Help me design a caching algorithm for our API responses

Claude: I'll help you design a caching strategy. Let me visualize
        different approaches...

[Claude creates diagrams, runs simulations, shows performance graphs]

You: The LRU with TTL hybrid looks good. Let me implement this locally.
> /teleport-to-cli
```

**In your terminal:**
```bash
claude --teleport xyz789abc

# Session loads with the algorithm design
> Now let's implement this in our actual codebase

Claude: I'll help you implement the LRU-TTL hybrid cache we designed.
        I can see your project structure. Let me create the cache
        module in the appropriate location...

[Claude reads local files, creates implementation, runs tests]
```

### Workflow 3: Team Handoffs via Teleport

**Scenario**: You've debugged an issue and want to hand off to a colleague.

```bash
# Your session
> /teleport --share --message "Found the bug - it's in the auth middleware.
   @sarah can you review my fix?"

Share link created:
https://claude.ai/code/shared/abc123xyz

This link expires in: 1 hour
Recipient can extend the session.
```

**Colleague receives the link:**
```
[Sarah opens the shared link]

Session shared by: your.email@example.com
Message: "Found the bug - it's in the auth middleware.
         @sarah can you review my fix?"

Claude: Welcome! I was working with your colleague on debugging an
        authentication issue. We identified the problem in the auth
        middleware - specifically in the token validation logic.
        Would you like me to walk you through the fix we developed?
```

### Workflow 4: Round-Trip Development

**Scenario**: Full round-trip between CLI, web, and back.

```
Day 1 - Morning (CLI):
└── Start feature development
    └── /teleport "lunch break"

Day 1 - Lunch (Mobile):
└── Review approach on phone
    └── Make notes and small edits

Day 1 - Afternoon (CLI):
└── claude --teleport xyz789
    └── Continue implementation

Day 1 - Evening (Tablet):
└── Review code at home
    └── /teleport-to-cli

Day 2 - Morning (CLI):
└── claude --teleport abc123
    └── Finish and merge
```

---

## Background Cloud Execution

### Dispatching Long-Running Tasks

Use the `&` suffix to send tasks to cloud execution:

```bash
# Inside Claude session
> Run comprehensive tests across all modules &

Dispatching to cloud execution...
Task ID: task_abc123xyz
Status URL: https://claude.ai/code/task/abc123xyz

You can:
- Continue working locally
- Check status with: /task-status abc123xyz
- View results on web: https://claude.ai/code/task/abc123xyz
```

### How Background Execution Works

```
┌────────────────────────────────────────────────────────────────┐
│                   Background Task Flow                          │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CLI Session                    Cloud Execution                 │
│  ┌─────────┐                   ┌─────────────────┐             │
│  │ claude  │──── dispatch ────►│ Cloud Container │             │
│  │         │                   │                 │             │
│  │ (free)  │                   │ Running task... │             │
│  │         │◄─── notify ───────│ Complete!       │             │
│  └─────────┘                   └─────────────────┘             │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

### Task Types Suitable for Background Execution

| Task Type | Example | Background Benefit |
|-----------|---------|-------------------|
| Test suites | `Run all tests &` | Free up terminal |
| Builds | `Build production bundle &` | Continue coding |
| Code analysis | `Analyze codebase for security issues &` | Non-blocking |
| Large refactors | `Apply this pattern to all files &` | Hours-long tasks |
| Data processing | `Process these log files &` | Resource-intensive |

### Monitoring Remote Tasks

**Check status from CLI:**
```bash
> /task-status abc123xyz

Task: abc123xyz
Status: Running (45% complete)
Started: 10 minutes ago
Estimated completion: 12 minutes
Current step: Running integration tests (15/34)
```

**List all background tasks:**
```bash
> /tasks

Active Tasks:
  task_abc123  Running   Test suite         45%   ~12min remaining
  task_def456  Queued    Security analysis  0%    Waiting...

Completed (last 24h):
  task_ghi789  Complete  Build             100%  Finished 2h ago
  task_jkl012  Failed    Deploy            Error at step 3
```

### Retrieving Results

**When a task completes:**
```
[Notification appears in CLI]

Background task completed: task_abc123xyz
Task: Run comprehensive tests across all modules
Duration: 23 minutes
Result: Success (340 tests passed, 2 skipped)

View full results? [y/n]: y

[Results displayed or opened in browser]
```

**Manually retrieve results:**
```bash
> /task-results abc123xyz

# Or pull results into current session
> /task-pull abc123xyz

Pulling results from cloud task...
✓ Test results imported
✓ Coverage report available
✓ 3 files modified during task

Claude: The test suite completed successfully. I found a few areas where
        we could improve coverage. Would you like me to show you the
        coverage report?
```

### Canceling Background Tasks

```bash
> /task-cancel abc123xyz

Are you sure you want to cancel task_abc123xyz? [y/n]: y

Task canceled.
Partial results available: https://claude.ai/code/task/abc123xyz
```

---

## Templates and Commands

### Core Teleport Commands

```bash
# ============================================
# TELEPORTING FROM CLI TO WEB
# ============================================

# Basic teleport - transfer session to web
/teleport

# Short form
/tp

# Teleport with context message
/teleport "Need to continue on mobile"
/tp "Switching to tablet for code review"

# Teleport options
/teleport --no-browser          # Don't auto-open browser
/teleport --minimal             # Conversation only, no files
/teleport --full                # Include all project files
/teleport --share               # Create shareable team link
/teleport --share --expires 2h  # Custom expiration

# ============================================
# TELEPORTING FROM WEB TO CLI
# ============================================

# Start CLI with teleport session
claude --teleport <session-id>
claude --teleport abc123xyz789

# Teleport with sync options
claude --teleport <id> --sync-all        # Sync all files
claude --teleport <id> --sync-none       # No file sync
claude --teleport <id> --sync-interactive # Choose files
claude --teleport <id> --sync-to ./dir   # Sync to directory

# Receive teleport in existing session
/teleport-receive <session-id>
/tpr <session-id>

# ============================================
# BACKGROUND CLOUD EXECUTION
# ============================================

# Dispatch task to cloud (add & suffix)
> Run tests &
> Build project &
> Analyze codebase for security issues &

# Task management
/tasks                     # List all tasks
/task-status <task-id>     # Check specific task
/task-results <task-id>    # View task results
/task-pull <task-id>       # Pull results into session
/task-cancel <task-id>     # Cancel running task

# ============================================
# SESSION MANAGEMENT
# ============================================

# View teleport history
/teleport-history

# Resume previous teleport
/teleport-resume <session-id>

# Delete teleport session
/teleport-delete <session-id>
```

### Configuration Templates

```bash
# ============================================
# CLAUDE CONFIG FOR TELEPORT
# ============================================

# Set default browser
claude config set browser chrome

# Configure auto-sync behavior
claude config set teleport.auto-sync true
claude config set teleport.sync-directory ./projects

# Set default teleport options
claude config set teleport.default-minimal false
claude config set teleport.auto-open-browser true

# Configure background task notifications
claude config set tasks.notify-on-complete true
claude config set tasks.notification-sound true
```

### Shell Aliases (Optional)

Add to your `.bashrc` or `.zshrc`:

```bash
# ============================================
# HELPFUL ALIASES FOR TELEPORT
# ============================================

# Quick teleport start
alias ctp='claude --teleport'

# Start Claude and immediately teleport last web session
alias ctp-last='claude --teleport-latest'

# Check background tasks
alias ctasks='claude --list-tasks'
```

---

## Real-World Examples

### Example 1: Complex Refactor - Desktop to Mobile

**Situation**: You're refactoring a payment processing module but need to commute home.

**Desktop Session (CLI):**
```bash
claude
> I need to refactor our payment processing module to support multiple
  payment providers. Currently we only support Stripe.

Claude: I'll help you refactor the payment module for multi-provider support.
        Let me examine the current implementation...

[Claude analyzes the codebase]

Claude: I've identified the key areas to modify:
        1. Create PaymentProvider interface
        2. Refactor StripeProvider to implement interface
        3. Create PaymentProviderFactory
        4. Update PaymentService to use factory
        ...

> Let's start with the interface design

[30 minutes of productive work]

> /teleport "Heading home, want to continue thinking about the factory pattern"

Teleporting session...
✓ Session ready at: https://claude.ai/code/session/pay_refactor_123
```

**During Commute (Mobile):**
```
You: While I'm on the train, let's discuss the factory pattern approach.
     Should we use abstract factory or simple factory?

Claude: Given our earlier analysis of the payment module, I'd recommend
        the abstract factory pattern because:

        1. We identified 3 payment providers (Stripe, PayPal, Square)
        2. Each provider needs multiple related objects (Client, Webhook, etc.)
        3. We want to ensure consistency within each provider family

        Here's the refined design...

[Continue architectural discussion]
```

**Back Home (CLI):**
```bash
claude --teleport pay_refactor_123 --sync-to ./my-project/payments

Session loaded with factory pattern discussion.

> Great, now let's implement the abstract factory we designed

Claude: I'll implement the abstract factory pattern we discussed during
        your commute. Starting with the base interfaces...

[Claude creates files locally]
```

### Example 2: Debug on Web, Finish in IDE

**Situation**: You're debugging a complex data transformation bug and need better
visualization than the terminal provides.

**CLI Session:**
```bash
claude
> There's a bug in our data pipeline. The aggregation step is producing
  wrong results for certain edge cases.

Claude: Let me examine the data pipeline code...

[Claude investigates]

Claude: I see the issue might be in the groupBy logic. However, to properly
        debug this, it would help to visualize the data at each transformation
        step. Would you like to teleport to the web where I can create
        interactive visualizations?

> Yes, let's do that
> /teleport "Need visualizations for data pipeline debugging"
```

**Web Session (claude.ai/code):**
```
Claude: Now that we're on the web, let me create a step-by-step
        visualization of your data pipeline...

[Claude generates interactive charts showing data flow]
[Claude creates a visual diff of expected vs actual output]

You: I can see it now - the issue is in step 3 where we're losing
     records with null values

Claude: Exactly! The groupBy is excluding null keys. Here's the fix...

You: Perfect, let me implement this in my actual codebase
> /teleport-to-cli
```

**CLI Session:**
```bash
claude --teleport debug_pipeline_456

Claude: Now that we're back in your local environment with the bug
        identified (null key exclusion in groupBy), let me apply
        the fix to your actual pipeline code...

[Claude modifies local files]

> Run the tests

Claude: Running tests...
        ✓ All 47 tests passing
        ✓ Edge case with null values now handled correctly
```

### Example 3: Share Work with Colleague

**Situation**: You've developed a solution and want a colleague to review it.

**Your Session:**
```bash
claude
> I've finished the new caching layer implementation. Create a teleport
  link for Sarah to review.

> /teleport --share --message "Sarah - please review the caching implementation.
  Focus on the eviction policy in cache_manager.py"

Creating shareable teleport link...
✓ Link created: https://claude.ai/code/shared/cache_review_789
✓ Expires in: 1 hour
✓ Link copied to clipboard

Share this link with your colleague. They can extend the session
if they need more time.
```

**Sarah's Review:**
```
[Sarah opens the link]

Claude: Welcome! Your colleague has been working on a new caching layer
        implementation. They specifically asked you to focus on the
        eviction policy in cache_manager.py.

        Here's a summary of what was implemented:
        1. LRU cache with TTL support
        2. Configurable eviction policies
        3. Cache warming strategies

        Would you like me to walk you through the eviction policy code?

Sarah: Yes, show me the eviction policy implementation

Claude: Here's the eviction policy from cache_manager.py...

[Sarah reviews with Claude's help, adds comments]

Sarah: I have some suggestions. Can I send these back to my colleague?

Claude: I'll create a summary of your review feedback that your
        colleague can receive...
```

### Example 4: Long-Running Task to Cloud

**Situation**: You need to run a comprehensive security audit that takes hours.

```bash
claude
> Run a comprehensive security audit on our codebase. Check for:
  - OWASP Top 10 vulnerabilities
  - Dependency vulnerabilities
  - Hardcoded secrets
  - SQL injection risks
  - XSS vulnerabilities
  &

Dispatching security audit to cloud execution...
Task ID: security_audit_101
Estimated duration: 2-3 hours
Status URL: https://claude.ai/code/task/security_audit_101

You'll be notified when complete. Continue working locally.

> Great, while that runs, let's work on the new feature

[Continue working normally]

[2 hours later - notification appears]

Background task completed: security_audit_101
Duration: 2h 14m
Findings: 3 High, 7 Medium, 12 Low severity issues
Full report: https://claude.ai/code/task/security_audit_101

View summary? [y/n]: y

Claude: The security audit found 22 issues total. The 3 high-severity
        findings are:
        1. SQL injection in user search (user_controller.py:145)
        2. Hardcoded API key in config (settings.py:23)
        3. Missing rate limiting on auth endpoints

        Would you like to address these now?
```

### Example 5: Code Review on Tablet, Fixes in Terminal

**Situation**: You want to review a PR on your tablet during lunch, then implement
fixes when back at your desk.

**Tablet (Web):**
```
You: Let's review PR #234 - the new authentication system

Claude: I'll analyze PR #234 for you...

[Claude fetches PR diff, provides detailed review]

Claude: I found several issues:
1. Token expiration not handled (auth.js:45)
2. Missing input validation (login.js:23)
3. Password hashing uses weak algorithm
4. Session storage is vulnerable to XSS

You: Good catches. Save this review for when I'm back at my desk.
> /teleport-to-cli

Teleport command for CLI:
claude --teleport pr_review_234
```

**Back at Desk (CLI):**
```bash
claude --teleport pr_review_234

Claude: Welcome back! We reviewed PR #234 and found 4 issues. Would you
        like me to help fix them in order of severity?

> Yes, let's start with the password hashing issue

Claude: I'll update the password hashing to use bcrypt with proper
        cost factor. Let me modify the auth module...

[Claude makes local changes]

> Run the auth tests

Claude: Running auth tests...
        ✓ 23/23 tests passing

        Ready to commit these changes?
```

### Example 6: Interview Prep - Web to Local

**Situation**: You're preparing for a coding interview, writing solutions on the
web, then testing them in your local environment.

**Web Session:**
```
You: Help me practice implementing a LRU cache for my interview

Claude: Let's implement an LRU cache step by step. I'll explain
        the approach and then we'll code it together...

[Claude teaches the concept with visualizations]

You: I think I understand. Let me try implementing it.

[You write the solution with Claude's guidance]

Claude: Excellent implementation! Let's verify it works correctly
        with some test cases...

[Claude runs tests in the web sandbox]

You: I want to test this in my actual development environment with
     my testing framework
> /teleport-to-cli
```

**Local Environment:**
```bash
claude --teleport interview_prep_567 --sync-to ./interview-practice

Claude: I've synced your LRU cache implementation to your local
        interview-practice directory. Let's run it with your
        testing framework.

> Run with pytest and show coverage

Claude: Running pytest...

[Tests run locally with full coverage report]
```

### Example 7: Documentation - Draft on Web, Finalize in IDE

**Situation**: You want to draft API documentation with visualizations, then
finalize it in your IDE for proper formatting and integration.

**Web Session:**
```
You: Help me create comprehensive API documentation for our REST endpoints

Claude: I'll help create detailed API documentation. Let me analyze
        your endpoints and create documentation with examples...

[Claude generates documentation with formatted examples]
[Claude creates diagrams showing request/response flows]
[Claude builds interactive API explorer]

You: This looks great. Let me finalize this in VS Code where I can
     integrate it with our documentation system
> /teleport-to-cli
```

**Local Session:**
```bash
claude --teleport api_docs_890 --sync-to ./docs/api

Claude: Documentation synced to ./docs/api. I see you're using
        VuePress for documentation. Let me format these files
        to match your existing documentation structure...

[Claude adapts documentation to local format]

> Build the docs and preview

Claude: Building documentation...
        Preview available at: http://localhost:8080/api-docs
```

---

## CLI vs Web Comparison

### Feature Comparison Table

| Feature | CLI | Web (claude.ai/code) |
|---------|-----|----------------------|
| **File Access** | Full local filesystem | Cloud sandbox only |
| **Code Execution** | Local shell/runtime | GVisor secure sandbox |
| **Git Integration** | Direct repository access | Limited (snapshot only) |
| **Visualizations** | Text-based only | Rich charts, diagrams |
| **Session Persistence** | Until terminal closes | Persists in cloud |
| **Mobile Access** | Not practical | Fully supported |
| **Offline Mode** | Partial (cached context) | Requires internet |
| **Resource Limits** | Your machine's specs | Managed cloud allocation |
| **File Size Limits** | None | 50MB per file |
| **Execution Time** | Unlimited | 10 minutes per operation |
| **Custom Tools** | Install anything | Pre-installed set |
| **Environment Variables** | Full access | Isolated per session |
| **Database Access** | Direct connection | Sandbox only |
| **Network Access** | Unrestricted | Controlled egress |
| **IDE Integration** | VS Code, JetBrains, etc. | Web editor only |
| **Keyboard Shortcuts** | Terminal standard | Web standard |
| **Copy/Paste** | Terminal behavior | Browser behavior |
| **Multi-file Editing** | Excellent | Good |
| **Long Operations** | Can interrupt | Background support |
| **Collaboration** | Not built-in | Shareable links |

### Performance Comparison

| Operation | CLI | Web |
|-----------|-----|-----|
| File read (1MB) | ~10ms | ~100ms |
| File write | Instant | ~200ms sync |
| Code execution | Native speed | Sandbox overhead |
| Test suite (100 tests) | ~5s (varies) | ~8s (varies) |
| Large file processing | Local resources | Cloud resources |
| Memory-intensive tasks | Your RAM | 8GB allocation |
| CPU-intensive tasks | Your CPU | Shared compute |

### Best Use Cases

**CLI Excels At:**
```
- Working with large codebases
- Git operations (commit, push, rebase)
- Database operations
- Running local servers
- Using custom/proprietary tools
- Air-gapped environments
- Maximum performance needs
- Long-running local processes
- IDE integration workflows
```

**Web Excels At:**
```
- Visualizations and diagrams
- Mobile/tablet work
- Quick prototyping
- Learning and exploration
- Sharing with colleagues
- Background cloud tasks
- Browser-based testing
- Consistent environment
- No local setup required
```

---

## When to Use Each Environment

### Choose CLI When:

1. **Full Filesystem Access Required**
   ```bash
   # Need to read/write across entire project
   claude
   > Refactor all services to use the new logging pattern
   ```

2. **Git Operations**
   ```bash
   # Commit, push, rebase, etc.
   > Create a feature branch and commit these changes
   ```

3. **Local Server Development**
   ```bash
   # Running and testing local servers
   > Start the dev server and test the new endpoints
   ```

4. **Database Work**
   ```bash
   # Direct database connections
   > Run this migration on the local database
   ```

5. **Custom Tool Requirements**
   ```bash
   # Need specific tools installed
   > Use terraform to plan this infrastructure change
   ```

6. **Performance-Critical Operations**
   ```bash
   # When speed matters
   > Process these 10GB of log files
   ```

7. **IDE Integration**
   ```bash
   # Working alongside VS Code, etc.
   > Update the file and I'll see it in my editor
   ```

### Choose Web When:

1. **Visualization Needs**
   ```
   # Charts, diagrams, visual debugging
   You: Show me a visualization of the data flow
   ```

2. **Mobile/Tablet Work**
   ```
   # On the go
   [Working from phone/tablet]
   ```

3. **Quick Prototyping**
   ```
   # No local setup needed
   You: Help me prototype this algorithm quickly
   ```

4. **Sharing with Team**
   ```
   # Easy collaboration
   You: Create a shareable link for code review
   ```

5. **Background Tasks**
   ```
   # Long-running operations
   > Run comprehensive analysis &
   ```

6. **Consistent Environment**
   ```
   # Same setup every time
   You: I need a clean Python 3.11 environment
   ```

7. **Learning/Exploration**
   ```
   # Safe sandbox for experimentation
   You: Let me try this potentially dangerous operation safely
   ```

### Hybrid Workflow Benefits

The real power comes from combining both:

```
┌─────────────────────────────────────────────────────────────────┐
│                     Optimal Hybrid Workflow                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Design Phase (Web)                                            │
│   ├── Visualize architecture                                    │
│   ├── Prototype algorithms                                      │
│   └── Get team feedback via sharing                            │
│              │                                                   │
│              ▼ /teleport                                        │
│   Implementation Phase (CLI)                                     │
│   ├── Write production code                                     │
│   ├── Integrate with codebase                                   │
│   └── Run tests locally                                         │
│              │                                                   │
│              ▼ /teleport                                        │
│   Review Phase (Web)                                            │
│   ├── Create shareable review link                              │
│   ├── Visualize changes                                         │
│   └── Discuss with team                                         │
│              │                                                   │
│              ▼ /teleport                                        │
│   Finalization Phase (CLI)                                       │
│   ├── Apply review feedback                                     │
│   ├── Final testing                                             │
│   └── Commit and push                                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Troubleshooting

### Common Issues and Solutions

#### Issue 1: "Teleport failed - Authentication error"

**Symptoms:**
```
> /teleport

Error: Teleport failed
Reason: Authentication error - session expired

[ERR-TP-001]
```

**Solutions:**
```bash
# 1. Re-authenticate
claude auth login

# 2. Check auth status
claude auth status

# 3. If status shows expired, refresh
claude auth refresh

# 4. Try teleport again
/teleport
```

#### Issue 2: "Context not transferring completely"

**Symptoms:**
- Some files missing after teleport
- Conversation history truncated
- Claude doesn't remember recent context

**Solutions:**
```bash
# 1. Check what was transferred
/teleport --verbose

# Output shows:
# Transferring...
# ✓ Conversation: 45 messages
# ⚠ Files: 8/12 transferred (4 exceeded size limit)
# ✓ Git state: captured

# 2. For large files, use minimal mode + manual upload
/teleport --minimal

# Then upload specific files on web

# 3. Check file size limits
# Files > 50MB are not transferred
# Split or compress large files first
```

#### Issue 3: "Session mismatch - cannot continue"

**Symptoms:**
```
claude --teleport abc123xyz

Error: Session mismatch
The target session has diverged from the source.
```

**Solutions:**
```bash
# 1. Check session status
claude --teleport abc123xyz --status

# Output might show:
# Source session: Modified locally after teleport
# Target session: Has additional messages
# Conflict: Cannot auto-merge

# 2. Force sync (overwrites local changes)
claude --teleport abc123xyz --force

# 3. Or start fresh with just conversation
claude --teleport abc123xyz --conversation-only

# 4. View diff before deciding
claude --teleport abc123xyz --show-diff
```

#### Issue 4: "Teleport timeout"

**Symptoms:**
```
> /teleport

Uploading session... 45%
Error: Teleport timeout after 60 seconds

[ERR-TP-004]
```

**Solutions:**
```bash
# 1. Check network connection
ping api.anthropic.com

# 2. Use minimal mode for faster transfer
/teleport --minimal

# 3. Increase timeout
/teleport --timeout 180

# 4. Check for large files
/teleport --dry-run  # Shows what would be transferred

# 5. Exclude problematic files
/teleport --exclude "*.log" --exclude "node_modules"
```

#### Issue 5: "Browser not opening"

**Symptoms:**
```
> /teleport

Teleport ready!
Opening browser to: https://claude.ai/code/session/abc123

Error: Could not open browser
```

**Solutions:**
```bash
# 1. Check browser configuration
claude config get browser

# 2. Set browser explicitly
claude config set browser chrome
# Or: firefox, safari, edge, brave

# 3. Use no-browser mode and copy URL manually
/teleport --no-browser

# 4. Set browser via environment variable
export CLAUDE_BROWSER=firefox
# Or on Windows: set CLAUDE_BROWSER=chrome

# 5. Test browser integration
claude config test-browser
```

#### Issue 6: "Files not syncing from web"

**Symptoms:**
```bash
claude --teleport abc123 --sync-all

Syncing files...
⚠ 3 files failed to sync
  - src/config.py (permission denied)
  - .env (blocked - contains secrets)
  - build/output (directory too large)
```

**Solutions:**
```bash
# 1. Check file permissions
ls -la ./src/config.py
chmod u+w ./src/config.py

# 2. Secret files are blocked for security
# Download manually if truly needed:
claude --teleport abc123 --include-secrets  # Requires confirmation

# 3. Large directories need explicit handling
claude --teleport abc123 --sync-to ./new-dir  # Fresh directory

# 4. Interactive mode to handle issues one by one
claude --teleport abc123 --sync-interactive
```

#### Issue 7: "Background task stuck"

**Symptoms:**
```
> /task-status abc123

Task: abc123
Status: Running (stuck at 67%)
Started: 3 hours ago
Last activity: 2 hours ago
```

**Solutions:**
```bash
# 1. Check task logs
/task-logs abc123

# 2. Attempt to resume
/task-resume abc123

# 3. Cancel and retrieve partial results
/task-cancel abc123 --save-partial

# 4. View partial results
/task-results abc123 --partial

# 5. Restart task
/task-restart abc123
```

### Error Code Reference

| Code | Meaning | Quick Fix |
|------|---------|-----------|
| ERR-TP-001 | Authentication failed | `claude auth login` |
| ERR-TP-002 | Session not found | Check session ID |
| ERR-TP-003 | Session expired | Start new teleport |
| ERR-TP-004 | Timeout | Use `--minimal` or `--timeout` |
| ERR-TP-005 | Network error | Check internet connection |
| ERR-TP-006 | Size limit exceeded | Exclude large files |
| ERR-TP-007 | Permission denied | Check file permissions |
| ERR-TP-008 | Session conflict | Use `--force` or `--show-diff` |
| ERR-TP-009 | Browser error | Configure browser manually |
| ERR-TP-010 | Rate limited | Wait and retry |

### Getting Help

```bash
# Built-in help
/help teleport

# Detailed troubleshooting
claude teleport --troubleshoot

# View recent teleport logs
claude logs --feature teleport --last 10

# Report an issue
claude feedback --category teleport
```

---

## Quick Reference Cheat Sheet

### Essential Commands

```
╔═══════════════════════════════════════════════════════════════════╗
║                    TELEPORT QUICK REFERENCE                        ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  CLI → WEB                                                        ║
║  ─────────────────────────────────────────────────────────────    ║
║  /teleport              Basic teleport to web                     ║
║  /tp                    Short form                                ║
║  /tp "message"          Teleport with context message             ║
║  /tp --share            Create shareable link                     ║
║  /tp --no-browser       Get URL without opening browser           ║
║                                                                    ║
║  WEB → CLI                                                        ║
║  ─────────────────────────────────────────────────────────────    ║
║  claude --teleport ID   Start CLI with teleport session           ║
║  /tpr ID                Receive teleport in active session        ║
║  --sync-all             Sync all files                            ║
║  --sync-interactive     Choose files to sync                      ║
║                                                                    ║
║  BACKGROUND TASKS                                                  ║
║  ─────────────────────────────────────────────────────────────    ║
║  > command &            Dispatch to cloud                         ║
║  /tasks                 List all tasks                            ║
║  /task-status ID        Check task status                         ║
║  /task-results ID       Get results                               ║
║  /task-cancel ID        Cancel task                               ║
║                                                                    ║
║  CONFIGURATION                                                     ║
║  ─────────────────────────────────────────────────────────────    ║
║  claude auth login      Authenticate                              ║
║  claude auth status     Check auth status                         ║
║  claude config set browser chrome                                 ║
║                                                                    ║
╚═══════════════════════════════════════════════════════════════════╝
```

### Teleport Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│                         YOUR WORKFLOW                            │
│                                                                  │
│    ┌──────────┐         ┌──────────┐         ┌──────────┐       │
│    │   CLI    │◄───────►│  CLOUD   │◄───────►│   WEB    │       │
│    │ Session  │ teleport│ Storage  │ teleport│ Session  │       │
│    └──────────┘         └──────────┘         └──────────┘       │
│         │                    │                    │              │
│         │                    │                    │              │
│    ┌────▼────┐          ┌────▼────┐         ┌────▼────┐         │
│    │  Local  │          │ Backgnd │         │  Mobile │         │
│    │  Files  │          │  Tasks  │         │ Tablet  │         │
│    └─────────┘          └─────────┘         └─────────┘         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Decision Tree: CLI or Web?

```
Need to work on code?
│
├─► Need full filesystem access? ─────────► CLI
│
├─► Need visualizations? ─────────────────► Web
│
├─► On mobile/tablet? ────────────────────► Web
│
├─► Need to share with team? ─────────────► Web (with share link)
│
├─► Running long task? ───────────────────► Background (&)
│
├─► Need Git operations? ─────────────────► CLI
│
├─► Quick prototype? ─────────────────────► Web
│
└─► Default for implementation ───────────► CLI
```

---

## Best Practices

### When to Teleport vs Stay Local

**Teleport When:**

1. **Context switch is imminent**
   ```bash
   # About to leave for meeting
   > /tp "Continuing after standup"
   ```

2. **Visualization would help**
   ```bash
   # Complex debugging
   > /tp "Need to visualize the data flow"
   ```

3. **Sharing is required**
   ```bash
   # Code review needed
   > /tp --share "Ready for review"
   ```

4. **Mobile access needed**
   ```bash
   # Leaving desk
   > /tp "Continue on phone"
   ```

**Stay Local When:**

1. **Active file editing**
   ```bash
   # In the middle of implementation
   # Stay in CLI for best performance
   ```

2. **Git workflow in progress**
   ```bash
   # About to commit/push
   # Stay in CLI for direct Git access
   ```

3. **Running tests/servers**
   ```bash
   # Local server running
   # Stay in CLI to monitor
   ```

### Preparing Context for Teleport

**Good Practice: Summarize Before Teleporting**
```bash
> Before I teleport, let's summarize:
  - We're refactoring the auth module
  - Completed: interface design
  - In progress: implementation
  - Next: unit tests

> /tp "Auth refactor - implementation in progress"
```

**Good Practice: Note Key Files**
```bash
> Key files to focus on when I return:
  - src/auth/provider.py (main work)
  - tests/auth/test_provider.py (need to write)

> /tp
```

**Good Practice: Save Checkpoints**
```bash
> Let's commit what we have before teleporting
  [Claude commits changes]

> /tp "Clean checkpoint - can safely continue anywhere"
```

### Security Considerations

**What's Safe to Teleport:**
- Source code (non-sensitive)
- Configuration files (without secrets)
- Documentation
- Test files

**What to Be Careful With:**
- Environment files (.env)
- Credentials or API keys
- Production database connections
- Private keys or certificates

**Best Practices:**
```bash
# 1. Use .teleportignore file
echo ".env" >> .teleportignore
echo "secrets/" >> .teleportignore
echo "*.pem" >> .teleportignore

# 2. Review before teleporting
/teleport --dry-run

# Output:
# Will transfer:
#   src/* (34 files)
#   tests/* (12 files)
# Will NOT transfer (in .teleportignore):
#   .env
#   secrets/api_key.json

# 3. Use minimal mode for sensitive projects
/teleport --minimal  # Conversation only

# 4. For shared links, set expiration
/teleport --share --expires 30m  # 30 minute expiration
```

### Optimizing Teleport Performance

**For Faster Teleports:**
```bash
# 1. Use minimal mode when files aren't needed
/teleport --minimal

# 2. Exclude large directories
/teleport --exclude "node_modules" --exclude "dist"

# 3. Exclude binary files
/teleport --exclude "*.bin" --exclude "*.exe"

# 4. Set up permanent exclusions
claude config set teleport.exclude "node_modules,dist,*.bin"
```

**For Better Context Transfer:**
```bash
# 1. Include specific files explicitly
/teleport --include "src/critical_file.py"

# 2. Use full mode for complete context
/teleport --full

# 3. Verify transfer
/teleport --verbose
```

---

## Advanced Patterns

### Pattern 1: Multi-Session Orchestration

Manage multiple teleport sessions simultaneously:

```bash
# Terminal 1: Main development
claude
> Working on feature A
> /teleport --name "feature-a"

# Terminal 2: Parallel work
claude
> Working on feature B
> /teleport --name "feature-b"

# On web: Switch between sessions
# claude.ai/code/session/feature-a
# claude.ai/code/session/feature-b

# Return to specific session
claude --teleport feature-a
```

### Pattern 2: Teleport Chains

Create a chain of teleports for complex workflows:

```bash
# Start on CLI
> /teleport --chain "step-1-design"

# Continue on web with visualization
# Make changes, then:
> /teleport-to-cli --chain "step-2-implement"

# Back to CLI
claude --teleport step-2-implement

# Final review on web
> /teleport --chain "step-3-review"

# View chain history
/teleport-chain-history

# Output:
# Chain: my-feature
# 1. step-1-design (CLI → Web)
# 2. step-2-implement (Web → CLI)
# 3. step-3-review (CLI → Web) [current]
```

### Pattern 3: Automated Teleport Triggers

Set up automatic teleports based on conditions:

```bash
# In .claude/config.yaml
teleport:
  auto_triggers:
    - condition: "session_idle > 30m"
      action: "teleport --minimal"
      message: "Auto-saved due to inactivity"

    - condition: "visualization_requested"
      action: "suggest_teleport"
      message: "Teleport to web for better visualization?"

    - condition: "share_requested"
      action: "teleport --share"
```

### Pattern 4: Team Teleport Workflows

For team collaboration:

```bash
# Developer A: Create shared workspace
> /teleport --team --workspace "sprint-42-auth"

# Developer B: Join workspace
claude --teleport-join "sprint-42-auth"

# Developer C: Join workspace
claude --teleport-join "sprint-42-auth"

# All developers see same session state
# Changes sync in near-real-time

# Transfer ownership
> /teleport-transfer @developer-b

# Leave workspace
> /teleport-leave
```

### Pattern 5: Conditional File Sync

Advanced file synchronization:

```bash
# Sync only changed files
claude --teleport abc123 --sync-changed-only

# Sync with merge strategy
claude --teleport abc123 --sync-merge  # Merge changes
claude --teleport abc123 --sync-ours   # Prefer local
claude --teleport abc123 --sync-theirs # Prefer remote

# Sync with backup
claude --teleport abc123 --sync-all --backup ./backup/

# Preview sync without applying
claude --teleport abc123 --sync-preview
```

### Pattern 6: Background Task Pipelines

Chain background tasks:

```bash
# Create pipeline of tasks
> Run linting & then Run tests & then Build project &

Pipeline created: pipeline_abc123
Tasks:
  1. lint_task (pending)
  2. test_task (waiting for #1)
  3. build_task (waiting for #2)

# Monitor pipeline
/pipeline-status abc123

# Output:
# Pipeline: abc123
# ├── lint_task: Complete ✓
# ├── test_task: Running (45%)
# └── build_task: Waiting

# Cancel pipeline
/pipeline-cancel abc123
```

---

## New in Claude Code v2.1.x

### /remote-env Command (v2.1.0)

View and manage remote session environment:

```
/remote-env
```

This command displays:
- Current remote session status
- Environment configuration
- Connected services
- Active teleport sessions

**Use cases:**
- Debug remote connection issues
- Verify environment state after teleport
- Check remote service availability

### Session URL Attribution in Git (v2.1.9)

Claude Code now includes session URLs in commits and PRs:

**Commit messages:**
```
fix: Resolve authentication timeout

Co-Authored-By: Claude <noreply@anthropic.com>

Generated in session: https://claude.ai/session/abc123
```

**Pull request descriptions:**
```markdown
## Changes

- Fixed authentication timeout handling
- Added retry logic for failed connections

---

🤖 Generated with Claude Code
Session: https://claude.ai/session/abc123
```

**Benefits:**
- Traceability of AI-assisted work
- Link back to conversation context
- Audit trail for code review

**Configuration:**

```json
{
  "git": {
    "includeSessionUrl": true
  }
}
```

Disable if you prefer not to include session URLs:

```json
{
  "git": {
    "includeSessionUrl": false
  }
}
```

### Enhanced Teleport Stability (v2.1.x)

Recent versions include:
- Improved session handoff reliability
- Better handling of large context transfers
- Reduced latency for teleport operations
- Enhanced error recovery for interrupted teleports

---

## Summary

### Key Takeaways

1. **Teleport enables seamless workflow continuity** between CLI and web interfaces,
   eliminating the traditional trade-off between local power and remote flexibility.

2. **Use CLI for**: full filesystem access, Git operations, local servers, custom tools,
   and performance-critical work.

3. **Use Web for**: visualizations, mobile access, sharing, background tasks, and
   quick prototyping.

4. **Hybrid workflows** combine the best of both worlds - design on web with
   visualizations, implement locally with full access.

5. **Background execution** with `&` enables long-running tasks without blocking
   your local development.

6. **Security-first approach**: sensitive files are never transferred by default,
   and shared links have expiration times.

### Quick Start Checklist

```
□ Install Claude Code CLI: npm install -g @anthropic-ai/claude-code
□ Authenticate: claude auth login
□ Start a session: claude
□ Work on your task
□ Teleport when needed: /teleport
□ Continue anywhere
□ Return to CLI: claude --teleport <session-id>
```

### What's Next?

After mastering Teleport, explore:
- **Background Task Automation** - Complex cloud execution pipelines
- **Team Collaboration** - Shared workspaces and handoffs
- **IDE Integration** - VS Code and JetBrains plugins with teleport support

---

## Appendix: Full Command Reference

```
TELEPORT COMMANDS
─────────────────

/teleport, /tp
    Teleport current session to web
    Options:
        --no-browser        Don't open browser automatically
        --minimal           Transfer conversation only
        --full              Include all project files
        --share             Create shareable link
        --expires <time>    Set expiration (e.g., "1h", "30m")
        --name <name>       Name the session for easy reference
        --message <msg>     Add context message
        --exclude <pattern> Exclude files matching pattern
        --include <pattern> Include specific files
        --dry-run           Show what would be transferred
        --verbose           Show detailed transfer progress
        --chain <name>      Create/continue a teleport chain
        --team              Create team-shareable session
        --workspace <name>  Create/join named workspace

/teleport-receive, /tpr <session-id>
    Receive teleport in active CLI session
    Options:
        --merge             Merge with current session
        --replace           Replace current session

/teleport-history
    Show teleport history

/teleport-resume <session-id>
    Resume a previous teleport session

/teleport-delete <session-id>
    Delete a teleport session

/teleport-chain-history
    Show teleport chain history

CLI FLAGS
─────────

claude --teleport <session-id>
    Start CLI with teleport session
    Options:
        --sync-all          Sync all files from session
        --sync-none         Don't sync any files
        --sync-interactive  Choose files interactively
        --sync-to <dir>     Sync to specific directory
        --sync-changed-only Sync only changed files
        --sync-merge        Merge file changes
        --sync-ours         Prefer local files
        --sync-theirs       Prefer remote files
        --sync-preview      Preview sync without applying
        --backup <dir>      Backup before syncing
        --conversation-only Load conversation only
        --force             Force sync, overwrite conflicts
        --show-diff         Show differences before syncing
        --status            Show session status only

claude --teleport-latest
    Resume most recent teleport session

claude --teleport-join <workspace>
    Join a team workspace

BACKGROUND TASKS
────────────────

> command &
    Dispatch command to cloud execution

/tasks
    List all background tasks

/task-status <task-id>
    Check task status

/task-results <task-id>
    View task results
    Options:
        --partial           View partial results

/task-pull <task-id>
    Pull results into current session

/task-cancel <task-id>
    Cancel running task
    Options:
        --save-partial      Save partial results

/task-logs <task-id>
    View task logs

/task-resume <task-id>
    Attempt to resume stuck task

/task-restart <task-id>
    Restart failed task

/pipeline-status <pipeline-id>
    Check pipeline status

/pipeline-cancel <pipeline-id>
    Cancel entire pipeline

CONFIGURATION
─────────────

claude config set browser <browser>
    Set default browser (chrome, firefox, safari, edge, brave)

claude config set teleport.auto-sync <bool>
    Enable/disable automatic file sync

claude config set teleport.sync-directory <path>
    Set default sync directory

claude config set teleport.default-minimal <bool>
    Set minimal mode as default

claude config set teleport.auto-open-browser <bool>
    Enable/disable automatic browser opening

claude config set teleport.exclude <patterns>
    Set permanent exclusion patterns

claude config set tasks.notify-on-complete <bool>
    Enable/disable completion notifications

claude config test-browser
    Test browser integration

TROUBLESHOOTING
───────────────

claude auth login
    Authenticate with Anthropic

claude auth status
    Check authentication status

claude auth refresh
    Refresh authentication

claude teleport --troubleshoot
    Run teleport diagnostics

claude logs --feature teleport --last <n>
    View recent teleport logs

claude feedback --category teleport
    Report teleport issues
```

---

*This tutorial is part of the Claude Code documentation series.*
*Version 2.1.x | Last Updated: January 2026*
