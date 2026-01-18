# Smart Git Commit

Create a well-crafted git commit with conventional commit format.

## Instructions

1. Run `git status` and `git diff --staged` to see what's being committed
2. If nothing is staged, run `git diff` to see unstaged changes and ask if I should stage them
3. Analyze the changes to understand:
   - What type of change (feat, fix, refactor, docs, test, chore, perf, style, build, ci)
   - What scope/area is affected
   - What the change accomplishes (the "why", not just the "what")
4. Run `git log --oneline -5` to match the repository's commit style
5. Draft a commit message following Conventional Commits:
   ```
   <type>(<scope>): <description>

   [optional body with more details]

   [optional footer]
   ```
6. Present the commit message for approval
7. Execute the commit with the approved message

## Arguments

$ARGUMENTS - Optional: specific files to commit or commit message hint

## Commit Types

- **feat**: New feature
- **fix**: Bug fix
- **refactor**: Code change that neither fixes a bug nor adds a feature
- **docs**: Documentation only
- **test**: Adding or fixing tests
- **chore**: Maintenance tasks
- **perf**: Performance improvement
- **style**: Code style (formatting, semicolons, etc.)
- **build**: Build system or dependencies
- **ci**: CI configuration

## Examples

- `/commit` - Commit all staged changes
- `/commit auth module` - Hint that changes relate to auth
- `/commit --all` - Stage and commit all changes
