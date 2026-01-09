# Documentation Update Strategy

## Overview

After setting up the monorepo, update `.claude/docs/` to reflect the new structure and configuration.

---

## Files to Update

### Primary: PROJECT_KNOWLEDGE.md

The main documentation file that should contain:
- Tech stack from all services
- Service descriptions and ports
- Development setup instructions

---

## Update Workflow

### Step 1: Read Existing Docs

```bash
# Check what exists
ls -la .claude/docs/
```

If `PROJECT_KNOWLEDGE.md` exists, preserve existing content and merge new information.

### Step 2: Extract Service Info

From each cloned service, gather:

**From package.json:**
- Package name and version
- Dependencies with versions
- Available scripts
- Framework detection

**From Dockerfile analysis:**
- Exposed ports
- Base images
- Build commands

**From docker-compose.yml:**
- Service names
- Port mappings
- Environment variables

### Step 3: Merge Information

Combine with existing documentation:
- Preserve existing sections
- Add new service information
- Update outdated information
- Keep cross-references valid

### Step 4: Update Timestamp

Always update the "Last updated" date:
```markdown
> Last updated: YYYY-MM-DD
```

---

## Section Templates

### Tech Stack Section

```markdown
## Tech Stack

### Backend
| Technology | Version | Purpose |
|------------|---------|---------|
| NestJS | 11.x | Framework |
| TypeORM | 0.3.x | Database ORM |
| PostgreSQL | 16 | Database |
| class-validator | 0.14.x | Validation |
| Passport | 0.7.x | Authentication |

### Frontend
| Technology | Version | Purpose |
|------------|---------|---------|
| React | 19.x | UI library |
| React Router | 7.x | Routing & SSR |
| Redux Toolkit | 2.x | State management |
| TanStack Query | 5.x | Data fetching |
| Tailwind CSS | 4.x | Styling |

### Dashboard
| Technology | Version | Purpose |
|------------|---------|---------|
| React | 19.x | UI library |
| Shadcn/UI | latest | Component library |
```

### Services Section

```markdown
## Services

| Service | Port | Profile | Description |
|---------|------|---------|-------------|
| postgres | 5432 | default | PostgreSQL 16 database |
| postgres-test | 5433 | test | Test database instance |
| backend | 4000 | dev | NestJS API server |
| backend-prod | 4000 | prod | Production API |
| frontend | 3000 | dev | React application |
| frontend-prod | 3000 | prod | Production frontend |
| frontend-dashboard | 3001 | dev | Admin dashboard |
| frontend-dashboard-prod | 3001 | prod | Production dashboard |
```

### Development Setup Section

```markdown
## Development Setup

### Prerequisites
- Docker & Docker Compose v2+
- Node.js 20+
- Make (optional, for convenience commands)

### Quick Start

```bash
# Install dependencies
make install

# Start all services in development mode
make dev

# Or without Make:
docker compose --profile dev up
```

### Available Commands

| Command | Description |
|---------|-------------|
| `make install` | Install all dependencies |
| `make dev` | Start all dev services |
| `make dev-backend` | Start backend only |
| `make dev-frontend` | Start frontend only |
| `make build` | Build all services |
| `make test` | Run all tests |
| `make docker-up` | Start Docker services |
| `make docker-down` | Stop Docker services |
| `make clean` | Clean all build artifacts |

### Environment Variables

Copy example files:
```bash
cp backend/.env.example backend/.env
cp frontend/.env.example frontend/.env
```

Required variables:
- `POSTGRES_USER` - Database username
- `POSTGRES_PASSWORD` - Database password
- `AUTH_JWT_SECRET` - JWT signing secret
```

---

## Creating New Documentation

If `PROJECT_KNOWLEDGE.md` doesn't exist, create with this structure:

```markdown
# Project Knowledge

> Monorepo Documentation
> Last updated: YYYY-MM-DD

## Product Overview

[Brief description of the monorepo and its services]

---

## Tech Stack

### Backend
| Technology | Version | Purpose |
|------------|---------|---------|

### Frontend
| Technology | Version | Purpose |
|------------|---------|---------|

---

## Services

| Service | Port | Profile | Description |
|---------|------|---------|-------------|
| postgres | 5432 | default | PostgreSQL database |
| backend | 4000 | dev | API server |
| frontend | 3000 | dev | Web application |

---

## Development Setup

### Prerequisites
- Docker & Docker Compose
- Node.js 20+

### Quick Start
```bash
make dev
```

---

## Project Structure

```
monorepo/
├── .claude/              # Claude Code configuration
│   ├── skills/           # Custom skills
│   ├── docs/             # Project documentation
│   └── commands/         # Slash commands
├── backend/              # NestJS API
├── frontend/             # React application
├── frontend-dashboard/   # Admin dashboard (optional)
├── docker-compose.yml    # Service definitions
└── Makefile             # Development commands
```

---

## Related Documentation

- [PROJECT_API.md](PROJECT_API.md) - API reference
- [PROJECT_DATABASE.md](PROJECT_DATABASE.md) - Database schema
- [BEST_PRACTICES.md](BEST_PRACTICES.md) - Coding standards
```

---

## Version Extraction

### From package.json

```javascript
// Read and extract
const pkg = require('./package.json');

// Dependencies
const deps = {
  ...pkg.dependencies,
  ...pkg.devDependencies
};

// Extract version (remove ^ or ~)
const version = deps['@nestjs/core'].replace(/[\^~]/, '');
// Result: "11.0.0"
```

### Common Package Mappings

| Package | Display Name |
|---------|--------------|
| `@nestjs/core` | NestJS |
| `react` | React |
| `react-router` | React Router |
| `@reduxjs/toolkit` | Redux Toolkit |
| `@tanstack/react-query` | TanStack Query |
| `typeorm` | TypeORM |
| `tailwindcss` | Tailwind CSS |

---

## Cross-Reference Updates

After setup, ensure these files link correctly:

```markdown
## Related Documentation

- [PROJECT_API.md](PROJECT_API.md) - API reference
- [PROJECT_DATABASE.md](PROJECT_DATABASE.md) - Database schema
- [BEST_PRACTICES.md](BEST_PRACTICES.md) - Coding standards
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - Common issues
```

Also link to relevant skills:
```markdown
## Development Guidelines

For detailed development patterns, see:
- [Backend Guidelines](../skills/backend-dev-guidelines/SKILL.md)
- [Frontend Guidelines](../skills/frontend-dev-guidelines/SKILL.md)
```
