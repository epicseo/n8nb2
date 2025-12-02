# 10_FRONTEND_STORES.md
<!-- repo: n8n | branch: claude/github-to-kb-converter-01JAurUearApTk4RAu9NPFAg | commit: 3c0e809e | generated: 2025-12-02 -->
<!-- tags: frontend, pinia, store, vue, state-management -->

## Contents
- [Store Overview](#store-overview)
- [Core Stores](#core-stores)
- [UI Stores](#ui-stores)
- [Workflow Stores](#workflow-stores)
- [Feature Stores](#feature-stores)
- [Store Patterns](#store-patterns)

---

## Store Overview
<!-- chunk: 10-overview | keywords: pinia, stores, state, vue | source: packages/frontend/editor-ui/src/app/stores/ -->

n8n frontend uses Pinia for state management. All stores are located in:
`packages/frontend/editor-ui/src/app/stores/`

### Store Registry
**File:** `packages/frontend/editor-ui/src/app/stores/index.ts`

| Store Name | Store ID | Purpose |
|------------|----------|---------|
| `useCanvasStore` | `canvas` | Canvas state and node positions |
| `useCloudPlanStore` | `STORES.CLOUD_PLAN` | Cloud subscription and usage |
| `useConsentStore` | `STORES.CONSENT` | User consent management |
| `useFocusPanelStore` | `STORES.FOCUS_PANEL` | Focus panel state |
| `useHistoryStore` | `STORES.HISTORY` | Undo/redo history |
| `useLogsStore` | `logs` | Execution logs panel |
| `useNodeTypesStore` | `STORES.NODE_TYPES` | Node type registry |
| `useNpsSurveyStore` | `npsSurvey` | NPS survey state |
| `usePostHog` | `posthog` | Feature flags and analytics |
| `usePushConnectionStore` | `STORES.PUSH` | WebSocket/SSE connection |
| `useRBACStore` | `STORES.RBAC` | Roles and permissions |
| `useRolesStore` | `roles` | Role definitions |
| `useSettingsStore` | `STORES.SETTINGS` | Application settings |
| `useUIStore` | `STORES.UI` | UI state and modals |
| `useVersionsStore` | `STORES.VERSIONS` | Version updates |
| `useWebhooksStore` | `STORES.WEBHOOKS` | Aggregator store |
| `useWorkflowStateStore` | `STORES.WORKFLOW_STATE` | Per-workflow state |
| `useWorkflowsEEStore` | `STORES.WORKFLOWS_EE` | Enterprise workflow features |
| `useWorkflowsStore` | `STORES.WORKFLOWS` | Main workflow store |

---

## Core Stores

### useWorkflowsStore
<!-- chunk: 10-workflows-store | keywords: workflow, nodes, connections, execution | source: packages/frontend/editor-ui/src/app/stores/workflows.store.ts -->

**File:** `packages/frontend/editor-ui/src/app/stores/workflows.store.ts`
**Store ID:** `STORES.WORKFLOWS`
**Lines:** 2006

The main workflow store managing workflow data, nodes, connections, and executions.

#### State Properties

```typescript
// Current workflow
workflow: Ref<IWorkflowDb>
workflowObject: Ref<Workflow>
totalWorkflowCount: Ref<number>

// Workflow registry
workflowsById: Ref<Record<string, IWorkflowDb>>
activeWorkflows: Ref<string[]>

// Execution state
workflowExecutionData: Ref<IExecutionResponse | null>
lastSuccessfulExecution: Ref<IExecutionResponse | null>
currentWorkflowExecutions: Ref<ExecutionSummary[]>
workflowExecutionStartedData: Ref<[string, IExecutionResponse] | null>
workflowExecutionResultDataLastUpdate: Ref<number>
workflowExecutionPairedItemMappings: Ref<Record<string, Set<string>>>
subWorkflowExecutionError: Ref<Error | null>
executionWaitingForWebhook: Ref<boolean>
activeExecutionId: Ref<string | null | undefined>
previousExecutionId: Ref<string | null | undefined>

// Node metadata
nodeMetadata: Ref<NodeMetadataMap>
usedCredentials: Ref<Record<string, IUsedCredential>>

// Chat state
chatMessages: Ref<string[]>
chatPartialExecutionDestinationNode: Ref<string | null>

// UI state
isInDebugMode: Ref<boolean>
selectedTriggerNodeName: Ref<string>
```

#### Computed Getters

```typescript
// Workflow properties
workflowName: string
workflowId: string
workflowVersionId: string
workflowSettings: IWorkflowSettings
workflowTags: string[]
allWorkflows: IWorkflowDb[]
isNewWorkflow: boolean
isWorkflowActive: boolean

// Node getters
allNodes: INodeUi[]
nodesByName: Record<string, INodeUi>
canvasNames: Set<string>
workflowTriggerNodes: INodeUi[]
currentWorkflowHasWebhookNode: boolean
nodesIssuesExist: boolean
workflowValidationIssues: INodeIssues | null

// Execution getters
getWorkflowRunData: IRunData | null
isWaitingExecution: boolean
isWorkflowRunning: boolean
executedNode: string | undefined
getAllLoadedFinishedExecutions: ExecutionSummary[]
getWorkflowExecution: IExecutionResponse | null
getPastChatMessages: string[]

// Connection getters
allConnections: IConnections
connectionsBySourceNode: (nodeName: string) => IConnection[]
connectionsByDestinationNode: (nodeName: string) => IConnection[]

// Trigger getters
selectableTriggerNodes: INodeUi[]
workflowExecutionTriggerNodeName: string | undefined

// Pinned data
pinnedWorkflowData: IPinData | undefined
```

#### Actions

```typescript
// Workflow CRUD
async createNewWorkflow(workflow: IWorkflowDataCreate): Promise<IWorkflowDb>
async fetchWorkflow(id: string): Promise<IWorkflowDb>
async updateWorkflow(id: string, data: IWorkflowDataUpdate): Promise<IWorkflowDb>
async deleteWorkflow(id: string): Promise<void>
async archiveWorkflow(id: string): Promise<void>
async unarchiveWorkflow(id: string): Promise<void>

// Workflow state
setWorkflow(workflow: IWorkflowDb): void
resetWorkflow(): void
addWorkflow(workflow: IWorkflowDb): void
setWorkflows(workflows: IWorkflowDb[]): void

// Node management
addNode(node: INodeUi): void
removeNode(node: INodeUi): void
removeNodeById(nodeId: string): void
setNodes(nodes: INodeUi[]): void
renameNodeSelectedAndExecution(oldName: string, newName: string): void
setNodePristine(nodeName: string, isPristine: boolean): void

// Connection management
addConnection(connection: IConnection): void
removeConnection(connection: IConnection): void
removeAllNodeConnection(node: INodeUi, type?: string): void
setConnections(connections: IConnections): void

// Execution
async runWorkflow(options: IStartRunData): Promise<IExecutionPushResponse>
async getExecution(id: string): Promise<IExecutionResponse>
async getPastExecutions(filter: IExecutionFlattedQueryFilter): Promise<ExecutionSummary[]>
async fetchExecutionDataById(id: string): Promise<IExecutionResponse | null>
setWorkflowExecutionRunData(runData: IRunData | null): void
updateNodeExecutionRunData(data: INodeExecutionData): void
clearNodeExecutionData(): void

// Pinned data
pinData(nodeName: string, data: INodeExecutionData[]): void
unpinData(nodeName: string): void
setWorkflowPinData(data: IPinData): void
pinDataByNodeName(nodeName: string): INodeExecutionData[] | undefined

// Credentials
setUsedCredentials(credentials: Record<string, IUsedCredential>): void
replaceInvalidWorkflowCredentials(credentials: ICredentialsResponse[]): void
assignCredentialToMatchingNodes(credential: ICredentialsResponse): void

// Metadata
setWorkflowMetadata(metadata: Partial<IWorkflowDb>): void
addToWorkflowMetadata(key: string, value: unknown): void
setWorkflowVersionId(versionId: string): void
setWorkflowScopes(scopes: string[]): void

// Tags
addWorkflowTagIds(tagIds: string[]): void
removeWorkflowTagId(tagId: string): void

// Chat
resetChatMessages(): void
appendChatMessage(message: string): void

// Utilities
getNodeByName(nodeName: string): INodeUi | null
getNodeById(nodeId: string): INodeUi | null
findNodeByPartialId(partialId: string): INodeUi | null
getPartialIdForNode(node: INodeUi): string
createWorkflowObject(nodes: INodeUi[], connections: IConnections): Workflow
```

#### Dependencies
- `useUIStore()`
- `useRootStore()`
- `useNDVStore()`
- `useNodeTypesStore()`
- `useSettingsStore()`
- `useSourceControlStore()`
- `useUsersStore()`

---

### useSettingsStore
<!-- chunk: 10-settings-store | keywords: settings, configuration, features | source: packages/frontend/editor-ui/src/app/stores/settings.store.ts -->

**File:** `packages/frontend/editor-ui/src/app/stores/settings.store.ts`
**Store ID:** `STORES.SETTINGS`

Application settings and feature flags.

#### State Properties

```typescript
initialized: Ref<boolean>
settings: Ref<FrontendSettings>
moduleSettings: Ref<FrontendModuleSettings>
userManagement: Ref<IUserManagementSettings>
templatesEndpointHealthy: Ref<boolean>
api: Ref<{ enabled: boolean; latestVersion: string; path: string; swaggerUi: boolean }>
mfa: Ref<{ enabled: boolean }>
folders: Ref<{ enabled: boolean }>
saveDataErrorExecution: Ref<WorkflowSettings.SaveDataExecution>
saveDataSuccessExecution: Ref<WorkflowSettings.SaveDataExecution>
saveManualExecutions: Ref<boolean>
saveDataProgressExecution: Ref<boolean>
isMFAEnforced: Ref<boolean>
```

#### Key Computed Getters

```typescript
// Deployment info
isDocker: boolean
databaseType: 'sqlite' | 'mariadb' | 'mysqldb' | 'postgresdb'
deploymentType: string
isCloudDeployment: boolean
nodeJsVersion: string

// Feature flags
isEnterpriseFeatureEnabled(feature: string): boolean
isAiAssistantEnabled: boolean
isAiBuilderEnabled: boolean
isAiAssistantOrBuilderEnabled: boolean
isCommunityNodesFeatureEnabled: boolean
isFoldersFeatureEnabled: boolean
isDataTableFeatureEnabled: boolean
isChatFeatureEnabled: boolean
areTagsEnabled: boolean
isTemplatesEnabled: boolean
isTelemetryEnabled: boolean
isMfaFeatureEnabled: boolean

// Configuration
showSetupPage: boolean
isSmtpSetup: boolean
pushBackend: 'sse' | 'websocket'
isQueueModeEnabled: boolean
isMultiMain: boolean
concurrency: number
isConcurrencyEnabled: boolean
isPublicApiEnabled: boolean
isPreviewMode: boolean
security: FrontendSettings['security']
```

#### Actions

```typescript
async getSettings(): Promise<void>
async initialize(): Promise<void>
setSettings(newSettings: FrontendSettings): void
setAllowedModules(allowedModules: AllowedModules): void
setSaveDataErrorExecution(newValue: WorkflowSettings.SaveDataExecution): void
setSaveDataSuccessExecution(newValue: WorkflowSettings.SaveDataExecution): void
setSaveManualExecutions(newValue: boolean): void
setSaveDataProgressExecution(newValue: boolean): void
stopShowingSetupPage(): void
disableTemplates(): void
async testTemplatesEndpoint(): Promise<void>
async getTimezones(): Promise<IDataObject>
async getModuleSettings(): Promise<void>
isModuleActive(moduleName: string): boolean
reset(): void
```

---

### useUIStore
<!-- chunk: 10-ui-store | keywords: ui, modals, theme, state | source: packages/frontend/editor-ui/src/app/stores/ui.store.ts -->

**File:** `packages/frontend/editor-ui/src/app/stores/ui.store.ts`
**Store ID:** `STORES.UI`

UI state management including modals, theme, and view state.

#### State Properties

```typescript
// Actions state
activeActions: Ref<string[]>
activeCredentialType: Ref<string | null>

// Theme
theme: Ref<ThemeOption>  // persisted to localStorage

// Modals
modalsById: Ref<Record<string, ModalState>>
modalStack: Ref<string[]>

// Sidebar
sidebarMenuCollapsedPreference: Ref<boolean>  // persisted
sidebarMenuCollapsed: Ref<boolean>

// View state
currentView: Ref<string>
stateIsDirty: Ref<boolean>
lastSelectedNode: Ref<string | null>
nodeViewOffsetPosition: Ref<[number, number]>
nodeViewInitialized: Ref<boolean>
addFirstStepOnLoad: Ref<boolean>

// Notifications
pendingNotificationsForViews: Ref<{ [key in VIEWS]?: NotificationOptions[] }>

// Execution
processingExecutionResults: Ref<boolean>
isBlankRedirect: Ref<boolean>

// Dynamic modules
moduleTabs: Ref<Record<string, Record<string, TabOptions[]>>>
registeredSettingsPages: Ref<Record<string, IMenuItem[]>>

// Canvas interaction
appGridDimensions: Ref<{ width: number; height: number }>
lastInteractedWithNodeConnection: Ref<Connection | undefined>
lastInteractedWithNodeHandle: Ref<string | null>
lastInteractedWithNodeId: Ref<string | undefined>
lastCancelledConnectionPosition: Ref<XYPosition | undefined>
```

#### Computed Getters

```typescript
appliedTheme: AppliedThemeOption
contextBasedTranslationKeys: object
lastInteractedWithNode: INode | null
isModalActiveById: { [key: string]: boolean }
activeModals: string[]
settingsSidebarItems: IMenuItem[]
isReadOnlyView: boolean
isActionActive: { [action: string]: boolean }
headerHeight: number
isAnyModalOpen: boolean
isProcessingExecutionResults: boolean
preferredSystemTheme: AppliedThemeOption
```

#### Actions

```typescript
// Theme
setTheme(newTheme: ThemeOption): void

// Modals
setMode(name: string, mode: string): void
setActiveId(name: string, activeId: string): void
setShowAuthSelector(name: string, showAuthSelector: boolean): void
setModalData(payload: { name: string; data: Record<string, unknown> }): void
openModal(name: string): void
openModalWithData(payload: { name: string; data: Record<string, unknown> }): void
closeModal(name: string): void

// Specific modal openers
openDeleteUserModal(id: string): void
openExistingCredential(id: string): void
openNewCredential(type: string, showAuthOptions?: boolean): void
openCommunityPackageUninstallConfirmModal(packageName: string): void
openCommunityPackageUpdateConfirmModal(packageName: string, source?: string): void
openDeleteFolderModal(id: string, eventBus: EventBus, content: object): void
openMoveToFolderModal(resourceType: string, resource: object, eventBus: EventBus): void

// Actions
addActiveAction(action: string): void
removeActiveAction(action: string): void

// Sidebar
toggleSidebarMenuCollapse(): void

// Notifications
setNotificationsForView(view: VIEWS, notifications: NotificationOptions[]): void

// Canvas interaction
resetLastInteractedWith(): void

// Module registration
registerCustomTabs(page: string, moduleName: string, tabs: TabOptions[]): void
registerSettingsPages(moduleName: string, items: IMenuItem[]): void

// Execution
setProcessingExecutionResults(value: boolean): void

// Modal registry
registerModal(modalKey: string, initialState?: Partial<ModalState>): void
unregisterModal(modalKey: string): void
initializeModalsFromRegistry(): void
cleanup(): void
```

---

### useNodeTypesStore
<!-- chunk: 10-nodetypes-store | keywords: nodes, types, registry, community | source: packages/frontend/editor-ui/src/app/stores/nodeTypes.store.ts -->

**File:** `packages/frontend/editor-ui/src/app/stores/nodeTypes.store.ts`
**Store ID:** `STORES.NODE_TYPES`

Node type registry and node information.

#### State Properties

```typescript
nodeTypes: Ref<NodeTypesByTypeNameAndVersion>
vettedCommunityNodeTypes: Ref<Map<string, CommunityNodeType>>
```

#### Computed Getters

```typescript
// Community nodes
communityNodeType(nodeTypeName: string): CommunityNodeType
officialCommunityNodeTypes: INodeTypeDescription[]
unofficialCommunityNodeTypes: INodeTypeDescription[]
communityNodesAndActions: any

// Node type access
allNodeTypes: INodeTypeDescription[]
allLatestNodeTypes: INodeTypeDescription[]
moduleEnabledNodeTypes: INodeTypeDescription[]
visibleNodeTypes: INodeTypeDescription[]
getNodeType(nodeTypeName: string, version?: number): INodeTypeDescription | null
getNodeVersions(nodeTypeName: string): number[]
getCredentialOnlyNodeType(nodeTypeName: string, version?: number): INodeTypeDescription | null

// Node type checks
isConfigNode(workflow: Workflow, node: INode, nodeTypeName: string): boolean
isTriggerNode(nodeTypeName: string): boolean
isToolNode(nodeTypeName: string): boolean
isCoreNodeType(nodeType: INodeTypeDescription): boolean
isConfigurableNode(workflow: Workflow, node: INode, nodeTypeName: string, version?: number): boolean
getIsNodeInstalled(nodeTypeName: string): boolean

// Node naming
nativelyNumberSuffixedDefaults: string[]

// Connection types
visibleNodeTypesByOutputConnectionTypeNames: { [key: string]: string[] }
visibleNodeTypesByInputConnectionTypeNames: { [key: string]: string[] }
```

#### Actions

```typescript
// Node type management
setNodeTypes(newNodeTypes: INodeTypeDescription[]): void
removeNodeTypes(nodeTypesToRemove: INodeTypeDescription[]): void
async getNodesInformation(nodeInfos: INodeTypeNameVersion[], replace?: boolean): Promise<INodeTypeDescription[]>
async getFullNodesProperties(nodesToBeFetched: string[], replaceNodeTypes?: boolean): Promise<void>
async getNodeTypes(): Promise<void>
async loadNodeTypesIfNotLoaded(): Promise<void>
async getNodeTranslationHeaders(): Promise<void>

// Dynamic node parameters
async getNodeParameterOptions(sendData: DynamicNodeParametersRequest): Promise<any>
async getResourceLocatorResults(sendData: ResourceLocatorRequest): Promise<any>
async getResourceMapperFields(sendData: ResourceMapperFieldsRequest): Promise<any>
async getLocalResourceMapperFields(sendData: ResourceMapperFieldsRequest): Promise<any>
async getNodeParameterActionResult(sendData: ActionResultRequest): Promise<any>

// Community nodes
async fetchCommunityNodePreviews(): Promise<void>
async getCommunityNodeAttributes(nodeName: string): Promise<any>
```

---

## UI Stores

### useRBACStore
<!-- chunk: 10-rbac-store | keywords: rbac, roles, permissions, scopes | source: packages/frontend/editor-ui/src/app/stores/rbac.store.ts -->

**File:** `packages/frontend/editor-ui/src/app/stores/rbac.store.ts`
**Store ID:** `STORES.RBAC`

Role-Based Access Control state.

#### State Properties

```typescript
globalRoles: Ref<Role[]>
rolesByProjectId: Ref<Record<string, string[]>>
globalScopes: Ref<Scope[]>
scopesByProjectId: Ref<Record<string, Scope[]>>
scopesByResourceId: Ref<Record<Resource, Record<string, Scope[]>>>
```

#### Actions

```typescript
addGlobalRole(role: Role): void
hasRole(role: Role): boolean
addGlobalScope(scope: Scope): void
setGlobalScopes(scopes: Scope[]): void
addProjectScope(scope: Scope, context: { projectId: string }): void
addResourceScope(scope: Scope, context: { resourceType: Resource; resourceId: string }): void
hasScope(
  scope: Scope,
  context?: { projectId?: string; resourceType?: Resource; resourceId?: string },
  options?: { mode?: 'oneOf' | 'allOf' }
): boolean
```

---

### useHistoryStore
<!-- chunk: 10-history-store | keywords: history, undo, redo, commands | source: packages/frontend/editor-ui/src/app/stores/history.store.ts -->

**File:** `packages/frontend/editor-ui/src/app/stores/history.store.ts`
**Store ID:** `STORES.HISTORY`

Undo/redo history management.

#### State Properties

```typescript
undoStack: Undoable[]
redoStack: Undoable[]
currentBulkAction: BulkCommand | null
bulkInProgress: boolean
```

#### Actions

```typescript
// Undo operations
popUndoableToUndo(): Undoable | undefined
pushCommandToUndo(undoable: Command, clearRedo: boolean): void
pushBulkCommandToUndo(undoable: BulkCommand, clearRedo: boolean): void
checkUndoStackLimit(): void
clearUndoStack(): void

// Redo operations
popUndoableToRedo(): Undoable | undefined
pushUndoableToRedo(undoable: Undoable): void
checkRedoStackLimit(): void
clearRedoStack(): void

// Bulk operations
startRecordingUndo(): void
stopRecordingUndo(): void

// Reset
reset(): void
```

---

### useLogsStore
<!-- chunk: 10-logs-store | keywords: logs, execution, panel, chat | source: packages/frontend/editor-ui/src/app/stores/logs.store.ts -->

**File:** `packages/frontend/editor-ui/src/app/stores/logs.store.ts`
**Store ID:** `logs`

Execution logs panel state.

#### State Properties

```typescript
isOpen: Ref<boolean>  // persisted to localStorage
preferPoppedOut: Ref<boolean>
height: Ref<number>
detailsState: Ref<LogDetailsPanelState>  // persisted
detailsStateSubNode: Ref<LogDetailsPanelState>  // persisted
isLogSelectionSyncedWithCanvas: Ref<boolean>  // persisted
isSubNodeSelected: Ref<boolean>
chatSessionId: Ref<string>
chatSessionMessages: Ref<ChatMessage[]>
```

#### Computed Getters

```typescript
state: LogsPanelState  // computed from isOpen and preferPoppedOut
```

#### Actions

```typescript
// Panel state
setHeight(value: number): void
toggleOpen(value?: boolean): void
setPreferPoppedOut(value: boolean): void
setSubNodeSelected(value: boolean): void
toggleInputOpen(open?: boolean): void
toggleOutputOpen(open?: boolean): void
toggleLogSelectionSync(value?: boolean): void

// Chat session
getNewSessionId(): string
resetChatSessionId(): void
resetMessages(): void
addChatMessage(message: ChatMessage): void
```

---

### useCanvasStore
<!-- chunk: 10-canvas-store | keywords: canvas, nodes, position, loading | source: packages/frontend/editor-ui/src/app/stores/canvas.store.ts -->

**File:** `packages/frontend/editor-ui/src/app/stores/canvas.store.ts`
**Store ID:** `canvas`

Canvas state and node positioning.

#### State Properties

```typescript
newNodeInsertPosition: Ref<XYPosition | null>
hasRangeSelection: Ref<boolean>
```

#### Computed Getters

```typescript
nodes: INodeUi[]  // from workflowStore.allNodes
aiNodes: INodeUi[]  // filters langchain/evaluation nodes
isLoading: boolean  // from loadingService
```

#### Actions

```typescript
setHasRangeSelection(value: boolean): void
startLoading(): void
setLoadingText(text: string): void
stopLoading(): void
```

---

### useFocusPanelStore
<!-- chunk: 10-focus-panel-store | keywords: focus, panel, parameters, width | source: packages/frontend/editor-ui/src/app/stores/focusPanel.store.ts -->

**File:** `packages/frontend/editor-ui/src/app/stores/focusPanel.store.ts`
**Store ID:** `STORES.FOCUS_PANEL`

Focus panel state for node parameter editing.

#### State Properties

```typescript
lastFocusTimestamp: Ref<number>
```

#### Computed Getters

```typescript
focusPanelData: FocusPanelDataByWid  // from localStorage
currentFocusPanelData: FocusPanelData
focusPanelActive: boolean
focusPanelWidth: number
focusedNodeParameters: RichFocusedNodeParameter[]
resolvedParameter: RichFocusedNodeParameter | undefined
focusedNodeParametersInTelemetryFormat: Array<{ parameterPath: string; nodeType: string; nodeId: string }>
```

#### Actions

```typescript
onNewWorkflowSave(wid: string): void
openWithFocusedNodeParameter(nodeParameter: NodeParameter): void
closeFocusPanel(): void
unsetParameters(): void
toggleFocusPanel(): void
updateWidth(width: number): void
```

---

## Feature Stores

### useCloudPlanStore
<!-- chunk: 10-cloud-store | keywords: cloud, plan, subscription, usage | source: packages/frontend/editor-ui/src/app/stores/cloudPlan.store.ts -->

**File:** `packages/frontend/editor-ui/src/app/stores/cloudPlan.store.ts`
**Store ID:** `STORES.CLOUD_PLAN`

Cloud subscription and usage tracking.

#### State Properties

```typescript
state: CloudPlanState  // { initialized, data, usage, loadingPlan }
currentUserCloudInfo: Ref<Cloud.UserAccount | null>
isDynamicTrialBannerDismissed: Ref<boolean>  // from localStorage
```

#### Computed Getters

```typescript
userIsTrialing: boolean
currentPlanData: Cloud | null
currentUsageData: any
selectedApps: any[]
codingSkill: number
dynamicTrialBannerText: string
shouldShowDynamicTrialBanner: boolean
trialExpired: boolean
allExecutionsUsed: boolean
hasCloudPlan: boolean
usageLeft: { workflowsLeft: number; executionsLeft: number }
trialDaysLeft: number
```

#### Actions

```typescript
reset(): void
async getUserCloudAccount(): Promise<void>
async getAutoLoginCode(): Promise<{ code: string }>
async getOwnerCurrentPlan(): Promise<Cloud>
async getInstanceCurrentUsage(): Promise<any>
startPollingInstanceUsageData(): void
async checkForCloudPlanData(): Promise<void>
async fetchUserCloudAccount(): Promise<void>
async initialize(): Promise<void>
async generateCloudDashboardAutoLoginLink(data: object): Promise<string>
dismissDynamicTrialBanner(): void
```

---

### usePushConnectionStore
<!-- chunk: 10-push-store | keywords: push, websocket, sse, realtime | source: packages/frontend/editor-ui/src/app/stores/pushConnection.store.ts -->

**File:** `packages/frontend/editor-ui/src/app/stores/pushConnection.store.ts`
**Store ID:** `STORES.PUSH`

WebSocket/SSE connection management.

#### State Properties

```typescript
outgoingQueue: Ref<unknown[]>
isConnectionRequested: Ref<boolean>
onMessageReceivedHandlers: Ref<OnPushMessageHandler[]>
```

#### Computed Getters

```typescript
useWebSockets: boolean
isConnected: boolean
```

#### Actions

```typescript
getConnectionUrl(): string
async onMessage(data: PushMessage): Promise<void>
serializeAndSend(message: unknown): void
pushConnect(): void
pushDisconnect(): void
clearQueue(): void
addEventListener(handler: OnPushMessageHandler): () => void
```

---

### usePostHog
<!-- chunk: 10-posthog-store | keywords: posthog, analytics, experiments, flags | source: packages/frontend/editor-ui/src/app/stores/posthog.store.ts -->

**File:** `packages/frontend/editor-ui/src/app/stores/posthog.store.ts`
**Store ID:** `posthog`

PostHog analytics and feature flags.

#### State Properties

```typescript
featureFlags: Ref<FeatureFlags | null>
trackedDemoExp: Ref<FeatureFlags>
overrides: Ref<Record<string, string | boolean>>
```

#### Actions

```typescript
reset(): void
getVariant(experiment: string): any
isVariantEnabled(experiment: string, variant: string): boolean
isFeatureEnabled(experiment: string): boolean
identify(): void
trackExperiment(featFlags: FeatureFlags, name: string): void
trackExperiments(featFlags: FeatureFlags): void
trackExperimentsDebounced(featFlags: FeatureFlags): void
init(evaluatedFeatureFlags?: FeatureFlags): void
setMetadata(metadata: object, target: string): void
capture(event: string, properties?: object): void
```

---

### useVersionsStore
<!-- chunk: 10-versions-store | keywords: versions, updates, whatsnew | source: packages/frontend/editor-ui/src/app/stores/versions.store.ts -->

**File:** `packages/frontend/editor-ui/src/app/stores/versions.store.ts`
**Store ID:** `STORES.VERSIONS`

Version update notifications.

#### State Properties

```typescript
versionNotificationSettings: Ref<IVersionNotificationSettings>
nextVersions: Ref<Version[]>
currentVersion: Ref<Version | undefined>
whatsNew: Ref<WhatsNewSection>
whatsNewCallout: Ref<NotificationHandle | undefined>
```

#### Computed Getters

```typescript
hasVersionUpdates: boolean
hasSignificantUpdates: boolean
latestVersion: Version
areNotificationsEnabled: boolean
infoUrl: string
readWhatsNewArticles: number[]
lastDismissedWhatsNewCallout: number[]
whatsNewArticles: WhatsNewArticle[]
```

#### Actions

```typescript
async fetchVersions(): Promise<void>
setVersions(params: { versions: Version[]; currentVersionName: string }): void
setWhatsNew(section: WhatsNewSection): void
setWhatsNewArticleRead(articleId: number): void
isWhatsNewArticleRead(articleId: number): boolean
closeWhatsNewCallout(): void
dismissWhatsNewCallout(): void
shouldShowWhatsNewCallout(): boolean
async fetchWhatsNew(): Promise<void>
initialize(settings: IVersionNotificationSettings): void
async checkForNewVersions(): Promise<void>
```

---

### useRolesStore
<!-- chunk: 10-roles-store | keywords: roles, permissions, custom | source: packages/frontend/editor-ui/src/app/stores/roles.store.ts -->

**File:** `packages/frontend/editor-ui/src/app/stores/roles.store.ts`
**Store ID:** `roles`

Role definitions and management.

#### State Properties

```typescript
roles: Ref<AllRolesMap>  // { global, project, credential, workflow }
projectRoleOrder: Ref<string[]>
```

#### Computed Getters

```typescript
projectRoleOrderMap: Map<string, number>
processedProjectRoles: Role[]  // filters out owner role
processedCredentialRoles: Role[]
processedWorkflowRoles: Role[]
```

#### Actions

```typescript
async fetchRoles(): Promise<void>
async createProjectRole(body: CreateRolePayload): Promise<Role>
async fetchRoleBySlug(payload: { slug: string; withUsageCount?: boolean }): Promise<Role>
async deleteProjectRole(slug: string): Promise<Role>
async updateProjectRole(slug: string, body: UpdateRolePayload): Promise<Role>
```

---

## Workflow Stores

### useWorkflowsEEStore
<!-- chunk: 10-workflows-ee-store | keywords: enterprise, sharing, workflow | source: packages/frontend/editor-ui/src/app/stores/workflows.ee.store.ts -->

**File:** `packages/frontend/editor-ui/src/app/stores/workflows.ee.store.ts`
**Store ID:** `STORES.WORKFLOWS_EE`

Enterprise workflow features.

#### Computed Getters

```typescript
getWorkflowOwnerName(workflowId: string, fallback?: string): string
```

#### Actions

```typescript
setWorkflowSharedWith(payload: { workflowId: string; sharedWithProjects: IProjectSharing[] }): void
async saveWorkflowSharedWith(payload: { sharedWithProjects: IProjectSharing[]; workflowId: string }): Promise<void>
```

---

### useWorkflowStateStore
<!-- chunk: 10-workflow-state-store | keywords: workflow, state, executing | source: packages/frontend/editor-ui/src/app/stores/workflowState.store.ts -->

**File:** `packages/frontend/editor-ui/src/app/stores/workflowState.store.ts`
**Store ID:** `STORES.WORKFLOW_STATE`

Per-workflow temporary state.

#### State/Computed

```typescript
executingNode: ReturnType<typeof useExecutingNode>
```

---

### useWebhooksStore
<!-- chunk: 10-webhooks-store | keywords: webhooks, aggregator, combined | source: packages/frontend/editor-ui/src/app/stores/webhooks.store.ts -->

**File:** `packages/frontend/editor-ui/src/app/stores/webhooks.store.ts`
**Store ID:** `STORES.WEBHOOKS`

Aggregator store combining multiple stores for webhook context.

Includes all state/getters/actions from:
- `useRootStore()`
- `useWorkflowsStore()`
- `useUIStore()`
- `useUsersStore()`
- `useNDVStore()`
- `useSettingsStore()`

---

## Store Patterns
<!-- chunk: 10-patterns | keywords: patterns, composition, best-practices | source: analysis -->

### Composition API Store Pattern

```typescript
// packages/frontend/editor-ui/src/app/stores/example.store.ts
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';
import { STORES } from '@n8n/stores';

export const useExampleStore = defineStore(STORES.EXAMPLE, () => {
  // State (refs)
  const items = ref<Item[]>([]);
  const loading = ref(false);
  const error = ref<string | null>(null);

  // Getters (computed)
  const itemCount = computed(() => items.value.length);

  const getItemById = computed(() => (id: string) => {
    return items.value.find(item => item.id === id);
  });

  // Actions (functions)
  async function fetchItems() {
    loading.value = true;
    error.value = null;
    try {
      items.value = await api.getItems();
    } catch (e) {
      error.value = e.message;
    } finally {
      loading.value = false;
    }
  }

  function addItem(item: Item) {
    items.value.push(item);
  }

  function reset() {
    items.value = [];
    loading.value = false;
    error.value = null;
  }

  // Return public API
  return {
    items,
    loading,
    error,
    itemCount,
    getItemById,
    fetchItems,
    addItem,
    reset,
  };
});
```

### localStorage Persistence Pattern

```typescript
import { useStorage } from '@vueuse/core';

export const useExampleStore = defineStore('example', () => {
  // Persisted state
  const preferences = useStorage('n8n-example-prefs', {
    theme: 'light',
    collapsed: false,
  });

  // Non-persisted state
  const tempData = ref<string | null>(null);

  return { preferences, tempData };
});
```

### Store Dependencies Pattern

```typescript
export const useChildStore = defineStore('child', () => {
  // Get parent store
  const parentStore = useParentStore();
  const settingsStore = useSettingsStore();

  // Use parent state
  const derivedValue = computed(() => {
    return parentStore.someValue && settingsStore.isFeatureEnabled;
  });

  // Call parent actions
  async function doSomething() {
    await parentStore.parentAction();
  }

  return { derivedValue, doSomething };
});
```

### Store Testing Pattern

```typescript
// packages/frontend/editor-ui/src/app/stores/__tests__/example.store.test.ts
import { createPinia, setActivePinia } from 'pinia';
import { useExampleStore } from '../example.store';

vi.mock('@/api', () => ({
  getItems: vi.fn(),
}));

describe('ExampleStore', () => {
  beforeEach(() => {
    setActivePinia(createPinia());
  });

  it('should initialize with empty state', () => {
    const store = useExampleStore();
    expect(store.items).toEqual([]);
    expect(store.loading).toBe(false);
  });

  it('should add item', () => {
    const store = useExampleStore();
    store.addItem({ id: '1', name: 'Test' });
    expect(store.itemCount).toBe(1);
  });
});
```

---

## Store Statistics
<!-- chunk: 10-stats | keywords: statistics, summary, counts | source: analysis -->

| Metric | Count |
|--------|-------|
| Total Stores | 19 |
| State Properties | 150+ |
| Computed Getters | 200+ |
| Actions | 400+ |
| Lines of Code | ~5000 |

### Most Used Store Dependencies

| Store | Used By |
|-------|---------|
| `useRootStore()` | 13 stores |
| `useSettingsStore()` | 10 stores |
| `useUIStore()` | 5 stores |
| `useWorkflowsStore()` | 6 stores |
| `useUsersStore()` | 4 stores |

### Largest Stores (by code)

| Store | Lines |
|-------|-------|
| `workflows.store.ts` | 2006 |
| `ui.store.ts` | ~750 |
| `nodeTypes.store.ts` | ~470 |
| `settings.store.ts` | ~410 |

→ Patterns: [[04_PATTERNS]]
→ Examples: [[07_EXAMPLES]]
→ Architecture: [[01_ARCHITECTURE]]
