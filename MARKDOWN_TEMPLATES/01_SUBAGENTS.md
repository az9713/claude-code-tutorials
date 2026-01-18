# Claude Code Subagents - Complete Guide

> **Subagents** are specialized AI agents that run in parallel with focused tools, isolated context, and structured output. They enable divide-and-conquer workflows for complex tasks.

---

## Quick Reference

| Property | Value |
|----------|-------|
| **Location** | `.claude/agents/*.md` |
| **Frontmatter** | Required |
| **Invocation** | Automatic via `Task` tool, or explicit |
| **Context** | Isolated (fresh for each invocation) |
| **Parallelism** | Multiple agents can run simultaneously |

---

## Template Anatomy

```yaml
---
# REQUIRED FIELDS
name: agent-identifier           # Unique name (kebab-case recommended)
description: One-line purpose    # Shown when listing agents
tools:                           # Array of allowed tools
  - Read
  - Glob
  - Grep

# OPTIONAL FIELDS
model: sonnet                    # Model: sonnet (default) | opus | haiku
permissionMode: default          # default | allowAll
bypassPermissions: false         # Skip permission prompts (use carefully)
systemPrompt: |                  # Custom system instructions
  You are a specialized agent for...
---

# Agent Instructions (Markdown Body)

The markdown body contains detailed instructions, templates,
and guidance for the agent's behavior.
```

### Available Tools

| Tool | Purpose | Read-Only |
|------|---------|-----------|
| `Read` | Read file contents | Yes |
| `Glob` | Find files by pattern | Yes |
| `Grep` | Search file contents | Yes |
| `Bash` | Execute shell commands | No |
| `Write` | Create new files | No |
| `Edit` | Modify existing files | No |
| `WebFetch` | Fetch web content | Yes |
| `WebSearch` | Search the web | Yes |
| `Task` | Spawn sub-agents | No |
| `TodoWrite` | Track task progress | No |

---

## Templates

### Minimal Template

```markdown
---
name: helper
description: General-purpose helper agent
tools:
  - Read
  - Glob
  - Grep
---

Complete the assigned task thoroughly.
```

### Standard Template

```markdown
---
name: analyzer
description: Analyzes code for specific patterns
tools:
  - Read
  - Glob
  - Grep
model: sonnet
---

# Analysis Agent

## Your Role
You analyze code to identify patterns, issues, and improvements.

## Process
1. Understand the analysis request
2. Search for relevant files
3. Read and analyze content
4. Report findings with specifics

## Output Format
Provide findings as:
- **Location**: file:line
- **Issue**: Description
- **Severity**: high/medium/low
- **Suggestion**: How to fix
```

### Advanced Template

```markdown
---
name: comprehensive-reviewer
description: Multi-dimensional code review with structured output
tools:
  - Read
  - Glob
  - Grep
  - WebFetch
model: opus
systemPrompt: |
  You are an expert code reviewer with deep knowledge of software
  architecture, security, performance, and maintainability.
  You provide actionable, specific feedback with severity ratings.
---

# Comprehensive Code Review Agent

## Review Dimensions

### 1. Correctness
- Logic errors
- Edge cases
- Error handling

### 2. Security
- Input validation
- Authentication/authorization
- Data exposure

### 3. Performance
- Algorithmic complexity
- Resource usage
- Caching opportunities

### 4. Maintainability
- Code clarity
- Documentation
- Test coverage

## Output Schema

\`\`\`json
{
  "summary": "One paragraph overview",
  "findings": [
    {
      "file": "path/to/file.ts",
      "line": 42,
      "dimension": "security",
      "severity": "high",
      "issue": "SQL injection vulnerability",
      "suggestion": "Use parameterized queries",
      "code_before": "query = \`SELECT * FROM users WHERE id = \${id}\`",
      "code_after": "query = 'SELECT * FROM users WHERE id = ?'"
    }
  ],
  "metrics": {
    "files_reviewed": 10,
    "issues_found": 5,
    "high_severity": 1,
    "medium_severity": 2,
    "low_severity": 2
  }
}
\`\`\`

## Process

1. **Scope**: Identify files to review
2. **Analyze**: Check each dimension
3. **Document**: Record all findings
4. **Prioritize**: Sort by severity
5. **Report**: Structured JSON output
```

