# Tutorial 45: Pull Request Workflows

> **Create and manage pull requests with Claude Code**

## Overview

Streamline PR creation and management with Claude Code.

---

## Part 1: Creating PRs

### Basic PR

```bash
Create a PR for these changes with a description
```

Claude will:
1. Stage changes
2. Create commit
3. Push branch
4. Open PR

### Detailed PR

```bash
Create a PR with:
- Title: "Add user authentication"
- Description with what/why/how
- Test instructions
- Screenshots if UI changes
```

---

## Part 2: PR Descriptions

### Generate Description

```bash
Generate a PR description for my current changes.
Include:
- Summary of changes
- Files modified
- Testing done
```

### Update Description

```bash
Update the PR description to include the breaking changes
we discussed.
```

---

## Part 3: Commit Strategy

### Atomic Commits

```bash
Split these changes into logical commits:
1. Refactor database layer
2. Add new API endpoints
3. Update tests
```

### Commit Messages

```bash
Generate conventional commit messages for my staged changes.
```

---

## Part 4: PR Review

### Self Review

```bash
Review my changes before I submit the PR.
Check for:
- Bugs
- Missing tests
- Documentation
```

### Address Feedback

```bash
Address this PR feedback:
[paste reviewer comment]

Make the changes and update the PR.
```

---

## Part 5: CI Integration

### GitHub Actions

PRs can trigger Claude for review:

```yaml
on:
  pull_request:
    types: [opened]
jobs:
  review:
    steps:
      - run: claude -p "Review this PR"
```

### GitLab CI

```yaml
claude:
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
  script:
    - claude -p "Review MR changes"
```

---

## Quick Reference

### PR Commands

| Task | Command |
|------|---------|
| Create PR | "Create a PR for these changes" |
| Add description | "Generate PR description" |
| Address feedback | "Address this PR comment: [...]" |

### Good PR Practices

- Atomic commits
- Clear descriptions
- Self-review first
- Link related issues

---

## Sources

- [Common Workflows](https://code.claude.com/docs/en/common-workflows.md)
- [GitHub Actions](https://code.claude.com/docs/en/github-actions.md)
