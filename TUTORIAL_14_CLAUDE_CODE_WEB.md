# Tutorial 14: Claude Code on the Web

> **Run Claude Code tasks asynchronously on secure cloud infrastructure**

## Overview

Claude Code on the Web lets you kick off Claude Code tasks from the Claude web app at [claude.ai/code](https://claude.ai/code). This is perfect for parallel work, repositories not on your local machine, and well-defined tasks that don't require frequent steering.

### What You'll Learn

- Starting and managing cloud-based Claude Code sessions
- Moving tasks between terminal and web (teleport feature)
- Configuring cloud environments
- Understanding network access controls
- Working with the Claude iOS app for mobile monitoring

### Prerequisites

- Claude Code installed locally
- Pro, Max, Team, or Enterprise subscription
- GitHub account connected to Claude

---

## Part 1: Getting Started with Claude Code on the Web

### What is Claude Code on the Web?

Claude Code on the web runs your coding tasks on Anthropic-managed virtual machines. This enables:

| Use Case | Description |
|----------|-------------|
| **Parallel work** | Tackle multiple bug fixes simultaneously |
| **Remote repositories** | Work on code you don't have checked out locally |
| **Answering questions** | Ask about code architecture without local setup |
| **Bug fixes** | Well-defined tasks that run autonomously |
| **Backend changes** | Write tests, then code to pass those tests |

### Who Can Use It?

Claude Code on the web is available to:
- Pro users
- Max users
- Team premium seat users
- Enterprise premium seat users

### Quick Start

1. Visit [claude.ai/code](https://claude.ai/code)
2. Connect your GitHub account
3. Install the Claude GitHub app in your repositories
4. Select your default environment
5. Submit your coding task
6. Review changes and create a pull request

---

## Part 2: How Cloud Sessions Work

When you start a task on Claude Code on the web:

```
┌─────────────────────────────────────────────────────────────────────┐
│ 1. Repository Cloning                                                │
│    Your repo is cloned to an Anthropic-managed virtual machine      │
├─────────────────────────────────────────────────────────────────────┤
│ 2. Environment Setup                                                 │
│    Claude prepares a secure cloud environment with your code        │
├─────────────────────────────────────────────────────────────────────┤
│ 3. Network Configuration                                             │
│    Internet access configured based on your settings                │
├─────────────────────────────────────────────────────────────────────┤
│ 4. Task Execution                                                    │
│    Claude analyzes code, makes changes, runs tests                  │
├─────────────────────────────────────────────────────────────────────┤
│ 5. Completion                                                        │
│    You're notified and can create a PR with the changes             │
└─────────────────────────────────────────────────────────────────────┘
```

### Cloud Environment Details

The universal image includes pre-installed:

**Languages & Runtimes:**
- Python 3.x with pip, poetry, and scientific libraries
- Node.js (latest LTS) with npm, yarn, pnpm, and bun
- Ruby 3.1.6, 3.2.6, 3.3.6 (default: 3.3.6) with rbenv
- PHP 8.4.14
- Java (OpenJDK) with Maven and Gradle
- Go (latest stable)
- Rust with cargo
- C++ (GCC and Clang)

**Databases:**
- PostgreSQL 16
- Redis 7.0

**Check Available Tools:**
```bash
check-tools
```

---

## Part 3: Moving Tasks Between Web and Terminal

### From Terminal to Web (& prefix)

Start a message with `&` inside Claude Code to send a task to run on the web:

```bash
& Fix the authentication bug in src/auth/login.ts
```

**What happens:**
1. Creates a new web session on claude.ai
2. Copies your current conversation context
3. Task runs in the cloud while you continue locally
4. Use `/tasks` to check progress

**Command line version:**
```bash
claude --remote "Fix the authentication bug in src/auth/login.ts"
```

### Tips for Background Tasks

**Plan Locally, Execute Remotely:**
```bash
# Start in plan mode to collaborate on the approach
claude --permission-mode plan

# Once satisfied, send to web for autonomous execution
& Execute the migration plan we discussed
```

**Run Tasks in Parallel:**
```bash
& Fix the flaky test in auth.spec.ts
& Update the API documentation
& Refactor the logger to use structured output
```

Each `&` creates its own independent web session. Monitor all with `/tasks`.

### From Web to Terminal (Teleport)

Several ways to pull a web session into your terminal:

| Method | Command | Description |
|--------|---------|-------------|
| `/teleport` | `/teleport` or `/tp` | Interactive picker of web sessions |
| `--teleport` | `claude --teleport` | From command line |
| `/tasks` | `/tasks` then press `t` | Teleport from task list |
| Web interface | Click "Open in CLI" | Copy command to paste |

**Teleport Requirements:**

| Requirement | Details |
|-------------|---------|
| Clean git state | No uncommitted changes (will prompt to stash) |
| Correct repository | Must be same repo, not a fork |
| Branch available | Web session branch must be pushed to remote |
| Same account | Same Claude.ai account as web session |

> **Note:** Session handoff is one-way. You can pull web sessions into terminal, but can't push an existing terminal session to web. The `&` prefix creates a *new* web session.

---

## Part 4: Environment Configuration

### Creating and Managing Environments

**Add a new environment:**
1. Select current environment to open the selector
2. Select "Add environment"
3. Specify name, network access level, and environment variables

**Update existing environment:**
1. Select current environment
2. Click settings button next to name
3. Update as needed

**Select default environment from terminal:**
```bash
/remote-env
```

### Environment Variables

Specify as key-value pairs in `.env` format:
```
API_KEY=your_api_key
DEBUG=true
```

### Dependency Management with SessionStart Hooks

Configure automatic dependency installation in `.claude/settings.json`:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/scripts/install_pkgs.sh"
          }
        ]
      }
    ]
  }
}
```

**Create the install script at `scripts/install_pkgs.sh`:**
```bash
#!/bin/bash
npm install
pip install -r requirements.txt
exit 0
```

Make it executable:
```bash
chmod +x scripts/install_pkgs.sh
```

### Local vs Remote Hook Execution

Run hooks only in remote environments:
```bash
#!/bin/bash

