# 07_EXAMPLES.md
<!-- repo: n8n | branch: claude/github-to-kb-converter-01JAurUearApTk4RAu9NPFAg | commit: 3c0e809e | generated: 2025-11-28 -->
<!-- tags: examples, code, templates, recipes, howto -->

## Contents
- [Quick Start](#quick-start)
- [Node Development](#node-development)
- [Service Development](#service-development)
- [Frontend Development](#frontend-development)
- [Testing Examples](#testing-examples)
- [API Usage](#api-usage)

---

## Quick Start
<!-- chunk: 07-quickstart | keywords: quickstart, setup, development | source: AGENTS.md -->

### Development Setup

```bash
# Clone repository
git clone https://github.com/n8n-io/n8n.git
cd n8n

# Install dependencies
pnpm install

# Build all packages
pnpm build > build.log 2>&1

# Start development server
pnpm dev
```

### Running Commands

```bash
# Build (always redirect output)
pnpm build > build.log 2>&1
tail -n 20 build.log

# Type checking
pnpm typecheck

# Linting
pnpm lint

# Testing
pnpm test

# Format code
pnpm format
```

### Working with Packages

```bash
# Navigate to package
pushd packages/cli

# Run package-specific commands
pnpm test
pnpm lint

# Return to root
popd
```

---

## Node Development
<!-- chunk: 07-node | keywords: node, development, inodetype | source: packages/nodes-base/nodes/ -->

### Basic Node Template

**Goal:** Create a simple node implementation
**API:** → [[02_CORE_API#inode-type]]

```typescript
// packages/nodes-base/nodes/MyNode/MyNode.node.ts
import type {
  IExecuteFunctions,
  INodeExecutionData,
  INodeType,
  INodeTypeDescription,
} from 'n8n-workflow';
import { NodeConnectionTypes } from 'n8n-workflow';

export class MyNode implements INodeType {
  description: INodeTypeDescription = {
    displayName: 'My Node',
    name: 'myNode',
    icon: 'file:myNode.svg',
    group: ['transform'],
    version: 1,
    description: 'Description of what this node does',
    defaults: {
      name: 'My Node',
    },
    inputs: [NodeConnectionTypes.Main],
    outputs: [NodeConnectionTypes.Main],
    properties: [
      {
        displayName: 'Operation',
        name: 'operation',
        type: 'options',
        noDataExpression: true,
        options: [
          { name: 'Get', value: 'get' },
          { name: 'Create', value: 'create' },
        ],
        default: 'get',
      },
      {
        displayName: 'ID',
        name: 'id',
        type: 'string',
        default: '',
        required: true,
        displayOptions: {
          show: { operation: ['get'] },
        },
      },
    ],
  };

  async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
    const items = this.getInputData();
    const returnData: INodeExecutionData[] = [];
    const operation = this.getNodeParameter('operation', 0) as string;

    for (let i = 0; i < items.length; i++) {
      try {
        if (operation === 'get') {
          const id = this.getNodeParameter('id', i) as string;
          // Your logic here
          returnData.push({ json: { id, result: 'success' } });
        }
      } catch (error) {
        if (this.continueOnFail()) {
          returnData.push({ json: { error: error.message } });
          continue;
        }
        throw error;
      }
    }

    return [returnData];
  }
}
```

---

### Trigger Node Template

**Goal:** Create a trigger/polling node
**API:** → [[02_CORE_API#trigger-context]]

```typescript
// packages/nodes-base/nodes/MyTrigger/MyTrigger.node.ts
import type {
  ITriggerFunctions,
  INodeType,
  INodeTypeDescription,
  ITriggerResponse,
} from 'n8n-workflow';
import { NodeConnectionTypes } from 'n8n-workflow';

export class MyTrigger implements INodeType {
  description: INodeTypeDescription = {
    displayName: 'My Trigger',
    name: 'myTrigger',
    icon: 'file:myTrigger.svg',
    group: ['trigger'],
    version: 1,
    description: 'Triggers on events',
    defaults: { name: 'My Trigger' },
    inputs: [],
    outputs: [NodeConnectionTypes.Main],
    properties: [
      {
        displayName: 'Interval',
        name: 'interval',
        type: 'number',
        default: 60,
        description: 'Interval in seconds',
      },
    ],
  };

  async trigger(this: ITriggerFunctions): Promise<ITriggerResponse> {
    const interval = this.getNodeParameter('interval') as number;

    const executeTrigger = () => {
      const data = { timestamp: new Date().toISOString() };
      this.emit([this.helpers.returnJsonArray([data])]);
    };

    // Register cron job
    this.helpers.registerCron(
      { expression: `*/${interval} * * * * *` },
      executeTrigger,
    );

    return {
      manualTriggerFunction: async () => executeTrigger(),
    };
  }
}
```

---

### Credential Type Template

**Goal:** Create a credential type
**API:** → [[02_CORE_API]]

```typescript
// packages/nodes-base/credentials/MyApi.credentials.ts
import type {
  IAuthenticateGeneric,
  ICredentialTestRequest,
  ICredentialType,
  INodeProperties,
} from 'n8n-workflow';

export class MyApi implements ICredentialType {
  name = 'myApi';
  displayName = 'My API';
  documentationUrl = 'https://docs.example.com';

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
      default: 'https://api.example.com',
    },
  ];

  authenticate: IAuthenticateGeneric = {
    type: 'generic',
    properties: {
      headers: {
        Authorization: '=Bearer {{ $credentials.apiKey }}',
      },
    },
  };

  test: ICredentialTestRequest = {
    request: {
      baseURL: '={{ $credentials.baseUrl }}',
      url: '/health',
    },
  };
}
```

---

## Service Development
<!-- chunk: 07-service | keywords: service, backend, controller | source: packages/cli/src/services/ -->

### Service Template

**Goal:** Create a backend service
**API:** → [[02_CORE_API#cli-services]]

```typescript
// packages/cli/src/services/my.service.ts
import { Service } from '@n8n/di';
import { Logger } from '@n8n/backend-common';
import type { User } from '@n8n/db';

@Service()
export class MyService {
  constructor(
    private readonly logger: Logger,
    private readonly myRepository: MyRepository,
    private readonly eventService: EventService,
  ) {}

  async findAll(user: User): Promise<MyEntity[]> {
    this.logger.debug('Finding all entities for user', { userId: user.id });
    return this.myRepository.find({ where: { userId: user.id } });
  }

  async create(user: User, data: CreateMyDto): Promise<MyEntity> {
    const entity = this.myRepository.create({
      ...data,
      userId: user.id,
    });

    const saved = await this.myRepository.save(entity);

    this.eventService.emit('my-entity-created', {
      userId: user.id,
      entityId: saved.id,
    });

    return saved;
  }

  async delete(user: User, id: string): Promise<void> {
    const entity = await this.myRepository.findOneBy({ id, userId: user.id });

    if (!entity) {
      throw new NotFoundError(`Entity ${id} not found`);
    }

    await this.myRepository.delete(id);
  }
}
```

---

### Controller Template

**Goal:** Create a REST controller
**API:** → [[02_CORE_API#cli-controllers]]

```typescript
// packages/cli/src/controllers/my.controller.ts
import { RestController, Get, Post, Delete, Body, Param } from '@n8n/decorators';
import type { AuthenticatedRequest } from '@n8n/api-types';

@RestController('/my-resource')
export class MyController {
  constructor(
    private readonly myService: MyService,
  ) {}

  @Get('/')
  async getAll(req: AuthenticatedRequest) {
    return this.myService.findAll(req.user);
  }

  @Get('/:id')
  async getOne(req: AuthenticatedRequest, @Param('id') id: string) {
    return this.myService.findOne(req.user, id);
  }

  @Post('/')
  async create(req: AuthenticatedRequest, @Body() body: CreateMyDto) {
    return this.myService.create(req.user, body);
  }

  @Delete('/:id')
  async delete(req: AuthenticatedRequest, @Param('id') id: string) {
    await this.myService.delete(req.user, id);
    return { success: true };
  }
}
```

---

## Frontend Development
<!-- chunk: 07-frontend | keywords: frontend, vue, pinia, store | source: packages/frontend/editor-ui/src/ -->

### Pinia Store Template

**Goal:** Create a Pinia store
**API:** → [[04_PATTERNS#frontend-store-pattern]]

```typescript
// packages/frontend/editor-ui/src/app/stores/my.store.ts
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';
import { STORES } from '@n8n/stores';

export const useMyStore = defineStore(STORES.MY, () => {
  // State
  const items = ref<MyItem[]>([]);
  const loading = ref(false);
  const error = ref<string | null>(null);

  // Getters (computed)
  const itemCount = computed(() => items.value.length);

  const getItemById = computed(() => (id: string) => {
    return items.value.find(item => item.id === id);
  });

  // Actions
  async function fetchItems() {
    loading.value = true;
    error.value = null;

    try {
      const response = await api.getMyItems();
      items.value = response.data;
    } catch (e) {
      error.value = e.message;
    } finally {
      loading.value = false;
    }
  }

  function addItem(item: MyItem) {
    items.value.push(item);
  }

  function removeItem(id: string) {
    const index = items.value.findIndex(item => item.id === id);
    if (index !== -1) {
      items.value.splice(index, 1);
    }
  }

  function reset() {
    items.value = [];
    loading.value = false;
    error.value = null;
  }

  return {
    // State
    items,
    loading,
    error,
    // Getters
    itemCount,
    getItemById,
    // Actions
    fetchItems,
    addItem,
    removeItem,
    reset,
  };
});
```

---

### Vue Component Template

**Goal:** Create a Vue component using store

```vue
<!-- packages/frontend/editor-ui/src/app/components/MyComponent.vue -->
<script setup lang="ts">
import { computed, onMounted } from 'vue';
import { useMyStore } from '@/app/stores/my.store';
import { useI18n } from 'vue-i18n';

const myStore = useMyStore();
const { t } = useI18n();

const items = computed(() => myStore.items);
const isLoading = computed(() => myStore.loading);

onMounted(async () => {
  await myStore.fetchItems();
});

function handleDelete(id: string) {
  myStore.removeItem(id);
}
</script>

<template>
  <div class="my-component">
    <h2>{{ t('myComponent.title') }}</h2>

    <n8n-loading v-if="isLoading" />

    <ul v-else>
      <li v-for="item in items" :key="item.id">
        {{ item.name }}
        <n8n-button
          size="small"
          type="danger"
          @click="handleDelete(item.id)"
        >
          {{ t('generic.delete') }}
        </n8n-button>
      </li>
    </ul>
  </div>
</template>

<style lang="scss" scoped>
.my-component {
  padding: var(--spacing-sm);

  h2 {
    font-size: var(--font-size-lg);
    margin-bottom: var(--spacing-md);
  }

  ul {
    list-style: none;
    padding: 0;
  }

  li {
    display: flex;
    justify-content: space-between;
    padding: var(--spacing-xs);
    border-bottom: var(--border);
  }
}
</style>
```

---

## Testing Examples
<!-- chunk: 07-testing | keywords: testing, jest, vitest, mock | source: packages/cli/test/, packages/frontend/ -->

### Backend Unit Test

**Goal:** Test a service with mocks
**API:** → [[04_PATTERNS#testing-patterns]]

```typescript
// packages/cli/src/services/__tests__/my.service.test.ts
import { mock } from 'jest-mock-extended';
import { MyService } from '../my.service';

describe('MyService', () => {
  const logger = mock<Logger>();
  const myRepository = mock<MyRepository>();
  const eventService = mock<EventService>();

  const service = new MyService(logger, myRepository, eventService);

  beforeEach(() => {
    jest.restoreAllMocks();
  });

  describe('findAll', () => {
    it('should return all entities for user', async () => {
      const user = mock<User>({ id: 'user-1' });
      const entities = [{ id: '1', name: 'Entity 1' }];

      myRepository.find.mockResolvedValue(entities);

      const result = await service.findAll(user);

      expect(result).toEqual(entities);
      expect(myRepository.find).toHaveBeenCalledWith({
        where: { userId: 'user-1' },
      });
    });
  });

  describe('create', () => {
    it('should create entity and emit event', async () => {
      const user = mock<User>({ id: 'user-1' });
      const data = { name: 'New Entity' };
      const savedEntity = { id: 'new-1', ...data, userId: 'user-1' };

      myRepository.create.mockReturnValue(savedEntity);
      myRepository.save.mockResolvedValue(savedEntity);

      const result = await service.create(user, data);

      expect(result).toEqual(savedEntity);
      expect(eventService.emit).toHaveBeenCalledWith('my-entity-created', {
        userId: 'user-1',
        entityId: 'new-1',
      });
    });
  });
});
```

---

### Frontend Store Test

**Goal:** Test a Pinia store

```typescript
// packages/frontend/editor-ui/src/app/stores/__tests__/my.store.test.ts
import { createPinia, setActivePinia } from 'pinia';
import { useMyStore } from '../my.store';

vi.mock('@/api', () => ({
  getMyItems: vi.fn(),
}));

describe('MyStore', () => {
  beforeEach(() => {
    setActivePinia(createPinia());
  });

  it('should initialize with empty state', () => {
    const store = useMyStore();

    expect(store.items).toEqual([]);
    expect(store.loading).toBe(false);
    expect(store.error).toBeNull();
  });

  it('should add item', () => {
    const store = useMyStore();
    const item = { id: '1', name: 'Test' };

    store.addItem(item);

    expect(store.items).toContain(item);
    expect(store.itemCount).toBe(1);
  });

  it('should remove item', () => {
    const store = useMyStore();
    store.items = [{ id: '1', name: 'Test' }];

    store.removeItem('1');

    expect(store.items).toEqual([]);
  });
});
```

---

## API Usage
<!-- chunk: 07-api | keywords: api, http, request, rest | source: analysis -->

### REST API Examples

**Create Workflow:**
```bash
curl -X POST http://localhost:5678/api/v1/workflows \
  -H "Content-Type: application/json" \
  -H "X-N8N-API-KEY: your-api-key" \
  -d '{
    "name": "My Workflow",
    "nodes": [],
    "connections": {},
    "active": false
  }'
```

**Execute Workflow:**
```bash
curl -X POST http://localhost:5678/api/v1/workflows/{workflowId}/execute \
  -H "X-N8N-API-KEY: your-api-key"
```

**Get Executions:**
```bash
curl http://localhost:5678/api/v1/executions \
  -H "X-N8N-API-KEY: your-api-key"
```

→ Full API Reference: [[02_CORE_API]]
→ Quick Reference: [[08_QUICK_REF]]
