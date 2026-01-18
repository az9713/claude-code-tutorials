# Tutorial 09: Claude Code Settings and Configuration

## Complete Guide to Customizing Your Claude Code Environment

This tutorial provides a comprehensive reference for configuring Claude Code. From basic user preferences to advanced enterprise setups, you'll learn how to customize every aspect of your Claude Code experience.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Configuration Locations](#2-configuration-locations)
3. [Complete Settings Reference](#3-complete-settings-reference)
4. [CLAUDE.md Configuration](#4-claudemd-configuration)
5. [Hand-Holding User Journey](#5-hand-holding-user-journey)
6. [Real-World Examples](#6-real-world-examples)
7. [.claudeignore Configuration](#7-claudeignore-configuration)
8. [Permission Configuration](#8-permission-configuration)
9. [Model Configuration](#9-model-configuration)
10. [Environment Variables](#10-environment-variables)
11. [Sharing Configurations](#11-sharing-configurations)
12. [Troubleshooting Configuration](#12-troubleshooting-configuration)
13. [Pros and Cons](#13-pros-and-cons)
14. [Quick Reference](#14-quick-reference)

---

## 1. Introduction

### Configuration Hierarchy in Claude Code

Claude Code uses a layered configuration system where settings can be defined at multiple levels. Understanding this hierarchy is essential for effective configuration management.

```
Priority (highest to lowest):
┌─────────────────────────────────────┐
│  1. Command-line arguments          │  --model, --permission-mode
├─────────────────────────────────────┤
│  2. Environment variables           │  ANTHROPIC_API_KEY, CLAUDE_CODE_*
├─────────────────────────────────────┤
│  3. Workspace settings              │  .claude/settings.local.json
├─────────────────────────────────────┤
│  4. Project settings                │  .claude/settings.json
├─────────────────────────────────────┤
│  5. User settings                   │  ~/.claude/settings.json
├─────────────────────────────────────┤
│  6. Default settings                │  Built into Claude Code
└─────────────────────────────────────┘
```

### User vs Project vs Workspace Settings

| Setting Level | Scope | Tracked in Git | Use Case |
|---------------|-------|----------------|----------|
| **User** | All projects | No | Personal preferences, API keys |
| **Project** | Single project | Yes | Team standards, project context |
| **Workspace** | Local override | No | Machine-specific paths, local secrets |

### Why Configuration Matters

Proper configuration provides:

1. **Consistency** - Same behavior across sessions and machines
2. **Security** - Control what Claude Code can access and modify
3. **Efficiency** - Pre-configure context so Claude understands your project
4. **Collaboration** - Share standards across team members
5. **Safety** - Prevent accidental destructive operations

---

## 2. Configuration Locations

### ~/.claude/ Directory Structure

The user configuration directory contains global settings:

```
~/.claude/
├── settings.json           # User-level settings
├── credentials.json        # API key storage (auto-generated)
├── statsig/               # Feature flags cache
├── logs/                  # Debug logs
│   └── claude-code-*.log
├── projects/              # Project session data
│   └── <project-hash>/
│       ├── context.json
│       └── history/
├── mcp/                   # MCP server data
│   └── servers/
└── cache/                 # Response cache
```

**Platform-specific locations:**

| Platform | Location |
|----------|----------|
| macOS | `~/.claude/` or `~/Library/Application Support/claude-code/` |
| Linux | `~/.claude/` or `~/.config/claude-code/` |
| Windows | `%USERPROFILE%\.claude\` or `%APPDATA%\claude-code\` |

### .claude/ Project Directory

Project-level configuration lives in the repository:

```
your-project/
├── .claude/
│   ├── settings.json       # Project settings (commit this)
│   ├── settings.local.json # Local overrides (gitignore this)
│   ├── commands/           # Custom slash commands
│   │   ├── build.md
│   │   └── test.md
│   └── mcp.json           # MCP server configuration
├── CLAUDE.md              # Project context document
├── .claudeignore          # Files to exclude from context
└── ... (your project files)
```

### settings.json Locations

```bash
# User settings (global)
~/.claude/settings.json

# Project settings (shared with team)
./.claude/settings.json

# Workspace settings (local machine only)
./.claude/settings.local.json
```

### Environment Variables

Environment variables override file-based settings:

```bash
# Essential
export ANTHROPIC_API_KEY="YOUR_API_KEY"

# Configuration
export CLAUDE_CODE_CONFIG_DIR="~/.claude"
export CLAUDE_CODE_PERMISSION_MODE="default"

# Networking
export HTTP_PROXY="http://proxy.company.com:8080"
export HTTPS_PROXY="http://proxy.company.com:8080"
export NO_PROXY="localhost,127.0.0.1,.internal.company.com"
```

### Command-Line Arguments

Arguments have highest priority:

```bash
# Model selection
claude --model claude-sonnet-4-20250514

# Permission mode
claude --permission-mode plan

# Working directory
claude --cwd /path/to/project

# Configuration file
claude --config /path/to/custom-settings.json

# Verbose output
claude --verbose

# Non-interactive mode
claude -p "your prompt here"
```

---

## 3. Complete Settings Reference

### settings.json Full Schema

```json
{
  "$schema": "https://claude.ai/schemas/claude-code-settings.json",

  "model": {
    "default": "claude-sonnet-4-20250514",
    "temperature": 0,
    "maxTokens": 16384,
    "extendedThinking": false,
    "thinkingBudget": 10000
  },

  "permissions": {
    "mode": "default",
    "allowedTools": ["Read", "Glob", "Grep", "Bash", "Write", "Edit"],
    "deniedTools": [],
    "allowedCommands": ["npm", "npx", "node", "git", "python"],
    "deniedCommands": ["rm -rf /", "sudo rm", "format"],
    "allowFileWrite": true,
    "allowFileDelete": false,
    "allowNetworkAccess": true,
    "requireConfirmation": ["Write", "Bash"],
    "trustWorkspace": false
  },

  "ui": {
    "theme": "auto",
    "colors": {
      "primary": "#7C3AED",
      "success": "#10B981",
      "warning": "#F59E0B",
      "error": "#EF4444"
    },
    "showTokenUsage": true,
    "showCost": true,
    "compactMode": false,
    "verbose": false,
    "markdown": true
  },

  "context": {
    "maxFileSize": 1048576,
    "maxTotalSize": 10485760,
    "excludePatterns": ["node_modules/**", "*.log", ".git/**"],
    "includePatterns": ["src/**", "lib/**", "*.md"],
    "autoloadFiles": ["CLAUDE.md", "README.md"],
    "respectGitignore": true
  },

  "tools": {
    "bash": {
      "shell": "/bin/bash",
      "timeout": 120000,
      "allowBackground": true
    },
    "edit": {
      "createBackups": true,
      "backupDir": ".claude/backups"
    },
    "webFetch": {
      "timeout": 30000,
      "maxSize": 5242880,
      "userAgent": "Claude-Code/1.0"
    }
  },

  "mcp": {
    "servers": {
      "filesystem": {
        "command": "npx",
        "args": ["-y", "@anthropic/mcp-server-filesystem", "/allowed/path"],
        "enabled": true
      }
    },
    "timeout": 30000,
    "retries": 3
  },

  "history": {
    "maxSessions": 100,
    "maxMessagesPerSession": 1000,
    "autoSave": true,
    "saveInterval": 30000
  },

  "telemetry": {
    "enabled": true,
    "shareUsageData": false,
    "shareCrashReports": true
  },

  "advanced": {
    "apiBaseUrl": "https://api.anthropic.com",
    "timeout": 300000,
    "retries": 3,
    "retryDelay": 1000,
    "cacheResponses": true,
    "cacheTTL": 3600000
  }
}
```

### Model Settings

```json
{
  "model": {
    "default": "claude-sonnet-4-20250514",
    "temperature": 0,
    "maxTokens": 16384,
    "extendedThinking": false,
    "thinkingBudget": 10000
  }
}
```

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `default` | string | `claude-sonnet-4-20250514` | Default model for conversations |
| `temperature` | number | 0 | Randomness (0=deterministic, 1=creative) |
| `maxTokens` | number | 16384 | Maximum response tokens |
| `extendedThinking` | boolean | false | Enable extended thinking mode |
| `thinkingBudget` | number | 10000 | Max thinking tokens when enabled |

### Permission Settings

```json
{
  "permissions": {
    "mode": "default",
    "allowedTools": ["Read", "Glob", "Grep"],
    "deniedTools": ["Bash"],
    "allowedCommands": ["npm test", "git status"],
    "deniedCommands": ["rm -rf"],
    "requireConfirmation": ["Write", "Edit"]
  }
}
```

| Setting | Type | Description |
|---------|------|-------------|
| `mode` | string | Permission mode: "default", "plan", "trust" |
| `allowedTools` | array | Tools Claude can use without asking |
| `deniedTools` | array | Tools Claude cannot use |
| `allowedCommands` | array | Shell commands pre-approved |
| `deniedCommands` | array | Shell commands always blocked |
| `requireConfirmation` | array | Tools needing explicit approval |

### UI Settings

```json
{
  "ui": {
    "theme": "dark",
    "showTokenUsage": true,
    "showCost": true,
    "compactMode": false,
    "verbose": false
  }
}
```

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `theme` | string | "auto" | Color theme: "light", "dark", "auto" |
| `showTokenUsage` | boolean | true | Display token counts |
| `showCost` | boolean | true | Display estimated costs |
| `compactMode` | boolean | false | Reduce output verbosity |
| `verbose` | boolean | false | Show detailed debug info |

### Context Settings

```json
{
  "context": {
    "maxFileSize": 1048576,
    "excludePatterns": ["node_modules/**", "*.log"],
    "includePatterns": ["src/**"],
    "autoloadFiles": ["CLAUDE.md"],
    "respectGitignore": true
  }
}
```

| Setting | Type | Description |
|---------|------|-------------|
| `maxFileSize` | number | Maximum file size to read (bytes) |
| `excludePatterns` | array | Glob patterns to always exclude |
| `includePatterns` | array | Glob patterns to prioritize |
| `autoloadFiles` | array | Files to load into context automatically |
| `respectGitignore` | boolean | Honor .gitignore patterns |

### Tool Settings

```json
{
  "tools": {
    "bash": {
      "shell": "/bin/bash",
      "timeout": 120000,
      "allowBackground": true,
      "env": {
        "NODE_ENV": "development"
      }
    },
    "edit": {
      "createBackups": true,
      "backupDir": ".claude/backups"
    }
  }
}
```

### MCP Settings

```json
{
  "mcp": {
    "servers": {
      "github": {
        "command": "npx",
        "args": ["-y", "@anthropic/mcp-server-github"],
        "env": {
          "GITHUB_TOKEN": "${GITHUB_TOKEN}"
        },
        "enabled": true
      }
    },
    "timeout": 30000,
    "retries": 3
  }
}
```

---

## 4. CLAUDE.md Configuration

### Purpose and Importance

CLAUDE.md is the single most important configuration file for Claude Code. It provides persistent context that Claude reads at the start of every session, helping it understand:

- Project structure and architecture
- Coding conventions and standards
- Important files and their purposes
- Team practices and workflows
- Common commands and patterns

### File Location

Claude Code searches for CLAUDE.md in this order:

```
1. ./CLAUDE.md           (project root)
2. ./.claude/CLAUDE.md   (claude config directory)
3. ./docs/CLAUDE.md      (documentation directory)
4. ../CLAUDE.md          (parent directory, for monorepos)
```

### Content Structure

```markdown
# Project Name

## Overview
Brief description of what this project does and its purpose.

## Architecture
High-level architecture description, key components, and data flow.

## Directory Structure
```
src/
├── components/    # React components
├── hooks/         # Custom React hooks
├── services/      # API service layer
├── utils/         # Utility functions
└── types/         # TypeScript type definitions
```

## Tech Stack
- Frontend: React 18, TypeScript 5
- Backend: Node.js 20, Express 4
- Database: PostgreSQL 15
- Testing: Jest, React Testing Library

## Coding Standards
- Use functional components with hooks
- Prefer named exports over default exports
- Use TypeScript strict mode
- Write tests for all business logic

## Common Commands
- `npm run dev` - Start development server
- `npm run test` - Run test suite
- `npm run build` - Production build
- `npm run lint` - Lint code

## Important Files
- `src/config/index.ts` - Application configuration
- `src/services/api.ts` - API client singleton
- `src/types/index.ts` - Shared type definitions

## Current Work
- [ ] Implementing user authentication
- [ ] Adding dark mode support
- [ ] Performance optimization

## Notes for Claude
- Always run tests after making changes
- Use the existing error handling patterns in src/utils/errors.ts
- Check src/types/ before creating new type definitions
```

### Examples of Good CLAUDE.md Files

**Example 1: React Application**

```markdown
# MyApp - React E-commerce Platform

## Overview
Full-stack e-commerce platform with React frontend and Node.js backend.
Serves 10k+ daily active users.

## Critical Information
- NEVER modify files in `src/legacy/` - deprecated code awaiting removal
- All API calls MUST go through `src/services/api.ts`
- Authentication tokens are in Redux store, not localStorage

## Architecture
```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   React     │────▶│   Express   │────▶│  PostgreSQL │
│   Frontend  │     │   Backend   │     │   Database  │
└─────────────┘     └─────────────┘     └─────────────┘
        │                   │
        │                   ▼
        │           ┌─────────────┐
        └──────────▶│   Redis     │
                    │   Cache     │
                    └─────────────┘
```

## Conventions
1. Components: PascalCase (`UserProfile.tsx`)
2. Hooks: camelCase with "use" prefix (`useAuth.ts`)
3. Utils: camelCase (`formatCurrency.ts`)
4. Types: PascalCase with "I" prefix for interfaces (`IUser.ts`)

## Testing Requirements
- Unit tests required for all utils (Jest)
- Component tests required for user-facing features (RTL)
- E2E tests for checkout flow (Playwright)
- Minimum 80% coverage for new code

## Environment Setup
```bash
cp .env.example .env.local
npm install
npm run db:migrate
npm run dev
```

## API Patterns
All API calls follow this pattern:
```typescript
// src/services/api.ts
const response = await api.get<UserResponse>('/users/:id');
// Always returns { data, error, loading }
```
```

**Example 2: Python Data Science Project**

```markdown
# DataPipeline - ML Feature Engineering

## Overview
ETL pipeline for machine learning feature engineering.
Processes 100GB+ daily data from multiple sources.

## Environment
- Python 3.11+ required
- Uses Poetry for dependency management
- Virtual env: `.venv/`

## Project Structure
```
src/
├── extractors/     # Data source connectors
├── transformers/   # Feature engineering
├── loaders/        # Output writers
├── models/         # ML model definitions
└── utils/          # Shared utilities

tests/
├── unit/           # Fast unit tests
├── integration/    # Database tests
└── fixtures/       # Test data
```

## Data Flow
1. Extract: Pull from S3, PostgreSQL, APIs
2. Transform: Clean, normalize, feature engineer
3. Load: Write to feature store (Redis) and data warehouse (Snowflake)

## Running Locally
```bash
poetry install
poetry run pytest  # Run tests
poetry run python -m src.pipeline --config config/local.yaml
```

## Coding Standards
- Type hints required for all functions
- Docstrings in Google format
- Black formatter, isort imports
- No print statements (use logging)

## Common Issues
- If Snowflake connection fails, check VPN status
- Large dataframes should use chunked processing
- Always close database connections in finally blocks
```

---

## 5. Hand-Holding User Journey

### Step 1: Find Your Configuration Files

**On macOS/Linux:**

```bash
# Check if user config directory exists
ls -la ~/.claude/

# If it doesn't exist, Claude Code creates it on first run
claude --version

# Now check again
ls -la ~/.claude/
```

**On Windows:**

```powershell
# Check user config directory
dir $env:USERPROFILE\.claude\

# Or using Command Prompt
dir %USERPROFILE%\.claude\
```

**Expected output:**

```
~/.claude/
├── settings.json       # Your user settings
├── credentials.json    # API credentials
└── projects/          # Project session data
```

### Step 2: Modify User Settings

**Create or edit `~/.claude/settings.json`:**

```bash
# macOS/Linux
nano ~/.claude/settings.json

# Or use any editor
code ~/.claude/settings.json
```

**Start with these recommended settings:**

```json
{
  "$schema": "https://claude.ai/schemas/claude-code-settings.json",

  "model": {
    "default": "claude-sonnet-4-20250514",
    "temperature": 0
  },

  "ui": {
    "theme": "auto",
    "showTokenUsage": true,
    "showCost": true
  },

  "permissions": {
    "mode": "default",
    "requireConfirmation": ["Write", "Edit", "Bash"]
  }
}
```

### Step 3: Set Up Project Configuration

**In your project directory:**

```bash
# Create the .claude directory
mkdir -p .claude

# Create project settings
cat > .claude/settings.json << 'EOF'
{
  "model": {
    "default": "claude-sonnet-4-20250514"
  },
  "permissions": {
    "allowedCommands": ["npm", "npx", "node", "git", "pytest"]
  },
  "context": {
    "excludePatterns": [
      "node_modules/**",
      "dist/**",
      "coverage/**",
      "*.log"
    ]
  }
}
EOF

# Create local overrides (don't commit this)
cat > .claude/settings.local.json << 'EOF'
{
  "permissions": {
    "mode": "trust"
  }
}
EOF

# Add local settings to gitignore
echo ".claude/settings.local.json" >> .gitignore
```

### Step 4: Configure CLAUDE.md

**Create a comprehensive CLAUDE.md in your project root:**

```bash
cat > CLAUDE.md << 'EOF'
# My Project

## Overview
[One paragraph describing what this project does]

## Quick Start
```bash
npm install
npm run dev
```

## Project Structure
```
src/
├── components/    # UI components
├── services/      # Business logic
└── utils/         # Helper functions
```

## Coding Standards
- [List your coding conventions]
- [Testing requirements]
- [Documentation requirements]

## Important Files
- `src/config.ts` - Configuration
- `src/types/` - Type definitions

## Commands
- `npm run dev` - Development server
- `npm run test` - Run tests
- `npm run build` - Production build
EOF
```

### Step 5: Set Up Team Configurations

**Create shareable team configuration:**

```bash
# Create team settings directory
mkdir -p .claude/team

# Create shared settings
cat > .claude/team/settings.json << 'EOF'
{
  "permissions": {
    "allowedCommands": [
      "npm",
      "npx",
      "node",
      "git",
      "docker",
      "docker-compose"
    ],
    "deniedCommands": [
      "rm -rf /",
      "sudo rm -rf",
      "git push --force origin main"
    ]
  },
  "context": {
    "excludePatterns": [
      "node_modules/**",
      ".git/**",
      "dist/**",
      "build/**",
      "coverage/**",
      "*.log",
      "*.env",
      ".env*"
    ]
  }
}
EOF

# Create setup script for team members
cat > .claude/setup.sh << 'EOF'
#!/bin/bash
# Setup script for team Claude Code configuration

echo "Setting up Claude Code for this project..."

# Copy team settings to local config
if [ ! -f .claude/settings.local.json ]; then
    echo '{}' > .claude/settings.local.json
    echo "Created local settings file"
fi

echo "Done! Run 'claude' to start."
EOF
chmod +x .claude/setup.sh
```

---

## 6. Real-World Examples

### Example 1: Python Project Configuration

**Project: FastAPI Backend Service**

**.claude/settings.json:**

```json
{
  "model": {
    "default": "claude-sonnet-4-20250514",
    "temperature": 0
  },
  "permissions": {
    "mode": "default",
    "allowedCommands": [
      "python",
      "pip",
      "poetry",
      "pytest",
      "black",
      "isort",
      "mypy",
      "alembic",
      "uvicorn"
    ],
    "deniedCommands": [
      "rm -rf",
      "drop database",
      "truncate"
    ]
  },
  "context": {
    "excludePatterns": [
      "__pycache__/**",
      "*.pyc",
      ".venv/**",
      "venv/**",
      ".pytest_cache/**",
      ".mypy_cache/**",
      "htmlcov/**",
      "*.egg-info/**",
      ".eggs/**"
    ],
    "autoloadFiles": [
      "CLAUDE.md",
      "pyproject.toml"
    ]
  },
  "tools": {
    "bash": {
      "shell": "/bin/bash",
      "env": {
        "PYTHONPATH": "${workspaceFolder}/src",
        "ENVIRONMENT": "development"
      }
    }
  }
}
```

**CLAUDE.md:**

```markdown
# FastAPI User Service

## Overview
RESTful API service for user management. Handles authentication,
authorization, and user profile operations.

## Tech Stack
- Python 3.11
- FastAPI 0.100+
- PostgreSQL 15
- SQLAlchemy 2.0
- Pydantic v2

## Project Structure
```
src/
├── api/
│   ├── v1/
│   │   ├── endpoints/
│   │   └── dependencies.py
│   └── router.py
├── core/
│   ├── config.py
│   ├── security.py
│   └── exceptions.py
├── models/
│   └── user.py
├── schemas/
│   └── user.py
├── services/
│   └── user_service.py
└── db/
    ├── session.py
    └── migrations/
```

## Commands
```bash
# Development
poetry run uvicorn src.main:app --reload

# Testing
poetry run pytest -v
poetry run pytest --cov=src --cov-report=html

# Linting
poetry run black src tests
poetry run isort src tests
poetry run mypy src

# Database
poetry run alembic upgrade head
poetry run alembic revision --autogenerate -m "description"
```

## Coding Standards
- All endpoints must have type hints
- All public functions need docstrings
- Use dependency injection for services
- Return Pydantic models from endpoints
- Async functions for I/O operations

## API Patterns
```python
# Endpoint pattern
@router.get("/{user_id}", response_model=UserResponse)
async def get_user(
    user_id: int,
    db: AsyncSession = Depends(get_db),
    current_user: User = Depends(get_current_user),
) -> UserResponse:
    """Get user by ID."""
    return await user_service.get_user(db, user_id)
```

## Testing
- Unit tests in `tests/unit/`
- Integration tests in `tests/integration/`
- Use pytest fixtures from `tests/conftest.py`
- Mock external services, not the database
```

**.claudeignore:**

```gitignore
# Python
__pycache__/
*.py[cod]
.venv/
venv/
*.egg-info/

# Testing
.pytest_cache/
.coverage
htmlcov/

# IDE
.idea/
.vscode/

# Environment
.env
.env.local
*.local

# Database
*.db
*.sqlite3
```

---

### Example 2: TypeScript Monorepo Configuration

**Project: Full-Stack Application with Turborepo**

**.claude/settings.json:**

```json
{
  "model": {
    "default": "claude-sonnet-4-20250514"
  },
  "permissions": {
    "mode": "default",
    "allowedCommands": [
      "npm",
      "npx",
      "node",
      "turbo",
      "pnpm",
      "tsx",
      "tsc",
      "eslint",
      "prettier",
      "jest",
      "vitest",
      "playwright",
      "prisma",
      "docker",
      "docker-compose"
    ]
  },
  "context": {
    "excludePatterns": [
      "node_modules/**",
      ".turbo/**",
      "dist/**",
      ".next/**",
      "coverage/**",
      "*.tsbuildinfo",
      ".pnpm-store/**"
    ],
    "autoloadFiles": [
      "CLAUDE.md",
      "turbo.json",
      "package.json"
    ]
  },
  "mcp": {
    "servers": {
      "database": {
        "command": "npx",
        "args": ["-y", "@anthropic/mcp-server-postgres"],
        "env": {
          "DATABASE_URL": "${DATABASE_URL}"
        }
      }
    }
  }
}
```

**CLAUDE.md:**

```markdown
# Acme Platform Monorepo

## Overview
Full-stack SaaS platform with web app, mobile app, and shared packages.

## Monorepo Structure
```
apps/
├── web/              # Next.js frontend
├── api/              # Express backend
├── mobile/           # React Native app
└── admin/            # Admin dashboard

packages/
├── ui/               # Shared React components
├── config/           # Shared configs (ESLint, TS)
├── database/         # Prisma schema & client
├── types/            # Shared TypeScript types
└── utils/            # Shared utilities
```

## Quick Start
```bash
pnpm install
pnpm db:generate
pnpm dev
```

## Per-App Commands
```bash
# Run specific app
turbo run dev --filter=web
turbo run dev --filter=api

# Build specific app
turbo run build --filter=web

# Test specific package
turbo run test --filter=@acme/ui
```

## Package Relationships
```
apps/web ─────────┬──────▶ packages/ui
                  ├──────▶ packages/types
                  └──────▶ packages/utils

apps/api ─────────┬──────▶ packages/database
                  ├──────▶ packages/types
                  └──────▶ packages/utils
```

## Coding Standards
- Shared code goes in `packages/`
- App-specific code stays in `apps/`
- Import from packages: `import { Button } from '@acme/ui'`
- All exports must be typed
- Use barrel exports (index.ts)

## Database Changes
```bash
# Modify schema
edit packages/database/prisma/schema.prisma

# Generate client
pnpm db:generate

# Create migration
pnpm db:migrate

# Push to dev (no migration)
pnpm db:push
```

## Adding New Packages
1. Create directory in `packages/`
2. Add `package.json` with `@acme/` prefix
3. Configure in `turbo.json` if needed
4. Add to dependent apps' package.json
```

---

### Example 3: Enterprise Security Configuration

**For: Financial Services Company**

**.claude/settings.json:**

```json
{
  "model": {
    "default": "claude-sonnet-4-20250514",
    "temperature": 0
  },
  "permissions": {
    "mode": "default",
    "allowedTools": [
      "Read",
      "Glob",
      "Grep"
    ],
    "deniedTools": [
      "WebFetch",
      "WebSearch"
    ],
    "allowedCommands": [
      "npm run",
      "npm test",
      "git status",
      "git diff",
      "git log",
      "make lint",
      "make test"
    ],
    "deniedCommands": [
      "curl",
      "wget",
      "ssh",
      "scp",
      "rsync",
      "rm -rf",
      "git push",
      "git checkout main",
      "docker push",
      "kubectl apply",
      "terraform apply",
      "npm publish",
      "aws",
      "gcloud",
      "az"
    ],
    "allowFileWrite": true,
    "allowFileDelete": false,
    "allowNetworkAccess": false,
    "requireConfirmation": [
      "Write",
      "Edit",
      "Bash"
    ]
  },
  "context": {
    "excludePatterns": [
      "**/*.pem",
      "**/*.key",
      "**/*.cert",
      "**/*.p12",
      "**/secrets/**",
      "**/credentials/**",
      "**/.env*",
      "**/config/prod/**",
      "**/node_modules/**"
    ]
  },
  "mcp": {
    "servers": {}
  },
  "telemetry": {
    "enabled": false,
    "shareUsageData": false,
    "shareCrashReports": false
  },
  "advanced": {
    "apiBaseUrl": "https://claude-api.internal.company.com"
  }
}
```

**.claudeignore:**

```gitignore
# Security-sensitive files
*.pem
*.key
*.cert
*.p12
*.pfx
*.jks

# Credentials
.env*
*.env
secrets/
credentials/
config/prod/
config/production/

# Compliance
audit/
compliance/
pci/

# Customer data
data/customers/
exports/

# Build artifacts
node_modules/
dist/
build/

# Logs (may contain sensitive data)
logs/
*.log
```

**CLAUDE.md:**

```markdown
# SecureBank Trading Platform

## Security Notice
This codebase handles financial data. Follow these rules:
1. NEVER expose API keys or credentials
2. NEVER log sensitive customer data
3. NEVER bypass authentication checks
4. ALWAYS use parameterized queries
5. ALWAYS validate and sanitize input

## Compliance Requirements
- PCI-DSS compliant code patterns required
- All changes need security review
- Audit logging mandatory for data access

## Development
```bash
# Local development only
make dev

# Run security linter
make security-check

# Run full test suite
make test-all
```

## Restricted Operations
The following require manual execution and approval:
- Database migrations
- Deployment to any environment
- Changes to authentication logic
- Changes to encryption/hashing

## Code Review Checklist
- [ ] No hardcoded secrets
- [ ] Input validation present
- [ ] SQL injection prevention
- [ ] XSS prevention
- [ ] Proper error handling (no stack traces to users)
- [ ] Audit logging added
```

---

### Example 4: Open Source Project Setup

**Project: Popular NPM Package**

**.claude/settings.json:**

```json
{
  "model": {
    "default": "claude-sonnet-4-20250514"
  },
  "permissions": {
    "mode": "default",
    "allowedCommands": [
      "npm",
      "npx",
      "node",
      "git",
      "changeset"
    ]
  },
  "context": {
    "autoloadFiles": [
      "CLAUDE.md",
      "CONTRIBUTING.md",
      "package.json"
    ],
    "excludePatterns": [
      "node_modules/**",
      "dist/**",
      "coverage/**"
    ]
  }
}
```

**CLAUDE.md:**

```markdown
# DataGrid - High-Performance Table Component

## Overview
Open source React data grid with virtual scrolling, sorting,
filtering, and editing. 50k+ weekly npm downloads.

## Contributing
See CONTRIBUTING.md for full guidelines.

Quick setup:
```bash
git clone https://github.com/org/datagrid
npm install
npm run dev
```

## Project Structure
```
src/
├── components/
│   ├── DataGrid.tsx       # Main component
│   ├── Row.tsx            # Row renderer
│   ├── Cell.tsx           # Cell renderer
│   └── Header.tsx         # Header component
├── hooks/
│   ├── useVirtualization.ts
│   ├── useSorting.ts
│   └── useFiltering.ts
├── utils/
│   └── helpers.ts
└── types/
    └── index.ts

examples/
├── basic/
├── advanced/
└── with-pagination/

docs/
└── (generated documentation)
```

## Development Commands
```bash
npm run dev          # Start dev server with examples
npm run test         # Run Jest tests
npm run test:watch   # Watch mode
npm run build        # Build for production
npm run docs         # Generate documentation
npm run changeset    # Create changeset for release
```

## Release Process
1. Create changesets for changes: `npm run changeset`
2. Version packages: `npm run version`
3. CI handles publishing on merge to main

## Code Style
- Functional components only
- Comprehensive JSDoc comments
- 100% TypeScript (no `any`)
- Tests for all public APIs
- Storybook stories for components

## Performance Requirements
- Initial render < 16ms for 1000 rows
- Scroll FPS > 55
- Memory usage < 50MB for 10k rows
- Bundle size < 20KB gzipped

## API Stability
Public APIs (marked @public) require:
- Deprecation notice one major version before removal
- Migration guide in changelog
- Backwards-compatible alternatives
```

---

### Example 5: Personal Development Setup

**For: Solo Developer with Multiple Projects**

**~/.claude/settings.json (User Settings):**

```json
{
  "model": {
    "default": "claude-sonnet-4-20250514",
    "temperature": 0
  },
  "ui": {
    "theme": "dark",
    "showTokenUsage": true,
    "showCost": true,
    "compactMode": false
  },
  "permissions": {
    "mode": "default",
    "allowedCommands": [
      "npm",
      "npx",
      "node",
      "pnpm",
      "yarn",
      "bun",
      "python",
      "pip",
      "poetry",
      "cargo",
      "go",
      "git",
      "docker",
      "docker-compose",
      "kubectl",
      "terraform",
      "make"
    ],
    "deniedCommands": [
      "rm -rf /",
      "rm -rf ~",
      "sudo rm -rf"
    ]
  },
  "context": {
    "excludePatterns": [
      "**/node_modules/**",
      "**/.git/**",
      "**/dist/**",
      "**/build/**",
      "**/__pycache__/**",
      "**/.venv/**",
      "**/target/**"
    ]
  },
  "mcp": {
    "servers": {
      "github": {
        "command": "npx",
        "args": ["-y", "@anthropic/mcp-server-github"],
        "env": {
          "GITHUB_TOKEN": "${GITHUB_TOKEN}"
        }
      },
      "filesystem": {
        "command": "npx",
        "args": ["-y", "@anthropic/mcp-server-filesystem", "~"],
        "enabled": true
      }
    }
  },
  "history": {
    "maxSessions": 50,
    "autoSave": true
  }
}
```

**Template CLAUDE.md for New Projects:**

```markdown
# Project Name

## Quick Start
```bash
# Installation
npm install

# Development
npm run dev

# Testing
npm run test
```

## Structure
[Add project structure here]

## Standards
- [Add coding standards here]

## Notes
- [Add any special notes for Claude here]
```

---

## 7. .claudeignore Configuration

### Purpose

The `.claudeignore` file tells Claude Code which files and directories to exclude from context. This:

1. Protects sensitive information
2. Reduces token usage
3. Focuses Claude on relevant code
4. Improves response speed

### Syntax

Uses gitignore-style patterns:

```gitignore
# Comments start with #
# Blank lines are ignored

# Exact file match
specific-file.txt

# Directory (trailing slash)
node_modules/

# Wildcard patterns
*.log
*.tmp
*.bak

# Double asterisk (recursive)
**/*.test.js
**/temp/**

# Negation (include despite earlier exclusion)
!important.log

# Character classes
*.[oa]
*.py[cod]

# Range patterns
file[0-9].txt
```

### Common Patterns

**Standard .claudeignore:**

```gitignore
# Dependencies
node_modules/
vendor/
.venv/
venv/
__pycache__/
*.pyc

# Build outputs
dist/
build/
out/
*.min.js
*.min.css
*.bundle.js

# IDE and editor
.idea/
.vscode/
*.swp
*.swo
*~

# OS files
.DS_Store
Thumbs.db

# Environment and secrets
.env
.env.*
*.local
secrets/
credentials/
*.pem
*.key

# Logs and caches
*.log
logs/
.cache/
.npm/
.yarn/

# Test outputs
coverage/
.nyc_output/
htmlcov/
.pytest_cache/

# Generated files
*.generated.*
*.auto.*

# Large data files
*.csv
*.json.gz
*.sqlite
*.db
data/raw/
```

### Security Considerations

**Always exclude:**

```gitignore
# Credentials and secrets
.env
.env.*
*.env
secrets/
credentials/
**/credentials/**
*.pem
*.key
*.p12
*.pfx
*.jks
*.keystore

# Cloud provider configs
.aws/
.gcloud/
.azure/
kubeconfig
*.kubeconfig

# SSH
.ssh/
id_rsa*
id_ed25519*

# Tokens and API keys
*token*
*apikey*
*api_key*

# Database dumps
*.sql
*.dump
*.bak

# Private configuration
config/production/
config/prod/
config/private/
```

---

## 8. Permission Configuration

### Permission Modes

Claude Code supports three permission modes:

| Mode | Description | Use Case |
|------|-------------|----------|
| `default` | Asks permission for file writes and commands | Most development work |
| `plan` | Read-only, plans but doesn't execute | Code review, learning |
| `trust` | Auto-approves most operations | Trusted, repetitive tasks |

**Setting permission mode:**

```bash
# Command line
claude --permission-mode plan

# Environment variable
export CLAUDE_CODE_PERMISSION_MODE=trust

# In settings.json
{
  "permissions": {
    "mode": "default"
  }
}
```

### Tool-Specific Permissions

```json
{
  "permissions": {
    "allowedTools": [
      "Read",
      "Glob",
      "Grep",
      "WebFetch"
    ],
    "deniedTools": [
      "WebSearch"
    ],
    "requireConfirmation": [
      "Write",
      "Edit",
      "Bash",
      "NotebookEdit"
    ]
  }
}
```

**Available tools:**

| Tool | Purpose | Risk Level |
|------|---------|------------|
| Read | Read files | Low |
| Glob | Find files by pattern | Low |
| Grep | Search file contents | Low |
| Write | Create/overwrite files | Medium |
| Edit | Modify files | Medium |
| Bash | Execute commands | High |
| NotebookEdit | Edit Jupyter notebooks | Medium |
| WebFetch | Fetch URLs | Low |
| WebSearch | Search the web | Low |

### Command Allowlists/Blocklists

```json
{
  "permissions": {
    "allowedCommands": [
      "npm run",
      "npm test",
      "npm install",
      "git status",
      "git diff",
      "git add",
      "git commit",
      "python -m pytest",
      "make test"
    ],
    "deniedCommands": [
      "rm -rf",
      "sudo",
      "chmod 777",
      "curl | bash",
      "wget | sh",
      "git push --force",
      "git push origin main",
      "npm publish",
      "docker push",
      "> /dev/sda"
    ]
  }
}
```

### Security Profiles

**High Security Profile:**

```json
{
  "permissions": {
    "mode": "default",
    "allowedTools": ["Read", "Glob", "Grep"],
    "deniedTools": ["Bash", "WebFetch", "WebSearch"],
    "allowFileWrite": false,
    "allowFileDelete": false,
    "allowNetworkAccess": false,
    "requireConfirmation": ["Read"]
  }
}
```

**Development Profile:**

```json
{
  "permissions": {
    "mode": "default",
    "allowedTools": ["Read", "Glob", "Grep", "Write", "Edit", "Bash"],
    "allowedCommands": ["npm", "git", "node", "python"],
    "deniedCommands": ["rm -rf /", "sudo rm"],
    "requireConfirmation": ["Bash"]
  }
}
```

**Automation Profile:**

```json
{
  "permissions": {
    "mode": "trust",
    "allowedTools": ["Read", "Glob", "Grep", "Write", "Edit", "Bash"],
    "allowedCommands": ["npm run build", "npm test", "git add", "git commit"],
    "deniedCommands": ["git push", "npm publish", "rm -rf"]
  }
}
```

---

## 9. Model Configuration

### Default Model Selection

```json
{
  "model": {
    "default": "claude-sonnet-4-20250514"
  }
}
```

**Available models:**

| Model | Best For | Cost | Speed |
|-------|----------|------|-------|
| `claude-opus-4-20250514` | Complex reasoning, difficult tasks | $$$$$ | Slowest |
| `claude-sonnet-4-20250514` | General development (recommended) | $$ | Fast |
| `claude-haiku-3-5-20241022` | Simple tasks, quick edits | $ | Fastest |

### Per-Task Model Override

```bash
# Use Opus for complex refactoring
claude --model claude-opus-4-20250514 -p "Refactor auth system to use JWT"

# Use Haiku for quick tasks
claude --model claude-haiku-3-5-20241022 -p "Add a comment to this function"
```

### Cost vs Capability Trade-offs

| Task | Recommended Model | Why |
|------|-------------------|-----|
| Architecture design | Opus 4 | Complex reasoning needed |
| Code review | Sonnet 4 | Good balance |
| Bug fixes | Sonnet 4 | General purpose |
| Simple edits | Haiku 3.5 | Fast and cheap |
| Documentation | Haiku 3.5 | Straightforward task |
| Complex debugging | Opus 4 | Deep analysis needed |

### Model-Specific Settings

```json
{
  "model": {
    "default": "claude-sonnet-4-20250514",
    "temperature": 0,
    "maxTokens": 16384,
    "extendedThinking": false,
    "thinkingBudget": 10000
  }
}
```

**Extended thinking (for complex tasks):**

```json
{
  "model": {
    "default": "claude-sonnet-4-20250514",
    "extendedThinking": true,
    "thinkingBudget": 20000
  }
}
```

---

## 10. Environment Variables

### ANTHROPIC_API_KEY

**Required for Claude Code to function:**

```bash
# Set in shell profile (~/.bashrc, ~/.zshrc)
export ANTHROPIC_API_KEY="YOUR_API_KEY_HERE"

# Or in .env file (don't commit!)
ANTHROPIC_API_KEY=YOUR_API_KEY_HERE

# Windows PowerShell
$env:ANTHROPIC_API_KEY = "YOUR_API_KEY_HERE"

# Windows Command Prompt
set ANTHROPIC_API_KEY=YOUR_API_KEY_HERE
```

### CLAUDE_CODE_* Variables

```bash
# Configuration directory
export CLAUDE_CODE_CONFIG_DIR="~/.claude"

# Permission mode
export CLAUDE_CODE_PERMISSION_MODE="default"  # default, plan, trust

# Default model
export CLAUDE_CODE_MODEL="claude-sonnet-4-20250514"

# Disable telemetry
export CLAUDE_CODE_TELEMETRY="false"

# Verbose logging
export CLAUDE_CODE_VERBOSE="true"

# Custom API endpoint
export CLAUDE_CODE_API_URL="https://api.anthropic.com"
```

### Proxy Configuration

```bash
# HTTP proxy
export HTTP_PROXY="http://proxy.company.com:8080"
export HTTPS_PROXY="http://proxy.company.com:8080"

# Bypass proxy for local addresses
export NO_PROXY="localhost,127.0.0.1,.internal.company.com"

# Proxy with authentication
export HTTP_PROXY="http://user:pass@proxy.company.com:8080"
```

### Custom Variables for Projects

```bash
# Project-specific
export PROJECT_ENV="development"
export DATABASE_URL="postgresql://localhost/myapp"
export REDIS_URL="redis://localhost:6379"

# Use in settings.json with ${VAR} syntax
{
  "mcp": {
    "servers": {
      "database": {
        "env": {
          "DATABASE_URL": "${DATABASE_URL}"
        }
      }
    }
  }
}
```

### Complete Environment Setup Script

```bash
#!/bin/bash
# ~/.claude/setup-env.sh

# Required
export ANTHROPIC_API_KEY="YOUR_API_KEY_HERE"

# Claude Code settings
export CLAUDE_CODE_PERMISSION_MODE="default"
export CLAUDE_CODE_MODEL="claude-sonnet-4-20250514"

# Networking (if behind proxy)
# export HTTP_PROXY="http://proxy.company.com:8080"
# export HTTPS_PROXY="http://proxy.company.com:8080"
# export NO_PROXY="localhost,127.0.0.1"

# MCP server tokens
export GITHUB_TOKEN="ghp_..."
export GITLAB_TOKEN="glpat-..."
export SLACK_TOKEN="xoxb-..."

echo "Claude Code environment configured"
```

Add to shell profile:

```bash
# In ~/.bashrc or ~/.zshrc
source ~/.claude/setup-env.sh
```

---

## 11. Sharing Configurations

### Team Configuration Sharing

**Recommended structure:**

```
project/
├── .claude/
│   ├── settings.json           # Team settings (commit)
│   ├── settings.local.json     # Personal overrides (gitignore)
│   ├── commands/               # Shared custom commands (commit)
│   │   ├── deploy.md
│   │   └── review.md
│   └── mcp.json               # MCP config (commit, no secrets)
├── CLAUDE.md                   # Project context (commit)
├── .claudeignore              # Exclusion patterns (commit)
└── .gitignore                 # Include .claude/settings.local.json
```

**.gitignore entries:**

```gitignore
# Claude Code local settings
.claude/settings.local.json
.claude/*.local.json

# Never commit credentials
.claude/credentials.json
```

### Git-Tracked vs Git-Ignored

| File | Track in Git? | Contains |
|------|---------------|----------|
| `.claude/settings.json` | Yes | Team standards |
| `.claude/settings.local.json` | No | Personal preferences |
| `.claude/commands/*.md` | Yes | Shared custom commands |
| `CLAUDE.md` | Yes | Project documentation |
| `.claudeignore` | Yes | Exclusion patterns |
| `.claude/mcp.json` | Maybe | MCP servers (no secrets) |

### Configuration Inheritance

Settings merge in this order (later overrides earlier):

```
1. Default settings (built-in)
       ↓
2. User settings (~/.claude/settings.json)
       ↓
3. Project settings (.claude/settings.json)
       ↓
4. Workspace settings (.claude/settings.local.json)
       ↓
5. Environment variables
       ↓
6. Command-line arguments
```

**Example of inheritance:**

```json
// ~/.claude/settings.json (User)
{
  "model": { "default": "claude-sonnet-4-20250514" },
  "permissions": { "mode": "default" }
}

// .claude/settings.json (Project)
{
  "permissions": {
    "allowedCommands": ["npm", "git"]
  }
}

// .claude/settings.local.json (Workspace)
{
  "permissions": { "mode": "trust" }
}

// Merged result:
{
  "model": { "default": "claude-sonnet-4-20250514" },  // from user
  "permissions": {
    "mode": "trust",                                   // from workspace
    "allowedCommands": ["npm", "git"]                 // from project
  }
}
```

### Plugin Configurations

Custom slash commands can be shared:

**.claude/commands/review.md:**

```markdown
# Code Review Command

Review the changes in the current branch compared to main.

## Instructions
1. Run `git diff main...HEAD` to see all changes
2. Analyze each changed file
3. Check for:
   - Code style issues
   - Potential bugs
   - Missing tests
   - Security concerns
4. Provide a summary with specific suggestions
```

**.claude/commands/deploy.md:**

```markdown
# Deploy Command

Deploy the application to the specified environment.

## Arguments
- $ENVIRONMENT - Target environment (staging, production)

## Instructions
1. Verify all tests pass: `npm test`
2. Build the application: `npm run build`
3. If staging: `npm run deploy:staging`
4. If production: Require explicit confirmation first
5. Verify deployment health
```

---

## 12. Troubleshooting Configuration

### Configuration Not Loading

**Symptoms:**
- Settings don't apply
- Using defaults instead of your settings

**Diagnosis:**

```bash
# Check if settings file exists
ls -la ~/.claude/settings.json
ls -la .claude/settings.json

# Validate JSON syntax
cat ~/.claude/settings.json | python -m json.tool

# Run with verbose to see config loading
claude --verbose
```

**Common fixes:**

1. **Invalid JSON:**
   ```bash
   # Check for syntax errors
   cat ~/.claude/settings.json | python -m json.tool
   ```

2. **Wrong location:**
   ```bash
   # Ensure correct path
   mkdir -p ~/.claude
   ```

3. **Permissions issue:**
   ```bash
   chmod 644 ~/.claude/settings.json
   ```

### Conflicting Settings

**Symptoms:**
- Unexpected behavior
- Settings seem to flip-flop

**Diagnosis:**

```bash
# Check all config locations
cat ~/.claude/settings.json
cat .claude/settings.json
cat .claude/settings.local.json
env | grep CLAUDE
```

**Resolution:**

1. Understand the priority order (see Configuration Inheritance)
2. Remove conflicting settings from lower-priority locations
3. Use `settings.local.json` for machine-specific overrides

### Reset to Defaults

**Full reset:**

```bash
# Backup current settings
cp -r ~/.claude ~/.claude.backup

# Remove user configuration
rm ~/.claude/settings.json

# Remove project configuration
rm -rf .claude/

# Restart Claude Code
claude
```

**Partial reset:**

```bash
# Reset only user settings
rm ~/.claude/settings.json

# Reset only project settings
rm .claude/settings.json
rm .claude/settings.local.json
```

### Debug Configuration

**Enable verbose mode:**

```bash
# Via command line
claude --verbose

# Via environment
export CLAUDE_CODE_VERBOSE=true
claude
```

**Check loaded configuration:**

```bash
# In Claude Code
> /config show

# Or ask Claude
"What settings are currently active?"
```

**View logs:**

```bash
# Check log directory
ls ~/.claude/logs/

# View recent logs
tail -f ~/.claude/logs/claude-code-*.log
```

**Common configuration errors:**

| Error | Cause | Fix |
|-------|-------|-----|
| "Invalid JSON" | Syntax error in settings | Validate with `python -m json.tool` |
| "Unknown setting" | Typo or deprecated setting | Check settings reference |
| "Permission denied" | Can't read/write config | Check file permissions |
| "API key not found" | Missing ANTHROPIC_API_KEY | Set environment variable |

---

## 13. Pros and Cons

### Customization Benefits

**Pros:**

| Benefit | Description |
|---------|-------------|
| **Efficiency** | Pre-configured context reduces setup time |
| **Consistency** | Same behavior across sessions |
| **Safety** | Permission controls prevent accidents |
| **Team Alignment** | Shared configs ensure standards |
| **Automation** | Trusted mode enables scripting |
| **Focus** | .claudeignore reduces noise |
| **Security** | Granular permission control |

### Complexity Considerations

**Cons:**

| Consideration | Description |
|---------------|-------------|
| **Learning Curve** | Many options to understand |
| **Debugging** | Config inheritance can confuse |
| **Maintenance** | Settings need updates over time |
| **Conflicts** | Team vs personal preferences |
| **Over-Configuration** | Too many restrictions reduce usefulness |

### Maintenance Overhead

**Recommendations:**

1. **Start simple** - Begin with minimal configuration
2. **Add as needed** - Configure when you hit pain points
3. **Document changes** - Comment your settings files
4. **Review periodically** - Update outdated settings
5. **Share knowledge** - Document team configurations

---

## 14. Quick Reference

### Settings Cheat Sheet

```bash
# View current configuration
claude --verbose
/config show

# Set model
claude --model claude-opus-4-20250514

# Set permission mode
claude --permission-mode trust

# Use specific config file
claude --config /path/to/settings.json

# Non-interactive mode
claude -p "your prompt"

# Resume last session
claude --continue
```

### Configuration File Locations

| File | Purpose |
|------|---------|
| `~/.claude/settings.json` | User-level settings |
| `.claude/settings.json` | Project settings (commit) |
| `.claude/settings.local.json` | Local overrides (don't commit) |
| `CLAUDE.md` | Project context |
| `.claudeignore` | Exclude files from context |
| `.claude/commands/*.md` | Custom slash commands |
| `.claude/mcp.json` | MCP server configuration |

### Essential Environment Variables

```bash
export ANTHROPIC_API_KEY="YOUR_API_KEY"
export CLAUDE_CODE_PERMISSION_MODE="default"
export CLAUDE_CODE_MODEL="claude-sonnet-4-20250514"
```

### Configuration Templates

**Minimal settings.json:**

```json
{
  "model": { "default": "claude-sonnet-4-20250514" },
  "ui": { "showCost": true }
}
```

**Recommended settings.json:**

```json
{
  "model": {
    "default": "claude-sonnet-4-20250514",
    "temperature": 0
  },
  "permissions": {
    "mode": "default",
    "allowedCommands": ["npm", "git", "node"],
    "deniedCommands": ["rm -rf /"]
  },
  "ui": {
    "showTokenUsage": true,
    "showCost": true
  }
}
```

**Secure settings.json:**

```json
{
  "permissions": {
    "mode": "default",
    "allowedTools": ["Read", "Glob", "Grep"],
    "deniedTools": ["WebFetch", "WebSearch"],
    "requireConfirmation": ["Write", "Edit", "Bash"],
    "allowFileDelete": false,
    "allowNetworkAccess": false
  }
}
```

### Common Patterns

**Python project:**

```json
{
  "permissions": {
    "allowedCommands": ["python", "pip", "poetry", "pytest", "black", "mypy"]
  },
  "context": {
    "excludePatterns": ["__pycache__/**", ".venv/**", "*.pyc"]
  }
}
```

**Node.js project:**

```json
{
  "permissions": {
    "allowedCommands": ["npm", "npx", "node", "yarn", "pnpm"]
  },
  "context": {
    "excludePatterns": ["node_modules/**", "dist/**", "coverage/**"]
  }
}
```

**Go project:**

```json
{
  "permissions": {
    "allowedCommands": ["go", "make", "docker"]
  },
  "context": {
    "excludePatterns": ["vendor/**", "bin/**"]
  }
}
```

**Rust project:**

```json
{
  "permissions": {
    "allowedCommands": ["cargo", "rustc", "rustfmt", "clippy"]
  },
  "context": {
    "excludePatterns": ["target/**"]
  }
}
```

---

## Summary

Claude Code's configuration system provides powerful customization while maintaining sensible defaults. Key takeaways:

1. **Configuration is layered** - User < Project < Workspace < Environment < CLI
2. **CLAUDE.md is essential** - Provide project context for better assistance
3. **Security matters** - Use permissions and .claudeignore appropriately
4. **Start simple** - Add configuration as needed
5. **Share team configs** - Use .claude/settings.json for consistency

With proper configuration, Claude Code becomes an even more powerful development partner, understanding your project's unique requirements and respecting your security and workflow preferences.

---

## Next Steps

- **Tutorial 10**: Advanced Workflows and Automation
- **Tutorial 11**: CI/CD Integration with Claude Code
- **Tutorial 12**: Building Custom Extensions

---

*This tutorial is part of the Claude Code documentation series.*