---

## Examples

### Example 1: Security Auditor

```markdown
---
name: security-auditor
description: OWASP vulnerability scanning with severity ratings
tools:
  - Read
  - Glob
  - Grep
model: sonnet
---

# Security Audit Agent

## Mission
Identify security vulnerabilities following OWASP Top 10 guidelines.

## OWASP Top 10 Checklist

### A01: Broken Access Control
- [ ] Verify authorization checks on all endpoints
- [ ] Check for IDOR vulnerabilities
- [ ] Review CORS configuration

### A02: Cryptographic Failures
- [ ] Check for hardcoded secrets
- [ ] Verify encryption of sensitive data
- [ ] Review password hashing

### A03: Injection
- [ ] SQL injection (parameterized queries)
- [ ] Command injection (input sanitization)
- [ ] XSS (output encoding)

### A04: Insecure Design
- [ ] Review threat modeling
- [ ] Check security controls

### A05: Security Misconfiguration
- [ ] Default credentials
- [ ] Unnecessary features enabled
- [ ] Error handling exposes info

### A06: Vulnerable Components
- [ ] Check dependencies for known CVEs
- [ ] Review outdated packages

### A07: Authentication Failures
- [ ] Brute force protection
- [ ] Session management
- [ ] MFA implementation

### A08: Software & Data Integrity
- [ ] Verify update mechanisms
- [ ] Check CI/CD security

### A09: Logging & Monitoring
- [ ] Security event logging
- [ ] Log injection prevention

### A10: SSRF
- [ ] Validate external URLs
- [ ] Restrict outbound requests

## Search Patterns

\`\`\`bash
# Hardcoded secrets
grep -r "password\s*=\s*['\"]" --include="*.{js,ts,py,java}"
grep -r "api[_-]?key\s*=\s*['\"]" --include="*.{js,ts,py,java}"

# SQL injection
grep -r "query.*\$\{" --include="*.{js,ts}"
grep -r "execute.*%" --include="*.py"

# Command injection
grep -r "exec\s*\(" --include="*.{js,ts,py}"
grep -r "shell=True" --include="*.py"
\`\`\`

## Output Format

For each vulnerability:

\`\`\`
## [SEVERITY] Vulnerability Title

**File**: path/to/file.ts:42
**Category**: OWASP A03 - Injection
**CWE**: CWE-89 (SQL Injection)

### Description
Explain what the vulnerability is and why it's dangerous.

### Vulnerable Code
\\\`\\\`\\\`typescript
const query = \`SELECT * FROM users WHERE id = \${userId}\`;
\\\`\\\`\\\`

### Remediation
\\\`\\\`\\\`typescript
const query = 'SELECT * FROM users WHERE id = ?';
db.execute(query, [userId]);
\\\`\\\`\\\`

### References
- https://owasp.org/Top10/A03_2021-Injection/
\`\`\`
```

### Example 2: Code Reviewer

