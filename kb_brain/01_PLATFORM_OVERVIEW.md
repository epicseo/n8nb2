# 01_PLATFORM_OVERVIEW.md
<!-- repo: n8n | version: 1.122.0 | generated: 2025-12-02 -->
<!-- tags: architecture, configuration, debugging, monorepo, deployment -->

## Contents
- [Repository Overview](#repository-overview)
- [Package Architecture](#package-architecture)
- [System Diagram](#system-diagram)
- [Configuration Reference](#configuration-reference)
- [Debugging Guide](#debugging-guide)

---

## Repository Overview
<!-- chunk: 01-overview | keywords: n8n, workflow, automation, monorepo -->

| Property | Value |
|----------|-------|
| Name | n8n-monorepo |
| Version | 1.122.0 |
| Type | Workflow Automation Platform |
| Structure | pnpm monorepo with Turbo |
| Node.js | >=22.16 |
| Package Manager | pnpm >=10.22.0 |

### Directory Structure

```
n8n/
├── packages/
│   ├── @n8n/                    # Scoped shared packages
│   │   ├── api-types/           # Shared FE/BE TypeScript interfaces
│   │   ├── config/              # Centralized configuration (Zod-based)
│   │   ├── db/                  # TypeORM entities & repositories
│   │   ├── di/                  # Dependency injection container
│   │   ├── nodes-langchain/     # AI/LangChain nodes (118 nodes)
│   │   ├── task-runner/         # JavaScript task execution
│   │   └── task-runner-python/  # Python task execution
│   │
│   ├── cli/                     # Express server, REST API, CLI
│   │   ├── bin/n8n              # Main entry point
│   │   └── src/
│   │       ├── controllers/     # HTTP request handlers
│   │       ├── services/        # Business logic
│   │       └── webhooks/        # Webhook processing
│   │
│   ├── core/                    # Workflow execution engine
│   │   └── src/execution-engine/
│   │
│   ├── frontend/                # Vue 3 frontend
│   │   └── editor-ui/           # Main editor application
│   │
│   ├── nodes-base/              # Built-in integration nodes (531 nodes)
│   │
│   └── workflow/                # Core workflow types & interfaces
│
├── turbo.json                   # Turbo build orchestration
└── pnpm-workspace.yaml          # Workspace configuration
```

---

## Package Architecture
<!-- chunk: 01-packages | keywords: packages, modules, dependencies -->

### Core Packages

| Package | Purpose |
|---------|---------|
| `@n8n/workflow` | Core types, interfaces, expression evaluation |
| `@n8n/core` | Workflow execution engine |
| `@n8n/cli` | Express server, REST API, CLI commands |
| `@n8n/db` | TypeORM entities, repositories, migrations |
| `@n8n/config` | Configuration management (Zod schemas) |
| `@n8n/di` | Dependency injection container |

### Node Packages

| Package | Nodes | Purpose |
|---------|-------|---------|
| `n8n-nodes-base` | 531 | Built-in integration nodes |
| `@n8n/nodes-langchain` | 118 | AI/LangChain integration nodes |

### Technology Stack

| Layer | Technology |
|-------|------------|
| Backend | Node.js >=22.16, TypeScript 5.9, Express 5 |
| Frontend | Vue 3, Vite, Pinia |
| Database | SQLite (dev), PostgreSQL (prod), MySQL |
| Queue | Bull + Redis |
| Testing | Jest, Vitest, Playwright |

---

## System Diagram
<!-- chunk: 01-diagram | keywords: architecture, diagram, flow -->

```
┌─────────────────────────────────────────────────────────────────┐
│                    FRONTEND (Vue 3 + Pinia)                      │
│  Stores: workflows, ui, nodeTypes, settings, rbac               │
└─────────────────────────────────────────────────────────────────┘
                         ↕ REST API / WebSocket
┌─────────────────────────────────────────────────────────────────┐
│                   CLI PACKAGE (Express Server)                   │
│  Controllers → Services → Repositories → Database               │
│                         ↓                                        │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              CORE (Execution Engine)                     │    │
│  │  WorkflowExecute → ExecuteContext → Node.execute()       │    │
│  └─────────────────────────────────────────────────────────┘    │
│                         ↓                                        │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │         NODES (531 base + 118 AI = 649 total)           │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

### Execution Flow

```
1. API Request: POST /workflows/:id/execute
2. Controller → Service → Load workflow from DB
3. Create WorkflowExecute instance
4. For each node: Create context → execute → route output
5. Store result → Return via WebSocket
```

---

## Configuration Reference
<!-- chunk: 01-config | keywords: environment, variables, settings -->

### Server Settings

| Variable | Default | Purpose |
|----------|---------|---------|
| `N8N_HOST` | `localhost` | Host name |
| `N8N_PORT` | `5678` | HTTP port |
| `N8N_PROTOCOL` | `http` | Protocol (http/https) |
| `N8N_ENCRYPTION_KEY` | - | Credential encryption key |
| `GENERIC_TIMEZONE` | `America/New_York` | Default timezone |

### Database Configuration

| Variable | Default | Purpose |
|----------|---------|---------|
| `DB_TYPE` | `sqlite` | Database type: sqlite, postgresdb, mysqldb |
| `DB_POSTGRESDB_HOST` | `localhost` | PostgreSQL host |
| `DB_POSTGRESDB_DATABASE` | `n8n` | Database name |
| `DB_POSTGRESDB_USER` | `postgres` | Username |
| `DB_POSTGRESDB_PASSWORD` | - | Password |
| `DB_POSTGRESDB_POOL_SIZE` | `2` | Connection pool size |

### Execution Settings

| Variable | Default | Purpose |
|----------|---------|---------|
| `EXECUTIONS_MODE` | `regular` | Mode: regular, queue |
| `EXECUTIONS_TIMEOUT` | `-1` | Timeout seconds (-1=unlimited) |
| `EXECUTIONS_DATA_PRUNE` | `true` | Enable pruning |
| `EXECUTIONS_DATA_MAX_AGE` | `336` | Max age hours |
| `N8N_CONCURRENCY_PRODUCTION_LIMIT` | `-1` | Concurrency limit |

### Queue/Redis (for scaling)

| Variable | Default | Purpose |
|----------|---------|---------|
| `QUEUE_BULL_REDIS_HOST` | `localhost` | Redis host |
| `QUEUE_BULL_REDIS_PORT` | `6379` | Redis port |
| `N8N_MULTI_MAIN_SETUP_ENABLED` | `false` | Enable multi-main |

### Logging

| Variable | Default | Purpose |
|----------|---------|---------|
| `N8N_LOG_LEVEL` | `info` | error, warn, info, debug |
| `N8N_LOG_OUTPUT` | `console` | console, file, or both |
| `N8N_LOG_FORMAT` | `text` | text or json |
| `N8N_LOG_SCOPES` | - | Filter by scope |

**Log Scopes:** `concurrency`, `task-runner`, `scaling`, `redis`, `pubsub`, `pruning`, `workflow-activation`, `circuit-breaker`

### Task Runner

| Variable | Default | Purpose |
|----------|---------|---------|
| `N8N_RUNNERS_ENABLED` | `false` | Enable task runners |
| `N8N_RUNNERS_MODE` | `internal` | internal or external |
| `N8N_RUNNERS_MAX_CONCURRENCY` | `10` | Max concurrent tasks |
| `N8N_RUNNERS_TASK_TIMEOUT` | `300` | Task timeout seconds |

---

## Debugging Guide
<!-- chunk: 01-debug | keywords: debug, troubleshooting, errors -->

### Error Types

| Error | Status | Resolution |
|-------|--------|------------|
| `BadRequestError` | 400 | Check request format |
| `UnauthenticatedError` | 401 | Re-authenticate |
| `ForbiddenError` | 403 | Request permissions |
| `NotFoundError` | 404 | Verify resource exists |
| `TooManyRequestsError` | 429 | Implement backoff |
| `InternalServerError` | 500 | Check server logs |

### Execution Errors

| Error | Cause | Resolution |
|-------|-------|------------|
| `NodeCrashedError` | Out of memory | Reduce batch size |
| `WorkflowCrashedError` | Total memory exhausted | Increase `--max-old-space-size` |
| `TaskRunnerOomError` | Runner OOM | Increase `N8N_RUNNERS_MAX_OLD_SPACE_SIZE` |
| `CredentialNotFoundError` | Credential missing | Recreate credential |

### Debug Commands

```bash
# View server info
n8n info

# Test database
n8n db:validate

# Export workflow
n8n export:workflow --id=<id>

# Execute workflow
n8n execute --id=<id>

# Test credentials
n8n credentials:test --id=<id>
```

### Health Checks

```bash
# Liveness
curl http://localhost:5678/healthz

# Readiness (includes DB)
curl http://localhost:5678/healthz/readiness
```

### Recommended Logging

```bash
# Development
N8N_LOG_LEVEL=debug N8N_LOG_OUTPUT=console

# Production
N8N_LOG_LEVEL=warn N8N_LOG_OUTPUT=console,file N8N_LOG_FORMAT=json

# Troubleshooting
N8N_LOG_LEVEL=debug N8N_LOG_SCOPES=task-runner,scaling
```

### Memory Debugging

```bash
# Increase heap size
NODE_OPTIONS="--max-old-space-size=4096" npm start

# For frontend builds
NODE_OPTIONS="--max-old-space-size=8192"
```

---

## CLI Commands

| Command | Purpose |
|---------|---------|
| `n8n start` | Start main server |
| `n8n start --tunnel` | Start with localtunnel |
| `n8n webhook` | Webhook-only server |
| `n8n worker` | Queue worker |
| `n8n execute --id=<id>` | Execute workflow |

## Build Commands

| Command | Purpose |
|---------|---------|
| `pnpm build > build.log 2>&1` | Build all packages |
| `pnpm dev` | Start development |
| `pnpm test` | Run all tests |
| `pnpm lint` | Lint code |
| `pnpm typecheck` | Type checking |

---

*Source: packages/@n8n/config/, packages/cli/src/*
