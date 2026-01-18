# CLAUDE.md - Complete Guide

> **CLAUDE.md** is the project context file that Claude Code automatically reads at the start of every conversation. It provides essential information about your project, coding conventions, and workflow preferences.

---

## Quick Reference

| Property | Value |
|----------|-------|
| **Location** | Project root (`./CLAUDE.md`) |
| **Frontmatter** | None (free-form markdown) |
| **Loading** | Automatic at conversation start |
| **Imports** | Supported via `@import ./path/to/file.md` |
| **Size** | Keep concise (aim for < 500 lines) |

---

## What to Include

### Essential Sections

| Section | Purpose | Priority |
|---------|---------|----------|
| **Project Overview** | What the project does | High |
| **Tech Stack** | Languages, frameworks, tools | High |
| **Key Commands** | Build, test, run commands | High |
| **Directory Structure** | Important folders/files | Medium |
| **Coding Conventions** | Project-specific standards | Medium |
| **Architecture** | System design overview | Medium |
| **Development Workflow** | PR process, deployment | Low |

### What NOT to Include

| Don't Include | Why |
|---------------|-----|
| Generic programming advice | Claude already knows |
| Language basics | Not project-specific |
| Secrets/credentials | Security risk |
| Frequently changing info | Becomes stale |
| Extremely detailed specs | Use @import instead |

---

## Template Anatomy

```markdown
# Project Name

Brief one-line description of what this project does.

## Tech Stack

- **Language**: TypeScript 5.x
- **Framework**: React 18, Next.js 14
- **Database**: PostgreSQL 15
- **Testing**: Jest, Playwright

## Quick Start

\`\`\`bash
npm install     # Install dependencies
npm run dev     # Start development server
npm test        # Run tests
npm run build   # Production build
\`\`\`

## Project Structure

\`\`\`
src/
├── app/           # Next.js app router
├── components/    # React components
├── lib/           # Utilities
└── services/      # Business logic
\`\`\`

## Conventions

- Use functional components with hooks
- Prefer named exports
- Tests live next to source files

## Architecture

[Brief system overview]

@import ./docs/architecture.md
```

---

## Templates

### Minimal Template

```markdown
# Project Name

Brief description.

## Tech Stack
- TypeScript, React, Node.js

## Commands
- `npm run dev` - Development
- `npm test` - Run tests
- `npm run build` - Build
```

### Standard Template

```markdown
# Project Name

One paragraph describing the project's purpose and main functionality.

## Tech Stack

| Category | Technology |
|----------|------------|
| Language | TypeScript 5.x |
| Frontend | React 18, Next.js 14 |
| Backend | Node.js, Express |
| Database | PostgreSQL 15 |
| Testing | Jest, Playwright |
| CI/CD | GitHub Actions |

## Quick Start

\`\`\`bash
# Install
npm install

# Environment setup
cp .env.example .env

# Run development
npm run dev

# Run tests
npm test
\`\`\`

## Project Structure

\`\`\`
src/
├── app/              # Next.js pages/routes
├── components/       # Reusable UI components
│   ├── ui/           # Base components
│   └── features/     # Feature-specific components
├── lib/              # Utilities and helpers
├── services/         # Business logic layer
├── hooks/            # Custom React hooks
└── types/            # TypeScript types
\`\`\`

## Key Files

| File | Purpose |
|------|---------|
| `src/app/layout.tsx` | Root layout |
| `src/lib/db.ts` | Database connection |
| `src/services/auth.ts` | Authentication logic |

## Conventions

### Naming
- Components: PascalCase (`UserProfile.tsx`)
- Utilities: camelCase (`formatDate.ts`)
- Constants: UPPER_SNAKE (`API_BASE_URL`)

### Components
- Functional components with hooks
- Props interfaces above component
- Named exports preferred

### Testing
- Tests in `__tests__` folders
- Name: `*.test.ts` or `*.spec.ts`
- Minimum 80% coverage

## Common Tasks

### Add a new API endpoint
1. Create route in `src/app/api/`
2. Add types in `src/types/`
3. Write tests
4. Update API docs

### Add a new component
1. Create in `src/components/`
2. Add Storybook story
3. Write unit tests
```

### Advanced Template

