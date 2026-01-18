# Claude Code Slash Commands - Complete Guide

> **Slash Commands** are custom shortcuts invoked with `/command-name`. They provide quick access to common actions and simple workflows without the full structure of Skills.

---

## Quick Reference

| Property | Value |
|----------|-------|
| **Location** | `.claude/commands/*.md` |
| **Frontmatter** | Optional |
| **Invocation** | `/command-name` or `/command-name args` |
| **Arguments** | Available as `$ARGUMENTS` placeholder |
| **Complexity** | Best for simple, 1-3 step actions |

---

## Commands vs Skills

| Aspect | Slash Commands | Skills |
|--------|----------------|--------|
| **Location** | `.claude/commands/` | `.claude/skills/*/SKILL.md` |
| **Complexity** | Simple (1-3 steps) | Complex (multi-step workflows) |
| **Checklists** | Not typical | Common pattern |
| **Context** | Always inherits | Can fork |
| **Use Case** | Quick actions | Repeatable processes |

**Rule of Thumb**: If it needs a checklist or multiple phases, use a Skill. If it's a quick action, use a Command.

---

## Template Anatomy

```yaml
---
# OPTIONAL FIELDS
description: One-line description    # Shown in /help
allowed-tools:                       # Restrict available tools
  - Bash
  - Read
argument-hint: <file>                # Shown in /help after command name
---

# Command Instructions (Markdown Body)

Instructions for what the command should do.
Use $ARGUMENTS for user-provided input.
```

### Frontmatter Fields

| Field | Required | Purpose |
|-------|----------|---------|
| `description` | No* | Shows in `/help` (*but highly recommended) |
| `allowed-tools` | No | Restricts which tools can be used |
| `argument-hint` | No | Shows expected arguments in `/help` |

---

## Templates

### Minimal Template

```markdown
---
description: Brief description of what this does
---

Do the thing.
```

### Standard Template

```markdown
---
description: Run project tests with coverage
allowed-tools:
  - Bash
  - Read
argument-hint: <test-pattern>
---

# Run Tests

Run the test suite for this project.

If $ARGUMENTS is provided, run only matching tests.
Otherwise, run all tests.

Include coverage report.
```

### Advanced Template

```markdown
---
description: Create a new React component with tests and stories
allowed-tools:
  - Read
  - Write
  - Glob
argument-hint: <ComponentName>
---

# Create Component

Create a new React component named: $ARGUMENTS

## Requirements

1. **Component File**: `src/components/$ARGUMENTS/$ARGUMENTS.tsx`
   - Functional component with TypeScript
   - Props interface
   - Default export

2. **Test File**: `src/components/$ARGUMENTS/$ARGUMENTS.test.tsx`
   - Basic render test
   - Props test

3. **Story File**: `src/components/$ARGUMENTS/$ARGUMENTS.stories.tsx`
   - Default story
   - With props variations

4. **Index File**: `src/components/$ARGUMENTS/index.ts`
   - Re-export component

## Before Creating

Check if component already exists:
- Look in `src/components/` for existing component
- If exists, ask user before overwriting

## After Creating

List the created files and suggest next steps.
```

---

## Examples

### Example 1: Commit Command

```markdown
---
description: Create a conventional commit with auto-generated message
allowed-tools:
  - Bash
  - Read
argument-hint: [type] [message]
---

# Smart Commit

Create a git commit following Conventional Commits format.

## Process

1. **Check Status**
   Run `git status` to see what's staged

2. **Analyze Changes**
   If no changes staged, inform user and exit.
   Otherwise, analyze the staged changes.

3. **Generate Message**
   Based on changes, suggest a commit message:
   - `feat:` for new features
   - `fix:` for bug fixes
   - `docs:` for documentation
   - `refactor:` for refactoring
   - `test:` for tests
   - `chore:` for maintenance

4. **User Override**
   If $ARGUMENTS provided, use that as the commit type/message.
   Format: `/commit fix corrected login validation`

5. **Create Commit**
   Create the commit with the message.

## Output

Show the commit hash and message after successful commit.
```

### Example 2: Review PR Command

```markdown
---
description: Review a pull request with structured feedback
allowed-tools:
  - Bash
  - Read
  - Glob
  - Grep
argument-hint: <PR-number-or-branch>
---

# Review Pull Request

Review the changes in PR or branch: $ARGUMENTS

## Process

1. **Get Changes**
   If numeric, fetch PR changes with `gh pr diff $ARGUMENTS`
   If branch name, diff against main: `git diff main...$ARGUMENTS`

2. **Analyze Files**
   For each changed file:
   - Read the file
   - Identify issues

3. **Check Categories**
   - [ ] Code correctness
   - [ ] Security concerns
   - [ ] Performance issues
   - [ ] Test coverage
   - [ ] Documentation

4. **Generate Review**
   Provide structured feedback:
   - Summary
   - File-by-file comments
   - Overall recommendation (approve/request changes)

## Output Format

\`\`\`
## PR Review: $ARGUMENTS

### Summary
[One paragraph overview]

### Findings

#### file1.ts
- Line 42: [Issue description]
- Line 87: [Issue description]

#### file2.ts
- Line 15: [Issue description]

### Verdict: [APPROVE / REQUEST CHANGES]

[Reasoning]
\`\`\`
```

