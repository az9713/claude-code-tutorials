# Code Review

Review code changes for quality, bugs, and best practices.

## Instructions

1. Determine what to review:
   - If a PR number is provided: `gh pr diff $PR_NUMBER`
   - If on a branch: `git diff main...HEAD`
   - If uncommitted changes: `git diff`
2. Analyze the changes for:
   - **Correctness**: Logic errors, edge cases, null checks
   - **Security**: Injection vulnerabilities, auth issues, data exposure
   - **Performance**: N+1 queries, unnecessary loops, memory leaks
   - **Maintainability**: Code clarity, naming, complexity
   - **Testing**: Coverage gaps, missing edge cases
   - **Standards**: Coding conventions, patterns consistency
3. Provide feedback in this format:

## Review Output Format

### 🔴 Critical Issues
Issues that must be fixed before merge.

### 🟡 Suggestions
Improvements that would make the code better.

### 🟢 Good Practices
Things done well (brief acknowledgment).

### Summary
Overall assessment and recommendation (approve/request changes).

## Arguments

$ARGUMENTS - PR number, file path, or branch name to review

## Examples

- `/review` - Review current uncommitted changes
- `/review 123` - Review PR #123
- `/review src/auth/` - Review changes in auth directory
- `/review feature/login` - Review specific branch