```markdown
# Project Name

[One paragraph description]

## Overview

### Purpose
What problem this solves and for whom.

### Key Features
- Feature 1
- Feature 2
- Feature 3

## Tech Stack

### Core
| Technology | Version | Purpose |
|------------|---------|---------|
| TypeScript | 5.3 | Language |
| React | 18.2 | UI Framework |
| Next.js | 14 | Full-stack Framework |

### Infrastructure
| Technology | Purpose |
|------------|---------|
| PostgreSQL | Primary database |
| Redis | Caching layer |
| S3 | File storage |

### Development
| Tool | Purpose |
|------|---------|
| ESLint | Linting |
| Prettier | Formatting |
| Husky | Git hooks |

## Quick Start

\`\`\`bash
# Prerequisites: Node 20+, pnpm 8+

# Clone and install
git clone <repo>
cd <project>
pnpm install

# Environment setup
cp .env.example .env.local
# Edit .env.local with your values

# Database setup
pnpm db:migrate
pnpm db:seed

# Start development
pnpm dev
\`\`\`

## Project Structure

\`\`\`
├── src/
│   ├── app/                 # Next.js App Router
│   │   ├── (auth)/          # Auth-required routes
│   │   ├── (public)/        # Public routes
│   │   └── api/             # API routes
│   ├── components/
│   │   ├── ui/              # shadcn/ui components
│   │   ├── forms/           # Form components
│   │   └── layouts/         # Layout components
│   ├── lib/
│   │   ├── db/              # Database utilities
│   │   ├── auth/            # Auth utilities
│   │   └── utils/           # General utilities
│   ├── services/            # Business logic
│   ├── hooks/               # Custom React hooks
│   └── types/               # TypeScript types
├── prisma/
│   ├── schema.prisma        # Database schema
│   └── migrations/          # Database migrations
├── tests/
│   ├── unit/                # Unit tests
│   ├── integration/         # Integration tests
│   └── e2e/                 # E2E tests (Playwright)
└── docs/                    # Documentation
\`\`\`

## Architecture

### Request Flow
\`\`\`
Client → Next.js API Route → Service Layer → Database
                ↓
         Middleware (auth, validation)
\`\`\`

### Key Design Decisions
1. **Service Layer**: Business logic separated from routes
2. **Repository Pattern**: Database access abstracted
3. **Feature Folders**: Related code co-located

@import ./docs/architecture.md

## Conventions

### Code Style

#### TypeScript
- Strict mode enabled
- Explicit return types on functions
- Prefer `interface` over `type` for objects
- Use `unknown` over `any`

#### React
- Functional components only
- Custom hooks for shared logic
- Props interfaces named `{Component}Props`
- Default exports for pages, named for components

#### File Naming
| Type | Convention | Example |
|------|------------|---------|
| Component | PascalCase | `UserProfile.tsx` |
| Hook | camelCase with `use` | `useAuth.ts` |
| Utility | camelCase | `formatDate.ts` |
| Type | PascalCase | `User.ts` |
| Test | `.test.ts` suffix | `auth.test.ts` |

### Git Conventions

#### Branch Naming
- `feature/description` - New features
- `fix/description` - Bug fixes
- `refactor/description` - Refactoring
- `docs/description` - Documentation

#### Commit Messages
Follow Conventional Commits:
\`\`\`
feat: add user authentication
fix: resolve login redirect issue
docs: update API documentation
refactor: simplify auth logic
test: add user service tests
\`\`\`

### Testing

| Type | Location | Command |
|------|----------|---------|
| Unit | `tests/unit/` | `pnpm test:unit` |
| Integration | `tests/integration/` | `pnpm test:int` |
| E2E | `tests/e2e/` | `pnpm test:e2e` |

Coverage requirement: 80% for new code

## API Reference

### Authentication
\`\`\`typescript
// Login
POST /api/auth/login
Body: { email: string, password: string }
Response: { token: string, user: User }

// Logout
POST /api/auth/logout
Response: { success: boolean }
\`\`\`

### Users
\`\`\`typescript
// Get user
GET /api/users/:id
Response: User

// Update user
PATCH /api/users/:id
Body: Partial<User>
Response: User
\`\`\`

## Common Tasks

### Adding a New Feature

1. **Plan**: Write ticket/spec
2. **Branch**: `git checkout -b feature/name`
3. **Implement**:
   - Add types in `src/types/`
   - Add service in `src/services/`
   - Add API route in `src/app/api/`
   - Add UI in `src/components/`
4. **Test**: Write unit and integration tests
5. **PR**: Create PR with description
6. **Review**: Address feedback
7. **Merge**: Squash and merge

### Database Changes

1. Update `prisma/schema.prisma`
2. Generate migration: `pnpm prisma migrate dev --name description`
3. Update types: `pnpm prisma generate`
4. Update affected services

## Deployment

### Environments
| Environment | URL | Branch |
|-------------|-----|--------|
| Development | localhost:3000 | - |
| Staging | staging.example.com | develop |
| Production | example.com | main |

### Deploy Process
1. PR merged to `main`
2. CI runs tests
3. Auto-deploy to staging
4. Manual promotion to production

## Troubleshooting

### Common Issues

**Database connection fails**
\`\`\`bash
# Check PostgreSQL is running
pg_isready

# Verify connection string in .env
\`\`\`

**Tests fail locally**
\`\`\`bash
# Reset test database
pnpm db:reset:test

# Run with verbose
pnpm test --verbose
\`\`\`

## Additional Resources

- [Architecture Decisions](./docs/adr/)
- [API Documentation](./docs/api/)
- [Contributing Guide](./CONTRIBUTING.md)
```