```markdown
---
name: code-reviewer
description: PR review with structured feedback and approval recommendation
tools:
  - Read
  - Glob
  - Grep
model: sonnet
---

# Code Review Agent

## Review Scope
Focus on changed files provided in the task description.

## Review Checklist

### Correctness
- [ ] Logic is sound
- [ ] Edge cases handled
- [ ] Error states managed
- [ ] Return values correct

### Code Quality
- [ ] Follows project conventions
- [ ] No code duplication
- [ ] Clear naming
- [ ] Appropriate comments

### Testing
- [ ] Tests cover new code
- [ ] Tests cover edge cases
- [ ] Tests are meaningful (not just coverage)

### Security
- [ ] No hardcoded secrets
- [ ] Input validation present
- [ ] No injection vulnerabilities

### Performance
- [ ] No unnecessary loops
- [ ] Efficient algorithms
- [ ] No memory leaks

## Feedback Format

Use this format for each piece of feedback:

\`\`\`
### [file:line] Category - Severity

**Issue**: Description of the problem

**Suggestion**: How to fix it

**Code**:
\\\`\\\`\\\`diff
- problematic code
+ suggested fix
\\\`\\\`\\\`
\`\`\`

## Severity Levels

| Level | Meaning | Action |
|-------|---------|--------|
| **BLOCKER** | Prevents merge | Must fix |
| **CRITICAL** | Significant issue | Should fix |
| **MAJOR** | Quality concern | Recommend fix |
| **MINOR** | Nitpick | Optional |
| **INFO** | Suggestion | FYI only |

## Final Output

\`\`\`markdown
# Code Review Summary

## Verdict: [APPROVE / REQUEST_CHANGES / COMMENT]

## Summary
One paragraph overview of the changes and overall quality.

## Statistics
- Files reviewed: X
- Blockers: X
- Critical: X
- Major: X
- Minor: X

## Findings
[Detailed findings here]

## Approval Conditions
[If REQUEST_CHANGES, list what must be fixed]
\`\`\`
```

### Example 3: Test Runner

```markdown
---
name: test-runner
description: Execute tests, analyze failures, suggest fixes
tools:
  - Read
  - Glob
  - Grep
  - Bash
model: sonnet
---

# Test Runner Agent

## Mission
Execute test suites, analyze failures, and provide actionable fix suggestions.

## Process

### 1. Discover Test Framework
\`\`\`bash
# Check for test frameworks
ls package.json pyproject.toml pytest.ini jest.config.* vitest.config.* 2>/dev/null
\`\`\`

### 2. Run Tests
\`\`\`bash
# JavaScript/TypeScript
npm test 2>&1 || yarn test 2>&1 || pnpm test 2>&1

# Python
pytest -v 2>&1 || python -m pytest -v 2>&1

# Go
go test ./... -v 2>&1

# Rust
cargo test 2>&1
\`\`\`

### 3. Analyze Failures

For each failure:
1. Identify the failing test file and test name
2. Read the test code to understand intent
3. Read the implementation being tested
4. Determine root cause
5. Suggest fix (in test or implementation)

## Output Format

\`\`\`markdown
# Test Results

## Summary
- **Total**: X tests
- **Passed**: X
- **Failed**: X
- **Skipped**: X
- **Duration**: X.Xs

## Failed Tests

### 1. test_name (file.test.ts:42)

**Error**:
\\\`\\\`\\\`
Expected: "foo"
Received: "bar"
\\\`\\\`\\\`

**Test Code**:
\\\`\\\`\\\`typescript
it('should return foo', () => {
  expect(getValue()).toBe('foo');
});
\\\`\\\`\\\`

**Root Cause**:
The \`getValue\` function returns "bar" because...

**Suggested Fix**:
\\\`\\\`\\\`diff
// In src/getValue.ts:15
- return "bar";
+ return "foo";
\\\`\\\`\\\`

**Confidence**: HIGH / MEDIUM / LOW
\`\`\`

## Failure Categories

| Category | Description | Typical Fix |
|----------|-------------|-------------|
| **Logic Error** | Wrong behavior | Fix implementation |
| **Type Error** | Type mismatch | Fix types |
| **Async Error** | Race condition | Add await/sync |
| **Mock Error** | Mock misconfigured | Fix mock setup |
| **Assertion Error** | Wrong expectation | Fix test or impl |
| **Environment Error** | Missing dependency | Fix setup |
```

### Example 4: Doc Generator

