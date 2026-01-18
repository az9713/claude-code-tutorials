# Tutorial 13: Session Management in Claude Code

## A Comprehensive Guide to Managing Conversations, Context, and Continuity

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Core Concepts](#core-concepts)
4. [Starting and Naming Sessions](#starting-and-naming-sessions)
5. [Resuming Sessions](#resuming-sessions)
6. [Session Storage Architecture](#session-storage-architecture)
7. [Context Checkpointing](#context-checkpointing)
8. [Managing Sessions](#managing-sessions)
9. [Real-World Examples](#real-world-examples)
10. [Session Storage Details](#session-storage-details)
11. [Troubleshooting](#troubleshooting)
12. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)
13. [Best Practices](#best-practices)
14. [Summary](#summary)

---

## Overview

### What Are Sessions?

A **session** in Claude Code represents a complete conversation between you and Claude, including all messages exchanged, tool calls executed, files read or modified, and the accumulated context that Claude uses to understand your project. Think of a session as a persistent workspace that remembers everything you've discussed and done together.

### Why Sessions Matter

Sessions are fundamental to productive development with Claude Code for several reasons:

1. **Continuity**: Continue complex work across multiple sittings without losing context
2. **Context Preservation**: Claude remembers what files you've discussed, decisions made, and work completed
3. **Efficiency**: Avoid re-explaining your project structure, coding conventions, or goals
4. **Collaboration**: Share session history with team members for knowledge transfer
5. **Recovery**: Rewind to previous states when something goes wrong
6. **Documentation**: Sessions serve as a record of your development process

### Session Persistence

Claude Code automatically persists your sessions in two complementary formats:

```
Session Data Flow:

User Input ──► Claude Processing ──► Response
     │                                   │
     └─────────► Session Store ◄─────────┘
                     │
         ┌───────────┴───────────┐
         ▼                       ▼
   SQLite Database         JSONL Transcripts
   (metadata, index)       (full conversation)
```

**SQLite Database**: Stores session metadata, making searches and session listing fast
**JSONL Transcripts**: Contains the complete conversation history in a portable format

This dual storage approach provides both performance (quick lookups via SQLite) and portability (human-readable JSONL files that can be backed up or shared).

---

## Prerequisites

Before diving into session management, ensure you have:

### Required Setup

1. **Claude Code Installed**
   ```bash
   # Verify installation
   claude --version

   # Expected output: claude-code v1.x.x or similar
   ```

2. **Basic Claude Code Familiarity**
   - You should be comfortable starting Claude Code
   - You understand the basic command/response flow
   - You've completed at least a few conversations

3. **Terminal/Shell Access**
   - Comfortable navigating directories
   - Basic understanding of file paths
   - Ability to run shell commands

### Recommended Knowledge

- Understanding of your project structure
- Familiarity with git (helpful for understanding checkpoints)
- Basic JSON knowledge (for understanding transcript files)

### Verify Your Setup

```bash
# Start Claude Code
claude

# You should see the interactive prompt
# Type /help to see available commands
/help

# Look for session-related commands like:
# /resume - Resume a previous session
# /rename - Rename the current session
```

---

## Core Concepts

### Session Lifecycle

Every session in Claude Code goes through a predictable lifecycle:

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Created   │ ──► │   Active    │ ──► │   Saved     │
│ (new/resume)│     │ (in use)    │     │ (persisted) │
└─────────────┘     └─────────────┘     └─────────────┘
                          │                    │
                          ▼                    ▼
                    ┌─────────────┐     ┌─────────────┐
                    │ Checkpoints │     │  Resumable  │
                    │  (rewind)   │     │  (future)   │
                    └─────────────┘     └─────────────┘
```

### Session Components

A session contains several types of data:

| Component | Description | Purpose |
|-----------|-------------|---------|
| Messages | User prompts and Claude responses | Conversation history |
| Tool Calls | Commands executed (read, edit, bash) | Action record |
| Context | Files read, project understanding | Knowledge state |
| Metadata | Name, timestamps, project path | Organization |
| Checkpoints | Saved states for rewinding | Recovery points |

### Session Storage

Claude Code stores session data in your home directory:

```
~/.claude/
├── sessions.db          # SQLite database for quick lookups
├── projects/
│   └── {project-hash}/
│       ├── transcripts/
│       │   └── {session-id}.jsonl    # Full conversation
│       └── context/
│           └── {session-id}/         # Saved context data
└── settings.json        # User preferences
```

### Session Identification

Sessions are identified by:

1. **Session ID**: A unique identifier (UUID or hash)
2. **Session Name**: Human-readable name (default or custom)
3. **Project Path**: The directory where the session was started
4. **Timestamp**: When the session was created/last used

---

## Starting and Naming Sessions

### Default Session Behavior

When you start Claude Code, it automatically creates a new session:

```bash
# Start Claude Code in your project directory
cd /path/to/your/project
claude

# A new session is created automatically
# Default name is based on timestamp and project
```

**Default Session Names**

By default, sessions are named using a combination of:
- Timestamp (when started)
- Project directory name
- Random identifier for uniqueness

Example default names:
```
2024-01-15-myproject-a3f2
2024-01-15-myproject-b7c1
2024-01-16-myproject-d4e8
```

### The /rename Command

Give your sessions meaningful names using the `/rename` command:

```bash
# Inside an active Claude Code session
/rename "feature-user-authentication"

# Claude confirms the rename
# Session renamed to: feature-user-authentication
```

**Syntax Options**

```bash
# With quotes (recommended for names with spaces)
/rename "my session name"

# Without quotes (for simple names)
/rename my-session-name

# With special characters (use quotes)
/rename "bugfix-#123-login-issue"
```

### Session Naming Conventions

Adopt consistent naming conventions for easier session management:

#### Convention 1: Project-Task-Date
```bash
# Format: {project}-{task}-{date}
/rename "webapp-auth-jan15"
/rename "webapp-api-refactor-jan16"
/rename "webapp-testing-jan17"
```

#### Convention 2: Type-Description-ID
```bash
# Format: {type}-{description}-{id}
/rename "feature-user-dashboard-42"
/rename "bugfix-login-redirect-123"
/rename "refactor-database-layer-7"
```

#### Convention 3: Sprint-Task
```bash
# Format: sprint{number}-{task}
/rename "sprint12-implement-oauth"
/rename "sprint12-fix-session-timeout"
/rename "sprint13-add-2fa"
```

#### Convention 4: Date-First (Chronological)
```bash
# Format: {YYYY-MM-DD}-{description}
/rename "2024-01-15-api-redesign"
/rename "2024-01-16-api-testing"
/rename "2024-01-17-api-documentation"
```

### Project-Based Naming Strategies

Different projects benefit from different naming strategies:

#### Solo Developer
```bash
# Simple, task-focused names
/rename "add-dark-mode"
/rename "fix-mobile-layout"
/rename "optimize-images"
```

#### Team Projects
```bash
# Include identifier or ticket number
/rename "JIRA-1234-payment-integration"
/rename "GH-567-security-patch"
/rename "TASK-89-performance-audit"
```

#### Multi-Repository Work
```bash
# Include repo name
/rename "frontend-repo-component-library"
/rename "backend-repo-api-v2"
/rename "shared-utils-new-helpers"
```

#### Client Projects
```bash
# Include client identifier
/rename "acme-corp-phase2-dashboard"
/rename "techstart-mvp-auth"
/rename "bigco-maintenance-q1"
```

### Naming Template

```bash
# Template: Start named session for feature work
claude
/rename "feature-{{feature-name}}-{{date}}"

# Template: Bug fix session
claude
/rename "bugfix-{{issue-id}}-{{brief-description}}"

# Template: Learning/exploration session
claude
/rename "explore-{{topic}}-{{date}}"

# Template: Code review session
claude
/rename "review-{{pr-number}}-{{author}}"
```

### When to Name Sessions

**Name immediately when:**
- Starting work on a specific feature
- Beginning a debugging investigation
- Starting a code review
- Working on a task you'll continue later

**Keep default name when:**
- Quick one-off questions
- Exploratory conversations
- Sessions you won't need to revisit

---

## Resuming Sessions

### The /resume Command

The `/resume` command is your gateway to continuing previous work:

```bash
# Start Claude Code
claude

# Use /resume to see available sessions
/resume
```

### Interactive Session Picker

When you run `/resume` without arguments, Claude Code presents an interactive picker:

```
┌─────────────────────────────────────────────────────────────┐
│                    Resume Session                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Recent Sessions:                                            │
│                                                              │
│  > feature-auth-jan15        2 hours ago     /home/user/app │
│    bugfix-login-123          1 day ago       /home/user/app │
│    refactor-api-v2           3 days ago      /home/user/app │
│    explore-new-framework     1 week ago      /home/user/app │
│                                                              │
│  ↑/↓ Navigate   Enter Select   / Search   Esc Cancel        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Navigation:**
- Arrow keys (`↑`/`↓`): Move through sessions
- `Enter`: Select and resume the highlighted session
- `/`: Start searching/filtering
- `Esc`: Cancel and start a new session

### Direct Resume with Session Name

Resume a specific session directly:

```bash
# Resume by exact name
/resume "feature-auth-jan15"

# Resume by partial match (if unique)
/resume auth-jan15

# Resume with project qualifier
/resume "myapp/feature-auth-jan15"
```

### Resume Recent Session

Quickly resume your last session:

```bash
# Resume the most recent session
/resume --recent

# Shorthand (if supported)
/resume -r

# Alternative: Resume last session from specific project
/resume --recent --project /path/to/project
```

### Session Search and Filtering

Find sessions efficiently with search:

```bash
# Search by keyword
/resume --search "authentication"

# Filter by date range
/resume --after 2024-01-01
/resume --before 2024-01-15

# Filter by project
/resume --project /path/to/project

# Combine filters
/resume --search "bugfix" --after 2024-01-10 --project ./myapp
```

### Resume Scenarios

#### Scenario 1: Continue Yesterday's Work
```bash
# You worked on a feature yesterday
claude
/resume --recent

# Or if you remember the name
/resume "feature-dashboard-jan14"
```

#### Scenario 2: Find a Specific Investigation
```bash
# You investigated a bug last week
claude
/resume --search "memory leak"

# Select from matching sessions
```

#### Scenario 3: Switch Between Multiple Features
```bash
# Working on feature A, need to switch to feature B
# End current conversation (Ctrl+C or type 'exit')

# Start new session and resume feature B
claude
/resume "feature-B-notifications"
```

### Resume Template

```bash
# Template: Resume specific session
/resume "feature-auth-2024-01-15"

# Template: Resume with search
/resume --search "{{keyword}}"

# Template: Resume most recent
/resume --recent

# Template: Resume from project
/resume --project "{{project-path}}" --recent
```

---

## Session Storage Architecture

### Overview of Storage Structure

Claude Code uses a sophisticated storage system to ensure session data is both accessible and durable:

```
~/.claude/
│
├── sessions.db                    # Central session index
│
├── projects/
│   │
│   ├── {project-hash-1}/          # Project: /home/user/webapp
│   │   ├── transcripts/
│   │   │   ├── session-a1b2c3.jsonl
│   │   │   ├── session-d4e5f6.jsonl
│   │   │   └── session-g7h8i9.jsonl
│   │   │
│   │   ├── context/
│   │   │   ├── session-a1b2c3/
│   │   │   │   ├── files.json
│   │   │   │   └── state.json
│   │   │   └── session-d4e5f6/
│   │   │       ├── files.json
│   │   │       └── state.json
│   │   │
│   │   └── CLAUDE.md              # Project memory file
│   │
│   └── {project-hash-2}/          # Project: /home/user/api
│       ├── transcripts/
│       ├── context/
│       └── CLAUDE.md
│
├── settings.json                  # User preferences
└── logs/                          # Debug logs
```

### SQLite Database Location

The SQLite database (`sessions.db`) is the central index for all sessions:

**Location:**
```bash
# Default location
~/.claude/sessions.db

# On Windows
C:\Users\{username}\.claude\sessions.db

# On macOS
/Users/{username}/.claude/sessions.db

# On Linux
/home/{username}/.claude/sessions.db
```

**Database Schema (Simplified):**
```sql
-- Sessions table
CREATE TABLE sessions (
    id TEXT PRIMARY KEY,
    name TEXT,
    project_path TEXT,
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    message_count INTEGER,
    is_archived BOOLEAN DEFAULT FALSE
);

-- Session metadata
CREATE TABLE session_metadata (
    session_id TEXT,
    key TEXT,
    value TEXT,
    FOREIGN KEY (session_id) REFERENCES sessions(id)
);

-- Index for fast searches
CREATE INDEX idx_sessions_name ON sessions(name);
CREATE INDEX idx_sessions_project ON sessions(project_path);
CREATE INDEX idx_sessions_updated ON sessions(updated_at);
```

### JSONL Transcript Files

Transcripts are stored in JSONL (JSON Lines) format, where each line is a valid JSON object:

**File Location:**
```
~/.claude/projects/{project-hash}/transcripts/{session-id}.jsonl
```

**JSONL Structure:**
```jsonl
{"type":"session_start","timestamp":"2024-01-15T10:00:00Z","version":"1.0"}
{"type":"user_message","timestamp":"2024-01-15T10:00:05Z","content":"Help me refactor the auth module"}
{"type":"assistant_message","timestamp":"2024-01-15T10:00:10Z","content":"I'll help you refactor..."}
{"type":"tool_call","timestamp":"2024-01-15T10:00:15Z","tool":"read_file","args":{"path":"/src/auth.js"}}
{"type":"tool_result","timestamp":"2024-01-15T10:00:16Z","result":"file contents..."}
{"type":"assistant_message","timestamp":"2024-01-15T10:00:20Z","content":"I see the current structure..."}
{"type":"checkpoint","timestamp":"2024-01-15T10:05:00Z","id":"cp-001"}
```

### What's Stored in Sessions

#### Messages
```json
{
  "type": "user_message",
  "timestamp": "2024-01-15T10:00:05Z",
  "content": "Help me implement user authentication",
  "context": {
    "working_directory": "/home/user/webapp",
    "active_file": "/home/user/webapp/src/auth.js"
  }
}
```

#### Tool Calls
```json
{
  "type": "tool_call",
  "timestamp": "2024-01-15T10:00:15Z",
  "tool": "edit_file",
  "args": {
    "path": "/home/user/webapp/src/auth.js",
    "old_string": "function login()",
    "new_string": "async function login()"
  },
  "result": {
    "success": true,
    "lines_changed": 1
  }
}
```

#### Context
```json
{
  "type": "context_update",
  "timestamp": "2024-01-15T10:00:20Z",
  "files_read": [
    "/home/user/webapp/src/auth.js",
    "/home/user/webapp/src/utils/crypto.js"
  ],
  "project_understanding": {
    "framework": "express",
    "language": "javascript",
    "patterns_detected": ["MVC", "middleware"]
  }
}
```

### Session Metadata

Metadata provides quick access to session information without parsing the full transcript:

```json
{
  "session_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "name": "feature-auth-jan15",
  "project_path": "/home/user/webapp",
  "created_at": "2024-01-15T10:00:00Z",
  "updated_at": "2024-01-15T14:30:00Z",
  "stats": {
    "message_count": 47,
    "tool_calls": 23,
    "files_modified": 5,
    "duration_minutes": 270
  },
  "tags": ["feature", "authentication", "in-progress"],
  "checkpoints": ["cp-001", "cp-002", "cp-003"]
}
```

---

## Context Checkpointing

### Understanding Checkpoints

Checkpoints are snapshots of your conversation state that allow you to "rewind" to previous points:

```
Session Timeline:

Start ───●───●───●───●───●───●───●───●───●───► Current
         │       │           │       │
         │       │           │       └── Checkpoint 4 (Current)
         │       │           │
         │       │           └── Checkpoint 3 (Before refactor)
         │       │
         │       └── Checkpoint 2 (After auth complete)
         │
         └── Checkpoint 1 (Initial setup)
```

### The Esc + Esc Rewind

The quickest way to rewind is the double-Escape shortcut:

```
┌─────────────────────────────────────────────────────────────┐
│                                                              │
│  Press Esc + Esc to rewind to the previous state            │
│                                                              │
│  Current state: Message #47 (after attempting refactor)     │
│                                                              │
│  [Esc] [Esc]                                                 │
│                                                              │
│  Rewinding... ◀◀                                            │
│                                                              │
│  Now at: Message #42 (before refactor attempt)              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**How to Use:**
1. Press `Esc` once - prepares for rewind
2. Press `Esc` again quickly - executes the rewind
3. You're now at the previous conversation state
4. Continue from this earlier point

### Conversation Branching

Rewinding allows you to explore different approaches:

```
Original Path:

Start ──► A ──► B ──► C ──► D (wrong direction)
                      │
                      │ [Rewind to C]
                      │
Branched Path:        └──► E ──► F ──► G (better approach)
```

**Example Workflow:**
```bash
# You're working on a feature
> Implement caching for the API responses

# Claude implements using Redis
# But you wanted in-memory caching

# Press Esc + Esc to rewind

# Now clarify your intent
> Implement in-memory caching using a simple LRU cache, not Redis

# Claude takes the different approach
```

### Undo/Redo Workflows

#### Simple Undo
```bash
# Made a mistake? Rewind!
# Press Esc + Esc

# You're back to the previous state
# Try again with a different approach
```

#### Multiple Undos
```bash
# Need to go back further?
# Press Esc + Esc multiple times

# First rewind: Back 1 step
# Second rewind: Back 2 steps
# Third rewind: Back 3 steps

# Each rewind takes you to a checkpoint
```

#### Redo (Continue Forward)
```bash
# After rewinding, if you want to go forward again:
# Simply continue the conversation

# The "undone" messages are still saved
# You can reference them if needed
```

### Recovering from Mistakes

#### Scenario: Unwanted File Changes
```bash
# Claude modified files incorrectly
> Refactor the entire codebase to use TypeScript
# Oops! You only wanted one file

# Press Esc + Esc to rewind

# Be more specific
> Convert only src/auth.js to TypeScript, keep other files as JavaScript
```

#### Scenario: Wrong Direction
```bash
# Claude started implementing the wrong pattern
> Implement state management
# Claude starts adding Redux, but you wanted Zustand

# Press Esc + Esc

> Implement state management using Zustand, not Redux
```

#### Scenario: Lost Context
```bash
# Claude seems to have forgotten important context
# Instead of re-explaining, rewind to where context was established

# Press Esc + Esc (multiple times if needed)

# Now continue from where context was clear
```

### Manual Checkpoint Creation

You can explicitly create checkpoints at important moments:

```bash
# After completing a significant piece of work
> Checkpoint: Authentication module complete

# Claude acknowledges and creates a named checkpoint
# This makes it easier to return to this exact state
```

### Checkpoint Best Practices

1. **Create checkpoints before risky changes**
   ```bash
   > Before we refactor the database layer, let's checkpoint here
   ```

2. **Create checkpoints at milestones**
   ```bash
   > Authentication is working. Let's checkpoint before moving to authorization.
   ```

3. **Name checkpoints descriptively**
   ```bash
   > Checkpoint: "API v1 complete - all tests passing"
   ```

---

## Managing Sessions

### Listing All Sessions

View your available sessions:

```bash
# In Claude Code, use the /sessions command (if available)
/sessions

# Or use /resume to see the session picker
/resume

# From command line (if supported)
claude sessions list
```

**Session List Output:**
```
┌──────────────────────────────────────────────────────────────────────────────┐
│ Sessions for: /home/user/webapp                                               │
├──────────────────────────────────────────────────────────────────────────────┤
│ Name                          │ Last Used      │ Messages │ Duration         │
├──────────────────────────────────────────────────────────────────────────────┤
│ feature-auth-jan15            │ 2 hours ago    │ 47       │ 4h 30m           │
│ bugfix-login-123              │ 1 day ago      │ 23       │ 1h 15m           │
│ refactor-api-v2               │ 3 days ago     │ 89       │ 8h 45m           │
│ explore-new-framework         │ 1 week ago     │ 15       │ 45m              │
│ initial-setup                 │ 2 weeks ago    │ 8        │ 20m              │
└──────────────────────────────────────────────────────────────────────────────┘
```

### Deleting Old Sessions

Remove sessions you no longer need:

```bash
# Delete a specific session
/session delete "old-session-name"

# Delete sessions older than 30 days
/session cleanup --older-than 30d

# Delete all sessions for a project
/session delete --project /old/project/path --all

# Interactive deletion
/session delete --interactive
```

**Confirmation Prompt:**
```
┌─────────────────────────────────────────────────────────────┐
│ Delete Session                                               │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│ Session: old-experiment-dec10                                │
│ Messages: 34                                                 │
│ Created: December 10, 2023                                   │
│ Last used: December 12, 2023                                 │
│                                                              │
│ This action cannot be undone.                                │
│                                                              │
│ [Delete] [Cancel]                                            │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Exporting Sessions

Export sessions for backup or sharing:

```bash
# Export to JSONL (default format)
/session export "feature-auth-jan15" --output ./exports/auth-session.jsonl

# Export to Markdown (readable format)
/session export "feature-auth-jan15" --format markdown --output ./exports/auth-session.md

# Export with context files
/session export "feature-auth-jan15" --include-context --output ./exports/auth-full/

# Export multiple sessions
/session export --search "feature-*" --output ./exports/all-features/
```

**Exported Markdown Format:**
```markdown
# Session: feature-auth-jan15

**Project:** /home/user/webapp
**Created:** January 15, 2024 10:00 AM
**Duration:** 4 hours 30 minutes
**Messages:** 47

---

## Conversation

### User (10:00 AM)
Help me implement user authentication for this Express app.

### Claude (10:00 AM)
I'll help you implement user authentication. Let me first examine your current project structure...

[Tool: read_file - /src/app.js]

I see you have a basic Express setup. Here's my plan for implementing authentication:
1. Add passport.js for authentication strategies
2. Create user model with secure password hashing
3. Implement login/logout routes
...
```

### Session Size and Cleanup

Monitor and manage session storage:

```bash
# Check total session storage size
/session stats

# Output:
# Total sessions: 47
# Total storage: 234 MB
# Largest session: refactor-api-v2 (45 MB)
# Average session: 5 MB
```

**Cleanup Strategies:**

```bash
# Remove sessions larger than 50MB
/session cleanup --larger-than 50MB

# Archive old sessions (compress but keep)
/session archive --older-than 60d

# Vacuum database (reclaim space)
/session vacuum

# Full cleanup routine
/session cleanup --older-than 90d --archive --vacuum
```

### Session Organization Commands

```bash
# Tag sessions for organization
/session tag "feature-auth-jan15" --add "completed"
/session tag "feature-auth-jan15" --add "authentication"

# Filter by tags
/resume --tag "in-progress"
/resume --tag "authentication"

# Archive completed work
/session archive --tag "completed"

# Bulk operations
/session bulk-tag --search "feature-*" --add "feature-work"
```

---

## Real-World Examples

### Example 1: Multi-Day Feature Development

**Scenario:** Building a user dashboard feature over three days

**Day 1: Initial Setup**
```bash
# Start fresh session
claude
/rename "feature-dashboard-sprint12"

# Begin work
> I need to build a user dashboard that shows activity, notifications,
> and account settings. Let's start with the component structure.

# Work for 3 hours on initial structure
# End of day - session auto-saves
```

**Day 2: Continue Implementation**
```bash
# Resume where you left off
claude
/resume "feature-dashboard-sprint12"

# Claude remembers everything from Day 1
> Let's continue with the notifications panel. Yesterday we set up
> the basic component structure.

# Claude recalls the structure and continues seamlessly
# Work on notifications for 4 hours
```

**Day 3: Testing and Polish**
```bash
# Resume again
claude
/resume "feature-dashboard-sprint12"

> Now let's add tests for the dashboard components and polish the UI.

# Complete the feature with full context
# Rename to indicate completion
/rename "feature-dashboard-sprint12-complete"
```

### Example 2: Bug Investigation Across Sessions

**Scenario:** Tracking down an intermittent bug

**Session 1: Initial Investigation**
```bash
claude
/rename "bug-memory-leak-investigation"

> Users are reporting the app slows down after 30 minutes of use.
> Help me investigate potential memory leaks.

# Analyze code, run diagnostics
# Narrow down to possible causes
# End session with hypotheses documented
```

**Session 2: Deep Dive**
```bash
claude
/resume "bug-memory-leak-investigation"

> Based on our analysis yesterday, let's focus on the event listener
> cleanup in the WebSocket module.

# Claude remembers previous analysis
# Deep dive into specific area
# Create checkpoints before changes
```

**Session 3: Fix and Verify**
```bash
claude
/resume "bug-memory-leak-investigation"

> Implement the fix we identified - proper cleanup of event listeners
> in the disconnect handler.

# Apply fix with full context
# Run verification tests
/rename "bug-memory-leak-fixed"
```

### Example 3: Code Review Spanning Days

**Scenario:** Reviewing a large PR over multiple sessions

**Initial Review**
```bash
claude
/rename "review-pr-456-new-api"

> Review PR #456 which adds a new REST API layer. Focus on
> architecture and security first.

# Review 30% of changes
# Note concerns and questions
# Auto-checkpoint before ending
```

**Continue Review**
```bash
claude
/resume "review-pr-456-new-api"

> Let's continue reviewing. Yesterday we covered the architecture.
> Now let's look at error handling and validation.

# Claude remembers previous concerns
# Continue systematic review
```

**Complete Review**
```bash
claude
/resume "review-pr-456-new-api"

> Summarize all our findings and generate the review feedback for
> the PR author.

# Generate comprehensive review
# All context from previous sessions included
```

### Example 4: Learning Session with Progress

**Scenario:** Learning a new framework over time

**Session 1: Basics**
```bash
claude
/rename "learn-rust-day1-basics"

> I'm learning Rust. Let's start with ownership and borrowing concepts.

# Work through examples
# Checkpoints at each concept mastered
```

**Session 2: Intermediate**
```bash
claude
/resume "learn-rust-day1-basics"
/rename "learn-rust-day2-lifetimes"

> Yesterday we covered ownership. Now let's tackle lifetimes.

# Build on previous knowledge
# Claude references earlier examples
```

**Session 3: Practice**
```bash
claude
/resume "learn-rust-day2-lifetimes"
/rename "learn-rust-day3-project"

> Let's apply what we learned by building a small CLI tool.

# Apply concepts from previous sessions
# Claude guides based on identified knowledge gaps
```

### Example 5: Team Handoff via Session Export

**Scenario:** Handing off work to a colleague

**Developer A's Work**
```bash
claude
/rename "backend-api-redesign-alice"

> Redesigning the backend API. Starting with authentication endpoints.

# Work for several days
# Document decisions in conversation
# Export when ready to hand off

/session export "backend-api-redesign-alice" \
  --format markdown \
  --include-context \
  --output ./handoff/api-redesign-context.md
```

**Developer B Continues**
```bash
# Developer B reads the exported session
# Starts new session with context

claude
/rename "backend-api-redesign-bob"

> I'm taking over the API redesign from Alice. I've read through
> the session export. Let's continue with the authorization layer.

# Reference the exported decisions
# Continue work with shared understanding
```

### Example 6: Debugging with Multiple Rewind Points

**Scenario:** Complex debugging requiring exploration

```bash
claude
/rename "debug-race-condition"

> There's a race condition in the order processing system.
> Let's investigate.

# Investigate initial hypothesis
> Let's check if it's in the payment processing...

# Checkpoint before testing fix
> Checkpoint: Before payment module changes

# Try fix A - doesn't work
# Rewind (Esc + Esc)

> Let's try a different approach - check the inventory update...

# Checkpoint before second attempt
> Checkpoint: Before inventory approach

# Try fix B - partial success
# Rewind to checkpoint

> Combine elements of both approaches...

# Finally solve with full history of attempts
```

### Example 7: Long Refactor with Checkpoint Saves

**Scenario:** Major codebase refactor

```bash
claude
/rename "refactor-monolith-to-services"

> We're breaking the monolith into microservices. Start with user service.

# Complete user service extraction
> Checkpoint: User service extracted and tested

# Continue with order service
> Checkpoint: Order service extracted and tested

# Work on notification service
# Encounter issue - rewind to last checkpoint

> Checkpoint: Before notification service issues

# Fix approach, continue
> Checkpoint: Notification service complete

# Final integration
> Checkpoint: All services integrated

# Each checkpoint represents a stable, tested state
# Can rewind to any point if needed
```

---

## Session Storage Details

### Detailed Storage Reference

| Location | Purpose | Format | Size Estimate |
|----------|---------|--------|---------------|
| `~/.claude/sessions.db` | Session index and metadata | SQLite | 1-10 MB |
| `~/.claude/projects/{hash}/transcripts/*.jsonl` | Full conversation history | JSONL | 100KB-50MB per session |
| `~/.claude/projects/{hash}/context/{id}/` | Saved context and state | JSON | 10KB-5MB per session |
| `~/.claude/projects/{hash}/CLAUDE.md` | Persistent project memory | Markdown | 1-100KB |
| `~/.claude/settings.json` | User preferences | JSON | 1-10KB |
| `~/.claude/logs/` | Debug and error logs | Text | 1-100MB |

### Platform-Specific Paths

**macOS:**
```bash
# Primary location
~/.claude/

# Full paths
/Users/{username}/.claude/sessions.db
/Users/{username}/.claude/projects/
/Users/{username}/.claude/settings.json
```

**Linux:**
```bash
# Primary location
~/.claude/

# Full paths
/home/{username}/.claude/sessions.db
/home/{username}/.claude/projects/
/home/{username}/.claude/settings.json

# XDG alternative (if configured)
$XDG_DATA_HOME/claude/
```

**Windows:**
```bash
# Primary location
C:\Users\{username}\.claude\

# Full paths
C:\Users\{username}\.claude\sessions.db
C:\Users\{username}\.claude\projects\
C:\Users\{username}\.claude\settings.json

# Alternative
%APPDATA%\claude\
```

### JSONL Transcript Deep Dive

**Complete Session Structure:**
```jsonl
{"version":"1.0","type":"session_init","timestamp":"2024-01-15T10:00:00Z","session_id":"abc123","project_path":"/home/user/webapp"}
{"type":"user_message","id":"msg-001","timestamp":"2024-01-15T10:00:05Z","content":"Help me implement authentication"}
{"type":"assistant_response","id":"msg-002","timestamp":"2024-01-15T10:00:08Z","content":"I'll help you implement authentication. Let me first examine your project...","thinking":"User wants auth implementation. Should check existing code first."}
{"type":"tool_use","id":"tool-001","timestamp":"2024-01-15T10:00:09Z","tool":"glob","input":{"pattern":"**/*.js"},"output":["src/app.js","src/routes/index.js"]}
{"type":"tool_use","id":"tool-002","timestamp":"2024-01-15T10:00:10Z","tool":"read","input":{"path":"/home/user/webapp/src/app.js"},"output":"const express = require('express')..."}
{"type":"assistant_response","id":"msg-003","timestamp":"2024-01-15T10:00:15Z","content":"I see you have a basic Express setup. Here's my plan..."}
{"type":"checkpoint","id":"cp-001","timestamp":"2024-01-15T10:30:00Z","name":"Before auth implementation","state_hash":"def456"}
{"type":"tool_use","id":"tool-003","timestamp":"2024-01-15T10:30:05Z","tool":"edit","input":{"path":"/home/user/webapp/src/app.js","old_string":"..","new_string":".."},"output":{"success":true}}
{"type":"session_end","timestamp":"2024-01-15T14:30:00Z","stats":{"messages":47,"tool_calls":23,"duration_seconds":16200}}
```

### SQLite Schema Reference

```sql
-- Main sessions table
CREATE TABLE sessions (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    project_path TEXT NOT NULL,
    project_hash TEXT NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    ended_at DATETIME,
    message_count INTEGER DEFAULT 0,
    tool_call_count INTEGER DEFAULT 0,
    is_archived BOOLEAN DEFAULT FALSE,
    is_deleted BOOLEAN DEFAULT FALSE
);

-- Checkpoints for rewind functionality
CREATE TABLE checkpoints (
    id TEXT PRIMARY KEY,
    session_id TEXT NOT NULL,
    name TEXT,
    message_index INTEGER NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    state_snapshot TEXT,
    FOREIGN KEY (session_id) REFERENCES sessions(id)
);

-- Full-text search for session content
CREATE VIRTUAL TABLE session_search USING fts5(
    session_id,
    content,
    tokenize='porter'
);

-- Session tags for organization
CREATE TABLE session_tags (
    session_id TEXT NOT NULL,
    tag TEXT NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (session_id, tag),
    FOREIGN KEY (session_id) REFERENCES sessions(id)
);

-- Indexes for performance
CREATE INDEX idx_sessions_project ON sessions(project_hash);
CREATE INDEX idx_sessions_updated ON sessions(updated_at DESC);
CREATE INDEX idx_sessions_name ON sessions(name);
CREATE INDEX idx_checkpoints_session ON checkpoints(session_id);
```

---

## Troubleshooting

### "Session Not Found" Errors

**Symptoms:**
```
Error: Session "feature-auth-jan15" not found
```

**Causes and Solutions:**

1. **Typo in session name**
   ```bash
   # Check exact name
   /resume
   # Use interactive picker to find correct name
   ```

2. **Session was deleted**
   ```bash
   # Sessions may have been cleaned up
   # Check if archived
   /session list --include-archived
   ```

3. **Different project context**
   ```bash
   # Sessions are project-specific
   # Ensure you're in the right directory
   pwd
   # Or specify project
   /resume --project /correct/path "session-name"
   ```

4. **Database corruption**
   ```bash
   # Try rebuilding the index
   /session rebuild-index
   ```

### Corrupted Session Recovery

**Symptoms:**
- Session loads but is incomplete
- Error messages about malformed data
- Missing messages in conversation

**Recovery Steps:**

1. **Try loading from JSONL directly**
   ```bash
   # JSONL files are the source of truth
   # Locate the transcript file
   ls ~/.claude/projects/*/transcripts/

   # The file contains your conversation
   # Even if SQLite index is corrupted
   ```

2. **Rebuild session index**
   ```bash
   /session rebuild-index
   # This re-scans JSONL files and rebuilds SQLite
   ```

3. **Manual recovery**
   ```bash
   # Export readable version
   /session export "corrupted-session" --format markdown

   # Start new session with context
   claude
   > I'm continuing from a previous session. Here's the context:
   > [paste relevant parts from export]
   ```

4. **Restore from backup**
   ```bash
   # If you have backups
   cp ~/backups/.claude/projects/*/transcripts/session.jsonl \
      ~/.claude/projects/*/transcripts/
   /session rebuild-index
   ```

### Session Won't Resume

**Symptoms:**
- `/resume` command hangs
- Session appears but won't load
- Errors about context or state

**Solutions:**

1. **Clear context cache**
   ```bash
   # Context files might be corrupted
   rm -rf ~/.claude/projects/*/context/problematic-session/
   /resume "session-name"
   ```

2. **Resume without context**
   ```bash
   /resume "session-name" --no-context
   # Loads messages but not saved state
   ```

3. **Check disk space**
   ```bash
   df -h ~/.claude/
   # Ensure sufficient space
   ```

4. **Check file permissions**
   ```bash
   ls -la ~/.claude/
   # Ensure read/write access
   chmod -R u+rw ~/.claude/
   ```

### Storage Space Issues

**Symptoms:**
- Disk full warnings
- Slow session operations
- New sessions won't save

**Solutions:**

1. **Check usage**
   ```bash
   du -sh ~/.claude/
   du -sh ~/.claude/projects/*/transcripts/
   ```

2. **Clean old sessions**
   ```bash
   /session cleanup --older-than 90d
   /session vacuum
   ```

3. **Archive large sessions**
   ```bash
   /session archive --larger-than 20MB
   # Compresses but keeps sessions
   ```

4. **Delete unnecessary sessions**
   ```bash
   /session delete --older-than 180d --force
   ```

5. **Move to larger disk**
   ```bash
   # Symlink .claude to different location
   mv ~/.claude /larger-disk/claude-data
   ln -s /larger-disk/claude-data ~/.claude
   ```

### Cross-Machine Session Sync

**Challenge:** Using Claude Code on multiple machines

**Solution 1: Cloud Sync (Dropbox/OneDrive/Google Drive)**
```bash
# Move .claude to synced folder
mv ~/.claude ~/Dropbox/claude-data

# Symlink on each machine
ln -s ~/Dropbox/claude-data ~/.claude
```

**Solution 2: Git Repository**
```bash
# Initialize git in .claude
cd ~/.claude
git init
git add -A
git commit -m "Initial session backup"

# Push to private repo
git remote add origin git@github.com:user/claude-sessions.git
git push -u origin main

# On other machines
git clone git@github.com:user/claude-sessions.git ~/.claude
```

**Solution 3: Manual Export/Import**
```bash
# Export on Machine A
/session export "important-session" --output ~/transfer/

# Copy to Machine B
scp ~/transfer/session.jsonl machineB:~/transfer/

# Import on Machine B
/session import ~/transfer/session.jsonl
```

**Caution:**
- Avoid editing the same session on multiple machines simultaneously
- Sync conflicts can corrupt sessions
- Use locking or coordinate access

### Common Error Messages

| Error | Cause | Solution |
|-------|-------|----------|
| `SQLITE_CORRUPT` | Database corruption | Run `/session rebuild-index` |
| `JSONL parse error at line N` | Malformed transcript | Edit JSONL to fix line N or truncate |
| `Session locked` | Concurrent access | Close other Claude instances |
| `Insufficient permissions` | File access denied | Check and fix file permissions |
| `Session too large to load` | Memory constraints | Load with `--no-context` flag |
| `Checkpoint not found` | Missing checkpoint data | Use a different checkpoint or skip |

---

## Quick Reference Cheat Sheet

### Session Commands

```bash
# Starting Sessions
claude                              # Start new session
claude -r                           # Resume recent session
claude --session "name"             # Start/resume named session

# Naming
/rename "new-name"                  # Rename current session
/rename "project-task-date"         # Recommended format

# Resuming
/resume                             # Interactive picker
/resume "session-name"              # Direct resume
/resume --recent                    # Resume last session
/resume --search "keyword"          # Search sessions

# Managing
/sessions                           # List all sessions
/session delete "name"              # Delete session
/session export "name"              # Export session
/session stats                      # Storage statistics
/session cleanup --older-than 30d   # Clean old sessions
```

### Keyboard Shortcuts

```
Esc + Esc       Rewind to previous state
Ctrl + C        Exit current session (saves automatically)
Ctrl + D        Exit (alternative)
Up/Down         Navigate session picker
Enter           Select session
/               Search in picker
```

### Session Naming Templates

```bash
# Feature work
/rename "feature-{name}-{date}"
/rename "feature-auth-jan15"

# Bug fixes
/rename "bugfix-{id}-{description}"
/rename "bugfix-123-login-error"

# Code reviews
/rename "review-pr{number}-{author}"
/rename "review-pr456-johndoe"

# Learning
/rename "learn-{topic}-{progress}"
/rename "learn-rust-ownership"

# Sprint work
/rename "sprint{n}-{task}"
/rename "sprint12-dashboard"
```

### Storage Locations

```bash
# Session database
~/.claude/sessions.db

# Transcripts
~/.claude/projects/{hash}/transcripts/{id}.jsonl

# Context
~/.claude/projects/{hash}/context/{id}/

# Project memory
~/.claude/projects/{hash}/CLAUDE.md

# Settings
~/.claude/settings.json
```

### Troubleshooting Quick Fixes

```bash
# Rebuild session index
/session rebuild-index

# Clear context cache
rm -rf ~/.claude/projects/*/context/

# Check storage
du -sh ~/.claude/

# Clean old sessions
/session cleanup --older-than 90d

# Vacuum database
/session vacuum
```

---

## Best Practices

### Naming Conventions for Teams

**Establish a Team Standard:**

```bash
# Agreed format: {ticket}-{type}-{developer}
# Examples:
/rename "PROJ-123-feature-alice"
/rename "PROJ-124-bugfix-bob"
/rename "PROJ-125-refactor-charlie"
```

**Benefits:**
- Easy to find sessions related to specific tickets
- Clear ownership for handoffs
- Consistent filtering and searching

**Team Naming Policy Template:**
```markdown
# Session Naming Convention

## Format
{PROJECT}-{TICKET_NUMBER}-{TYPE}-{DEVELOPER_INITIALS}

## Types
- feature: New functionality
- bugfix: Bug repairs
- refactor: Code improvements
- review: Code reviews
- explore: Investigation/research

## Examples
- WEBAPP-123-feature-JS
- WEBAPP-124-bugfix-MK
- WEBAPP-125-refactor-AL

## Rules
1. Always include ticket number for tracked work
2. Use lowercase, hyphens for separators
3. Keep names under 40 characters
```

### When to Create New vs Resume Sessions

**Create New Session When:**
- Starting a completely new task
- Context from previous work isn't relevant
- Working on a different project
- Previous session is getting too long (>100 messages)
- You want a fresh start without old context

**Resume Existing Session When:**
- Continuing work from previous day
- Following up on earlier investigation
- Claude needs context from previous discussion
- You're in the middle of a multi-step process
- You need to reference earlier decisions

**Decision Flowchart:**
```
Is this related to previous work?
├── No ──► Create new session
└── Yes ──► Is the previous session >100 messages?
            ├── Yes ──► Create new, reference old
            └── No ──► Resume previous session
```

### Session Cleanup Routines

**Weekly Cleanup:**
```bash
# Every Friday
/session cleanup --older-than 14d --interactive
/session vacuum
```

**Monthly Cleanup:**
```bash
# First of each month
/session archive --older-than 30d
/session cleanup --older-than 60d
/session stats  # Review storage usage
```

**Project Completion Cleanup:**
```bash
# When project/sprint ends
/session export --search "sprint12-*" --output ./archives/sprint12/
/session archive --search "sprint12-*"
```

**Automated Cleanup Script:**
```bash
#!/bin/bash
# cleanup-claude-sessions.sh

# Archive old sessions
claude --command "/session archive --older-than 60d"

# Delete very old sessions
claude --command "/session cleanup --older-than 180d"

# Vacuum database
claude --command "/session vacuum"

# Report
claude --command "/session stats"
```

### Backup Strategies

**Strategy 1: Regular Exports**
```bash
# Weekly backup script
#!/bin/bash
BACKUP_DIR=~/backups/claude-sessions/$(date +%Y-%m-%d)
mkdir -p $BACKUP_DIR

# Export all recent sessions
claude --command "/session export --after $(date -d '7 days ago' +%Y-%m-%d) --output $BACKUP_DIR/"

# Compress
tar -czf ~/backups/claude-sessions-$(date +%Y-%m-%d).tar.gz $BACKUP_DIR
```

**Strategy 2: Cloud Sync**
```bash
# Symlink to cloud storage
ln -s ~/.claude ~/Dropbox/claude-data

# Or use rclone for more control
rclone sync ~/.claude remote:claude-backup --exclude "logs/*"
```

**Strategy 3: Git Backup**
```bash
# Daily git backup
cd ~/.claude
git add -A
git commit -m "Session backup $(date +%Y-%m-%d)"
git push origin main
```

**Strategy 4: Full Directory Copy**
```bash
# Simple rsync backup
rsync -av ~/.claude/ /backup/location/claude-$(date +%Y%m%d)/
```

### Session Hygiene Tips

1. **Name sessions immediately** - Don't rely on default names
2. **Create checkpoints before major changes** - Easy recovery
3. **Review and clean monthly** - Prevent storage bloat
4. **Export important sessions** - Create portable backups
5. **Use consistent naming** - Makes searching easier
6. **Don't let sessions get too long** - Start fresh after ~100 messages
7. **Archive, don't delete** - You might need old context later

---

## Summary

Session management is a powerful feature of Claude Code that enables continuous, context-aware collaboration. By mastering sessions, you can:

1. **Maintain Continuity** - Pick up exactly where you left off
2. **Preserve Context** - Claude remembers your project, decisions, and preferences
3. **Recover Gracefully** - Rewind when things go wrong
4. **Organize Work** - Name and tag sessions for easy retrieval
5. **Collaborate** - Export and share session context with teammates

### Key Takeaways

- **Always name important sessions** using a consistent convention
- **Use `/resume`** to continue previous work with full context
- **Create checkpoints** before risky changes with explicit requests
- **Use `Esc + Esc`** to rewind when you need to try a different approach
- **Clean up regularly** to manage storage and keep sessions organized
- **Back up important sessions** through exports or directory backups

### Quick Start Commands

```bash
# Start and name a new session
claude
/rename "my-project-task-today"

# Resume your last session
claude
/resume --recent

# Find a specific session
/resume --search "authentication"

# Create a checkpoint before risky work
> Let's checkpoint here before we refactor

# Rewind if something goes wrong
# Press Esc + Esc

# Clean up old sessions monthly
/session cleanup --older-than 30d
```

With effective session management, Claude Code becomes not just an AI assistant, but a persistent collaborator that grows more helpful over time as it accumulates context about your projects, preferences, and working style.

---

## Appendix: Session Management Command Reference

### Complete Command List

| Command | Description | Example |
|---------|-------------|---------|
| `/rename "name"` | Rename current session | `/rename "feature-auth"` |
| `/resume` | Interactive session picker | `/resume` |
| `/resume "name"` | Resume specific session | `/resume "feature-auth"` |
| `/resume --recent` | Resume last session | `/resume --recent` |
| `/resume --search "query"` | Search sessions | `/resume --search "auth"` |
| `/sessions` | List all sessions | `/sessions` |
| `/session stats` | Storage statistics | `/session stats` |
| `/session export` | Export session | `/session export "name"` |
| `/session delete` | Delete session | `/session delete "name"` |
| `/session cleanup` | Clean old sessions | `/session cleanup --older-than 30d` |
| `/session archive` | Archive sessions | `/session archive "name"` |
| `/session vacuum` | Optimize database | `/session vacuum` |
| `/session rebuild-index` | Rebuild session index | `/session rebuild-index` |

### Keyboard Shortcuts Reference

| Shortcut | Action |
|----------|--------|
| `Esc + Esc` | Rewind to previous checkpoint |
| `Ctrl + C` | Exit session (auto-saves) |
| `Ctrl + D` | Exit session (alternative) |
| `Up Arrow` | Previous command/Navigate picker |
| `Down Arrow` | Next item in picker |
| `Enter` | Select/Execute |
| `/` | Search in picker |
| `Esc` | Cancel current operation |

---

*This tutorial is part of the Claude Code documentation series. For more tutorials, see the main documentation index.*