# Only run in remote environments
if [ "$CLAUDE_CODE_REMOTE" != "true" ]; then
  exit 0
fi

npm install
pip install -r requirements.txt
```

### Persisting Environment Variables

SessionStart hooks can persist variables for subsequent commands by writing to `CLAUDE_ENV_FILE`:
```bash
echo "MY_VAR=value" >> "$CLAUDE_ENV_FILE"
```

---

## Part 5: Network Access and Security

### Network Access Levels

| Level | Description |
|-------|-------------|
| Limited (default) | Access only to allowlisted domains |
| No internet | Completely isolated (still can reach Anthropic API) |
| Full internet | Unrestricted access |

### GitHub Proxy

All GitHub operations go through a dedicated proxy that:
- Manages GitHub authentication securely
- Uses scoped credentials inside the sandbox
- Restricts git push to current working branch
- Enables seamless cloning, fetching, and PR operations

### Security Proxy

Outbound internet traffic passes through HTTP/HTTPS proxy providing:
- Protection against malicious requests
- Rate limiting and abuse prevention
- Content filtering

### Default Allowed Domains

When using "Limited" network access, these are allowed:

**Anthropic Services:**
- api.anthropic.com, claude.ai

**Version Control:**
- github.com, gitlab.com, bitbucket.org (and subdomains)

**Container Registries:**
- registry-1.docker.io, ghcr.io, public.ecr.aws

**Cloud Platforms:**
- *.googleapis.com, *.amazonaws.com, azure.com

**Package Managers:**
- npmjs.org, pypi.org, rubygems.org, crates.io, proxy.golang.org, maven.org

**Linux Distributions:**
- archive.ubuntu.com, security.ubuntu.com

### Security Best Practices

1. **Principle of least privilege**: Only enable minimum network access required
2. **Audit regularly**: Review allowed domains periodically
3. **Use HTTPS**: Always prefer HTTPS endpoints

---

## Part 6: Security and Isolation

Claude Code on the web provides strong security guarantees:

| Feature | Description |
|---------|-------------|
| **Isolated VMs** | Each session runs in isolated, Anthropic-managed VM |
| **Network controls** | Limited by default, can be disabled |
| **Credential protection** | Sensitive credentials never inside sandbox |
| **Secure analysis** | Code analyzed within isolated VMs before PR creation |

> **Note:** With network access disabled, Claude Code can still communicate with the Anthropic API, which may allow data to exit the isolated VM.

---

## Part 7: Practical Examples

### Example 1: Bug Fix Workflow

```bash
# Start in plan mode locally
claude --permission-mode plan

# Discuss the bug and create a plan
You: The login page crashes when users enter special characters in email

# Claude explains the issue and proposes a fix