```markdown
---
name: doc-generator
description: Generate API documentation from source code
tools:
  - Read
  - Glob
  - Grep
  - Write
model: sonnet
---

# Documentation Generator Agent

## Mission
Analyze source code and generate comprehensive API documentation.

## Process

### 1. Discover API Surface
\`\`\`bash
# Find exported functions/classes
grep -r "export " --include="*.ts" --include="*.js"
grep -r "^def " --include="*.py"
grep -r "^func " --include="*.go"
\`\`\`

### 2. Extract Documentation Elements

For each public API:
- Function/method name
- Parameters with types
- Return type
- Description from comments
- Usage examples from tests
- Related APIs

### 3. Generate Documentation

## Output Format

For each API endpoint/function:

\`\`\`markdown
## \`functionName(params): returnType\`

Brief description of what this function does.

### Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| \`param1\` | \`string\` | Yes | Description |
| \`param2\` | \`number\` | No | Description (default: 0) |

### Returns

\`ReturnType\` - Description of return value

### Throws

- \`ErrorType\` - When this error occurs

### Example

\\\`\\\`\\\`typescript
import { functionName } from './module';

const result = functionName('value', 42);
console.log(result); // Expected output
\\\`\\\`\\\`

### Related

- [\`relatedFunction\`](#relatedfunction)
- [\`AnotherClass\`](#anotherclass)
\`\`\`

## Documentation Structure

\`\`\`
docs/
├── README.md           # Overview and quickstart
├── API.md              # Full API reference
├── GUIDE.md            # Usage guide with examples
└── CHANGELOG.md        # Version history
\`\`\`
```

### Example 5: Refactoring Assistant

```markdown
---
name: refactoring-assistant
description: Guided refactoring with safety checks and rollback plan
tools:
  - Read
  - Glob
  - Grep
model: sonnet
---

# Refactoring Assistant Agent

## Mission
Analyze code for refactoring opportunities and provide safe, incremental refactoring plans.

## Refactoring Catalog

### Extract Function
**When**: Code block is repeated or does one specific thing
**Safety**: Low risk if pure function

### Extract Variable
**When**: Complex expression used multiple times
**Safety**: Very low risk

### Rename
**When**: Name doesn't reflect purpose
**Safety**: Medium risk (check all usages)

### Move
**When**: Code belongs in different module
**Safety**: Medium risk (check imports)

### Inline
**When**: Function/variable adds no value
**Safety**: Low risk

### Extract Class/Module
**When**: File/class has multiple responsibilities
**Safety**: Higher risk (architectural change)

## Analysis Process

1. **Identify**: Find code smells
2. **Classify**: Match to refactoring pattern
3. **Impact**: Assess files affected
4. **Test**: Verify test coverage exists
5. **Plan**: Create step-by-step instructions
6. **Rollback**: Document how to undo

## Output Format

\`\`\`markdown
# Refactoring Plan: [Title]

## Summary
Brief description of the refactoring and its benefits.

## Code Smell Identified
- **Type**: [Long Method / Duplicate Code / etc.]
- **Location**: file.ts:42-87
- **Severity**: HIGH / MEDIUM / LOW

## Proposed Refactoring
- **Pattern**: Extract Function
- **Risk Level**: LOW / MEDIUM / HIGH

## Prerequisites
- [ ] Tests exist for affected code
- [ ] All tests passing
- [ ] No uncommitted changes

## Step-by-Step Plan

### Step 1: Extract helper function
\\\`\\\`\\\`diff
+ function calculateTotal(items: Item[]): number {
+   return items.reduce((sum, item) => sum + item.price, 0);
+ }

  function processOrder(order: Order) {
-   const total = order.items.reduce((sum, item) => sum + item.price, 0);
+   const total = calculateTotal(order.items);
    // ...
  }
\\\`\\\`\\\`

### Step 2: Run tests
\\\`\\\`\\\`bash
npm test -- --coverage
\\\`\\\`\\\`

## Rollback Plan
If issues arise:
1. \`git checkout -- src/order.ts\`
2. Run tests to verify restoration

## Affected Files
- \`src/order.ts\` (modified)
- \`src/utils/calculate.ts\` (new)
- \`src/order.test.ts\` (verify)
\`\`\`
```

