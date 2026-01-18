# Claude Code Agent Workflows Tutorial

A comprehensive guide to using Claude Code agents throughout the software development lifecycle.

---

## Table of Contents

1. [Overview](#overview)
2. [Phase 1: Requirements & Specification](#phase-1-requirements--specification)
3. [Phase 2: Architecture & Design](#phase-2-architecture--design)
4. [Phase 3: Planning](#phase-3-planning)
5. [Phase 4: Implementation](#phase-4-implementation)
6. [Phase 5: Testing](#phase-5-testing)
7. [Phase 6: Code Review](#phase-6-code-review)
8. [Phase 7: Deployment & CI/CD](#phase-7-deployment--cicd)
9. [Advanced Patterns](#advanced-patterns)
10. [Quick Reference](#quick-reference)

---

## Overview

### What Are Agents?

Agents are specialized AI assistants that run in isolated contexts. Each agent has:
- **Separate context window** - doesn't pollute your main conversation
- **Specific tools** - restricted to what's needed for their task
- **Independent execution** - can run in background or parallel

### Native Built-in Agents

| Agent | Best For |
|-------|----------|
| `Explore` | Codebase exploration, understanding existing code |
| `Plan` | Architecture design, implementation planning |
| `general-purpose` | Complex multi-step research tasks |
| `Bash` | Running shell commands |
| `claude-code-guide` | Claude Code documentation questions |

### How to Invoke Agents

```bash
# Direct request (Claude auto-delegates)
"Explore the authentication system in this codebase"

# Explicit agent request
"Use the Plan agent to design a caching layer"

# Parallel agents
"Launch two Explore agents to analyze the frontend and backend separately"
```

---

## Phase 1: Requirements & Specification

### Goal: Turn ideas into clear, actionable specifications

### Workflow 1.1: Research Existing Patterns

Before writing specs, understand what exists.

```
User: "I want to add user notifications. Use the Explore agent to find
how notifications or messaging is currently handled in this codebase."
```

**What happens:**
- Explore agent searches for notification-related code
- Identifies existing patterns, services, database schemas
- Returns summary without cluttering your main context

### Workflow 1.2: Competitive Research

```
User: "Use the doc-fetcher agent to research best practices for
real-time notification systems. Look at how Slack, Discord, and
GitHub handle notifications."
```

**What happens:**
- Agent searches the web for documentation
- Compiles findings on notification patterns
- Returns actionable insights

### Workflow 1.3: PRD Writing with Context

After research, write your PRD with full context:

```
User: "Based on the exploration results, help me write a PRD for
the notification system. Include:
- Problem statement
- User stories
- Success metrics
- Technical constraints we discovered"
```

### Example PRD Structure

```markdown
# Product Requirements Document: Notification System

## Problem Statement
Users have no way to know when important events occur...

## User Stories
1. As a user, I want to receive notifications when...
2. As an admin, I want to configure notification rules...

## Technical Constraints (from Explore agent)
- Existing WebSocket infrastructure at `src/services/socket.ts`
- Current event system uses Redis pub/sub
- Database has `notifications` table (unused)

## Success Metrics
- 90% of notifications delivered within 5 seconds
- User engagement with notifications > 40%
```

---

## Phase 2: Architecture & Design

### Goal: Design system architecture before writing code

### Workflow 2.1: Parallel Architecture Exploration

Launch multiple agents to analyze different system aspects:

```
User: "Launch three Explore agents in parallel:
1. Analyze the current database schema and relationships
2. Explore the API layer and existing endpoints
3. Examine the frontend state management patterns"
```

**Benefits:**
- 3x faster than sequential exploration
- Each agent has focused context
- Results don't interfere with each other

### Workflow 2.2: Architecture Planning

```
User: "Use the Plan agent to design the architecture for our
notification system. Consider:
- The WebSocket infrastructure we found
- Redis pub/sub for event distribution
- Database schema for notification storage
- API endpoints needed
- Frontend components required"
```

**Plan agent returns:**
```markdown
## Notification System Architecture

### Components
1. NotificationService (backend)
   - Location: src/services/notifications/
   - Responsibilities: Create, store, deliver notifications

2. NotificationGateway (WebSocket)
   - Location: src/gateways/notification.gateway.ts
   - Connects to existing socket infrastructure

3. NotificationStore (frontend)
   - Location: src/stores/notifications.ts
   - Uses existing Zustand pattern

### Data Flow
User Action → Event Bus → NotificationService →
  → Database (persist)
  → Redis (broadcast)
  → WebSocket (deliver)
  → Frontend Store (display)

### Database Schema
- notifications (id, user_id, type, content, read_at, created_at)
- notification_preferences (user_id, channel, enabled)
```

### Workflow 2.3: Security Review of Design

```
User: "Use the security-auditor agent to review this notification
architecture for potential vulnerabilities"
```

**Security agent checks:**
- Authorization for notification access
- Data exposure risks
- WebSocket security
- Rate limiting needs

---

## Phase 3: Planning

### Goal: Break architecture into actionable implementation steps

### Workflow 3.1: Implementation Planning

```
User: "Use the Plan agent to create a detailed implementation plan
for the notification system. Break it into small, testable chunks
that can be implemented incrementally."
```

**Plan agent returns:**
```markdown
## Implementation Plan: Notification System

### Step 1: Database Layer
Files to create/modify:
- migrations/001_create_notifications.sql
- src/models/notification.model.ts
- src/repositories/notification.repository.ts

Deliverable: Database schema and data access layer
Test: Unit tests for repository methods

### Step 2: Core Service
Files to create/modify:
- src/services/notifications/notification.service.ts
- src/services/notifications/notification.types.ts
- src/services/notifications/index.ts

Deliverable: Business logic for CRUD operations
Test: Unit tests with mocked repository

### Step 3: Event Integration
Files to create/modify:
- src/events/notification.events.ts
- src/services/notifications/notification.publisher.ts

Deliverable: Event emission on notification creation
Test: Integration test with Redis

### Step 4: WebSocket Delivery
Files to create/modify:
- src/gateways/notification.gateway.ts
- src/services/notifications/notification.subscriber.ts

Deliverable: Real-time delivery to connected clients
Test: WebSocket integration test

### Step 5: REST API
Files to create/modify:
- src/controllers/notification.controller.ts
- src/routes/notification.routes.ts

Deliverable: API endpoints for notification management
Test: API integration tests

### Step 6: Frontend Components
Files to create/modify:
- src/components/NotificationBell.tsx
- src/components/NotificationList.tsx
- src/stores/notifications.ts
- src/hooks/useNotifications.ts

Deliverable: UI for displaying notifications
Test: Component tests with React Testing Library
```

### Workflow 3.2: Dependency Analysis

```
User: "Use the Explore agent to verify all files mentioned in the
plan exist and identify any missing dependencies"
```

### Workflow 3.3: Effort Estimation Context

```
User: "Use the Explore agent to find similar features in the
codebase (like the existing messaging system) to understand
the complexity patterns"
```

---

## Phase 4: Implementation

### Goal: Write code following the plan

### Workflow 4.1: Sequential Implementation with Context

For each step in your plan:

```
User: "Let's implement Step 1: Database Layer. First, use the
Explore agent to examine existing migration patterns and model
definitions in this codebase."
```

Then implement:

```
User: "Now implement the notification database layer following
the patterns we found. Create:
1. The migration file
2. The notification model
3. The repository with CRUD methods"
```

### Workflow 4.2: Parallel Implementation

For independent components:

```
User: "Launch two builder agents in parallel:
1. Implement the notification service (backend)
2. Implement the notification store (frontend)

These are independent and can be built simultaneously."
```

### Workflow 4.3: Implementation with Continuous Testing

```
User: "Implement the NotificationService. After each method,
write the corresponding unit test. Use TDD approach:
1. Write failing test
2. Implement to pass
3. Refactor"
```

### Workflow 4.4: Complex Feature Implementation

For complex features, use the orchestrator pattern:

```
User: "Use the orchestrator agent to coordinate implementing
the WebSocket notification delivery:
1. First explore existing WebSocket code
2. Plan the integration approach
3. Implement the gateway
4. Write integration tests
5. Review the implementation"
```

---

## Phase 5: Testing

### Goal: Comprehensive test coverage

### Workflow 5.1: Test Generation

```
User: "Use the test-writer agent to create comprehensive tests
for the NotificationService. Cover:
- Happy path for all methods
- Edge cases (empty lists, null values)
- Error conditions (database failures, invalid input)
- Boundary conditions (max notifications, pagination)"
```

### Workflow 5.2: Parallel Test Writing

```
User: "Launch three test-writer agents in parallel:
1. Unit tests for notification.service.ts
2. Integration tests for notification.controller.ts
3. E2E tests for the notification flow"
```

### Workflow 5.3: Test Review and Gap Analysis

```
User: "Use the Explore agent to analyze test coverage:
1. Find all notification-related source files
2. Find all notification-related test files
3. Identify files without corresponding tests"
```

### Workflow 5.4: Mutation Testing Preparation

```
User: "Use the security-auditor agent to review our tests:
- Are there assertions that could pass with wrong code?
- Are edge cases properly covered?
- Are error paths tested?"
```

---

## Phase 6: Code Review

### Goal: Quality assurance before merge

### Workflow 6.1: Automated Code Review

```
User: "Use the reviewer agent to perform a code review of all
files changed for the notification feature. Check for:
- Code quality and readability
- Adherence to project patterns
- Potential bugs or issues
- Performance concerns
- Security vulnerabilities"
```

### Workflow 6.2: Focused Security Review

```
User: "Use the security-auditor agent to review the notification
system specifically for:
- Authentication/authorization issues
- Data exposure risks
- Input validation
- SQL injection possibilities
- XSS vulnerabilities in frontend"
```

### Workflow 6.3: Architecture Compliance Review

```
User: "Use the Explore agent to verify our implementation matches
the original architecture plan:
1. Are all planned files created?
2. Do the data flows match the design?
3. Are there any deviations that need documentation?"
```

### Workflow 6.4: PR Description Generation

```
User: "Generate a comprehensive PR description for the notification
feature. Use the Explore agent to:
1. List all changed files
2. Summarize the changes in each
3. Document any breaking changes
4. List testing performed"
```

**Example PR Output:**
```markdown
## Summary
Implements real-time notification system with WebSocket delivery.

## Changes
- **Database**: New notifications table with preferences
- **Backend**: NotificationService with pub/sub integration
- **API**: CRUD endpoints at /api/notifications
- **Frontend**: NotificationBell and NotificationList components

## Testing
- 45 unit tests (100% pass)
- 12 integration tests (100% pass)
- Manual E2E testing completed

## Breaking Changes
None

## Deployment Notes
- Run migration: `npm run migrate`
- Redis 6.0+ required for pub/sub features
```

---

## Phase 7: Deployment & CI/CD

### Goal: Safe deployment with proper automation

### Workflow 7.1: CI Pipeline Analysis

```
User: "Use the Explore agent to examine our current CI/CD setup:
1. Find all GitHub Actions workflows
2. Identify the deployment process
3. List environment configurations"
```

### Workflow 7.2: Deployment Checklist Generation

```
User: "Based on the notification feature, use the Plan agent to
create a deployment checklist:
1. Pre-deployment requirements
2. Database migration steps
3. Environment variable changes
4. Feature flag configuration
5. Rollback procedure"
```

**Plan agent returns:**
```markdown
## Deployment Checklist: Notification System

### Pre-deployment
- [ ] All tests passing in CI
- [ ] Code review approved
- [ ] Documentation updated
- [ ] Redis 6.0+ available in production

### Database
- [ ] Backup production database
- [ ] Run migration in staging first
- [ ] Verify migration: `SELECT * FROM notifications LIMIT 1`
- [ ] Run migration in production

### Environment Variables
- [ ] NOTIFICATION_REDIS_URL - Redis connection for pub/sub
- [ ] NOTIFICATION_WS_PATH - WebSocket endpoint path
- [ ] NOTIFICATION_RATE_LIMIT - Max notifications per minute

### Feature Flags
- [ ] Enable FEATURE_NOTIFICATIONS for beta users first
- [ ] Monitor error rates for 24 hours
- [ ] Gradually roll out to all users

### Rollback Procedure
1. Disable feature flag immediately
2. No database rollback needed (additive only)
3. Deploy previous version if critical
```

### Workflow 7.3: CI Workflow Updates

```
User: "Help me update the CI workflow to include notification
system tests. Use the Explore agent to find the current workflow
file, then suggest the necessary changes."
```

### Workflow 7.4: Monitoring Setup

```
User: "Use the doc-fetcher agent to research best practices for
monitoring notification systems, then help me set up appropriate
alerts and dashboards."
```

---

## Advanced Patterns

### Pattern 1: Parallel Exploration Pipeline

For large codebases, explore multiple areas simultaneously:

```
User: "Launch five Explore agents in parallel to analyze:
1. Authentication system
2. Database layer
3. API structure
4. Frontend architecture
5. Testing patterns

I need a complete codebase understanding before starting work."
```

### Pattern 2: Review-Fix Loop

Automate the review and fix cycle:

```
User: "Run this loop:
1. Use reviewer agent to find issues
2. Use fixer agent to address each issue
3. Use reviewer agent to verify fixes
Continue until no issues remain."
```

### Pattern 3: Documentation Generation

Generate comprehensive documentation:

```
User: "Launch parallel agents:
1. Explore agent: Find all public APIs
2. Explore agent: Find all configuration options
3. doc-fetcher: Get API documentation best practices

Then compile into API documentation."
```

### Pattern 4: Incremental Refactoring

Safe refactoring with continuous verification:

```
User: "Use the refactorer agent to improve the notification service:
1. Identify code smells
2. Propose refactoring for each
3. Implement one at a time
4. Run tests after each change
5. Only proceed if tests pass"
```

### Pattern 5: Cross-Cutting Concern Analysis

Analyze concerns that span the codebase:

```
User: "Launch parallel Explore agents to find all places where:
1. User authentication is checked
2. Database transactions are used
3. Error handling patterns
4. Logging is implemented

I need to ensure consistency across the codebase."
```

---

## Quick Reference

### Agent Selection Guide

| Task | Recommended Agent |
|------|-------------------|
| Understand existing code | `Explore` |
| Design architecture | `Plan` |
| Research best practices | `doc-fetcher` |
| Write implementation | `builder` or direct |
| Write tests | `test-writer` |
| Review code quality | `reviewer` |
| Find security issues | `security-auditor` |
| Fix bugs | `fixer` |
| Improve code structure | `refactorer` |
| Run shell commands | `Bash` |
| Complex multi-step tasks | `general-purpose` |
| Coordinate multiple agents | `orchestrator` |

### Parallelization Rules

**CAN parallelize:**
- Multiple Explore agents on different areas
- Independent test writing
- Review + security audit (both read-only)
- Frontend + backend implementation (if independent)

**CANNOT parallelize:**
- Tasks that depend on each other's output
- Multiple writes to the same file
- Sequential workflow steps

### Best Practices

1. **Start with Explore** - Understand before implementing
2. **Plan before building** - Use Plan agent for non-trivial features
3. **Parallelize when possible** - Multiple agents = faster results
4. **Keep context clean** - Use agents to isolate heavy operations
5. **Review everything** - Use reviewer and security-auditor before merge
6. **Test continuously** - Write tests alongside implementation
7. **Document decisions** - Capture why, not just what

### Common Commands

```bash
# Explore codebase
"Use Explore agent to understand the authentication system"

# Plan feature
"Use Plan agent to design implementation for [feature]"

# Parallel exploration
"Launch two Explore agents in parallel to analyze [area1] and [area2]"

# Code review
"Use reviewer agent to review changes in [files]"

# Security check
"Use security-auditor to check for vulnerabilities in [area]"

# Generate tests
"Use test-writer to create tests for [component]"

# Fix issues
"Use fixer agent to address the issues found by reviewer"
```

---

## Conclusion

Effective use of Claude Code agents transforms the development process:

1. **Requirements**: Research with `Explore` and `doc-fetcher`
2. **Architecture**: Design with `Plan`, verify with `security-auditor`
3. **Planning**: Break down with `Plan`, verify with `Explore`
4. **Implementation**: Build with focused context, parallelize independent work
5. **Testing**: Generate comprehensive tests with `test-writer`
6. **Review**: Quality check with `reviewer` and `security-auditor`
7. **Deployment**: Plan rollout with `Plan`, verify with `Explore`

The key is matching the right agent to each task and leveraging parallelization for speed.

---

*Tutorial created for Claude Code v2.1.11*
