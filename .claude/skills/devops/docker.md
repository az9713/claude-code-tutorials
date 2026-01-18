---
name: docker
description: Docker and container best practices
---

# Docker Best Practices

## Dockerfile Optimization

### Multi-Stage Builds

```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app

# Install dependencies first (better caching)
COPY package*.json ./
RUN npm ci

# Copy source and build
COPY . .
RUN npm run build

# Production stage
FROM node:20-alpine AS runner
WORKDIR /app

# Don't run as root
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 appuser

# Copy only production files
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./

USER appuser

EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### Layer Caching

```dockerfile
# ❌ Bad: Invalidates cache on any change
COPY . .
RUN npm ci

# ✅ Good: Dependencies cached separately
COPY package*.json ./
RUN npm ci
COPY . .
```

### Image Size Reduction

```dockerfile
# Use slim/alpine base images
FROM node:20-alpine          # ~50MB vs ~350MB for full
FROM python:3.11-slim        # ~150MB vs ~900MB for full

# Clean up in the same layer
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

# Use .dockerignore
# .dockerignore contents:
# node_modules
# .git
# *.md
# .env*
# dist
# coverage
```

## Docker Compose

### Development Setup

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - .:/app
      - /app/node_modules  # Don't overwrite node_modules
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://postgres:postgres@db:5432/app
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:15-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: app
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

### Production Overrides

```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    restart: unless-stopped
    environment:
      - NODE_ENV=production
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
```

## Common Commands

```bash
# Build and run
docker compose up --build

# Run in background
docker compose up -d

# View logs
docker compose logs -f app

# Execute command in container
docker compose exec app sh

# Stop and remove
docker compose down

# Remove volumes too
docker compose down -v

# Build specific service
docker compose build app

# Scale service
docker compose up -d --scale app=3
```

## Health Checks

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1
```

```typescript
// Health endpoint
app.get('/health', (req, res) => {
  const health = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    checks: {
      database: checkDatabase(),
      redis: checkRedis()
    }
  };

  const isHealthy = Object.values(health.checks).every(c => c.status === 'ok');
  res.status(isHealthy ? 200 : 503).json(health);
});
```

## Security Best Practices

```dockerfile
# Run as non-root user
RUN addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G appgroup
USER appuser

# Read-only filesystem where possible
# In docker-compose:
read_only: true
tmpfs:
  - /tmp

# Don't store secrets in images
# Use Docker secrets or environment variables

# Scan images for vulnerabilities
# docker scout cve myimage:latest
```

## Debugging

```bash
# Inspect container
docker inspect <container>

# View resource usage
docker stats

# Check logs
docker logs <container> --tail 100 -f

# Access shell
docker exec -it <container> sh

# Copy files from container
docker cp <container>:/app/logs ./logs
```
