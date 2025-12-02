# 11_WORKFLOW_TYPES.md
<!-- repo: n8n | branch: claude/github-to-kb-converter-01JAurUearApTk4RAu9NPFAg | commit: 3c0e809e | generated: 2025-12-02 -->
<!-- tags: workflow, types, interfaces, typescript, core -->

## Contents
- [Node Interfaces](#node-interfaces)
- [Workflow Interfaces](#workflow-interfaces)
- [Execution Interfaces](#execution-interfaces)
- [Credential Interfaces](#credential-interfaces)
- [Connection Types](#connection-types)
- [Property Types](#property-types)
- [Helper Functions](#helper-functions)

---

## Node Interfaces
<!-- chunk: 11-node-interfaces | keywords: node, inode, inodetype, description | source: packages/workflow/src/interfaces.ts -->

### INode
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Represents a node instance in a workflow

```typescript
interface INode {
  id: string;
  name: string;
  type: string;
  typeVersion: number;
  position: [number, number];
  disabled?: boolean;
  notes?: string;
  notesInFlow?: boolean;
  retryOnFail?: boolean;
  maxTries?: number;
  waitBetweenTries?: number;
  alwaysOutputData?: boolean;
  executeOnce?: boolean;
  continueOnFail?: boolean;
  onError?: OnError;
  parameters: INodeParameters;
  credentials?: INodeCredentials;
  webhookId?: string;
  extendsCredential?: string;
}
```

### INodeType
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Interface implemented by all node types

```typescript
interface INodeType {
  description: INodeTypeDescription;
  supplyData?: ISupplyDataFunctions;
  execute?: (this: IExecuteFunctions) => Promise<INodeExecutionData[][]>;
  poll?: (this: IPollFunctions) => Promise<INodeExecutionData[][] | null>;
  trigger?: (this: ITriggerFunctions) => Promise<ITriggerResponse | undefined>;
  webhook?: (this: IWebhookFunctions) => Promise<IWebhookResponseData>;
  methods?: {
    loadOptions?: Record<string, (this: ILoadOptionsFunctions) => Promise<INodePropertyOptions[]>>;
    listSearch?: Record<string, (this: ILoadOptionsFunctions, filter?: string, paginationToken?: string) => Promise<INodeListSearchResult>>;
    credentialTest?: Record<string, (this: ICredentialTestFunctions, credential: ICredentialsDecrypted) => Promise<INodeCredentialTestResult>>;
    resourceMapping?: Record<string, (this: ILoadOptionsFunctions) => Promise<ResourceMapperFields>>;
    localResourceMapping?: Record<string, (this: ILocalLoadOptionsFunctions) => Promise<ResourceMapperFields>>;
    actionHandler?: Record<string, (this: ILoadOptionsFunctions, payload: IDataObject | string) => Promise<INodePropertyOptions[] | NodeParameterValue>>;
  };
}
```

### INodeTypeDescription
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Complete metadata for a node type

```typescript
interface INodeTypeDescription extends INodeTypeBaseDescription {
  version: number | number[];
  defaults: INodeParameters;
  eventTriggerDescription?: string;
  activationMessage?: string;
  inputs: Array<NodeConnectionType | INodeInputConfiguration>;
  requiredInputs?: string | number[] | number;
  inputNames?: string[];
  outputs: Array<NodeConnectionType | INodeOutputConfiguration>;
  outputNames?: string[];
  properties: INodeProperties[];
  credentials?: INodeCredentialDescription[];
  maxNodes?: number;
  polling?: boolean;
  supportsCORS?: boolean;
  requestDefaults?: IHttpRequestOptions;
  requestOperations?: INodeRequestOperations;
  hooks?: INodeHooks;
  webhooks?: IWebhookDescription[];
  translation?: Record<string, object>;
  mockManualExecution?: boolean;
  extendsCredential?: string;
  hints?: NodeHint[];
  codex?: CodexData;
  usableAsTool?: boolean;
  __canvasActions?: Array<{ type: string; value: object }>;
}
```

### INodeTypeBaseDescription
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Base metadata for node types

```typescript
interface INodeTypeBaseDescription {
  displayName: string;
  name: string;
  icon?: ThemeIconName | ThemeIconData;
  iconColor?: ThemeIconColor;
  iconUrl?: string;
  badgeIconUrl?: string;
  group: string[];
  description: string;
  documentationUrl?: string;
  subtitle?: string;
  defaultVersion?: number;
  codex?: CodexData;
  parameterPane?: 'wide';
  hidden?: boolean;
  usableAsTool?: boolean;
  alias?: string[];
}
```

### INodeProperties
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Single parameter definition for a node

```typescript
interface INodeProperties {
  displayName: string;
  name: string;
  type: NodePropertyTypes;
  typeOptions?: INodePropertyTypeOptions;
  default: NodeParameterValueType;
  description?: string;
  hint?: string;
  displayOptions?: IDisplayOptions;
  options?: Array<INodePropertyOptions | INodeProperties | INodePropertyCollection>;
  placeholder?: string;
  isNodeSetting?: boolean;
  noDataExpression?: boolean;
  required?: boolean;
  routing?: INodePropertyRouting;
  credentialTypes?: string[];
  extractValue?: INodePropertyValueExtractor;
  modes?: INodePropertyMode[];
  requiresDataPath?: 'single' | 'multiple';
  doNotInherit?: boolean;
  validateType?: FieldType;
  ignoreValidationDuringExecution?: boolean;
}
```

### INodeExecutionData
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Execution data for a single item

```typescript
interface INodeExecutionData {
  json: IDataObject;
  binary?: IBinaryKeyData;
  error?: NodeError;
  pairedItem?: IPairedItemData | IPairedItemData[];
  metadata?: IItemMetadata;
  index?: number;
}
```

### INodeParameters
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Dictionary of parameter values

```typescript
type INodeParameters = Record<string, NodeParameterValueType>;
```

### INodeCredentials
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Credential references for a node

```typescript
interface INodeCredentials {
  [key: string]: INodeCredentialsDetails;
}
```

### INodeCredentialsDetails
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Details of a credential reference

```typescript
interface INodeCredentialsDetails {
  id: string | null;
  name: string;
}
```

### INodeIssues
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Validation issues for a node

```typescript
interface INodeIssues {
  execution?: boolean;
  credentials?: INodeIssueObjectProperty;
  parameters?: INodeIssueObjectProperty;
  typeUnknown?: boolean;
  input?: INodeIssueData;
}
```

---

## Workflow Interfaces
<!-- chunk: 11-workflow-interfaces | keywords: workflow, iworkflow, base, settings | source: packages/workflow/src/interfaces.ts -->

### IWorkflowBase
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Core workflow structure

```typescript
interface IWorkflowBase {
  id: string;
  name: string;
  active: boolean;
  createdAt: Date;
  updatedAt: Date;
  nodes: INode[];
  connections: IConnections;
  settings?: IWorkflowSettings;
  staticData?: IDataObject;
  meta?: WorkflowFEMeta;
  pinData?: IPinData;
  versionId?: string;
  tags?: ITag[] | string[];
}
```

### IWorkflowSettings
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Workflow-level configuration

```typescript
interface IWorkflowSettings {
  timezone?: string;
  errorWorkflow?: string;
  callerIds?: string;
  callerPolicy?: WorkflowCallerPolicy;
  saveDataErrorExecution?: SaveDataExecution;
  saveDataSuccessExecution?: SaveDataExecution;
  saveManualExecutions?: boolean;
  saveExecutionProgress?: boolean;
  executionTimeout?: number;
  executionOrder?: 'v0' | 'v1';
}
```

### IWorkflowMetadata
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Minimal workflow identification

```typescript
interface IWorkflowMetadata {
  id?: string;
  name?: string;
  active?: boolean;
}
```

### IWorkflowCredentials
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Encrypted credentials by type and ID

```typescript
interface IWorkflowCredentials {
  [credentialType: string]: {
    [id: string]: ICredentialsEncrypted;
  };
}
```

### IWorkflowExecuteAdditionalData
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Execution context and utilities for nodes

```typescript
interface IWorkflowExecuteAdditionalData {
  credentialsHelper: ICredentialsHelper;
  executeWorkflow: (
    workflowInfo: IExecuteWorkflowInfo,
    additionalData: IWorkflowExecuteAdditionalData,
    options: ExecuteWorkflowOptions
  ) => Promise<IWorkflowExecuteProcess>;
  executionId?: string;
  restartExecutionId?: string;
  hooks?: WorkflowHooks;
  httpResponse?: express.Response;
  httpRequest?: express.Request;
  restApiUrl: string;
  instanceBaseUrl: string;
  formWaitingBaseUrl: string;
  webhookBaseUrl: string;
  webhookWaitingBaseUrl: string;
  webhookTestBaseUrl: string;
  currentNodeParameters?: INodeParameters;
  executionTimeoutTimestamp?: number;
  userId?: string;
  variables: IDataObject;
  secretsHelpers: SecretsHelpersBase;
}
```

### IConnections
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Workflow connection map

```typescript
interface IConnections {
  [sourceNodeName: string]: INodeConnections;
}

interface INodeConnections {
  [connectionType: string]: IConnection[][];
}

interface IConnection {
  node: string;
  type: NodeConnectionType;
  index: number;
}
```

---

## Execution Interfaces
<!-- chunk: 11-execution-interfaces | keywords: execution, run, task, functions | source: packages/workflow/src/interfaces.ts -->

### IExecuteFunctions
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Complete execution context for node.execute()

```typescript
interface IExecuteFunctions extends FunctionsBase {
  continueOnFail(): boolean;
  evaluateExpression(expression: string, itemIndex: number): NodeParameterValueType;
  executeWorkflow(
    workflowInfo: IExecuteWorkflowInfo,
    inputData?: INodeExecutionData[],
    parentCallbackManager?: CallbackManager
  ): Promise<IWorkflowExecuteProcess>;
  getContext(type: ContextType): IContextObject;
  getInputData(inputIndex?: number, connectionType?: NodeConnectionType): INodeExecutionData[];
  getInputSourceData(inputIndex?: number, connectionType?: NodeConnectionType): ISourceData;
  getNodeParameter<T extends ValidParamType>(
    parameterName: string,
    itemIndex: number,
    fallbackValue?: T,
    options?: IGetNodeParameterOptions
  ): T;
  getWorkflowDataProxy(itemIndex: number): IWorkflowDataProxyData;
  putExecutionToWait(waitTill: Date): Promise<void>;
  sendMessageToUI(...args: unknown[]): void;
  sendResponse(response: IExecuteResponsePromiseData): void;
  startJob<T = unknown>(jobType: string, settings: unknown, itemIndex: number): Promise<T>;
  logNodeOutput(...args: unknown[]): void;
  addInputData(connectionType: NodeConnectionType, data: INodeExecutionData[][]): { index: number };
  addOutputData(connectionType: NodeConnectionType, currentNodeRunIndex: number, data: INodeExecutionData[][]): void;
  nodeHelpers: NodeHelperFunctions;
  getParentCallbackManager(): CallbackManager | undefined;
  manageLoops(): void;
}
```

### ITriggerFunctions
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Context for trigger nodes

```typescript
interface ITriggerFunctions extends FunctionsBase {
  emit(
    data: INodeExecutionData[][],
    responsePromise?: IDeferredPromise<IExecuteResponsePromiseData>,
    donePromise?: IDeferredPromise<IRun>
  ): void;
  emitError(error: Error, responsePromise?: IDeferredPromise<IExecuteResponsePromiseData>): void;
  getMode(): WorkflowExecuteMode;
  getActivationMode(): WorkflowActivateMode;
  getNodeParameter<T extends ValidParamType>(
    parameterName: string,
    fallbackValue?: T,
    options?: IGetNodeParameterOptions
  ): T;
  helpers: TriggerHelperFunctions;
}
```

### IPollFunctions
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Context for poll trigger nodes

```typescript
interface IPollFunctions extends FunctionsBase {
  __emit(data: INodeExecutionData[][]): void;
  __emitError(error: Error): void;
  getMode(): WorkflowExecuteMode;
  getActivationMode(): WorkflowActivateMode;
  getNodeParameter<T extends ValidParamType>(
    parameterName: string,
    fallbackValue?: T,
    options?: IGetNodeParameterOptions
  ): T;
  helpers: PollHelperFunctions;
}
```

### IWebhookFunctions
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Context for webhook handlers

```typescript
interface IWebhookFunctions extends FunctionsBase {
  getBodyData(): IDataObject;
  getHeaderData(): IncomingHttpHeaders;
  getMode(): WorkflowExecuteMode;
  getNodeParameter<T extends ValidParamType>(
    parameterName: string,
    fallbackValue?: T,
    options?: IGetNodeParameterOptions
  ): T;
  getNodeWebhookUrl(name: string): string | undefined;
  getParamsData(): object;
  getQueryData(): object;
  getRequestObject(): express.Request;
  getResponseObject(): express.Response;
  getWebhookName(): string;
  nodeHelpers: NodeHelperFunctions;
  helpers: WebhookHelperFunctions;
}
```

### ILoadOptionsFunctions
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Context for dynamic options loading

```typescript
interface ILoadOptionsFunctions extends FunctionsBase {
  getNodeParameter<T extends ValidParamType>(
    parameterName: string,
    fallbackValue?: T,
    options?: IGetNodeParameterOptions
  ): T;
  getCurrentNodeParameter(parameterName: string): NodeParameterValueType | undefined;
  getCurrentNodeParameters(): INodeParameters | undefined;
  helpers: RequestHelperFunctions;
}
```

### IRun
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Complete execution run with status and timing

```typescript
interface IRun {
  data: IRunExecutionData;
  finished?: boolean;
  mode: WorkflowExecuteMode;
  waitTill?: Date | null;
  startedAt: Date;
  stoppedAt?: Date;
  status: ExecutionStatus;
}
```

### IRunData
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Results by node name

```typescript
interface IRunData {
  [nodeName: string]: ITaskData[];
}
```

### ITaskData
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Single task execution data

```typescript
interface ITaskData {
  startTime: number;
  executionTime: number;
  executionStatus?: ExecutionStatus;
  data?: ITaskDataConnections;
  inputOverride?: ITaskDataConnections;
  error?: ExecutionBaseError;
  hints?: NodeExecutionHint[];
  metadata?: ITaskMetadata;
  source: ITaskDataConnectionsSource[];
}
```

### ITaskMetadata
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Task metadata

```typescript
interface ITaskMetadata {
  subRun?: IRunData;
  parentExecution?: IExecutionRelation;
  subExecutions?: IExecutionRelation[];
  timeSaved?: number;
}
```

### ExecutionSummary
**File:** `packages/workflow/src/interfaces.ts`
**Description:** High-level execution summary

```typescript
interface ExecutionSummary {
  id: string;
  finished?: boolean;
  mode: WorkflowExecuteMode;
  retryOf?: string | null;
  retrySuccessId?: string | null;
  waitTill?: Date | null;
  startedAt: Date;
  stoppedAt?: Date;
  workflowId: string;
  workflowName?: string;
  status: ExecutionStatus;
  lastNodeExecuted?: string;
  executionError?: ExecutionError;
  nodeExecutionStatus?: Record<string, ITaskData>;
}
```

---

## Credential Interfaces
<!-- chunk: 11-credential-interfaces | keywords: credential, authentication, icredential | source: packages/workflow/src/interfaces.ts -->

### ICredentialsDecrypted
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Decrypted credential with metadata

```typescript
interface ICredentialsDecrypted<T = ICredentialDataDecryptedObject> {
  id: string;
  name: string;
  type: string;
  data?: T;
}
```

### ICredentialsEncrypted
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Encrypted storage format

```typescript
interface ICredentialsEncrypted {
  id?: string;
  name: string;
  type: string;
  data?: string;
}
```

### ICredentialType
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Credential type definition

```typescript
interface ICredentialType {
  name: string;
  displayName: string;
  icon?: ThemeIconName | ThemeIconData;
  iconColor?: ThemeIconColor;
  iconUrl?: string;
  extends?: string[];
  properties: INodeProperties[];
  documentationUrl?: string;
  __overwrittenProperties?: string[];
  authenticate?: IAuthenticateGeneric;
  preAuthentication?: (this: IHttpRequestHelper, credentials: ICredentialDataDecryptedObject) => Promise<IDataObject>;
  test?: ICredentialTestRequest;
  genericAuth?: boolean;
  httpRequestNode?: INodeHttpRequestMethods;
}
```

### ICredentialTestRequest
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Credential test configuration

```typescript
interface ICredentialTestRequest {
  request: IHttpRequestOptions;
  rules?: ICredentialTestRequestRule[];
}
```

### INodeCredentialTestResult
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Result of credential test

```typescript
interface INodeCredentialTestResult {
  status: 'OK' | 'Error';
  message: string;
}
```

---

## Connection Types
<!-- chunk: 11-connection-types | keywords: connection, nodeconnectiontypes, enum | source: packages/workflow/src/constants.ts -->

### NodeConnectionTypes
**File:** `packages/workflow/src/constants.ts`
**Description:** Enum of all connection types

```typescript
enum NodeConnectionTypes {
  Main = 'main',
  AiAgent = 'ai_agent',
  AiChain = 'ai_chain',
  AiDocument = 'ai_document',
  AiEmbedding = 'ai_embedding',
  AiLanguageModel = 'ai_languageModel',
  AiMemory = 'ai_memory',
  AiOutputParser = 'ai_outputParser',
  AiRetriever = 'ai_retriever',
  AiReranker = 'ai_reranker',
  AiTextSplitter = 'ai_textSplitter',
  AiTool = 'ai_tool',
  AiVectorStore = 'ai_vectorStore',
}
```

### NodeConnectionType
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Type alias for connection types

```typescript
type NodeConnectionType = `${NodeConnectionTypes}`;
```

### INodeInputConfiguration
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Socket input definition

```typescript
interface INodeInputConfiguration {
  type: NodeConnectionType;
  displayName?: string;
  maxConnections?: number;
  required?: boolean;
  filter?: INodeInputFilter;
}
```

### INodeOutputConfiguration
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Socket output definition

```typescript
interface INodeOutputConfiguration {
  type: NodeConnectionType;
  displayName?: string;
  maxConnections?: number;
  category?: 'error';
}
```

---

## Property Types
<!-- chunk: 11-property-types | keywords: property, types, nodepropertytypes | source: packages/workflow/src/interfaces.ts -->

### NodePropertyTypes
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Available property input types

```typescript
type NodePropertyTypes =
  | 'boolean'
  | 'button'
  | 'collection'
  | 'color'
  | 'credentialsSelect'
  | 'dateTime'
  | 'fixedCollection'
  | 'filter'
  | 'hidden'
  | 'json'
  | 'multi-select'
  | 'multiOptions'
  | 'notice'
  | 'number'
  | 'options'
  | 'resourceLocator'
  | 'resourceMapper'
  | 'string'
  | 'workflowSelector'
  | 'curlImport'
  | 'assignment'
  | 'assignmentCollection'
  | 'credentials';
```

### FieldType
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Resource mapper field types

```typescript
type FieldType =
  | 'string'
  | 'number'
  | 'dateTime'
  | 'boolean'
  | 'time'
  | 'array'
  | 'object'
  | 'options'
  | 'url'
  | 'jwt';
```

### GenericValue
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Basic data types

```typescript
type GenericValue = string | object | number | boolean | undefined | null;
```

### IDataObject
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Dictionary type

```typescript
interface IDataObject {
  [key: string]: GenericValue | IDataObject | GenericValue[] | IDataObject[];
}
```

### NodeParameterValue
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Simple parameter values

```typescript
type NodeParameterValue = string | number | boolean | undefined | null;
```

### NodeParameterValueType
**File:** `packages/workflow/src/interfaces.ts`
**Description:** All parameter types including complex

```typescript
type NodeParameterValueType =
  | NodeParameterValue
  | INodeParameters
  | NodeParameterValue[]
  | INodeParameters[];
```

### ExpressionString
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Expression syntax marker

```typescript
type ExpressionString = `={{${string}}}`;
```

---

## Helper Functions
<!-- chunk: 11-helper-functions | keywords: helpers, functions, utilities | source: packages/workflow/src/interfaces.ts -->

### BaseHelperFunctions
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Base helper functions available to all contexts

```typescript
interface BaseHelperFunctions {
  createDeferredPromise<T = void>(): IDeferredPromise<T>;
  returnJsonArray(jsonData: IDataObject | IDataObject[]): INodeExecutionData[];
}
```

### RequestHelperFunctions
**File:** `packages/workflow/src/interfaces.ts`
**Description:** HTTP request helper functions

```typescript
interface RequestHelperFunctions {
  httpRequest(requestOptions: IHttpRequestOptions): Promise<any>;
  httpRequestWithAuthentication(
    this: IAllExecuteFunctions,
    credentialsType: string,
    requestOptions: IHttpRequestOptions,
    additionalCredentialOptions?: IAdditionalCredentialOptions
  ): Promise<any>;
  requestWithAuthenticationPaginated(
    this: IAllExecuteFunctions,
    requestOptions: IRequestOptionsSimplified,
    itemIndex: number,
    paginationOptions: PaginationOptions,
    credentialsType?: string,
    additionalCredentialOptions?: IAdditionalCredentialOptions
  ): Promise<any[]>;
  request(uriOrObject: string | IHttpRequestOptions, options?: IRequestOptions): Promise<any>;
  requestOAuth1(this: IAllExecuteFunctions, credentialsType: string, requestOptions: IHttpRequestOptions): Promise<any>;
  requestOAuth2(
    this: IAllExecuteFunctions,
    credentialsType: string,
    requestOptions: IHttpRequestOptions,
    oAuth2Options?: IOAuth2Options
  ): Promise<any>;
}
```

### BinaryHelperFunctions
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Binary data helper functions

```typescript
interface BinaryHelperFunctions {
  binaryToBuffer(body: Buffer | Readable): Promise<Buffer>;
  binaryToString(body: Buffer | Readable, encoding?: BufferEncoding): Promise<string>;
  getBinaryPath(binaryDataId: string): string;
  getBinaryStream(binaryDataId: string, chunkSize?: number): Promise<Readable>;
  getBinaryMetadata(binaryDataId: string): Promise<BinaryMetadata>;
  prepareBinaryData(binaryData: Buffer | Readable, filePath?: string, mimeType?: string): Promise<IBinaryData>;
  setBinaryDataBuffer(data: IBinaryData, binaryData: Buffer): Promise<IBinaryData>;
  copyBinaryFile(): Promise<never>;
  assertBinaryData(itemIndex: number, propertyName: string): IBinaryData;
  getBinaryDataBuffer(itemIndex: number, propertyName: string): Promise<Buffer>;
}
```

### FileSystemHelperFunctions
**File:** `packages/workflow/src/interfaces.ts`
**Description:** File access helper functions

```typescript
interface FileSystemHelperFunctions {
  createReadStream(path: string): Readable;
  getStoragePath(): string;
  writeContentToFile(path: string, content: string | Buffer | Readable, flag?: string): Promise<void>;
}
```

### DeduplicationHelperFunctions
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Item deduplication helper functions

```typescript
interface DeduplicationHelperFunctions {
  checkProcessedAndRecord(items: DeduplicationScope[], value: string | number, mode: 'create' | 'update'): Promise<DeduplicationCheckResult>;
  removeProcessed(items: DeduplicationScope[], value: string | number): Promise<void>;
  clearAllProcessedItems(items: DeduplicationScope[]): Promise<void>;
  getProcessedDataCount(scope: DeduplicationScope): Promise<number>;
}
```

### SSHTunnelFunctions
**File:** `packages/workflow/src/interfaces.ts`
**Description:** SSH tunnel helper functions

```typescript
interface SSHTunnelFunctions {
  getSSHClient(credentials: SSHCredentials): Promise<SSHClient>;
}
```

### SchedulingFunctions
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Cron scheduling functions

```typescript
interface SchedulingFunctions {
  registerCron(cronExpression: CronExpression, onTick: () => void): void;
}
```

---

## Execution Status Types
<!-- chunk: 11-status-types | keywords: status, execution, mode | source: packages/workflow/src/interfaces.ts -->

### ExecutionStatus
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Execution states

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

### WorkflowExecuteMode
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Execution contexts

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
  | 'evaluation';
```

### WorkflowActivateMode
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Activation contexts

```typescript
type WorkflowActivateMode =
  | 'activate'
  | 'update'
  | 'init'
  | 'leadershipChange';
```

### OnError
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Error handling modes

```typescript
type OnError = 'continueErrorOutput' | 'continueRegularOutput' | 'stopWorkflow';
```

### WebhookType
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Webhook types

```typescript
type WebhookType = 'default' | 'setup';
```

### WebhookResponseMode
**File:** `packages/workflow/src/interfaces.ts`
**Description:** Webhook response modes

```typescript
type WebhookResponseMode =
  | 'immediately'
  | 'lastNode'
  | 'onReceived'
  | 'responseNode'
  | 'waitForFormReload'
  | 'formPage';
```

---

## Data Table Types
<!-- chunk: 11-data-table-types | keywords: datatable, column, row | source: packages/workflow/src/data-table.types.ts -->

### DataTable
**File:** `packages/workflow/src/data-table.types.ts`
**Description:** Table structure

```typescript
interface DataTable {
  id: string;
  name: string;
  columns: DataTableColumn[];
  createdAt: Date;
  updatedAt: Date;
}
```

### DataTableColumn
**File:** `packages/workflow/src/data-table.types.ts`
**Description:** Column definition

```typescript
interface DataTableColumn {
  id: string;
  name: string;
  type: 'string' | 'number' | 'boolean' | 'date';
  index: number;
  createdAt: Date;
  updatedAt: Date;
}
```

### DataTableRow
**File:** `packages/workflow/src/data-table.types.ts`
**Description:** Row data

```typescript
interface DataTableRow {
  _id: string;
  _createdAt: Date;
  _updatedAt: Date;
  [columnName: string]: unknown;
}
```

### IDataTableProjectService
**File:** `packages/workflow/src/data-table.types.ts`
**Description:** Row/column CRUD operations

```typescript
interface IDataTableProjectService {
  addRows(tableId: string, data: object[], options: AddRowsOptions): Promise<AddRowsResult>;
  getRows(tableId: string, options?: GetRowsOptions): Promise<GetRowsResult>;
  updateRows(tableId: string, filter: DataTableFilter, data: object, options?: UpdateRowsOptions): Promise<UpdateRowsResult>;
  deleteRows(tableId: string, filter: DataTableFilter, options?: DeleteRowsOptions): Promise<DeleteRowsResult>;
  upsertRows(tableId: string, filter: DataTableFilter, data: object, options?: UpsertRowsOptions): Promise<UpsertRowsResult>;
  addColumn(tableId: string, column: CreateColumn): Promise<DataTableColumn>;
  deleteColumn(tableId: string, columnId: string): Promise<void>;
  updateColumn(tableId: string, columnId: string, column: UpdateColumn): Promise<DataTableColumn>;
  moveColumn(tableId: string, columnId: string, targetIndex: number): Promise<DataTableColumn[]>;
  lookupColumn(tableId: string, columnName: string): Promise<DataTableColumn | undefined>;
}
```

---

## Workflow Class
<!-- chunk: 11-workflow-class | keywords: workflow, class, execution | source: packages/workflow/src/workflow.ts -->

### Workflow
**File:** `packages/workflow/src/workflow.ts`
**Description:** Main workflow execution class

```typescript
class Workflow {
  id: string;
  name: string;
  nodes: INodes;
  connectionsBySourceNode: IConnection;
  connectionsByDestinationNode: IConnection;
  nodeTypes: INodeTypes;
  expression: Expression;
  active: boolean;
  settings: IWorkflowSettings;
  staticData: IDataObject;
  timezone: string;

  constructor(parameters: {
    id?: string;
    name?: string;
    nodes: INode[];
    connections: IConnections;
    active: boolean;
    nodeTypes: INodeTypes;
    staticData?: IDataObject;
    settings?: IWorkflowSettings;
  });

  // Node access
  getNode(nodeName: string): INode | null;
  getNodes(): INodes;
  getTriggerNodes(): INode[];
  getPollNodes(): INode[];
  getStartNode(destinationNode?: string): INode | undefined;
  getChildNodes(nodeName: string, type?: NodeConnectionType, depth?: number): string[];
  getParentNodes(nodeName: string, type?: NodeConnectionType, depth?: number): string[];

  // Static data
  getStaticData(type: string, node?: INode): IDataObject;
  setStaticData(type: string, node: INode, value: IDataObject): void;

  // Expression evaluation
  resolveSimpleParameterValue(
    parameterValue: NodeParameterValue,
    siblingParameters: INodeParameters,
    runExecutionData: IRunExecutionData | null,
    runIndex: number,
    itemIndex: number,
    activeNodeName: string,
    connectionInputData: INodeExecutionData[],
    mode: WorkflowExecuteMode,
    additionalKeys: IWorkflowDataProxyAdditionalKeys,
    executeData?: IExecuteData,
    defaultTimezone?: string,
    selfData?: IDataObject
  ): NodeParameterValue;

  // Workflow queries
  checkIfWorkflowCanBeActivated(ignoreVersionIssues?: boolean): boolean;
  checkReadyForExecution(data?: { executionMode?: WorkflowExecuteMode }): void;
  renameNodeInParameterValue(
    parameterValue: NodeParameterValueType,
    currentName: string,
    newName: string,
    type?: 'expression' | 'simpleParameter'
  ): NodeParameterValueType;
}
```

---

## Expression Class
<!-- chunk: 11-expression-class | keywords: expression, evaluation, template | source: packages/workflow/src/expression.ts -->

### Expression
**File:** `packages/workflow/src/expression.ts`
**Description:** Expression evaluation

```typescript
class Expression {
  workflow: Workflow;
  additionalData: IWorkflowExecuteAdditionalData;

  constructor(workflow: Workflow);

  resolveSimpleParameterValue(
    parameterValue: NodeParameterValue,
    siblingParameters: INodeParameters,
    runExecutionData: IRunExecutionData | null,
    runIndex: number,
    itemIndex: number,
    activeNodeName: string,
    connectionInputData: INodeExecutionData[],
    mode: WorkflowExecuteMode,
    additionalKeys: IWorkflowDataProxyAdditionalKeys,
    executeData?: IExecuteData,
    defaultTimezone?: string,
    selfData?: IDataObject
  ): NodeParameterValue;

  getParameterValue(
    parameterValue: NodeParameterValueType,
    runExecutionData: IRunExecutionData | null,
    runIndex: number,
    itemIndex: number,
    activeNodeName: string,
    connectionInputData: INodeExecutionData[],
    mode: WorkflowExecuteMode,
    additionalKeys: IWorkflowDataProxyAdditionalKeys,
    executeData?: IExecuteData,
    returnObjectAsString?: boolean,
    selfData?: IDataObject
  ): NodeParameterValueType;
}
```

---

## Type Statistics
<!-- chunk: 11-stats | keywords: statistics, summary, counts | source: analysis -->

| Category | Count |
|----------|-------|
| Node Interfaces | 15+ |
| Workflow Interfaces | 10+ |
| Execution Interfaces | 20+ |
| Credential Interfaces | 8+ |
| Connection Types | 13 |
| Property Types | 21+ |
| Helper Function Types | 10+ |
| Status/Mode Types | 8 |
| **Total Exported Types** | **100+** |

**Main Source File:**
`packages/workflow/src/interfaces.ts` - 3200+ lines, 315+ exported items

→ API Types: [[02_CORE_API]]
→ Error Reference: [[09_ERROR_REFERENCE]]
→ Patterns: [[04_PATTERNS]]
