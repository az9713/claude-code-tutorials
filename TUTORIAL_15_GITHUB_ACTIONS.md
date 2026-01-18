# Tutorial 15: GitHub Actions Integration with Claude Code

## A Comprehensive Guide to CI/CD Automation with AI-Powered Code Assistance

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Core Concepts](#core-concepts)
4. [Setting Up claude-code-action](#setting-up-claude-code-action)
5. [@claude Mentions in PRs](#claude-mentions-in-prs)
6. [Automated Code Review](#automated-code-review)
7. [Issue-to-Code Automation](#issue-to-code-automation)
8. [Real-World Examples](#real-world-examples)
9. [Configuration Options Reference](#configuration-options-reference)
10. [Troubleshooting Guide](#troubleshooting-guide)
11. [Best Practices](#best-practices)
12. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)
13. [Summary](#summary)

---

## Overview

### What is GitHub Actions Integration with Claude Code?

GitHub Actions integration with Claude Code brings AI-powered code assistance directly into your
CI/CD pipeline. This powerful combination allows teams to automate code reviews, generate
implementations from issues, fix bugs on demand, and maintain code quality at scale.

### Why Integrate Claude Code with GitHub Actions?

Modern software development demands speed without sacrificing quality. By integrating Claude Code
into your GitHub workflows, you can:

- **Automate Code Reviews**: Get instant, thorough code reviews on every pull request
- **Accelerate Bug Fixes**: Mention @claude in a PR comment to get fix suggestions immediately
- **Generate Implementations**: Convert issues into working code automatically
- **Maintain Consistency**: Ensure coding standards are followed across your entire codebase
- **Scale Your Team**: Let Claude handle routine reviews while humans focus on complex decisions
- **Reduce Review Bottlenecks**: No more waiting days for code reviews

### How It Works

The integration works through the `claude-code-action` GitHub Action, which:

1. **Monitors Events**: Listens for PR opens, comments, issue labels, and other GitHub events
2. **Processes Context**: Gathers relevant code, comments, and repository context
3. **Invokes Claude**: Sends the context to Claude Code for analysis or implementation
4. **Takes Action**: Posts comments, creates commits, opens PRs, or performs other GitHub actions

```
+------------------+     +-------------------+     +------------------+
|  GitHub Event    | --> | claude-code-action| --> | Claude Code API  |
|  (PR, Comment,   |     | (Process Context) |     | (Analysis/Code)  |
|   Issue Label)   |     |                   |     |                  |
+------------------+     +-------------------+     +------------------+
                                  |
                                  v
                         +-------------------+
                         |  GitHub Action    |
                         |  (Comment, Commit,|
                         |   Create PR)      |
                         +-------------------+
```

### Benefits for Different Team Roles

**For Developers:**
- Get instant feedback on code changes
- Request specific help with @claude mentions
- Automate tedious code review responses

**For Team Leads:**
- Ensure consistent review quality
- Reduce review bottlenecks
- Track code quality metrics over time

**For DevOps Engineers:**
- Easy integration with existing workflows
- Configurable triggers and actions
- Secure API key management

---

## Prerequisites

Before setting up GitHub Actions integration with Claude Code, ensure you have the following:

### 1. GitHub Repository

You need a GitHub repository where you have administrative access to:
- Configure repository settings
- Add secrets
- Modify workflow files
- Manage Actions permissions

```bash
# Verify you have a GitHub repository
# Navigate to your repo and check Settings > Actions

# If creating a new repo:
gh repo create my-project --public --clone
cd my-project
```

### 2. Anthropic API Key

An Anthropic API key is required to authenticate with the Claude API.

**Getting Your API Key:**

1. Visit [console.anthropic.com](https://console.anthropic.com)
2. Sign in or create an account
3. Navigate to API Keys section
4. Click "Create Key"
5. Copy the key (it won't be shown again)

```
API Key Format: YOUR_API_KEY-xxxxxxxxxxxx...
```

**Important Security Notes:**
- Never commit API keys to your repository
- Always use GitHub Secrets for storage
- Rotate keys periodically
- Use organization-level secrets for multiple repos

### 3. GitHub Actions Knowledge

Basic understanding of GitHub Actions concepts:

**Workflow Files:**
```yaml
# .github/workflows/example.yml
name: Example Workflow
on: [push, pull_request]  # Triggers

jobs:
  example-job:            # Job definition
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4  # Action usage
      - run: echo "Hello World"    # Shell command
```

**Key Concepts to Understand:**
- **Workflows**: Automated processes defined in YAML files
- **Events**: Triggers that start workflows (push, PR, comment, etc.)
- **Jobs**: Groups of steps that run on the same runner
- **Steps**: Individual tasks within a job
- **Actions**: Reusable units of code (like `claude-code-action`)
- **Secrets**: Encrypted environment variables
- **Runners**: Machines that execute workflows

### 4. Repository Structure

Your repository should have a basic structure:

```
my-project/
├── .github/
│   └── workflows/       # Workflow files go here
│       └── claude.yml
├── src/                 # Source code
├── tests/               # Test files
├── README.md
└── package.json (or equivalent)
```

### 5. Required Permissions

Ensure your repository has the correct permissions:

1. **Actions Permissions:**
   - Settings > Actions > General
   - "Allow all actions and reusable workflows" (or specifically allow `anthropics/claude-code-action`)

2. **Workflow Permissions:**
   - Settings > Actions > General > Workflow permissions
   - "Read and write permissions"
   - "Allow GitHub Actions to create and approve pull requests"

### Verification Checklist

Before proceeding, verify:

- [ ] You have a GitHub repository with admin access
- [ ] You have an Anthropic API key
- [ ] You understand basic GitHub Actions concepts
- [ ] Your repository has `.github/workflows/` directory
- [ ] Actions are enabled for your repository
- [ ] Workflow permissions are configured correctly

---

## Core Concepts

### Understanding claude-code-action

The `claude-code-action` is an official GitHub Action that integrates Claude Code into your
workflows. It acts as a bridge between GitHub events and Claude's AI capabilities.

**Key Features:**
- Event-driven activation
- Context-aware processing
- Multiple operation modes
- Configurable behavior
- Secure API handling

### Architecture Overview

```
GitHub Repository
       |
       v
+-------------------------------+
|     GitHub Events             |
|  - pull_request               |
|  - issue_comment              |
|  - issues (labeled)           |
|  - push                       |
|  - schedule                   |
+-------------------------------+
       |
       v
+-------------------------------+
|   claude-code-action          |
|                               |
|  +-------------------------+  |
|  | Event Parser            |  |
|  +-------------------------+  |
|              |                |
|              v                |
|  +-------------------------+  |
|  | Context Gatherer        |  |
|  | - Code diff             |  |
|  | - File contents         |  |
|  | - Comments              |  |
|  | - Issue body            |  |
|  +-------------------------+  |
|              |                |
|              v                |
|  +-------------------------+  |
|  | Claude API Client       |  |
|  +-------------------------+  |
|              |                |
|              v                |
|  +-------------------------+  |
|  | Response Handler        |  |
|  | - Post comments         |  |
|  | - Create commits        |  |
|  | - Open PRs              |  |
|  +-------------------------+  |
+-------------------------------+
```

### @claude Mentions

The @claude mention system allows developers to interact with Claude directly in PR comments
and issue discussions.

**How It Works:**

1. Developer writes a comment containing `@claude`
2. GitHub triggers `issue_comment` event
3. Workflow detects the trigger phrase
4. Claude processes the request
5. Claude responds with a comment

**Example Interaction:**

```
Developer Comment:
"@claude can you review this function for potential memory leaks?"

Claude Response:
"I've analyzed the function and found the following potential issues:

1. **Line 45**: The `buffer` allocation is never freed in the error path
2. **Line 67**: The loop may cause unbounded memory growth
3. **Line 89**: Consider using a memory pool for frequent allocations

Here's a suggested fix:
[code block with fix]"
```

### PR Automation Workflows

Claude can automate various PR-related tasks:

**Automatic Review:**
- Triggered when PR is opened or updated
- Analyzes code changes
- Posts review comments
- Suggests improvements

**Fix on Request:**
- Triggered by @claude mention
- Implements requested fixes
- Creates commits
- Updates the PR

**Test Generation:**
- Analyzes new code
- Generates relevant tests
- Commits test files
- Runs test suite

### Issue-to-Code Pipeline

Convert issues directly into implementations:

```
Issue Created
     |
     v
Label Added: "claude-implement"
     |
     v
Claude Reads Issue
     |
     v
Claude Generates Implementation
     |
     v
Claude Creates Branch
     |
     v
Claude Opens PR
     |
     v
Human Reviews and Merges
```

### Operation Modes

The `claude-code-action` supports multiple modes:

| Mode | Description | Use Case |
|------|-------------|----------|
| `review` | Analyzes code and provides feedback | Code review automation |
| `implement` | Generates code from requirements | Issue-to-PR automation |
| `fix` | Fixes issues in existing code | Bug fix automation |
| `test` | Generates tests for code | Test coverage improvement |
| `docs` | Generates documentation | Documentation automation |

---

## Setting Up claude-code-action

### Step 1: Installing the Action

The `claude-code-action` is installed by referencing it in your workflow file. No separate
installation is required.

**Basic Reference:**
```yaml
- uses: anthropics/claude-code-action@v1
```

**Version Pinning:**
```yaml
# Pin to specific version for stability
- uses: anthropics/claude-code-action@v1.2.3

# Or use major version tag for automatic minor/patch updates
- uses: anthropics/claude-code-action@v1
```

### Step 2: API Key Configuration

Store your Anthropic API key as a GitHub Secret.

**Using GitHub UI:**

1. Navigate to your repository
2. Go to Settings > Secrets and variables > Actions
3. Click "New repository secret"
4. Name: `ANTHROPIC_API_KEY`
5. Value: Your API key (YOUR_API_KEY_HERE)
6. Click "Add secret"

**Using GitHub CLI:**
```bash
# Set secret via CLI
gh secret set ANTHROPIC_API_KEY

# You'll be prompted to enter the value
# Or pipe it from a file
cat ~/anthropic-key.txt | gh secret set ANTHROPIC_API_KEY
```

**Organization-Level Secrets:**

For multiple repositories:

1. Go to Organization Settings
2. Secrets and variables > Actions
3. New organization secret
4. Select repository access policy

### Step 3: Basic Workflow File

Create your first Claude Code workflow:

**File: `.github/workflows/claude-review.yml`**

```yaml
# =============================================================================
# Basic Claude Code Review Workflow
# =============================================================================
# This workflow provides automated code review on pull requests
# and responds to @claude mentions in PR comments.

name: Claude Code Review

# =============================================================================
# Triggers
# =============================================================================
on:
  # Trigger on pull request events
  pull_request:
    types:
      - opened        # When PR is first opened
      - synchronize   # When new commits are pushed to PR
      - reopened      # When PR is reopened

  # Trigger on comments in PRs and issues
  issue_comment:
    types:
      - created       # When a new comment is posted

# =============================================================================
# Jobs
# =============================================================================
jobs:
  claude-review:
    # Only run if:
    # - It's a PR event, OR
    # - It's a comment containing @claude on a PR
    if: |
      github.event_name == 'pull_request' ||
      (github.event_name == 'issue_comment' &&
       github.event.issue.pull_request &&
       contains(github.event.comment.body, '@claude'))

    runs-on: ubuntu-latest

    # Required permissions
    permissions:
      contents: read        # Read repository contents
      pull-requests: write  # Comment on PRs
      issues: write         # Comment on issues

    steps:
      # Step 1: Checkout the repository
      - name: Checkout Repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for better context

      # Step 2: Run Claude Code Action
      - name: Claude Code Review
        uses: anthropics/claude-code-action@v1
        with:
          # Authentication
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}

          # Configuration
          model: claude-sonnet-4-20250514
          trigger_phrase: "@claude"
          mode: review
          max_tokens: 4096
```

### Step 4: Permission Configuration

Proper permissions are crucial for the action to work correctly.

**Repository Settings:**

1. **Enable Actions:**
   - Settings > Actions > General
   - "Allow all actions and reusable workflows"

2. **Workflow Permissions:**
   - Settings > Actions > General > Workflow permissions
   - Select "Read and write permissions"
   - Check "Allow GitHub Actions to create and approve pull requests"

**Workflow-Level Permissions:**

```yaml
jobs:
  claude-review:
    permissions:
      # Minimum required permissions
      contents: read          # Read code
      pull-requests: write    # Comment on PRs

      # Additional permissions for advanced features
      issues: write           # Comment on issues
      checks: write           # Create check runs
      statuses: write         # Update commit status
```

**Permission Matrix:**

| Feature | contents | pull-requests | issues | checks |
|---------|----------|---------------|--------|--------|
| Review PR | read | write | - | - |
| Comment on Issue | read | - | write | - |
| Create Commits | write | write | - | - |
| Create PR | write | write | - | - |
| Update Status | read | - | - | write |

### Step 5: Verifying the Setup

After configuration, verify everything works:

**Test 1: Manual Workflow Trigger**
```yaml
# Add workflow_dispatch for manual testing
on:
  workflow_dispatch:  # Allows manual trigger
  pull_request:
    types: [opened]
```

Then trigger via GitHub UI: Actions > Select Workflow > Run workflow

**Test 2: Create Test PR**
```bash
# Create a test branch
git checkout -b test/claude-integration

# Make a simple change
echo "// Test change" >> src/index.js

# Commit and push
git add .
git commit -m "Test: Verify Claude integration"
git push origin test/claude-integration

# Create PR
gh pr create --title "Test Claude Integration" --body "Testing @claude review"
```

**Test 3: Check Workflow Logs**
- Go to Actions tab
- Click on the workflow run
- Review step outputs
- Verify Claude responded

### Complete Basic Setup Example

Here's a complete, production-ready basic setup:

```yaml
# =============================================================================
# .github/workflows/claude-code.yml
# Complete Claude Code Integration Setup
# =============================================================================

name: Claude Code Assistant

on:
  pull_request:
    types: [opened, synchronize, reopened]
  issue_comment:
    types: [created]

# Prevent concurrent runs on the same PR
concurrency:
  group: claude-${{ github.event.pull_request.number || github.event.issue.number }}
  cancel-in-progress: false

jobs:
  claude-assistant:
    # Conditional execution
    if: |
      github.event_name == 'pull_request' ||
      (github.event_name == 'issue_comment' &&
       github.event.issue.pull_request &&
       contains(github.event.comment.body, '@claude'))

    runs-on: ubuntu-latest
    timeout-minutes: 10

    permissions:
      contents: read
      pull-requests: write
      issues: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Run Claude Code
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          model: claude-sonnet-4-20250514
          trigger_phrase: "@claude"
        env:
          # Optional: Additional environment configuration
          CLAUDE_TIMEOUT: "60000"
```

---

## @claude Mentions in PRs

### How @claude Mentions Work

When you mention @claude in a PR comment, the GitHub Action:

1. Detects the `issue_comment` event
2. Verifies the comment contains the trigger phrase
3. Extracts the request from the comment
4. Gathers PR context (diff, files, previous comments)
5. Sends request to Claude
6. Posts Claude's response as a new comment

### Mentioning Claude in PR Comments

**Basic Mention:**
```
@claude please review this PR
```

**Specific Request:**
```
@claude can you check the error handling in the `processData` function?
```

**Multiple Requests:**
```
@claude please:
1. Review the SQL queries for potential injection vulnerabilities
2. Check if the error messages expose sensitive information
3. Suggest improvements for the logging statements
```

### Code Review Requests

**General Review:**
```
@claude review this code for best practices and potential issues
```

**Focused Review:**
```
@claude review the authentication logic in src/auth/login.js
```

**Security Review:**
```
@claude perform a security review focusing on:
- Input validation
- SQL injection
- XSS vulnerabilities
- Authentication bypasses
```

**Performance Review:**
```
@claude analyze these changes for performance implications:
- Are there any N+1 query patterns?
- Could any operations be batched?
- Are there memory leak risks?
```

**Example Workflow for Review Requests:**

```yaml
name: Claude Review Request

on:
  issue_comment:
    types: [created]

jobs:
  review-request:
    if: |
      github.event.issue.pull_request &&
      contains(github.event.comment.body, '@claude') &&
      contains(github.event.comment.body, 'review')

    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          mode: review
          review_criteria: |
            - Code correctness
            - Error handling
            - Security best practices
            - Performance considerations
            - Code style consistency
```

### Fix Suggestions

**Request a Fix:**
```
@claude fix the null pointer exception in the handleSubmit function
```

**Request with Context:**
```
@claude the tests are failing because the mock isn't set up correctly.
Can you fix the test setup in tests/api.test.js?
```

**Request Specific Changes:**
```
@claude please fix:
1. The race condition in the async handler
2. The memory leak in the event listener
3. The incorrect date formatting
```

**Fix Workflow Configuration:**

```yaml
name: Claude Fix

on:
  issue_comment:
    types: [created]

jobs:
  claude-fix:
    if: |
      github.event.issue.pull_request &&
      contains(github.event.comment.body, '@claude') &&
      contains(github.event.comment.body, 'fix')

    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write

    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.ref }}
          fetch-depth: 0

      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          mode: fix
          auto_commit: true
          commit_message_prefix: "fix: "
```

### Implementation Help

**Request Implementation:**
```
@claude implement input validation for the email field
```

**Request with Specifications:**
```
@claude add pagination to the user list endpoint:
- Page size: 20 items
- Support cursor-based pagination
- Include total count in response
- Add appropriate error handling
```

**Request Following Patterns:**
```
@claude implement the deleteUser function following the same pattern as createUser
```

**Implementation Workflow:**

```yaml
name: Claude Implementation

on:
  issue_comment:
    types: [created]

jobs:
  implement:
    if: |
      github.event.issue.pull_request &&
      contains(github.event.comment.body, '@claude') &&
      (contains(github.event.comment.body, 'implement') ||
       contains(github.event.comment.body, 'add') ||
       contains(github.event.comment.body, 'create'))

    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write

    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.ref }}

      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          mode: implement
          auto_commit: true
```

### Advanced @claude Interactions

**Asking Questions:**
```
@claude why was this implementation approach chosen over using a queue?
```

**Requesting Explanations:**
```
@claude explain what this regex pattern does: ^(?=.*[A-Za-z])(?=.*\d)[A-Za-z\d]{8,}$
```

**Requesting Tests:**
```
@claude add unit tests for the validateEmail function covering:
- Valid email formats
- Invalid formats
- Edge cases (empty string, null, undefined)
- Maximum length handling
```

**Requesting Documentation:**
```
@claude add JSDoc comments to all exported functions in this file
```

---

## Automated Code Review

### Triggering Reviews on PR Open

Configure automatic reviews when PRs are opened:

```yaml
# =============================================================================
# Automated Code Review on PR Open
# =============================================================================

name: Automated Code Review

on:
  pull_request:
    types:
      - opened        # Review when PR is created
      - synchronize   # Re-review when commits are pushed
      - reopened      # Review when PR is reopened

jobs:
  auto-review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Automated Review
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          mode: review
          auto_review: true
```

### Review Criteria Configuration

Customize what Claude looks for during reviews:

```yaml
- name: Configured Review
  uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    github_token: ${{ secrets.GITHUB_TOKEN }}
    mode: review
    review_criteria: |
      ## Code Quality
      - Code follows project conventions
      - Functions are appropriately sized
      - Variables have meaningful names
      - No dead code or commented-out code

      ## Security
      - Input validation is present
      - No hardcoded secrets
      - SQL queries use parameterized statements
      - Authentication checks are in place

      ## Performance
      - No obvious performance issues
      - Queries are optimized
      - Caching is used where appropriate

      ## Testing
      - New code has corresponding tests
      - Edge cases are covered
      - Tests are meaningful, not just for coverage

      ## Documentation
      - Public APIs are documented
      - Complex logic has comments
      - README is updated if needed
```

**Using External Criteria File:**

```yaml
- name: Load Review Criteria
  id: criteria
  run: |
    echo "criteria<<EOF" >> $GITHUB_OUTPUT
    cat .github/review-criteria.md >> $GITHUB_OUTPUT
    echo "EOF" >> $GITHUB_OUTPUT

- name: Review with Criteria
  uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    github_token: ${{ secrets.GITHUB_TOKEN }}
    review_criteria: ${{ steps.criteria.outputs.criteria }}
```

### Comment Formatting

Configure how Claude formats review comments:

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    github_token: ${{ secrets.GITHUB_TOKEN }}
    comment_format: |
      ## Claude Code Review

      ### Summary
      {summary}

      ### Issues Found
      {issues}

      ### Suggestions
      {suggestions}

      ### Code Quality Score
      {score}/10

      ---
      *Automated review by Claude Code*
```

**Review Comment Types:**

| Type | Description | When Used |
|------|-------------|-----------|
| `inline` | Comments on specific lines | Specific code issues |
| `summary` | Overall PR summary | General feedback |
| `suggestion` | Code suggestions with diffs | Improvement proposals |
| `issue` | Problem that should be fixed | Bugs, security issues |

### Approval Workflows

Configure automatic approval based on review results:

```yaml
name: Auto-Approve Clean PRs

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review-and-approve:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write

    steps:
      - uses: actions/checkout@v4

      - name: Claude Review
        id: review
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          mode: review
          output_review_result: true

      - name: Auto-Approve if Clean
        if: steps.review.outputs.issues_count == 0
        uses: actions/github-script@v7
        with:
          script: |
            await github.rest.pulls.createReview({
              owner: context.repo.owner,
              repo: context.repo.repo,
              pull_number: context.issue.number,
              event: 'APPROVE',
              body: 'Automatically approved - no issues found by Claude Code review.'
            });
```

**Conditional Approval Flow:**

```yaml
- name: Determine Approval
  id: determine
  run: |
    ISSUES="${{ steps.review.outputs.issues_count }}"
    SEVERITY="${{ steps.review.outputs.max_severity }}"

    if [ "$ISSUES" = "0" ]; then
      echo "action=approve" >> $GITHUB_OUTPUT
    elif [ "$SEVERITY" = "low" ]; then
      echo "action=comment" >> $GITHUB_OUTPUT
    else
      echo "action=request_changes" >> $GITHUB_OUTPUT
    fi

- name: Take Action
  uses: actions/github-script@v7
  with:
    script: |
      const action = '${{ steps.determine.outputs.action }}';
      const events = {
        'approve': 'APPROVE',
        'comment': 'COMMENT',
        'request_changes': 'REQUEST_CHANGES'
      };

      await github.rest.pulls.createReview({
        owner: context.repo.owner,
        repo: context.repo.repo,
        pull_number: context.issue.number,
        event: events[action],
        body: '${{ steps.review.outputs.summary }}'
      });
```

### Full Automated Review Workflow

```yaml
# =============================================================================
# Complete Automated Review Workflow
# =============================================================================

name: Complete Code Review

on:
  pull_request:
    types: [opened, synchronize, reopened]
    branches:
      - main
      - develop

concurrency:
  group: review-${{ github.event.pull_request.number }}
  cancel-in-progress: true

jobs:
  automated-review:
    runs-on: ubuntu-latest
    timeout-minutes: 15

    permissions:
      contents: read
      pull-requests: write
      checks: write

    outputs:
      issues_count: ${{ steps.review.outputs.issues_count }}
      review_summary: ${{ steps.review.outputs.summary }}

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Get Changed Files
        id: changed
        uses: tj-actions/changed-files@v41
        with:
          files: |
            **/*.js
            **/*.ts
            **/*.py
            **/*.go

      - name: Skip if No Code Changes
        if: steps.changed.outputs.any_changed != 'true'
        run: echo "No code changes detected, skipping review"

      - name: Claude Code Review
        if: steps.changed.outputs.any_changed == 'true'
        id: review
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          mode: review
          files: ${{ steps.changed.outputs.all_changed_files }}
          review_criteria: |
            Focus on:
            1. Logic errors and bugs
            2. Security vulnerabilities
            3. Performance issues
            4. Code style consistency

      - name: Create Check Run
        uses: actions/github-script@v7
        with:
          script: |
            const issues = parseInt('${{ steps.review.outputs.issues_count }}' || '0');
            const conclusion = issues === 0 ? 'success' : issues < 3 ? 'neutral' : 'failure';

            await github.rest.checks.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              name: 'Claude Code Review',
              head_sha: context.payload.pull_request.head.sha,
              status: 'completed',
              conclusion: conclusion,
              output: {
                title: `Found ${issues} issue(s)`,
                summary: '${{ steps.review.outputs.summary }}'
              }
            });
```

---

## Issue-to-Code Automation

### Claude Reading Issues

Configure Claude to read and understand issues:

```yaml
name: Issue Handler

on:
  issues:
    types: [labeled]

jobs:
  read-issue:
    if: github.event.label.name == 'claude-implement'
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Process Issue
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          mode: implement
          issue_number: ${{ github.event.issue.number }}
```

### Generating Implementation

How Claude generates code from issues:

```yaml
# =============================================================================
# Issue-to-Implementation Workflow
# =============================================================================

name: Implement from Issue

on:
  issues:
    types: [labeled]

jobs:
  implement:
    if: github.event.label.name == 'claude-implement'
    runs-on: ubuntu-latest

    permissions:
      contents: write
      pull-requests: write
      issues: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Generate Implementation
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          mode: implement
          implementation_config: |
            # Read issue title and body for requirements
            # Analyze existing codebase for patterns
            # Generate implementation following project conventions
            # Include tests for new functionality
            # Update documentation if needed
```

### Creating PRs Automatically

Full workflow for automatic PR creation:

```yaml
# =============================================================================
# Complete Issue-to-PR Automation
# =============================================================================

name: Issue to PR

on:
  issues:
    types: [labeled]

jobs:
  create-implementation-pr:
    if: github.event.label.name == 'claude-implement'
    runs-on: ubuntu-latest
    timeout-minutes: 30

    permissions:
      contents: write
      pull-requests: write
      issues: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Create Branch
        id: branch
        run: |
          BRANCH="claude/issue-${{ github.event.issue.number }}"
          git checkout -b "$BRANCH"
          echo "branch=$BRANCH" >> $GITHUB_OUTPUT

      - name: Implement Feature
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          mode: implement
          issue_number: ${{ github.event.issue.number }}
          auto_commit: true

      - name: Push Branch
        run: |
          git push origin ${{ steps.branch.outputs.branch }}

      - name: Create Pull Request
        uses: actions/github-script@v7
        with:
          script: |
            const issue = context.payload.issue;

            const { data: pr } = await github.rest.pulls.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: `Implement: ${issue.title}`,
              body: `## Implementation

            This PR implements the feature requested in #${issue.number}.

            ### Changes Made
            - [Claude will list changes here]

            ### Testing
            - [ ] Tests added
            - [ ] Manual testing completed

            ### Related Issue
            Closes #${issue.number}

            ---
            *Generated by Claude Code*`,
              head: '${{ steps.branch.outputs.branch }}',
              base: 'main'
            });

            // Link PR to issue
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: issue.number,
              body: `Implementation PR created: #${pr.number}`
            });
```

### Review and Merge Flow

Complete implementation review workflow:

```yaml
# =============================================================================
# Implementation Review and Merge Flow
# =============================================================================

name: Implementation Flow

on:
  issues:
    types: [labeled]
  pull_request:
    types: [opened]
  pull_request_review:
    types: [submitted]

jobs:
  # Stage 1: Create implementation
  implement:
    if: |
      github.event_name == 'issues' &&
      github.event.label.name == 'claude-implement'
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
      issues: write

    steps:
      - uses: actions/checkout@v4

      - name: Implement and Create PR
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          mode: implement
          create_pr: true

  # Stage 2: Self-review the implementation
  self-review:
    if: |
      github.event_name == 'pull_request' &&
      startsWith(github.event.pull_request.head.ref, 'claude/')
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write

    steps:
      - uses: actions/checkout@v4

      - name: Self Review
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          mode: review
          self_review: true

  # Stage 3: Handle human review feedback
  handle-feedback:
    if: |
      github.event_name == 'pull_request_review' &&
      startsWith(github.event.pull_request.head.ref, 'claude/') &&
      github.event.review.state == 'changes_requested'
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write

    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.ref }}

      - name: Address Feedback
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          mode: fix
          review_comments: true
          auto_commit: true
```

### Issue Templates for Claude

Create issue templates that Claude can understand well:

**File: `.github/ISSUE_TEMPLATE/feature_request_claude.yml`**

```yaml
name: Feature Request (Claude Implementable)
description: Request a feature that Claude can implement
labels: ["enhancement"]

body:
  - type: markdown
    attributes:
      value: |
        ## Feature Request for Claude Implementation
        Please provide clear requirements for Claude to implement.

  - type: input
    id: title
    attributes:
      label: Feature Title
      description: A concise title for the feature
      placeholder: "Add user authentication endpoint"
    validations:
      required: true

  - type: textarea
    id: description
    attributes:
      label: Description
      description: Describe what you want to implement
      placeholder: |
        Create a new API endpoint for user authentication that:
        - Accepts email and password
        - Validates credentials against the database
        - Returns a JWT token on success
    validations:
      required: true

  - type: textarea
    id: acceptance-criteria
    attributes:
      label: Acceptance Criteria
      description: List specific criteria for the implementation
      placeholder: |
        - [ ] Endpoint accepts POST requests at /api/auth/login
        - [ ] Returns 200 with JWT on valid credentials
        - [ ] Returns 401 on invalid credentials
        - [ ] Includes rate limiting
    validations:
      required: true

  - type: textarea
    id: technical-notes
    attributes:
      label: Technical Notes
      description: Any technical constraints or preferences
      placeholder: |
        - Use the existing User model
        - Follow the pattern in userController.js
        - Add tests in tests/auth/

  - type: dropdown
    id: complexity
    attributes:
      label: Estimated Complexity
      options:
        - Small (single file change)
        - Medium (multiple files)
        - Large (new feature across codebase)
    validations:
      required: true
```

---

## Real-World Examples

### Example 1: Automated Code Review on Every PR

**Goal:** Review every PR automatically with comprehensive checks.

```yaml
# =============================================================================
# .github/workflows/auto-review.yml
# Automated Code Review on Every PR
# =============================================================================

name: Automated PR Review

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  comprehensive-review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
      checks: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Get PR Diff
        id: diff
        run: |
          git diff origin/${{ github.base_ref }}...HEAD > diff.txt
          echo "lines=$(wc -l < diff.txt)" >> $GITHUB_OUTPUT

      - name: Skip Large PRs
        if: steps.diff.outputs.lines > 2000
        run: |
          echo "PR is too large for automated review"
          echo "Please break into smaller PRs"
          exit 0

      - name: Claude Review
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          mode: review
          review_criteria: |
            ## Automated Review Checklist

            ### Code Quality
            - [ ] Code is readable and well-organized
            - [ ] Functions are small and focused
            - [ ] No code duplication

            ### Security
            - [ ] No sensitive data exposed
            - [ ] Input validation present
            - [ ] Authentication/authorization checked

            ### Performance
            - [ ] No obvious performance issues
            - [ ] Database queries optimized
            - [ ] No memory leaks

            ### Testing
            - [ ] Tests added for new functionality
            - [ ] Edge cases covered

            ### Documentation
            - [ ] Code is self-documenting
            - [ ] Complex logic explained
```

### Example 2: @claude Fix This Bug in PR Comment

**Goal:** Fix bugs when developers request via comment.

```yaml
# =============================================================================
# .github/workflows/claude-fix.yml
# Fix Bugs on @claude Request
# =============================================================================

name: Claude Bug Fix

on:
  issue_comment:
    types: [created]

jobs:
  fix-bug:
    if: |
      github.event.issue.pull_request &&
      contains(github.event.comment.body, '@claude') &&
      (contains(github.event.comment.body, 'fix') ||
       contains(github.event.comment.body, 'bug'))

    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write

    steps:
      - name: Get PR Branch
        id: pr
        uses: actions/github-script@v7
        with:
          script: |
            const { data: pr } = await github.rest.pulls.get({
              owner: context.repo.owner,
              repo: context.repo.repo,
              pull_number: context.issue.number
            });
            return pr.head.ref;
          result-encoding: string

      - name: Checkout PR Branch
        uses: actions/checkout@v4
        with:
          ref: ${{ steps.pr.outputs.result }}
          fetch-depth: 0

      - name: Configure Git
        run: |
          git config user.name "Claude Code"
          git config user.email "claude@anthropic.com"

      - name: Fix Bug
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          mode: fix
          trigger_phrase: "@claude"
          auto_commit: true
          commit_message_prefix: "fix: "

      - name: Push Fix
        run: |
          git push origin ${{ steps.pr.outputs.result }}

      - name: Comment Result
        uses: actions/github-script@v7
        with:
          script: |
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body: 'Fix has been committed and pushed. Please review the changes.'
            });
```

### Example 3: @claude Add Tests for This Function

**Goal:** Generate tests when requested.

```yaml
# =============================================================================
# .github/workflows/claude-tests.yml
# Generate Tests on Request
# =============================================================================

name: Claude Test Generator

on:
  issue_comment:
    types: [created]

jobs:
  generate-tests:
    if: |
      github.event.issue.pull_request &&
      contains(github.event.comment.body, '@claude') &&
      (contains(github.event.comment.body, 'test') ||
       contains(github.event.comment.body, 'tests'))

    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write

    steps:
      - name: Get PR Info
        id: pr
        uses: actions/github-script@v7
        with:
          script: |
            const { data: pr } = await github.rest.pulls.get({
              owner: context.repo.owner,
              repo: context.repo.repo,
              pull_number: context.issue.number
            });
            return { ref: pr.head.ref, sha: pr.head.sha };

      - name: Checkout
        uses: actions/checkout@v4
        with:
          ref: ${{ fromJSON(steps.pr.outputs.result).ref }}

      - name: Configure Git
        run: |
          git config user.name "Claude Code"
          git config user.email "claude@anthropic.com"

      - name: Generate Tests
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          mode: test
          test_config: |
            framework: jest
            style: describe/it
            coverage_target: 80%
            include:
              - unit tests
              - edge cases
              - error handling
          auto_commit: true
          commit_message_prefix: "test: "

      - name: Push Tests
        run: git push

      - name: Run Tests
        run: npm test

      - name: Report Results
        uses: actions/github-script@v7
        with:
          script: |
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body: 'Tests have been generated and committed. Test run completed.'
            });
