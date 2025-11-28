# 04_PATTERNS.md
<!-- repo: n8n | branch: claude/github-to-kb-converter-01JAurUearApTk4RAu9NPFAg | commit: 3c0e809e | generated: 2025-11-28 -->
<!-- tags: patterns, conventions, code-style, architecture, best-practices -->

## Contents
- [File Naming Conventions](#file-naming-conventions)
- [Dependency Injection Pattern](#dependency-injection-pattern)
- [Controller-Service-Repository Pattern](#controller-service-repository-pattern)
- [Node Development Pattern](#node-development-pattern)
- [Error Handling Pattern](#error-handling-pattern)
- [Frontend Store Pattern](#frontend-store-pattern)
- [Testing Patterns](#testing-patterns)
- [Code Style](#code-style)

---

## File Naming Conventions
<!-- chunk: 04-naming | keywords: naming, files, conventions | source: analysis -->

### Backend (CLI Package)
| Suffix | Purpose | Example |
|--------|---------|---------|
| `.controller.ts` | HTTP request handlers | `users.controller.ts` |
| `.service.ts` | Business logic | `user.service.ts` |
| `.repository.ts` | Data access | `credentials.repository.ts` |
| `.error.ts` | Error classes | `auth.error.ts` |
| `.test.ts` | Test files | `users.controller.test.ts` |
| `.dto.ts` | Data transfer objects | `login-request.dto.ts` |

### Frontend (Editor UI)
| Suffix | Purpose | Example |
|--------|---------|---------|
| `.store.ts` | Pinia stores | `workflows.store.ts` |
| `.store.test.ts` | Store tests | `rbac.store.test.ts` |
| `.vue` | Vue components | `WorkflowCanvas.vue` |
| `.utils.ts` | Utility functions | `nodeView.utils.ts` |

### Nodes-Base
| Suffix | Purpose | Example |
|--------|---------|---------|
| `.node.ts` | Node implementation | `Discord.node.ts` |
| `.node.json` | Node metadata | `Discord.node.json` |
| `.credentials.ts` | Credential type | `DiscordApi.credentials.ts` |
| `router.ts` | Operation router | `actions/router.ts` |

---

## Dependency Injection Pattern
<!-- chunk: 04-di | keywords: di, dependency, injection, service | source: packages/@n8n/di/ -->

### Service Registration
```typescript
// packages/cli/src/services/user.service.ts
import { Service } from '@n8n/di';
import { Logger } from '@n8n/backend-common';

@Service()
export class UserService {
  constructor(
    private readonly logger: Logger,
    private readonly userRepository: UserRepository,
    private readonly mailer: UserManagementMailer,
    private readonly eventService: EventService,
  ) {}

  async update(userId: string, data: Partial<User>) {
    const user = await this.userRepository.findOneBy({ id: userId });
    if (user) {
      await this.userRepository.save({ ...user, ...data });
    }
  }
}
```

### Key Points
- Use `@Service()` decorator from `@n8n/di`
- All dependencies injected through constructor
- Use `private readonly` for dependencies
- Services are singletons by default

---

## Controller-Service-Repository Pattern
<!-- chunk: 04-csr | keywords: controller, service, repository, mvc | source: packages/cli/src/ -->

### Controller Layer
Handles HTTP requests, validation, response formatting.

```typescript
// packages/cli/src/controllers/users.controller.ts
import { RestController, Get, Delete, Patch } from '@n8n/decorators';

@RestController('/users')
export class UsersController {
  constructor(
    private readonly userService: UserService,
    private readonly eventService: EventService,
  ) {}

  @Get('/')
  @Scopes('user:list')
  async listUsers(req: AuthenticatedRequest): Promise<PublicUser[]> {
    const users = await this.userService.findAll();
    return users.map(u => this.userService.toPublic(u));
  }

  @Delete('/:id')
  @Scopes('user:delete')
  async deleteUser(req: AuthenticatedRequest, @Param('id') id: string) {
    await this.userService.delete(id);
    this.eventService.emit('user-deleted', { userId: id });
    return { success: true };
  }
}
```

### Service Layer
Contains business logic, orchestrates operations.

```typescript
// packages/cli/src/services/user.service.ts
@Service()
export class UserService {
  constructor(
    private readonly userRepository: UserRepository,
    private readonly credentialsRepository: CredentialsRepository,
  ) {}

  async delete(userId: string): Promise<void> {
    // Business logic: transfer resources, cleanup
    await this.credentialsRepository.transferToProject(userId, defaultProjectId);
    await this.userRepository.delete(userId);
  }
}
```

### Repository Layer
Data access abstraction via TypeORM.

```typescript
// Usage in service
const user = await this.userRepository.findOneBy({ id: userId });
await this.userRepository.save(user);
await this.userRepository.delete({ id: userId });
```

---

## Node Development Pattern
<!-- chunk: 04-node | keywords: node, development, inode, inodetype | source: packages/nodes-base/nodes/ -->

### Standard Node Structure
```typescript
// packages/nodes-base/nodes/Discord/v2/DiscordV2.node.ts
import type {
  IExecuteFunctions,
  INodeExecutionData,
  INodeType,
  INodeTypeDescription,
} from 'n8n-workflow';

export class DiscordV2 implements INodeType {
  description: INodeTypeDescription = {
    displayName: 'Discord',
    name: 'discord',
    icon: 'file:discord.svg',
    group: ['output'],
    version: 2,
    subtitle: '={{ $parameter["operation"] + ": " + $parameter["resource"] }}',
    description: 'Sends data to Discord',
    defaults: { name: 'Discord' },
    inputs: [NodeConnectionTypes.Main],
    outputs: [NodeConnectionTypes.Main],
    credentials: [
      {
        name: 'discordBotApi',
        required: true,
        displayOptions: { show: { authentication: ['botToken'] } },
      },
    ],
    properties: [
      {
        displayName: 'Authentication',
        name: 'authentication',
        type: 'options',
        options: [
          { name: 'Bot Token', value: 'botToken' },
          { name: 'OAuth2', value: 'oAuth2' },
        ],
        default: 'botToken',
      },
      // More properties...
    ],
  };

  methods = { loadOptions, listSearch };

  async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
    return await router.call(this);
  }
}
```

### Trigger Node Pattern
```typescript
export class CronTrigger implements INodeType {
  description: INodeTypeDescription = {
    displayName: 'Cron',
    name: 'cron',
    group: ['trigger', 'schedule'],
    inputs: [],
    outputs: [NodeConnectionTypes.Main],
    // ...
  };

  async trigger(this: ITriggerFunctions): Promise<ITriggerResponse> {
    const executeTrigger = () => {
      this.emit([this.helpers.returnJsonArray([{}])]);
    };

    this.helpers.registerCron({ expression: '0 * * * *' }, executeTrigger);

    return {
      manualTriggerFunction: async () => executeTrigger(),
    };
  }
}
```

### Credential Type Pattern
```typescript
// packages/nodes-base/credentials/N8nApi.credentials.ts
export class N8nApi implements ICredentialType {
  name = 'n8nApi';
  displayName = 'n8n API';
  documentationUrl = 'https://docs.n8n.io/api/';

  properties: INodeProperties[] = [
    {
      displayName: 'API Key',
      name: 'apiKey',
      type: 'string',
      typeOptions: { password: true },
      default: '',
    },
    {
      displayName: 'Base URL',
      name: 'baseUrl',
      type: 'string',
      default: '',
    },
  ];

  authenticate: IAuthenticateGeneric = {
    type: 'generic',
    properties: {
      headers: { 'X-N8N-API-KEY': '={{ $credentials.apiKey }}' },
    },
  };

  test: ICredentialTestRequest = {
    request: {
      baseURL: '={{ $credentials.baseUrl }}',
      url: '/workflows?limit=5',
    },
  };
}
```

---

## Error Handling Pattern
<!-- chunk: 04-errors | keywords: error, handling, exceptions | source: packages/cli/src/errors/ -->

### Error Class Hierarchy
```
BaseError (n8n-workflow)
├── ResponseError (HTTP errors)
│   ├── BadRequestError (400)
│   ├── AuthError (401)
│   ├── ForbiddenError (403)
│   ├── NotFoundError (404)
│   ├── ConflictError (409)
│   ├── InternalServerError (500)
│   └── ServiceUnavailableError (503)
├── UserError (user-caused errors)
├── OperationalError (operational issues)
├── UnexpectedError (unexpected conditions)
├── NodeOperationError (node-specific)
└── WorkflowOperationError (workflow-specific)
```

### Creating Custom Errors
```typescript
// packages/cli/src/errors/response-errors/not-found.error.ts
import { ResponseError } from './abstract/response.error';

export class NotFoundError extends ResponseError {
  // Type guard helper
  static isDefinedAndNotNull<T>(
    value: T | undefined | null,
    message: string,
  ): asserts value is T {
    if (value === undefined || value === null) {
      throw new NotFoundError(message);
    }
  }

  constructor(message: string, hint?: string) {
    super(message, 404, 404, hint);
  }
}
```

### Using Errors in Services
```typescript
import { UserError, UnexpectedError } from 'n8n-workflow';
import { NotFoundError } from '@/errors/response-errors/not-found.error';

@Service()
export class WorkflowService {
  async get(id: string): Promise<WorkflowEntity> {
    const workflow = await this.repository.findOne({ where: { id } });

    // Type guard pattern
    NotFoundError.isDefinedAndNotNull(workflow, `Workflow ${id} not found`);

    return workflow;
  }

  async validate(workflow: WorkflowEntity): Promise<void> {
    if (!workflow.nodes.length) {
      throw new UserError('Workflow must have at least one node');
    }
  }
}
```

---

## Frontend Store Pattern
<!-- chunk: 04-store | keywords: pinia, store, frontend, vue | source: packages/frontend/editor-ui/src/app/stores/ -->

### Composition API Store
```typescript
// packages/frontend/editor-ui/src/app/stores/rbac.store.ts
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';
import { STORES } from '@n8n/stores';

export const useRBACStore = defineStore(STORES.RBAC, () => {
  // State
  const globalRoles = ref<Role[]>([]);
  const globalScopes = ref<Scope[]>([]);
  const scopesByProjectId = ref<Record<string, Scope[]>>({});

  // Getters (computed)
  const isAdmin = computed(() => globalRoles.value.includes('global:admin'));

  const hasScope = computed(() => (scope: Scope) => {
    return globalScopes.value.includes(scope);
  });

  // Actions
  function addGlobalRole(role: Role) {
    if (!globalRoles.value.includes(role)) {
      globalRoles.value.push(role);
    }
  }

  function setGlobalScopes(scopes: Scope[]) {
    globalScopes.value = scopes;
  }

  // Return public API
  return {
    globalRoles,
    globalScopes,
    isAdmin,
    hasScope,
    addGlobalRole,
    setGlobalScopes,
  };
});
```

### Store Usage in Components
```typescript
// In Vue component
import { useRBACStore } from '@/app/stores/rbac.store';

const rbacStore = useRBACStore();

// Access state
if (rbacStore.isAdmin) {
  // Admin-only logic
}

// Call actions
rbacStore.addGlobalRole('global:member');
```

---

## Testing Patterns
<!-- chunk: 04-testing | keywords: testing, jest, vitest, mock | source: packages/cli/test/, packages/frontend/ -->

### Backend Unit Test (Jest)
```typescript
// packages/cli/src/controllers/__tests__/users.controller.test.ts
import { mock } from 'jest-mock-extended';
import type { EventService } from '@/events/event.service';

describe('UsersController', () => {
  const eventService = mock<EventService>();
  const userRepository = mock<UserRepository>();

  const controller = new UsersController(
    mock(), // logger
    userRepository,
    mock(), // projectService
    eventService,
  );

  beforeEach(() => {
    jest.restoreAllMocks();
  });

  describe('changeGlobalRole', () => {
    it('should emit user-changed-role event', async () => {
      userRepository.findOne.mockResolvedValue(mock<User>({ id: '456' }));

      await controller.changeGlobalRole(
        mock({ user: { id: '123' } }),
        mock(),
        { newRoleName: 'global:member' },
        '456',
      );

      expect(eventService.emit).toHaveBeenCalledWith('user-changed-role', {
        userId: '123',
        targetUserId: '456',
        targetUserNewRole: 'global:member',
      });
    });
  });
});
```

### Frontend Store Test (Vitest)
```typescript
// packages/frontend/editor-ui/src/app/stores/rbac.store.test.ts
import { createPinia, setActivePinia } from 'pinia';
import { useRBACStore } from '@/app/stores/rbac.store';

describe('RBAC store', () => {
  beforeEach(() => {
    setActivePinia(createPinia());
  });

  it('should add global scope', () => {
    const rbacStore = useRBACStore();
    rbacStore.addGlobalScope('workflow:list');
    expect(rbacStore.globalScopes).toContain('workflow:list');
  });

  it('should not duplicate scopes', () => {
    const rbacStore = useRBACStore();
    rbacStore.addGlobalScope('workflow:list');
    rbacStore.addGlobalScope('workflow:list');
    expect(rbacStore.globalScopes.filter(s => s === 'workflow:list')).toHaveLength(1);
  });
});
```

---

## Code Style
<!-- chunk: 04-style | keywords: style, formatting, linting | source: biome.jsonc, .prettierrc.js -->

### TypeScript Rules
- **NEVER use `any` type** — use proper types or `unknown`
- **Avoid type casting with `as`** — use type guards instead
- **Define shared interfaces in `@n8n/api-types`**

### Formatting (Biome/Prettier)
| Rule | Value |
|------|-------|
| Indent | Tab (width: 2) |
| Line width | 100 |
| Semicolons | Always |
| Trailing commas | All |
| Quotes | Single (JS), Double (JSX) |
| Line endings | LF |

### Git Hooks (Lefthook)
| Hook | Action |
|------|--------|
| `pre-commit` | Biome check, Prettier format |
| `pre-commit` | Stylelint for CSS/Vue |
| `pre-commit` | actionlint for GitHub workflows |

### Best Practices
1. **Services handle business logic** — Controllers only route
2. **Repositories for data access** — Never query DB in controllers
3. **Event-driven communication** — Use EventEmitter for loose coupling
4. **Type safety everywhere** — No implicit any
5. **Test with mocks** — Use `jest-mock-extended`

→ Debug Guide: [[06_DEBUG]]
→ Examples: [[07_EXAMPLES]]
