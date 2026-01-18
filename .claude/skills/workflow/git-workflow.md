---
name: git-workflow
description: Git workflow patterns for branching, commits, PRs, and collaboration
---

# Git Workflow Patterns

## Branch Naming Conventions

Use descriptive, kebab-case branch names:

```
feature/<ticket>-<description>    # feat/AUTH-123-user-login
fix/<ticket>-<description>        # fix/BUG-456-null-pointer
hotfix/<description>              # hotfix/critical-security-patch
refactor/<description>            # refactor/extract-auth-service
docs/<description>                # docs/api-documentation
test/<description>                # test/add-integration-tests
chore/<description>               # chore/update-dependencies
```

## Commit Message Format (Conventional Commits)

```
<type>(<scope>): <subject>

[optional body]

[optional footer(s)]
```

**Types:** feat, fix, docs, style, refactor, perf, test, build, ci, chore

**Examples:**
```
feat(auth): add OAuth2 Google login

Implements Google OAuth2 authentication flow with:
- Token exchange endpoint
- User profile fetching
- Session management

Closes #123
```

## Git Workflow Commands

### Starting New Work

```bash
# Update main and create feature branch
git checkout main && git pull
git checkout -b feature/TICKET-description

# Or use worktrees for parallel work
git worktree add ../project-feature feature/TICKET-description
```

### During Development

```bash
# Stage specific files
git add src/auth/*.ts

# Stage interactively
git add -p

# Commit with message
git commit -m "feat(auth): add login validation"

# Amend last commit (before push)
git commit --amend

# Stash work in progress
git stash push -m "WIP: auth refactor"
git stash pop
```

### Keeping Branch Updated

```bash
# Rebase onto main (preferred for clean history)
git fetch origin
git rebase origin/main

# Or merge if team prefers
git merge origin/main
```

### Before Creating PR

```bash
# Squash commits if needed
git rebase -i HEAD~3

# Push to remote
git push -u origin HEAD

# Force push after rebase (only on feature branches!)
git push --force-with-lease
```

### Pull Request Process

1. Create PR with clear description
2. Request reviews from appropriate team members
3. Address review feedback
4. Squash and merge (or rebase, per team convention)
5. Delete feature branch after merge

## Git Best Practices

1. **Commit early, commit often** - Small, focused commits
2. **Never force push to shared branches** - Only on personal feature branches
3. **Write meaningful commit messages** - Future you will thank you
4. **Keep PRs small** - Easier to review, faster to merge
5. **Rebase before merge** - Clean, linear history
6. **Delete merged branches** - Keep repository clean

## Handling Common Scenarios

### Undo Last Commit (Keep Changes)
```bash
git reset --soft HEAD~1
```

### Undo Last Commit (Discard Changes)
```bash
git reset --hard HEAD~1
```

### Cherry-pick a Commit
```bash
git cherry-pick <commit-sha>
```

### Find When Bug Was Introduced
```bash
git bisect start
git bisect bad HEAD
git bisect good v1.0.0
# Test each commit, mark good/bad
git bisect reset
```

### Recover Deleted Branch
```bash
git reflog
git checkout -b recovered-branch <sha>
```