```

### Example 4: Issue Labeled "claude-implement" Creates PR

**Goal:** Automatically implement features from labeled issues.

```yaml
# =============================================================================
# .github/workflows/implement-issue.yml
# Implement Issues Labeled for Claude
# =============================================================================

name: Implement Issue

on:
  issues:
    types: [labeled]

jobs:
  implement-feature:
    if: github.event.label.name == 'claude-implement'
    runs-on: ubuntu-latest
    timeout-minutes: 30

    permissions:
      contents: write
      pull-requests: write
      issues: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Configure Git
        run: |
          git config user.name "Claude Code"
          git config user.email "claude@anthropic.com"

      - name: Create Feature Branch
        id: branch
        run: |
          BRANCH="feature/issue-${{ github.event.issue.number }}"
          git checkout -b "$BRANCH"
          echo "branch=$BRANCH" >> $GITHUB_OUTPUT

      - name: Implement Feature
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          mode: implement
          issue_number: ${{ github.event.issue.number }}
          auto_commit: true
          include_tests: true

      - name: Push Branch
        run: git push -u origin ${{ steps.branch.outputs.branch }}

      - name: Create PR
        id: create-pr
        uses: actions/github-script@v7
        with:
          script: |
            const issue = context.payload.issue;

            const { data: pr } = await github.rest.pulls.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: `feat: ${issue.title}`,
              body: `## Summary
            Implements #${issue.number}

            ## Changes
            This PR was automatically generated by Claude Code to implement the feature
            requested in the linked issue.

            ## Checklist
            - [ ] Code review completed
            - [ ] Tests pass
            - [ ] Documentation updated

            Closes #${issue.number}`,
              head: '${{ steps.branch.outputs.branch }}',
              base: 'main',
              draft: true
            });

            core.setOutput('pr_number', pr.number);
            core.setOutput('pr_url', pr.html_url);

      - name: Update Issue
        uses: actions/github-script@v7
        with:
          script: |
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.payload.issue.number,
              body: `Implementation PR created: ${{ steps.create-pr.outputs.pr_url }}

            Please review the changes and mark ready for review when satisfied.`
            });

            await github.rest.issues.removeLabel({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.payload.issue.number,
              name: 'claude-implement'
            });

            await github.rest.issues.addLabels({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.payload.issue.number,
              labels: ['in-progress']
            });
