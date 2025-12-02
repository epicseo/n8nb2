# 03_PATTERNS_EXAMPLES.md
<!-- repo: n8n | version: 1.122.0 | generated: 2025-12-02 -->
<!-- tags: patterns, errors, stores, examples, best-practices -->

## Contents
- [Development Patterns](#development-patterns)
- [Error Reference](#error-reference)
- [Frontend Stores](#frontend-stores)
- [Code Examples](#code-examples)

---

## Development Patterns
<!-- chunk: 03-patterns | keywords: patterns, conventions, architecture -->

### File Naming Conventions

| Suffix | Purpose | Package |
|--------|---------|---------|
| `.controller.ts` | HTTP handlers | cli |
| `.service.ts` | Business logic | cli |
| `.repository.ts` | Data access | cli |
| `.store.ts` | Pinia stores | frontend |
| `.node.ts` | Node implementation | nodes-base |
| `.credentials.ts` | Credential type | nodes-base |

### Dependency Injection

```typescript
import { Service } from '@n8n/di';

@Service()
export class UserService {
  constructor(
    private readonly logger: Logger,
    private readonly userRepository: UserRepository,
  ) {}
}
```

### Controller Pattern

```typescript
import { RestController, Get, Scopes } from '@n8n/decorators';

@RestController('/users')
export class UsersController {
  constructor(private readonly userService: UserService) {}

  @Get('/')
  @Scopes('user:list')
  async listUsers(req: AuthenticatedRequest): Promise<PublicUser[]> {
    return this.userService.findAll();
  }
}
```

### Node Development Pattern

```typescript
export class MyNode implements INodeType {
  description: INodeTypeDescription = {
    displayName: 'My Node',
    name: 'myNode',
    group: ['output'],
    version: 1,
    inputs: [NodeConnectionTypes.Main],
    outputs: [NodeConnectionTypes.Main],
    properties: [/* ... */],
  };

  async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
    const items = this.getInputData();
    // Process items
    return [items];
  }
}
```

### Frontend Store Pattern

```typescript
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';

export const useMyStore = defineStore('myStore', () => {
  // State
  const items = ref<Item[]>([]);

  // Getters
  const itemCount = computed(() => items.value.length);

  // Actions
  function addItem(item: Item) {
    items.value.push(item);
  }

  return { items, itemCount, addItem };
});
```

---

## Error Reference
<!-- chunk: 03-errors | keywords: errors, http, status, exceptions -->

### HTTP Response Errors

| Error | Status | When Thrown |
|-------|--------|-------------|
| `BadRequestError` | 400 | Invalid request parameters |
| `UnauthenticatedError` | 401 | Missing authentication |
| `AuthError` | 401 | Invalid credentials |
| `ForbiddenError` | 403 | Access denied |
| `InvalidMfaCodeError` | 403 | MFA code validation failure |
| `NotFoundError` | 404 | Resource not found |
| `WebhookNotFoundError` | 404 | Webhook path not registered |
| `ConflictError` | 409 | Duplicate resources |
| `ContentTooLargeError` | 413 | Payload exceeds limit |
| `TooManyRequestsError` | 429 | Rate limit exceeded |
| `InternalServerError` | 500 | Server error |
| `ServiceUnavailableError` | 503 | Database unavailable |

### Base Error Classes

| Class | Level | Use Case |
|-------|-------|----------|
| `UserError` | info | User-triggered errors (invalid input) |
| `OperationalError` | warning | Transient issues (network, timeout) |
| `UnexpectedError` | error | Code logic mistakes |

### Execution Errors

| Error | Cause |
|-------|-------|
| `NodeCrashedError` | Out of memory in node |
| `WorkflowCrashedError` | Total memory exhausted |
| `NodeApiError` | External API errors |
| `NodeOperationError` | Node execution failure |
| `ExecutionCancelledError` | Manual or timeout cancellation |
| `TaskRunnerOomError` | Task runner out of memory |

### Error Usage

```typescript
import { NotFoundError } from '@/errors/response-errors/not-found.error';
import { UserError } from 'n8n-workflow';

// Type guard pattern
NotFoundError.isDefinedAndNotNull(resource, 'Resource not found');

// User error
if (!valid) {
  throw new UserError('Invalid input provided');
}
```

---

## Frontend Stores
<!-- chunk: 03-stores | keywords: pinia, stores, state, vue -->

### Core Stores

| Store | Purpose | Key State |
|-------|---------|-----------|
| `useWorkflowsStore` | Workflow management | workflow, nodes, connections, executions |
| `useSettingsStore` | App configuration | settings, features, modules |
| `useUIStore` | UI state | modals, theme, sidebar |
| `useNodeTypesStore` | Node registry | nodeTypes, community nodes |
| `useRBACStore` | Permissions | roles, scopes |

### useWorkflowsStore Key APIs

```typescript
// State
workflow: IWorkflowDb
allNodes: INodeUi[]
workflowExecutionData: IExecutionResponse

// Actions
createNewWorkflow(workflow): Promise<IWorkflowDb>
addNode(node: INodeUi): void
addConnection(connection: IConnection): void
runWorkflow(options): Promise<IExecutionPushResponse>
pinData(nodeName, data): void
```

### useUIStore Key APIs

```typescript
// State
theme: 'light' | 'dark' | 'system'
modalsById: Record<string, ModalState>
sidebarMenuCollapsed: boolean

// Actions
setTheme(theme): void
openModal(name): void
closeModal(name): void
toggleSidebarMenuCollapse(): void
```

### useSettingsStore Key APIs

```typescript
// Getters
isEnterpriseFeatureEnabled(feature): boolean
isAiAssistantEnabled: boolean
isCloudDeployment: boolean
isMfaFeatureEnabled: boolean

// Actions
async getSettings(): Promise<void>
async initialize(): Promise<void>
```

### usePushConnectionStore

```typescript
// Manages WebSocket/SSE for real-time updates
pushConnect(): void
pushDisconnect(): void
addEventListener(handler): () => void
```

---

## Code Examples
<!-- chunk: 03-examples | keywords: examples, code, implementation -->

### Trigger Node Example

```typescript
export class MyTrigger implements INodeType {
  description = {
    displayName: 'My Trigger',
    name: 'myTrigger',
    group: ['trigger'],
    inputs: [],
    outputs: [NodeConnectionTypes.Main],
  };

  async trigger(this: ITriggerFunctions): Promise<ITriggerResponse> {
    const executeTrigger = () => {
      this.emit([this.helpers.returnJsonArray([{ triggered: true }])]);
    };

    // Register cron or event listener
    this.helpers.registerCron({ expression: '0 * * * *' }, executeTrigger);

    return { manualTriggerFunction: async () => executeTrigger() };
  }
}
```

### Credential Type Example

```typescript
export class MyApi implements ICredentialType {
  name = 'myApi';
  displayName = 'My API';
  properties: INodeProperties[] = [
    {
      displayName: 'API Key',
      name: 'apiKey',
      type: 'string',
      typeOptions: { password: true },
      default: '',
    },
  ];

  authenticate: IAuthenticateGeneric = {
    type: 'generic',
    properties: {
      headers: { 'Authorization': '={{ "Bearer " + $credentials.apiKey }}' },
    },
  };

  test: ICredentialTestRequest = {
    request: { url: 'https://api.example.com/me' },
  };
}
```

### Service with Error Handling

```typescript
@Service()
export class ResourceService {
  async get(id: string): Promise<Resource> {
    const resource = await this.repository.findOne({ where: { id } });
    NotFoundError.isDefinedAndNotNull(resource, `Resource ${id} not found`);
    return resource;
  }

  async validate(data: ResourceData): Promise<void> {
    if (!data.name) {
      throw new UserError('Name is required');
    }
  }
}
```

### Store Test Example

```typescript
import { createPinia, setActivePinia } from 'pinia';
import { useMyStore } from '../my.store';

describe('MyStore', () => {
  beforeEach(() => {
    setActivePinia(createPinia());
  });

  it('should add item', () => {
    const store = useMyStore();
    store.addItem({ id: '1', name: 'Test' });
    expect(store.itemCount).toBe(1);
  });
});
```

---

## Statistics

| Category | Count |
|----------|-------|
| Error Classes | 135+ |
| Frontend Stores | 19 |
| Store Actions | 400+ |
| Store Getters | 200+ |

---

*Source: packages/cli/src/, packages/workflow/src/errors/, packages/frontend/editor-ui/src/app/stores/*
