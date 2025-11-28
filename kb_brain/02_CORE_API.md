# 02_CORE_API.md
<!-- repo: n8n | branch: claude/github-to-kb-converter-01JAurUearApTk4RAu9NPFAg | commit: 3c0e809e | generated: 2025-11-28 -->
<!-- tags: api, functions, controllers, services, core -->

## Contents
- [CLI Controllers](#cli-controllers)
- [CLI Services](#cli-services)
- [Core Package APIs](#core-package-apis)
- [Workflow Package APIs](#workflow-package-apis)

---

## CLI Controllers
<!-- chunk: 02-controllers | keywords: controller, rest, http, endpoint | source: packages/cli/src/controllers/ -->

### Authentication Controller
**File:** `packages/cli/src/controllers/auth.controller.ts`
**Base Path:** `/`

#### `POST /login`
<!-- chunk: 02-login | keywords: login, authenticate, auth -->
```typescript
login(req: AuthenticatedRequest, res: Response, payload: LoginRequestDto): Promise<PublicUser | undefined>
```
| Param | Type | Required | Purpose |
|-------|------|----------|---------|
| emailOrLdapLoginId | string | Yes | Email or LDAP login |
| password | string | Yes | User password |
| mfaCode | string | No | MFA code if enabled |
| mfaRecoveryCode | string | No | Recovery code alternative |

Returns: `PublicUser` — Authenticated user object
Raises: `AuthError` — Invalid credentials

#### `GET /login`
```typescript
currentUser(req: AuthenticatedRequest): Promise<PublicUser>
```
Returns: `PublicUser` — Currently authenticated user

#### `POST /logout`
```typescript
logout(req: AuthenticatedRequest, res: Response): Promise<{ loggedOut: boolean }>
```
Returns: `{ loggedOut: true }` — Logout confirmation

---

### Users Controller
**File:** `packages/cli/src/controllers/users.controller.ts`
**Base Path:** `/users`

#### `GET /`
<!-- chunk: 02-users-list | keywords: users, list, filter -->
```typescript
listUsers(req: AuthenticatedRequest, query: UsersListFilterDto): Promise<{ data: PublicUser[], count: number }>
```
| Param | Type | Required | Purpose |
|-------|------|----------|---------|
| limit | number | No | Results per page |
| cursor | string | No | Pagination cursor |
| includeRole | boolean | No | Include user roles |

Returns: Paginated list of users
Scope: `user:list`

#### `DELETE /:id`
```typescript
deleteUser(req: AuthenticatedRequest, params: { id: string }, query: { transferId?: string }): Promise<{ success: boolean }>
```
| Param | Type | Required | Purpose |
|-------|------|----------|---------|
| id | string | Yes | User ID to delete |
| transferId | string | No | Project to transfer resources |

Returns: `{ success: true }`
Scope: `user:delete`

#### `PATCH /:id/role`
```typescript
changeGlobalRole(req: AuthenticatedRequest, res: Response, payload: RoleChangeRequestDto, id: string): Promise<{ success: boolean }>
```
| Param | Type | Required | Purpose |
|-------|------|----------|---------|
| id | string | Yes | Target user ID |
| newRoleName | string | Yes | New role (global:admin, global:member) |

Scope: `user:changeRole`
License: `feat:advancedPermissions`

---

### Credentials Controller
**File:** `packages/cli/src/credentials/credentials.controller.ts`
**Base Path:** `/credentials`

#### `GET /`
<!-- chunk: 02-credentials-list | keywords: credentials, list -->
```typescript
getMany(req: AuthenticatedRequest, query: CredentialsGetManyRequestQuery): Promise<CredentialWithScopesAndData[]>
```
| Param | Type | Purpose |
|-------|------|---------|
| includeScopes | boolean | Include permission scopes |
| includeData | boolean | Include decrypted data |
| onlySharedWithMe | boolean | Filter to shared only |

#### `POST /`
```typescript
createCredential(req: AuthenticatedRequest, payload: CreateCredentialDto): Promise<CredentialsEntity>
```
| Param | Type | Required | Purpose |
|-------|------|----------|---------|
| name | string | Yes | Credential name |
| type | string | Yes | Credential type |
| data | object | Yes | Credential data |

#### `POST /test`
```typescript
testCredentials(req: AuthenticatedRequest, payload: ICredentialsDecrypted): Promise<INodeCredentialTestResult>
```
Returns: `{ status: 'OK' | 'Error', message?: string }`

#### `DELETE /:credentialId`
```typescript
deleteCredential(req: AuthenticatedRequest, params: { credentialId: string }): Promise<{ success: boolean }>
```
Scope: `credential:delete`

---

### Workflows Controller
**File:** `packages/cli/src/workflows/workflows.controller.ts`
**Base Path:** `/workflows`

#### `GET /`
<!-- chunk: 02-workflows-list | keywords: workflows, list -->
```typescript
getMany(req: ListQuery.Request, res: Response): Promise<PaginatedResponse<WorkflowEntity>>
```

#### `POST /`
```typescript
create(req: AuthenticatedRequest, payload: CreateWorkflowDto): Promise<WorkflowEntity>
```
| Param | Type | Required | Purpose |
|-------|------|----------|---------|
| name | string | Yes | Workflow name |
| nodes | INode[] | Yes | Workflow nodes |
| connections | IConnections | Yes | Node connections |
| settings | IWorkflowSettings | No | Workflow settings |

#### `PATCH /:workflowId`
```typescript
update(req: AuthenticatedRequest, params: { workflowId: string }, payload: UpdateWorkflowDto): Promise<WorkflowEntity>
```
Scope: `workflow:update`

#### `POST /:workflowId/run`
```typescript
runManually(req: AuthenticatedRequest, params: { workflowId: string }, payload: ManualRunPayload): Promise<IExecutionResponse>
```
Starts manual workflow execution.

---

### AI Controller
**File:** `packages/cli/src/controllers/ai.controller.ts`
**Base Path:** `/ai`

#### `POST /build`
<!-- chunk: 02-ai-build | keywords: ai, builder, workflow -->
```typescript
buildWorkflow(req: AuthenticatedRequest, res: Response, payload: AiBuilderChatRequestDto): Promise<StreamingResponse>
```
License: `feat:aiBuilder`
Returns: Streaming response with workflow suggestions

#### `POST /chat`
```typescript
chat(req: AuthenticatedRequest, res: Response, payload: AiChatRequestDto): Promise<StreamingResponse>
```
License: `feat:aiBuilder`

#### `POST /ask`
```typescript
ask(req: AuthenticatedRequest, payload: AiAskRequestDto): Promise<AiResponse>
```

---

### Node Types Controller
**File:** `packages/cli/src/controllers/node-types.controller.ts`
**Base Path:** `/node-types`

#### `POST /`
<!-- chunk: 02-node-types | keywords: nodes, types, description -->
```typescript
getNodeInfo(req: AuthenticatedRequest, payload: { nodeInfos: INodeTypeNameVersion[] }): Promise<INodeTypeDescription[]>
```
Returns node type descriptions with translations.

---

### Additional Controller Endpoints

| Controller | Base Path | Key Endpoints |
|------------|-----------|---------------|
| Active Workflows | `/active-workflows` | GET /, POST /activate, POST /deactivate |
| API Keys | `/api-keys` | GET /, POST /, DELETE /:id |
| Executions | `/executions` | GET /, GET /:id, DELETE /:id, POST /:id/retry |
| Projects | `/projects` | GET /, POST /, PATCH /:id, DELETE /:id |
| Tags | `/tags` | GET /, POST /, PATCH /:id, DELETE /:id |
| Me | `/me` | GET /, PATCH /, PATCH /password, POST /survey |
| MFA | `/mfa` | POST /enable, POST /disable, POST /verify |
| Owner | `/owner` | POST /setup, POST /dismiss-banner |
| Settings | `/settings` | GET / |

---

## CLI Services
<!-- chunk: 02-services | keywords: service, business-logic | source: packages/cli/src/services/ -->

### UserService
**File:** `packages/cli/src/services/user.service.ts`

#### `update`
```typescript
async update(userId: string, data: Partial<User>): Promise<void>
```
Updates user record with partial data.

#### `updateSettings`
```typescript
async updateSettings(userId: string, newSettings: Partial<IUserSettings>): Promise<void>
```
Updates user preferences (timezone, etc.).

#### `toPublic`
```typescript
async toPublic(user: User, options?: ToPublicOptions): Promise<PublicUser>
```
| Param | Type | Purpose |
|-------|------|---------|
| withInviteUrl | boolean | Generate invite URL |
| inviterId | string | Inviter for invite URL |
| withScopes | boolean | Include permission scopes |

#### `inviteUsers`
```typescript
async inviteUsers(owner: User, invitations: Invitation[]): Promise<void>
```
Sends invitation emails to new users.

---

### CredentialsService
**File:** `packages/cli/src/credentials/credentials.service.ts`

#### `getMany`
```typescript
async getMany(user: User, options: GetManyOptions): Promise<ICredentialDataDecryptedObject[]>
```
| Option | Type | Purpose |
|--------|------|---------|
| listQueryOptions | ListQueryOptions | Pagination/filtering |
| includeScopes | boolean | Include permissions |
| includeData | boolean | Include decrypted data |

#### `getOne`
```typescript
async getOne(user: User, credentialId: string, includeData?: boolean): Promise<ICredentialDataDecryptedObject>
```

#### `test`
```typescript
async test(userId: string, credentials: ICredentialsDecrypted): Promise<INodeCredentialTestResult>
```

#### `decrypt`
```typescript
decrypt(credential: Credential, fullData?: boolean): ICredentialDataDecryptedObject
```

---

### WorkflowService
**File:** `packages/cli/src/workflows/workflow.service.ts`

#### Key Methods
| Method | Signature | Purpose |
|--------|-----------|---------|
| `get` | `(id: string, relations?: string[]): Promise<WorkflowEntity>` | Get workflow by ID |
| `create` | `(workflow: WorkflowEntity, user: User): Promise<WorkflowEntity>` | Create new workflow |
| `update` | `(id: string, workflow: Partial<WorkflowEntity>): Promise<WorkflowEntity>` | Update workflow |
| `delete` | `(id: string): Promise<void>` | Delete workflow |
| `activate` | `(id: string): Promise<void>` | Activate workflow |
| `deactivate` | `(id: string): Promise<void>` | Deactivate workflow |

---

### ExecutionService
**File:** `packages/cli/src/executions/execution.service.ts`

#### `getExecutions`
```typescript
async getExecutions(filter: IGetExecutionsQueryFilter): Promise<ExecutionSummaries>
```

#### `stopExecution`
```typescript
async stopExecution(executionId: string): Promise<StopResult>
```

---

## Core Package APIs
<!-- chunk: 02-core | keywords: core, execution, engine | source: packages/core/src/ -->

### WorkflowExecute
**File:** `packages/core/src/execution-engine/workflow-execute.ts`

#### `constructor`
```typescript
constructor(
  additionalData: IWorkflowExecuteAdditionalData,
  mode: WorkflowExecuteMode,
  runExecutionData?: IRunExecutionData
)
```

#### `run`
<!-- chunk: 02-workflow-run | keywords: execute, run, workflow -->
```typescript
run(
  workflow: Workflow,
  startNode?: INode,
  destinationNode?: IDestinationNode,
  pinData?: IPinData,
  triggerToStartFrom?: TriggerInfo
): PCancelable<IRun>
```
| Param | Type | Purpose |
|-------|------|---------|
| workflow | Workflow | Workflow to execute |
| startNode | INode | Optional start node |
| destinationNode | IDestinationNode | Target node for partial execution |
| pinData | IPinData | Pinned node data |

Returns: Cancellable promise resolving to `IRun`

#### `runPartialWorkflow2`
```typescript
runPartialWorkflow2(
  workflow: Workflow,
  runData: IRunData,
  pinData?: IPinData,
  dirtyNodeNames?: string[],
  destinationNode?: IDestinationNode,
  agentRequest?: AiAgentRequest
): PCancelable<IRun>
```
Execute partial workflow from specific nodes.

#### Properties
| Property | Type | Purpose |
|----------|------|---------|
| `status` | ExecutionStatus | Current execution status |
| `timedOut` | boolean | Whether execution timed out |

---

### ExecuteContext
**File:** `packages/core/src/execution-engine/node-execution-context/execute-context.ts`

#### `helpers` Object
<!-- chunk: 02-execute-helpers | keywords: helpers, node, context -->
```typescript
readonly helpers: {
  // Data helpers
  returnJsonArray(values: unknown[]): INodeExecutionData[];
  copyInputItems(inputs: INodeExecutionData[], props: string[]): INodeExecutionData[];
  normalizeItems(items: unknown[]): INodeExecutionData[];

  // Request helpers
  request(options: RequestOptions): Promise<any>;
  requestWithAuthentication(credType: string, options: RequestOptions): Promise<any>;

  // Binary helpers
  assertBinaryData(itemIndex: number, propertyName: string): void;
  getBinaryDataBuffer(itemIndex: number, propertyName: string): Promise<Buffer>;
  copyBinaryFile(filePath: string, fileName: string, mimeType?: string): Promise<string>;

  // Scheduling helpers
  registerCron(cronDef: ICronDefinition, onTick: () => void): void;

  // Promise helpers
  createDeferredPromise(): DeferredPromise<unknown>;
}
```

#### `getNodeParameter`
```typescript
getNodeParameter(
  parameterName: string,
  itemIndex: number,
  fallbackValue?: unknown,
  options?: IGetNodeParameterOptions
): NodeParameterValue
```

#### `getCredentials`
```typescript
async getCredentials<T extends object>(type: string): Promise<T>
```

#### `getInputData`
```typescript
getInputData(inputIndex?: number, connectionType?: string): INodeExecutionData[]
```

---

### TriggerContext
**File:** `packages/core/src/execution-engine/node-execution-context/trigger-context.ts`

#### `emit`
```typescript
readonly emit: (data: INodeExecutionData[][]) => void
```
Emit execution data from trigger.

#### `emitError`
```typescript
readonly emitError: (error: Error) => void
```

#### `getActivationMode`
```typescript
getActivationMode(): WorkflowActivateMode
```
Returns: `'init'` | `'create'` | `'update'` | `'activate'` | `'manual'`

---

### PollContext
**File:** `packages/core/src/execution-engine/node-execution-context/poll-context.ts`

Same interface as TriggerContext with `__emit` and `__emitError` methods.

---

## Workflow Package APIs
<!-- chunk: 02-workflow | keywords: workflow, class, types | source: packages/workflow/src/ -->

### Workflow Class
**File:** `packages/workflow/src/workflow.ts`

#### Constructor
```typescript
constructor(parameters: WorkflowParameters)
```

#### Node Methods
| Method | Signature | Purpose |
|--------|-----------|---------|
| `setNodes` | `(nodes: INode[]): void` | Set workflow nodes |
| `getNode` | `(nodeName: string): INode \| null` | Get node by name |
| `getNodes` | `(nodeNames: string[]): INode[]` | Get multiple nodes |
| `getTriggerNodes` | `(): INode[]` | Get all trigger nodes |
| `getPollNodes` | `(): INode[]` | Get all poll nodes |

#### Connection Methods
| Method | Signature | Purpose |
|--------|-----------|---------|
| `setConnections` | `(connections: IConnections): void` | Set node connections |

#### Data Methods
| Method | Signature | Purpose |
|--------|-----------|---------|
| `getStaticData` | `(type: 'global' \| 'node', node?: INode): IDataObject` | Get persisted data |
| `setPinData` | `(pinData: IPinData \| undefined): void` | Set pinned data |
| `getPinDataOfNode` | `(nodeName: string): INodeExecutionData[] \| undefined` | Get node's pinned data |

---

### NodeHelpers
**File:** `packages/workflow/src/node-helpers.ts`

#### Key Functions
<!-- chunk: 02-node-helpers | keywords: helpers, node, utilities -->

| Function | Signature | Purpose |
|----------|-----------|---------|
| `displayParameter` | `(parameter: INodeProperties, nodeValues: INodeParameters): boolean` | Check if parameter should display |
| `getNodeParameters` | `(properties: INodeProperties[], values: INodeParameters, returnDefaults: boolean): INodeParameters \| null` | Get resolved parameters |
| `getNodeParametersIssues` | `(nodeType: INodeTypeDescription, nodeValues: INodeParameters): INodeIssues` | Validate node parameters |
| `isTriggerNode` | `(nodeTypeData: INodeTypeDescription): boolean` | Check if node is trigger |
| `isSubNodeType` | `(typeDescription: INodeTypeDescription): boolean` | Check if sub-node type |
| `getNodeWebhookPath` | `(node: INode, mode: string): string` | Get webhook path |
| `getNodeWebhookUrl` | `(baseUrl: string, node: INode, mode: string): string` | Get full webhook URL |

---

### Expression Class
**File:** `packages/workflow/src/expression.ts`

#### `resolveSimpleParameterValue`
```typescript
resolveSimpleParameterValue(
  parameterValue: NodeParameterValue,
  siblingParameters: INodeParameters,
  runExecutionData: IRunExecutionData | null,
  runIndex: number,
  itemIndex: number,
  activeNodeName: string,
  connectionInputData: INodeExecutionData[],
  mode: WorkflowExecuteMode,
  additionalKeys: IWorkflowDataProxyAdditionalKeys
): NodeParameterValue
```
Resolves parameter value, executing expressions like `{{ $json.field }}`.

#### Static Methods
| Method | Purpose |
|--------|---------|
| `initializeGlobalContext(data: IDataObject)` | Initialize expression global context |
| `resolveWithoutWorkflow(expression: string, data: IDataObject)` | Evaluate expression without workflow |

→ Data Models: [[03_DATA_MODELS]]
→ Examples: [[07_EXAMPLES]]
