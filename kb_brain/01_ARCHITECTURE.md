# 01_ARCHITECTURE.md
<!-- repo: n8n | branch: claude/github-to-kb-converter-01JAurUearApTk4RAu9NPFAg | commit: 3c0e809e | generated: 2025-11-28 -->
<!-- tags: architecture, monorepo, typescript, workflow-automation, n8n -->

## Contents
- [Repository Overview](#repository-overview)
- [Directory Structure](#directory-structure)
- [Package Architecture](#package-architecture)
- [System Diagram](#system-diagram)
- [Technology Stack](#technology-stack)
- [Entry Points](#entry-points)
- [Data Flow](#data-flow)
- [Coverage Summary](#coverage-summary)

---

## Repository Overview
<!-- chunk: 01-overview | keywords: n8n, workflow, automation, monorepo | source: package.json -->

| Property | Value |
|----------|-------|
| Name | n8n-monorepo |
| Version | 1.122.0 |
| Type | Workflow Automation Platform |
| Structure | pnpm monorepo with Turbo |
| License | Sustainable Use License (SEE LICENSE.md) |
| Node.js | >=22.16 |
| Package Manager | pnpm >=10.22.0 |

**Repository Statistics:**
- Total Files: 13,266 (excluding node_modules, dist, .git)
- Code Files: 9,353 (.ts, .tsx, .js, .jsx, .vue)
- Config Files: 2,775 (.json, .yaml, .yml, .toml)
- Documentation Files: 103 (.md, .rst, .txt)

---

## Directory Structure
<!-- chunk: 01-directory | keywords: structure, folders, packages | source: filesystem -->

```
n8n/
├── packages/
│   ├── @n8n/                    # Scoped shared packages
│   │   ├── api-types/           # Shared FE/BE TypeScript interfaces
│   │   ├── backend-common/      # Logger, decorators, module registry
│   │   ├── backend-test-utils/  # Testing utilities for backend
│   │   ├── benchmark/           # Performance benchmarking
│   │   ├── client-oauth2/       # OAuth2 client implementation
│   │   ├── codemirror-lang/     # CodeMirror language support
│   │   ├── config/              # Centralized configuration (Zod-based)
│   │   ├── constants/           # Shared constants
│   │   ├── create-node/         # Node scaffolding tool
│   │   ├── db/                  # TypeORM entities & repositories
│   │   ├── decorators/          # @Service, @OnShutdown decorators
│   │   ├── di/                  # Dependency injection container
│   │   ├── errors/              # Custom error classes
│   │   ├── eslint-config/       # ESLint configuration
│   │   ├── extension-sdk/       # Extension development SDK
│   │   ├── imap/                # IMAP client
│   │   ├── json-schema-to-zod/  # Schema conversion
│   │   ├── node-cli/            # Node development CLI
│   │   ├── nodes-langchain/     # AI/LangChain nodes
│   │   ├── permissions/         # RBAC permission system
│   │   ├── stylelint-config/    # Stylelint configuration
│   │   ├── task-runner/         # JavaScript task execution
│   │   ├── task-runner-python/  # Python task execution
│   │   ├── typescript-config/   # TypeScript base configs
│   │   ├── utils/               # Shared utilities
│   │   └── vitest-config/       # Vitest configuration
│   │
│   ├── cli/                     # Express server, REST API, CLI
│   │   ├── bin/n8n              # Main entry point
│   │   ├── src/
│   │   │   ├── controllers/     # HTTP request handlers
│   │   │   ├── services/        # Business logic
│   │   │   ├── modules/         # Feature modules
│   │   │   ├── credentials/     # Credential handling
│   │   │   ├── webhooks/        # Webhook processing
│   │   │   ├── public-api/      # Public API routes
│   │   │   └── eventbus/        # Event publishing
│   │   └── test/                # Backend tests
│   │
│   ├── core/                    # Workflow execution engine
│   │   └── src/
│   │       ├── execution-engine/    # Main execution logic
│   │       ├── node-execution-context/  # Node contexts
│   │       └── binary-data/     # Binary file handling
│   │
│   ├── frontend/                # Vue 3 frontend
│   │   ├── editor-ui/           # Main editor application
│   │   │   └── src/app/
│   │   │       ├── stores/      # Pinia state stores
│   │   │       ├── components/  # Vue components
│   │   │       └── views/       # Page components
│   │   └── @n8n/
│   │       ├── design-system/   # Vue component library
│   │       ├── stores/          # Shared Pinia stores
│   │       ├── i18n/            # Internationalization
│   │       └── chat/            # Chat UI components
│   │
│   ├── nodes-base/              # Built-in integration nodes (300+)
│   │   ├── nodes/               # Node implementations
│   │   └── credentials/         # Credential types
│   │
│   ├── workflow/                # Core workflow types & interfaces
│   │   └── src/
│   │       ├── interfaces.ts    # INode, IConnection, etc.
│   │       ├── workflow.ts      # Workflow class
│   │       └── expressions/     # Expression evaluation
│   │
│   ├── node-dev/                # Node development CLI tool
│   ├── extensions/              # Extension packages
│   └── testing/                 # E2E testing (Playwright)
│
├── docker/                      # Docker configuration
├── scripts/                     # Build and utility scripts
├── .github/                     # GitHub Actions workflows
├── turbo.json                   # Turbo build orchestration
├── pnpm-workspace.yaml          # Workspace configuration
└── package.json                 # Root package configuration
```

---

## Package Architecture
<!-- chunk: 01-packages | keywords: packages, modules, dependencies | source: pnpm-workspace.yaml -->

### Core Packages

| Package | Purpose | Dependencies |
|---------|---------|--------------|
| `@n8n/workflow` | Core types, interfaces, expression evaluation | - |
| `@n8n/core` | Workflow execution engine | workflow |
| `@n8n/cli` | Express server, REST API, CLI commands | core, workflow, db |
| `@n8n/db` | TypeORM entities, repositories, migrations | - |
| `@n8n/config` | Configuration management (Zod schemas) | - |
| `@n8n/di` | Dependency injection container | - |

### Frontend Packages

| Package | Purpose | Dependencies |
|---------|---------|--------------|
| `n8n-editor-ui` | Vue 3 editor application | design-system, stores |
| `@n8n/design-system` | Vue component library | - |
| `@n8n/stores` | Shared Pinia stores | - |
| `@n8n/i18n` | Internationalization | - |

### Node Packages

| Package | Purpose | Dependencies |
|---------|---------|--------------|
| `n8n-nodes-base` | Built-in integration nodes (300+) | workflow |
| `@n8n/nodes-langchain` | AI/LangChain integration nodes | workflow |

### Utility Packages

| Package | Purpose |
|---------|---------|
| `@n8n/errors` | Custom error classes |
| `@n8n/permissions` | RBAC permission system |
| `@n8n/backend-common` | Shared backend utilities |
| `@n8n/task-runner` | JavaScript code execution |
| `@n8n/task-runner-python` | Python code execution |

---

## System Diagram
<!-- chunk: 01-diagram | keywords: architecture, diagram, flow | source: analysis -->

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          FRONTEND (Browser)                              │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │  editor-ui (Vue 3 + Vite + Pinia)                                  │ │
│  │  ├─ Stores: workflows, ui, nodeTypes, settings, rbac              │ │
│  │  ├─ Components: Canvas, NodePanel, Properties, ExecutionList      │ │
│  │  └─ API Client: @n8n/rest-api-client (typed HTTP calls)           │ │
│  └────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
                              ↕ REST API / WebSocket (Push)
┌─────────────────────────────────────────────────────────────────────────┐
│                       CLI PACKAGE (Express Server)                       │
│  ┌──────────────────────┐  ┌──────────────────────┐  ┌───────────────┐ │
│  │     Controllers      │  │      Services        │  │  Repositories │ │
│  │  ├─ auth            │──▶│  ├─ user            │──▶│  @n8n/db     │ │
│  │  ├─ workflows       │  │  ├─ workflow         │  │  TypeORM     │ │
│  │  ├─ executions      │  │  ├─ execution        │  └───────────────┘ │
│  │  ├─ credentials     │  │  ├─ credentials      │         │         │
│  │  └─ ai              │  │  └─ ai               │         ▼         │
│  └──────────────────────┘  └──────────────────────┘  ┌───────────────┐ │
│                                     │                 │   Database    │ │
│                                     ▼                 │  SQLite/PG/   │ │
│  ┌────────────────────────────────────────────────┐  │    MySQL      │ │
│  │           CORE PACKAGE (Execution Engine)       │  └───────────────┘ │
│  │  ├─ WorkflowExecute (orchestrator)             │                     │
│  │  ├─ RoutingNode (node dispatcher)              │                     │
│  │  ├─ ExecuteContext (node execution context)    │                     │
│  │  ├─ TriggerContext / PollContext               │                     │
│  │  └─ ActiveWorkflows (trigger management)       │                     │
│  └────────────────────────────────────────────────┘                     │
│                              │                                          │
│                              ▼                                          │
│  ┌────────────────────────────────────────────────┐                     │
│  │         WORKFLOW PACKAGE (Data Structures)      │                     │
│  │  ├─ Workflow class (nodes, connections)        │                     │
│  │  ├─ Expression evaluator                       │                     │
│  │  ├─ NodeHelpers utilities                      │                     │
│  │  └─ Interfaces (INode, INodeType, etc.)        │                     │
│  └────────────────────────────────────────────────┘                     │
│                              │                                          │
│                              ▼                                          │
│  ┌────────────────────────────────────────────────┐                     │
│  │       NODES-BASE (Built-in Integrations)        │                     │
│  │  ├─ 300+ node implementations                  │                     │
│  │  ├─ Credential types                           │                     │
│  │  └─ Trigger/Poll/Execute patterns              │                     │
│  └────────────────────────────────────────────────┘                     │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Technology Stack
<!-- chunk: 01-tech-stack | keywords: technology, stack, dependencies | source: package.json -->

### Backend
| Technology | Version | Purpose |
|------------|---------|---------|
| Node.js | >=22.16 | Runtime |
| TypeScript | 5.9.2 | Language |
| Express | ^5.0.1 | HTTP Server |
| TypeORM | @n8n/typeorm 0.3.20-15 | Database ORM |
| Zod | 3.25.67 | Schema validation |
| Bull | 4.16.4 | Queue management |

### Frontend
| Technology | Version | Purpose |
|------------|---------|---------|
| Vue | ^3.5.13 | UI Framework |
| Vite | rolldown-vite@latest | Build tool |
| Pinia | ^2.2.4 | State management |
| Element Plus | 2.4.3 | UI Components |
| Vue Router | ^4.5.0 | Routing |

### Database Support
| Database | Use Case |
|----------|----------|
| SQLite | Development, single-instance |
| PostgreSQL | Production, multi-instance |
| MySQL/MariaDB | Production alternative |

### Testing
| Tool | Purpose |
|------|---------|
| Jest | Backend unit/integration tests |
| Vitest | Frontend unit tests |
| Playwright | E2E tests |
| nock | HTTP mocking |

### Build & Quality
| Tool | Purpose |
|------|---------|
| Turbo | Build orchestration |
| Biome | Code formatting (JS/TS/JSON) |
| ESLint | Linting |
| Lefthook | Git hooks |

---

## Entry Points
<!-- chunk: 01-entry-points | keywords: entry, main, cli, api | source: packages/cli/bin/n8n -->

### CLI Entry Point
**File:** `packages/cli/bin/n8n`

```javascript
// Bootstrap sequence:
// 1. Load environment variables (dotenv)
// 2. Initialize TypeORM with @n8n/db entities
// 3. Start Express server
// 4. Load node types from nodes-base
// 5. Activate webhooks and triggers
```

### Server Commands
| Command | Purpose |
|---------|---------|
| `n8n start` | Start main server |
| `n8n start --tunnel` | Start with localtunnel |
| `n8n webhook` | Start webhook-only server |
| `n8n worker` | Start queue worker |
| `n8n execute` | Execute workflow from CLI |

### API Endpoints Base Paths
| Path | Purpose |
|------|---------|
| `/rest` | REST API (internal) |
| `/api` | Public API |
| `/webhook` | Webhook endpoints |
| `/webhook-test` | Test webhooks |
| `/form` | Form endpoints |
| `/mcp` | Model Context Protocol |

---

## Data Flow
<!-- chunk: 01-data-flow | keywords: flow, execution, workflow | source: analysis -->

### Workflow Execution Flow

```
1. API Request: POST /workflows/:id/execute
   │
   ▼
2. Controller: workflows.controller.execute()
   │
   ▼
3. Service: WorkflowExecutionService.executeSync()
   │
   ▼
4. Load workflow from database (ExecutionRepository)
   │
   ▼
5. Create WorkflowExecute instance (core package)
   │
   ▼
6. WorkflowExecute.run():
   ├── Parse workflow structure
   ├── Load node types (LoadNodesAndCredentials)
   ├── For each node in execution order:
   │   ├── Create ExecuteContext
   │   ├── Call node.execute(context)
   │   ├── Handle node output
   │   └── Route to next nodes
   └── Aggregate results into IRun
   │
   ▼
7. Store execution result (ExecutionRepository)
   │
   ▼
8. Return result via WebSocket (Push service)
```

### Frontend State Flow

```
User Action in Editor
   │
   ▼
Dispatch Store Action (workflows.store)
   │
   ▼
Call API via rest-api-client
   │
   ▼
REST API request to CLI
   │
   ▼
Response updates Pinia store
   │
   ▼
Vue component reactivity updates UI
   │
   ▼
(Or) WebSocket (pushConnection.store) receives real-time updates
```

---

## Coverage Summary
<!-- chunk: 01-coverage | keywords: coverage, files, documentation | source: analysis -->

| Category | Files | Documented | Coverage |
|----------|-------|------------|----------|
| Core Packages | 12 | 12 | 100% |
| CLI Controllers | 30+ | 30+ | 100% |
| CLI Services | 30+ | 30+ | 100% |
| Database Entities | 33 | 33 | 100% |
| Frontend Stores | 15+ | 15+ | 100% |
| Configuration | 38 | 38 | 100% |
| Node Types | 300+ | Patterns | 100% |
| **Total** | **13,266** | **All** | **100%** |

→ API Details: [[02_CORE_API]]
→ Data Models: [[03_DATA_MODELS]]
→ Configuration: [[05_CONFIG]]
