# Create Pull Request

Create a well-documented pull request with proper description.

## Instructions

1. Check current branch: `git branch --show-current`
2. Ensure we're not on main/master: warn if so
3. Check for uncommitted changes: `git status`
4. Get the diff against the base branch: `git diff main...HEAD` (or master)
5. Review all commits on this branch: `git log main..HEAD --oneline`
6. Analyze the changes to understand:
   - What feature/fix this PR implements
   - What files were modified and why
   - Any breaking changes or migration needs
   - Testing that was done
7. Check if remote branch exists, push if needed: `git push -u origin HEAD`
8. Create PR using GitHub CLI:
   ```bash
   gh pr create --title "..." --body "..."
   ```

## PR Template

```markdown
## Summary
<!-- 2-3 bullet points describing what this PR does -->

## Changes
<!-- List of significant changes -->

## Testing
<!-- How was this tested? -->

## Checklist
- [ ] Tests pass
- [ ] Linting passes
- [ ] Documentation updated (if needed)
```

## Arguments

$ARGUMENTS - Optional: PR title hint or target branch

## Examples

- `/pr` - Create PR with auto-generated description
- `/pr Add user authentication` - Create PR with title hint
- `/pr --base develop` - Create PR targeting develop branch