---

## Examples

### Example 1: Minimal Project

```markdown
# Personal Blog

A static blog built with Astro and Markdown.

## Tech Stack
- Astro 4.x
- Tailwind CSS
- Markdown for content

## Commands
- `npm run dev` - Start dev server (localhost:4321)
- `npm run build` - Build for production
- `npm run preview` - Preview production build

## Structure
\`\`\`
src/
├── pages/       # Routes
├── layouts/     # Page layouts
├── components/  # UI components
└── content/     # Blog posts (markdown)
\`\`\`

## Adding a Post
Create `src/content/blog/post-name.md` with frontmatter:
\`\`\`yaml
---
title: Post Title
date: 2024-01-15
tags: [tag1, tag2]
---
\`\`\`
```

### Example 2: TypeScript React Project

```markdown
# TaskFlow - Project Management App

A modern project management application with real-time collaboration.

## Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | React 18, TypeScript, Vite |
| State | Zustand, React Query |
| Styling | Tailwind CSS, shadcn/ui |
| Backend | Node.js, Express, tRPC |
| Database | PostgreSQL, Prisma |
| Real-time | Socket.io |
| Testing | Vitest, Playwright |

## Quick Start

\`\`\`bash
# Install dependencies
pnpm install

# Start database
docker-compose up -d postgres

# Run migrations
pnpm db:migrate

# Start development
pnpm dev        # Starts both frontend and backend
\`\`\`

## Project Structure

\`\`\`
apps/
├── web/               # React frontend
│   ├── src/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── pages/
│   │   ├── stores/
│   │   └── utils/
│   └── package.json
└── api/               # Express backend
    ├── src/
    │   ├── routes/
    │   ├── services/
    │   ├── middleware/
    │   └── utils/
    └── package.json
packages/
├── ui/                # Shared UI components
├── types/             # Shared TypeScript types
└── utils/             # Shared utilities
\`\`\`

## Conventions

### Components
\`\`\`typescript
// Props interface above component
interface ButtonProps {
  variant: 'primary' | 'secondary';
  onClick: () => void;
  children: React.ReactNode;
}

// Functional component with explicit types
export function Button({ variant, onClick, children }: ButtonProps) {
  return <button className={styles[variant]} onClick={onClick}>{children}</button>;
}
\`\`\`

### API Routes
\`\`\`typescript
// Use tRPC for type-safe APIs
export const taskRouter = router({
  list: publicProcedure
    .input(z.object({ projectId: z.string() }))
    .query(({ input }) => taskService.list(input.projectId)),

  create: protectedProcedure
    .input(createTaskSchema)
    .mutation(({ input, ctx }) => taskService.create(input, ctx.user)),
});
\`\`\`

### State Management
- Server state: React Query
- Client state: Zustand
- Forms: React Hook Form + Zod

## Key Files

| File | Purpose |
|------|---------|
| `apps/web/src/App.tsx` | Root component |
| `apps/api/src/index.ts` | API entry point |
| `packages/types/src/index.ts` | Shared types |
| `prisma/schema.prisma` | Database schema |

## Scripts

\`\`\`bash
pnpm dev           # Start all apps
pnpm build         # Build all apps
pnpm test          # Run all tests
pnpm lint          # Lint all packages
pnpm db:studio     # Open Prisma Studio
\`\`\`
```

### Example 3: Python Django Project