```

### Example 5: Nightly Code Quality Checks

**Goal:** Run comprehensive code analysis every night.

```yaml
# =============================================================================
# .github/workflows/nightly-quality.yml
# Nightly Code Quality Analysis
# =============================================================================

name: Nightly Quality Check

on:
  schedule:
    - cron: '0 2 * * *'  # Run at 2 AM UTC daily
  workflow_dispatch:      # Allow manual trigger

jobs:
  quality-analysis:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      issues: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Code Quality Analysis
        id: analysis
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          mode: analyze
          analysis_config: |
            checks:
              - code_duplication
              - complexity_analysis
              - security_scan
              - dependency_review
              - dead_code_detection
              - performance_hotspots

      - name: Create Report Issue
        if: steps.analysis.outputs.issues_found == 'true'
        uses: actions/github-script@v7
        with:
          script: |
            const today = new Date().toISOString().split('T')[0];

            await github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: `Code Quality Report - ${today}`,
              body: `## Nightly Code Quality Report

            **Date:** ${today}
            **Status:** Issues Found

            ${{ steps.analysis.outputs.report }}

            ## Action Items
            ${{ steps.analysis.outputs.action_items }}

            ---
            *Generated by Claude Code nightly analysis*`,
              labels: ['code-quality', 'automated']
            });
