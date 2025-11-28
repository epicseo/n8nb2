# 03_DATA_MODELS.md
<!-- repo: n8n | branch: claude/github-to-kb-converter-01JAurUearApTk4RAu9NPFAg | commit: 3c0e809e | generated: 2025-11-28 -->
<!-- tags: database, typeorm, entities, interfaces, types -->

## Contents
- [TypeORM Entities](#typeorm-entities)
- [Core Interfaces](#core-interfaces)
- [Enums and Constants](#enums-and-constants)
- [Relationship Diagram](#relationship-diagram)

---

## TypeORM Entities
<!-- chunk: 03-entities | keywords: entity, typeorm, database | source: packages/@n8n/db/src/entities/ -->

### WorkflowEntity
**File:** `packages/@n8n/db/src/entities/workflow-entity.ts`

| Column | Type | Constraints | Purpose |
|--------|------|-------------|---------|
| id | string | PK (nano-id) | Unique workflow identifier |
| name | string | Length: 1-128, Unique | Workflow display name |
| description | string \| null | text | Detailed description |
| active | boolean | | Activation status |
| isArchived | boolean | default: false | Soft-delete flag |
| nodes | INode[] | JSON | Node definitions |
| connections | IConnections | JSON | Node connections |
| settings | IWorkflowSettings | JSON | Workflow settings |
| staticData | IDataObject | JSON | Persistent data |
| meta | WorkflowFEMeta | JSON | Frontend metadata |
| versionId | string | Length: 36 | Current version |
| triggerCount | number | default: 0 | Trigger execution counter |
| pinData | ISimplifiedPinData | JSON | Pinned node data |
| createdAt | Date | Timestamp | Creation time |
| updatedAt | Date | Timestamp | Last update time |

**Relationships:**
- `tags` → ManyToMany with TagEntity
- `shared` → OneToMany with SharedWorkflow
- `statistics` → OneToMany with WorkflowStatistics
- `parentFolder` → ManyToOne with Folder (CASCADE)

---

### ExecutionEntity
**File:** `packages/@n8n/db/src/entities/execution-entity.ts`

| Column | Type | Purpose |
|--------|------|---------|
| id | string | Auto-generated PK |
| mode | WorkflowExecuteMode | Execution mode |
| status | ExecutionStatus | Current status |
| createdAt | Date | Creation timestamp |
| startedAt | Date \| null | Actual start time |
| stoppedAt | Date | Execution stop time |
| deletedAt | Date \| null | Soft delete timestamp |
| workflowId | string \| null | Reference to workflow |
| waitTill | Date \| null | Wait-until timestamp |
| retryOf | string \| null | Original execution ID |
| retrySuccessId | string \| null | Successful retry ID |

**Relationships:**
- `metadata` → OneToMany with ExecutionMetadata
- `executionData` → OneToOne with ExecutionData
- `workflow` → ManyToOne with WorkflowEntity
- `annotation` → OneToOne with ExecutionAnnotation (EE)

**Indexes:**
- `(workflowId, id)` - Composite
- `(waitTill, id)` - Composite
- `(workflowId, finished, id)` - Composite

---

### CredentialsEntity
**File:** `packages/@n8n/db/src/entities/credentials-entity.ts`

| Column | Type | Constraints | Purpose |
|--------|------|-------------|---------|
| id | string | PK (nano-id) | Unique credential ID |
| name | string | Length: 3-128 | Credential name |
| data | string | text | Encrypted credential data |
| type | string | Length: 128, Indexed | Credential type |
| isManaged | boolean | default: false | Managed by n8n |
| isGlobal | boolean | default: false | Available to all users |
| createdAt | Date | Timestamp | Creation time |
| updatedAt | Date | Timestamp | Last update time |

**Relationships:**
- `shared` → OneToMany with SharedCredentials

---

### User
**File:** `packages/@n8n/db/src/entities/user.ts`

| Column | Type | Purpose |
|--------|------|---------|
| id | string | UUID primary key |
| email | string \| null | Unique, lowercase |
| firstName | string \| null | Length: 32 |
| lastName | string \| null | Length: 32 |
| password | string \| null | Hashed password |
| personalizationAnswers | IPersonalizationSurveyAnswers | JSON |
| settings | IUserSettings | JSON preferences |
| disabled | boolean | Account status |
| mfaEnabled | boolean | 2FA status |
| mfaSecret | string \| null | 2FA secret |
| mfaRecoveryCodes | string[] | Recovery codes |
| isPending | boolean | Pending setup |
| lastActiveAt | Date \| null | Last activity |

**Relationships:**
- `role` → ManyToOne with Role
- `authIdentities` → OneToMany with AuthIdentity
- `apiKeys` → OneToMany with ApiKey
- `projectRelations` → OneToMany with ProjectRelation

---

### Project
**File:** `packages/@n8n/db/src/entities/project.ts`

| Column | Type | Purpose |
|--------|------|---------|
| id | string | Nano-id primary key |
| name | string | Length: 255 |
| type | 'personal' \| 'team' | Project type |
| icon | {type, value} \| null | Icon configuration |
| description | string \| null | Length: 512 |

**Relationships:**
- `projectRelations` → OneToMany with ProjectRelation
- `sharedCredentials` → OneToMany with SharedCredentials
- `sharedWorkflows` → OneToMany with SharedWorkflow

---

### ExecutionData
**File:** `packages/@n8n/db/src/entities/execution-data.ts`

| Column | Type | Purpose |
|--------|------|---------|
| executionId | string | PK, FK to ExecutionEntity |
| data | string | text, Execution results |
| workflowData | IWorkflowBase | JSON, Workflow snapshot |

**Relationships:**
- `execution` → OneToOne with ExecutionEntity (CASCADE)

---

### SharedWorkflow
**File:** `packages/@n8n/db/src/entities/shared-workflow.ts`

| Column | Type | Purpose |
|--------|------|---------|
| workflowId | string | PK, FK to workflow |
| projectId | string | PK, FK to project |
| role | WorkflowSharingRole | Permission level |

---

### SharedCredentials
**File:** `packages/@n8n/db/src/entities/shared-credentials.ts`

| Column | Type | Purpose |
|--------|------|---------|
| credentialsId | string | PK, FK to credential |
| projectId | string | PK, FK to project |
| role | CredentialSharingRole | Permission level |

---

### TagEntity
**File:** `packages/@n8n/db/src/entities/tag-entity.ts`

| Column | Type | Constraints | Purpose |
|--------|------|-------------|---------|
| id | string | Nano-id PK | Tag ID |
| name | string | Length: 1-24, Unique | Tag name |

**Relationships:**
- `workflows` → ManyToMany with WorkflowEntity

---

### Folder
**File:** `packages/@n8n/db/src/entities/folder.ts`

| Column | Type | Purpose |
|--------|------|---------|
| id | string | Nano-id PK |
| name | string | Folder name |
| parentFolderId | string \| null | Parent folder ref |
| projectId | string | Home project ref |

**Relationships:**
- `parentFolder` → ManyToOne with Folder (CASCADE)
- `subFolders` → OneToMany with Folder
- `workflows` → OneToMany with WorkflowEntity

---

### WebhookEntity
**File:** `packages/@n8n/db/src/entities/webhook-entity.ts`

| Column | Type | Purpose |
|--------|------|---------|
| webhookPath | string | PK, Webhook path |
| method | IHttpRequestMethods | PK, HTTP method |
| workflowId | string | FK to workflow |
| node | string | Node name |
| webhookId | string \| null | Unique webhook ID |

---

### ApiKey
**File:** `packages/@n8n/db/src/entities/api-key.ts`

| Column | Type | Purpose |
|--------|------|---------|
| id | string | Nano-id PK |
| userId | string | FK to user |
| label | string | Unique with userId |
| scopes | ApiKeyScope[] | Permission scopes |
| apiKey | string | Unique, Indexed |
| audience | ApiKeyAudience | default: 'public-api' |

---

### WorkflowHistory
**File:** `packages/@n8n/db/src/entities/workflow-history.ts`

| Column | Type | Purpose |
|--------|------|---------|
| versionId | string | PK, Version ID |
| workflowId | string | FK to workflow |
| nodes | INode[] | JSON, Nodes at version |
| connections | IConnections | JSON, Connections at version |
| authors | string | Version authors |

---

### WorkflowStatistics
**File:** `packages/@n8n/db/src/entities/workflow-statistics.ts`

| Column | Type | Purpose |
|--------|------|---------|
| name | StatisticsNames | PK, Statistic type |
| workflowId | string | PK, FK to workflow |
| count | number | Total count |
| rootCount | number | Root-level count |
| latestEvent | Date | Latest update |

---

### Role & Scope
**File:** `packages/@n8n/db/src/entities/role.ts`

**Role Entity:**
| Column | Type | Purpose |
|--------|------|---------|
| slug | string | PK, Role identifier |
| displayName | string | Display name |
| systemRole | boolean | System-managed |
| roleType | string | global/project/workflow/credential |

**Scope Entity:**
| Column | Type | Purpose |
|--------|------|---------|
| slug | string | PK, Scope identifier |
| displayName | string | Display name |

---

## Core Interfaces
<!-- chunk: 03-interfaces | keywords: interface, types, typescript | source: packages/workflow/src/interfaces.ts -->

### INode
```typescript
interface INode {
  id: string;                        // Node identifier
  name: string;                      // Display name
  typeVersion: number;               // Node type version
  type: string;                      // Node type (e.g., "n8n-nodes-base.slack")
  position: [number, number];        // [x, y] coordinates
  disabled?: boolean;                // Execution disabled
  notes?: string;                    // Notes/documentation
  notesInFlow?: boolean;             // Show notes in workflow
  retryOnFail?: boolean;             // Retry on failure
  maxTries?: number;                 // Maximum retry attempts
  waitBetweenTries?: number;         // Wait milliseconds
  alwaysOutputData?: boolean;        // Always output data
  executeOnce?: boolean;             // Execute only once
  onError?: OnError;                 // Error handling mode
  continueOnFail?: boolean;          // Continue on error
  parameters: INodeParameters;       // Node configuration
  credentials?: INodeCredentials;    // Credentials used
  webhookId?: string;                // Webhook identifier
}
```

---

### IConnection & IConnections
```typescript
interface IConnection {
  node: string;                      // Target node name
  type: NodeConnectionType;          // Connection type
  index: number;                     // Input index on destination
}

interface IConnections {
  [sourceNodeName: string]: {
    [connectionType: string]: IConnection[][];
  };
}
```

---

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

---

### INodeTypeDescription
```typescript
interface INodeTypeDescription {
  displayName: string;
  name: string;
  icon?: string;
  group: string[];
  version: number | number[];
  description: string;
  defaults: NodeDefaults;
  inputs: Array<NodeConnectionType | INodeInputConfiguration>;
  outputs: Array<NodeConnectionType | INodeOutputConfiguration>;
  properties: INodeProperties[];
  credentials?: INodeCredentialDescription[];
  webhooks?: IWebhookDescription[];
  polling?: boolean;
  maxNodes?: number;
}
```

---

### INodeExecutionData
```typescript
interface INodeExecutionData {
  json: IDataObject;                              // Primary data
  binary?: IBinaryKeyData;                        // Binary files
  error?: NodeApiError | NodeOperationError;      // Error info
  pairedItem?: IPairedItemData | number;          // Source pairing
  metadata?: {
    subExecution: RelatedExecution;
  };
}
```

---

### IRunExecutionData
```typescript
interface IRunExecutionData {
  version: 1;
  startData?: {
    startNodes?: StartNodeData[];
    destinationNode?: IDestinationNode;
  };
  resultData: {
    error?: ExecutionError;
    runData: IRunData;
    pinData?: IPinData;
    lastNodeExecuted?: string;
  };
  executionData?: {
    contextData: IExecuteContextData;
    nodeExecutionStack: IExecuteData[];
    waitingExecution: IWaitingForExecution;
  };
  waitTill?: Date;
}
```

---

### IWorkflowSettings
```typescript
interface IWorkflowSettings {
  saveDataSuccessExecution?: 'all' | 'none';
  saveDataErrorExecution?: 'all' | 'none';
  saveManualExecutions?: boolean;
  saveExecutionProgress?: boolean;
  executionTimeout?: number;
  errorWorkflow?: string;
  timezone?: string;
  executionOrder?: 'v0' | 'v1';
  callerPolicy?: WorkflowCallerPolicyDefaultOption;
}
```

---

## Enums and Constants
<!-- chunk: 03-enums | keywords: enum, constants, status | source: packages/workflow/src/ -->

### ExecutionStatus
**File:** `packages/workflow/src/execution-status.ts`
```typescript
type ExecutionStatus =
  | 'canceled'
  | 'crashed'
  | 'error'
  | 'new'
  | 'running'
  | 'success'
  | 'unknown'
  | 'waiting';
```

---

### WorkflowExecuteMode
```typescript
type WorkflowExecuteMode =
  | 'cli'
  | 'error'
  | 'integrated'
  | 'internal'
  | 'manual'
  | 'retry'
  | 'trigger'
  | 'webhook'
  | 'evaluation'
  | 'chat';
```

---

### NodeConnectionTypes
```typescript
const NodeConnectionTypes = {
  Main: 'main',
  AiAgent: 'ai_agent',
  AiChain: 'ai_chain',
  AiDocument: 'ai_document',
  AiEmbedding: 'ai_embedding',
  AiLanguageModel: 'ai_languageModel',
  AiMemory: 'ai_memory',
  AiOutputParser: 'ai_outputParser',
  AiRetriever: 'ai_retriever',
  AiTool: 'ai_tool',
  AiVectorStore: 'ai_vectorStore',
};
```

---

### StatisticsNames
```typescript
enum StatisticsNames {
  productionSuccess = 'production_success',
  productionError = 'production_error',
  manualSuccess = 'manual_success',
  manualError = 'manual_error',
  dataLoaded = 'data_loaded',
}
```

---

### AuthProviderType
```typescript
type AuthProviderType = 'ldap' | 'email' | 'saml' | 'oidc';
```

---

### ProjectType
```typescript
type ProjectType = 'personal' | 'team';
```

---

## Relationship Diagram
<!-- chunk: 03-relationships | keywords: diagram, relationships, erd | source: analysis -->

```
┌─────────────┐       ┌─────────────────┐       ┌─────────────┐
│    User     │───────│ ProjectRelation │───────│   Project   │
└─────────────┘       └─────────────────┘       └─────────────┘
      │                                               │
      │                                               │
      ▼                                               ▼
┌─────────────┐       ┌─────────────────┐       ┌─────────────┐
│   ApiKey    │       │ SharedWorkflow  │◄──────│  Workflow   │
└─────────────┘       └─────────────────┘       └─────────────┘
                                                      │
                            ┌─────────────────────────┼─────────────────────────┐
                            │                         │                         │
                            ▼                         ▼                         ▼
                      ┌───────────┐           ┌─────────────┐           ┌───────────────┐
                      │    Tag    │           │  Execution  │           │WorkflowHistory│
                      └───────────┘           └─────────────┘           └───────────────┘
                                                    │
                                                    ▼
                                             ┌─────────────┐
                                             │ExecutionData│
                                             └─────────────┘

┌─────────────────┐       ┌───────────────────┐
│SharedCredentials│◄──────│   Credentials     │
└─────────────────┘       └───────────────────┘

┌─────────────┐       ┌─────────────┐
│    Role     │───────│    Scope    │  (ManyToMany)
└─────────────┘       └─────────────┘
```

**Relationship Types:**
- **ManyToMany:** Workflow ↔ Tag, Role ↔ Scope
- **OneToMany:** User → ApiKey, Workflow → Execution, Project → SharedWorkflow
- **ManyToOne:** Execution → Workflow, SharedWorkflow → Project

→ API Details: [[02_CORE_API]]
→ Configuration: [[05_CONFIG]]