```markdown
# E-Commerce Platform

A scalable e-commerce backend built with Django and Django REST Framework.

## Tech Stack

- **Language**: Python 3.11
- **Framework**: Django 4.2, DRF 3.14
- **Database**: PostgreSQL 15
- **Cache**: Redis 7
- **Task Queue**: Celery
- **Search**: Elasticsearch 8
- **Testing**: pytest, factory_boy

## Quick Start

\`\`\`bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Environment setup
cp .env.example .env

# Database setup
python manage.py migrate
python manage.py createsuperuser

# Run development server
python manage.py runserver
\`\`\`

## Project Structure

\`\`\`
src/
├── config/              # Django settings
│   ├── settings/
│   │   ├── base.py
│   │   ├── development.py
│   │   └── production.py
│   ├── urls.py
│   └── wsgi.py
├── apps/
│   ├── users/           # User management
│   ├── products/        # Product catalog
│   ├── orders/          # Order processing
│   ├── payments/        # Payment handling
│   └── notifications/   # Email/SMS notifications
├── common/              # Shared utilities
│   ├── models.py        # Base models
│   ├── permissions.py   # DRF permissions
│   └── pagination.py    # Custom pagination
└── tests/               # Test files
\`\`\`

## Conventions

### Models
\`\`\`python
from common.models import BaseModel

class Product(BaseModel):
    """Product in the catalog."""

    name = models.CharField(max_length=255)
    price = models.DecimalField(max_digits=10, decimal_places=2)

    class Meta:
        ordering = ['-created_at']
\`\`\`

### Views
\`\`\`python
from rest_framework import viewsets
from .models import Product
from .serializers import ProductSerializer

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    permission_classes = [IsAuthenticatedOrReadOnly]
\`\`\`

### Tests
\`\`\`python
import pytest
from .factories import ProductFactory

@pytest.mark.django_db
class TestProductAPI:
    def test_list_products(self, api_client):
        ProductFactory.create_batch(3)
        response = api_client.get('/api/products/')
        assert response.status_code == 200
        assert len(response.data['results']) == 3
\`\`\`

## Commands

\`\`\`bash
# Development
python manage.py runserver
python manage.py shell_plus

# Database
python manage.py makemigrations
python manage.py migrate
python manage.py dbshell

# Testing
pytest
pytest --cov=src
pytest -x -vv  # Stop on first failure, verbose

# Celery
celery -A config worker -l info
celery -A config beat -l info
\`\`\`

## API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/auth/login/` | POST | User login |
| `/api/products/` | GET | List products |
| `/api/products/{id}/` | GET | Product detail |
| `/api/orders/` | POST | Create order |
| `/api/orders/{id}/` | GET | Order detail |

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `DEBUG` | Debug mode | False |
| `DATABASE_URL` | PostgreSQL URL | - |
| `REDIS_URL` | Redis URL | - |
| `SECRET_KEY` | Django secret | - |
```

### Example 4: Monorepo Project

```markdown
# Acme Platform

Monorepo containing all Acme platform applications and shared packages.

## Tech Stack

- **Monorepo**: Turborepo, pnpm workspaces
- **Frontend**: React, Next.js
- **Backend**: Node.js, Fastify
- **Mobile**: React Native, Expo
- **Shared**: TypeScript, Zod

## Quick Start

\`\`\`bash
# Install pnpm if needed
npm install -g pnpm

# Install all dependencies
pnpm install

# Start all apps in development
pnpm dev

# Start specific app
pnpm --filter @acme/web dev
pnpm --filter @acme/api dev
\`\`\`

## Monorepo Structure

\`\`\`
├── apps/
│   ├── web/                 # Main web application (Next.js)
│   ├── admin/               # Admin dashboard (Next.js)
│   ├── api/                 # Backend API (Fastify)
│   ├── mobile/              # Mobile app (React Native)
│   └── docs/                # Documentation site (Docusaurus)
├── packages/
│   ├── ui/                  # Shared UI components
│   ├── config-eslint/       # ESLint configuration
│   ├── config-typescript/   # TypeScript configuration
│   ├── database/            # Prisma client and schema
│   ├── types/               # Shared TypeScript types
│   ├── utils/               # Shared utilities
│   └── validation/          # Zod schemas
├── tooling/
│   ├── scripts/             # Build and deploy scripts
│   └── docker/              # Docker configurations
├── turbo.json               # Turborepo configuration
├── pnpm-workspace.yaml      # Workspace configuration
└── package.json             # Root package.json
\`\`\`

## Package Dependencies

\`\`\`
@acme/web ──────┬── @acme/ui
                ├── @acme/types
                ├── @acme/utils
                └── @acme/validation

@acme/api ──────┬── @acme/database
                ├── @acme/types
                └── @acme/validation

@acme/ui ───────┬── @acme/types
                └── @acme/utils
\`\`\`

## Commands

### Root Commands
\`\`\`bash
pnpm dev              # Start all apps
pnpm build            # Build all apps
pnpm test             # Test all packages
pnpm lint             # Lint all packages
pnpm typecheck        # Type check all packages
\`\`\`

### Filtered Commands
\`\`\`bash
pnpm --filter @acme/web dev      # Start web app only
pnpm --filter @acme/api build    # Build API only
pnpm --filter "@acme/*" test     # Test all @acme packages
pnpm --filter "...@acme/ui" build  # Build ui and dependents
\`\`\`

### Adding Dependencies
\`\`\`bash
# Add to specific package
pnpm --filter @acme/web add lodash

# Add to root (dev dependency)
pnpm add -D -w turbo

# Add internal package dependency
pnpm --filter @acme/web add @acme/ui
\`\`\`

## Conventions

### Package Naming
- Apps: `@acme/app-name`
- Packages: `@acme/package-name`
- Configs: `@acme/config-name`

### Import Paths
\`\`\`typescript
// Internal packages
import { Button } from '@acme/ui';
import { User } from '@acme/types';
import { formatDate } from '@acme/utils';

// Relative imports within package
import { useAuth } from '../hooks/useAuth';
\`\`\`

### Creating New Packages
1. Create folder in `packages/` or `apps/`
2. Add `package.json` with `@acme/` prefix
3. Add to `pnpm-workspace.yaml` if needed
4. Configure in `turbo.json`
5. Run `pnpm install`

## CI/CD

- **PR Checks**: Lint, typecheck, test affected packages
- **Main Branch**: Build all, deploy to staging
- **Tags**: Deploy to production

Turborepo caches builds for unchanged packages.
```

