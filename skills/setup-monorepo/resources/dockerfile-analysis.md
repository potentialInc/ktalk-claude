# Dockerfile Analysis Guide

## Overview

This guide describes how to analyze Dockerfiles from cloned repositories to extract configuration for the unified docker-compose.yml.

---

## Analysis Priority

Analyze files in this order:
1. `Dockerfile` - Production configuration
2. `Dockerfile.dev` - Development configuration
3. `docker-compose.yml` - Existing compose config
4. `package.json` - Scripts and dependencies

---

## Extraction Patterns

### Base Image

```dockerfile
FROM node:20-alpine
FROM node:20-alpine AS builder
```

**Extract**: `node:20-alpine`

### Exposed Ports

```dockerfile
EXPOSE 4000
EXPOSE 3000
```

**Extract**: Port numbers for compose configuration

### Working Directory

```dockerfile
WORKDIR /app
```

**Extract**: For volume mount paths

### Build Commands

```dockerfile
RUN npm ci
RUN npm run build
```

**Extract**: Build steps for multi-stage builds

### Environment Variables

```dockerfile
ENV NODE_ENV=production
ENV PORT=4000
```

**Extract**: Required environment variables

### Entrypoint/CMD

```dockerfile
CMD ["npm", "run", "start:prod"]
CMD ["node", "dist/main.js"]
```

**Extract**: Start commands for compose

---

## Handling Missing Dockerfiles

### Detection Logic

```
if (exists Dockerfile) -> use Dockerfile
else if (exists Dockerfile.dev) -> use Dockerfile.dev
else if (exists package.json) -> generate default Node.js Dockerfile
else -> skip service with warning
```

### Default Node.js Dockerfile (Development)

```dockerfile
FROM node:20-alpine

WORKDIR /app

# Install dependencies first (better caching)
COPY package*.json ./
RUN npm install

# Copy source code
COPY . .

# Default port (override in compose)
EXPOSE 3000

# Development command
CMD ["npm", "run", "dev"]
```

### Default NestJS Backend Dockerfile

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

EXPOSE 4000

CMD ["npm", "run", "start:dev"]
```

### Default React/Vite Frontend Dockerfile

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

EXPOSE 3000

CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]
```

---

## Package.json Analysis

When no Dockerfile exists, analyze package.json:

### Port Detection

```json
{
  "scripts": {
    "dev": "vite --port 5173",
    "start:dev": "nest start --watch"
  }
}
```

**Common patterns:**
- `--port XXXX` in scripts
- Default NestJS: 3000 or 4000
- Default Vite/React: 5173 or 3000
- Default Next.js: 3000

### Framework Detection

| Pattern | Framework | Default Port |
|---------|-----------|--------------|
| `@nestjs/core` | NestJS | 4000 |
| `react-router` | React Router 7 | 3000 |
| `next` | Next.js | 3000 |
| `vite` | Vite | 5173 |
| `express` | Express | 3000 |

### Dependency Version Extraction

```json
{
  "dependencies": {
    "@nestjs/core": "^11.0.0",
    "react": "^19.0.0",
    "typeorm": "^0.3.0"
  }
}
```

**Extract for documentation**: Technology name and version

---

## Multi-Stage Build Detection

Look for patterns:
```dockerfile
FROM node:20-alpine AS builder
...
FROM node:20-alpine AS production
COPY --from=builder ...
```

**If detected:**
- Use appropriate target in compose
- Reference correct stage for dev/prod profiles

**Production compose:**
```yaml
build:
  context: ./service
  dockerfile: Dockerfile
  target: production
```

---

## Volume Mount Strategy

### Development Mounts

```yaml
volumes:
  - ./service:/app           # Source code
  - /app/node_modules        # Exclude node_modules
  - ./service/logs:/app/logs # Persist logs
```

### Production Mounts

```yaml
volumes:
  - ./service/logs:/app/logs  # Logs only
```

---

## Health Check Extraction

### From Dockerfile

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:4000/health || exit 1
```

### Default Health Checks by Framework

**NestJS Backend:**
```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:4000/health"]
  interval: 30s
  timeout: 10s
  retries: 3
```

**React Frontend:**
```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:3000"]
  interval: 30s
  timeout: 10s
  retries: 3
```

---

## Environment Variable Collection

### From Dockerfile

```dockerfile
ENV NODE_ENV=production
ENV PORT=4000
ENV DATABASE_URL=postgresql://...
```

### From .env.example

If `.env.example` exists, parse for required variables:

```env
# Database
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DATABASE=app_db

# JWT
AUTH_JWT_SECRET=your-secret

# API
PORT=4000
```

### Common Variables by Service Type

**Backend:**
- `NODE_ENV`
- `PORT`
- `POSTGRES_HOST`, `POSTGRES_PORT`, `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DATABASE`
- `AUTH_JWT_SECRET`

**Frontend:**
- `NODE_ENV`
- `VITE_API_URL` or `NEXT_PUBLIC_API_URL`