```

### Example 6: Documentation Generation on Release

**Goal:** Auto-generate documentation when a release is created.

```yaml
# =============================================================================
# .github/workflows/release-docs.yml
# Generate Documentation on Release
# =============================================================================

name: Release Documentation

on:
  release:
    types: [published]

jobs:
  generate-docs:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event.release.tag_name }}

      - name: Generate Documentation
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          mode: docs
          docs_config: |
            generate:
              - API documentation
              - Changelog entry
              - Migration guide (if breaking changes)
              - Usage examples
            format: markdown
            output_dir: docs/

      - name: Commit Documentation
        run: |
          git config user.name "Claude Code"
          git config user.email "claude@anthropic.com"
          git add docs/
          git commit -m "docs: Generate documentation for ${{ github.event.release.tag_name }}"
          git push origin HEAD:main

      - name: Update Release Notes
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const changelog = fs.readFileSync('docs/CHANGELOG.md', 'utf8');

            await github.rest.repos.updateRelease({
              owner: context.repo.owner,
              repo: context.repo.repo,
              release_id: context.payload.release.id,
              body: context.payload.release.body + '\n\n---\n\n' + changelog
            });
```

### Example 7: Security Scanning with Claude

**Goal:** Perform AI-powered security analysis.

```yaml
# =============================================================================
# .github/workflows/security-scan.yml
# AI-Powered Security Scanning
# =============================================================================

