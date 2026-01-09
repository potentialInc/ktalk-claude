# Docker Compose Templates

## Complete Template Structure

```yaml
version: '3.8'

services:
  # ==================== Database ====================
  postgres:
    image: postgres:16-alpine
    container_name: ${PROJECT_NAME:-monorepo}-postgres
    restart: unless-stopped
    ports:
      - '${POSTGRES_PORT:-5432}:5432'
    environment:
      POSTGRES_USER: ${POSTGRES_USER:-postgres}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-postgres}
      POSTGRES_DB: ${POSTGRES_DATABASE:-app_db}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app-network
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U ${POSTGRES_USER:-postgres}']
      interval: 10s
      timeout: 5s
      retries: 5

  postgres-test:
    image: postgres:16-alpine
    container_name: ${PROJECT_NAME:-monorepo}-postgres-test
    restart: unless-stopped
    ports:
      - '5433:5432'
    environment:
      POSTGRES_USER: test_user
      POSTGRES_PASSWORD: test_password
      POSTGRES_DB: app_test_db
    volumes:
      - postgres_test_data:/var/lib/postgresql/data
    networks:
      - app-network
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U test_user']
      interval: 10s
      timeout: 5s
      retries: 5
    profiles:
      - test

  # ==================== Backend ====================
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    container_name: ${PROJECT_NAME:-monorepo}-backend
    restart: unless-stopped
    ports:
      - '${BACKEND_PORT:-4000}:4000'
    environment:
      NODE_ENV: development
      PORT: 4000
      POSTGRES_HOST: postgres
      POSTGRES_PORT: 5432
      POSTGRES_USER: ${POSTGRES_USER:-postgres}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-postgres}
      POSTGRES_DATABASE: ${POSTGRES_DATABASE:-app_db}
    env_file:
      - ./backend/.env
    volumes:
      - ./backend:/app
      - /app/node_modules
      - ./backend/logs:/app/logs
    depends_on:
      postgres:
        condition: service_healthy
    networks:
      - app-network
    profiles:
      - dev

  backend-prod:
    build:
      context: ./backend
      dockerfile: Dockerfile
      target: production
    container_name: ${PROJECT_NAME:-monorepo}-backend-prod
    restart: unless-stopped
    ports:
      - '${BACKEND_PORT:-4000}:4000'
    environment:
      NODE_ENV: production
      PORT: 4000
      POSTGRES_HOST: postgres
      POSTGRES_PORT: 5432
    env_file:
      - ./backend/.env
    volumes:
      - ./backend/logs:/app/logs
    depends_on:
      postgres:
        condition: service_healthy
    networks:
      - app-network
    profiles:
      - prod

  # ==================== Frontend App ====================
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    container_name: ${PROJECT_NAME:-monorepo}-frontend
    restart: unless-stopped
    ports:
      - '${FRONTEND_PORT:-3000}:3000'
    environment:
      NODE_ENV: development
      VITE_API_URL: http://localhost:4000
    volumes:
      - ./frontend:/app
      - /app/node_modules
    depends_on:
      - backend
    networks:
      - app-network
    profiles:
      - dev

  frontend-prod:
    build:
      context: ./frontend
      dockerfile: Dockerfile
      target: production
    container_name: ${PROJECT_NAME:-monorepo}-frontend-prod
    restart: unless-stopped
    ports:
      - '${FRONTEND_PORT:-3000}:3000'
    environment:
      NODE_ENV: production
    depends_on:
      - backend-prod
    networks:
      - app-network
    profiles:
      - prod

  # ==================== Dashboard (Optional) ====================
  frontend-dashboard:
    build:
      context: ./frontend-dashboard
      dockerfile: Dockerfile.dev
    container_name: ${PROJECT_NAME:-monorepo}-dashboard
    restart: unless-stopped
    ports:
      - '${DASHBOARD_PORT:-3001}:3000'
    environment:
      NODE_ENV: development
      VITE_API_URL: http://localhost:4000
    volumes:
      - ./frontend-dashboard:/app
      - /app/node_modules
    depends_on:
      - backend
    networks:
      - app-network
    profiles:
      - dev

  frontend-dashboard-prod:
    build:
      context: ./frontend-dashboard
      dockerfile: Dockerfile
      target: production
    container_name: ${PROJECT_NAME:-monorepo}-dashboard-prod
    restart: unless-stopped
    ports:
      - '${DASHBOARD_PORT:-3001}:3000'
    environment:
      NODE_ENV: production
    depends_on:
      - backend-prod
    networks:
      - app-network
    profiles:
      - prod

networks:
  app-network:
    driver: bridge

volumes:
  postgres_data:
  postgres_test_data:
```