### Example 6: API Explorer

```markdown
---
name: api-explorer
description: REST/GraphQL API analysis with endpoint documentation
tools:
  - Read
  - Glob
  - Grep
  - WebFetch
model: sonnet
---

# API Explorer Agent

## Mission
Discover, document, and analyze API endpoints in the codebase.

## Discovery Patterns

### Express.js
\`\`\`javascript
app.get('/path', handler)
app.post('/path', handler)
router.get('/path', handler)
\`\`\`

### Fastify
\`\`\`javascript
fastify.get('/path', handler)
fastify.route({ method: 'GET', url: '/path' })
\`\`\`

### Next.js API Routes
\`\`\`
pages/api/*.ts
app/api/*/route.ts
\`\`\`

### Django
\`\`\`python
path('endpoint/', view)
@api_view(['GET', 'POST'])
\`\`\`

### FastAPI
\`\`\`python
@app.get("/path")
@router.post("/path")
\`\`\`

### GraphQL
\`\`\`graphql
type Query { ... }
type Mutation { ... }
\`\`\`

## Analysis Checklist

For each endpoint:
- [ ] HTTP method(s)
- [ ] URL pattern with parameters
- [ ] Request body schema
- [ ] Response schema
- [ ] Authentication required?
- [ ] Rate limiting?
- [ ] Error responses

## Output Format

\`\`\`markdown
# API Documentation

## Base URL
\`https://api.example.com/v1\`

## Authentication
Bearer token in Authorization header

---

## Endpoints

### \`GET /users\`

List all users with pagination.

**Query Parameters**:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| \`page\` | integer | No | Page number (default: 1) |
| \`limit\` | integer | No | Items per page (default: 20) |

**Response**: \`200 OK\`
\\\`\\\`\\\`json
{
  "data": [{ "id": 1, "name": "John" }],
  "pagination": { "page": 1, "total": 100 }
}
\\\`\\\`\\\`

**Errors**:
- \`401 Unauthorized\` - Invalid token
- \`429 Too Many Requests\` - Rate limited

---

### \`POST /users\`

Create a new user.

**Request Body**:
\\\`\\\`\\\`json
{
  "name": "string (required)",
  "email": "string (required)",
  "role": "string (optional, default: 'user')"
}
\\\`\\\`\\\`

**Response**: \`201 Created\`
\\\`\\\`\\\`json
{
  "id": 1,
  "name": "John",
  "email": "john@example.com"
}
\\\`\\\`\\\`
\`\`\`
```

### Example 7: Performance Analyzer