name: Security Scan

on:
  pull_request:
    types: [opened, synchronize]
    paths:
      - '**/*.js'
      - '**/*.ts'
      - '**/*.py'
      - '**/*.go'
  schedule:
    - cron: '0 6 * * 1'  # Weekly on Monday at 6 AM UTC

jobs:
  security-scan:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
      security-events: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Security Analysis
        id: security
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          mode: security
          security_config: |
            checks:
              - sql_injection
              - xss_vulnerabilities
              - authentication_bypass
              - insecure_deserialization
              - hardcoded_secrets
              - path_traversal
              - command_injection
              - insecure_crypto
              - sensitive_data_exposure

            severity_threshold: medium
            fail_on_high: true

      - name: Upload SARIF
        if: steps.security.outputs.sarif_file
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: ${{ steps.security.outputs.sarif_file }}

      - name: Comment on PR
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            const findings = JSON.parse('${{ steps.security.outputs.findings }}');

            if (findings.length > 0) {
              const body = `## Security Scan Results

            Found **${findings.length}** potential security issue(s).

            | Severity | Issue | Location |
            |----------|-------|----------|
            ${findings.map(f => `| ${f.severity} | ${f.title} | ${f.location} |`).join('\n')}

            Please review and address these findings before merging.`;

              await github.rest.issues.createComment({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: context.issue.number,
                body: body
              });
            }

      - name: Fail on High Severity
        if: steps.security.outputs.high_severity_count > 0
        run: |
          echo "High severity security issues found!"
          exit 1