### Example 5: Microservices Project

```markdown
# Order Management System

Microservices-based order management with event-driven architecture.

## Architecture Overview

\`\`\`
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Gateway   │────▶│   Orders    │────▶│  Inventory  │
│   Service   │     │   Service   │     │   Service   │
└─────────────┘     └─────────────┘     └─────────────┘
       │                   │                    │
       │                   ▼                    │
       │            ┌─────────────┐            │
       │            │    Kafka    │◀───────────┘
       │            │   (Events)  │
       │            └─────────────┘
       │                   │
       ▼                   ▼
┌─────────────┐     ┌─────────────┐
│    Users    │     │  Payments   │
│   Service   │     │   Service   │
└─────────────┘     └─────────────┘
\`\`\`

## Services

| Service | Port | Tech | Database |
|---------|------|------|----------|
| Gateway | 3000 | Express | - |
| Orders | 3001 | Fastify | PostgreSQL |
| Inventory | 3002 | Fastify | PostgreSQL |
| Users | 3003 | Fastify | PostgreSQL |
| Payments | 3004 | Fastify | PostgreSQL |

## Quick Start

\`\`\`bash
# Start infrastructure
docker-compose up -d postgres kafka redis

# Install all services
pnpm install

# Run migrations for all services
pnpm db:migrate

# Start all services
pnpm dev
\`\`\`

## Project Structure

\`\`\`
├── services/
│   ├── gateway/           # API Gateway
│   │   ├── src/
│   │   │   ├── routes/
│   │   │   ├── middleware/
│   │   │   └── proxy/
│   │   └── Dockerfile
│   ├── orders/            # Orders Service
│   │   ├── src/
│   │   │   ├── api/
│   │   │   ├── domain/
│   │   │   ├── events/
│   │   │   └── repository/
│   │   └── Dockerfile
│   ├── inventory/         # Inventory Service
│   ├── users/             # Users Service
│   └── payments/          # Payments Service
├── packages/
│   ├── events/            # Event schemas (Avro/Protobuf)
│   ├── types/             # Shared types
│   └── utils/             # Shared utilities
├── infrastructure/
│   ├── docker-compose.yml
│   ├── k8s/               # Kubernetes manifests
│   └── terraform/         # Infrastructure as code
└── docs/
    ├── api/               # OpenAPI specs
    └── architecture/      # Architecture decisions
\`\`\`

## Events

### Event Catalog

| Event | Producer | Consumers |
|-------|----------|-----------|
| `order.created` | Orders | Inventory, Payments |
| `order.cancelled` | Orders | Inventory, Payments |
| `payment.completed` | Payments | Orders |
| `inventory.reserved` | Inventory | Orders |

### Event Schema
\`\`\`typescript
interface OrderCreatedEvent {
  type: 'order.created';
  data: {
    orderId: string;
    userId: string;
    items: Array<{ productId: string; quantity: number }>;
    totalAmount: number;
  };
  metadata: {
    timestamp: string;
    correlationId: string;
  };
}
\`\`\`

## Service Communication

### Sync (HTTP)
- Gateway → Services for queries
- Service → Service for immediate needs

### Async (Kafka)
- State changes
- Cross-service notifications
- Saga orchestration

## Commands

\`\`\`bash
# All services
pnpm dev                  # Start all
pnpm build                # Build all
pnpm test                 # Test all

# Single service
pnpm --filter @oms/orders dev

# Infrastructure
docker-compose up -d      # Start infra
docker-compose logs -f kafka  # View Kafka logs

# Kafka
pnpm kafka:topics         # Create topics
pnpm kafka:console        # Consumer console
\`\`\`

## Deployment

### Local (Docker Compose)
\`\`\`bash
docker-compose up --build
\`\`\`

### Kubernetes
\`\`\`bash
kubectl apply -f infrastructure/k8s/
\`\`\`

## Observability

- **Tracing**: Jaeger (trace requests across services)
- **Metrics**: Prometheus + Grafana
- **Logging**: ELK Stack (centralized logs)
- **Health**: `/health` endpoint on each service
```

