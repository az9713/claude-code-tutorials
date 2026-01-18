# Tutorial 21: CLAUDE.md Mastery

## Complete Guide to Project Memory and Context in Claude Code

**Difficulty**: Beginner to Intermediate
**Duration**: 45-60 minutes
**Last Updated**: January 2026

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Core Concepts](#core-concepts)
4. [Understanding CLAUDE.md](#understanding-claudemd)
5. [The 4-Level Hierarchy](#the-4-level-hierarchy)
6. [Creating Effective CLAUDE.md Files](#creating-effective-claudemd-files)
7. [Modular Rules System](#modular-rules-system)
8. [Advanced Features](#advanced-features)
9. [Templates](#templates)
10. [Real-World Examples](#real-world-examples)
11. [Hierarchy Precedence](#hierarchy-precedence)
12. [Troubleshooting](#troubleshooting)
13. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)
14. [Best Practices](#best-practices)
15. [Exercises](#exercises)
16. [Summary](#summary)

---

## Overview

### What is CLAUDE.md?

CLAUDE.md is Claude Code's **persistent memory system** - a markdown file that provides project context, coding standards, architectural guidelines, and custom instructions that Claude automatically loads at the start of every session.

Think of CLAUDE.md as your project's "developer handbook" that Claude reads before starting any work. Instead of repeatedly explaining your project's conventions, architecture, or preferences, you write them once in CLAUDE.md, and Claude remembers them throughout your development workflow.

### Why CLAUDE.md Matters

Without CLAUDE.md:
```
You: "Add a new user service"
Claude: *Creates a service with generic patterns*
You: "No, we use a specific pattern here..."
Claude: *Modifies to match*
You: "Also, we always use TypeScript strict mode..."
Claude: *Updates again*
You: "And our naming convention is..."
```

With CLAUDE.md:
```
You: "Add a new user service"
Claude: *Creates service following your exact patterns, naming conventions,
        TypeScript settings, and project structure automatically*
```

### The Memory System

Claude Code's memory system operates on multiple levels:

```
┌─────────────────────────────────────────────────────────────────┐
│                     CLAUDE'S CONTEXT                            │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────────┐  ┌──────────────────┐  ┌───────────────┐ │
│  │   CLAUDE.md      │  │   Session        │  │   Codebase    │ │
│  │   Files          │  │   History        │  │   Analysis    │ │
│  │   (Persistent)   │  │   (Temporary)    │  │   (Dynamic)   │ │
│  └──────────────────┘  └──────────────────┘  └───────────────┘ │
│           │                    │                    │           │
│           v                    v                    v           │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │              Combined Understanding                          ││
│  │   Project rules + Current conversation + Code context        ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

### Project Context Benefits

1. **Consistency**: Every interaction follows your project's standards
2. **Efficiency**: No need to repeat instructions
3. **Onboarding**: New team members get Claude up to speed instantly
4. **Documentation**: CLAUDE.md serves as living documentation
5. **Quality**: Reduces errors from miscommunication

---

## Prerequisites

### Required Knowledge

Before starting this tutorial, you should have:

1. **Claude Code Installed and Running**
   ```bash
   # Verify installation
   claude --version

   # Should output something like:
   # Claude Code v1.x.x
   ```

2. **Basic Understanding of Prompts**
   - You know how to interact with Claude Code
   - You understand that prompts are instructions to Claude
   - You've completed at least a few coding tasks with Claude

3. **Markdown Familiarity**
   - Headers (`#`, `##`, `###`)
   - Code blocks (``` ```)
   - Lists (`-` or `1.`)
   - Basic formatting (`**bold**`, `*italic*`)

4. **Project Structure Understanding**
   - You know what your project's root directory is
   - You understand file paths
   - You know your project's basic architecture

### Recommended Background

- Completed Tutorial 01: Getting Started with Claude Code
- Used Claude Code for at least 5-10 tasks
- Experience with at least one programming project

### Tools Needed

- Text editor (VS Code, Vim, etc.)
- Terminal/Command line access
- A project to work with (or create a test project)

---

## Core Concepts

### The 4-Level Hierarchy

Claude Code's CLAUDE.md system operates on four hierarchical levels:

```
┌────────────────────────────────────────────────────────────────┐
│                    CLAUDE.md Hierarchy                          │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Level 4: User (~/.claude/CLAUDE.md)                          │
│   ├── Personal preferences                                      │
│   ├── Highest priority for personal settings                   │
│   └── Applies to ALL your projects                             │
│                         ▲                                       │
│                         │ (adds to)                             │
│                         │                                       │
│   Level 3: Rules (.claude/rules/*.md)                          │
│   ├── Modular rule files                                        │
│   ├── Path-specific rules                                       │
│   └── Organized by concern                                      │
│                         ▲                                       │
│                         │ (adds to)                             │
│                         │                                       │
│   Level 2: Project (./CLAUDE.md)                               │
│   ├── Project root file                                         │
│   ├── Team-shared context                                       │
│   └── Core project information                                  │
│                         ▲                                       │
│                         │ (adds to)                             │
│                         │                                       │
│   Level 1: Enterprise (/etc/claude/CLAUDE.md)                  │
│   ├── Organization-wide rules                                   │
│   ├── Managed by administrators                                 │
│   └── Cannot be overridden                                      │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

### @imports - Modular Content

The `@import` syntax allows you to break up CLAUDE.md into manageable pieces:

```markdown
# Main CLAUDE.md

@import .claude/rules/typescript.md
@import .claude/rules/testing.md
@import docs/architecture.md
```

This keeps your main CLAUDE.md clean while organizing rules by concern.

### Path-Specific Rules

Rules can apply only to specific directories or file types:

```markdown
# .claude/rules/frontend.md

@path: src/components/**
@path: src/pages/**

## React Component Rules
- Use functional components
- Props interfaces required
```

These rules only activate when Claude is working in matching paths.

### Key Terminology

| Term | Definition |
|------|------------|
| **CLAUDE.md** | Markdown file containing project context |
| **Rule File** | Modular markdown file in `.claude/rules/` |
| **@import** | Directive to include another file's content |
| **@path** | Directive to limit rules to specific paths |
| **Context** | Information Claude uses to understand your project |
| **Hierarchy** | The 4-level system of CLAUDE.md precedence |

---

## Understanding CLAUDE.md

### Purpose: Persistent Project Memory

CLAUDE.md serves as Claude's long-term memory for your project. Unlike conversation history that resets between sessions, CLAUDE.md persists and is loaded fresh each time.

**What CLAUDE.md Provides:**

```
┌─────────────────────────────────────────────────────────────┐
│                 CLAUDE.md Contents                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  PROJECT IDENTITY                                           │
│  ├── What is this project?                                  │
│  ├── What problem does it solve?                            │
│  └── Who is the target user?                                │
│                                                             │
│  TECHNICAL CONTEXT                                          │
│  ├── Tech stack details                                     │
│  ├── Architecture patterns                                  │
│  ├── Key dependencies                                       │
│  └── Environment setup                                      │
│                                                             │
│  CODING STANDARDS                                           │
│  ├── Naming conventions                                     │
│  ├── Code organization                                      │
│  ├── Testing requirements                                   │
│  └── Documentation standards                                │
│                                                             │
│  WORKFLOW INFORMATION                                       │
│  ├── Common commands                                        │
│  ├── Deployment process                                     │
│  ├── Git conventions                                        │
│  └── Review process                                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Automatic Loading at Session Start

When you start Claude Code in a directory, it automatically:

1. **Searches for CLAUDE.md** in the project root
2. **Loads any @imports** referenced in the file
3. **Applies path-specific rules** based on context
4. **Combines with user-level settings** from `~/.claude/CLAUDE.md`

```
Session Start
     │
     v
┌─────────────────────┐
│ Check for CLAUDE.md │
└─────────────────────┘
     │
     v
┌─────────────────────┐
│ Parse @import       │
│ directives          │
└─────────────────────┘
     │
     v
┌─────────────────────┐
│ Load rule files     │
│ from .claude/rules/ │
└─────────────────────┘
     │
     v
┌─────────────────────┐
│ Load user CLAUDE.md │
│ ~/.claude/CLAUDE.md │
└─────────────────────┘
     │
     v
┌─────────────────────┐
│ Combine all context │
│ Ready to work!      │
└─────────────────────┘
```

### How Claude Uses CLAUDE.md Content

Claude doesn't just read CLAUDE.md - it **actively applies** the content:

**Example Scenario:**

Your CLAUDE.md says:
```markdown
## Naming Conventions
- React components: PascalCase
- Utility functions: camelCase
- Constants: SCREAMING_SNAKE_CASE
- Files: kebab-case.ts
```

When you ask: "Create a utility function for date formatting"

Claude will:
1. Name the function `formatDate` (camelCase)
2. Create file `date-utils.ts` (kebab-case)
3. Any constants like `DEFAULT_FORMAT` (SCREAMING_SNAKE_CASE)

**Claude's Internal Processing:**

```
┌─────────────────────────────────────────────────────────────┐
│                  Claude's Reasoning                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  User Request: "Create a utility function for dates"       │
│                                                             │
│  CLAUDE.md Context Applied:                                 │
│  ├── Check naming conventions → camelCase for utilities    │
│  ├── Check file naming → kebab-case.ts                     │
│  ├── Check project structure → utils/ directory            │
│  ├── Check TypeScript rules → strict mode, explicit types  │
│  └── Check testing rules → create accompanying test        │
│                                                             │
│  Result: Generates code following ALL specified patterns   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### What to Include vs. What to Omit

**Include:**
- Standards that should ALWAYS apply
- Project-specific patterns
- Non-obvious conventions
- Commonly referenced information

**Omit:**
- Obvious programming practices
- Information easily found in code
- Frequently changing details
- Sensitive credentials (use .env)

---

## The 4-Level Hierarchy

### Level 1: Enterprise CLAUDE.md

**Location**: `/etc/claude/CLAUDE.md` (Linux/macOS) or system equivalent

**Purpose**: Organization-wide standards that apply to ALL projects

**Who Manages It**: IT administrators, DevOps teams

**Characteristics**:
- Cannot be overridden by lower levels
- Enforces compliance requirements
- Sets security standards
- Defines approved technologies

**Example Enterprise CLAUDE.md:**

```markdown
# Enterprise Development Standards

## Security Requirements
- All API keys must use environment variables
- No hardcoded credentials in any file
- All user input must be sanitized
- HTTPS required for all external calls

## Compliance
- GDPR compliance required for user data
- Audit logging for all data access
- Data retention policies must be followed

## Approved Technologies
- Languages: TypeScript, Python, Go
- Databases: PostgreSQL, MongoDB
- Cloud: AWS only
- CI/CD: GitHub Actions

## Code Review Requirements
- All changes require peer review
- Security review for auth changes
- Performance review for database changes

## Documentation
- All public APIs must be documented
- README required in all repositories
- Architecture Decision Records (ADRs) for major decisions
```

**When Enterprise Rules Apply:**
```
┌─────────────────────────────────────────────────────────────┐
│  User: "Store the API key in config.ts"                    │
│                                                             │
│  Claude (checking Enterprise CLAUDE.md):                   │
│  "I notice the enterprise standards require API keys       │
│   to use environment variables. Instead of storing it      │
│   in config.ts, I'll set up a .env file and use           │
│   process.env.API_KEY."                                    │
└─────────────────────────────────────────────────────────────┘
```

### Level 2: Project CLAUDE.md

**Location**: `./CLAUDE.md` in project root

**Purpose**: Project-specific context shared by the entire team

**Who Manages It**: Tech leads, senior developers, team collectively

**Characteristics**:
- Committed to version control
- Shared with all team members
- Describes project-specific patterns
- Core development reference

**Example Project CLAUDE.md:**

```markdown
# TaskFlow - Project Management Application

## Overview
TaskFlow is a modern project management tool built for agile teams.
It features real-time collaboration, sprint planning, and analytics.

## Tech Stack
- **Frontend**: React 18, TypeScript, TailwindCSS
- **Backend**: Node.js, Express, GraphQL
- **Database**: PostgreSQL with Prisma ORM
- **Cache**: Redis
- **Testing**: Jest, Playwright
- **Deployment**: Docker, Kubernetes, AWS

## Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                      Client (React SPA)                     │
└─────────────────────────────────────────────────────────────┘
                              │
                              v
┌─────────────────────────────────────────────────────────────┐
│                    API Gateway (Express)                    │
└─────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              v               v               v
        ┌─────────┐     ┌─────────┐     ┌─────────┐
        │  Auth   │     │  Tasks  │     │Analytics│
        │ Service │     │ Service │     │ Service │
        └─────────┘     └─────────┘     └─────────┘
              │               │               │
              v               v               v
        ┌─────────────────────────────────────────┐
        │              PostgreSQL                 │
        └─────────────────────────────────────────┘
```

## Project Structure
```
taskflow/
├── apps/
│   ├── web/              # React frontend
│   ├── api/              # Express backend
│   └── docs/             # Documentation site
├── packages/
│   ├── ui/               # Shared UI components
│   ├── types/            # Shared TypeScript types
│   └── utils/            # Shared utilities
├── prisma/               # Database schema
├── docker/               # Docker configurations
└── scripts/              # Build and deploy scripts
```

## Coding Standards
- Use functional components with hooks
- Prefer composition over inheritance
- Keep functions under 50 lines
- Maximum file size: 300 lines
- All exports must be named (no default exports)

## Key Files
- `apps/api/src/schema.graphql` - API schema
- `prisma/schema.prisma` - Database models
- `packages/types/index.ts` - Shared types
- `apps/web/src/config.ts` - Frontend config

## Common Commands
- `pnpm dev` - Start all services
- `pnpm test` - Run test suite
- `pnpm db:migrate` - Run migrations
- `pnpm build` - Build for production
- `pnpm lint` - Run linting

## Branch Naming
- `feature/TASK-123-description`
- `bugfix/TASK-123-description`
- `hotfix/description`

## Commit Messages
Follow conventional commits:
- `feat: add user authentication`
- `fix: resolve memory leak in task list`
- `docs: update API documentation`
- `refactor: simplify task creation flow`
```

### Level 3: Rules Files

**Location**: `.claude/rules/*.md`

**Purpose**: Modular, organized rule files for specific concerns

**Who Manages It**: Developers working on specific areas

**Characteristics**:
- Organized by domain/concern
- Can be path-specific
- Easier to maintain than one large file
- Allows granular updates

**Example Rules Structure:**

```
.claude/
└── rules/
    ├── typescript.md       # TypeScript conventions
    ├── testing.md          # Testing requirements
    ├── react.md            # React patterns
    ├── api.md              # API development rules
    ├── database.md         # Database conventions
    ├── security.md         # Security practices
    └── git.md              # Git workflow rules
```

**Example: .claude/rules/react.md**

```markdown
# React Development Rules

@path: apps/web/**
@path: packages/ui/**

## Component Structure

Every React component should follow this structure:

```typescript
// 1. Imports
import React from 'react';
import { useQuery } from '@tanstack/react-query';

// 2. Types
interface ComponentProps {
  title: string;
  onAction: () => void;
}

// 3. Component
export function Component({ title, onAction }: ComponentProps) {
  // 3a. Hooks
  const { data } = useQuery(['key'], fetchData);

  // 3b. Handlers
  const handleClick = () => {
    onAction();
  };

  // 3c. Render
  return (
    <div>
      <h1>{title}</h1>
      <button onClick={handleClick}>Action</button>
    </div>
  );
}
```

## Hooks Rules
- Custom hooks must start with `use`
- Keep hooks at the top of components
- Extract complex logic to custom hooks
- Maximum 5 useState calls per component

## State Management
- Local state: useState for component-only state
- Shared state: Zustand stores in `/stores`
- Server state: TanStack Query

## Performance
- Memoize expensive computations with useMemo
- Memoize callbacks with useCallback
- Use React.memo for list items
- Lazy load routes with React.lazy
```

### Level 4: User CLAUDE.md

**Location**: `~/.claude/CLAUDE.md`

**Purpose**: Personal preferences that apply to ALL your projects

**Who Manages It**: Individual developer

**Characteristics**:
- Not shared with team
- Personal coding style preferences
- Learning notes
- Accessibility preferences

**Example User CLAUDE.md:**

```markdown
# My Personal Claude Preferences

## Communication Style
- Be concise and direct
- Skip obvious explanations
- Show code first, explain after
- Use bullet points over paragraphs

## Coding Preferences
- I prefer explicit over implicit
- Always show full file paths
- Include relevant imports in examples
- Add comments for complex logic

## My Common Patterns
- I use `pnpm` instead of `npm` or `yarn`
- I prefer absolute imports with `@/` prefix
- I like descriptive variable names
- I use async/await over .then()

## Learning Notes
- Currently learning: Rust, WebAssembly
- Ask me about new patterns before using them
- Explain advanced TypeScript features

## Accessibility
- I use Vim keybindings
- Prefer terminal-based solutions
- Dark mode references in examples

## Things I Forget
- Always remind me to update tests
- Prompt for git commits after changes
- Check for TypeScript errors before done
```

---

## Creating Effective CLAUDE.md Files

### Project-Level CLAUDE.md

#### Essential Sections

**1. Project Description**

```markdown
# ProjectName

## Overview
One paragraph describing what the project does, its purpose,
and its target users. This helps Claude understand the context
for all decisions.

Example:
"QuickCart is an e-commerce platform for small businesses.
It provides inventory management, order processing, and
customer analytics. Our users are non-technical shop owners
who need simple, reliable tools."
```

**2. Architecture Overview**

```markdown
## Architecture

### High-Level Design
Brief description of the overall architecture pattern
(monolith, microservices, serverless, etc.)

### System Diagram
```
[ASCII diagram showing main components and data flow]
```

### Key Design Decisions
- Why we chose X over Y
- Trade-offs we made
- Constraints we work within
```

**3. Coding Standards**

```markdown
## Coding Standards

### General Principles
- [Principle 1]
- [Principle 2]

### Naming Conventions
| Type | Convention | Example |
|------|------------|---------|
| Variables | camelCase | `userName` |
| Constants | SCREAMING_SNAKE | `MAX_RETRIES` |
| Functions | camelCase | `getUserById` |
| Classes | PascalCase | `UserService` |
| Files | kebab-case | `user-service.ts` |

### Code Organization
- How files should be structured
- Import ordering
- Function ordering within files
```

**4. Important Files/Directories**

```markdown
## Important Files

### Configuration
- `config/` - All configuration files
- `.env.example` - Environment template
- `tsconfig.json` - TypeScript settings

### Core Code
- `src/core/` - Business logic
- `src/api/` - API endpoints
- `src/models/` - Data models

### Entry Points
- `src/index.ts` - Main entry
- `src/cli.ts` - CLI entry
- `src/worker.ts` - Background worker
```

**5. Common Tasks**

```markdown
## Common Tasks

### Development
```bash
# Start development server
npm run dev

# Run with debugging
npm run dev:debug
```

### Testing
```bash
# Run all tests
npm test

# Run specific test file
npm test -- user.test.ts

# Run with coverage
npm run test:coverage
```

### Database
```bash
# Run migrations
npm run db:migrate

# Seed database
npm run db:seed

# Reset database
npm run db:reset
```

### Deployment
```bash
# Build for production
npm run build

# Deploy to staging
npm run deploy:staging

# Deploy to production
npm run deploy:prod
```
```

#### Writing Tips

**Be Specific, Not Generic:**

```markdown
# Bad (too generic)
## Testing
Write good tests.

# Good (specific and actionable)
## Testing
- Unit tests required for all utility functions
- Integration tests for API endpoints
- Use `describe` blocks named after the function/component
- Use `it` blocks that read as sentences: "it should return null when..."
- Mock external services using `jest.mock()`
- Minimum 80% line coverage enforced by CI
```

**Include the "Why":**

```markdown
# Bad (just rules)
## Database
Always use transactions for multi-table updates.

# Good (rules with reasoning)
## Database
Always use transactions for multi-table updates.

> We use eventual consistency for analytics data but strict
> consistency for financial transactions. When updating order
> status, always wrap the order update and inventory adjustment
> in a transaction to prevent overselling.
```

**Keep It Current:**

```markdown
## Recent Changes (Keep Updated!)

### January 2026
- Migrated from Jest to Vitest for faster tests
- New authentication system using Clerk
- Deprecated: `src/auth/legacy/` - do not modify

### December 2025
- Added GraphQL layer
- New caching strategy with Redis
```

### Modular Organization Best Practices

When your CLAUDE.md grows beyond 200 lines, consider splitting:

```
CLAUDE.md (50-100 lines)
├── Project overview
├── Quick reference
├── @import statements
└── Team contact info

.claude/rules/
├── language-specific/
│   ├── typescript.md
│   ├── python.md
│   └── sql.md
├── domain-specific/
│   ├── auth.md
│   ├── payments.md
│   └── notifications.md
├── process/
│   ├── git.md
│   ├── review.md
│   └── deploy.md
└── testing/
    ├── unit.md
    ├── integration.md
    └── e2e.md
```

---

## Modular Rules System

### Breaking Up Large CLAUDE.md Files

**Signs You Need to Modularize:**

1. CLAUDE.md exceeds 300 lines
2. Different team members edit different sections
3. Rules only apply to specific parts of codebase
4. You find yourself scrolling to find sections

**Modularization Strategy:**

```
Before: One large CLAUDE.md (500+ lines)
┌─────────────────────────────────────────┐
│  Project Overview                       │
│  Tech Stack                             │
│  TypeScript Rules                       │
│  React Rules                            │
│  API Rules                              │
│  Database Rules                         │
│  Testing Rules                          │
│  Git Workflow                           │
│  Deployment                             │
│  Security                               │
│  ...                                    │
└─────────────────────────────────────────┘

After: Main file + modular rules
┌─────────────────────────────────────────┐
│  CLAUDE.md (slim)                       │
│  ├── Project Overview                   │
│  ├── Tech Stack                         │
│  ├── Quick Commands                     │
│  └── @imports                           │
└─────────────────────────────────────────┘
            │
            ├── @import .claude/rules/typescript.md
            ├── @import .claude/rules/react.md
            ├── @import .claude/rules/api.md
            ├── @import .claude/rules/database.md
            ├── @import .claude/rules/testing.md
            └── @import .claude/rules/security.md
```

### Rule File Naming Conventions

**By Technology:**
```
.claude/rules/
├── typescript.md
├── react.md
├── nodejs.md
├── postgresql.md
└── redis.md
```

**By Domain:**
```
.claude/rules/
├── authentication.md
├── authorization.md
├── payments.md
├── notifications.md
└── analytics.md
```

**By Layer:**
```
.claude/rules/
├── frontend.md
├── backend.md
├── database.md
├── infrastructure.md
└── testing.md
```

**Hybrid Approach (Recommended):**
```
.claude/rules/
├── languages/
│   ├── typescript.md
│   └── sql.md
├── frameworks/
│   ├── react.md
│   └── express.md
├── domains/
│   ├── auth.md
│   └── payments.md
└── processes/
    ├── testing.md
    ├── git.md
    └── deploy.md
```

### Conditional Rule Loading

Rules in `.claude/rules/` are always loaded, but you can make them contextual with `@path`:

```markdown
# .claude/rules/frontend.md

@path: src/frontend/**
@path: src/components/**
@path: packages/ui/**

## Frontend-Specific Rules
These rules only apply when working in frontend code.
```

When Claude works on `src/api/users.ts`, these rules won't apply.
When Claude works on `src/frontend/pages/Home.tsx`, they will.

### Path-Specific Rules

**Single Path:**
```markdown
@path: src/components/**
```

**Multiple Paths:**
```markdown
@path: src/components/**
@path: src/pages/**
@path: packages/ui/**
```

**File Type Specific:**
```markdown
@path: **/*.test.ts
@path: **/*.spec.ts
```

**Negation (exclude paths):**
```markdown
@path: src/**
@path: !src/legacy/**
```

**Example: Different Rules for Different Areas:**

```markdown
# .claude/rules/legacy.md
@path: src/legacy/**

## Legacy Code Rules
- Do not refactor without explicit request
- Maintain backwards compatibility
- Add tests before any changes
- Document all modifications
- Use legacy naming conventions (snake_case)
```

```markdown
# .claude/rules/modern.md
@path: src/v2/**
@path: src/new/**

## Modern Code Rules
- Use latest TypeScript features
- Strict typing required
- Follow new architecture patterns
- Use modern naming (camelCase)
```

---

## Advanced Features

### @import Syntax

The `@import` directive allows you to include content from other files.

**Basic Syntax:**
```markdown
@import path/to/file.md
```

**Relative Paths (from CLAUDE.md location):**
```markdown
@import .claude/rules/typescript.md
@import docs/architecture.md
@import ../shared/standards.md
```

**Multiple Imports:**
```markdown
# CLAUDE.md

# My Project

@import .claude/rules/general.md
@import .claude/rules/typescript.md
@import .claude/rules/testing.md
@import .claude/rules/security.md
```

**Importing from Different Locations:**
```markdown
# Import project rules
@import .claude/rules/project.md

# Import from documentation
@import docs/api-patterns.md

# Import shared team standards
@import ~/team-standards/coding.md
```

### Conditional Imports

While `@import` itself is unconditional, you can organize files for conditional loading:

```
.claude/rules/
├── always/           # Always loaded
│   ├── general.md
│   └── security.md
└── conditional/      # Path-specific (use @path in files)
    ├── frontend.md   # @path: src/frontend/**
    ├── backend.md    # @path: src/api/**
    └── mobile.md     # @path: src/mobile/**
```

**Main CLAUDE.md:**
```markdown
# Import always-needed rules
@import .claude/rules/always/general.md
@import .claude/rules/always/security.md

# Import conditional rules (they self-filter with @path)
@import .claude/rules/conditional/frontend.md
@import .claude/rules/conditional/backend.md
@import .claude/rules/conditional/mobile.md
```

### Import Best Practices

**1. Order Matters:**
```markdown
# Import general rules first
@import .claude/rules/general.md

# Then technology-specific
@import .claude/rules/typescript.md

# Then domain-specific (more specific overrides general)
@import .claude/rules/auth.md
```

**2. Avoid Circular Imports:**
```markdown
# BAD: Don't do this
# In rules/a.md: @import rules/b.md
# In rules/b.md: @import rules/a.md
```

**3. Keep Import Chains Shallow:**
```markdown
# GOOD: Direct imports
@import .claude/rules/typescript.md
@import .claude/rules/react.md

# AVOID: Deep nesting
# CLAUDE.md → rules/main.md → rules/lang.md → rules/ts.md
```

**4. Comment Your Imports:**
```markdown
# Core project rules
@import .claude/rules/general.md

# Language-specific standards
@import .claude/rules/typescript.md
@import .claude/rules/python.md

# Testing requirements
@import .claude/rules/testing.md
```

### Path-Specific Rules Deep Dive

**Combining Path Rules:**

```markdown
# .claude/rules/components.md

# These rules apply to component directories
@path: src/components/**
@path: src/shared/components/**
@path: packages/ui/src/**

## Component Guidelines

### File Structure
Each component gets its own directory:
```
ComponentName/
├── index.ts           # Re-exports
├── ComponentName.tsx  # Main component
├── ComponentName.test.tsx
├── ComponentName.styles.ts
└── types.ts           # Component-specific types
```

### Required Exports
- Named export of component
- Type export of props interface
- No default exports
```

**File Extension Rules:**

```markdown
# .claude/rules/test-files.md

@path: **/*.test.ts
@path: **/*.test.tsx
@path: **/*.spec.ts

## Test File Rules

### Structure
- Use describe blocks for grouping
- Use beforeEach for setup
- Use afterEach for cleanup
- One assertion per test preferred

### Naming
- Describe: Component or function name
- It: "should [behavior] when [condition]"

### Mocking
- Mock at the highest practical level
- Prefer jest.mock over manual mocks
- Reset mocks in afterEach
```

**Exclusion Patterns:**

```markdown
# .claude/rules/modern-code.md

# Apply to all source code except legacy
@path: src/**
@path: !src/legacy/**
@path: !src/deprecated/**
@path: !src/vendor/**

## Modern Code Standards
These apply everywhere except excluded directories.
```

---

## Templates

### Template 1: Basic CLAUDE.md

```markdown
# Project Name

## Overview
Brief description of the project, its purpose, and target users.

## Tech Stack
- **Frontend**: React 18 with TypeScript
- **Backend**: Node.js with Express
- **Database**: PostgreSQL
- **Testing**: Jest + React Testing Library
- **Deployment**: Docker + AWS

## Project Structure
```
src/
├── components/    # React components
├── hooks/         # Custom React hooks
├── services/      # API services
├── utils/         # Utility functions
├── types/         # TypeScript types
└── config/        # Configuration
```

## Coding Standards
- Use functional components with hooks
- Prefer named exports over default exports
- Use TypeScript strict mode
- Write tests for all new code
- Maximum function length: 50 lines

## Important Files
- `src/config/index.ts` - App configuration
- `src/services/api.ts` - API client setup
- `src/types/index.ts` - Shared type definitions
- `.env.example` - Environment variables template

## Common Tasks
```bash
# Development
npm run dev          # Start dev server
npm run dev:debug    # Start with debugging

# Testing
npm test             # Run tests
npm run test:watch   # Watch mode
npm run test:cov     # With coverage

# Building
npm run build        # Production build
npm run lint         # Run linter
npm run typecheck    # Check types
```

## Git Workflow
- Branch from `main` for features
- Use conventional commits
- Squash merge to main
- Delete branches after merge
```

### Template 2: TypeScript Rules

```markdown
# TypeScript Rules

## Type Definitions
- Always define explicit return types for functions
- Use interfaces for object shapes
- Use type for unions and intersections
- Avoid `any` - use `unknown` if type is truly unknown

## Naming Conventions
| Type | Convention | Example |
|------|------------|---------|
| Interfaces | PascalCase, I prefix | `IUserProps` |
| Types | PascalCase | `UserData` |
| Enums | PascalCase, singular | `Status` |
| Generics | Single capital letter | `T`, `K`, `V` |

## Strict Mode Settings
tsconfig.json must include:
```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true
  }
}
```

## Function Patterns
```typescript
// Good: Explicit types
function getUser(id: string): Promise<User | null> {
  // ...
}

// Good: Generic with constraints
function merge<T extends object>(a: T, b: Partial<T>): T {
  // ...
}

// Avoid: Implicit any
function process(data) { } // Bad
function process(data: unknown) { } // Better
```

## Import Organization
```typescript
// 1. External libraries
import React from 'react';
import { useQuery } from '@tanstack/react-query';

// 2. Internal absolute imports
import { UserService } from '@/services/user';

// 3. Relative imports
import { helper } from './utils';

// 4. Types (import type)
import type { User } from '@/types';
```
```

### Template 3: Testing Rules

```markdown
# Testing Rules

## Test File Location
- Unit tests: `__tests__/` directory next to source
- Integration tests: `tests/integration/`
- E2E tests: `tests/e2e/`

## Test File Naming
- Unit: `ComponentName.test.tsx`
- Integration: `feature-name.integration.test.ts`
- E2E: `user-journey.e2e.test.ts`

## Test Structure
```typescript
describe('UserService', () => {
  // Setup
  let service: UserService;

  beforeEach(() => {
    service = new UserService();
  });

  afterEach(() => {
    jest.resetAllMocks();
  });

  describe('getUser', () => {
    it('should return user when found', async () => {
      // Arrange
      const mockUser = { id: '1', name: 'Test' };

      // Act
      const result = await service.getUser('1');

      // Assert
      expect(result).toEqual(mockUser);
    });

    it('should return null when not found', async () => {
      // ...
    });
  });
});
```

## Naming Convention
- Describe: Component/function name
- It: "should [expected behavior] when [condition]"

## Coverage Requirements
| Type | Minimum |
|------|---------|
| Line coverage | 80% |
| Branch coverage | 75% |
| Critical paths | 100% |

## Mocking Guidelines
- Mock external APIs and services
- Use factories for test data
- Prefer dependency injection for testability
- Reset mocks between tests
```

### Template 4: Path-Specific Component Rules

```markdown
# Component Rules

@path: src/components/**
@path: packages/ui/src/**

## Component Structure

Every component file should follow this order:

1. Imports
2. Type definitions
3. Constants
4. Component function
5. Helper functions
6. Styled components (if any)
7. Export

## Example Structure
```typescript
// 1. Imports
import React, { useState, useCallback } from 'react';
import { Button } from '@/components/ui';
import type { User } from '@/types';

// 2. Types
interface UserCardProps {
  user: User;
  onSelect: (user: User) => void;
}

// 3. Constants
const ANIMATION_DURATION = 200;

// 4. Component
export function UserCard({ user, onSelect }: UserCardProps) {
  const [isHovered, setIsHovered] = useState(false);

  const handleClick = useCallback(() => {
    onSelect(user);
  }, [user, onSelect]);

  return (
    <div
      onMouseEnter={() => setIsHovered(true)}
      onMouseLeave={() => setIsHovered(false)}
    >
      <h3>{user.name}</h3>
      <Button onClick={handleClick}>Select</Button>
    </div>
  );
}

// 5. Helpers (if needed)
function formatUserName(user: User): string {
  return `${user.firstName} ${user.lastName}`;
}
```

## Required Patterns
- Every component must have TypeScript interface for props
- Use React.memo for list item components
- Extract to custom hooks if > 3 useState calls
- No inline styles - use styled-components or Tailwind
- Prop drilling max depth: 2 levels, then use context

## Component Complexity Limits
- Max lines per component: 200
- Max useState per component: 5
- Max useEffect per component: 3
- Extract to hooks if limits exceeded
```

### Template 5: @import Usage

```markdown
# My Project

## Quick Reference
This is a TypeScript React application with a Node.js backend.

## Imports

# Core language and framework rules
@import .claude/rules/typescript.md
@import .claude/rules/react.md
@import .claude/rules/nodejs.md

# Testing standards
@import .claude/rules/testing.md
@import .claude/rules/e2e-testing.md

# Domain-specific rules
@import .claude/rules/auth.md
@import .claude/rules/payments.md

# Process and workflow
@import .claude/rules/git.md
@import .claude/rules/code-review.md

## Project-Specific Context

### Current Sprint Focus
- User authentication overhaul
- Payment integration with Stripe
- Performance optimization

### Known Issues
- Memory leak in notification service (investigating)
- Slow query in user search (indexed, deploying fix)

### Team Contacts
- Frontend: @alice
- Backend: @bob
- DevOps: @charlie
```

---

## Real-World Examples

### Example 1: Monorepo with Package-Specific Rules

**Project Structure:**
```
monorepo/
├── CLAUDE.md
├── .claude/rules/
│   ├── general.md
│   └── packages/
│       ├── web.md
│       ├── mobile.md
│       ├── api.md
│       └── shared.md
├── packages/
│   ├── web/           # React web app
│   ├── mobile/        # React Native app
│   ├── api/           # Express backend
│   └── shared/        # Shared utilities
└── ...
```

**CLAUDE.md:**
```markdown
# MyCompany Monorepo

## Overview
Unified repository containing web app, mobile app, API, and shared libraries.

## Package Overview
| Package | Tech | Owner |
|---------|------|-------|
| @myco/web | React, TypeScript | Web Team |
| @myco/mobile | React Native | Mobile Team |
| @myco/api | Node, Express | Backend Team |
| @myco/shared | TypeScript | Platform Team |

## Workspace Commands
```bash
pnpm install          # Install all dependencies
pnpm dev              # Start all services
pnpm dev:web          # Start web only
pnpm dev:api          # Start API only
pnpm build            # Build all packages
pnpm test             # Test all packages
pnpm affected:test    # Test only changed packages
```

## Package Rules
@import .claude/rules/general.md
@import .claude/rules/packages/web.md
@import .claude/rules/packages/mobile.md
@import .claude/rules/packages/api.md
@import .claude/rules/packages/shared.md

## Cross-Package Dependencies
- All packages can depend on `@myco/shared`
- Web and Mobile can depend on `@myco/api-client`
- Never create circular dependencies
```

**.claude/rules/packages/web.md:**
```markdown
# Web Package Rules

@path: packages/web/**

## Tech Stack
- React 18 with TypeScript
- TanStack Query for data fetching
- Zustand for state management
- TailwindCSS for styling

## Component Location
- Pages: `src/pages/`
- Features: `src/features/[feature]/`
- Shared: `src/components/`

## State Management
- Server state: TanStack Query only
- UI state: Zustand stores
- Form state: React Hook Form
```

### Example 2: Legacy Codebase with Migration Guidelines

```markdown
# Legacy Application Migration

## Project Status
This is a gradual migration from jQuery + PHP to React + Node.js.

## Directory Structure
```
legacy-app/
├── legacy/           # DO NOT MODIFY unless critical
│   ├── php/          # PHP backend (deprecated)
│   └── jquery/       # jQuery frontend (deprecated)
├── src/              # New React frontend
├── api/              # New Node.js backend
└── migration/        # Migration utilities
```

## Migration Rules

### For Legacy Code (`legacy/`)
- Bug fixes ONLY - no new features
- Document any changes extensively
- Add migration TODO comments
- Create corresponding tests in new system

### For New Code (`src/`, `api/`)
- Follow modern patterns
- Create API compatibility layers
- Document legacy equivalents
- Feature flag new implementations

## Migration Patterns

### Converting jQuery to React
```javascript
// Legacy jQuery
$('.user-list').on('click', '.user-item', function() {
  showUserDetail($(this).data('id'));
});

// New React equivalent
function UserList({ users }) {
  return users.map(user => (
    <UserItem
      key={user.id}
      user={user}
      onClick={() => showUserDetail(user.id)}
    />
  ));
}
```

### API Compatibility
When creating new API endpoints, always:
1. Create new endpoint in `/api/v2/`
2. Create adapter in `/api/compat/` for legacy calls
3. Update migration tracking spreadsheet

## Feature Flags
All new features must be behind flags:
```typescript
if (featureFlags.isEnabled('new-user-dashboard')) {
  return <NewUserDashboard />;
}
return <LegacyUserDashboard />;
```
```

### Example 3: Open Source Project with Contribution Rules

```markdown
# OpenWidget - Open Source UI Component Library

## Welcome Contributors!
Thank you for your interest in contributing to OpenWidget.

## Project Overview
OpenWidget is a React component library focused on accessibility
and customization. We support React 17+ and provide full
TypeScript definitions.

## Before Contributing

### Read First
- [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md)
- [CONTRIBUTING.md](./CONTRIBUTING.md)
- [ARCHITECTURE.md](./docs/ARCHITECTURE.md)

### Environment Setup
```bash
git clone https://github.com/openwidget/openwidget
cd openwidget
pnpm install
pnpm dev        # Start Storybook
pnpm test       # Run tests
```

## Contribution Rules

### Commit Convention
We use conventional commits:
- `feat(Button): add loading state`
- `fix(Modal): resolve focus trap issue`
- `docs(README): update installation guide`
- `test(Select): add keyboard navigation tests`

### PR Requirements
- [ ] Tests pass locally
- [ ] Storybook stories added/updated
- [ ] Documentation updated
- [ ] Changeset created (`pnpm changeset`)
- [ ] Accessibility tested

### Component Checklist
New components must have:
- [ ] TypeScript interface with JSDoc
- [ ] Unit tests (90%+ coverage)
- [ ] Storybook stories
- [ ] README in component folder
- [ ] Accessibility attributes (ARIA)
- [ ] Keyboard navigation support

## Coding Standards

### Accessibility First
All components must:
- Support keyboard navigation
- Include proper ARIA labels
- Meet WCAG 2.1 AA standards
- Work with screen readers

### Styling
- Use CSS Modules with TypeScript
- Support custom className
- Support style prop
- Use CSS custom properties for theming

### Testing
- Test user behavior, not implementation
- Use @testing-library/react
- Test accessibility with jest-axe
```

### Example 4: Enterprise Project with Compliance Requirements

```markdown
# FinanceTracker - Enterprise Financial Application

## Compliance Notice
This application handles financial data and must comply with:
- SOC 2 Type II
- GDPR
- PCI-DSS (for payment processing)

## Security Rules

### Never Commit
- API keys, tokens, or credentials
- Customer personal data
- Internal IP addresses or hostnames
- Debug logs containing user data

### Required for All Code
- Input validation on all endpoints
- Parameterized queries (no SQL concatenation)
- Encryption for data at rest and in transit
- Audit logging for data access

### Authentication
- Use Auth0 for all authentication
- JWT tokens with 15-minute expiry
- Refresh tokens stored in httpOnly cookies
- MFA required for admin operations

## Data Handling

### PII Fields
The following are PII and require special handling:
- Email addresses
- Phone numbers
- Social security numbers
- Bank account numbers
- Credit card numbers

### Encryption Requirements
```typescript
// Always use the approved encryption utilities
import { encrypt, decrypt } from '@/security/crypto';

// Never store PII in plain text
const encryptedSSN = encrypt(ssn);

// Never log PII
logger.info('User updated', { userId }); // Good
logger.info('User updated', { ssn });    // VIOLATION
```

### Audit Logging
All data access must be logged:
```typescript
import { auditLog } from '@/security/audit';

async function getUserData(userId: string, requesterId: string) {
  auditLog({
    action: 'USER_DATA_ACCESS',
    targetId: userId,
    requesterId,
    timestamp: new Date().toISOString()
  });
  // ... fetch data
}
```

## Code Review Requirements
- Security review required for auth changes
- Compliance review for data handling changes
- Performance review for database changes
- Minimum 2 approvals for production code
```

### Example 5: Multi-Language Project (Frontend + Backend)

```markdown
# FullStackApp - TypeScript Frontend + Python Backend

## Language-Specific Rules

### TypeScript (Frontend)
@import .claude/rules/typescript.md
@import .claude/rules/react.md

### Python (Backend)
@import .claude/rules/python.md
@import .claude/rules/fastapi.md

## Project Structure
```
fullstack-app/
├── frontend/          # React + TypeScript
│   ├── src/
│   └── package.json
├── backend/           # Python + FastAPI
│   ├── app/
│   ├── tests/
│   └── pyproject.toml
└── shared/
    └── api-schema/    # OpenAPI definitions
```

## API Contract
The frontend and backend share an OpenAPI schema.

### Generating Types
```bash
# Generate TypeScript types from OpenAPI
cd frontend && pnpm generate:api-types

# Validate backend against schema
cd backend && pytest tests/test_api_schema.py
```

### Schema Location
- Schema: `shared/api-schema/openapi.yaml`
- Frontend types: `frontend/src/api/generated/`
- Backend validators: Auto-generated by FastAPI

## Cross-Language Conventions

### Naming Translation
| Concept | TypeScript | Python |
|---------|------------|--------|
| Variables | camelCase | snake_case |
| Types/Classes | PascalCase | PascalCase |
| Constants | SCREAMING_SNAKE | SCREAMING_SNAKE |
| Files | kebab-case.ts | snake_case.py |

### Data Transfer
- Use ISO 8601 for dates
- Use camelCase in JSON (frontend serializes)
- Use snake_case in Python (Pydantic handles conversion)

## Development Workflow
```bash
# Start both services
docker-compose up

# Frontend only
cd frontend && pnpm dev

# Backend only
cd backend && uvicorn app.main:app --reload

# Run all tests
./scripts/test-all.sh
```
```

### Example 6: Microservices with Service-Specific Rules

```markdown
# MicroMart - E-commerce Microservices Platform

## Service Overview
```
┌─────────────────────────────────────────────────────────────┐
│                        API Gateway                          │
└─────────────────────────────────────────────────────────────┘
       │              │               │              │
       v              v               v              v
┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│    User     │ │   Product   │ │    Order    │ │   Payment   │
│   Service   │ │   Service   │ │   Service   │ │   Service   │
└─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘
       │              │               │              │
       v              v               v              v
┌─────────────────────────────────────────────────────────────┐
│                    Message Queue (RabbitMQ)                 │
└─────────────────────────────────────────────────────────────┘
```

## Service Rules
@import .claude/rules/services/user-service.md
@import .claude/rules/services/product-service.md
@import .claude/rules/services/order-service.md
@import .claude/rules/services/payment-service.md

## Inter-Service Communication

### Synchronous (HTTP)
- Use for queries (read operations)
- Timeout: 5 seconds
- Retry: 3 times with exponential backoff

### Asynchronous (Events)
- Use for commands (write operations)
- Event naming: `{service}.{entity}.{action}`
- Examples: `user.account.created`, `order.status.updated`

### Event Schema
```typescript
interface DomainEvent {
  eventId: string;           // UUID
  eventType: string;         // user.account.created
  aggregateId: string;       // Entity ID
  timestamp: string;         // ISO 8601
  version: number;           // Schema version
  payload: Record<string, unknown>;
}
```

## Service Independence Rules
- Each service owns its data
- No direct database access across services
- Use events for eventual consistency
- Each service has its own repository

## Local Development
```bash
# Start all services
docker-compose up -d

# Start specific service
docker-compose up user-service

# View logs
docker-compose logs -f order-service

# Run service tests
cd services/order-service && npm test
```
```

### Example 7: Personal Project with Learning Notes

```markdown
# My Rust Learning Project

## About This Project
I'm learning Rust by building a CLI task manager. This is a
personal learning project, so code quality will improve over time.

## Current Learning Goals
- [ ] Understand ownership and borrowing
- [ ] Practice error handling with Result
- [ ] Learn async Rust with Tokio
- [ ] Explore the Clap library for CLI

## Things I'm Still Learning

### Ownership
I sometimes get confused about:
- When to use `&` vs `&mut` vs ownership transfer
- String vs &str decisions

**Claude, please explain ownership concepts when relevant.**

### Error Handling
I'm not sure when to use:
- `unwrap()` vs `?` operator
- Custom error types vs anyhow

**Claude, help me use proper error handling patterns.**

## Current Architecture Understanding
```
src/
├── main.rs        # Entry point, CLI setup
├── commands/      # CLI command handlers
├── models/        # Data structures
├── storage/       # File-based persistence
└── utils/         # Helper functions
```

## Notes to Self
- Always run `cargo clippy` before committing
- Write tests first (trying TDD)
- Don't fight the borrow checker - it's teaching me

## Questions to Explore
- How does async compare to threads?
- When should I use generics vs trait objects?
- What's the idiomatic way to handle optional values?

## Resources I'm Using
- The Rust Book: https://doc.rust-lang.org/book/
- Rust by Example: https://doc.rust-lang.org/rust-by-example/
- Jon Gjengset's videos: https://www.youtube.com/c/JonGjengset

## Progress Log
### Week 1
- Basic CLI structure working
- File I/O implemented
- Struggling with lifetimes

### Week 2
- Lifetimes clicking more
- Added async file operations
- Started writing tests
```

---

## Hierarchy Precedence

### Understanding Priority Levels

The 4-level hierarchy determines which rules take priority when there are conflicts:

| Level | Location | Priority | Scope | Override Behavior |
|-------|----------|----------|-------|-------------------|
| Enterprise | `/etc/claude/CLAUDE.md` | Lowest | Organization-wide | **Cannot be overridden** |
| Project | `./CLAUDE.md` | Medium | Single project | Adds to enterprise |
| Rules | `.claude/rules/*.md` | High | Project-specific | Adds to project |
| User | `~/.claude/CLAUDE.md` | Highest | Personal | Personal preferences win |

### How Conflicts Resolve

```
┌─────────────────────────────────────────────────────────────┐
│                  Conflict Resolution Flow                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Enterprise says: "Use 4-space indentation"                 │
│       │                                                     │
│       v                                                     │
│  Project says: "Use 2-space indentation"                    │
│       │                                                     │
│       v                                                     │
│  RESULT: 4-space wins (Enterprise cannot be overridden)     │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Project says: "Use camelCase naming"                       │
│       │                                                     │
│       v                                                     │
│  Rules say: "Use PascalCase for components"                 │
│       │                                                     │
│       v                                                     │
│  User says: "I prefer snake_case"                           │
│       │                                                     │
│       v                                                     │
│  RESULT: snake_case for user's preference (non-enterprise)  │
│          PascalCase for component-specific rules            │
│          camelCase as general fallback                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Non-Conflicting Rules Stack

When rules don't conflict, they stack additively:

```markdown
# Enterprise: Security requirements
- Must validate input
- Must use parameterized queries

# Project: Coding standards
- Use TypeScript strict mode
- Maximum 50 lines per function

# Rules: Testing requirements
- 80% coverage required
- Use React Testing Library

# User: Preferences
- Show verbose error messages
- Include inline comments

# RESULT: All rules apply together
```

### Best Practices for Hierarchy

**1. Don't Fight Enterprise Rules:**
```markdown
# Bad: Trying to override in project CLAUDE.md
## Indentation
Use 2-space indentation.  # Won't work if enterprise says 4

# Good: Accept enterprise rules, add project specifics
## Formatting
Following enterprise 4-space indentation.
Additional: Use trailing commas in multiline.
```

**2. Use Rules for Specificity:**
```markdown
# Project CLAUDE.md - General rules
Use TypeScript for all code.

# .claude/rules/legacy.md - Specific override
@path: src/legacy/**
Legacy code uses JavaScript. Do not convert to TypeScript.
```

**3. Keep User Preferences Personal:**
```markdown
# Good user preferences
- Communication style
- Learning notes
- Personal shortcuts

# Bad user preferences (should be in project)
- Project architecture
- Team coding standards
- Business logic rules
```

---

## Troubleshooting

### CLAUDE.md Not Being Read

**Symptoms:**
- Claude doesn't follow your CLAUDE.md rules
- Claude asks about things documented in CLAUDE.md
- Project-specific patterns not being used

**Solutions:**

1. **Check File Location**
   ```bash
   # CLAUDE.md must be in project root
   ls -la CLAUDE.md

   # Should see the file in your project root
   # If not, create it there
   ```

2. **Check File Name (Case Sensitive)**
   ```bash
   # Must be exactly "CLAUDE.md" (uppercase)
   # Wrong: claude.md, Claude.md, CLAUDE.MD
   ```

3. **Check File Encoding**
   ```bash
   # Must be UTF-8
   file CLAUDE.md
   # Should show: UTF-8 Unicode text
   ```

4. **Restart Claude Code Session**
   ```bash
   # CLAUDE.md is loaded at session start
   # Exit and restart Claude Code
   claude  # Start fresh session
   ```

5. **Check for Syntax Errors**
   ```markdown
   # Ensure proper markdown formatting
   # No unclosed code blocks
   # No invalid @import paths
   ```

### @import Not Working

**Symptoms:**
- Imported rules not being applied
- No error but imports seem ignored
- Partial imports working

**Solutions:**

1. **Check Path is Correct**
   ```markdown
   # Paths are relative to CLAUDE.md location
   @import .claude/rules/typescript.md    # Good
   @import ../other-project/rules.md      # May not work
   ```

2. **Check File Exists**
   ```bash
   # Verify the imported file exists
   ls -la .claude/rules/typescript.md
   ```

3. **Check File is Valid Markdown**
   ```bash
   # Open and verify content
   cat .claude/rules/typescript.md
   ```

4. **Avoid Circular Imports**
   ```markdown
   # Bad: a.md imports b.md imports a.md
   # Check your import chains
   ```

5. **Check Import Syntax**
   ```markdown
   # Correct
   @import .claude/rules/test.md

   # Wrong
   @import: .claude/rules/test.md
   @import(.claude/rules/test.md)
   import .claude/rules/test.md
   ```

### Rules Not Applying

**Symptoms:**
- Claude ignores specific rules
- Path-specific rules not working
- Inconsistent rule application

**Solutions:**

1. **Verify @path Syntax**
   ```markdown
   # Correct
   @path: src/components/**

   # Wrong
   @path src/components/**
   @path: "src/components/**"
   path: src/components/**
   ```

2. **Check Path Matching**
   ```markdown
   # For file: src/components/Button/index.tsx

   @path: src/components/**     # Matches
   @path: src/components/       # Doesn't match (no wildcard)
   @path: components/**         # Doesn't match (missing src/)
   ```

3. **Confirm Working Directory**
   ```bash
   # Claude needs to be in correct directory
   pwd  # Verify you're in project root
   ```

4. **Check Rule Conflicts**
   ```markdown
   # If enterprise rule conflicts, enterprise wins
   # Check /etc/claude/CLAUDE.md if applicable
   ```

### Too Much Context

**Symptoms:**
- Claude seems slow
- Claude mentions context limits
- Important rules being ignored (pushed out)

**Solutions:**

1. **Trim CLAUDE.md Size**
   ```markdown
   # Aim for under 500 lines total
   # Remove redundant information
   # Be concise
   ```

2. **Use @import for Organization**
   ```markdown
   # Instead of 1000-line CLAUDE.md
   # Use focused imports

   @import .claude/rules/essential.md      # 100 lines
   @import .claude/rules/typescript.md     # 50 lines
   # Load only what's needed
   ```

3. **Use Path-Specific Rules**
   ```markdown
   # Rules only load when path matches
   @path: src/frontend/**

   # These 200 lines only load for frontend work
   ```

4. **Remove Obvious Information**
   ```markdown
   # Bad: Stating the obvious
   "Use proper variable names"
   "Write clean code"

   # Good: Project-specific patterns
   "Variables must use the I-prefix for interfaces"
   "Services follow the Repository pattern"
   ```

### Conflicting Rules

**Symptoms:**
- Claude seems confused
- Inconsistent code style
- Claude asks for clarification frequently

**Solutions:**

1. **Identify Conflicts**
   ```markdown
   # Check for contradictions
   # Project: "Use 2-space indentation"
   # Rules: "Use 4-space indentation"
   ```

2. **Establish Clear Priority**
   ```markdown
   # In project CLAUDE.md
   ## Rule Priority
   When rules conflict, follow this order:
   1. Security rules (always highest)
   2. Language-specific rules
   3. General style preferences
   ```

3. **Make Rules Specific**
   ```markdown
   # Instead of conflicting general rules
   # Use path-specific rules

   @path: src/backend/**
   Use 4-space indentation.

   @path: src/frontend/**
   Use 2-space indentation.
   ```

4. **Add Exceptions Explicitly**
   ```markdown
   ## Indentation
   Use 2-space indentation everywhere.

   **Exception**: Python files use 4-space (PEP 8).
   **Exception**: Makefiles use tabs.
   ```

---

## Quick Reference Cheat Sheet

### File Locations

```
Enterprise:  /etc/claude/CLAUDE.md (Linux/macOS)
Project:     ./CLAUDE.md (project root)
Rules:       ./.claude/rules/*.md
User:        ~/.claude/CLAUDE.md
```

### Essential Syntax

```markdown
# Headers
# H1 - Main title
## H2 - Sections
### H3 - Subsections

# Code Blocks
```language
code here
```

# @import (include other files)
@import .claude/rules/filename.md

# @path (conditional rules)
@path: src/components/**
@path: **/*.test.ts
```

### Common Patterns

```markdown
# Tech Stack Table
| Layer | Technology |
|-------|------------|
| Frontend | React |
| Backend | Node.js |
| Database | PostgreSQL |

# Project Structure
```
src/
├── components/
├── services/
└── utils/
```

# Code Examples
```typescript
// Example code here
```

# Commands
```bash
npm run dev
npm test
```
```

### CLAUDE.md Skeleton

```markdown
# Project Name

## Overview
[One paragraph description]

## Tech Stack
- Frontend: [...]
- Backend: [...]
- Database: [...]

## Project Structure
[Directory tree]

## Coding Standards
[Key rules]

## Important Files
[Critical paths]

## Common Commands
[Dev commands]

## Imports
@import .claude/rules/[...].md
```

### Path Pattern Reference

| Pattern | Matches |
|---------|---------|
| `src/**` | All files under src/ |
| `src/*.ts` | TypeScript files directly in src/ |
| `**/*.test.ts` | All test files anywhere |
| `src/components/**` | All component files |
| `!src/legacy/**` | Exclude legacy directory |

### Hierarchy Quick Reference

```
Priority (High to Low):
1. User (~/.claude/CLAUDE.md) - Personal
2. Rules (.claude/rules/*.md) - Modular
3. Project (./CLAUDE.md) - Team
4. Enterprise (/etc/claude/CLAUDE.md) - Org

Note: Enterprise CANNOT be overridden
```

---

## Best Practices

### Keep CLAUDE.md Focused and Concise

**Do:**
- Include information Claude needs frequently
- Write clear, actionable rules
- Use examples liberally
- Update as project evolves

**Don't:**
- Include everything about your project
- Write essay-length explanations
- Duplicate information from code
- Leave outdated information

**Example of Focused Content:**

```markdown
# Good: Focused and actionable
## API Error Handling
All API endpoints must:
1. Return appropriate HTTP status codes
2. Include error body: `{ error: string, code: string }`
3. Log errors with correlation ID

Example:
```typescript
throw new ApiError(404, 'USER_NOT_FOUND', 'User does not exist');
```
```

### Use Rules for Modular Organization

**Organize by Concern:**
```
.claude/rules/
├── core/
│   ├── typescript.md
│   └── testing.md
├── frontend/
│   ├── react.md
│   └── styling.md
├── backend/
│   ├── api.md
│   └── database.md
└── process/
    ├── git.md
    └── review.md
```

**Benefits:**
- Easier to find and update rules
- Different team members can own different rules
- Cleaner version control history
- Path-specific loading reduces context

### Update CLAUDE.md as Project Evolves

**Schedule Regular Reviews:**
- Sprint planning: Check if anything changed
- After major features: Update architecture
- After incidents: Add learned lessons
- Quarterly: Full review and cleanup

**Track Changes:**
```markdown
## Changelog

### 2026-01-15
- Added new authentication rules
- Deprecated old payment integration
- Updated testing requirements to 80%

### 2025-12-01
- Initial CLAUDE.md created
- Basic project structure documented
```

### Team Review of CLAUDE.md Changes

**Treat CLAUDE.md Like Code:**
- PR required for changes
- Team review process
- Test that rules work as expected
- Document reasoning for rules

**Review Checklist:**
```markdown
## CLAUDE.md Change Review

- [ ] Rules are clear and unambiguous
- [ ] Examples are provided where helpful
- [ ] No conflicts with existing rules
- [ ] Path patterns are correct
- [ ] Team agrees on changes
- [ ] No sensitive information included
```

### Additional Best Practices

**1. Version Your CLAUDE.md:**
```markdown
# Project Name
> CLAUDE.md version: 2.3
> Last updated: 2026-01-15
```

**2. Include "Why" Not Just "What":**
```markdown
# Good
## Error Handling
Use Result types instead of exceptions.

> Why: We had production incidents from uncaught exceptions.
> This forces explicit error handling at compile time.
```

**3. Provide Negative Examples:**
```markdown
## Naming

```typescript
// Good
const userAccountBalance = getBalance(userId);

// Bad - avoid abbreviations
const usrAcctBal = getBal(uid);
```
```

**4. Link to External Docs:**
```markdown
## Architecture
For detailed architecture decisions, see:
- [ADR-001: Database Choice](docs/adr/001-database.md)
- [ADR-002: Auth System](docs/adr/002-auth.md)
```

---

## Exercises

### Exercise 1: Create Your First CLAUDE.md

**Objective:** Create a basic CLAUDE.md for a project.

**Steps:**
1. Choose an existing project (or create a simple one)
2. Create a CLAUDE.md with:
   - Project overview
   - Tech stack
   - 3-5 coding standards
   - Project structure
   - 2-3 common commands

**Validation:** Start a Claude Code session and ask "What is this project?" - Claude should answer using your CLAUDE.md.

### Exercise 2: Modularize with Rules

**Objective:** Break a large CLAUDE.md into modular rules.

**Steps:**
1. Create `.claude/rules/` directory
2. Create at least 3 rule files:
   - `typescript.md` - Language rules
   - `testing.md` - Testing requirements
   - `git.md` - Git workflow
3. Use `@import` in main CLAUDE.md
4. Test that rules still apply

**Validation:** All rules from separate files should be followed by Claude.

### Exercise 3: Path-Specific Rules

**Objective:** Create rules that only apply to specific directories.

**Steps:**
1. Create `.claude/rules/frontend.md`
2. Add `@path: src/frontend/**` at the top
3. Add frontend-specific rules
4. Create `.claude/rules/backend.md` with different rules
5. Test both contexts

**Validation:** Ask Claude to create a component (should follow frontend rules) then an API endpoint (should follow backend rules).

### Exercise 4: User Preferences

**Objective:** Set up personal CLAUDE.md preferences.

**Steps:**
1. Create `~/.claude/CLAUDE.md`
2. Add your communication preferences
3. Add your personal coding style notes
4. Add any learning notes

**Validation:** Claude should adjust its responses to match your preferences across different projects.

### Exercise 5: Full Stack Setup

**Objective:** Create a complete CLAUDE.md system for a full-stack project.

**Steps:**
1. Main CLAUDE.md with project overview
2. Rules for frontend (React/TypeScript)
3. Rules for backend (Node/Python)
4. Rules for database
5. Rules for testing
6. Path-specific rules for each area
7. Proper @import structure

**Validation:** Claude should seamlessly switch contexts when working in different parts of the codebase.

---

## Summary

### What You Learned

In this tutorial, you mastered:

1. **CLAUDE.md Fundamentals**
   - Purpose as persistent project memory
   - Automatic loading at session start
   - How Claude uses the content

2. **The 4-Level Hierarchy**
   - Enterprise (organization-wide, cannot override)
   - Project (team-shared, in version control)
   - Rules (modular, path-specific)
   - User (personal preferences)

3. **Creating Effective Content**
   - Essential sections to include
   - Writing clear, actionable rules
   - Using examples and code blocks

4. **Advanced Features**
   - @import for modular organization
   - @path for conditional rules
   - Hierarchy precedence

5. **Best Practices**
   - Keep content focused
   - Modularize with rules
   - Update regularly
   - Team review process

### Key Takeaways

```
┌─────────────────────────────────────────────────────────────┐
│                    CLAUDE.md Key Points                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. CLAUDE.md = Project Memory                              │
│     Write once, Claude remembers always                     │
│                                                             │
│  2. 4 Levels, Clear Hierarchy                               │
│     Enterprise → Project → Rules → User                     │
│                                                             │
│  3. Modularize with .claude/rules/                          │
│     Organized, maintainable, path-specific                  │
│                                                             │
│  4. @import and @path                                       │
│     Powerful tools for organization                         │
│                                                             │
│  5. Keep It Updated                                         │
│     Living documentation, not set-and-forget                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Next Steps

1. **Create your project's CLAUDE.md** - Start with the basic template
2. **Add rules incrementally** - Don't try to document everything at once
3. **Review and refine** - Improve based on how Claude responds
4. **Share with team** - Get feedback and alignment
5. **Explore advanced patterns** - Path-specific rules, complex imports

### Additional Resources

- [Claude Code Documentation](https://docs.anthropic.com/claude-code)
- [Markdown Guide](https://www.markdownguide.org/)
- [Glob Patterns Reference](https://github.com/isaacs/minimatch)

---

## Appendix: Complete Template Collection

### Minimal CLAUDE.md

```markdown
# Project Name

## Overview
[One sentence description]

## Commands
- `npm run dev` - Start development
- `npm test` - Run tests
```

### Standard CLAUDE.md

```markdown
# Project Name

## Overview
[Project description, purpose, users]

## Tech Stack
[List technologies]

## Structure
[Directory tree]

## Standards
[5-10 key rules]

## Commands
[Common commands]

## Rules
@import .claude/rules/[language].md
@import .claude/rules/testing.md
```

### Complete CLAUDE.md

```markdown
# Project Name
> Version: 1.0 | Updated: YYYY-MM-DD

## Overview
[Comprehensive description]

## Tech Stack
[Detailed technology list with versions]

## Architecture
[Diagrams and patterns]

## Structure
[Full directory tree with descriptions]

## Standards
[Comprehensive coding standards]

## Key Files
[Critical paths with descriptions]

## Commands
[All common commands with explanations]

## Workflow
[Development, review, deploy process]

## Rules
@import .claude/rules/[...].md

## Team
[Contacts and responsibilities]

## Changelog
[Recent updates]
```

---

**End of Tutorial 21: CLAUDE.md Mastery**

*You now have the knowledge to create powerful, organized CLAUDE.md configurations that will make your Claude Code experience significantly more productive.*