```

---

## Configuration Options Reference

### Complete Options Table

| Option | Type | Description | Default | Required |
|--------|------|-------------|---------|----------|
| `anthropic_api_key` | string | Anthropic API key for authentication | - | Yes |
| `github_token` | string | GitHub token for API access | `GITHUB_TOKEN` | Yes |
| `model` | string | Claude model to use | `claude-sonnet-4-20250514` | No |
| `trigger_phrase` | string | Comment trigger phrase | `@claude` | No |
| `mode` | string | Operation mode (review/implement/fix/test/docs) | `review` | No |
| `max_tokens` | integer | Maximum response tokens | `4096` | No |
| `temperature` | float | Response randomness (0-1) | `0.3` | No |
| `timeout` | integer | Request timeout in seconds | `300` | No |
| `auto_commit` | boolean | Automatically commit changes | `false` | No |
| `auto_review` | boolean | Auto-review on PR open | `false` | No |
| `create_pr` | boolean | Create PR after implementation | `false` | No |
| `include_tests` | boolean | Include tests in implementation | `true` | No |
| `review_criteria` | string | Custom review checklist | - | No |
| `files` | string | Specific files to analyze | - | No |
| `exclude_files` | string | Files to exclude from analysis | - | No |
| `issue_number` | integer | Issue number for implementation | - | No |
| `commit_message_prefix` | string | Prefix for auto-commits | - | No |
| `output_review_result` | boolean | Output review results as step outputs | `false` | No |

### Model Options

| Model | Description | Best For |
|-------|-------------|----------|
| `claude-sonnet-4-20250514` | Balanced performance and speed | General code review |
| `claude-opus-4-20250514` | Highest capability | Complex implementations |
| `claude-haiku-3-5-20241022` | Fastest responses | Simple tasks, high volume |

### Mode Descriptions

| Mode | Description | Creates Commits | Creates PRs |
|------|-------------|-----------------|-------------|
| `review` | Analyzes code and provides feedback | No | No |
| `implement` | Generates code from requirements | Yes (optional) | Yes (optional) |
| `fix` | Fixes issues in existing code | Yes (optional) | No |
| `test` | Generates tests for code | Yes (optional) | No |
| `docs` | Generates documentation | Yes (optional) | No |
| `security` | Security-focused analysis | No | No |
| `analyze` | General code analysis | No | No |

### Environment Variables

```yaml
env:
  CLAUDE_TIMEOUT: "60000"           # Timeout in milliseconds
  CLAUDE_MAX_RETRIES: "3"           # Max retry attempts
  CLAUDE_LOG_LEVEL: "info"          # Logging verbosity
  CLAUDE_CACHE_ENABLED: "true"      # Enable response caching
  CLAUDE_CONTEXT_WINDOW: "100000"   # Max context size
```

### Output Variables

| Output | Type | Description |
|--------|------|-------------|
| `review_summary` | string | Summary of the review |
| `issues_count` | integer | Number of issues found |
| `issues_found` | boolean | Whether issues were found |
| `high_severity_count` | integer | High severity issue count |
| `changes_made` | boolean | Whether changes were made |
| `commit_sha` | string | SHA of created commit |
| `pr_number` | integer | Number of created PR |
| `pr_url` | string | URL of created PR |
| `sarif_file` | string | Path to SARIF output |
| `findings` | JSON | Detailed findings array |

---

## Troubleshooting Guide

### "API key invalid" Errors

**Symptoms:**
- Error: "Invalid API key"
- Error: "Authentication failed"
- 401 Unauthorized responses

**Solutions:**

1. **Verify Secret Name:**
   ```yaml
   # Ensure the secret name matches exactly
   anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
   ```

2. **Check Secret Value:**
   ```bash
   # API key should start with YOUR_API_KEY-
   # Re-add the secret if unsure
   gh secret set ANTHROPIC_API_KEY
   ```

3. **Verify Organization Access:**
   - If using org-level secrets, ensure repository has access
   - Settings > Secrets > Repository access

4. **Check Key Status:**
   - Visit console.anthropic.com
   - Verify key is active and not expired
   - Check usage limits

### Permission Denied Issues

**Symptoms:**
- Error: "Resource not accessible by integration"
- Error: "Permission denied"
- Cannot comment on PR

**Solutions:**

1. **Add Workflow Permissions:**
   ```yaml
   permissions:
     contents: write
     pull-requests: write
     issues: write
   ```

2. **Update Repository Settings:**
   - Settings > Actions > General
   - Workflow permissions: "Read and write permissions"
   - Check "Allow GitHub Actions to create and approve pull requests"

3. **Use Explicit Token:**
   ```yaml
   - uses: actions/checkout@v4
     with:
       token: ${{ secrets.PAT_TOKEN }}  # Personal Access Token
   ```

4. **Check Branch Protection:**
   - Ensure workflow has permission to push to target branch
   - May need to allow Actions to bypass branch protection

### Rate Limiting

**Symptoms:**
- Error: "Rate limit exceeded"
- 429 Too Many Requests
- Workflow suddenly failing after working

**Solutions:**

1. **Add Retry Logic:**
   ```yaml
   - uses: anthropics/claude-code-action@v1
     with:
       max_retries: 3
       retry_delay: 5000
   ```

2. **Reduce Concurrent Runs:**
   ```yaml
   concurrency:
     group: claude-${{ github.ref }}
     cancel-in-progress: true
   ```

3. **Use Caching:**
   ```yaml
   - uses: anthropics/claude-code-action@v1
     with:
       cache_responses: true
   ```

4. **Optimize Token Usage:**
   - Review only changed files
   - Use smaller context windows
   - Batch similar requests

### Action Timeout

**Symptoms:**
- Workflow cancelled due to timeout
- Incomplete responses
- Action hangs indefinitely

**Solutions:**

1. **Increase Timeout:**
   ```yaml
   jobs:
     review:
       timeout-minutes: 30  # Increase from default 6 hours max
       steps:
         - uses: anthropics/claude-code-action@v1
           with:
             timeout: 600  # 10 minutes
   ```

2. **Break Into Smaller Tasks:**
   ```yaml
   # Instead of analyzing entire repo
   - uses: anthropics/claude-code-action@v1
     with:
       files: ${{ steps.changed.outputs.files }}  # Only changed files
   ```

3. **Use Streaming:**
   ```yaml
   - uses: anthropics/claude-code-action@v1
     with:
       streaming: true
   ```

### Context Too Large

**Symptoms:**
- Error: "Context length exceeded"
- Error: "Input too long"
- Truncated responses

**Solutions:**

1. **Limit Files:**
   ```yaml
   - uses: anthropics/claude-code-action@v1
     with:
       max_files: 20
       max_file_size: 50000  # Characters
   ```

2. **Exclude Large Files:**
   ```yaml
   - uses: anthropics/claude-code-action@v1
     with:
       exclude_files: |
         **/*.min.js
         **/dist/**
         **/node_modules/**
         **/*.lock
   ```

3. **Use Focused Analysis:**
   ```yaml
   - uses: anthropics/claude-code-action@v1
     with:
       mode: review
       focus: security  # Only security-relevant context
   ```

4. **Chunk Large PRs:**
   ```yaml
   - name: Check PR Size
     id: size
     run: |
       FILES=$(git diff --name-only origin/main | wc -l)
       echo "files=$FILES" >> $GITHUB_OUTPUT

   - name: Skip Large PRs
     if: steps.size.outputs.files > 50
     run: |
       echo "PR too large for automated review"
       echo "Please review manually or split into smaller PRs"
   ```

### Common Error Messages

| Error Message | Cause | Solution |
|---------------|-------|----------|
| "Invalid API key" | Wrong or expired key | Re-add secret |
| "Resource not accessible" | Missing permissions | Add permissions block |
| "Rate limit exceeded" | Too many requests | Add retry/concurrency |
| "Context length exceeded" | Too much code | Limit files/context |
| "Timeout exceeded" | Long running task | Increase timeout |
| "No trigger phrase found" | Comment missing @claude | Check trigger_phrase setting |
| "Branch not found" | Checkout issue | Use correct ref |
| "Push rejected" | Branch protection | Check permissions |

---

## Best Practices

### Cost Optimization for CI

**1. Smart Triggering:**
```yaml
on:
  pull_request:
    types: [opened]  # Not synchronize - review once, not every commit
    paths:
      - 'src/**'     # Only on source changes
      - '!**/*.md'   # Exclude documentation