### Example 6: Enterprise Compliance

```markdown
# HealthGuard - Healthcare Management Platform

HIPAA-compliant healthcare management system with SOC 2 Type II certification.

## Compliance Requirements

| Standard | Status | Last Audit |
|----------|--------|------------|
| HIPAA | Compliant | 2024-01 |
| SOC 2 Type II | Certified | 2024-02 |
| GDPR | Compliant | 2024-01 |

## Tech Stack

- **Backend**: Java 17, Spring Boot 3
- **Frontend**: React 18, TypeScript
- **Database**: PostgreSQL 15 (encrypted at rest)
- **Queue**: RabbitMQ (TLS)
- **Cache**: Redis (encrypted)
- **Auth**: Keycloak (OIDC)

## Security Requirements

### Data Classification

| Level | Examples | Requirements |
|-------|----------|--------------|
| PHI | Patient records, diagnoses | Encryption, audit logging, access control |
| PII | Names, addresses, SSN | Encryption, access control |
| Confidential | Internal reports | Access control |
| Public | Marketing content | None |

### Code Requirements

1. **Never log PHI/PII** - Use `[REDACTED]` placeholders
2. **Always encrypt** - PHI at rest and in transit
3. **Audit everything** - All data access must be logged
4. **Validate inputs** - All external data must be validated
5. **Principle of least privilege** - Minimal permissions

## Quick Start

\`\`\`bash
# Prerequisites: Java 17, Node 20, Docker

# Start infrastructure (encrypted)
docker-compose -f docker-compose.secure.yml up -d

# Backend
cd backend
./mvnw spring-boot:run -Dspring.profiles.active=local

# Frontend
cd frontend
npm install
npm run dev
\`\`\`

## Project Structure

\`\`\`
├── backend/
│   ├── src/main/java/com/healthguard/
│   │   ├── config/
│   │   │   ├── SecurityConfig.java    # Auth configuration
│   │   │   └── AuditConfig.java       # Audit logging
│   │   ├── domain/
│   │   │   ├── patient/               # Patient domain
│   │   │   └── appointment/           # Appointment domain
│   │   ├── security/
│   │   │   ├── audit/                 # Audit trail
│   │   │   └── encryption/            # Data encryption
│   │   └── compliance/
│   │       ├── hipaa/                 # HIPAA controls
│   │       └── gdpr/                  # GDPR controls
│   └── src/test/
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── hooks/
│   │   └── services/
│   └── cypress/                       # E2E tests
└── compliance/
    ├── policies/                      # Security policies
    ├── runbooks/                      # Incident response
    └── audits/                        # Audit documentation
\`\`\`

## Conventions

### PHI Handling

\`\`\`java
// NEVER do this
log.info("Patient data: " + patient.toString());

// DO this
log.info("Accessing patient record: {}", patient.getId());
auditService.log(AuditEvent.PATIENT_ACCESSED, patient.getId(), currentUser());
\`\`\`

### Encryption

\`\`\`java
// All PHI fields must use @Encrypted annotation
@Entity
public class Patient {
    @Id
    private UUID id;

    @Encrypted
    private String ssn;

    @Encrypted
    private String medicalRecordNumber;
}
\`\`\`

### Access Control

\`\`\`java
// All endpoints must declare required roles
@GetMapping("/patients/{id}")
@PreAuthorize("hasRole('PROVIDER') or hasRole('ADMIN')")
@Audited(action = "VIEW_PATIENT")
public Patient getPatient(@PathVariable UUID id) {
    return patientService.findById(id);
}
\`\`\`

## Testing Requirements

| Type | Coverage | Requirement |
|------|----------|-------------|
| Unit | 80% | All business logic |
| Integration | 70% | All API endpoints |
| Security | 100% | All auth flows |
| Compliance | 100% | All HIPAA controls |

\`\`\`bash
# Run all tests with compliance checks
./mvnw verify -Pcompliance

# Security scan
./mvnw dependency-check:check
\`\`\`

## Incident Response

If you discover a security issue:
1. **Do not** commit or discuss in public channels
2. **Do** report to security@healthguard.com
3. **Do** document in secure incident tracker
4. See `compliance/runbooks/incident-response.md`

## Environment Variables

All sensitive configuration via HashiCorp Vault:
- Database credentials
- API keys
- Encryption keys

**Never commit secrets to the repository.**

@import ./compliance/security-checklist.md
```