### Example 3: Deploy Staging Command

```markdown
---
description: Deploy current branch to staging environment
allowed-tools:
  - Bash
  - Read
argument-hint: [--skip-tests]
---

# Deploy to Staging

Deploy the current branch to staging environment.

## Pre-Flight Checks

1. **Branch Check**
   - Verify on a feature branch (not main/master)
   - Warn if uncommitted changes exist

2. **Test Check** (unless --skip-tests in $ARGUMENTS)
   - Run test suite
   - Abort if tests fail

3. **Build Check**
   - Run production build
   - Verify build succeeds

## Deployment

1. **Push Branch**
   \`\`\`bash
   git push origin HEAD
   \`\`\`

2. **Trigger Deploy**
   \`\`\`bash
   # Using GitHub Actions
   gh workflow run deploy-staging.yml -f branch=$(git branch --show-current)

   # Or direct deploy script
   ./scripts/deploy-staging.sh
   \`\`\`

3. **Wait for Completion**
   Monitor deployment status

## Post-Deploy

- Provide staging URL
- Suggest smoke test checklist
- Show rollback command if needed
```

### Example 4: Create Component Command

```markdown
---
description: Scaffold a new React component with TypeScript
allowed-tools:
  - Write
  - Glob
argument-hint: <ComponentName>
---

# Create React Component

Create a new React component: $ARGUMENTS

## Validation

- Component name must be PascalCase
- Must not already exist in `src/components/`

## Files to Create

### Component (`src/components/$ARGUMENTS/$ARGUMENTS.tsx`)

\`\`\`tsx
import { type FC } from 'react';

interface ${ARGUMENTS}Props {
  className?: string;
}

export const $ARGUMENTS: FC<${ARGUMENTS}Props> = ({ className }) => {
  return (
    <div className={className}>
      $ARGUMENTS
    </div>
  );
};
\`\`\`

### Test (`src/components/$ARGUMENTS/$ARGUMENTS.test.tsx`)

\`\`\`tsx
import { render, screen } from '@testing-library/react';
import { $ARGUMENTS } from './$ARGUMENTS';

describe('$ARGUMENTS', () => {
  it('renders without crashing', () => {
    render(<$ARGUMENTS />);
    expect(screen.getByText('$ARGUMENTS')).toBeInTheDocument();
  });
});
\`\`\`

### Index (`src/components/$ARGUMENTS/index.ts`)

\`\`\`ts
export { $ARGUMENTS } from './$ARGUMENTS';
export type { ${ARGUMENTS}Props } from './$ARGUMENTS';
\`\`\`

## After Creation

Show created files and suggest:
- Adding to component index
- Writing additional tests
- Adding Storybook story
```

### Example 5: Run Tests Command

```markdown
---
description: Run tests with smart defaults based on context
allowed-tools:
  - Bash
  - Read
  - Glob
argument-hint: [pattern] [--watch] [--coverage]
---

# Smart Test Runner

Run tests intelligently based on context.

## Detect Test Framework

Check for:
- `jest.config.*` → Jest
- `vitest.config.*` → Vitest
- `pytest.ini` or `pyproject.toml` → pytest
- `go.mod` → Go test
- `Cargo.toml` → Cargo test

## Determine Scope

1. **If $ARGUMENTS contains pattern**
   Run only matching tests

2. **If in a specific directory**
   Run tests for that directory

3. **If recent git changes**
   Suggest running tests for changed files

4. **Otherwise**
   Run all tests

## Flags

Parse $ARGUMENTS for:
- `--watch` or `-w`: Watch mode
- `--coverage` or `-c`: Coverage report
- `--verbose` or `-v`: Verbose output

## Execute

\`\`\`bash
# JavaScript/TypeScript
npm test -- $pattern $flags

# Python
pytest $pattern $flags

# Go
go test $pattern $flags

# Rust
cargo test $pattern $flags
\`\`\`

## Output

- Show pass/fail summary
- Highlight failures with file:line
- Show coverage summary if requested
```

### Example 6: Generate Docs Command

