---
name: ci-cd
description: CI/CD pipeline patterns for GitHub Actions, GitLab CI, etc.
---

# CI/CD Pipeline Patterns

## GitHub Actions

### Basic Node.js Pipeline

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [18, 20]

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Type check
        run: npm run typecheck

      - name: Test
        run: npm test -- --coverage

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        if: matrix.node-version == 20

  build:
    runs-on: ubuntu-latest
    needs: test

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - run: npm ci
      - run: npm run build

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build
          path: dist/
```

### Deploy Pipeline

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - run: npm ci
      - run: npm run build

      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
```

### Docker Build & Push

```yaml
name: Docker

on:
  push:
    tags: ['v*']

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/${{ github.repository }}
          tags: |
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

## GitLab CI

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

variables:
  NODE_VERSION: "20"

.node-template:
  image: node:${NODE_VERSION}-alpine
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/

test:
  extends: .node-template
  stage: test
  script:
    - npm ci
    - npm run lint
    - npm run typecheck
    - npm test -- --coverage
  coverage: '/Lines\s*:\s*(\d+\.?\d*)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

build:
  extends: .node-template
  stage: build
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

deploy:
  stage: deploy
  image: alpine:latest
  only:
    - main
  environment:
    name: production
    url: https://myapp.com
  script:
    - apk add --no-cache curl
    - curl -X POST $DEPLOY_WEBHOOK_URL
```

## Pipeline Best Practices

### Caching

```yaml
# GitHub Actions - npm cache
- uses: actions/setup-node@v4
  with:
    cache: 'npm'

# Explicit cache
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

### Secrets Management

```yaml
# Use GitHub Secrets
env:
  API_KEY: ${{ secrets.API_KEY }}

# Use environment-specific secrets
jobs:
  deploy:
    environment: production  # Secrets scoped to this environment
    steps:
      - run: echo ${{ secrets.PROD_API_KEY }}
```

### Conditional Jobs

```yaml
jobs:
  deploy:
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'

  # Only on PRs
  preview:
    if: github.event_name == 'pull_request'

  # Skip if commit message contains [skip ci]
  test:
    if: "!contains(github.event.head_commit.message, '[skip ci]')"
```

### Matrix Builds

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    node: [18, 20, 22]
    exclude:
      - os: windows-latest
        node: 18
    include:
      - os: ubuntu-latest
        node: 20
        coverage: true
```

## CI/CD Checklist

- [ ] Tests run on every PR
- [ ] Linting and type checking included
- [ ] Build artifacts uploaded
- [ ] Caching configured for dependencies
- [ ] Secrets not exposed in logs
- [ ] Deploy requires approval for production
- [ ] Notifications on failure
- [ ] Branch protection rules in place
