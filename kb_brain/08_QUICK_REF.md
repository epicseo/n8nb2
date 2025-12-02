# 08_QUICK_REF.md
<!-- repo: n8n | version: 1.122.0 | generated: 2025-12-02 -->
<!-- tags: index, quick-reference, commands, glossary, prompts -->

## Contents
- [KB Index](#kb-index)
- [Quick Command Reference](#quick-command-reference)
- [Common Tasks](#common-tasks)
- [Node Type Quick Reference](#node-type-quick-reference)
- [Expression Cheatsheet](#expression-cheatsheet)
- [Environment Variables](#environment-variables)
- [System Prompts](#system-prompts)

---

## KB Index
<!-- chunk: 08-index | keywords: index, files, contents -->

### File Map

| File | Contents |
|------|----------|
| `01_PLATFORM_OVERVIEW.md` | Architecture, packages, config, debugging |
| `02_CORE_API.md` | REST API, controllers, services, entities, types |
| `03_PATTERNS_EXAMPLES.md` | Dev patterns, error classes, stores, examples |
| `04_NODE_REFERENCE.md` | All 649 nodes with JSON examples |
| `05_CREDENTIALS.md` | All 389 credential types |
| `06_WORKFLOW_BUILDER.md` | JSON schema, expressions, templates |
| `07_ADVANCED_PATTERNS.md` | Binary data, HTTP, errors, pagination |
| `08_QUICK_REF.md` | This file - index and quick reference |

### Cross-Reference Guide

| Looking For | See |
|-------------|-----|
| Architecture | `01_PLATFORM_OVERVIEW.md` |
| API Endpoints | `02_CORE_API.md` |
| Database Entities | `02_CORE_API.md` |
| Development Patterns | `03_PATTERNS_EXAMPLES.md` |
| Error Classes | `03_PATTERNS_EXAMPLES.md` |
| Frontend Stores | `03_PATTERNS_EXAMPLES.md` |
| Node Types | `04_NODE_REFERENCE.md` |
| Credentials | `05_CREDENTIALS.md` |
| Workflow JSON | `06_WORKFLOW_BUILDER.md` |
| Expressions | `06_WORKFLOW_BUILDER.md` |
| Templates | `06_WORKFLOW_BUILDER.md` |
| File/Binary | `07_ADVANCED_PATTERNS.md` |
| HTTP Request | `07_ADVANCED_PATTERNS.md` |
| Error Handling | `07_ADVANCED_PATTERNS.md` |
| Pagination | `07_ADVANCED_PATTERNS.md` |

---

## Quick Command Reference
<!-- chunk: 08-commands | keywords: commands, pnpm, cli -->

### Development
```bash
pnpm install                    # Install dependencies
pnpm build > build.log 2>&1     # Build all packages
pnpm dev                        # Start dev server (http://localhost:5678)
pnpm dev:ai                     # Start with AI features
pnpm typecheck                  # TypeScript checks
pnpm lint                       # Run linter
pnpm test                       # Run all tests
```

### Package-Specific
```bash
cd packages/cli && pnpm test    # Test CLI package
cd packages/editor-ui && pnpm lint   # Lint frontend
cd packages/workflow && pnpm typecheck  # Check types
```

### Git
```bash
git checkout -b feat/my-feature  # New branch
git add -p                       # Interactive staging
git commit -m "feat: description"  # Commit
git push -u origin HEAD          # Push branch
```

### n8n CLI
```bash
n8n start                        # Start n8n
n8n export:workflow --id=<id>    # Export workflow
n8n import:workflow --input=<file>  # Import workflow
n8n execute --id=<id>            # Execute workflow
```

---

## Common Tasks
<!-- chunk: 08-tasks | keywords: tasks, howto -->

### Create New Node
1. Create `packages/nodes-base/nodes/MyNode/MyNode.node.ts`
2. Implement `INodeType` interface
3. Add to `package.json`
4. Run `pnpm build`

### Create New Controller
1. Create `packages/cli/src/controllers/my.controller.ts`
2. Use `@RestController` decorator
3. Register in module
4. Add types to `@n8n/api-types`

### Create New Store
1. Create `packages/frontend/editor-ui/src/app/stores/my.store.ts`
2. Use `defineStore` from Pinia
3. Export state, getters, actions

### Add Environment Variable
1. Add to `packages/@n8n/config/src/config.ts`
2. Use Zod for validation
3. Document in `.env.example`

---

## Node Type Quick Reference
<!-- chunk: 08-nodes-quick | keywords: nodes, types, quick -->

### Triggers
| Type | Description |
|------|-------------|
| `n8n-nodes-base.manualTrigger` | Manual execution |
| `n8n-nodes-base.webhook` | HTTP endpoint |
| `n8n-nodes-base.scheduleTrigger` | Cron/interval |
| `@n8n/n8n-nodes-langchain.chatTrigger` | AI chat |

### Flow Control
| Type | Description |
|------|-------------|
| `n8n-nodes-base.if` | Conditional (2 outputs) |
| `n8n-nodes-base.switch` | Multi-way routing |
| `n8n-nodes-base.merge` | Combine branches |
| `n8n-nodes-base.splitInBatches` | Batch processing |

### Data Transform
| Type | Description |
|------|-------------|
| `n8n-nodes-base.set` | Edit/add fields |
| `n8n-nodes-base.code` | Custom JS/Python |
| `n8n-nodes-base.filter` | Filter items |
| `n8n-nodes-base.aggregate` | Group data |

### Integration
| Type | Description |
|------|-------------|
| `n8n-nodes-base.httpRequest` | API calls |
| `n8n-nodes-base.slack` | Slack messaging |
| `n8n-nodes-base.googleSheets` | Spreadsheets |
| `n8n-nodes-base.postgres` | PostgreSQL |

### AI/LangChain
| Type | Description |
|------|-------------|
| `@n8n/n8n-nodes-langchain.agent` | AI agent |
| `@n8n/n8n-nodes-langchain.lmChatOpenAi` | OpenAI |
| `@n8n/n8n-nodes-langchain.memoryBufferWindow` | Memory |
| `@n8n/n8n-nodes-langchain.toolCalculator` | Tool |

---

## Expression Cheatsheet
<!-- chunk: 08-expressions | keywords: expressions, cheatsheet -->

### Variables
```javascript
$json                    // Current item
$json.field              // Field access
$input.first()           // First input
$input.all()             // All items
$('NodeName').item.json  // Other node
$now                     // Current time
$env.VAR                 // Environment
$execution.id            // Execution ID
```

### String Methods
```javascript
.toUpperCase()           // UPPERCASE
.toLowerCase()           // lowercase
.isEmail()               // Validate email
.extractEmail()          // Extract email
.base64Encode()          // Encode
.hash('sha256')          // Hash
```

### Array Methods
```javascript
.first()                 // First item
.last()                  // Last item
.sum()                   // Sum numbers
.unique()                // Dedupe
.pluck('field')          // Extract field
```

### Date Methods
```javascript
$now.toISO()                    // ISO string
$now.toFormat('yyyy-MM-dd')     // Format
$now.plus(7, 'days')            // Add time
$now.minus(1, 'month')          // Subtract
$now.startOf('day')             // Start of day
```

### Conditional
```javascript
condition ? ifTrue : ifFalse
$json.value ?? 'default'
$json.user?.email
```

---

## Environment Variables
<!-- chunk: 08-env | keywords: environment, config -->

### Core
```bash
N8N_HOST=localhost
N8N_PORT=5678
N8N_PROTOCOL=http
N8N_ENCRYPTION_KEY=<key>
```

### Database
```bash
DB_TYPE=sqlite|postgresdb|mysqldb
DB_POSTGRESDB_HOST=localhost
DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_USER=n8n
DB_POSTGRESDB_PASSWORD=<password>
```

### Execution
```bash
EXECUTIONS_MODE=regular|queue
EXECUTIONS_TIMEOUT=3600
EXECUTIONS_DATA_SAVE_ON_SUCCESS=all
```

### Logging
```bash
N8N_LOG_LEVEL=info|debug|warn|error
N8N_LOG_OUTPUT=console|file
```

---

## System Prompts
<!-- chunk: 08-prompts | keywords: prompts, ai, system -->

### n8n Expert System Prompt

```markdown
You are an expert n8n developer with deep knowledge of the n8n workflow
automation platform (version 1.122.0).

## Technology Stack
- Backend: Node.js 22+, TypeScript 5.9, Express 5.0
- Frontend: Vue 3.5, Vite, Pinia
- Database: TypeORM with SQLite/PostgreSQL/MySQL
- Build: pnpm workspaces + Turbo
- Testing: Jest, Vitest, Playwright

## Key Patterns
- Use `@Service()` for dependency injection
- Use `@RestController()` for HTTP endpoints
- Use `defineStore()` for frontend state
- Never use `any` type or `ApplicationError`

## Error Classes
- UserError: User-triggered errors
- OperationalError: Transient issues
- UnexpectedError: Programming errors
- NodeApiError: External API errors

## When Helping
1. Reference KB files for detailed info
2. Provide file:line references
3. Use correct n8n patterns
4. Check types in workflow package
5. Suggest pnpm commands
```

### Workflow Builder System Prompt

```markdown
You are an expert n8n workflow builder specializing in creating valid
n8n workflow JSON files that can be imported directly into n8n.

## Workflow JSON Structure
{
  "name": "Workflow Name",
  "nodes": [],
  "connections": {},
  "settings": { "executionOrder": "v1" },
  "active": false
}

## Node Structure
{
  "id": "unique-uuid",
  "name": "Node Display Name",
  "type": "n8n-nodes-base.nodetype",
  "typeVersion": 1.0,
  "position": [250, 300],
  "parameters": {}
}

## Connection Structure
{
  "SourceNode": {
    "main": [[{ "node": "TargetNode", "type": "main", "index": 0 }]]
  }
}

## AI Connections
- ai_languageModel: LLM provider (max 1)
- ai_memory: Conversation history (max 1)
- ai_tool: Agent tools (unlimited)
- ai_vectorStore: Vector database (max 1)

## Validation Checklist
- All nodes have unique id and name
- All nodes have correct type and typeVersion
- Positions don't overlap
- All connections reference existing nodes
- Expressions use ={{ }} syntax
```

---

## Statistics Summary

| Category | Count |
|----------|-------|
| Total Nodes | 649 |
| Credential Types | 389 |
| Error Classes | 135+ |
| Frontend Stores | 19 |
| Connection Types | 13 |
| API Controllers | 29 |
| Database Entities | 15 |

---

## Glossary

| Term | Definition |
|------|------------|
| **Node** | Building block of a workflow |
| **Trigger** | Node that starts workflow execution |
| **Workflow** | Collection of connected nodes |
| **Execution** | Single run of a workflow |
| **Credential** | Stored authentication info |
| **Expression** | Dynamic value using `={{ }}` syntax |
| **Binary Data** | File/image data attached to items |
| **Pinned Data** | Test data stored with node |
| **Webhook** | HTTP endpoint trigger |
| **Agent** | AI node with tools and memory |

---

*Source: n8n repository v1.122.0*
