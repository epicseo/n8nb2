# 08_QUICK_REF.md
<!-- repo: n8n | branch: claude/github-to-kb-converter-01JAurUearApTk4RAu9NPFAg | commit: 3c0e809e | generated: 2025-11-28 -->
<!-- tags: reference, cheatsheet, index, quick, lookup -->

## Contents
- [Command Cheatsheet](#command-cheatsheet)
- [Function Index](#function-index)
- [Class Index](#class-index)
- [Entity Index](#entity-index)
- [Environment Variable Index](#environment-variable-index)
- [Error Code Index](#error-code-index)
- [API Endpoint Index](#api-endpoint-index)
- [File Purpose Index](#file-purpose-index)
- [Glossary](#glossary)

---

## Command Cheatsheet
<!-- chunk: 08-commands | keywords: commands, cli, pnpm | source: package.json -->

### Development Commands

| Command | Purpose |
|---------|---------|
| `pnpm install` | Install dependencies |
| `pnpm build > build.log 2>&1` | Build all packages |
| `pnpm dev` | Start development server |
| `pnpm dev:be` | Start backend only |
| `pnpm dev:fe` | Start frontend only |
| `pnpm dev:ai` | Start AI development |

### Quality Commands

| Command | Purpose |
|---------|---------|
| `pnpm typecheck` | Run TypeScript checks |
| `pnpm lint` | Run linter |
| `pnpm lint:fix` | Fix lint issues |
| `pnpm format` | Format code |

### Testing Commands

| Command | Purpose |
|---------|---------|
| `pnpm test` | Run all tests |
| `pnpm test:affected` | Test changed files |
| `pnpm test:ci` | CI test runner |
| `cd packages/cli && pnpm test` | Package-specific tests |

### n8n CLI Commands

| Command | Purpose |
|---------|---------|
| `n8n start` | Start server |
| `n8n start --tunnel` | Start with tunnel |
| `n8n webhook` | Webhook-only server |
| `n8n worker` | Queue worker |
| `n8n execute --id=<id>` | Execute workflow |
| `n8n export:workflow --id=<id>` | Export workflow |

---

## Function Index
<!-- chunk: 08-functions | keywords: function, method, api | source: analysis -->

### Core Functions

| Function | Module | Link |
|----------|--------|------|
| `WorkflowExecute.run` | core | [[02_CORE_API#workflowexecute]] |
| `WorkflowExecute.runPartialWorkflow2` | core | [[02_CORE_API#workflowexecute]] |
| `Workflow.getNode` | workflow | [[02_CORE_API#workflow-class]] |
| `Workflow.getTriggerNodes` | workflow | [[02_CORE_API#workflow-class]] |
| `Workflow.getStaticData` | workflow | [[02_CORE_API#workflow-class]] |
| `Expression.resolveSimpleParameterValue` | workflow | [[02_CORE_API#expression-class]] |

### Node Helpers

| Function | Module | Link |
|----------|--------|------|
| `displayParameter` | node-helpers | [[02_CORE_API#nodehelpers]] |
| `getNodeParameters` | node-helpers | [[02_CORE_API#nodehelpers]] |
| `getNodeParametersIssues` | node-helpers | [[02_CORE_API#nodehelpers]] |
| `isTriggerNode` | node-helpers | [[02_CORE_API#nodehelpers]] |
| `getNodeWebhookUrl` | node-helpers | [[02_CORE_API#nodehelpers]] |

### Service Methods

| Function | Service | Link |
|----------|---------|------|
| `UserService.update` | cli | [[02_CORE_API#userservice]] |
| `UserService.toPublic` | cli | [[02_CORE_API#userservice]] |
| `CredentialsService.getMany` | cli | [[02_CORE_API#credentialsservice]] |
| `CredentialsService.test` | cli | [[02_CORE_API#credentialsservice]] |
| `CredentialsService.decrypt` | cli | [[02_CORE_API#credentialsservice]] |

---

## Class Index
<!-- chunk: 08-classes | keywords: class, type, interface | source: analysis -->

### Core Classes

| Class | Package | Link |
|-------|---------|------|
| `WorkflowExecute` | core | [[02_CORE_API#workflowexecute]] |
| `ExecuteContext` | core | [[02_CORE_API#executecontext]] |
| `TriggerContext` | core | [[02_CORE_API#triggercontext]] |
| `PollContext` | core | [[02_CORE_API#pollcontext]] |
| `Workflow` | workflow | [[02_CORE_API#workflow-class]] |
| `Expression` | workflow | [[02_CORE_API#expression-class]] |

### Error Classes

| Class | Status | Link |
|-------|--------|------|
| `BadRequestError` | 400 | [[06_DEBUG#http-response-errors]] |
| `AuthError` | 401 | [[06_DEBUG#http-response-errors]] |
| `ForbiddenError` | 403 | [[06_DEBUG#http-response-errors]] |
| `NotFoundError` | 404 | [[06_DEBUG#http-response-errors]] |
| `InternalServerError` | 500 | [[06_DEBUG#http-response-errors]] |
| `UserError` | - | [[06_DEBUG#execution-errors]] |
| `NodeOperationError` | - | [[06_DEBUG#execution-errors]] |

---

## Entity Index
<!-- chunk: 08-entities | keywords: entity, database, table | source: analysis -->

| Entity | Table | Link |
|--------|-------|------|
| `WorkflowEntity` | workflow_entity | [[03_DATA_MODELS#workflowentity]] |
| `ExecutionEntity` | execution_entity | [[03_DATA_MODELS#executionentity]] |
| `CredentialsEntity` | credentials_entity | [[03_DATA_MODELS#credentialsentity]] |
| `User` | user | [[03_DATA_MODELS#user]] |
| `Project` | project | [[03_DATA_MODELS#project]] |
| `TagEntity` | tag_entity | [[03_DATA_MODELS#tagentity]] |
| `Folder` | folder | [[03_DATA_MODELS#folder]] |
| `SharedWorkflow` | shared_workflow | [[03_DATA_MODELS#sharedworkflow]] |
| `SharedCredentials` | shared_credentials | [[03_DATA_MODELS#sharedcredentials]] |
| `ExecutionData` | execution_data | [[03_DATA_MODELS#executiondata]] |
| `WebhookEntity` | webhook_entity | [[03_DATA_MODELS#webhookentity]] |
| `ApiKey` | api_key | [[03_DATA_MODELS#apikey]] |
| `Role` | role | [[03_DATA_MODELS#role--scope]] |
| `WorkflowHistory` | workflow_history | [[03_DATA_MODELS#workflowhistory]] |

---

## Environment Variable Index
<!-- chunk: 08-env | keywords: environment, variable, config | source: analysis -->

### Server

| Variable | Default | Link |
|----------|---------|------|
| `N8N_HOST` | localhost | [[05_CONFIG#server-settings]] |
| `N8N_PORT` | 5678 | [[05_CONFIG#server-settings]] |
| `N8N_PROTOCOL` | http | [[05_CONFIG#server-settings]] |
| `N8N_ENCRYPTION_KEY` | - | [[05_CONFIG#general-settings]] |

### Database

| Variable | Default | Link |
|----------|---------|------|
| `DB_TYPE` | sqlite | [[05_CONFIG#database-type]] |
| `DB_POSTGRESDB_HOST` | localhost | [[05_CONFIG#postgresql-configuration]] |
| `DB_POSTGRESDB_DATABASE` | n8n | [[05_CONFIG#postgresql-configuration]] |

### Execution

| Variable | Default | Link |
|----------|---------|------|
| `EXECUTIONS_MODE` | regular | [[05_CONFIG#execution-mode]] |
| `EXECUTIONS_TIMEOUT` | -1 | [[05_CONFIG#timeouts]] |
| `EXECUTIONS_DATA_PRUNE` | true | [[05_CONFIG#data-retention]] |

### Logging

| Variable | Default | Link |
|----------|---------|------|
| `N8N_LOG_LEVEL` | info | [[05_CONFIG#log-settings]] |
| `N8N_LOG_OUTPUT` | console | [[05_CONFIG#log-settings]] |
| `N8N_LOG_SCOPES` | - | [[05_CONFIG#log-scopes]] |

### Queue/Redis

| Variable | Default | Link |
|----------|---------|------|
| `QUEUE_BULL_REDIS_HOST` | localhost | [[05_CONFIG#redis-configuration]] |
| `QUEUE_BULL_REDIS_PORT` | 6379 | [[05_CONFIG#redis-configuration]] |

---

## Error Code Index
<!-- chunk: 08-errors | keywords: error, code, status | source: analysis -->

| Code | Error | Link |
|------|-------|------|
| 400 | BadRequestError | [[06_DEBUG#http-response-errors]] |
| 401 | AuthError, UnauthenticatedError | [[06_DEBUG#http-response-errors]] |
| 403 | ForbiddenError, InvalidMfaCodeError | [[06_DEBUG#http-response-errors]] |
| 404 | NotFoundError, WebhookNotFoundError | [[06_DEBUG#http-response-errors]] |
| 409 | ConflictError | [[06_DEBUG#http-response-errors]] |
| 413 | ContentTooLargeError | [[06_DEBUG#http-response-errors]] |
| 429 | TooManyRequestsError | [[06_DEBUG#http-response-errors]] |
| 500 | InternalServerError | [[06_DEBUG#http-response-errors]] |
| 503 | ServiceUnavailableError | [[06_DEBUG#http-response-errors]] |

---

## API Endpoint Index
<!-- chunk: 08-endpoints | keywords: api, endpoint, rest | source: analysis -->

### Authentication

| Method | Path | Link |
|--------|------|------|
| POST | `/login` | [[02_CORE_API#authentication-controller]] |
| GET | `/login` | [[02_CORE_API#authentication-controller]] |
| POST | `/logout` | [[02_CORE_API#authentication-controller]] |

### Users

| Method | Path | Link |
|--------|------|------|
| GET | `/users` | [[02_CORE_API#users-controller]] |
| DELETE | `/users/:id` | [[02_CORE_API#users-controller]] |
| PATCH | `/users/:id/role` | [[02_CORE_API#users-controller]] |

### Credentials

| Method | Path | Link |
|--------|------|------|
| GET | `/credentials` | [[02_CORE_API#credentials-controller]] |
| POST | `/credentials` | [[02_CORE_API#credentials-controller]] |
| POST | `/credentials/test` | [[02_CORE_API#credentials-controller]] |
| DELETE | `/credentials/:id` | [[02_CORE_API#credentials-controller]] |

### Workflows

| Method | Path | Link |
|--------|------|------|
| GET | `/workflows` | [[02_CORE_API#workflows-controller]] |
| POST | `/workflows` | [[02_CORE_API#workflows-controller]] |
| PATCH | `/workflows/:id` | [[02_CORE_API#workflows-controller]] |
| POST | `/workflows/:id/run` | [[02_CORE_API#workflows-controller]] |

### AI

| Method | Path | Link |
|--------|------|------|
| POST | `/ai/build` | [[02_CORE_API#ai-controller]] |
| POST | `/ai/chat` | [[02_CORE_API#ai-controller]] |
| POST | `/ai/ask` | [[02_CORE_API#ai-controller]] |

### Health

| Method | Path | Purpose |
|--------|------|---------|
| GET | `/healthz` | Liveness check |
| GET | `/healthz/readiness` | Readiness check |

---

## File Purpose Index
<!-- chunk: 08-files | keywords: file, path, purpose | source: analysis -->

### Entry Points

| File | Purpose |
|------|---------|
| `packages/cli/bin/n8n` | CLI entry point |
| `packages/frontend/editor-ui/src/main.ts` | Frontend entry |

### Core Files

| File | Purpose |
|------|---------|
| `packages/core/src/execution-engine/workflow-execute.ts` | Execution orchestrator |
| `packages/workflow/src/workflow.ts` | Workflow class |
| `packages/workflow/src/interfaces.ts` | Core interfaces |

### Configuration

| File | Purpose |
|------|---------|
| `packages/@n8n/config/src/index.ts` | Configuration root |
| `turbo.json` | Build orchestration |
| `pnpm-workspace.yaml` | Workspace config |

### Frontend

| File | Purpose |
|------|---------|
| `packages/frontend/editor-ui/src/app/stores/workflows.store.ts` | Workflow state |
| `packages/frontend/editor-ui/src/app/stores/ui.store.ts` | UI state |

---

## Glossary
<!-- chunk: 08-glossary | keywords: glossary, terms, definitions | source: analysis -->

| Term | Definition |
|------|------------|
| **Workflow** | Automated process defined by nodes and connections |
| **Node** | Single step in a workflow (trigger, action, etc.) |
| **Trigger Node** | Node that starts workflow execution |
| **Poll Node** | Node that periodically checks for new data |
| **Execution** | Single run of a workflow |
| **Credential** | Authentication data for external services |
| **Project** | Container for workflows and credentials |
| **Pinned Data** | Fixed test data for nodes |
| **Expression** | Dynamic value using `{{ }}` syntax |
| **Webhook** | HTTP endpoint for triggering workflows |
| **Task Runner** | Isolated environment for code execution |
| **Static Data** | Data persisted across workflow executions |
| **RBAC** | Role-Based Access Control |
| **Pinia** | Vue.js state management library |
| **TypeORM** | Database ORM used in n8n |
| **Turbo** | Build system for monorepos |

---

## Quick Links

| Topic | Link |
|-------|------|
| Architecture Overview | [[01_ARCHITECTURE]] |
| API Reference | [[02_CORE_API]] |
| Data Models | [[03_DATA_MODELS]] |
| Coding Patterns | [[04_PATTERNS]] |
| Configuration | [[05_CONFIG]] |
| Debugging | [[06_DEBUG]] |
| Examples | [[07_EXAMPLES]] |