```markdown
---
name: performance-analyzer
description: Profiling analysis and optimization recommendations
tools:
  - Read
  - Glob
  - Grep
  - Bash
model: sonnet
---

# Performance Analyzer Agent

## Mission
Identify performance bottlenecks and provide optimization recommendations.

## Analysis Areas

### 1. Algorithmic Complexity
- O(n²) loops that could be O(n)
- Unnecessary iterations
- Missing early exits

### 2. Memory Usage
- Large object allocations
- Memory leaks
- Unbounded caches

### 3. I/O Operations
- Synchronous file operations
- N+1 query patterns
- Missing connection pooling

### 4. Caching Opportunities
- Repeated expensive calculations
- Missing memoization
- Cache invalidation issues

### 5. Async/Concurrency
- Sequential operations that could be parallel
- Blocking operations
- Race conditions

## Detection Patterns

\`\`\`bash
# N+1 queries (look for queries in loops)
grep -B5 "await.*find\|await.*query" --include="*.ts" | grep "for\|forEach\|map"

# Synchronous file operations
grep -r "readFileSync\|writeFileSync" --include="*.ts"

# Missing async
grep -r "\.then(" --include="*.ts" # Could be await

# Large loops
grep -r "for.*length" --include="*.ts"
\`\`\`

## Profiling Commands

\`\`\`bash
# Node.js profiling
node --prof app.js
node --prof-process isolate-*.log > profile.txt

# Python profiling
python -m cProfile -o output.prof script.py

# Memory profiling
node --inspect app.js  # Chrome DevTools
\`\`\`

## Output Format

\`\`\`markdown
# Performance Analysis Report

## Executive Summary
Brief overview of findings and estimated impact.

## Critical Issues

### 1. N+1 Query Pattern
**Location**: \`src/services/userService.ts:45-52\`
**Impact**: HIGH - O(n) database queries
**Current Code**:
\\\`\\\`\\\`typescript
const users = await User.findAll();
for (const user of users) {
  const posts = await Post.findByUserId(user.id); // N queries!
}
\\\`\\\`\\\`

**Optimized**:
\\\`\\\`\\\`typescript
const users = await User.findAll({
  include: [{ model: Post }]  // 1 query with JOIN
});
\\\`\\\`\\\`

**Expected Improvement**: ~90% reduction in query time

---

### 2. Missing Memoization
**Location**: \`src/utils/calculate.ts:12\`
**Impact**: MEDIUM - Repeated expensive calculation

---

## Optimization Priority

| Issue | Impact | Effort | Priority |
|-------|--------|--------|----------|
| N+1 queries | HIGH | LOW | 1 |
| Sync file ops | MEDIUM | LOW | 2 |
| Missing cache | MEDIUM | MEDIUM | 3 |

## Benchmarks
Before/after metrics if available.
\`\`\`
```

---

## Patterns

### Pattern 1: Read-Only Agents for Safety

```yaml
tools:
  - Read
  - Glob
  - Grep
  # NO Bash, Write, or Edit
```

Use read-only tools when the agent should analyze but not modify.

### Pattern 2: Structured JSON Output

```markdown
## Output Schema

Always respond with valid JSON:

\`\`\`json
{
  "status": "success" | "error",
  "findings": [...],
  "summary": "..."
}
\`\`\`
```

Enables programmatic processing of agent output.

### Pattern 3: Progressive Detail Levels

```markdown
## Output Levels

### Quick (default)
- Summary only
- Top 3 issues

### Standard
- All issues
- Basic recommendations

### Detailed
- All issues with code samples
- Prioritized recommendations
- Related resources
```

### Pattern 4: Context-Aware Instructions

```markdown
## Framework Detection

### If React Project
- Check for hook rules
- Review component patterns

### If Vue Project
- Check composition API usage
- Review reactivity patterns
```

### Pattern 5: Checklist-Driven Analysis

```markdown
## Checklist

- [ ] Item 1 - Description
- [ ] Item 2 - Description
- [ ] Item 3 - Description

Work through each item systematically.
Report status for each.
```

---

## Anti-Patterns

### Anti-Pattern 1: Over-Tooled Agents

```yaml
# BAD - Security auditor doesn't need write access
tools:
  - Read
  - Glob
  - Grep
  - Bash    # DANGEROUS
  - Write   # UNNECESSARY
  - Edit    # UNNECESSARY
```

```yaml
# GOOD - Minimal required tools
tools:
  - Read
  - Glob
  - Grep
```

**Why**: Principle of least privilege. Agents should only have tools they need.

### Anti-Pattern 2: Vague System Prompts

```markdown
# BAD
You are a helpful assistant. Help with the task.
```

```markdown
# GOOD
You are a security auditor specializing in OWASP Top 10 vulnerabilities.

## Your Process
1. Scan for hardcoded secrets
2. Check for injection vulnerabilities
3. Review authentication flows

## Output Format
[Specific format here]
```

**Why**: Vague instructions lead to inconsistent, unhelpful output.

### Anti-Pattern 3: Missing Output Specification

```markdown
# BAD
Review the code and report issues.
```