### Example 7: Open Source Project

```markdown
# CloudQuery - Cloud Resource Query Language

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![CI](https://github.com/org/cloudquery/actions/workflows/ci.yml/badge.svg)](actions)

Query your cloud resources using SQL-like syntax.

## Features

- Multi-cloud support (AWS, GCP, Azure)
- SQL-like query language
- Plugin architecture for extensibility
- Blazing fast with Rust core

## Quick Start

\`\`\`bash
# Install via Homebrew
brew install cloudquery

# Or via npm
npm install -g cloudquery

# Configure provider
cloudquery init aws

# Run a query
cloudquery query "SELECT * FROM ec2_instances WHERE state = 'running'"
\`\`\`

## Project Structure

\`\`\`
├── core/                    # Rust core engine
│   ├── src/
│   │   ├── parser/          # Query parser
│   │   ├── planner/         # Query planner
│   │   ├── executor/        # Query executor
│   │   └── providers/       # Provider interface
│   └── Cargo.toml
├── cli/                     # CLI application (Node.js)
│   ├── src/
│   └── package.json
├── providers/               # Cloud providers
│   ├── aws/
│   ├── gcp/
│   └── azure/
├── plugins/                 # Plugin examples
├── docs/                    # Documentation
│   ├── getting-started.md
│   ├── query-syntax.md
│   └── providers/
└── examples/                # Example queries
\`\`\`

## Development Setup

\`\`\`bash
# Prerequisites: Rust 1.70+, Node 20+

# Clone the repository
git clone https://github.com/org/cloudquery
cd cloudquery

# Build core
cd core && cargo build

# Build CLI
cd ../cli && npm install && npm run build

# Run tests
cargo test
npm test
\`\`\`

## Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

### Contribution Process

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Add tests for new functionality
5. Ensure all tests pass (`make test`)
6. Commit with conventional commits (`feat: add amazing feature`)
7. Push to your fork
8. Open a Pull Request

### Code Style

- **Rust**: Follow rustfmt and clippy suggestions
- **TypeScript**: Prettier + ESLint (config included)
- **Documentation**: Required for public APIs

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):
- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation
- `test:` Tests
- `refactor:` Code refactoring
- `chore:` Maintenance

## Commands

\`\`\`bash
# Build everything
make build

# Run tests
make test

# Lint
make lint

# Generate docs
make docs

# Release (maintainers only)
make release VERSION=1.2.3
\`\`\`

## Architecture

See [ARCHITECTURE.md](docs/ARCHITECTURE.md) for detailed design documentation.

## Roadmap

- [x] AWS provider
- [x] Query parser
- [ ] GCP provider (in progress)
- [ ] Azure provider
- [ ] Web UI
- [ ] Plugin marketplace

## License

MIT License - see [LICENSE](LICENSE) for details.

## Community

- [Discord](https://discord.gg/cloudquery)
- [Twitter](https://twitter.com/cloudquery)
- [GitHub Discussions](https://github.com/org/cloudquery/discussions)

## Acknowledgments

Built with amazing open source projects:
- [Rust](https://www.rust-lang.org/)
- [Node.js](https://nodejs.org/)
- [SQLite](https://sqlite.org/)
```

---

## Patterns

### Pattern 1: Use @import for Large Sections

```markdown
# Project

## Overview
[Keep in main file]

## Architecture
@import ./docs/architecture.md

## API Reference
@import ./docs/api-reference.md
```

Keep CLAUDE.md focused; import detailed docs.

### Pattern 2: Tech Stack Table

```markdown
## Tech Stack

| Layer | Technology | Version |
|-------|------------|---------|
| Frontend | React | 18.2 |
| Backend | Node.js | 20 LTS |
| Database | PostgreSQL | 15 |
```

Clear, scannable format.

### Pattern 3: Command Reference Table

```markdown
## Commands

| Command | Description |
|---------|-------------|
| `npm run dev` | Start development |
| `npm test` | Run tests |
| `npm run build` | Production build |
```

