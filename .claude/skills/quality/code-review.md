---
name: code-review
description: Code review checklist and best practices
---

# Code Review Best Practices

## Review Checklist

### Correctness
- [ ] Does the code do what it's supposed to do?
- [ ] Are edge cases handled?
- [ ] Are error conditions handled gracefully?
- [ ] Is the logic correct? No off-by-one errors?
- [ ] Are there any race conditions?

### Security
- [ ] Is user input validated and sanitized?
- [ ] Are SQL/NoSQL queries parameterized?
- [ ] Is sensitive data properly protected?
- [ ] Are authentication/authorization checks in place?
- [ ] No hardcoded secrets or credentials?

### Performance
- [ ] Are there any N+1 query problems?
- [ ] Is data being fetched efficiently?
- [ ] Are there unnecessary loops or computations?
- [ ] Is caching used appropriately?
- [ ] Are database queries indexed?

### Maintainability
- [ ] Is the code easy to understand?
- [ ] Are variable/function names descriptive?
- [ ] Is the code DRY (Don't Repeat Yourself)?
- [ ] Is complexity kept low?
- [ ] Are functions small and focused?

### Testing
- [ ] Are there tests for new functionality?
- [ ] Do tests cover edge cases?
- [ ] Are tests readable and maintainable?
- [ ] Is test coverage adequate?

### Documentation
- [ ] Is complex logic documented?
- [ ] Are public APIs documented?
- [ ] Is the PR description clear?
- [ ] Are breaking changes noted?

## Giving Feedback

### Be Constructive

```markdown
# ❌ Not helpful
"This code is bad"
"Wrong"
"Why would you do it this way?"

# ✅ Constructive
"Consider using `Array.find()` here instead of a loop - it's more readable
and clearly expresses the intent to find a single item."

"This could lead to a null pointer exception if `user` is undefined.
Could you add a null check? Example: `if (!user) return null;`"
```

### Use Prefixes

```markdown
# Indicate severity/type
[nit] Minor style suggestion, optional
[suggestion] Would improve the code, consider it
[question] Need clarification, not blocking
[blocking] Must be fixed before merge

# Examples
[nit] Could use a more descriptive variable name here
[suggestion] This could be simplified using destructuring
[question] What's the reason for this specific timeout value?
[blocking] This SQL query is vulnerable to injection
```

### Praise Good Work

```markdown
"Nice use of the strategy pattern here - makes this much more extensible!"
"Good catch on that edge case!"
"This refactoring makes the code much cleaner."
```

## Receiving Feedback

### Best Practices

1. **Don't take it personally** - Review is about the code, not you
2. **Assume good intent** - Reviewers want to help improve the code
3. **Ask for clarification** - If feedback is unclear, ask questions
4. **Explain your reasoning** - If you disagree, explain why
5. **Be open to learning** - Different perspectives can improve code

### Responding to Feedback

```markdown
# Acknowledging feedback
"Good catch! Fixed in abc123"
"You're right, I've updated the approach"

# Asking for clarification
"Could you elaborate on what you mean by 'more defensive'?"
"What would you suggest as an alternative?"

# Respectfully disagreeing
"I considered that approach, but went with this because [reason].
What do you think?"
```

## Review Workflow

### Before Requesting Review

1. Self-review your changes first
2. Ensure CI passes (tests, lint, build)
3. Write a clear PR description
4. Keep PRs small and focused (< 400 lines ideal)
5. Add relevant context and screenshots

### PR Description Template

```markdown
## Summary
Brief description of what this PR does

## Changes
- Added X
- Modified Y
- Fixed Z

## Testing
- [ ] Unit tests added/updated
- [ ] Manual testing done
- How to test: [steps]

## Screenshots (if UI changes)
Before | After
--- | ---

## Notes for Reviewers
Any specific areas to focus on or context needed
```

### Reviewer Guidelines

1. **Respond promptly** - Don't block others for long
2. **Review thoroughly** - One careful review > multiple quick ones
3. **Test locally if needed** - For complex changes
4. **Approve when ready** - Don't hold up for minor nits

## Common Patterns to Look For

### Code Smells

| Pattern | Issue | Suggestion |
|---------|-------|------------|
| Deeply nested conditionals | Hard to read | Extract functions, use early returns |
| Magic numbers/strings | Unclear meaning | Use named constants |
| Long functions (>50 lines) | Too much responsibility | Extract smaller functions |
| Duplicate code | Maintenance burden | Extract shared function |
| Commented-out code | Clutter | Remove (git has history) |
| TODO without issue | Forgotten work | Create ticket or remove |

### Red Flags

```typescript
// 🚩 Swallowing errors
try { doSomething(); } catch (e) { }

// 🚩 console.log in production code
console.log('debug:', data);

// 🚩 Hardcoded credentials
const apiKey = 'sk_live_abc123';

// 🚩 SQL concatenation
const query = `SELECT * FROM users WHERE id = '${userId}'`;

// 🚩 Disabling security
// eslint-disable-next-line @typescript-eslint/no-explicit-any
```