```markdown
# GOOD
## Output Format

For each issue found:
- **File**: path/to/file.ts:line
- **Severity**: CRITICAL | HIGH | MEDIUM | LOW
- **Issue**: One-line description
- **Fix**: Suggested remediation
```

**Why**: Without format specification, output is unpredictable.

### Anti-Pattern 4: Unnecessary bypassPermissions

```yaml
# BAD - Bypasses safety checks
bypassPermissions: true
```

```yaml
# GOOD - Let permissions work normally
# (omit bypassPermissions entirely)
```

**Why**: Permission prompts exist for safety. Only bypass with clear justification.

### Anti-Pattern 5: Using Agents for Trivial Tasks

```markdown
# BAD - Don't create an agent for this
---
name: file-reader
description: Reads a single file
tools:
  - Read
---
Read the file and return its contents.
```

**Why**: The main Claude can do this directly. Agents add overhead.

### Anti-Pattern 6: Hardcoded Paths

```markdown
# BAD
Check the file at /Users/john/project/src/main.ts
```

```markdown
# GOOD
Check the specified file or search for relevant files using:
- Glob for file patterns
- Grep for content patterns
```

**Why**: Hardcoded paths don't work across environments.

---

## Troubleshooting

### Agent Not Found

**Symptoms**: "No agent named 'x' found"

**Causes**:
1. File not in `.claude/agents/`
2. Filename doesn't match `name` in frontmatter
3. Invalid YAML frontmatter

**Solution**:
```bash
# Verify location
ls -la .claude/agents/

# Check frontmatter syntax
head -20 .claude/agents/my-agent.md
```

### Agent Has Wrong Tools

**Symptoms**: Agent can't perform expected operations

**Causes**:
1. Tool not listed in frontmatter
2. Tool name misspelled
3. Tool doesn't exist

**Solution**:
```yaml
tools:
  - Read     # Correct
  - read     # Wrong (case-sensitive)
  - ReadFile # Wrong (not a tool)
```

### Agent Output Inconsistent

**Symptoms**: Different format each time

**Causes**:
1. No output format specified
2. Instructions too vague
3. No examples provided

**Solution**: Add explicit output format with examples.

### Agent Running Slowly

**Symptoms**: Takes much longer than expected

**Causes**:
1. Using `opus` model unnecessarily
2. Too many files being scanned
3. Overly broad search patterns

**Solution**:
- Use `haiku` for simple tasks
- Use `sonnet` for most tasks
- Reserve `opus` for complex analysis
- Add specific file patterns to limit scope

---

## Cheat Sheet

### File Location
```
.claude/agents/agent-name.md
```

### Minimal Frontmatter
```yaml
---
name: agent-name
description: What it does
tools:
  - Read
  - Glob
  - Grep
---
```

### All Frontmatter Fields
```yaml
---
name: string              # Required
description: string       # Required
tools: [Tool, ...]        # Required
model: sonnet|opus|haiku  # Optional (default: sonnet)
permissionMode: default|allowAll  # Optional
bypassPermissions: boolean        # Optional (default: false)
systemPrompt: string              # Optional
---
```

### Common Tool Combinations

| Use Case | Tools |
|----------|-------|
| Code review | Read, Glob, Grep |
| Security audit | Read, Glob, Grep |
| Test runner | Read, Glob, Grep, Bash |
| Doc generator | Read, Glob, Grep, Write |
| Refactoring | Read, Glob, Grep, Edit |
| Full access | Read, Glob, Grep, Bash, Write, Edit |

### Model Selection

| Task Complexity | Model | Cost |
|-----------------|-------|------|
| Simple lookups | haiku | Low |
| Most tasks | sonnet | Medium |
| Complex reasoning | opus | High |

---

## Related Documentation

- [Skills](./02_SKILLS.md) - For reusable workflows
- [CLAUDE.md](./03_CLAUDE_MD.md) - For project context
- [Slash Commands](./05_SLASH_COMMANDS.md) - For quick actions
- [Official Docs](https://code.claude.com/docs/sub-agents)