```markdown
---
description: Generate documentation for specified code
allowed-tools:
  - Read
  - Write
  - Glob
  - Grep
argument-hint: <file-or-directory>
---

# Generate Documentation

Generate documentation for: $ARGUMENTS

## Process

1. **Discover Files**
   If directory, find all source files.
   If file, document that file.

2. **Analyze Code**
   For each file, extract:
   - Exported functions/classes
   - Type definitions
   - Parameters and return types
   - Existing JSDoc/docstrings

3. **Generate Docs**
   Create markdown documentation with:
   - Function signatures
   - Parameter descriptions
   - Return value descriptions
   - Usage examples

## Output Format

### For Functions

\`\`\`markdown
## `functionName(params): returnType`

Description of what the function does.

### Parameters

| Name | Type | Description |
|------|------|-------------|
| `param1` | `string` | Description |

### Returns

`ReturnType` - Description

### Example

\\\`\\\`\\\`typescript
const result = functionName('value');
\\\`\\\`\\\`
\`\`\`

### For Classes

\`\`\`markdown
## `ClassName`

Description of the class.

### Constructor

\`\`\`typescript
new ClassName(options: Options)
\`\`\`

### Methods

#### `methodName()`
...
\`\`\`

## Output Location

Write to `docs/api/$ARGUMENTS.md` or update existing.
```

### Example 7: Debug Issue Command

```markdown
---
description: Investigate and debug a reported issue
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
argument-hint: <issue-description-or-number>
---

# Debug Issue

Investigate: $ARGUMENTS

## Investigation Process

### 1. Understand the Issue

If $ARGUMENTS is a GitHub issue number:
\`\`\`bash
gh issue view $ARGUMENTS
\`\`\`

Otherwise, work with the provided description.

### 2. Search Codebase

Look for relevant code:
- Search for keywords from the issue
- Find related files and functions
- Look for error messages

### 3. Trace the Flow

Starting from the reported behavior:
- Identify entry point
- Trace through code path
- Find where behavior diverges from expected

### 4. Check Recent Changes

\`\`\`bash
git log --oneline -20 --all -- <relevant-files>
\`\`\`

Look for recent changes that might have introduced the issue.

### 5. Reproduce

Suggest steps to reproduce:
- Specific conditions
- Test data needed
- Commands to run

## Output

\`\`\`markdown
## Issue Analysis: $ARGUMENTS

### Summary
[One paragraph understanding of the issue]

### Relevant Code
- `file1.ts:42` - [why relevant]
- `file2.ts:87` - [why relevant]

### Potential Causes
1. [Cause 1] - [evidence]
2. [Cause 2] - [evidence]

### Suggested Investigation
1. [Step 1]
2. [Step 2]

### Possible Fixes
- [Fix approach 1]
- [Fix approach 2]
\`\`\`
```

---

## Patterns

### Pattern 1: Default Behavior with Override

```markdown
If $ARGUMENTS provided, use it.
Otherwise, use sensible default.
```

```markdown
---
description: Run linter on files
argument-hint: [files]
---

Lint the specified files, or all files if none specified.

If $ARGUMENTS is provided:
  Lint only those files.
Otherwise:
  Lint all source files in the project.
```

### Pattern 2: Flag Parsing

```markdown
Parse $ARGUMENTS for common flags like --verbose, --dry-run.
```

```markdown
---
description: Clean build artifacts
argument-hint: [--dry-run]
---

Clean build artifacts from the project.

Parse $ARGUMENTS:
- `--dry-run`: Show what would be deleted without deleting

If dry-run, list files that would be removed.
Otherwise, remove them.
```

### Pattern 3: Context-Aware Commands

```markdown
Detect project type and adapt behavior.
```

```markdown
---
description: Start development server
---

Detect project type and start appropriate dev server:

- If `package.json` with `dev` script: `npm run dev`
- If `package.json` with `start` script: `npm start`
- If `pyproject.toml`: `poetry run python -m app`
- If `Makefile` with `dev` target: `make dev`
- If `docker-compose.yml`: `docker-compose up`
```

### Pattern 4: Safety Checks

```markdown
Verify preconditions before destructive actions.
```

```markdown
---
description: Reset database to clean state
allowed-tools:
  - Bash
---

Reset the database to a clean state.

## Safety Checks

1. Verify NOT in production environment
2. Confirm with user before proceeding
3. Create backup before reset

## Only proceed if all checks pass.
```

### Pattern 5: Output Templates

```markdown
Define consistent output format.
```

```markdown
## Output Format

\`\`\`
Status: [SUCCESS/FAILURE]
Duration: [time]
Details:
- [item 1]
- [item 2]
\`\`\`
```

---

## Anti-Patterns

### Anti-Pattern 1: No Argument Handling

```markdown
# BAD
---
description: Analyze code
---
Analyze the code.
```

```markdown
# GOOD
---
description: Analyze specified code
argument-hint: <file-or-pattern>
---
Analyze: $ARGUMENTS

If no argument provided, ask what to analyze.
```

**Why**: Users expect commands to accept arguments.

### Anti-Pattern 2: Missing Description

