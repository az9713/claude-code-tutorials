# Tutorial 03: Claude Code Subagents

## Complete Guide to Parallel Task Execution with Isolated Contexts

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Installation & Setup](#2-installation--setup)
3. [Built-in Agents Deep Dive](#3-built-in-agents-deep-dive)
4. [Hand-Holding User Journey](#4-hand-holding-user-journey)
5. [Real-World Examples](#5-real-world-examples)
6. [Creating Custom Agents](#6-creating-custom-agents)
7. [Parallel vs Sequential Execution](#7-parallel-vs-sequential-execution)
8. [Background Agents](#8-background-agents)
9. [Pros and Cons](#9-pros-and-cons)
10. [When to Use Subagents](#10-when-to-use-subagents)
11. [Comparison with Skills](#11-comparison-with-skills)
12. [Quick Reference](#12-quick-reference)

---

## 1. Introduction

### What Are Subagents in Claude Code?

Subagents are isolated Claude instances that execute specific tasks within their own context. Think of them as specialized workers that can be dispatched to handle particular jobs while the main Claude Code session continues orchestrating the overall workflow.

When you spawn a subagent, you're essentially creating a new Claude session with:
- Its own conversation context (isolated from the parent)
- A specific set of tools it can access
- A focused task or objective
- Independent execution that can run in parallel or background

```
┌─────────────────────────────────────────────────────────────┐
│                    Main Claude Code Session                  │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │  Subagent 1  │  │  Subagent 2  │  │  Subagent 3  │       │
│  │  (Explore)   │  │   (Bash)     │  │   (Plan)     │       │
│  │              │  │              │  │              │       │
│  │ Own context  │  │ Own context  │  │ Own context  │       │
│  │ Own tools    │  │ Own tools    │  │ Own tools    │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Why Isolated Context Matters

Context isolation is fundamental to effective subagent usage:

1. **Memory Efficiency**: Each subagent only loads what it needs. An agent exploring your authentication module doesn't need context about your UI components.

2. **Parallel Execution**: Isolated contexts allow multiple agents to run simultaneously without stepping on each other's toes.

3. **Focused Expertise**: An agent with a narrow context can provide more relevant, accurate responses within its domain.

4. **Failure Isolation**: If one subagent encounters an error, it doesn't pollute the main session or other subagents.

5. **Security Boundaries**: Agents can be restricted to specific tools and permissions, reducing risk of unintended actions.

**Example of Context Isolation in Practice:**

```
Main Session Context: 50,000 tokens (entire project overview)
│
├── Explore Agent Context: 8,000 tokens (just file discovery)
├── Bash Agent Context: 3,000 tokens (just command execution)
└── Plan Agent Context: 15,000 tokens (just architecture analysis)
```

Each subagent operates with only the context it needs, making responses faster and more focused.

### The Task Tool Explained

The `Task` tool is the mechanism Claude Code uses to spawn and manage subagents. When you or Claude invokes a subagent, the Task tool handles:

1. **Agent Selection**: Choosing the right agent type for the job
2. **Context Initialization**: Setting up the isolated environment
3. **Tool Provisioning**: Giving the agent access to its allowed tools
4. **Execution Management**: Running the agent (sync, async, or background)
5. **Result Collection**: Gathering and returning the agent's output

**Task Tool Parameters:**

| Parameter | Description | Example |
|-----------|-------------|---------|
| `description` | What the agent should accomplish | "Find all authentication-related files" |
| `prompt` | Detailed instructions for the agent | "Search for files containing login, auth, or session handling" |
| `agent` | Which agent type to use | "Explore", "Bash", "Plan" |
| `background` | Run without blocking | `true` / `false` |

---

## 2. Installation & Setup

### Built-in Agents (No Setup Needed)

Claude Code comes with several built-in agents ready to use immediately:

```
Built-in Agents (Available Immediately)
├── Bash         - Command execution
├── Explore      - Codebase exploration
├── Plan         - Architecture planning
├── general-purpose - Complex multi-step tasks
├── claude-code-guide - Documentation and help
└── statusline-setup - UI configuration
```

To verify available agents, you can ask Claude Code:

```
You: What agents are available?

Claude: The following built-in agents are available:
- Bash: For executing shell commands
- Explore: For exploring and understanding codebases
- Plan: For creating technical plans and architecture
- general-purpose: For complex, multi-step tasks
- claude-code-guide: For Claude Code documentation and help
- statusline-setup: For configuring the status line UI
```

### Creating Custom Agents

Custom agents are defined in markdown or YAML files with specific frontmatter. They live in your agents directory.

**Agent File Structure:**

```yaml
---
name: my-custom-agent
description: Brief description of what this agent does
model: claude-sonnet-4-20250514  # Optional: override default model
tools:
  - Read
  - Glob
  - Grep
permission_mode: default  # Or: bypassPermissions, askUser
---

# System Prompt

You are a specialized agent for [specific purpose].

## Your Capabilities
- Capability 1
- Capability 2

## Guidelines
- Guideline 1
- Guideline 2
```

### Agent Directory Structure (~/.claude/agents/)

Custom agents are stored in a specific directory hierarchy:

```
~/.claude/
├── agents/                    # Your custom agents
│   ├── security-auditor.md    # Security-focused agent
│   ├── test-runner.md         # Testing specialist
│   ├── doc-generator.md       # Documentation agent
│   └── team-specific/         # Subdirectories work too
│       ├── frontend-review.md
│       └── backend-review.md
├── settings.json              # Claude Code settings
└── credentials.json           # API credentials
```

**Creating Your Agent Directory:**

```bash
# Create the agents directory
mkdir -p ~/.claude/agents

# Create your first custom agent
cat > ~/.claude/agents/hello-world.md << 'EOF'
---
name: hello-world
description: A simple test agent
tools:
  - Read
---

# Hello World Agent

You are a friendly agent that greets users and helps them understand how agents work.

When invoked, explain:
1. That you are a custom subagent
2. You have access to the Read tool
3. You operate in an isolated context
EOF
```

**Verifying Agent Installation:**

```
You: Use the hello-world agent to introduce itself

Claude: [Invokes hello-world agent]

Agent Response: Hello! I'm the hello-world agent, a custom subagent you created.
I have access to the Read tool and operate in my own isolated context...
```

---

## 3. Built-in Agents Deep Dive

### Bash Agent

**Purpose:** Execute shell commands and scripts with focused precision.

**Tools Available:**
- `Bash` - Execute shell commands

**Limitations:**
- No file reading/writing tools (uses shell commands instead)
- No web access
- Inherits system permissions but respects Claude Code sandbox

**Best Use Cases:**
- Running build commands
- Executing test suites
- Git operations
- System information gathering
- Package management

**Example Usage:**

```
Prompt: "Use the Bash agent to check the Node.js version and list installed global packages"

Bash Agent executes:
1. node --version
2. npm list -g --depth=0

Returns: Structured output with versions and packages
```

---

### Explore Agent

**Purpose:** Navigate and understand codebases without making changes.

**Tools Available:**
- `Read` - Read file contents
- `Glob` - Find files by pattern
- `Grep` - Search file contents

**Limitations:**
- Read-only (cannot modify files)
- No command execution
- No web access

**Best Use Cases:**
- Understanding project structure
- Finding related code
- Locating dependencies
- Tracing code flows
- Answering "where is X defined?"

**Example Usage:**

```
Prompt: "Use Explore to find all React components that use the useAuth hook"

Explore Agent:
1. Greps for "useAuth" across *.tsx files
2. Reads matching files to understand context
3. Returns list of components with usage patterns
```

---

### Plan Agent

**Purpose:** Create comprehensive technical plans and architecture designs.

**Tools Available:**
- `Read` - Read existing code
- `Glob` - Find relevant files
- `Grep` - Search for patterns

**Limitations:**
- Read-only (planning, not implementation)
- No command execution
- No file modification

**Best Use Cases:**
- Feature planning
- Refactoring strategies
- Architecture decisions
- Migration planning
- Technical specifications

**Example Usage:**

```
Prompt: "Use Plan to design a caching layer for our API"

Plan Agent:
1. Analyzes existing API structure
2. Identifies hot paths and bottlenecks
3. Returns detailed caching strategy with:
   - Cache invalidation approach
   - Data structure recommendations
   - Implementation phases
```

---

### General-Purpose Agent

**Purpose:** Handle complex, multi-step tasks that require multiple tools.

**Tools Available:**
- `Read` - Read files
- `Write` - Write files
- `Edit` - Edit files
- `Glob` - Find files
- `Grep` - Search content
- `Bash` - Execute commands

**Limitations:**
- Full toolset means larger context
- May be overkill for simple tasks

**Best Use Cases:**
- Complex refactoring
- Multi-file changes
- Tasks requiring both reading and writing
- Workflows with dependencies

**Example Usage:**

```
Prompt: "Use general-purpose agent to add error handling to all API endpoints"

General-Purpose Agent:
1. Finds all API endpoint files
2. Reads current implementations
3. Edits each file to add try/catch
4. Runs tests to verify changes
```

---

### Claude-Code-Guide Agent

**Purpose:** Provide documentation and help about Claude Code features.

**Tools Available:**
- Limited to documentation access
- Read access to help resources

**Limitations:**
- Cannot execute code
- Cannot modify files
- Information-only

**Best Use Cases:**
- Learning Claude Code features
- Understanding available tools
- Finding best practices
- Troubleshooting issues

**Example Usage:**

```
Prompt: "Use claude-code-guide to explain how memory files work"

Guide Agent:
Returns comprehensive explanation of:
- CLAUDE.md and CLAUDE.local.md
- Memory persistence
- Best practices for memory organization
```

---

### Statusline-Setup Agent

**Purpose:** Configure Claude Code's terminal status line UI.

**Tools Available:**
- Configuration writing
- Settings management

**Limitations:**
- Specific to UI configuration
- Cannot perform general tasks

**Best Use Cases:**
- Customizing status display
- Setting up notifications
- Configuring visual preferences
- Workspace-specific UI

**Example Usage:**

```
Prompt: "Use statusline-setup to configure a minimal status display"

Statusline Agent:
Configures status line with:
- Token count only
- No model indicator
- Compact format
```

---

## 4. Hand-Holding User Journey

### Step 1: Use Your First Built-in Agent

Let's start with the simplest agent interaction.

**Goal:** Use the Explore agent to understand your project structure.

**Command:**
```
You: Use the Explore agent to map out this project's directory structure and identify the main entry points
```

**What Happens:**
1. Claude Code invokes the Task tool with agent="Explore"
2. Explore agent receives limited tools (Read, Glob, Grep)
3. Agent analyzes the project independently
4. Results return to main session

**Expected Output:**
```
[Task: Explore agent analyzing project structure]

Project Structure Analysis:
├── src/
│   ├── index.ts        <- Main entry point
│   ├── components/     <- React components
│   └── utils/          <- Utility functions
├── tests/
│   └── *.test.ts       <- Test files
└── package.json        <- Project configuration

Entry Points Identified:
1. src/index.ts - Application bootstrap
2. src/server.ts - API server entry
```

---

### Step 2: Run Agents in Parallel

**Goal:** Explore multiple parts of your codebase simultaneously.

**Command:**
```
You: Run three Explore agents in parallel:
1. One to analyze the authentication system
2. One to analyze the database layer
3. One to analyze the API routes
```

**What Happens:**
1. Claude Code spawns three Explore agents simultaneously
2. Each works in isolation with its own context
3. All three execute at the same time
4. Results are collected and synthesized

**Expected Output:**
```
[Task 1: Explore - Authentication System] Running...
[Task 2: Explore - Database Layer] Running...
[Task 3: Explore - API Routes] Running...

[All tasks completed]

Authentication System (Agent 1):
- Uses JWT tokens with 24h expiry
- Middleware in src/auth/middleware.ts
- User model in src/models/User.ts

Database Layer (Agent 2):
- PostgreSQL with Prisma ORM
- 12 models defined
- Migration history shows 23 migrations

API Routes (Agent 3):
- 45 endpoints across 8 route files
- RESTful design with /api/v1 prefix
- OpenAPI spec in docs/api.yaml
```

---

### Step 3: Run Agents in Background

**Goal:** Start a long-running task without blocking your session.

**Command:**
```
You: Run the Bash agent in background to execute our full test suite, then continue helping me with other tasks
```

**What Happens:**
1. Bash agent starts in background mode
2. Main session remains interactive
3. You can do other work while tests run
4. Results arrive when complete

**Expected Output:**
```
[Background Task Started: Bash agent running test suite]
Task ID: bg-task-001

You can continue working. I'll notify you when tests complete.

---

You: While that runs, explain the authentication flow

Claude: [Continues with explanation while tests run]

---

[Background Task bg-task-001 Completed]

Test Results:
✓ 234 tests passed
✗ 2 tests failed
  - user.auth.test.ts:45 - Token expiry check
  - api.routes.test.ts:89 - Rate limiting
```

---

### Step 4: Create a Custom Agent

**Goal:** Build a specialized agent for your specific needs.

**Step 4a: Create the agent file**

```bash
cat > ~/.claude/agents/code-reviewer.md << 'EOF'
---
name: code-reviewer
description: Performs thorough code reviews with security and performance focus
model: claude-sonnet-4-20250514
tools:
  - Read
  - Glob
  - Grep
---

# Code Reviewer Agent

You are an expert code reviewer with deep knowledge of security vulnerabilities, performance optimization, and clean code principles.

## Review Checklist

For every code review, check:

### Security
- [ ] SQL injection vulnerabilities
- [ ] XSS attack vectors
- [ ] Authentication/authorization gaps
- [ ] Sensitive data exposure
- [ ] Input validation

### Performance
- [ ] N+1 query problems
- [ ] Memory leaks
- [ ] Unnecessary re-renders (React)
- [ ] Missing indexes
- [ ] Inefficient algorithms

### Code Quality
- [ ] SOLID principles adherence
- [ ] DRY violations
- [ ] Proper error handling
- [ ] Clear naming conventions
- [ ] Adequate documentation

## Output Format

Provide findings in this structure:

```
## Summary
[Brief overview]

## Critical Issues
[Must fix before merge]

## Suggestions
[Nice to have improvements]

## Approval Status
[APPROVED / CHANGES REQUESTED]
```
EOF
```

**Step 4b: Use your custom agent**

```
You: Use the code-reviewer agent to review the changes in src/auth/login.ts

Claude: [Invokes code-reviewer agent]

Code Review: src/auth/login.ts

## Summary
The login implementation is generally solid but has two security concerns.

## Critical Issues
1. Line 45: Password compared using == instead of timing-safe comparison
2. Line 67: Rate limiting missing for failed attempts

## Suggestions
1. Consider adding 2FA support
2. Log authentication events for audit trail

## Approval Status
CHANGES REQUESTED
```

---

### Step 5: Share Agents with Team

**Goal:** Make your custom agents available to your team.

**Option A: Version Control (Recommended)**

```bash
# Add agents to your project
mkdir -p .claude/agents
cp ~/.claude/agents/code-reviewer.md .claude/agents/

# Commit to repository
git add .claude/agents/
git commit -m "Add team code-reviewer agent"
git push
```

Team members get agents automatically when they clone/pull.

**Option B: Shared Network Location**

```bash
# Set up shared agents directory
mkdir -p /shared/team/claude-agents

# Symlink to your local directory
ln -s /shared/team/claude-agents ~/.claude/team-agents
```

**Option C: Agent Package**

Create a distributable package:

```bash
# Create package
mkdir claude-agents-package
cp -r ~/.claude/agents/* claude-agents-package/

# Add installation script
cat > claude-agents-package/install.sh << 'EOF'
#!/bin/bash
mkdir -p ~/.claude/agents
cp -r ./*.md ~/.claude/agents/
echo "Agents installed successfully!"
EOF

# Distribute via your preferred method
zip -r claude-agents.zip claude-agents-package/
```

---

## 5. Real-World Examples

### Example 1: Parallel Codebase Exploration (3 Explore Agents)

**Scenario:** You've joined a new project and need to understand its architecture quickly.

**Implementation:**

```
You: I need to understand this codebase. Run three Explore agents in parallel:
1. Agent 1: Map the overall architecture and identify key design patterns
2. Agent 2: Analyze the data layer - models, database, and data flow
3. Agent 3: Understand the API surface - endpoints, authentication, and contracts
```

**Behind the Scenes:**

```javascript
// Claude Code internally does something like:
await Promise.all([
  Task.run({
    agent: "Explore",
    prompt: "Map overall architecture and design patterns",
    context: { focus: "high-level structure" }
  }),
  Task.run({
    agent: "Explore",
    prompt: "Analyze data layer comprehensively",
    context: { focus: "data and persistence" }
  }),
  Task.run({
    agent: "Explore",
    prompt: "Document API surface completely",
    context: { focus: "API and contracts" }
  })
]);
```

**Output:**

```
=== Architecture Agent Results ===
Pattern: Hexagonal Architecture
- Core domain in /src/domain
- Ports in /src/ports
- Adapters in /src/adapters
Key Dependencies: TypeScript, Zod, Prisma

=== Data Layer Agent Results ===
Database: PostgreSQL
ORM: Prisma with 15 models
Key Entities:
- User (central, many relations)
- Organization (multi-tenant support)
- Event (audit logging)

=== API Agent Results ===
Style: REST with OpenAPI 3.0
Authentication: JWT + API Keys
Endpoints: 67 total
- /api/v1/users/* (12 endpoints)
- /api/v1/orgs/* (8 endpoints)
- /api/v1/events/* (15 endpoints)
Rate Limiting: 1000 req/min per API key
```

---

### Example 2: Background Test Runner

**Scenario:** Run comprehensive tests without blocking your workflow.

**Implementation:**

```
You: Start a background Bash agent to run our full test suite including e2e tests. I need to continue working on feature development.
```

**Agent Configuration (if custom):**

```yaml
---
name: test-runner
description: Comprehensive test execution agent
tools:
  - Bash
---

# Test Runner Agent

Execute the complete test suite in order:
1. Unit tests (jest)
2. Integration tests
3. E2E tests (playwright)

Report format:
- Summary first
- Failed tests with details
- Coverage report
```

**Execution Flow:**

```
[Background Agent Started: test-runner]
Task ID: test-run-001

> Continuing with your request...

You: Let's work on the new feature in src/features/notifications

Claude: [Works with you on notifications feature]

...15 minutes later...

[Background Task Completed: test-run-001]

Test Results Summary:
────────────────────
Unit Tests:     156/158 passed (2 failed)
Integration:    45/45 passed
E2E Tests:      23/24 passed (1 failed)

Failed Tests:
1. unit/auth.test.ts:89
   - Expected token expiry to be 3600, got 7200

2. e2e/checkout.spec.ts:156
   - Timeout waiting for payment confirmation

Coverage: 78.5% (target: 80%)
```

---

### Example 3: Security Audit Pipeline

**Scenario:** Comprehensive security review of your application.

**Implementation:**

```
You: Run a security audit pipeline with specialized agents:
1. Dependency scanner - check for vulnerable packages
2. Code analyzer - find security anti-patterns
3. Secret detector - find exposed credentials
4. Permission auditor - check access controls
```

**Custom Agent: security-scanner.md**

```yaml
---
name: security-scanner
description: Multi-phase security audit
tools:
  - Bash
  - Read
  - Glob
  - Grep
---

# Security Scanner Agent

Perform comprehensive security analysis.

## Phase 1: Dependencies
- Run npm audit / pip safety check
- Check for outdated packages
- Identify packages with known CVEs

## Phase 2: Code Patterns
Search for:
- eval() usage
- SQL string concatenation
- Hardcoded credentials
- Disabled security features
- Unsafe deserialization

## Phase 3: Secrets Detection
Scan for:
- API keys (pattern: [A-Za-z0-9]{32,})
- AWS credentials
- Private keys
- Connection strings

## Phase 4: Access Control
Verify:
- Authentication on all routes
- Authorization checks
- CORS configuration
- Rate limiting
```

**Output:**

```
Security Audit Report
═══════════════════════════════════════════

🔴 CRITICAL (2 findings)
1. Exposed AWS credentials in .env.example (line 15)
2. SQL injection vulnerability in /src/db/queries.ts:67

🟡 HIGH (4 findings)
1. Package 'lodash' has known prototype pollution (CVE-2021-23337)
2. Missing rate limiting on /api/auth/login
3. CORS allows all origins in production config
4. Session cookie missing 'secure' flag

🟢 MEDIUM (7 findings)
[Listed...]

🔵 LOW (12 findings)
[Listed...]

Recommendations:
1. Rotate exposed AWS credentials immediately
2. Use parameterized queries in queries.ts
3. Update lodash to 4.17.21+
```

---

### Example 4: Code Review with Reviewer Agent

**Scenario:** Automated code review before PR merge.

**Custom Agent: pr-reviewer.md**

```yaml
---
name: pr-reviewer
description: Pull request code reviewer
tools:
  - Bash
  - Read
  - Glob
  - Grep
---

# PR Reviewer Agent

You are a senior developer reviewing pull requests.

## Review Process

1. Get changed files from git
2. Analyze each change for:
   - Correctness
   - Performance implications
   - Security concerns
   - Test coverage
   - Documentation needs

## Review Standards

- Changes must have tests
- No console.log in production code
- Error handling required for async operations
- Types must be explicit (no 'any')

## Output Format

```markdown
## PR Review: [branch-name]

### Overview
[Brief description of changes]

### File-by-File Review
[Detailed comments per file]

### Required Changes
[Must fix before approval]

### Suggestions
[Optional improvements]

### Verdict
APPROVE / REQUEST CHANGES / COMMENT
```
```

**Usage:**

```
You: Use pr-reviewer to review the changes between main and feature/user-settings

Claude: [Invokes pr-reviewer agent]

## PR Review: feature/user-settings

### Overview
Adds user settings page with preferences for notifications, theme, and privacy.

### File-by-File Review

**src/pages/Settings.tsx**
- Line 45: Consider memoizing the preference update handler
- Line 78: Good use of optimistic updates
- Line 92: Missing error boundary for this component

**src/api/settings.ts**
- Line 23: Add retry logic for network failures
+ Line 34: Nice input validation with Zod

**src/components/ThemeToggle.tsx**
- Clean implementation, no issues

### Required Changes
1. Add error handling in Settings.tsx:92
2. Include unit tests for ThemeToggle

### Suggestions
1. Consider extracting form logic to custom hook
2. Add loading skeleton for better UX

### Verdict
REQUEST CHANGES
```

---

### Example 5: Multi-Agent Refactoring Workflow

**Scenario:** Large-scale refactoring requiring analysis, planning, and implementation.

**Implementation:**

```
You: Coordinate a refactoring of our authentication system:

Phase 1: Explore agent - Analyze current auth implementation
Phase 2: Plan agent - Design new implementation
Phase 3: general-purpose agent - Execute refactoring
Phase 4: Bash agent - Run tests and verify
```

**Execution:**

```
[Phase 1: Explore Agent]
Analyzing current authentication...

Current State:
- JWT implementation in /src/auth
- 3 middleware functions
- Scattered across 8 files
- Inconsistent error handling
- No refresh token support

[Phase 2: Plan Agent]
Designing refactored architecture...

Proposed Changes:
1. Consolidate auth logic in /src/auth/core
2. Implement refresh token rotation
3. Add consistent AuthError class
4. Create auth middleware factory
5. Add rate limiting per user

Implementation Order:
1. Create AuthError class
2. Build core auth module
3. Implement middleware factory
4. Add refresh tokens
5. Migrate existing endpoints
6. Remove deprecated code

[Phase 3: general-purpose Agent]
Executing refactoring...

Changes Made:
- Created /src/auth/core/index.ts
- Added /src/auth/errors/AuthError.ts
- Refactored 8 files to use new patterns
- Added refresh token support
- Updated 12 test files

[Phase 4: Bash Agent]
Running verification...

Results:
✓ All 89 auth tests passing
✓ TypeScript compilation successful
✓ No linting errors
✓ Coverage: 94%
```

---

### Example 6: Documentation Generation

**Scenario:** Generate comprehensive API documentation from code.

**Custom Agent: doc-generator.md**

```yaml
---
name: doc-generator
description: Generates documentation from code
tools:
  - Read
  - Glob
  - Grep
  - Write
---

# Documentation Generator Agent

Generate comprehensive documentation by analyzing source code.

## Documentation Types

1. **API Documentation**
   - Endpoint descriptions
   - Request/response schemas
   - Authentication requirements
   - Example requests

2. **Code Documentation**
   - Function signatures
   - Parameter descriptions
   - Return values
   - Usage examples

3. **Architecture Documentation**
   - System overview
   - Component relationships
   - Data flow diagrams (Mermaid)

## Output Standards
- Use Markdown format
- Include code examples
- Add Mermaid diagrams where helpful
- Follow existing doc style
```

**Usage:**

```
You: Use doc-generator to create API documentation for our /api/users endpoints

Claude: [Invokes doc-generator agent]

Generated: docs/api/users.md

# Users API

## Overview
User management endpoints for CRUD operations and authentication.

## Endpoints

### GET /api/users
List all users with pagination.

**Authentication:** Required (Bearer token)

**Query Parameters:**
| Parameter | Type   | Default | Description |
|-----------|--------|---------|-------------|
| page      | number | 1       | Page number |
| limit     | number | 20      | Items per page |

**Response:**
```json
{
  "users": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150
  }
}
```

[Additional endpoints documented...]
```

---

### Example 7: CI/CD Automation

**Scenario:** Automate deployment pipeline validation and execution.

**Custom Agent: deploy-agent.md**

```yaml
---
name: deploy-agent
description: CI/CD pipeline automation
tools:
  - Bash
  - Read
  - Glob
---

# Deployment Agent

Automate and validate deployment processes.

## Pre-Deployment Checklist
- [ ] All tests passing
- [ ] No security vulnerabilities
- [ ] Build successful
- [ ] Environment variables set
- [ ] Database migrations ready

## Deployment Process
1. Run pre-deployment checks
2. Build application
3. Run smoke tests
4. Deploy to staging
5. Run integration tests on staging
6. Deploy to production (if approved)
7. Verify production health

## Rollback Procedure
If any step fails:
1. Identify failure point
2. Stop deployment
3. Rollback to previous version
4. Notify team
```

**Usage:**

```
You: Use deploy-agent to prepare and validate a staging deployment

Claude: [Invokes deploy-agent agent]

Deployment Preparation Report
═══════════════════════════════

Pre-Deployment Checks:
✓ Tests: 234/234 passing
✓ Security: No critical vulnerabilities
✓ Build: Successful (2.3s)
✓ Environment: All variables configured
✓ Migrations: 2 pending (safe to apply)

Build Output:
- Bundle size: 1.2MB (within limits)
- Chunks: 12 (optimized)
- Source maps: Generated

Staging Deployment Ready:
- Branch: release/v2.1.0
- Commit: a1b2c3d4
- Changes: 45 files modified

Recommended Actions:
1. Review pending migrations:
   - 20240115_add_user_preferences
   - 20240116_index_optimization

2. Execute deployment:
   $ npm run deploy:staging

3. Post-deployment:
   - Verify health: /api/health
   - Run smoke tests: npm run test:smoke
```

---

## 6. Creating Custom Agents

### Agent YAML/MD Format

Custom agents use markdown files with YAML frontmatter:

```yaml
---
# Required fields
name: my-agent-name          # Unique identifier
description: What this agent does  # Brief description

# Optional fields
model: claude-sonnet-4-20250514   # Override default model
tools:                        # List of allowed tools
  - Read
  - Write
  - Bash
permission_mode: default      # Permission handling
---

# Agent System Prompt

Everything below the frontmatter becomes the system prompt.
This defines the agent's behavior, expertise, and guidelines.
```

### Defining Tools

Available tools for agents:

| Tool | Description | Use Case |
|------|-------------|----------|
| `Read` | Read file contents | Code analysis |
| `Write` | Create/overwrite files | Code generation |
| `Edit` | Modify existing files | Refactoring |
| `Glob` | Find files by pattern | Discovery |
| `Grep` | Search file contents | Pattern matching |
| `Bash` | Execute commands | Build, test, deploy |
| `WebFetch` | Fetch web content | External resources |
| `WebSearch` | Search the web | Research |

**Tool Combination Patterns:**

```yaml
# Read-only analysis agent
tools:
  - Read
  - Glob
  - Grep

# Full development agent
tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash

# Research agent
tools:
  - Read
  - Glob
  - WebSearch
  - WebFetch
```

### System Prompts

Effective system prompts define agent behavior:

```markdown
# Security Auditor Agent

You are a security expert specializing in application security.

## Your Expertise
- OWASP Top 10 vulnerabilities
- Authentication and authorization patterns
- Cryptographic best practices
- Secure coding guidelines

## Analysis Process
1. First, understand the application architecture
2. Identify security-critical components
3. Analyze for common vulnerabilities
4. Provide actionable recommendations

## Output Requirements
- Always explain the risk level (Critical/High/Medium/Low)
- Provide specific file and line references
- Include remediation code examples
- Never suggest disabling security features

## Constraints
- Do not modify code directly
- Report findings, don't fix them
- Maintain confidentiality of findings
```

### Model Selection

Override the default model for specific needs:

```yaml
---
name: complex-analyzer
model: claude-opus-4-20250514  # Use Opus for complex reasoning
tools:
  - Read
  - Glob
---
```

**Model Selection Guidelines:**

| Model | Best For |
|-------|----------|
| `claude-opus-4-5-20251101` | Complex reasoning, architecture |
| `claude-sonnet-4-20250514` | Balanced performance (default) |
| `claude-haiku-3-5-20241022` | Fast, simple tasks |

### Permission Modes

Control how agents handle permissions:

```yaml
---
name: automated-fixer
permission_mode: bypassPermissions  # Don't ask for confirmation
tools:
  - Read
  - Edit
---
```

**Permission Mode Options:**

| Mode | Behavior |
|------|----------|
| `default` | Ask user for confirmation on sensitive actions |
| `bypassPermissions` | Execute without asking |
| `askUser` | Always ask before any action |

---

## 7. Parallel vs Sequential Execution

### When to Parallelize

**Parallelize when:**
- Tasks are independent (no shared state)
- Each task can complete without the other's result
- You're exploring or analyzing (read-only operations)
- Tasks access different parts of the codebase

```
Good Parallelization:
┌─────────────────┐
│ Analyze /src    │──┐
└─────────────────┘  │
┌─────────────────┐  ├──> Combined Results
│ Analyze /tests  │──┤
└─────────────────┘  │
┌─────────────────┐  │
│ Analyze /docs   │──┘
└─────────────────┘
```

**Run sequentially when:**
- Task B depends on Task A's output
- Tasks modify the same files
- Order matters for correctness
- Resource constraints exist

```
Sequential Required:
┌─────────────────┐
│ Plan Changes    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Implement Plan  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Test Changes    │
└─────────────────┘
```

### Dependencies Between Agents

**Handling Agent Dependencies:**

```
You: I need a sequential workflow:
1. First, use Explore to analyze our current API structure
2. Then, use Plan to design improvements based on that analysis
3. Finally, use general-purpose to implement the first improvement

Please wait for each step to complete before starting the next.
```

Claude Code handles this automatically, passing context between steps.

### Resource Considerations

**Memory:**
- Each agent consumes context window space
- Parallel agents multiply memory usage
- Limit parallel agents to 3-5 for large tasks

**API Rate Limits:**
- Parallel agents make concurrent API calls
- May hit rate limits faster
- Consider sequential for rate-limited scenarios

**Execution Time:**
- Parallel reduces total time (wall clock)
- But increases resource consumption
- Balance based on urgency vs. cost

```
Time Comparison:
Sequential: Task1(5s) → Task2(3s) → Task3(4s) = 12s total
Parallel:   Task1(5s) | Task2(3s) | Task3(4s) = 5s total (max of all)
```

---

## 8. Background Agents

### How to Launch in Background

Request background execution explicitly:

```
You: Run the test suite in background using Bash agent
```

Or for custom agents:

```
You: Launch the security-scanner agent in background to audit the entire codebase
```

**Background Task Properties:**
- Non-blocking (main session continues)
- Independent execution
- Results delivered when complete
- Can check status periodically

### Checking Agent Status

Query running background tasks:

```
You: What background tasks are running?

Claude: Active Background Tasks:
1. [bg-001] security-scanner - Running (45% complete)
   Started: 2 minutes ago

2. [bg-002] test-runner - Running (78% complete)
   Started: 5 minutes ago
```

### Retrieving Results

Results arrive automatically when complete:

```
[Background Task Completed: bg-002]

Test Results:
✓ 234 tests passed
✗ 2 tests failed

Details:
...
```

Or request results explicitly:

```
You: Show me the results from the security-scanner task

Claude: [Retrieves results from bg-001]

Security Scan Complete:
...
```

---

## 9. Pros and Cons

### Context Isolation Benefits

**Advantages:**

1. **Focused Expertise**
   - Agent only sees relevant context
   - Responses are more targeted
   - Less confusion from unrelated code

2. **Memory Efficiency**
   - Smaller context per agent
   - Can handle larger codebases
   - Faster response times

3. **Parallel Execution**
   - Multiple tasks simultaneously
   - Reduced total execution time
   - Better resource utilization

4. **Failure Isolation**
   - One agent's error doesn't affect others
   - Easier debugging
   - More reliable workflows

5. **Security Boundaries**
   - Agents have only necessary permissions
   - Reduced blast radius for mistakes
   - Auditable access patterns

### Overhead Considerations

**Disadvantages:**

1. **Coordination Complexity**
   - Managing multiple agents
   - Passing context between them
   - Synchronization challenges

2. **Context Loss**
   - Agent doesn't see main session history
   - May repeat analysis already done
   - Requires explicit context passing

3. **API Usage**
   - Each agent makes separate API calls
   - Higher token consumption
   - Potential rate limiting

4. **Debugging Difficulty**
   - Errors may occur in any agent
   - Harder to trace issues
   - Multiple log streams

### When Agents Add Complexity

**Avoid agents when:**
- Task is simple and single-purpose
- Full context is needed
- Real-time interaction required
- Task has many fine-grained dependencies

**Example of Unnecessary Agent:**

```
Bad: "Use an agent to rename this variable"
Good: Just rename the variable directly

Bad: "Use 5 agents to review 5 lines of code"
Good: Review the code directly
```

---

## 10. When to Use Subagents

### Decision Flowchart

```
┌─────────────────────────────────────────┐
│           Is the task complex?          │
│         (Multiple steps/files)          │
└────────────────────┬────────────────────┘
                     │
           ┌─────────┴─────────┐
           │                   │
          Yes                  No
           │                   │
           ▼                   ▼
┌──────────────────┐  ┌──────────────────┐
│ Can parts run    │  │ Do it directly,  │
│ independently?   │  │ no agents needed │
└────────┬─────────┘  └──────────────────┘
         │
   ┌─────┴─────┐
   │           │
  Yes          No
   │           │
   ▼           ▼
┌──────────┐  ┌───────────────────┐
│ PARALLEL │  │ Is isolation      │
│ AGENTS   │  │ beneficial?       │
└──────────┘  └─────────┬─────────┘
                        │
                  ┌─────┴─────┐
                  │           │
                 Yes          No
                  │           │
                  ▼           ▼
           ┌──────────┐  ┌──────────────┐
           │SEQUENTIAL│  │ Do directly, │
           │ AGENTS   │  │ no agents    │
           └──────────┘  └──────────────┘
```

### Subagents vs Direct Execution

| Scenario | Use Agents | Use Direct |
|----------|------------|------------|
| Exploring large codebase | Yes | - |
| Editing one file | - | Yes |
| Running parallel analyses | Yes | - |
| Quick command execution | - | Yes |
| Long-running background task | Yes | - |
| Interactive debugging | - | Yes |
| Multi-step automated workflow | Yes | - |
| Simple question answering | - | Yes |

### Subagents vs Skills

| Aspect | Subagents | Skills |
|--------|-----------|--------|
| **Context** | Isolated | Shared with main |
| **Execution** | Parallel/Background | Sequential in main |
| **Purpose** | Task execution | Behavior modification |
| **State** | Independent | Uses main session state |
| **Best for** | Heavy lifting | Workflow patterns |

---

## 11. Comparison with Skills

### Detailed Comparison Table

| Feature | Subagents | Skills |
|---------|-----------|--------|
| **Definition** | Isolated Claude instances | Loaded instructions/behaviors |
| **Context Sharing** | No - fully isolated | Yes - shares main context |
| **Parallel Execution** | Yes - can run simultaneously | No - sequential only |
| **Background Execution** | Yes - non-blocking | No - always blocking |
| **Custom Tools** | Yes - can limit tools | No - uses main session tools |
| **Model Override** | Yes - can use different model | No - uses session model |
| **State Persistence** | No - fresh each invocation | Yes - within session |
| **Use Case** | Heavy computation, exploration | Workflows, templates |
| **Overhead** | Higher (new context) | Lower (same context) |
| **Debugging** | Harder (separate logs) | Easier (same session) |

### When to Use Which

**Use Subagents for:**
- Large codebase exploration
- Long-running computations
- Parallel task execution
- Isolated security-sensitive operations
- Tasks that don't need main session context

**Use Skills for:**
- Workflow automation (git, PR creation)
- Code review checklists
- Template-based operations
- Tasks needing main session context
- Quick behavior modifications

### Combining Both

The most powerful workflows combine agents and skills:

```
Workflow: Feature Development

1. Skill: brainstorming (planning phase)
   ├── Establishes requirements
   └── Defines acceptance criteria

2. Subagent: Explore (analysis phase)
   ├── Finds related code
   └── Returns independently

3. Subagent: Plan (design phase)
   ├── Creates implementation plan
   └── Returns with blueprint

4. Skill: test-driven-development (implementation)
   ├── Guides TDD workflow
   └── Uses main session context

5. Subagent: Bash (background testing)
   ├── Runs full test suite
   └── Reports when done

6. Skill: verification-before-completion
   ├── Ensures quality
   └── Checks all criteria met
```

---

## 12. Quick Reference

### Agent Command Patterns

**Invoke Built-in Agent:**
```
Use the [Explore/Bash/Plan] agent to [task description]
```

**Parallel Agents:**
```
Run these agents in parallel:
1. Explore agent to analyze [area 1]
2. Explore agent to analyze [area 2]
3. Bash agent to run [command]
```

**Background Agent:**
```
Run [agent] in background to [long-running task]
```

**Check Background Status:**
```
What background tasks are running?
```

**Sequential Agents:**
```
First use [Agent 1] to [task 1], then use [Agent 2] to [task 2] based on those results
```

### Custom Agent Template

Create a file at `~/.claude/agents/my-agent.md`:

```yaml
---
name: my-custom-agent
description: Brief description of purpose
model: claude-sonnet-4-20250514
tools:
  - Read
  - Glob
  - Grep
  - Bash
permission_mode: default
---

# Agent Name

Brief description of what this agent does and its expertise.

## Capabilities
- Capability 1
- Capability 2
- Capability 3

## Process
1. Step 1 of how agent operates
2. Step 2 of how agent operates
3. Step 3 of how agent operates

## Output Format
Describe expected output structure.

## Constraints
- Constraint 1
- Constraint 2

## Examples
Provide example inputs and outputs.
```

### Common Agent Patterns

**Exploration Pattern:**
```
Use Explore to:
1. Find all files matching [pattern]
2. Analyze their structure
3. Report key findings
```

**Audit Pattern:**
```
Use [security/quality] agent to:
1. Scan entire codebase
2. Check for [issues]
3. Generate report with severity levels
```

**Pipeline Pattern:**
```
Sequential workflow:
1. Analyze with Explore
2. Plan with Plan
3. Implement with general-purpose
4. Verify with Bash
```

**Parallel Analysis Pattern:**
```
Parallel exploration:
1. Agent 1 -> /src
2. Agent 2 -> /tests
3. Agent 3 -> /docs
Combine results for full picture
```

---

## Summary

Subagents are powerful tools for handling complex, parallelizable, or long-running tasks in Claude Code. They provide:

- **Isolation**: Each agent has its own context and tools
- **Parallelism**: Multiple agents can run simultaneously
- **Background execution**: Long tasks don't block your workflow
- **Specialization**: Custom agents for specific purposes

Use them wisely:
- Prefer direct execution for simple tasks
- Use agents when isolation or parallelism provides value
- Combine with skills for comprehensive workflows
- Create custom agents for repeated specialized tasks

Remember: Agents are tools, not solutions. The goal is to complete tasks efficiently, and sometimes that means not using agents at all.

---

*This tutorial is part of the Claude Code documentation series.*
*Last updated: January 2026*
