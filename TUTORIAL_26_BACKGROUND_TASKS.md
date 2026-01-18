# Tutorial 26: Background Tasks and Remote Execution

> **Run Claude Code tasks asynchronously while you focus on other work**

## Overview

Background tasks allow you to send work to Claude Code that runs independently while you continue working. This is ideal for long-running operations, parallel development, and tasks that don't require constant interaction.

### What You'll Learn

- Starting background tasks with the `&` prefix
- Managing multiple parallel tasks
- Monitoring task progress with `/tasks`
- Best practices for autonomous execution

### Prerequisites

- Claude Code installed
- Pro, Max, Team, or Enterprise subscription (for web-based tasks)

---

## Part 1: Understanding Background Tasks

### What Are Background Tasks?

Background tasks run Claude Code sessions independently of your main terminal:

| Feature | Description |
|---------|-------------|
| **Asynchronous** | Tasks run without blocking your terminal |
| **Parallel** | Multiple tasks can run simultaneously |
| **Web-based** | Tasks execute on Anthropic's cloud infrastructure |
| **Persistent** | Progress is saved even if you close your terminal |

### When to Use Background Tasks

- **Bug fixes** - Well-defined issues with clear solutions
- **Refactoring** - Code improvements that don't need steering
- **Documentation** - Generating docs for code
- **Testing** - Writing or updating test suites
- **Multiple features** - Working on several unrelated tasks

---

## Part 2: Starting Background Tasks

### The `&` Prefix

Start any message with `&` to send it as a background task:

```bash
& Fix the authentication bug in src/auth/login.ts
```

This creates a new web session that:
1. Clones your repository
2. Copies conversation context
3. Runs the task autonomously
4. Notifies you when complete

### Command Line Method

```bash
claude --remote "Fix the authentication bug in src/auth/login.ts"
```

### Examples

**Simple bug fix:**
```bash
& Fix the null pointer exception in UserService.getById()
```

**Refactoring:**
```bash
& Refactor the PaymentController to use dependency injection
```

**Documentation:**
```bash
& Generate JSDoc comments for all functions in src/utils/
```

**Test writing:**
```bash
& Write unit tests for the EmailValidator class with >90% coverage
```

---

## Part 3: Managing Multiple Tasks

### Running Tasks in Parallel

Send multiple tasks simultaneously:

```bash
& Fix the login validation bug
& Update the API documentation
& Refactor the logger to use structured output
& Add pagination to the user list endpoint
```

Each `&` creates an independent web session.

### Viewing All Tasks

Use `/tasks` to see all background tasks:

```
/tasks
```

This shows:
- Task status (running, completed, failed)
- Task description
- Duration
- Branch information

### Interacting with Tasks

From the `/tasks` list:
- Press `Enter` to view task details
- Press `t` to teleport the task to your terminal
- Press `r` to refresh status

---

## Part 4: Task Lifecycle

### Task States

| State | Description |
|-------|-------------|
| **Queued** | Task is waiting to start |
| **Running** | Task is actively executing |
| **Completed** | Task finished successfully |
| **Failed** | Task encountered an error |
| **Cancelled** | Task was manually stopped |

### Completion Notifications

When a task completes:
1. You receive a notification (if enabled)
2. Changes are committed to a branch
3. You can create a PR or teleport to review

### Bringing Tasks Back

Use `/teleport` or `/tp` to bring a web session to your terminal:

```bash
/teleport
```

This opens an interactive picker to select which session to bring back.

---

## Part 5: Best Practices

### 1. Write Clear, Self-Contained Tasks

**Good:**
```bash
& Fix the TypeError in src/components/Dashboard.tsx line 45 where
  user.profile is accessed before null check. Add proper null handling.
```

**Avoid:**
```bash
& Fix the bug
```

### 2. Plan Locally, Execute Remotely

Start in plan mode to collaborate on the approach:

```bash
claude --permission-mode plan

# Discuss the task and create a plan
# Then send to background for execution
& Execute the migration plan we discussed
```

### 3. Use for Independent Tasks

Background tasks work best for:
- Tasks that don't depend on each other
- Tasks with clear success criteria
- Tasks that don't need ongoing steering

### 4. Monitor Resource Usage

Multiple parallel tasks consume rate limits proportionally. Be mindful when running many simultaneous tasks.

### 5. Review Before Merging

Always review background task changes before merging:
- Check the diff carefully
- Run tests locally
- Verify the fix addresses the issue

---

## Part 6: Advanced Patterns

### Pattern 1: Divide and Conquer

Split large tasks into parallel subtasks:

```bash
& Migrate UserService to TypeScript
& Migrate PaymentService to TypeScript
& Migrate NotificationService to TypeScript
& Update type definitions in types/
```

### Pattern 2: Test-First Development

```bash
& First write comprehensive tests for the OrderProcessor class,
  then implement the class to pass all tests.
  Include edge cases for empty orders, cancelled orders, and refunds.
```

### Pattern 3: Exploration Tasks

```bash
& Analyze the authentication codebase and create a report on:
  1. Current security vulnerabilities
  2. Recommended improvements
  3. Migration path to OAuth2
  
  Save the report to docs/auth-analysis.md
```

### Pattern 4: Batch Updates

```bash
& Update all React class components in src/components/ to functional
  components with hooks. Preserve all existing functionality.
```

---

## Part 7: Troubleshooting

### Task Not Starting

| Issue | Solution |
|-------|----------|
| Rate limited | Wait for quota reset or reduce parallel tasks |
| Not subscribed | Ensure you have Pro, Max, Team, or Enterprise |
| Repository access | Connect GitHub account at claude.ai/code |

### Task Failed

1. Check `/tasks` for error details
2. Teleport the task to review the state
3. Review any partial changes
4. Restart with more specific instructions

### Can't Teleport

| Issue | Solution |
|-------|----------|
| Uncommitted changes | Stash or commit local changes first |
| Wrong repository | Navigate to the correct repo |
| Branch missing | Ensure the web session pushed changes |

---

## Quick Reference

### Commands

| Command | Description |
|---------|-------------|
| `& <prompt>` | Start background task |
| `claude --remote "<prompt>"` | Start from command line |
| `/tasks` | View all background tasks |
| `/teleport` or `/tp` | Bring task to terminal |

### Task Management Shortcuts

| In /tasks | Action |
|-----------|--------|
| `Enter` | View task details |
| `t` | Teleport to terminal |
| `r` | Refresh status |
| `q` | Exit task list |

---

## Sources

- [Claude Code on the Web](https://code.claude.com/docs/en/claude-code-on-the-web.md)
- [Common Workflows](https://code.claude.com/docs/en/common-workflows.md)

---

## Next Steps

1. **Try a simple task** - Start with a well-defined bug fix
2. **Run parallel tasks** - Work on multiple features simultaneously
3. **Combine with teleport** - Review and continue tasks locally
