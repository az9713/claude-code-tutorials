# Tutorial 30: CI/CD Best Practices

> **Optimize Claude Code in automated pipelines**

## Overview

Best practices for integrating Claude Code into CI/CD workflows effectively and securely.

---

## Part 1: Security Best Practices

### Never Commit Secrets

```yaml
# Use CI/CD variables
variables:
  ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

### Use OIDC When Possible

No long-lived keys:

```yaml
# AWS Bedrock OIDC
- aws sts assume-role-with-web-identity
  --role-arn "$AWS_ROLE_TO_ASSUME"
  --role-session-name "ci-claude"
```

### Limit Permissions

```yaml
--permission-mode acceptEdits
--allowedTools "Read(*) Edit(*) Write(*)"
```

### Review All Changes

Treat Claude's PRs like any contributor's code.

---

## Part 2: Cost Optimization

### Set Timeouts

```yaml
timeout: 30 minutes
```

### Limit Iterations

```bash
claude -p "..." --max-turns 10
```

### Use Specific Prompts

```yaml
# Bad
-p "Fix bugs"

# Good  
-p "Fix the TypeError in UserService.getById on line 45"
```

### Control Concurrency

Limit parallel Claude jobs to manage rate limits.

---

## Part 3: Performance Optimization

### Keep CLAUDE.md Focused

Short, clear guidelines process faster.

### Provide Clear Context

```yaml
-p "Fix bug in @src/auth/login.ts:45-60"
```

### Cache Dependencies

```yaml
cache:
  key: npm-${{ hashFiles('package-lock.json') }}
  paths:
    - node_modules/
```

---

## Part 4: Workflow Patterns

### Issue-Triggered

```yaml
on:
  issues:
    types: [labeled]
jobs:
  claude:
    if: github.event.label.name == 'claude'
```

### PR Review

```yaml
on:
  pull_request:
    types: [opened, synchronize]
jobs:
  review:
    steps:
      - run: claude -p "Review this PR for bugs and security issues"
```

### Scheduled Analysis

```yaml
on:
  schedule:
    - cron: '0 2 * * 1'  # Weekly Monday 2am
jobs:
  audit:
    steps:
      - run: claude -p "Audit codebase for security vulnerabilities"
```

---

## Part 5: Error Handling

### Retry Logic

```yaml
- run: claude -p "..."
  retry:
    max_attempts: 3
    delay: 30s
```

### Fallback Behavior

```yaml
- run: claude -p "..." || echo "Claude failed, continuing"
  continue-on-error: true
```

### Notifications

```yaml
- run: |
    if ! claude -p "..."; then
      slack-notify "Claude job failed"
    fi
```

---

## Part 6: Testing Claude's Output

### Verify Changes

```yaml
- run: claude -p "Fix the bug"
- run: npm test
- run: npm run lint
```

### Require Passing Tests

```yaml
jobs:
  claude:
    steps:
      - run: claude -p "Implement feature"
      - run: npm test
        if: always()
```

---

## Quick Reference

### Recommended Flags

| Flag | Purpose |
|------|---------|
| `--permission-mode acceptEdits` | Auto-accept file changes |
| `--max-turns N` | Limit iterations |
| `--allowedTools "..."` | Restrict tool access |
| `--output-format json` | Structured output |
| `-p "..."` | Non-interactive prompt |

### Security Checklist

- [ ] API keys in CI/CD variables
- [ ] OIDC for cloud providers
- [ ] Minimal tool permissions
- [ ] Review all generated PRs
- [ ] Branch protection enabled

---

## Sources

- [GitHub Actions](https://code.claude.com/docs/en/github-actions.md)
- [GitLab CI/CD](https://code.claude.com/docs/en/gitlab-ci-cd.md)
- [Headless Mode](https://code.claude.com/docs/en/headless.md)