```

**2. Model Selection:**
```yaml
# Use Haiku for simple tasks
- uses: anthropics/claude-code-action@v1
  with:
    model: claude-haiku-3-5-20241022  # Cheaper for simple reviews
```

**3. Context Reduction:**
```yaml
- uses: anthropics/claude-code-action@v1
  with:
    files: ${{ steps.changed.outputs.files }}  # Only changed files
    max_tokens: 2048  # Limit response size
```

**4. Caching:**
```yaml
- uses: anthropics/claude-code-action@v1
  with:
    cache_responses: true
    cache_ttl: 3600  # 1 hour
```

**5. Skip Non-Essential Runs:**
```yaml
jobs:
  review:
    # Skip for documentation-only changes
    if: |
      !contains(github.event.head_commit.message, '[skip ci]') &&
      !contains(github.event.head_commit.message, '[docs only]')
```

### Rate Limit Management

**1. Concurrency Control:**
```yaml
concurrency:
  group: claude-${{ github.repository }}
  cancel-in-progress: false  # Don't cancel, queue instead
```

**2. Staggered Execution:**
```yaml
- name: Random Delay
  run: sleep $((RANDOM % 30))  # 0-30 second delay
```

**3. Request Batching:**
```yaml
# Instead of multiple workflows, batch in one
- uses: anthropics/claude-code-action@v1
  with:
    batch_operations: true
    operations: |
      - mode: review
      - mode: security
      - mode: docs
```

**4. Fallback Handling:**
```yaml
- uses: anthropics/claude-code-action@v1
  id: claude
  continue-on-error: true

- name: Fallback on Rate Limit
  if: failure() && contains(steps.claude.outputs.error, 'rate limit')
  run: |
    echo "Rate limited - scheduling retry"
    # Schedule retry workflow
```

### Security Considerations

**1. Secret Management:**
```yaml
# Use environment-level secrets for sensitive repos
environment:
  name: production
  url: https://github.com

# Reference secrets securely
anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

**2. Input Validation:**
```yaml
# Validate comment before processing
- name: Validate Input
  run: |
    COMMENT='${{ github.event.comment.body }}'
    # Check for injection attempts
    if echo "$COMMENT" | grep -qE '[\$\`]|{{'; then
      echo "Suspicious input detected"
      exit 1
    fi
```

**3. Output Sanitization:**
```yaml
# Don't expose sensitive info in logs
- uses: anthropics/claude-code-action@v1
  with:
    mask_secrets: true
    log_level: warn  # Reduce logging verbosity
```

**4. Least Privilege:**
```yaml
permissions:
  contents: read       # Only write when needed
  pull-requests: write
  # Don't request unnecessary permissions
```

**5. Audit Logging:**
```yaml
- name: Log Action
  run: |
    echo "Claude action triggered by: ${{ github.actor }}"
    echo "Event: ${{ github.event_name }}"
    echo "Ref: ${{ github.ref }}"
```

### Team Workflow Integration

**1. Clear Documentation:**
```markdown
# Team Claude Integration Guide

## How to Use @claude

1. In any PR comment, mention @claude with your request
2. Wait for Claude to respond (usually < 2 minutes)
3. Review Claude's suggestions and apply as needed

## Labels

- `claude-implement`: Claude will create implementation PR
- `claude-review`: Request additional Claude review
- `needs-human-review`: Claude flagged for human attention
```

**2. Review Escalation:**
```yaml
- uses: anthropics/claude-code-action@v1
  id: review

- name: Escalate if Needed
  if: steps.review.outputs.high_severity_count > 0
  uses: actions/github-script@v7
  with:
    script: |
      await github.rest.issues.addLabels({
        owner: context.repo.owner,
        repo: context.repo.repo,
        issue_number: context.issue.number,
        labels: ['needs-human-review', 'security-review']
      });

      await github.rest.issues.addAssignees({
        owner: context.repo.owner,
        repo: context.repo.repo,
        issue_number: context.issue.number,
        assignees: ['security-team-lead']
      });
```

**3. Team Notifications:**
```yaml
- name: Notify Team
  if: steps.review.outputs.issues_count > 5
  uses: slackapi/slack-github-action@v1
  with:
    channel-id: 'dev-alerts'
    slack-message: |
      PR #${{ github.event.pull_request.number }} has ${{ steps.review.outputs.issues_count }} issues flagged by Claude.
      Please review: ${{ github.event.pull_request.html_url }}
```

**4. Metrics Tracking:**
```yaml
- name: Track Metrics
  run: |
    curl -X POST "${{ secrets.METRICS_ENDPOINT }}" \
      -H "Content-Type: application/json" \
      -d '{
        "event": "claude_review",
        "pr": "${{ github.event.pull_request.number }}",
        "issues_found": "${{ steps.review.outputs.issues_count }}",
        "timestamp": "'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"
      }'
```

---

## Quick Reference Cheat Sheet

### Essential Workflow Templates

**Minimal Review Setup:**
```yaml
name: Claude Review
on: [pull_request]
jobs:
  review:
    runs-on: ubuntu-latest
    permissions: { contents: read, pull-requests: write }
    steps:
      - uses: actions/checkout@v4
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

**@claude Comment Handler:**
```yaml
name: Claude Comment
on: { issue_comment: { types: [created] } }
jobs:
  respond:
    if: contains(github.event.comment.body, '@claude')
    runs-on: ubuntu-latest
    permissions: { contents: write, pull-requests: write }
    steps:
      - uses: actions/checkout@v4
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          trigger_phrase: "@claude"
```

**Issue Implementation:**
```yaml
name: Implement Issue
on: { issues: { types: [labeled] } }
jobs:
  implement:
    if: github.event.label.name == 'claude-implement'
    runs-on: ubuntu-latest
    permissions: { contents: write, pull-requests: write, issues: write }
    steps:
      - uses: actions/checkout@v4
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          mode: implement
          create_pr: true
