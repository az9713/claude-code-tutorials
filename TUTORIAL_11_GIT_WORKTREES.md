# Tutorial 11: Git Worktrees with Claude Code

## Complete Guide to Parallel Development Using Git Worktrees

**Difficulty Level:** Intermediate
**Estimated Reading Time:** 45-60 minutes
**Hands-On Time:** 2-3 hours

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Core Concepts](#core-concepts)
4. [Creating Worktrees](#creating-worktrees)
5. [Claude Code Session Per Worktree](#claude-code-session-per-worktree)
6. [Parallel Development Workflow](#parallel-development-workflow)
7. [Managing Worktrees](#managing-worktrees)
8. [Templates and Scripts](#templates-and-scripts)
9. [Real-World Examples](#real-world-examples)
10. [Comparison with Other Approaches](#comparison-with-other-approaches)
11. [Troubleshooting](#troubleshooting)
12. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)
13. [Best Practices](#best-practices)
14. [Summary](#summary)

---

## Overview

### What Are Git Worktrees?

Git worktrees are a powerful yet often overlooked feature that allows you to have multiple working directories attached to a single Git repository. Instead of having just one checkout of your code, you can have several, each pointing to a different branch or commit.

Think of it this way: normally, your Git repository has one "working tree" - the directory where your files live and where you make changes. With git worktrees, you can create additional linked working directories, each with its own checked-out branch, while sharing the same `.git` data.

```
Traditional Git Setup:
my-project/
├── .git/              # Repository data
├── src/               # Working tree (one branch at a time)
├── tests/
└── package.json

With Worktrees:
my-project/            # Main worktree (main branch)
├── .git/
├── src/
└── ...

my-project-feature-a/  # Linked worktree (feature-a branch)
├── .git -> ../my-project/.git
├── src/
└── ...

my-project-hotfix/     # Linked worktree (hotfix branch)
├── .git -> ../my-project/.git
├── src/
└── ...
```

### Why Use Worktrees with Claude Code?

When combining Git worktrees with Claude Code, you unlock powerful parallel development capabilities:

1. **True Session Isolation**: Each worktree gets its own Claude Code session with independent context, conversation history, and project understanding.

2. **Parallel AI-Assisted Development**: Work on multiple features simultaneously with separate Claude instances helping on each.

3. **Context Switching Without Cost**: No need to stash changes, commit partial work, or lose Claude's context when switching tasks.

4. **Clean Comparison Environment**: Test different AI-suggested implementations side by side.

5. **Safe Experimentation**: Try radical changes in an isolated worktree without risking your main work.

6. **Review Without Disruption**: Review PRs in a separate worktree while keeping your feature work untouched.

### The Power of Isolation

Consider this scenario: You're deep into implementing a new authentication system with Claude Code's help. The AI has built up context about your auth flow, user model, and security requirements. Suddenly, a critical bug is reported in production.

**Without worktrees:**
- Stash your auth changes
- Switch branches
- Lose Claude's authentication context
- Fix the bug
- Switch back
- Pop your stash
- Try to rebuild context with Claude

**With worktrees:**
- Open a new terminal
- Navigate to your hotfix worktree
- Start a fresh Claude session focused on the bug
- Fix it, commit, push
- Return to your auth worktree - Claude's context is exactly where you left it

---

## Prerequisites

### Git Version Requirements

Git worktrees were introduced in Git 2.5 (July 2015), but we recommend Git 2.20 or later for the best experience.

Check your Git version:

```bash
git --version
```

Expected output:
```
git version 2.40.0
```

If you need to upgrade:

**Windows (Git for Windows):**
```bash
# Download from https://git-scm.com/download/win
# Or use winget:
winget install Git.Git
```

**macOS:**
```bash
brew install git
# or
brew upgrade git
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt update
sudo apt install git
```

**Linux (Fedora/RHEL):**
```bash
sudo dnf install git
```

### Claude Code Setup

Ensure Claude Code is installed and configured:

```bash
# Verify installation
claude --version

# Login if needed
claude login

# Test basic functionality
claude "Hello, confirm you're working"
```

### System Requirements

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| Disk Space | 2x repo size per worktree | 3x for comfortable work |
| RAM | 4GB | 8GB+ for multiple Claude sessions |
| CPU | 2 cores | 4+ cores for parallel work |

### Knowledge Prerequisites

Before diving in, you should be comfortable with:

- Basic Git operations (clone, commit, push, pull)
- Git branching and merging
- Command-line navigation
- Running Claude Code in a project

---

## Core Concepts

### Understanding the Working Tree

In Git terminology, the "working tree" (or "work tree") is the directory containing the actual files you edit. It's distinct from:

- **The Git directory (.git)**: Contains all version control data
- **The staging area (index)**: Holds files prepared for the next commit
- **The repository**: The collection of all commits and branches

```
┌─────────────────────────────────────────────────┐
│                 Git Repository                   │
│  ┌─────────────────────────────────────────┐    │
│  │            .git directory               │    │
│  │  ┌─────────┐  ┌─────────┐  ┌────────┐  │    │
│  │  │ objects │  │  refs   │  │ config │  │    │
│  │  └─────────┘  └─────────┘  └────────┘  │    │
│  └─────────────────────────────────────────┘    │
│                      ▲                          │
│                      │ references               │
│  ┌─────────────────────────────────────────┐    │
│  │           Working Tree                  │    │
│  │   src/  tests/  package.json  ...       │    │
│  └─────────────────────────────────────────┘    │
└─────────────────────────────────────────────────┘
```

### Linked Worktrees Explained

When you create a linked worktree, Git creates a new directory that:

1. Has its own working tree (your files)
2. Has its own index (staging area)
3. **Shares** the `.git` directory with the main worktree
4. **Shares** the object database (commits, blobs, trees)
5. Has its own `HEAD` (current branch pointer)

```
Main Worktree                    Linked Worktree
my-project/                      my-project-feature/
├── .git/           ◄──────────► .git (file, points to main)
│   ├── objects/    (shared)     ├── src/
│   ├── refs/       (shared)     ├── tests/
│   ├── config      (shared)     └── package.json
│   └── worktrees/
│       └── my-project-feature/
│           ├── HEAD
│           ├── index
│           └── gitdir
├── src/
├── tests/
└── package.json
```

The linked worktree's `.git` is actually a file (not a directory) containing a path reference:

```bash
cat my-project-feature/.git
# Output: gitdir: /path/to/my-project/.git/worktrees/my-project-feature
```

### Session Isolation with Claude Code

Each worktree directory is independent, which means:

1. **Separate Claude Sessions**: When you run `claude` in different worktrees, you get distinct sessions.

2. **Independent CLAUDE.md Files**: Each worktree can have its own project context file.

3. **Isolated Conversation History**: Changes discussed in one session don't affect others.

4. **Different Project States**: Claude sees the files as they exist in that specific worktree.

```
Worktree A (feature-auth)          Worktree B (feature-payments)
┌────────────────────────┐        ┌────────────────────────┐
│ Claude Session A       │        │ Claude Session B       │
│ ┌────────────────────┐ │        │ ┌────────────────────┐ │
│ │ Context: auth flow │ │        │ │ Context: payment   │ │
│ │ Files: auth/*.ts   │ │        │ │ Files: payment/*.ts│ │
│ │ History: auth work │ │        │ │ History: pay work  │ │
│ └────────────────────┘ │        │ └────────────────────┘ │
│                        │        │                        │
│ CLAUDE.md:             │        │ CLAUDE.md:             │
│ "Working on OAuth2"    │        │ "Stripe integration"   │
└────────────────────────┘        └────────────────────────┘
```

### How Git Tracks Worktrees

Git maintains worktree information in the main repository's `.git/worktrees/` directory:

```
.git/worktrees/
├── feature-auth/
│   ├── HEAD          # Points to feature-auth branch
│   ├── ORIG_HEAD     # Previous HEAD for recovery
│   ├── index         # Staging area for this worktree
│   ├── gitdir        # Path to the worktree's .git file
│   └── locked        # Present if worktree is locked
└── hotfix-123/
    ├── HEAD
    ├── index
    └── gitdir
```

---

## Creating Worktrees

### Basic Worktree Creation

The fundamental command for creating a worktree is:

```bash
git worktree add <path> <branch>
```

**Example: Check out an existing branch in a new worktree:**

```bash
# From your main project directory
cd ~/projects/my-app

# Create a worktree for an existing branch
git worktree add ../my-app-feature-auth feature-auth
```

This creates:
- A new directory `../my-app-feature-auth`
- Checks out the `feature-auth` branch in that directory
- Links it to your main repository

### Creating a New Branch with a Worktree

More commonly, you'll want to create a new branch and worktree simultaneously:

```bash
git worktree add -b <new-branch> <path> <start-point>
```

**Example: Create a new feature branch from main:**

```bash
# Create new branch 'feature-payments' from 'main' in new worktree
git worktree add -b feature-payments ../my-app-payments main
```

**Example: Create a hotfix branch from production:**

```bash
git worktree add -b hotfix/login-bug ../hotfix-login production
```

### Directory Structure Best Practices

#### Approach 1: Sibling Directories (Recommended)

Place worktrees as siblings to your main project:

```
~/projects/
├── my-app/              # Main worktree (main branch)
├── my-app-feature-a/    # Feature A worktree
├── my-app-feature-b/    # Feature B worktree
└── my-app-hotfix/       # Hotfix worktree
```

**Advantages:**
- Easy to navigate with `cd ../my-app-feature-a`
- Clear visual separation in file explorer
- IDE can open each as separate project

#### Approach 2: Worktrees Subdirectory

Create a dedicated worktrees folder:

```
~/projects/
├── my-app/              # Main worktree
└── my-app-worktrees/
    ├── feature-a/
    ├── feature-b/
    └── hotfix/
```

**Advantages:**
- Keeps worktrees organized together
- Main project directory stays clean
- Easy cleanup (delete entire worktrees folder)

#### Approach 3: Branch-Based Naming

Name directories after their branches:

```
~/projects/
├── my-app/
├── feature-user-auth/
├── feature-payment-flow/
└── hotfix-session-timeout/
```

### Naming Conventions

Adopt consistent naming for clarity:

**Pattern 1: Project-Purpose**
```bash
git worktree add ../myapp-auth feature/auth
git worktree add ../myapp-payments feature/payments
git worktree add ../myapp-hotfix hotfix/bug-123
```

**Pattern 2: Purpose-Ticket**
```bash
git worktree add ../feature-AUTH-123 feature/AUTH-123
git worktree add ../hotfix-BUG-456 hotfix/BUG-456
```

**Pattern 3: Short Descriptive**
```bash
git worktree add ../auth feature/user-authentication
git worktree add ../pay feature/payment-integration
```

### Complete Creation Examples

**Example 1: New feature from main**
```bash
cd ~/projects/my-app

# Create worktree with new branch
git worktree add -b feature/user-dashboard ../my-app-dashboard main

# Navigate to it
cd ../my-app-dashboard

# Verify branch
git branch
# * feature/user-dashboard
```

**Example 2: Existing remote branch**
```bash
# Fetch latest from remote
git fetch origin

# Create worktree for existing remote branch
git worktree add ../my-app-review origin/feature/api-refactor

# Or track it properly
git worktree add --track -b feature/api-refactor ../my-app-review origin/feature/api-refactor
```

**Example 3: Specific commit or tag**
```bash
# Worktree at a specific tag (detached HEAD)
git worktree add ../my-app-v1.0 v1.0.0

# Worktree at specific commit
git worktree add ../my-app-debug abc123f
```

---

## Claude Code Session Per Worktree

### Running Claude in Each Worktree

Each worktree is a completely independent directory, so starting Claude there creates a fresh session:

```bash
# Terminal 1: Main worktree
cd ~/projects/my-app
claude
# Claude starts with context of main branch

# Terminal 2: Feature worktree
cd ~/projects/my-app-feature-auth
claude
# Claude starts with context of feature-auth branch
```

### Initializing Project Context with /init

When you start Claude in a new worktree, initialize project context:

```bash
cd ../my-app-feature-auth
claude
```

Inside Claude:
```
/init
```

Claude will:
1. Analyze the project structure
2. Create or update CLAUDE.md
3. Build understanding of the codebase
4. Note any existing conventions

**Customizing CLAUDE.md per Worktree:**

You might want different context in different worktrees:

```markdown
# CLAUDE.md in main worktree
## Project: My App (Main Development)
Focus: General development and integration
Current sprint goals: ...

# CLAUDE.md in feature-auth worktree
## Project: My App (Authentication Feature)
Focus: Implementing OAuth2 authentication
Key files: src/auth/*, src/middleware/auth.ts
External docs: https://oauth.net/2/
Current status: Implementing token refresh
```

### Session Naming with /rename

Give each session a meaningful name for easy identification:

```bash
# In feature-auth worktree
claude
/rename "auth-feature-dev"

# In payments worktree
claude
/rename "payments-integration"

# In hotfix worktree
claude
/rename "hotfix-login-bug"
```

Benefits of named sessions:
- Quickly identify sessions in process lists
- Meaningful session history
- Team communication clarity

### Independent Conversation History

Each worktree maintains its own conversation history:

```
my-app-feature-auth/.claude/
└── conversations/
    └── auth-feature-dev/
        ├── 2024-01-15_session.json
        └── 2024-01-16_session.json

my-app-payments/.claude/
└── conversations/
    └── payments-integration/
        ├── 2024-01-15_session.json
        └── 2024-01-16_session.json
```

This means:
- Discussions about auth don't pollute payments context
- You can resume exactly where you left off in each worktree
- Claude's understanding stays focused on the relevant feature

### Session Management Commands

Useful commands for managing sessions across worktrees:

```bash
# List active Claude sessions (all terminals)
ps aux | grep claude

# Resume a specific session
claude --resume auth-feature-dev

# Start fresh in a worktree (ignore history)
claude --no-history

# Continue most recent session
claude --continue
```

### Multi-Terminal Setup

For efficient parallel development, set up multiple terminals:

**Using tmux:**
```bash
# Create new tmux session with panes
tmux new-session -s dev \; \
  split-window -h \; \
  split-window -v \; \
  select-pane -t 0 \; \
  split-window -v

# Pane 0: Main worktree
# Pane 1: Feature worktree A
# Pane 2: Feature worktree B
# Pane 3: Tests/commands
```

**Using VS Code:**
1. Open each worktree as a separate VS Code window
2. Use VS Code's integrated terminal
3. Run Claude in each terminal

**Using Windows Terminal:**
```powershell
# Open multiple tabs, each in different worktree
wt -d ~/projects/my-app `; new-tab -d ~/projects/my-app-feature `; new-tab -d ~/projects/my-app-hotfix
```

---

## Parallel Development Workflow

### Working on Multiple Features Simultaneously

The true power of worktrees with Claude Code emerges when developing multiple features in parallel.

**Scenario: Building Auth and Payments Features**

```bash
# Setup (one-time)
cd ~/projects/my-app
git worktree add -b feature/auth ../my-app-auth main
git worktree add -b feature/payments ../my-app-payments main

# Terminal 1: Auth work
cd ~/projects/my-app-auth
claude
/init
/rename "auth-session"
```

In the auth session:
```
> I'm implementing OAuth2 authentication with Google and GitHub providers.
  Let's start by creating the auth service structure.
```

```bash
# Terminal 2: Payments work
cd ~/projects/my-app-payments
claude
/init
/rename "payments-session"
```

In the payments session:
```
> I'm integrating Stripe for payment processing.
  We need subscription management and one-time payments.
```

Now you have two completely independent development tracks:
- Auth session builds understanding of auth flow, providers, tokens
- Payments session builds understanding of Stripe API, webhooks, subscriptions

### Switching Between Worktrees

Physical switching is simple - just change directories:

```bash
# Switch to auth worktree
cd ~/projects/my-app-auth

# Check what branch you're on
git branch
# * feature/auth

# Switch to payments worktree
cd ~/projects/my-app-payments

# Check branch
git branch
# * feature/payments
```

**Key Insight:** Your Claude sessions remain active in each terminal. Switching directories doesn't affect them.

### Comparing Implementations Across Worktrees

Use worktrees to compare different approaches:

**Scenario: Evaluating Two Cache Implementations**

```bash
# Create two worktrees from same starting point
git worktree add -b experiment/redis-cache ../cache-redis main
git worktree add -b experiment/memcached ../cache-memcached main
```

In redis worktree with Claude:
```
> Let's implement a caching layer using Redis.
  Focus on connection pooling and automatic reconnection.
```

In memcached worktree with Claude:
```
> Let's implement a caching layer using Memcached.
  Focus on consistent hashing and failover.
```

After implementation, compare:

```bash
# Compare the implementations
diff -r ~/projects/cache-redis/src/cache ~/projects/cache-memcached/src/cache

# Or use a visual diff tool
code --diff ~/projects/cache-redis/src/cache/client.ts ~/projects/cache-memcached/src/cache/client.ts
```

### Synchronizing Changes Between Worktrees

Sometimes you need to share changes between worktrees:

**Method 1: Cherry-pick specific commits**
```bash
# In feature-b worktree, get a commit from feature-a
cd ~/projects/my-app-feature-b
git cherry-pick <commit-hash-from-feature-a>
```

**Method 2: Merge from shared branch**
```bash
# Both features need a utility from main
cd ~/projects/my-app-feature-a
git merge main  # Get latest utilities

cd ~/projects/my-app-feature-b
git merge main  # Same utilities now available
```

**Method 3: Create shared branch**
```bash
# In main worktree
cd ~/projects/my-app
git checkout -b shared/utilities main

# Add shared code, commit, push
git push -u origin shared/utilities

# In each feature worktree
cd ~/projects/my-app-feature-a
git merge shared/utilities
```

### Workflow Example: Complete Feature Development

```bash
# 1. Create feature worktree
cd ~/projects/my-app
git worktree add -b feature/user-profiles ../profiles main
cd ../profiles

# 2. Start Claude session
claude
/init
/rename "profiles-feature"

# 3. Develop with Claude's help
> Let's create a user profiles feature with avatar upload,
  bio editing, and social links.

# 4. Regular commits as you work
git add -A
git commit -m "Add profile model and basic routes"

# 5. Keep branch updated with main
git fetch origin
git rebase origin/main

# 6. Push for review
git push -u origin feature/user-profiles

# 7. After merge, cleanup
cd ~/projects/my-app
git worktree remove ../profiles
```

---

## Managing Worktrees

### Viewing All Worktrees

List all worktrees connected to your repository:

```bash
git worktree list
```

Output:
```
/home/user/projects/my-app           abc1234 [main]
/home/user/projects/my-app-auth      def5678 [feature/auth]
/home/user/projects/my-app-payments  ghi9012 [feature/payments]
```

**Verbose output:**
```bash
git worktree list --porcelain
```

Output:
```
worktree /home/user/projects/my-app
HEAD abc1234567890abcdef1234567890abcdef123456
branch refs/heads/main

worktree /home/user/projects/my-app-auth
HEAD def5678901234567890abcdef1234567890abcdef
branch refs/heads/feature/auth

worktree /home/user/projects/my-app-payments
HEAD ghi9012345678901234567890abcdef1234567890
branch refs/heads/feature/payments
```

### Removing Worktrees

When you're done with a worktree, clean it up properly:

```bash
# Remove a worktree (directory must be clean)
git worktree remove ../my-app-feature-auth

# Force remove (discards changes)
git worktree remove --force ../my-app-feature-auth
```

**What happens during removal:**
1. Git deletes the worktree directory
2. Git removes tracking info from `.git/worktrees/`
3. The branch remains (not deleted)

**To also delete the branch:**
```bash
git worktree remove ../my-app-feature-auth
git branch -d feature/auth  # Delete if merged
git branch -D feature/auth  # Force delete
```

### Pruning Stale Worktrees

If worktree directories are manually deleted, Git still tracks them:

```bash
# See what would be pruned
git worktree prune --dry-run

# Actually prune stale entries
git worktree prune

# Verbose output
git worktree prune -v
```

**When to prune:**
- After manually deleting worktree directories
- After system crashes
- Periodically as maintenance

### Handling Locked Worktrees

Git can lock worktrees to prevent accidental removal:

**Lock a worktree:**
```bash
git worktree lock ../my-app-important

# With a reason
git worktree lock --reason "Long-running experiment, don't delete" ../my-app-experiment
```

**Check lock status:**
```bash
git worktree list
# Shows [locked] for locked worktrees
```

**Unlock a worktree:**
```bash
git worktree unlock ../my-app-important
```

**Why lock worktrees:**
- Prevent accidental deletion of important work
- Mark worktrees with uncommitted experiments
- Protect long-running development branches

### Moving Worktrees

If you need to relocate a worktree:

```bash
# Move worktree to new location
git worktree move ../my-app-feature ../new-location/my-app-feature
```

**Note:** This only works if the worktree is clean (no uncommitted changes).

### Repairing Worktrees

If worktrees get corrupted or paths change:

```bash
# Repair all worktrees
git worktree repair

# Repair specific worktrees
git worktree repair ../my-app-feature ../my-app-hotfix
```

---

## Templates and Scripts

### Template: Feature Worktree Creation

```bash
#!/bin/bash
# create-feature-worktree.sh
# Usage: ./create-feature-worktree.sh feature-name [base-branch]

FEATURE_NAME=$1
BASE_BRANCH=${2:-main}
WORKTREE_PATH="../${FEATURE_NAME}"

if [ -z "$FEATURE_NAME" ]; then
    echo "Usage: $0 <feature-name> [base-branch]"
    exit 1
fi

# Ensure we're in a git repository
if ! git rev-parse --git-dir > /dev/null 2>&1; then
    echo "Error: Not in a git repository"
    exit 1
fi

# Fetch latest
git fetch origin

# Create worktree with new branch
git worktree add -b "feature/${FEATURE_NAME}" "${WORKTREE_PATH}" "${BASE_BRANCH}"

# Navigate to new worktree
cd "${WORKTREE_PATH}"

# Initialize Claude session
echo "Starting Claude session..."
echo "Run '/init' to initialize project context"
echo "Run '/rename ${FEATURE_NAME}-session' to name your session"

# Start Claude
claude
```

### Template: Hotfix Worktree Creation

```bash
#!/bin/bash
# create-hotfix-worktree.sh
# Usage: ./create-hotfix-worktree.sh issue-id [production-branch]

ISSUE_ID=$1
PROD_BRANCH=${2:-production}
WORKTREE_PATH="../hotfix-${ISSUE_ID}"

if [ -z "$ISSUE_ID" ]; then
    echo "Usage: $0 <issue-id> [production-branch]"
    exit 1
fi

# Fetch latest
git fetch origin

# Create worktree from production
git worktree add -b "hotfix/${ISSUE_ID}" "${WORKTREE_PATH}" "origin/${PROD_BRANCH}"

cd "${WORKTREE_PATH}"

echo "Hotfix worktree created at ${WORKTREE_PATH}"
echo "Branch: hotfix/${ISSUE_ID}"
echo "Based on: ${PROD_BRANCH}"
echo ""
echo "Quick commands:"
echo "  cd ${WORKTREE_PATH}"
echo "  claude"
echo "  /rename hotfix-${ISSUE_ID}"
```

### Template: PR Review Worktree

```bash
#!/bin/bash
# review-pr.sh
# Usage: ./review-pr.sh pr-number

PR_NUMBER=$1

if [ -z "$PR_NUMBER" ]; then
    echo "Usage: $0 <pr-number>"
    exit 1
fi

WORKTREE_PATH="../review-pr-${PR_NUMBER}"

# Fetch PR branch
git fetch origin "pull/${PR_NUMBER}/head:pr-${PR_NUMBER}"

# Create worktree
git worktree add "${WORKTREE_PATH}" "pr-${PR_NUMBER}"

cd "${WORKTREE_PATH}"

echo "Review worktree created at ${WORKTREE_PATH}"
echo ""
echo "Start Claude for AI-assisted review:"
echo "  claude"
echo "  /rename pr-${PR_NUMBER}-review"
echo ""
echo "When done:"
echo "  cd .."
echo "  git worktree remove ${WORKTREE_PATH}"
echo "  git branch -D pr-${PR_NUMBER}"
```

### Template: Cleanup Script

```bash
#!/bin/bash
# cleanup-worktrees.sh
# Safely remove merged feature worktrees

echo "Current worktrees:"
git worktree list
echo ""

# Get list of worktrees (excluding main)
WORKTREES=$(git worktree list --porcelain | grep "^worktree" | cut -d' ' -f2 | grep -v "$(pwd)")

for WT in $WORKTREES; do
    BRANCH=$(git -C "$WT" branch --show-current)

    # Check if branch is merged into main
    if git branch --merged main | grep -q "$BRANCH"; then
        echo "Worktree $WT (branch: $BRANCH) is merged into main"
        read -p "Remove this worktree? (y/n) " -n 1 -r
        echo
        if [[ $REPLY =~ ^[Yy]$ ]]; then
            git worktree remove "$WT"
            git branch -d "$BRANCH"
            echo "Removed worktree and branch"
        fi
    fi
done

# Prune any stale entries
git worktree prune -v
```

### Template: Quick Worktree + Claude Start

```bash
# Add to your .bashrc or .zshrc

# Quick feature start
newfeature() {
    local name=$1
    local base=${2:-main}
    git worktree add -b "feature/$name" "../$name" "$base" && \
    cd "../$name" && \
    claude
}

# Quick hotfix start
hotfix() {
    local id=$1
    git worktree add -b "hotfix/$id" "../hotfix-$id" production && \
    cd "../hotfix-$id" && \
    claude
}

# List worktrees with status
wt() {
    git worktree list
}

# Usage:
# newfeature user-auth
# hotfix BUG-123
# wt
```

---

## Real-World Examples

### Example 1: Parallel Feature Development (Auth + Payments)

**Scenario:** You're building an e-commerce platform and need to implement both authentication and payment processing. These are large features that will take several days each.

**Setup:**
```bash
cd ~/projects/ecommerce
git worktree add -b feature/oauth-auth ../ecommerce-auth main
git worktree add -b feature/stripe-payments ../ecommerce-payments main
```

**Auth Development (Terminal 1):**
```bash
cd ~/projects/ecommerce-auth
claude
/init
/rename "oauth-implementation"
```

Conversation with Claude:
```
> I need to implement OAuth2 authentication with Google, GitHub, and
  Microsoft providers. Let's start with the provider abstraction layer.

Claude: I'll help you build a flexible OAuth2 implementation. Let's start
with a provider interface that can handle multiple OAuth providers...
[Claude provides implementation]

> Now let's add the token refresh logic and session management.

Claude: Building on the provider abstraction, here's how we can handle
token refresh with automatic retry...
[Development continues with full context]
```

**Payments Development (Terminal 2):**
```bash
cd ~/projects/ecommerce-payments
claude
/init
/rename "stripe-integration"
```

Conversation with Claude:
```
> I need to integrate Stripe for payments. We need subscriptions,
  one-time payments, and webhook handling.

Claude: I'll help you build a comprehensive Stripe integration. Let's
start with the payment service architecture...
[Claude provides implementation]

> The webhook signature verification is failing in tests.

Claude: Looking at your webhook handler, the issue is likely with how
we're accessing the raw request body. Here's the fix...
[Debug continues with payment-specific context]
```

**Key Benefits:**
- Each Claude session maintains deep context for its feature
- No context switching overhead
- Can work on auth in the morning, payments in the afternoon
- Both sessions remember previous discussions

---

### Example 2: Hotfix While Feature in Progress

**Scenario:** You're mid-way through a complex refactoring when a critical production bug is reported.

**Initial State:**
```bash
# You're in the middle of a refactor
cd ~/projects/app
# Feature branch has uncommitted changes
git status
# Modified: src/core/engine.ts (500+ lines changed)
```

**Without worktrees (painful):**
```bash
git stash push -m "Refactor in progress"  # Hope nothing breaks
git checkout production
git checkout -b hotfix/critical-bug
# Fix bug... but you've lost all refactor context
git checkout feature/refactor
git stash pop  # Merge conflicts!
```

**With worktrees (smooth):**
```bash
# Don't touch your current work at all!
# Just create a hotfix worktree
git worktree add -b hotfix/critical-bug ../app-hotfix production

# New terminal
cd ~/projects/app-hotfix
claude
/rename "critical-hotfix"
```

Hotfix conversation with Claude:
```
> Production users are getting 500 errors on checkout.
  Error log shows: "Cannot read property 'id' of undefined"
  in OrderProcessor.ts line 234.

Claude: Let me examine the OrderProcessor. The issue appears to be
a race condition where the order object isn't fully populated...
[Claude helps identify and fix the bug]
```

Meanwhile, in your original terminal:
```bash
# Your refactor work is exactly where you left it
# Claude session still has full refactor context
```

After hotfix is deployed:
```bash
git worktree remove ../app-hotfix
git branch -d hotfix/critical-bug  # Merged to production
```

---

### Example 3: Code Review in Separate Worktree

**Scenario:** A teammate submitted a PR for review. You want to run the code, explore it with Claude's help, but keep your current work intact.

**Setup:**
```bash
# Fetch the PR
git fetch origin pull/142/head:pr-142

# Create review worktree
git worktree add ../review-142 pr-142
cd ../review-142
```

**Review with Claude:**
```bash
claude
/init
/rename "pr-142-review"
```

Conversation:
```
> I'm reviewing PR #142 which adds a caching layer. Please analyze
  the implementation for potential issues.

Claude: I'll review the caching implementation. Let me examine the
key files...

Looking at src/cache/CacheManager.ts, I notice several concerns:
1. No TTL validation on cache entries
2. Memory cache has no size limit
3. Redis connection isn't properly pooled
...

> Can you suggest specific improvements for the TTL handling?

Claude: Here's how the TTL handling could be improved...
[Claude provides specific code suggestions]
```

**Run tests in review environment:**
```bash
npm install
npm test
npm run lint
```

**Leave review feedback with full context, then cleanup:**
```bash
cd ~/projects/app
git worktree remove ../review-142
git branch -D pr-142
```

---

### Example 4: A/B Testing Implementations

**Scenario:** You need to evaluate two different approaches to implementing a search feature: Elasticsearch vs. Algolia.

**Setup:**
```bash
cd ~/projects/app
git worktree add -b experiment/elasticsearch ../search-elastic main
git worktree add -b experiment/algolia ../search-algolia main
```

**Elasticsearch Implementation:**
```bash
cd ~/projects/search-elastic
claude
/rename "elasticsearch-experiment"
```

```
> Let's implement full-text search using Elasticsearch. Focus on
  relevance tuning and fuzzy matching.

Claude: I'll help implement Elasticsearch search. For e-commerce
products, we'll want to configure custom analyzers and field boosting...
[Implementation proceeds]
```

**Algolia Implementation:**
```bash
cd ~/projects/search-algolia
claude
/rename "algolia-experiment"
```

```
> Let's implement full-text search using Algolia. Focus on
  relevance tuning and instant search.

Claude: Algolia has a different approach - it's optimized for
instant search with typo tolerance built-in...
[Implementation proceeds]
```

**Compare Results:**
```bash
# Run benchmarks in each
cd ~/projects/search-elastic && npm run benchmark
cd ~/projects/search-algolia && npm run benchmark

# Compare implementations
diff -r ~/projects/search-elastic/src/search ~/projects/search-algolia/src/search
```

---

### Example 5: Long-Running Refactor Isolation

**Scenario:** You're undertaking a major refactor (migrating from REST to GraphQL) that will take weeks, while still needing to support the current codebase.

**Setup:**
```bash
cd ~/projects/api
git worktree add -b refactor/graphql-migration ../api-graphql main
```

**Refactor Worktree (dedicated work):**
```bash
cd ~/projects/api-graphql
claude
/rename "graphql-migration"
```

Over several weeks:
```
> Let's continue the GraphQL migration. Last session we converted
  the User resolvers. Now let's tackle Orders.

Claude: Continuing from our User resolver work, I can see we
established patterns for authentication and pagination. Let's
apply those same patterns to Orders...
[Claude remembers context from previous sessions]
```

**Main Worktree (regular maintenance):**
```bash
cd ~/projects/api
# Continue normal bug fixes and small features
# These can be merged to main without affecting graphql work
```

**Periodic sync:**
```bash
cd ~/projects/api-graphql
git fetch origin
git rebase origin/main
# Resolve any conflicts, continue refactor
```

---

### Example 6: PR Review Without Stashing

**Scenario:** You're in the middle of writing tests when asked to urgently review a PR.

**Current State:**
```bash
cd ~/projects/app
git status
# Modified: tests/unit/UserService.test.ts
# Modified: tests/integration/auth.test.ts
# New file: tests/fixtures/users.json
```

**Quick Review Setup:**
```bash
# Don't stash! Create review worktree
git worktree add ../quick-review origin/feature/new-api
cd ../quick-review
```

**Review Process:**
```bash
claude
/rename "quick-review"

> Please review the changes in this PR for the new API endpoints.
  Focus on error handling and input validation.
```

```bash
# Run the code
npm test
npm run e2e

# Back to your tests when done
cd ~/projects/app
git worktree remove ../quick-review
# Your test files are exactly as you left them!
```

---

### Example 7: Experimenting Without Risk

**Scenario:** You want to try a radically different approach to a feature, but don't want to mess up your current implementation.

**Setup:**
```bash
cd ~/projects/app
git worktree add -b experiment/wild-idea ../wild-experiment HEAD
cd ../wild-experiment
```

**Experiment Freely:**
```bash
claude
/rename "wild-experiment"
```

```
> I want to completely reimagine our state management. Let's try
  replacing Redux with a custom observable-based solution. This
  is just an experiment - feel free to be creative.

Claude: Interesting experiment! Let's build a minimal observable
store from scratch. We can use Proxies for automatic tracking...
[Wild experimentation proceeds]
```

If it works out:
```bash
git push -u origin experiment/wild-idea
# Create PR for team review
```

If it doesn't:
```bash
cd ~/projects/app
git worktree remove --force ../wild-experiment
git branch -D experiment/wild-idea
# Nothing in main was affected!
```

---

## Comparison with Other Approaches

### Comparison Table

| Approach | Pros | Cons |
|----------|------|------|
| **Git Worktrees** | Shared .git saves disk space; Instant branch switching; True isolation; Independent Claude sessions; Clean git history | Requires Git 2.5+; Can't checkout same branch twice; Learning curve; Requires disk space for each checkout |
| **Git Stash** | Simple and quick; Built into basic Git workflow; No extra directories; Familiar to most developers | Loses Claude context on switch; Stash conflicts possible; Can forget stashed changes; Only temporary storage |
| **Multiple Clones** | Complete isolation; Can checkout same branch; Works with any Git version; Familiar mental model | Uses 2-3x disk space; Separate .git per clone; Must sync remotes separately; No shared object database |
| **Branches Only** | Simplest approach; No extra tooling; Single directory to manage; Standard Git workflow | Must commit or stash to switch; Loses Claude context on switch; Can't compare side-by-side; Work interruption required |

### When to Use Each Approach

**Use Git Worktrees When:**
- Working on multiple features simultaneously
- Need to preserve Claude session context
- Reviewing PRs while actively developing
- Running long experiments alongside main work
- Team is comfortable with intermediate Git concepts
- Disk space is limited but need isolation

**Use Git Stash When:**
- Quick context switches (< 5 minutes)
- Simple, single-file changes to set aside
- You'll return immediately after a brief interruption
- Working alone on simple projects
- Claude context isn't critical

**Use Multiple Clones When:**
- Need to checkout the same branch in multiple places
- Complete isolation is required (different Git configs)
- Working with submodules that cause issues
- Teaching or demos where worktrees add confusion
- Organization has policies against worktrees

**Use Branches Only When:**
- Simple, linear workflow
- Features are small and quick
- Single developer, single task at a time
- Learning Git basics
- CI/CD handles all parallel work

### Detailed Comparison

#### Disk Space Usage

```
Repository Size: 500MB

Multiple Clones:
- Clone 1: 500MB (.git) + 100MB (working files) = 600MB
- Clone 2: 500MB (.git) + 100MB (working files) = 600MB
- Clone 3: 500MB (.git) + 100MB (working files) = 600MB
Total: 1.8GB

Worktrees:
- Main:      500MB (.git) + 100MB (working files) = 600MB
- Worktree 1: 0 (.git shared) + 100MB (working files) = 100MB
- Worktree 2: 0 (.git shared) + 100MB (working files) = 100MB
Total: 800MB
```

#### Context Switching Speed

| Approach | Time to Switch | Claude Context |
|----------|---------------|----------------|
| Worktrees | Instant (cd) | Preserved |
| Stash | 5-30 seconds | Lost |
| Multiple Clones | Instant (cd) | Preserved |
| Branch Switch | 5-60 seconds | Lost |

#### Complexity

```
Learning Curve:
1. Branches Only     ████░░░░░░ (Beginner)
2. Git Stash         █████░░░░░ (Beginner+)
3. Git Worktrees     ███████░░░ (Intermediate)
4. Multiple Clones   ██████░░░░ (Intermediate)

Daily Usage Complexity:
1. Multiple Clones   ███░░░░░░░ (Low - familiar pattern)
2. Worktrees         ████░░░░░░ (Low - once learned)
3. Git Stash         █████░░░░░ (Medium - stack management)
4. Branches Only     ███████░░░ (High - constant context switching)
```

---

## Troubleshooting

### "fatal: 'branch' is already checked out"

**Error Message:**
```
fatal: 'feature/my-feature' is already checked out at '/path/to/other/worktree'
```

**Cause:** Git prevents the same branch from being checked out in multiple worktrees to avoid conflicts.

**Solutions:**

1. **Use a different branch name:**
```bash
git worktree add -b feature/my-feature-v2 ../new-worktree main
```

2. **Find where the branch is checked out:**
```bash
git worktree list
# Shows all worktrees and their branches
```

3. **Remove the other worktree if no longer needed:**
```bash
git worktree remove /path/to/other/worktree
```

4. **Create a detached HEAD worktree:**
```bash
git worktree add --detach ../temp-worktree feature/my-feature
```

---

### Worktree Corruption Recovery

**Symptoms:**
- `git worktree list` shows incorrect information
- Can't remove or access a worktree
- Git commands fail with worktree-related errors

**Recovery Steps:**

1. **Try automatic repair:**
```bash
git worktree repair
```

2. **Manually clean up if repair fails:**
```bash
# Remove the worktree directory
rm -rf /path/to/broken/worktree

# Prune the stale entry
git worktree prune -v
```

3. **Check and clean .git/worktrees:**
```bash
ls -la .git/worktrees/
# Remove entries for non-existent worktrees
rm -rf .git/worktrees/broken-worktree-name
```

4. **Verify recovery:**
```bash
git worktree list
git status
```

---

### Session Sync Issues

**Problem:** Claude seems to have outdated information about files in the worktree.

**Causes and Solutions:**

1. **Files changed externally:**
```bash
# In Claude session
> /refresh
# Or simply reference the file again
> Please re-read src/app.ts and continue from there
```

2. **Git operations outside Claude:**
```bash
# After pulling or merging in terminal
# Tell Claude about the changes
> I just pulled the latest changes from main. Key files affected
  were src/config.ts and src/database.ts. Please review these
  before we continue.
```

3. **Stale session context:**
```bash
# Start fresh if context is too confused
claude --no-history
/init
```

---

### Disk Space Management

**Problem:** Worktrees are consuming too much disk space.

**Solutions:**

1. **List worktrees and their sizes:**
```bash
git worktree list | while read wt rest; do
    echo -n "$wt: "
    du -sh "$wt" 2>/dev/null | cut -f1
done
```

2. **Remove unused worktrees:**
```bash
# List all worktrees
git worktree list

# Remove those you don't need
git worktree remove ../old-feature

# Prune stale entries
git worktree prune
```

3. **Clean up within worktrees:**
```bash
# In each worktree, clean untracked files
cd ../worktree
git clean -fd       # Remove untracked files
npm cache clean     # Clear npm cache
rm -rf node_modules # Can be reinstalled
```

4. **Shallow worktrees (advanced):**
```bash
# If you only need recent history
git worktree add --detach ../shallow-worktree HEAD
cd ../shallow-worktree
git fetch --depth=1 origin main
```

---

### "Worktree is locked" Errors

**Error Message:**
```
fatal: '/path/to/worktree' is locked
```

**Solutions:**

1. **Check why it's locked:**
```bash
git worktree list
# Look for [locked] marker

# Check for lock reason
cat .git/worktrees/worktree-name/locked
```

2. **Unlock if intentionally locked:**
```bash
git worktree unlock ../locked-worktree
```

3. **Force removal if necessary:**
```bash
git worktree remove --force ../locked-worktree
```

---

### Branch Tracking Issues

**Problem:** Worktree branch isn't tracking remote properly.

**Solution:**
```bash
cd ../worktree

# Check current tracking
git branch -vv

# Set up tracking
git branch --set-upstream-to=origin/feature/my-branch

# Or when pushing
git push -u origin feature/my-branch
```

---

### Common Error Reference

| Error | Cause | Quick Fix |
|-------|-------|-----------|
| "branch already checked out" | Same branch in two worktrees | Use different branch or remove other worktree |
| "not a git repository" | .git file broken | Repair with `git worktree repair` |
| "worktree is locked" | Explicitly locked | `git worktree unlock <path>` |
| "invalid path" | Worktree moved/deleted | `git worktree prune` |
| "refusing to remove" | Uncommitted changes | Commit, stash, or use `--force` |

---

## Quick Reference Cheat Sheet

### Essential Commands

```bash
# ═══════════════════════════════════════════════════════════
#                    GIT WORKTREE CHEAT SHEET
# ═══════════════════════════════════════════════════════════

# ─────────────────────── CREATING ───────────────────────────

# New worktree with new branch from main
git worktree add -b feature/name ../feature-name main

# New worktree with existing branch
git worktree add ../feature-name existing-branch

# Worktree at specific commit (detached HEAD)
git worktree add ../debug-point abc1234

# Worktree for a PR
git fetch origin pull/123/head:pr-123
git worktree add ../pr-review pr-123

# ─────────────────────── VIEWING ────────────────────────────

# List all worktrees
git worktree list

# Detailed worktree info
git worktree list --porcelain

# ─────────────────────── REMOVING ───────────────────────────

# Remove a worktree (must be clean)
git worktree remove ../feature-name

# Force remove (discards changes)
git worktree remove --force ../feature-name

# Remove worktree AND delete branch
git worktree remove ../feature-name && git branch -d feature/name

# ─────────────────────── MAINTENANCE ────────────────────────

# Prune stale worktree references
git worktree prune

# Repair worktree paths
git worktree repair

# Lock a worktree (prevent removal)
git worktree lock ../important-work

# Unlock a worktree
git worktree unlock ../important-work

# Move a worktree
git worktree move ../old-path ../new-path

# ─────────────────────── CLAUDE CODE ────────────────────────

# Start session in worktree
cd ../feature-name && claude

# Initialize project context
/init

# Name the session
/rename "feature-name-session"

# Resume specific session
claude --resume "feature-name-session"

# ═══════════════════════════════════════════════════════════
```

### Common Workflows

```bash
# ═══════════════════════════════════════════════════════════
#                   COMMON WORKFLOWS
# ═══════════════════════════════════════════════════════════

# ─────────────── NEW FEATURE WORKFLOW ───────────────────────

git worktree add -b feature/awesome ../awesome main
cd ../awesome
claude
# /init
# /rename "awesome-feature"
# ... develop ...
git push -u origin feature/awesome
# After merge:
cd ../main-project
git worktree remove ../awesome

# ─────────────── HOTFIX WORKFLOW ────────────────────────────

git worktree add -b hotfix/urgent ../urgent-fix production
cd ../urgent-fix
claude
# /rename "urgent-hotfix"
# ... fix ...
git push -u origin hotfix/urgent
# After merge:
git worktree remove ../urgent-fix

# ─────────────── PR REVIEW WORKFLOW ─────────────────────────

git fetch origin pull/456/head:pr-456
git worktree add ../review-456 pr-456
cd ../review-456
claude
# Review with Claude's help
cd ../main-project
git worktree remove ../review-456
git branch -D pr-456

# ─────────────── PARALLEL FEATURES ──────────────────────────

git worktree add -b feature/auth ../auth main
git worktree add -b feature/payments ../payments main
# Terminal 1: cd ../auth && claude
# Terminal 2: cd ../payments && claude

# ═══════════════════════════════════════════════════════════
```

### Shell Aliases

Add to your `~/.bashrc` or `~/.zshrc`:

```bash
# ═══════════════════════════════════════════════════════════
#                   RECOMMENDED ALIASES
# ═══════════════════════════════════════════════════════════

# List worktrees
alias gwl='git worktree list'

# Quick worktree add
gwa() {
    git worktree add -b "$1" "../$1" "${2:-main}" && cd "../$1"
}

# Quick worktree remove
gwr() {
    cd .. && git worktree remove "./$1" && cd -
}

# Worktree add and start Claude
gwac() {
    git worktree add -b "$1" "../$1" "${2:-main}" && cd "../$1" && claude
}

# Prune stale worktrees
alias gwp='git worktree prune -v'

# Show worktree status with sizes
gwls() {
    git worktree list | while read wt branch; do
        size=$(du -sh "$wt" 2>/dev/null | cut -f1)
        echo "$wt ($size) - $branch"
    done
}

# Usage examples:
# gwl                    # List worktrees
# gwa my-feature         # Create feature worktree from main
# gwa hotfix prod        # Create worktree from prod branch
# gwac new-feature       # Create worktree and start Claude
# gwr old-feature        # Remove worktree
# gwp                    # Prune stale worktrees
# gwls                   # List worktrees with disk usage
```

---

## Best Practices

### When to Use Worktrees vs. Branches

**Use Worktrees For:**
- Features taking more than a few hours
- Work you might need to pause and resume
- Parallel development with separate Claude contexts
- Code reviews you need to run and test
- Experiments you want to compare with current implementation
- Any time you'd lose important Claude context by switching

**Stick to Branches For:**
- Quick fixes (< 30 minutes)
- Sequential work where you finish before switching
- Simple, independent changes
- When disk space is critically limited
- When the team isn't familiar with worktrees

### Cleanup Routines

**Daily Cleanup:**
```bash
# Check for worktrees you've forgotten about
git worktree list

# Remove any completed work
git worktree remove ../finished-feature
```

**Weekly Cleanup:**
```bash
# Prune stale entries
git worktree prune -v

# Check disk usage
du -sh ../my-app*

# Remove merged branches
git branch --merged main | grep -v main | xargs git branch -d
```

**Monthly Cleanup Script:**
```bash
#!/bin/bash
# monthly-cleanup.sh

echo "=== Worktree Cleanup ==="
git worktree list

echo ""
echo "=== Pruning stale entries ==="
git worktree prune -v

echo ""
echo "=== Checking for merged worktrees ==="
for wt in $(git worktree list --porcelain | grep "^worktree" | cut -d' ' -f2); do
    branch=$(git -C "$wt" branch --show-current 2>/dev/null)
    if [ -n "$branch" ]; then
        if git branch --merged main | grep -q "$branch"; then
            echo "MERGED: $wt ($branch)"
        fi
    fi
done

echo ""
echo "=== Disk usage by worktree ==="
git worktree list | while read wt rest; do
    du -sh "$wt" 2>/dev/null
done
```

### Team Conventions

**Suggested Team Standards:**

1. **Naming Convention:**
```bash
# Standard pattern: project-purpose-ticket
myapp-feature-AUTH-123
myapp-hotfix-BUG-456
myapp-review-PR-789
```

2. **Directory Location:**
```bash
# All worktrees sibling to main project
~/projects/myapp/           # main
~/projects/myapp-feature-x/ # feature worktree
~/projects/myapp-hotfix-y/  # hotfix worktree
```

3. **Session Naming:**
```bash
# Match session to worktree purpose
/rename "AUTH-123-implementation"
/rename "BUG-456-investigation"
/rename "PR-789-review"
```

4. **Cleanup Responsibility:**
- Developer removes their own worktrees after merge
- Weekly team reminder to clean up stale worktrees
- CI can report on long-lived worktree branches

5. **Documentation:**
```bash
# In each worktree's CLAUDE.md, note:
## Worktree Purpose
Implementing OAuth authentication for ticket AUTH-123

## Key Decisions Made
- Using passport.js for OAuth providers
- Storing tokens in Redis with 1-hour TTL

## Status
In progress - completing token refresh logic
```

### Performance Tips

1. **Limit Concurrent Worktrees:**
   - 2-3 active worktrees is comfortable
   - 5+ can become confusing and disk-heavy

2. **Use Shallow Clones for Reviews:**
```bash
# When you only need recent history
git worktree add --detach ../quick-review HEAD
cd ../quick-review
git fetch --depth=1 origin feature-branch
git checkout feature-branch
```

3. **Share node_modules with Symlinks:**
```bash
# After initial setup in main worktree
cd ../feature-worktree
ln -s ../main-project/node_modules ./node_modules
```

4. **Exclude Worktrees from Backup:**
```bash
# Add to your backup exclude list
*-worktree/
*-feature-*/
*-hotfix-*/
```

### Security Considerations

1. **Secrets and Credentials:**
   - Each worktree may have its own `.env` file
   - Never commit `.env` files
   - Consider using a secret manager that works across worktrees

2. **Gitignore Consistency:**
   - Ensure `.gitignore` covers worktree-specific files
   - Add patterns for any worktree-local configuration

3. **Claude Session Data:**
   - Session history may contain sensitive information
   - Clean up `.claude/` directories in worktrees before sharing

---

## Summary

### Key Takeaways

1. **Git worktrees enable true parallel development** by allowing multiple working directories to share a single repository.

2. **Each worktree gets its own Claude Code session**, maintaining independent context, conversation history, and project understanding.

3. **The primary benefit is zero-cost context switching** - you never lose your work or Claude's accumulated understanding when jumping between tasks.

4. **Worktrees are especially powerful for:**
   - Parallel feature development
   - Handling urgent hotfixes
   - Reviewing PRs without disruption
   - Comparing different implementations
   - Long-running experiments

5. **Management is straightforward:**
   - `git worktree add` to create
   - `git worktree list` to view
   - `git worktree remove` to clean up
   - `git worktree prune` to maintain

6. **Best practices emphasize:**
   - Consistent naming conventions
   - Regular cleanup routines
   - Team coordination on standards
   - Appropriate use cases (not everything needs a worktree)

### Next Steps

1. **Practice with a simple project:**
   - Create a feature worktree
   - Start a Claude session in each
   - Experience the context isolation firsthand

2. **Set up your aliases:**
   - Add the shell aliases from this tutorial
   - Customize them for your workflow

3. **Establish team conventions:**
   - Agree on naming patterns
   - Set cleanup expectations
   - Document in your team's guidelines

4. **Integrate into your daily workflow:**
   - Start using worktrees for your next multi-day feature
   - Use them for code reviews
   - Create hotfix worktrees when production issues arise

### Additional Resources

- [Official Git Worktree Documentation](https://git-scm.com/docs/git-worktree)
- [Pro Git Book - Worktrees Section](https://git-scm.com/book/en/v2)
- [Claude Code Documentation](https://docs.anthropic.com/claude-code)

---

**Document Version:** 1.0
**Last Updated:** 2024
**Compatibility:** Git 2.20+, Claude Code 1.0+

---

*This tutorial is part of the Claude Code comprehensive documentation series.*