Quick reference format.

### Pattern 4: Structured Conventions

```markdown
## Conventions

### Naming
- Components: PascalCase
- Files: kebab-case
- Variables: camelCase

### Structure
- One component per file
- Tests next to source
```

Organized by category.

### Pattern 5: Visual Architecture

```markdown
## Architecture

\`\`\`
Client → API Gateway → Services → Database
            ↓
        Auth Layer
\`\`\`
```

Quick visual overview.

---

## Anti-Patterns

### Anti-Pattern 1: Including Obvious Programming Practices

```markdown
# BAD
## Conventions
- Use meaningful variable names
- Write comments for complex code
- Don't repeat yourself (DRY)
```

```markdown
# GOOD
## Conventions
- Use `useQuery` hook for server state
- Prefix boolean props with `is`/`has`
- Components in `src/components/ui/` are design-system primitives
```

**Why**: Claude knows general best practices. Include project-specific rules only.

### Anti-Pattern 2: Stale/Outdated Information

```markdown
# BAD
## Commands
- `npm run dev` - Start development
- `grunt build` - Build (NOTE: We migrated from Grunt to Webpack in 2019)
```

**Why**: Outdated info causes confusion. Keep CLAUDE.md current.

### Anti-Pattern 3: Excessive Length

```markdown
# BAD
[2000+ lines of documentation including full API specs, database schemas,
complete style guide, every decision ever made...]
```

```markdown
# GOOD
# Project

## Quick Reference
[Essential info]

## Detailed Documentation
@import ./docs/api.md
@import ./docs/architecture.md
```

**Why**: Long files are hard to maintain and slow to process.

### Anti-Pattern 4: Missing Tech Stack

```markdown
# BAD
# Project

## About
This project does things.

## Commands
npm run dev
```

```markdown
# GOOD
# Project

## Tech Stack
- TypeScript, React, Next.js
- PostgreSQL, Prisma
- Jest, Playwright

## Commands
...
```

**Why**: Claude needs to know the technology to give relevant advice.

### Anti-Pattern 5: Including Secrets

```markdown
# BAD
## Configuration
DATABASE_URL=postgres://user:password@localhost/db
API_KEY=sk_live_abc123
```

```markdown
# GOOD
## Configuration
See `.env.example` for required environment variables.
```

**Why**: CLAUDE.md is committed to version control. Never include secrets.

### Anti-Pattern 6: Duplicating README

```markdown
# BAD
[Entire contents of README.md copy-pasted]
```

```markdown
# GOOD
# Project

## For Claude
[Claude-specific context]

## General Documentation
See README.md for installation and usage.
```

**Why**: CLAUDE.md should complement README, not duplicate it.

---

## Troubleshooting

### CLAUDE.md Not Being Read

**Symptoms**: Claude doesn't know project context

**Causes**:
1. File not named exactly `CLAUDE.md` (case-sensitive)
2. File not in project root
3. File has syntax errors

**Solution**:
```bash
# Verify location
ls CLAUDE.md

# Check for BOM or encoding issues
file CLAUDE.md
```

### @import Not Working

**Symptoms**: Imported content not available

**Causes**:
1. Path doesn't exist
2. Path is absolute instead of relative
3. Circular imports

**Solution**:
```markdown
# Use relative paths from CLAUDE.md location
@import ./docs/architecture.md    # Good
@import /docs/architecture.md     # Bad (absolute)
@import ../other/file.md          # Be careful with parent dirs
```

### Context Too Large

**Symptoms**: Slow responses, truncation

**Causes**:
1. CLAUDE.md too long
2. Too many imports
3. Very detailed content

**Solution**:
- Keep CLAUDE.md under 500 lines
- Use @import strategically
- Summarize instead of including everything

---

## Cheat Sheet

### File Location
```
./CLAUDE.md  (project root)
```

### Import Syntax
```markdown
@import ./path/to/file.md
```

### Essential Sections
```markdown
# Project Name

## Tech Stack
[Languages, frameworks, tools]

## Commands
[Build, test, run]

## Structure
[Key directories]

## Conventions
[Project-specific rules]
```

### Recommended Size
- Target: 100-300 lines
- Maximum: 500 lines
- Use @import for more

---

## Related Documentation

- [Subagents](./01_SUBAGENTS.md) - For parallel specialized work
- [Skills](./02_SKILLS.md) - For repeatable workflows
- [Modular Rules](./04_MODULAR_RULES.md) - For path-specific standards
- [Official Docs](https://code.claude.com/docs/memory)