# Send to cloud for implementation
& Implement the fix we discussed for the login validation bug
```

### Example 2: Parallel Feature Development

```bash
# Start multiple independent tasks
& Add dark mode toggle to settings page
& Implement API rate limiting middleware
& Update user profile to support avatar uploads
& Fix mobile responsiveness issues on dashboard

# Monitor all tasks
/tasks

# Teleport to completed task
/teleport
```

### Example 3: Remote Repository Review

1. Go to [claude.ai/code](https://claude.ai/code)
2. Select a repository from GitHub
3. Ask: "Review the authentication implementation and suggest improvements"
4. Claude analyzes the code and provides recommendations
5. Ask follow-up: "Implement the JWT refresh token suggestion"
6. Create PR with changes

### Example 4: Test-Driven Development

```bash
& First write comprehensive tests for the PaymentProcessor class,
  then implement the class to pass all tests.
  Include edge cases for failed payments, refunds, and currency conversion.
```

### Example 5: Documentation Generation

```bash
& Generate API documentation for all endpoints in src/api/,
  including request/response examples and error codes.
  Output as OpenAPI 3.0 spec.
```

---

## Part 8: Best Practices

### 1. Use CLAUDE.md for Project Context

Create `CLAUDE.md` in your repository root to document:
- Project architecture
- Coding standards
- Testing requirements
- Deployment procedures

Claude reads this file during runs and follows your conventions.

### 2. Leverage AGENTS.md

If you have an `AGENTS.md` file, source it in your `CLAUDE.md`:
```markdown
@AGENTS.md
```

This maintains a single source of truth for agent instructions.

### 3. Design Short, Focused Tasks

| ✅ Good Task | ❌ Poor Task |
|-------------|-------------|
| "Fix the null pointer in UserService.getById()" | "Fix all the bugs" |
| "Add input validation to the registration form" | "Make the app more secure" |
| "Refactor PaymentController to use dependency injection" | "Clean up the codebase" |

### 4. Use Plan Mode for Complex Tasks

Always start complex tasks in plan mode locally:
```bash
claude --permission-mode plan
```

Collaborate on the approach, then send to web for execution.

### 5. Monitor from Mobile

Use the Claude iOS app to:
- Check task progress on the go
- Steer Claude with follow-up messages
- Review and approve changes
- Get notifications when tasks complete

---

## Part 9: Pricing and Limitations

### Pricing

Claude Code on the web shares rate limits with all other Claude and Claude Code usage. Running multiple parallel tasks consumes rate limits proportionately.

### Current Limitations

| Limitation | Details |
|------------|---------|
| **Repository authentication** | Can only move sessions from web to local with same account |
| **Platform restrictions** | Only works with GitHub repositories (not GitLab, Bitbucket) |

---

## Quick Reference

### Commands

| Command | Description |
|---------|-------------|
| `& <prompt>` | Send task to run on web |
| `claude --remote "<prompt>"` | Start web session from command line |
| `/tasks` | View all background tasks |
| `/teleport` or `/tp` | Interactive session picker |
| `claude --teleport` | Teleport from command line |
| `/remote-env` | Select default remote environment |

### Environment Variables

| Variable | Purpose |
|----------|---------|
| `CLAUDE_CODE_REMOTE` | `"true"` when running in cloud |
| `CLAUDE_PROJECT_DIR` | Project directory path |
| `CLAUDE_ENV_FILE` | File to write persistent env vars |

### Troubleshooting

| Issue | Solution |
|-------|----------|
| Can't teleport | Check for uncommitted changes, stash if needed |
| Branch not found | Ensure web session pushed changes to remote |
| Wrong repository | Run teleport from correct repo checkout |
| Authentication fails | Verify same Claude.ai account |

---

## Sources

- [Claude Code on the Web - Official Documentation](https://code.claude.com/docs/en/claude-code-on-the-web.md)
- [Hooks Configuration](https://code.claude.com/docs/en/hooks.md)
- [Settings Reference](https://code.claude.com/docs/en/settings.md)
- [Security](https://code.claude.com/docs/en/security.md)

---

## Next Steps

1. **Set up your first cloud session** - Visit [claude.ai/code](https://claude.ai/code) and connect a repository
2. **Try the teleport feature** - Start a task on web, then bring it to your terminal
3. **Configure SessionStart hooks** - Automate environment setup for your projects
4. **Explore parallel execution** - Run multiple tasks simultaneously with `&` prefix
