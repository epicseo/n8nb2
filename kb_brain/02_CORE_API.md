# 02_CORE_API.md
<!-- repo: n8n | version: 1.122.0 | generated: 2025-12-02 -->
<!-- tags: api, types, interfaces, database, entities -->

## Contents
- [REST API Controllers](#rest-api-controllers)
- [Core Services](#core-services)
- [Database Entities](#database-entities)
- [Core Interfaces](#core-interfaces)
- [Execution Types](#execution-types)
- [Connection Types](#connection-types)

---

## REST API Controllers
<!-- chunk: 02-controllers | keywords: controller, rest, http, endpoint -->

### Authentication (`/`)

| Method | Path | Purpose |
|--------|------|---------|
| POST | `/login` | Authenticate user |
| GET | `/login` | Get current user |
| POST | `/logout` | End session |

### Users (`/users`)

| Method | Path | Purpose |
|--------|------|---------|
| GET | `/` | List users |
| DELETE | `/:id` | Delete user |
| PATCH | `/:id/role` | Change role |

### Workflows (`/workflows`)

| Method | Path | Purpose |
|--------|------|---------|
| GET | `/` | List workflows |
| POST | `/` | Create workflow |
| PATCH | `/:id` | Update workflow |
| POST | `/:id/run` | Execute workflow |

### Credentials (`/credentials`)

| Method | Path | Purpose |
|--------|------|---------|
| GET | `/` | List credentials |
| POST | `/` | Create credential |
| POST | `/test` | Test credential |
| DELETE | `/:id` | Delete credential |

### Executions (`/executions`)

| Method | Path | Purpose |
|--------|------|---------|
| GET | `/` | List executions |
| GET | `/:id` | Get execution |
| DELETE | `/:id` | Delete execution |
| POST | `/:id/retry` | Retry execution |

### AI (`/ai`)

| Method | Path | Purpose |
|--------|------|---------|
| POST | `/build` | AI workflow builder |
| POST | `/chat` | AI chat |
| POST | `/ask` | AI question |

### Health

| Method | Path | Purpose |
|--------|------|---------|
| GET | `/healthz` | Liveness check |
| GET | `/healthz/readiness` | Readiness check |

---

## Core Services
<!-- chunk: 02-services | keywords: service, business-logic -->

### UserService

```typescript
update(userId: string, data: Partial<User>): Promise<void>
toPublic(user: User, options?: ToPublicOptions): Promise<PublicUser>
inviteUsers(owner: User, invitations: Invitation[]): Promise<void>
```

### CredentialsService

```typescript
getMany(user: User, options: GetManyOptions): Promise<ICredentialDataDecryptedObject[]>
test(userId: string, credentials: ICredentialsDecrypted): Promise<INodeCredentialTestResult>
decrypt(credential: Credential, fullData?: boolean): ICredentialDataDecryptedObject
```

### WorkflowService

```typescript
get(id: string, relations?: string[]): Promise<WorkflowEntity>
create(workflow: WorkflowEntity, user: User): Promise<WorkflowEntity>
update(id: string, workflow: Partial<WorkflowEntity>): Promise<WorkflowEntity>
delete(id: string): Promise<void>
activate(id: string): Promise<void>
deactivate(id: string): Promise<void>
```

### ExecutionService

```typescript
getExecutions(filter: IGetExecutionsQueryFilter): Promise<ExecutionSummaries>
stopExecution(executionId: string): Promise<StopResult>
```

---

## Database Entities
<!-- chunk: 02-entities | keywords: entity, typeorm, database -->

### WorkflowEntity

| Column | Type | Purpose |
|--------|------|---------|
| id | string | Nano-id PK |
| name | string | Display name (1-128 chars) |
| active | boolean | Activation status |
| nodes | INode[] | Node definitions (JSON) |
| connections | IConnections | Node connections (JSON) |
| settings | IWorkflowSettings | Workflow settings (JSON) |
| staticData | IDataObject | Persistent data (JSON) |
| pinData | ISimplifiedPinData | Pinned test data |

### ExecutionEntity

| Column | Type | Purpose |
|--------|------|---------|
| id | string | Auto-generated PK |
| mode | WorkflowExecuteMode | Execution mode |
| status | ExecutionStatus | Current status |
| workflowId | string | Reference to workflow |
| startedAt | Date | Start timestamp |
| stoppedAt | Date | Stop timestamp |
| waitTill | Date | Wait-until timestamp |

### CredentialsEntity

| Column | Type | Purpose |
|--------|------|---------|
| id | string | Nano-id PK |
| name | string | Credential name (3-128 chars) |
| type | string | Credential type |
| data | string | Encrypted data |

### User

| Column | Type | Purpose |
|--------|------|---------|
| id | string | UUID PK |
| email | string | Unique email |
| password | string | Hashed password |
| mfaEnabled | boolean | 2FA status |

### Project

| Column | Type | Purpose |
|--------|------|---------|
| id | string | Nano-id PK |
| name | string | Project name |
| type | string | 'personal' or 'team' |

### SharedWorkflow / SharedCredentials

| Column | Type | Purpose |
|--------|------|---------|
| workflowId/credentialsId | string | Resource FK |
| projectId | string | Project FK |
| role | string | Permission level |

---

## Core Interfaces
<!-- chunk: 02-interfaces | keywords: interface, types, typescript -->

### INode
```typescript
interface INode {
  id: string;
  name: string;
  type: string;                    // e.g., "n8n-nodes-base.slack"
  typeVersion: number;
  position: [number, number];      // [x, y] coordinates
  disabled?: boolean;
  parameters: INodeParameters;
  credentials?: INodeCredentials;
  // Error handling
  continueOnFail?: boolean;
  retryOnFail?: boolean;
  maxTries?: number;
  onError?: 'continueErrorOutput' | 'continueRegularOutput' | 'stopWorkflow';
}
```

### INodeType
```typescript
interface INodeType {
  description: INodeTypeDescription;
  execute?(this: IExecuteFunctions): Promise<INodeExecutionData[][]>;
  trigger?(this: ITriggerFunctions): Promise<ITriggerResponse | undefined>;
  webhook?(this: IWebhookFunctions): Promise<IWebhookResponseData>;
  poll?(this: IPollFunctions): Promise<INodeExecutionData[][] | null>;
  methods?: {
    loadOptions?: Record<string, (this: ILoadOptionsFunctions) => Promise<INodePropertyOptions[]>>;
    credentialTest?: Record<string, ICredentialTestFunction>;
  };
}
```

### INodeExecutionData
```typescript
interface INodeExecutionData {
  json: IDataObject;                 // Primary JSON data
  binary?: IBinaryKeyData;           // Binary files
  error?: NodeError;                 // Error info
  pairedItem?: IPairedItemData;      // Source tracking
}
```

### IWorkflowBase
```typescript
interface IWorkflowBase {
  id: string;
  name: string;
  active: boolean;
  nodes: INode[];
  connections: IConnections;
  settings?: IWorkflowSettings;
  staticData?: IDataObject;
  pinData?: IPinData;
}
```

### IWorkflowSettings
```typescript
interface IWorkflowSettings {
  timezone?: string;
  errorWorkflow?: string;
  executionTimeout?: number;
  saveDataErrorExecution?: 'all' | 'none';
  saveDataSuccessExecution?: 'all' | 'none';
  saveManualExecutions?: boolean;
  executionOrder?: 'v0' | 'v1';
}
```

### IConnections
```typescript
interface IConnections {
  [sourceNodeName: string]: {
    [connectionType: string]: IConnection[][];
  };
}

interface IConnection {
  node: string;           // Target node name
  type: NodeConnectionType;
  index: number;          // Input index
}
```

---

## Execution Types
<!-- chunk: 02-execution | keywords: execution, run, status -->

### WorkflowExecute Class

```typescript
class WorkflowExecute {
  constructor(
    additionalData: IWorkflowExecuteAdditionalData,
    mode: WorkflowExecuteMode,
    runExecutionData?: IRunExecutionData
  );

  run(
    workflow: Workflow,
    startNode?: INode,
    destinationNode?: IDestinationNode,
    pinData?: IPinData
  ): PCancelable<IRun>;

  runPartialWorkflow2(
    workflow: Workflow,
    runData: IRunData,
    pinData?: IPinData,
    dirtyNodeNames?: string[]
  ): PCancelable<IRun>;
}
```

### ExecuteContext Helpers

```typescript
interface ExecuteHelpers {
  // Data
  returnJsonArray(values: unknown[]): INodeExecutionData[];
  normalizeItems(items: unknown[]): INodeExecutionData[];

  // Requests
  request(options: RequestOptions): Promise<any>;
  requestWithAuthentication(credType: string, options: RequestOptions): Promise<any>;

  // Binary
  assertBinaryData(itemIndex: number, propertyName: string): void;
  getBinaryDataBuffer(itemIndex: number, propertyName: string): Promise<Buffer>;
  prepareBinaryData(buffer: Buffer, fileName?: string, mimeType?: string): Promise<IBinaryData>;
}
```

### IRun
```typescript
interface IRun {
  data: IRunExecutionData;
  finished?: boolean;
  mode: WorkflowExecuteMode;
  startedAt: Date;
  stoppedAt?: Date;
  status: ExecutionStatus;
  waitTill?: Date | null;
}
```

### ExecutionStatus
```typescript
type ExecutionStatus =
  | 'canceled' | 'crashed' | 'error'
  | 'new' | 'running' | 'success'
  | 'unknown' | 'waiting';
```

### WorkflowExecuteMode
```typescript
type WorkflowExecuteMode =
  | 'cli' | 'error' | 'integrated' | 'internal'
  | 'manual' | 'retry' | 'trigger' | 'webhook'
  | 'evaluation' | 'chat';
```

---

## Connection Types
<!-- chunk: 02-connections | keywords: connection, types, ai -->

### NodeConnectionTypes

| Type | Purpose |
|------|---------|
| `main` | Standard data flow |
| `ai_languageModel` | LLM provider (max 1) |
| `ai_memory` | Conversation history (max 1) |
| `ai_tool` | Agent tools (many) |
| `ai_vectorStore` | Vector database (max 1) |
| `ai_embedding` | Embedding model |
| `ai_retriever` | Document retriever |
| `ai_outputParser` | Output parser |
| `ai_textSplitter` | Text splitter |
| `ai_document` | Document loader |
| `ai_agent` | Sub-agent |
| `ai_chain` | LangChain chain |
| `ai_reranker` | Reranker model |

### Property Types

```typescript
type NodePropertyTypes =
  | 'boolean' | 'number' | 'string'
  | 'options' | 'multiOptions' | 'collection'
  | 'fixedCollection' | 'json' | 'color'
  | 'dateTime' | 'filter' | 'hidden'
  | 'resourceLocator' | 'resourceMapper'
  | 'credentialsSelect' | 'workflowSelector'
  | 'assignment' | 'assignmentCollection';
```

---

## Helper Functions
<!-- chunk: 02-helpers | keywords: helpers, binary, request -->

### Request Helpers

```typescript
httpRequest(requestOptions: IHttpRequestOptions): Promise<any>
httpRequestWithAuthentication(credType: string, options: IHttpRequestOptions): Promise<any>
requestWithAuthenticationPaginated(options: IRequestOptions, itemIndex: number, pagination: PaginationOptions): Promise<any[]>
```

### Binary Helpers

```typescript
binaryToBuffer(body: Buffer | Readable): Promise<Buffer>
binaryToString(body: Buffer | Readable, encoding?: BufferEncoding): Promise<string>
getBinaryStream(binaryDataId: string): Promise<Readable>
prepareBinaryData(data: Buffer | Readable, filePath?: string, mimeType?: string): Promise<IBinaryData>
getBinaryDataBuffer(itemIndex: number, propertyName: string): Promise<Buffer>
```

### File System Helpers

```typescript
createReadStream(path: string): Readable
getStoragePath(): string
writeContentToFile(path: string, content: string | Buffer, flag?: string): Promise<void>
```

---

## Type Statistics

| Category | Count |
|----------|-------|
| API Endpoints | 30+ |
| Services | 10+ |
| Database Entities | 15 |
| Core Interfaces | 50+ |
| Execution Types | 20+ |
| Connection Types | 13 |
| **Total Types** | **100+** |

---

*Source: packages/cli/src/, packages/@n8n/db/src/, packages/workflow/src/*
