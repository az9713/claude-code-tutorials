---
name: debugging
description: Systematic debugging approach - diagnose before fixing
---

# Systematic Debugging

## The Debugging Process

```
┌─────────────────────────────────────────────────────────┐
│ 1. REPRODUCE → 2. ISOLATE → 3. DIAGNOSE → 4. FIX → 5. VERIFY │
└─────────────────────────────────────────────────────────┘
```

## Step 1: Reproduce the Bug

Before anything else, reliably reproduce the issue:

```bash
# Document exact steps
1. Navigate to /users/settings
2. Click "Update Profile"
3. Enter invalid email "test@"
4. Click Submit
5. ERROR: Unhandled exception (expected: validation error)
```

**Questions to answer:**
- Does it happen every time?
- Does it happen in all environments?
- What are the exact inputs that trigger it?
- When did it start happening?

## Step 2: Isolate the Problem

Narrow down where the bug lives:

### Binary Search Approach
```typescript
// Add logging at midpoints to narrow down
console.log('Checkpoint A - data:', data);  // Works here
// ... 50 lines of code ...
console.log('Checkpoint B - data:', data);  // Broken here
// Narrow down between A and B
```

### Git Bisect for Regression
```bash
git bisect start
git bisect bad HEAD
git bisect good v1.0.0
# Git will help you find the commit that introduced the bug
```

### Minimal Reproduction
Strip away everything unrelated until you have the smallest code that reproduces the bug.

## Step 3: Diagnose Root Cause

### Read the Error Carefully
```
TypeError: Cannot read property 'email' of undefined
    at validateUser (src/validators/user.ts:23:15)
    at handleSubmit (src/components/Form.tsx:45:10)
```

**Extract information:**
- Error type: `TypeError`
- What's undefined: `user` object
- Where: `validateUser` at line 23
- Call chain: `handleSubmit` → `validateUser`

### Common Bug Categories

| Symptom | Likely Cause |
|---------|--------------|
| `undefined is not a function` | Wrong import, typo, missing method |
| `Cannot read property of undefined` | Null/undefined not checked |
| Works locally, fails in prod | Environment differences, missing env vars |
| Intermittent failures | Race condition, timing issue |
| Wrong data displayed | State management, stale closure |
| Memory leak | Event listeners not cleaned, circular refs |

### Debugging Tools

```typescript
// Strategic console.log
console.log('Function called with:', { args, context: this });

// Debugger statement
debugger; // Pauses execution in DevTools

// JSON for complex objects
console.log(JSON.stringify(data, null, 2));

// Stack trace
console.trace('How did we get here?');
```

## Step 4: Fix the Bug

### Fix Only the Bug
```typescript
// ❌ Don't refactor while fixing
function validateUser(user) {
  // FIXED: Add null check
  // Also: renamed variable, extracted function, added types...
}

// ✅ Minimal, focused fix
function validateUser(user) {
  if (!user) return { valid: false, error: 'User required' };
  // ... rest unchanged
}
```

### Write a Test First
```typescript
// Proves the bug exists and prevents regression
it('handles undefined user gracefully', () => {
  expect(() => validateUser(undefined)).not.toThrow();
  expect(validateUser(undefined)).toEqual({
    valid: false,
    error: 'User required'
  });
});
```

## Step 5: Verify the Fix

1. **Run the reproduction steps** - Bug should be gone
2. **Run the test suite** - No regressions
3. **Test edge cases** - Similar scenarios work
4. **Review in context** - Makes sense with surrounding code

## Debugging Checklist

- [ ] Can I reproduce it reliably?
- [ ] Do I understand the error message?
- [ ] Have I identified the root cause (not just symptom)?
- [ ] Is my fix minimal and focused?
- [ ] Have I added a test to prevent regression?
- [ ] Have I verified the fix works?
- [ ] Have I checked for similar bugs elsewhere?
