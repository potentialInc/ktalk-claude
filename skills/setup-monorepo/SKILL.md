---
skill_name: setup-monorepo
applies_to_local_project_only: true
tags: [monorepo, docker, setup, infrastructure, devops]
related_skills: [backend-dev-guidelines, frontend-dev-guidelines]
---

# Setup Monorepo Skill

## Purpose

Initialize a monorepo development environment by cloning multiple GitHub repositories and creating a unified Docker Compose configuration with all services integrated.

## When to Use This Skill

Use this skill when you need to:
- Set up a new monorepo from existing repositories
- Clone multiple repos into a unified project structure
- Create a unified docker-compose.yml for multiple services
- Generate development infrastructure (Makefile, Docker configs)

---

## Quick Start

### Invoke via Command

```bash
/setup-monorepo backend=https://github.com/org/backend frontend-app=https://github.com/org/frontend
```

### With All Three Services

```bash
/setup-monorepo backend=https://github.com/org/backend frontend-app=https://github.com/org/frontend frontend-dashboard=https://github.com/org/dashboard
```

### Interactive Mode

Run without arguments to be prompted for each URL:
```bash
/setup-monorepo
```

---

## Workflow Overview

### Phase 1: Repository Cloning

1. Parse provided GitHub URLs from arguments
2. Validate URL format and accessibility
3. Clone each repository to target directory:
   - `backend=` → `./backend/`
   - `frontend-app=` → `./frontend/`
   - `frontend-dashboard=` → `./frontend-dashboard/`
4. Remove `.claude/` from clones (keep only root config)
5. Remove `.git/` to detach from original repos

### Phase 2: Dockerfile Analysis

For each cloned service, analyze in order:

1. **Dockerfile / Dockerfile.dev** - Extract base image, ports, commands
2. **docker-compose.yml** - Extract service configuration hints
3. **package.json** - Extract dependencies, scripts, detect framework

Information extracted:
- Base image (e.g., `node:20-alpine`)
- Exposed ports
- Environment variables needed
- Build and start commands
- Health check configurations

### Phase 3: Docker Compose Generation

Create unified `docker-compose.yml` with:
- PostgreSQL database (with health checks)
- All cloned services configured
- Profile-based activation (dev/prod/test)
- Shared network for service communication
- Volume mounts for development hot-reload

### Phase 4: Makefile Generation

Generate project Makefile with:
- Installation commands
- Development startup commands
- Build and test commands
- Docker management commands

### Phase 5: Documentation Update

Update `.claude/docs/PROJECT_KNOWLEDGE.md`:
- Tech stack from all services
- Service port mappings
- Development setup instructions

---

## Service Configuration

### Default Port Assignments

| Service | Dev Port | Profile |
|---------|----------|---------|
| postgres | 5432 | default |
| postgres-test | 5433 | test |
| backend | 4000 | dev, prod |
| frontend | 3000 | dev, prod |
| frontend-dashboard | 3001 | dev, prod |

### Development Service Pattern

```yaml
service-name:
  build:
    context: ./service-dir
    dockerfile: Dockerfile.dev
  ports:
    - '${PORT:-default}:internal'
  environment:
    NODE_ENV: development
  volumes:
    - ./service-dir:/app
    - /app/node_modules  # Exclude node_modules
  depends_on:
    postgres:
      condition: service_healthy
  networks:
    - app-network
  profiles:
    - dev
```

---

## Error Handling

### Missing Dockerfile

When no Dockerfile is found, a default is generated based on the detected framework:

**NestJS Backend:**
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
EXPOSE 4000
CMD ["npm", "run", "start:dev"]
```

**React Frontend:**
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
EXPOSE 3000
CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]
```

### Port Conflicts

If ports conflict:
1. Default ports are assigned first
2. Conflicts are resolved by incrementing port numbers
3. User is warned about any port changes

### Clone Failures

1. URL format is validated
2. Repository accessibility is checked
3. SSH vs HTTPS alternatives are suggested if one fails

---

## Generated Files

After running the skill, these files are created/updated:

| File | Purpose |
|------|---------|
| `docker-compose.yml` | Unified service configuration |
| `Makefile` | Development commands |
| `.claude/docs/PROJECT_KNOWLEDGE.md` | Updated documentation |
| `*/Dockerfile.dev` | Generated if missing |

---

## Example Output

```
Monorepo Setup Complete!

Services Configured:
  - postgres (port 5432)
  - backend (port 4000)
  - frontend (port 3000)
  - frontend-dashboard (port 3001)

Files Created/Updated:
  - docker-compose.yml
  - Makefile
  - .claude/docs/PROJECT_KNOWLEDGE.md

Next Steps:
1. Create .env file with required variables
2. Run: make install
3. Run: make dev
4. Access services at configured ports
```

---

## Reference Files

For detailed implementation guidance:

- [Dockerfile Analysis Guide](resources/dockerfile-analysis.md) - How to parse and analyze Dockerfiles
- [Docker Compose Templates](resources/docker-compose-template.md) - Complete service templates
- [Documentation Strategy](resources/documentation-strategy.md) - How to update project docs