```markdown
# BAD - Won't show in /help
---
allowed-tools:
  - Bash
---
Deploy the app.
```

```markdown
# GOOD - Shows in /help
---
description: Deploy application to production
allowed-tools:
  - Bash
---
Deploy the app.
```

**Why**: Without `description`, command won't appear in `/help`.

### Anti-Pattern 3: Duplicating Built-in Commands

```markdown
# BAD - /help already exists
---
description: Show available commands
---
List all commands.
```

**Why**: Don't duplicate Claude Code's built-in commands.

### Anti-Pattern 4: Complex Multi-Step Logic

```markdown
# BAD - Too complex for a command
---
description: Full code review workflow
---

1. Check out PR branch
2. Run tests
3. Run linter
4. Check security
5. Review each file
6. Generate report
7. Post comments
8. Make recommendation
```

```markdown
# GOOD - Use a Skill instead
# File: .claude/skills/code-review/SKILL.md
---
name: code-review
description: Full code review workflow
---
[Detailed workflow here]
```

**Why**: Complex workflows belong in Skills, not Commands.

### Anti-Pattern 5: Hardcoded Paths

```markdown
# BAD
---
description: Run tests
---
Run tests in /Users/john/project/tests
```

```markdown
# GOOD
---
description: Run tests
argument-hint: [path]
---
Run tests in the specified path or detect test directory automatically.
```

**Why**: Hardcoded paths don't work across environments.

### Anti-Pattern 6: No Error Guidance

```markdown
# BAD
---
description: Build project
---
Build the project.
```

```markdown
# GOOD
---
description: Build project
---
Build the project.

If build fails:
- Show the error message clearly
- Suggest common fixes
- Point to relevant documentation
```

**Why**: Commands should help users recover from errors.

---

## Troubleshooting

### Command Not in /help

**Symptoms**: Command exists but doesn't appear in `/help`

**Causes**:
1. Missing `description` field
2. Invalid YAML frontmatter
3. File not in `.claude/commands/`

**Solution**:
```yaml
---
description: This field is required  # <- Add this
---
```

### $ARGUMENTS Not Working

**Symptoms**: Literal `$ARGUMENTS` in output

**Causes**:
1. User invoked without arguments
2. Typo in placeholder

**Solution**:
```markdown
If $ARGUMENTS is empty or not provided:
  Ask user what they want to analyze.
Otherwise:
  Proceed with: $ARGUMENTS
```

### Command Runs Wrong Tools

**Symptoms**: Command tries to use disallowed tools

**Causes**:
1. `allowed-tools` too restrictive
2. Instructions reference unavailable tools

**Solution**:
```yaml
---
allowed-tools:
  - Bash
  - Read
  - Glob
  - Grep  # Add missing tools
---
```

### Command Too Slow

**Symptoms**: Command takes long to execute

**Causes**:
1. Too many sequential operations
2. Reading unnecessary files
3. Running expensive operations

**Solution**:
- Limit scope with specific patterns
- Add early exit conditions
- Consider if this should be a background task

---

## Built-in Commands Reference

These commands are built into Claude Code. Don't duplicate them:

| Command | Purpose |
|---------|---------|
| `/help` | Show available commands |
| `/clear` | Clear conversation |
| `/compact` | Compact conversation history |
| `/config` | Edit configuration |
| `/cost` | Show token usage |
| `/doctor` | Check Claude Code health |
| `/init` | Initialize project |
| `/login` | Authenticate |
| `/logout` | Sign out |
| `/memory` | Manage memory |
| `/model` | Switch models |
| `/permissions` | Manage permissions |
| `/review` | Review recent changes |
| `/status` | Show status |
| `/vim` | Toggle vim mode |

---

## Cheat Sheet

### File Location
```
.claude/commands/command-name.md
```

### Minimal Frontmatter
```yaml
---
description: What this command does
---
```

### All Frontmatter Fields
```yaml
---
description: string      # Recommended - shows in /help
allowed-tools:           # Optional - restrict tools
  - Bash
  - Read
argument-hint: <arg>     # Optional - shows in /help
---
```

### Argument Placeholder
```markdown
$ARGUMENTS - Replaced with user input after /command
```

### Invocation
```
/command-name
/command-name argument1 argument2
```

### When to Use Commands vs Skills

| Use Commands For | Use Skills For |
|------------------|----------------|
| Quick actions | Multi-step workflows |
| Simple operations | Checklists |
| Single-purpose | Complex processes |
| 1-3 steps | 4+ steps |
| No phases | Multiple phases |

---

## Related Documentation

- [Skills](./02_SKILLS.md) - For complex workflows
- [Subagents](./01_SUBAGENTS.md) - For parallel specialized work
- [CLAUDE.md](./03_CLAUDE_MD.md) - For project context
- [Official Docs](https://code.claude.com/docs/slash-commands)