```

### Common Commands Reference

| Command | Description |
|---------|-------------|
| `@claude review` | Request code review |
| `@claude fix` | Request bug fix |
| `@claude add tests` | Generate tests |
| `@claude explain` | Explain code |
| `@claude improve` | Suggest improvements |
| `@claude document` | Add documentation |
| `@claude security check` | Security analysis |
| `@claude optimize` | Performance optimization |

### Permission Cheat Sheet

| Task | contents | pull-requests | issues | checks |
|------|----------|---------------|--------|--------|
| Review only | read | write | - | - |
| Fix code | write | write | - | - |
| Create PR | write | write | - | - |
| Comment issue | read | - | write | - |
| Create check | read | - | - | write |
| Full access | write | write | write | write |

### Trigger Events Reference

| Event | Description | Common Use |
|-------|-------------|------------|
| `pull_request: opened` | PR created | Auto-review |
| `pull_request: synchronize` | Commits pushed | Re-review |
| `issue_comment: created` | New comment | @claude handler |
| `issues: labeled` | Label added | Implementation trigger |
| `schedule: cron` | Scheduled time | Nightly checks |
| `workflow_dispatch` | Manual trigger | Testing |

### Mode Quick Reference

| Mode | Creates Changes | Use When |
|------|-----------------|----------|
| `review` | No | Reviewing PRs |
| `implement` | Yes | Creating from issues |
| `fix` | Yes | Fixing bugs |
| `test` | Yes | Generating tests |
| `docs` | Yes | Writing documentation |
| `security` | No | Security scanning |

### Troubleshooting Quick Fixes

| Problem | Quick Fix |
|---------|-----------|
| Auth failed | Re-add `ANTHROPIC_API_KEY` secret |
| Permission denied | Add `permissions:` block |
| Rate limited | Add `concurrency:` group |
| Timeout | Increase `timeout-minutes:` |
| Context too large | Add `exclude_files:` |

---

## Summary

### What You Have Learned

In this tutorial, you have learned how to:

1. **Set Up claude-code-action**
   - Install and configure the GitHub Action
   - Securely manage API keys with GitHub Secrets
   - Configure proper permissions for different use cases

2. **Use @claude Mentions**
   - Request code reviews via PR comments
   - Ask for bug fixes with natural language
   - Generate tests and documentation on demand

3. **Automate Code Review**
   - Trigger reviews automatically on PR open
   - Configure custom review criteria
   - Set up approval workflows based on review results

4. **Implement Issue-to-Code Automation**
   - Convert issues into working implementations
   - Automatically create PRs from labeled issues
   - Manage the review and merge flow

5. **Apply Best Practices**
   - Optimize costs with smart triggering
   - Manage rate limits effectively
   - Secure your workflows properly
   - Integrate with team processes

### Key Takeaways

- **Start Simple**: Begin with basic review automation and expand gradually
- **Security First**: Always use secrets, never commit API keys
- **Optimize Costs**: Use appropriate models and limit context size
- **Monitor Usage**: Track API usage and set up alerts
- **Human Oversight**: Claude assists, humans decide

### Next Steps

1. **Implement Basic Setup**: Start with the minimal review workflow
2. **Add @claude Support**: Enable interactive comment handling
3. **Configure Issue Automation**: Set up issue-to-PR flow
4. **Customize for Your Team**: Adapt review criteria and workflows
5. **Monitor and Iterate**: Track effectiveness and improve over time

### Additional Resources

- [claude-code-action Repository](https://github.com/anthropics/claude-code-action)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Anthropic API Documentation](https://docs.anthropic.com)
- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)

---

## Appendix: Complete Workflow Examples

### Production-Ready Complete Workflow

```yaml
# =============================================================================
# .github/workflows/claude-complete.yml
# Production-Ready Claude Code Integration
# =============================================================================

name: Claude Code Assistant

on:
  pull_request:
    types: [opened, synchronize, reopened]
  issue_comment:
    types: [created]
  issues:
    types: [labeled]
  schedule:
    - cron: '0 2 * * 1'  # Weekly Monday 2 AM

concurrency:
  group: claude-${{ github.event.pull_request.number || github.event.issue.number || github.ref }}
  cancel-in-progress: false

env:
  CLAUDE_MODEL: claude-sonnet-4-20250514

jobs:
  # Job 1: Automated PR Review
  auto-review:
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    timeout-minutes: 15
    permissions:
      contents: read
      pull-requests: write
      checks: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Get Changed Files
        id: changed
        uses: tj-actions/changed-files@v41

      - name: Claude Review
        if: steps.changed.outputs.any_changed == 'true'
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          model: ${{ env.CLAUDE_MODEL }}
          mode: review
          files: ${{ steps.changed.outputs.all_changed_files }}

  # Job 2: Handle @claude Comments
  comment-handler:
    if: |
      github.event_name == 'issue_comment' &&
      github.event.issue.pull_request &&
      contains(github.event.comment.body, '@claude')
    runs-on: ubuntu-latest
    timeout-minutes: 20
    permissions:
      contents: write
      pull-requests: write

    steps:
      - name: Get PR Branch
        id: pr
        uses: actions/github-script@v7
        with:
          script: |
            const { data: pr } = await github.rest.pulls.get({
              owner: context.repo.owner,
              repo: context.repo.repo,
              pull_number: context.issue.number
            });
            return pr.head.ref;
          result-encoding: string

      - name: Checkout
        uses: actions/checkout@v4
        with:
          ref: ${{ steps.pr.outputs.result }}
          fetch-depth: 0

      - name: Configure Git
        run: |
          git config user.name "Claude Code"
          git config user.email "claude@anthropic.com"

      - name: Process Comment
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          model: ${{ env.CLAUDE_MODEL }}
          trigger_phrase: "@claude"
          auto_commit: true

      - name: Push Changes
        run: |
          git push origin ${{ steps.pr.outputs.result }} || true

  # Job 3: Implement from Issues
  implement-issue:
    if: |
      github.event_name == 'issues' &&
      github.event.label.name == 'claude-implement'
    runs-on: ubuntu-latest
    timeout-minutes: 30
    permissions:
      contents: write
      pull-requests: write
      issues: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Configure Git
        run: |
          git config user.name "Claude Code"
          git config user.email "claude@anthropic.com"

      - name: Create Branch
        id: branch
        run: |
          BRANCH="claude/issue-${{ github.event.issue.number }}"
          git checkout -b "$BRANCH"
          echo "branch=$BRANCH" >> $GITHUB_OUTPUT

      - name: Implement
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          model: ${{ env.CLAUDE_MODEL }}
          mode: implement
          issue_number: ${{ github.event.issue.number }}
          auto_commit: true
          include_tests: true

      - name: Push and Create PR
        run: |
          git push -u origin ${{ steps.branch.outputs.branch }}

      - name: Create Pull Request
        uses: actions/github-script@v7
        with:
          script: |
            const issue = context.payload.issue;
            const { data: pr } = await github.rest.pulls.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: `feat: ${issue.title}`,
              body: `Implements #${issue.number}\n\nGenerated by Claude Code`,
              head: '${{ steps.branch.outputs.branch }}',
              base: 'main',
              draft: true
            });

            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: issue.number,
              body: `Implementation PR: #${pr.number}`
            });

  # Job 4: Weekly Code Analysis
  weekly-analysis:
    if: github.event_name == 'schedule'
    runs-on: ubuntu-latest
    timeout-minutes: 30
    permissions:
      contents: read
      issues: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Code Analysis
        id: analysis
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          model: ${{ env.CLAUDE_MODEL }}
          mode: analyze

      - name: Create Report
        if: steps.analysis.outputs.issues_found == 'true'
        uses: actions/github-script@v7
        with:
          script: |
            const date = new Date().toISOString().split('T')[0];
            await github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: `Weekly Code Analysis - ${date}`,
              body: '${{ steps.analysis.outputs.report }}',
              labels: ['code-quality', 'automated']
            });
```

---

*This tutorial provides a comprehensive guide to integrating Claude Code with GitHub Actions. For the latest updates and additional features, refer to the official documentation.*