---

## Service-Specific Templates

### NestJS Backend

```yaml
backend:
  build:
    context: ./backend
    dockerfile: Dockerfile.dev
  ports:
    - '4000:4000'
  environment:
    NODE_ENV: development
    PORT: 4000
    # Database
    POSTGRES_HOST: postgres
    POSTGRES_PORT: 5432
    POSTGRES_USER: ${POSTGRES_USER:-postgres}
    POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-postgres}
    POSTGRES_DATABASE: ${POSTGRES_DATABASE:-app_db}
    # JWT
    AUTH_JWT_SECRET: ${AUTH_JWT_SECRET:-dev-secret}
  volumes:
    - ./backend:/app
    - /app/node_modules
  depends_on:
    postgres:
      condition: service_healthy
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:4000/health"]
    interval: 30s
    timeout: 10s
    retries: 3
```

### React Router 7 Frontend

```yaml
frontend:
  build:
    context: ./frontend
    dockerfile: Dockerfile.dev
  ports:
    - '3000:3000'
  environment:
    NODE_ENV: development
    VITE_API_URL: http://localhost:4000
  volumes:
    - ./frontend:/app
    - /app/node_modules
    - /app/.react-router
  depends_on:
    - backend
```

### Vite React Frontend

```yaml
frontend:
  build:
    context: ./frontend
    dockerfile: Dockerfile.dev
  ports:
    - '5173:5173'
  environment:
    NODE_ENV: development
    VITE_API_URL: http://localhost:4000
  volumes:
    - ./frontend:/app
    - /app/node_modules
  command: npm run dev -- --host 0.0.0.0
```

---

## Environment Variable Strategy

### Root .env File Template

```env
# Project
PROJECT_NAME=my-monorepo

# Database
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DATABASE=app_db
POSTGRES_PORT=5432

# Backend
BACKEND_PORT=4000
AUTH_JWT_SECRET=your-secret-key

# Frontend
FRONTEND_PORT=3000
DASHBOARD_PORT=3001
```

### Service-Specific .env Files

Each service can have its own `.env`:
- `./backend/.env`
- `./frontend/.env`
- `./frontend-dashboard/.env`

Use `env_file` directive to load these:
```yaml
env_file:
  - ./backend/.env
```

---

## Profile Usage

### Development Profile

```bash
docker compose --profile dev up
```

Starts: postgres, backend, frontend, frontend-dashboard

### Production Profile

```bash
docker compose --profile prod up
```

Starts: postgres, backend-prod, frontend-prod, frontend-dashboard-prod

### Test Profile

```bash
docker compose --profile test up -d postgres-test
```

Starts: postgres-test (isolated test database)

### Combined Profiles

```bash
docker compose --profile dev --profile test up
```

---

## Network Configuration

All services use a single bridge network:

```yaml
networks:
  app-network:
    driver: bridge
```

Services communicate via container names:
- Backend connects to `postgres:5432`
- Frontend connects to `backend:4000` (internal) or `localhost:4000` (browser)

---

## Volume Patterns

### Named Volumes (Data Persistence)

```yaml
volumes:
  postgres_data:
  postgres_test_data:
```

### Bind Mounts (Development)

```yaml
volumes:
  - ./backend:/app           # Source code
  - /app/node_modules        # Preserve container node_modules
  - ./backend/logs:/app/logs # Persistent logs
```

### Why Exclude node_modules?

```yaml
- /app/node_modules
```

This creates an anonymous volume that:
1. Prevents host node_modules from overwriting container's
2. Allows different platform-specific binaries
3. Improves performance on macOS/Windows

---

## Health Check Patterns

### PostgreSQL

```yaml
healthcheck:
  test: ['CMD-SHELL', 'pg_isready -U ${POSTGRES_USER:-postgres}']
  interval: 10s
  timeout: 5s
  retries: 5
```

### NestJS Backend

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:4000/health"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 40s
```

### React Frontend

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:3000"]
  interval: 30s
  timeout: 10s
  retries: 3
```

---

## Dependency Management

### Conditional Start

```yaml
depends_on:
  postgres:
    condition: service_healthy
```

Ensures postgres is healthy before starting backend.

### Service Dependencies

```
postgres (healthy) → backend → frontend
                            → frontend-dashboard
```
