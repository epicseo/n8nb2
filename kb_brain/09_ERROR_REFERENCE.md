# 09_ERROR_REFERENCE.md
<!-- repo: n8n | branch: claude/github-to-kb-converter-01JAurUearApTk4RAu9NPFAg | commit: 3c0e809e | generated: 2025-12-02 -->
<!-- tags: errors, exceptions, error-handling, http-status, debugging -->

## Contents
- [Base Error Classes](#base-error-classes)
- [HTTP Response Errors](#http-response-errors)
- [Workflow Errors](#workflow-errors)
- [Execution Errors](#execution-errors)
- [Node Errors](#node-errors)
- [Credential Errors](#credential-errors)
- [Task Runner Errors](#task-runner-errors)
- [Data Table Errors](#data-table-errors)
- [Cache Errors](#cache-errors)
- [CLI Errors](#cli-errors)
- [Module Errors](#module-errors)

---

## Base Error Classes
<!-- chunk: 09-base-errors | keywords: base, error, abstract, root | source: packages/workflow/src/errors/base/ -->

### BaseError
**File:** `packages/workflow/src/errors/base/base.error.ts:11`
**Extends:** `Error`
**Description:** Abstract base class for all n8n errors

```typescript
abstract class BaseError extends Error {
  level: ErrorLevel;
  shouldReport: boolean;
  description?: string;
  tags?: Record<string, string>;
  extra?: JsonObject;
  packageName?: string;

  constructor(message: string, opts?: {
    level?: ErrorLevel;
    description?: string;
    shouldReport?: boolean;
    tags?: Record<string, string>;
    extra?: JsonObject;
  });
}
```

### OperationalError
**File:** `packages/workflow/src/errors/base/operational.error.ts:15`
**Extends:** `BaseError`
**Default Level:** `'warning'`
**Description:** Indicates transient issues (network failures, database timeouts, external API errors)

```typescript
class OperationalError extends BaseError {
  constructor(message: string, opts?: OperationalErrorOptions);
}
```
**When to use:** Network timeouts, temporary service unavailability, rate limiting

### UnexpectedError
**File:** `packages/workflow/src/errors/base/unexpected.error.ts:15`
**Extends:** `BaseError`
**Default Level:** `'error'`
**Description:** Indicates code logic mistakes, unhandled cases, assertion failures

```typescript
class UnexpectedError extends BaseError {
  constructor(message: string, opts?: UnexpectedErrorOptions);
}
```
**When to use:** Programming errors, invalid state, assertion failures

### UserError
**File:** `packages/workflow/src/errors/base/user.error.ts:16`
**Extends:** `BaseError`
**Default Level:** `'info'`
**Description:** Indicates user-triggered errors (invalid input, unauthorized access)

```typescript
class UserError extends BaseError {
  constructor(message: string, opts?: UserErrorOptions);
}
```
**When to use:** Invalid user input, permission denied, resource not found by user request

### ApplicationError (DEPRECATED)
**File:** `packages/@n8n/errors/src/application.error.ts:9`
**Extends:** `Error`
**Description:** Legacy base error class - use UserError, OperationalError, or UnexpectedError instead

---

## HTTP Response Errors
<!-- chunk: 09-http-errors | keywords: http, response, status, api | source: packages/cli/src/errors/response-errors/ -->

### ResponseError (Abstract)
**File:** `packages/cli/src/errors/response-errors/abstract/response.error.ts:6`
**Extends:** `BaseError`
**Description:** Base class for HTTP response errors with status codes

```typescript
abstract class ResponseError extends BaseError {
  httpStatusCode: number;
  errorCode?: number;
  hint?: string;
  meta?: Record<string, unknown>;

  constructor(
    message: string,
    httpStatusCode: number,
    errorCode?: number,
    hint?: string,
    cause?: unknown
  );
}
```

### BadRequestError
**File:** `packages/cli/src/errors/response-errors/bad-request.error.ts:3`
**HTTP Status:** `400`
**Extends:** `ResponseError`

```typescript
class BadRequestError extends ResponseError {
  constructor(message: string, errorCode?: number);
}
```
**When thrown:** Invalid request parameters, malformed JSON, validation failures

### UnauthenticatedError
**File:** `packages/cli/src/errors/response-errors/unauthenticated.error.ts:3`
**HTTP Status:** `401`
**Extends:** `ResponseError`

```typescript
class UnauthenticatedError extends ResponseError {
  constructor(message?: string, hint?: string);
}
```
**Default message:** "Unauthenticated"
**When thrown:** Missing authentication, expired session

### AuthError
**File:** `packages/cli/src/errors/response-errors/auth.error.ts:3`
**HTTP Status:** `401`
**Extends:** `ResponseError`

```typescript
class AuthError extends ResponseError {
  constructor(message: string, errorCode?: number);
}
```
**When thrown:** Invalid credentials, authentication failure

### ForbiddenError
**File:** `packages/cli/src/errors/response-errors/forbidden.error.ts:3`
**HTTP Status:** `403`
**Extends:** `ResponseError`

```typescript
class ForbiddenError extends ResponseError {
  constructor(message?: string, hint?: string);
}
```
**Default message:** "Forbidden"
**When thrown:** Access denied, insufficient permissions

### InvalidMfaCodeError
**File:** `packages/cli/src/errors/response-errors/invalid-mfa-code.error.ts:3`
**HTTP Status:** `403`
**Extends:** `ForbiddenError`

```typescript
class InvalidMfaCodeError extends ForbiddenError {
  constructor(hint?: string);
}
```
**Message:** "Invalid two-factor code."
**When thrown:** MFA code validation failure

### InvalidMfaRecoveryCodeError
**File:** `packages/cli/src/errors/response-errors/invalid-mfa-recovery-code-error.ts:3`
**HTTP Status:** `403`
**Extends:** `ForbiddenError`

```typescript
class InvalidMfaRecoveryCodeError extends ForbiddenError {
  constructor(hint?: string);
}
```
**Message:** "Invalid MFA recovery code"

### NotFoundError
**File:** `packages/cli/src/errors/response-errors/not-found.error.ts:3`
**HTTP Status:** `404`
**Extends:** `ResponseError`

```typescript
class NotFoundError extends ResponseError {
  constructor(message: string, hint?: string);

  // Type guard helper
  static isDefinedAndNotNull<T>(
    value: T | undefined | null,
    message: string,
    hint?: string
  ): asserts value is T;
}
```
**When thrown:** Resource not found, invalid IDs

### WebhookNotFoundError
**File:** `packages/cli/src/errors/response-errors/webhook-not-found.error.ts:35`
**HTTP Status:** `404`
**Extends:** `NotFoundError`

```typescript
class WebhookNotFoundError extends NotFoundError {
  constructor(
    options: { path: string; httpMethod?: string; webhookMethods?: string[] },
    additionalInfo?: { hint?: string }
  );

  static webhookNotFoundErrorMessage(options): string;
}
```
**When thrown:** Webhook path not registered, wrong HTTP method

### ConflictError
**File:** `packages/cli/src/errors/response-errors/conflict.error.ts:3`
**HTTP Status:** `409`
**Extends:** `ResponseError`

```typescript
class ConflictError extends ResponseError {
  constructor(message: string, hint?: string);
}
```
**When thrown:** Duplicate resources, naming conflicts

### ContentTooLargeError
**File:** `packages/cli/src/errors/response-errors/content-too-large.error.ts:3`
**HTTP Status:** `413`
**Extends:** `ResponseError`

```typescript
class ContentTooLargeError extends ResponseError {
  constructor(message: string, hint?: string);
}
```
**When thrown:** Payload exceeds size limits

### UnprocessableRequestError
**File:** `packages/cli/src/errors/response-errors/unprocessable.error.ts:3`
**HTTP Status:** `422`
**Extends:** `ResponseError`

```typescript
class UnprocessableRequestError extends ResponseError {
  constructor(message: string, hint?: string);
}
```
**When thrown:** Semantically invalid request

### TooManyRequestsError
**File:** `packages/cli/src/errors/response-errors/too-many-requests.error.ts:3`
**HTTP Status:** `429`
**Extends:** `ResponseError`

```typescript
class TooManyRequestsError extends ResponseError {
  constructor(message: string, hint?: string);
}
```
**When thrown:** Rate limit exceeded

### InternalServerError
**File:** `packages/cli/src/errors/response-errors/internal-server.error.ts:3`
**HTTP Status:** `500`
**Extends:** `ResponseError`

```typescript
class InternalServerError extends ResponseError {
  constructor(message?: string, cause?: unknown);
}
```
**When thrown:** Unhandled server errors

### NotImplementedError
**File:** `packages/cli/src/errors/response-errors/not-implemented.error.ts:3`
**HTTP Status:** `501`
**Extends:** `ResponseError`

```typescript
class NotImplementedError extends ResponseError {
  constructor(message: string, hint?: string);
}
```
**When thrown:** Feature not implemented

### ServiceUnavailableError
**File:** `packages/cli/src/errors/response-errors/service-unavailable.error.ts:3`
**HTTP Status:** `503`
**Extends:** `ResponseError`

```typescript
class ServiceUnavailableError extends ResponseError {
  constructor(message: string, errorCode?: number);
}
```
**When thrown:** Database unavailable, service down

### LicenseEulaRequiredError
**File:** `packages/cli/src/errors/response-errors/license-eula-required.error.ts:17`
**HTTP Status:** `400`
**Extends:** `ResponseError`

```typescript
class LicenseEulaRequiredError extends ResponseError {
  constructor(message: string, meta: { eulaUrl: string });
}
```
**When thrown:** EULA acceptance required

### TransferCredentialError
**File:** `packages/cli/src/errors/response-errors/transfer-credential.error.ts:3`
**HTTP Status:** `400`
**Extends:** `ResponseError`

```typescript
class TransferCredentialError extends ResponseError {
  constructor(message: string);
}
```
**When thrown:** Credential transfer failures

### TransferWorkflowError
**File:** `packages/cli/src/errors/response-errors/transfer-workflow.error.ts:3`
**HTTP Status:** `400`
**Extends:** `ResponseError`

```typescript
class TransferWorkflowError extends ResponseError {
  constructor(message: string);
}
```
**When thrown:** Workflow transfer failures

---

## Workflow Errors
<!-- chunk: 09-workflow-errors | keywords: workflow, operation, activation | source: packages/workflow/src/errors/ -->

### WorkflowOperationError
**File:** `packages/workflow/src/errors/workflow-operation.error.ts:7`
**Extends:** `ExecutionBaseError`
**Default Level:** `'warning'`

```typescript
class WorkflowOperationError extends ExecutionBaseError {
  node?: INode;
  timestamp: number;

  constructor(message: string, node?: INode, description?: string);
}
```
**When thrown:** Workflow-level operational failures

### WorkflowActivationError
**File:** `packages/workflow/src/errors/workflow-activation.error.ts:15`
**Extends:** `ExecutionBaseError`

```typescript
class WorkflowActivationError extends ExecutionBaseError {
  node?: INode;
  workflowId?: string;

  constructor(message: string, options?: {
    cause?: Error;
    node?: INode;
    level?: ErrorLevel;
    workflowId?: string;
  });
}
```
**Level detection:** Based on error message (ETIMEDOUT, ECONNREFUSED, EAUTH → warning)
**When thrown:** Workflow activation/trigger setup failures

### WorkflowDeactivationError
**File:** `packages/workflow/src/errors/workflow-deactivation.error.ts:3`
**Extends:** `WorkflowActivationError`

```typescript
class WorkflowDeactivationError extends WorkflowActivationError {}
```
**When thrown:** Workflow deactivation failures

### WorkflowConfigurationError
**File:** `packages/workflow/src/errors/workflow-configuration.error.ts:6`
**Extends:** `NodeOperationError`

```typescript
class WorkflowConfigurationError extends NodeOperationError {}
```
**When thrown:** Workflow configuration/validation issues

### SubworkflowOperationError
**File:** `packages/workflow/src/errors/subworkflow-operation.error.ts:3`
**Extends:** `WorkflowOperationError`

```typescript
class SubworkflowOperationError extends WorkflowOperationError {
  description: string;
  override cause: Error | undefined;

  constructor(message: string, description: string);
}
```
**When thrown:** Subworkflow execution failures

### CliWorkflowOperationError
**File:** `packages/workflow/src/errors/cli-subworkflow-operation.error.ts:3`
**Extends:** `SubworkflowOperationError`

```typescript
class CliWorkflowOperationError extends SubworkflowOperationError {}
```
**When thrown:** CLI subworkflow operations

### WebhookPathTakenError
**File:** `packages/workflow/src/errors/webhook-taken.error.ts:3`
**Extends:** `WorkflowActivationError`

```typescript
class WebhookPathTakenError extends WorkflowActivationError {
  constructor(nodeName: string, cause?: Error);
}
```
**When thrown:** Webhook path already registered by another workflow

### WorkflowHasIssuesError
**File:** `packages/core/src/errors/workflow-has-issues.error.ts:3`
**Extends:** `WorkflowOperationError`

```typescript
class WorkflowHasIssuesError extends WorkflowOperationError {}
```
**Message:** "The workflow has issues and cannot be executed for that reason. Please fix them first."

### WorkflowCrashedError
**File:** `packages/cli/src/errors/workflow-crashed.error.ts:3`
**Extends:** `WorkflowOperationError`

```typescript
class WorkflowCrashedError extends WorkflowOperationError {}
```
**Message:** "Workflow did not finish, possible out-of-memory issue"

### WorkflowMissingIdError
**File:** `packages/cli/src/errors/workflow-missing-id.error.ts:4`
**Extends:** `UnexpectedError`

```typescript
class WorkflowMissingIdError extends UnexpectedError {
  constructor(workflow: Workflow | IWorkflowBase);
}
```

### WorkflowHistoryVersionNotFoundError
**File:** `packages/cli/src/errors/workflow-history-version-not-found.error.ts:3`
**Extends:** `UnexpectedError`

```typescript
class WorkflowHistoryVersionNotFoundError extends UnexpectedError {}
```

---

## Execution Errors
<!-- chunk: 09-execution-errors | keywords: execution, run, cancelled, timeout | source: packages/workflow/src/errors/ -->

### ExecutionBaseError (Abstract)
**File:** `packages/workflow/src/errors/abstract/execution-base.error.ts:10`
**Extends:** `ApplicationError`

```typescript
abstract class ExecutionBaseError extends ApplicationError {
  description?: string;
  cause?: Error | JsonObject;
  errorResponse?: JsonObject;
  timestamp: number;
  context: IExecutionContext;
  lineNumber?: number;
  functionality?: FunctionalityType;

  constructor(message: string, options?: ExecutionBaseErrorOptions);
  toJSON(): JsonObject;
}
```

### ExecutionCancelledError (Abstract)
**File:** `packages/workflow/src/errors/execution-cancelled.error.ts:3`
**Extends:** `ExecutionBaseError`
**Level:** `'warning'`

```typescript
abstract class ExecutionCancelledError extends ExecutionBaseError {
  constructor(executionId: string);
}
```
**Message:** "The execution was cancelled"

### ManualExecutionCancelledError
**File:** `packages/workflow/src/errors/execution-cancelled.error.ts:13`
**Extends:** `ExecutionCancelledError`

```typescript
class ManualExecutionCancelledError extends ExecutionCancelledError {}
```
**Message:** "The execution was cancelled manually"

### TimeoutExecutionCancelledError
**File:** `packages/workflow/src/errors/execution-cancelled.error.ts:20`
**Extends:** `ExecutionCancelledError`

```typescript
class TimeoutExecutionCancelledError extends ExecutionCancelledError {}
```
**Message:** "The execution was cancelled because it timed out"

### SystemShutdownExecutionCancelledError
**File:** `packages/workflow/src/errors/execution-cancelled.error.ts:27`
**Extends:** `ExecutionCancelledError`

```typescript
class SystemShutdownExecutionCancelledError extends ExecutionCancelledError {}
```
**Message:** "The execution was cancelled because the system is shutting down"

### ExecutionNotFoundError
**File:** `packages/cli/src/errors/execution-not-found-error.ts:3`
**Extends:** `UnexpectedError`

```typescript
class ExecutionNotFoundError extends UnexpectedError {
  constructor(executionId: string);
}
```

### QueuedExecutionRetryError
**File:** `packages/cli/src/errors/queued-execution-retry.error.ts:3`
**Extends:** `UnexpectedError`

```typescript
class QueuedExecutionRetryError extends UnexpectedError {}
```
**Message:** "Execution is queued to run (not yet started) so it cannot be retried"

### AbortedExecutionRetryError
**File:** `packages/cli/src/errors/aborted-execution-retry.error.ts:3`
**Extends:** `UnexpectedError`

```typescript
class AbortedExecutionRetryError extends UnexpectedError {}
```
**Message:** "The execution was aborted before starting, so it cannot be retried"

### MissingExecutionStopError
**File:** `packages/cli/src/errors/missing-execution-stop.error.ts:3`
**Extends:** `UserError`

```typescript
class MissingExecutionStopError extends UserError {
  constructor(executionId: string);
}
```

---

## Node Errors
<!-- chunk: 09-node-errors | keywords: node, operation, api, ssl | source: packages/workflow/src/errors/ -->

### NodeError (Abstract)
**File:** `packages/workflow/src/errors/abstract/node.error.ts:37`
**Extends:** `ExecutionBaseError`

```typescript
abstract class NodeError extends ExecutionBaseError {
  messages: string[];

  constructor(node: INode, error: Error | JsonObject);

  findProperty(obj: JsonObject, ...keys: string[]): string | null;
  addToMessages(value: string | undefined): void;
  setDescriptiveErrorMessage(rawErrorMessage: string, ...args: string[]): void;
}
```
**Contains:** `COMMON_ERRORS` mapping for system-level error codes

### NodeApiError
**File:** `packages/workflow/src/errors/node-api.error.ts:120`
**Extends:** `NodeError`

```typescript
class NodeApiError extends NodeError {
  httpCode?: string;

  constructor(node: INode, errorResponse: JsonObject, options?: NodeApiErrorOptions);

  setDescriptionFromXml(xml: string): void;
  setDefaultStatusCodeMessage(): void;
}
```
**Contains:** `STATUS_CODE_MESSAGES` mapping for HTTP status descriptions
**When thrown:** External API errors (404, 500, etc.)

### NodeOperationError
**File:** `packages/workflow/src/errors/node-operation.error.ts:9`
**Extends:** `NodeError`
**Default Level:** `'warning'`

```typescript
class NodeOperationError extends NodeError {
  type?: string;

  constructor(node: INode, error: Error | string | JsonObject, options?: NodeOperationErrorOptions);
}
```
**When thrown:** Node execution failures, invalid operations

### NodeSslError
**File:** `packages/workflow/src/errors/node-ssl.error.ts:3`
**Extends:** `ExecutionBaseError`

```typescript
class NodeSslError extends ExecutionBaseError {
  constructor(cause: Error);
}
```
**Message:** "SSL Issue: consider using the 'Ignore SSL issues' option"

### NodeCrashedError
**File:** `packages/cli/src/errors/node-crashed.error.ts:4`
**Extends:** `NodeOperationError`

```typescript
class NodeCrashedError extends NodeOperationError {
  constructor(node: INode);
}
```
**Message:** "Node crashed, possible out-of-memory issue"

### TriggerCloseError
**File:** `packages/workflow/src/errors/trigger-close.error.ts:8`
**Extends:** `ApplicationError`

```typescript
class TriggerCloseError extends ApplicationError {
  node: INode;

  constructor(node: INode, options: { cause?: Error; level?: ErrorLevel });
}
```
**When thrown:** Trigger cleanup/close failures

---

## Credential Errors
<!-- chunk: 09-credential-errors | keywords: credential, authentication, secret | source: packages/cli/src/errors/ -->

### CredentialNotFoundError
**File:** `packages/cli/src/errors/credential-not-found.error.ts:3`
**Extends:** `UserError`

```typescript
class CredentialNotFoundError extends UserError {
  constructor(credentialId: string, credentialType: string);
}
```

### CredentialsOverwritesAlreadySetError
**File:** `packages/cli/src/errors/credentials-overwrites-already-set.error.ts:3`
**Extends:** `UserError`

```typescript
class CredentialsOverwritesAlreadySetError extends UserError {}
```
**Message:** "Credentials overwrites may not be set more than once."

### UnrecognizedCredentialTypeError
**File:** `packages/core/src/errors/unrecognized-credential-type.error.ts:3`
**Extends:** `UserError`

```typescript
class UnrecognizedCredentialTypeError extends UserError {
  constructor(credentialType: string);
}
```

---

## Task Runner Errors
<!-- chunk: 09-task-runner-errors | keywords: task, runner, worker, timeout | source: packages/cli/src/task-runners/errors/ -->

### TaskRunnerDisconnectedError
**File:** `packages/cli/src/task-runners/errors/task-runner-disconnected-error.ts:4`
**Extends:** `UnexpectedError`

```typescript
class TaskRunnerDisconnectedError extends UnexpectedError {
  runnerId: string;
  description: string;

  constructor(runnerId: TaskRunner['id'], isCloudDeployment: boolean);
}
```

### TaskRunnerOomError
**File:** `packages/cli/src/task-runners/errors/task-runner-oom-error.ts:5`
**Extends:** `UserError`

```typescript
class TaskRunnerOomError extends UserError {
  runnerId: string;
  description: string;

  constructor(runnerId: TaskRunner['id'], isCloudDeployment: boolean);
}
```
**When thrown:** Task runner out of memory

### TaskRunnerFailedHeartbeatError
**File:** `packages/cli/src/task-runners/errors/task-runner-failed-heartbeat.error.ts:3`
**Extends:** `UserError`

```typescript
class TaskRunnerFailedHeartbeatError extends UserError {
  description: string;

  constructor(heartbeatInterval: number, isSelfHosted: boolean);
}
```

### TaskRunnerRestartLoopError
**File:** `packages/cli/src/task-runners/errors/task-runner-restart-loop-error.ts:3`
**Extends:** `UnexpectedError`

```typescript
class TaskRunnerRestartLoopError extends UnexpectedError {
  howManyTimes: number;
  timePeriodMs: number;

  constructor(howManyTimes: number, timePeriodMs: number);
}
```

### TaskRequestTimeoutError
**File:** `packages/cli/src/task-runners/errors/task-request-timeout.error.ts:3`
**Extends:** `OperationalError`

```typescript
class TaskRequestTimeoutError extends OperationalError {
  description: string;

  constructor(options: { timeout: number; isSelfHosted: boolean });
}
```

### TaskRunnerExecutionTimeoutError
**File:** `packages/cli/src/task-runners/task-broker/errors/task-runner-execution-timeout.error.ts:4`
**Extends:** `OperationalError`

```typescript
class TaskRunnerExecutionTimeoutError extends OperationalError {
  description: string;

  constructor(options: { taskTimeout: number; isSelfHosted: boolean; mode: string });
}
```

### TaskRunnerAcceptTimeoutError
**File:** `packages/cli/src/task-runners/task-broker/errors/task-runner-accept-timeout.error.ts:3`
**Extends:** `OperationalError`
**Level:** `'warning'`

```typescript
class TaskRunnerAcceptTimeoutError extends OperationalError {
  constructor(taskId: string, runnerId: string);
}
```

### TaskDeferredError
**File:** `packages/cli/src/task-runners/task-broker/errors/task-deferred.error.ts:3`
**Extends:** `UserError`

```typescript
class TaskDeferredError extends UserError {}
```
**Message:** "Task deferred until runner is ready"

### TaskRejectError
**File:** `packages/cli/src/task-runners/task-broker/errors/task-reject.error.ts:3`
**Extends:** `UserError`

```typescript
class TaskRejectError extends UserError {
  reason: string;

  constructor(reason: string);
}
```

### MissingAuthTokenError
**File:** `packages/cli/src/task-runners/errors/missing-auth-token.error.ts:1`
**Extends:** `Error`

```typescript
class MissingAuthTokenError extends Error {}
```
**When thrown:** Missing auth token in external runner mode

### MissingRequirementsError
**File:** `packages/cli/src/task-runners/errors/missing-requirements.error.ts:10`
**Extends:** `UserError`

```typescript
class MissingRequirementsError extends UserError {
  constructor(reasonId: 'python' | 'venv');
}
```
**When thrown:** Python or venv not available for task runner

---

## Data Table Errors
<!-- chunk: 09-data-table-errors | keywords: data, table, column, row | source: packages/cli/src/modules/data-table/errors/ -->

### DataTableNotFoundError
**File:** `packages/cli/src/modules/data-table/errors/data-table-not-found.error.ts:3`
**Extends:** `UserError`
**Level:** `'warning'`

```typescript
class DataTableNotFoundError extends UserError {
  constructor(dataTableId: string);
}
```

### DataTableColumnNotFoundError
**File:** `packages/cli/src/modules/data-table/errors/data-table-column-not-found.error.ts:3`
**Extends:** `UserError`
**Level:** `'warning'`

```typescript
class DataTableColumnNotFoundError extends UserError {
  constructor(dataTableId: string, columnId: string);
}
```

### DataTableNameConflictError
**File:** `packages/cli/src/modules/data-table/errors/data-table-name-conflict.error.ts:3`
**Extends:** `UserError`
**Level:** `'warning'`

```typescript
class DataTableNameConflictError extends UserError {
  constructor(name: string);
}
```

### DataTableColumnNameConflictError
**File:** `packages/cli/src/modules/data-table/errors/data-table-column-name-conflict.error.ts:3`
**Extends:** `UserError`
**Level:** `'warning'`

```typescript
class DataTableColumnNameConflictError extends UserError {
  constructor(columnName: string, dataTableName: string);
}
```

### DataTableSystemColumnNameConflictError
**File:** `packages/cli/src/modules/data-table/errors/data-table-system-column-name-conflict.error.ts:3`
**Extends:** `UserError`
**Level:** `'warning'`

```typescript
class DataTableSystemColumnNameConflictError extends UserError {
  constructor(columnName: string, type?: string);
}
```
**When thrown:** Reserved column name used

### DataTableValidationError
**File:** `packages/cli/src/modules/data-table/errors/data-table-validation.error.ts:3`
**Extends:** `UserError`
**Level:** `'warning'`

```typescript
class DataTableValidationError extends UserError {
  constructor(msg: string);
}
```

### FileUploadError
**File:** `packages/cli/src/modules/data-table/errors/data-table-file-upload.error.ts:3`
**Extends:** `UserError`
**Level:** `'warning'`

```typescript
class FileUploadError extends UserError {
  constructor(msg: string);
}
```

---

## Cache Errors
<!-- chunk: 09-cache-errors | keywords: cache, memory, refresh | source: packages/cli/src/errors/cache-errors/ -->

### UncacheableValueError
**File:** `packages/cli/src/errors/cache-errors/uncacheable-value.error.ts:3`
**Extends:** `UnexpectedError`

```typescript
class UncacheableValueError extends UnexpectedError {
  constructor(key: string);
}
```
**Hint:** "Does the value contain circular references?"

### MalformedRefreshValueError
**File:** `packages/cli/src/errors/cache-errors/malformed-refresh-value.error.ts:3`
**Extends:** `UnexpectedError`

```typescript
class MalformedRefreshValueError extends UnexpectedError {}
```
**Message:** "Refresh value must have the same number of values as keys"

---

## CLI Errors
<!-- chunk: 09-cli-errors | keywords: cli, command, configuration | source: packages/cli/src/errors/ -->

### InvalidConcurrencyLimitError
**File:** `packages/cli/src/errors/invalid-concurrency-limit.error.ts:3`
**Extends:** `UserError`

```typescript
class InvalidConcurrencyLimitError extends UserError {
  constructor(value: number);
}
```

### InvalidRoleError
**File:** `packages/cli/src/errors/invalid-role.error.ts:3`
**Extends:** `UnexpectedError`

```typescript
class InvalidRoleError extends UnexpectedError {}
```

### UnknownExecutionModeError
**File:** `packages/cli/src/errors/unknown-execution-mode.error.ts:3`
**Extends:** `UnexpectedError`

```typescript
class UnknownExecutionModeError extends UnexpectedError {
  constructor(mode: string);
}
```

### FeatureNotLicensedError
**File:** `packages/cli/src/errors/feature-not-licensed.error.ts:4`
**Extends:** `UserError`
**Level:** `'warning'`

```typescript
class FeatureNotLicensedError extends UserError {
  constructor(feature: LICENSE_FEATURES);
}
```

### VariableCountLimitReachedError
**File:** `packages/cli/src/errors/variable-count-limit-reached.error.ts:3`
**Extends:** `UserError`

```typescript
class VariableCountLimitReachedError extends UserError {}
```

### VariableValidationError
**File:** `packages/cli/src/errors/variable-validation.error.ts:3`
**Extends:** `UnexpectedError`

```typescript
class VariableValidationError extends UnexpectedError {}
```

### SharedWorkflowNotFoundError
**File:** `packages/cli/src/errors/shared-workflow-not-found.error.ts:3`
**Extends:** `UserError`

```typescript
class SharedWorkflowNotFoundError extends UserError {}
```

### DeduplicationError
**File:** `packages/cli/src/errors/deduplication.error.ts:3`
**Extends:** `UnexpectedError`

```typescript
class DeduplicationError extends UnexpectedError {
  constructor(message: string);
}
```

### MaxStalledCountError
**File:** `packages/cli/src/errors/max-stalled-count.error.ts:6`
**Extends:** `OperationalError`
**Level:** `'warning'`

```typescript
class MaxStalledCountError extends OperationalError {
  constructor(cause: Error);
}
```

### NonJsonBodyError
**File:** `packages/cli/src/errors/non-json-body.error.ts:3`
**Extends:** `UserError`

```typescript
class NonJsonBodyError extends UserError {}
```
**Message:** "Body must be valid JSON. Please make sure `content-type` is `application/json`."

### SubworkflowPolicyDenialError
**File:** `packages/cli/src/errors/subworkflow-policy-denial.error.ts:28`
**Extends:** `WorkflowOperationError`

```typescript
class SubworkflowPolicyDenialError extends WorkflowOperationError {
  static readonly SUBWORKFLOW_DENIAL_BASE_DESCRIPTION: string;

  constructor(options: {
    subworkflowId: string;
    subworkflowProject: string;
    instanceUrl: string;
    hasReadAccess: boolean;
    ownerName: string;
    node: INode;
  });
}
```

### FolderNotFoundError
**File:** `packages/cli/src/errors/folder-not-found.error.ts:3`
**Extends:** `OperationalError`
**Level:** `'warning'`

```typescript
class FolderNotFoundError extends OperationalError {
  constructor(folderId: string);
}
```

### RedactableError
**File:** `packages/cli/src/errors/redactable.error.ts:3`
**Extends:** `UnexpectedError`

```typescript
class RedactableError extends UnexpectedError {
  constructor(fieldName: string, args: string);
}
```

### PostgresLiveRowsRetrievalError
**File:** `packages/cli/src/errors/postgres-live-rows-retrieval.error.ts:3`
**Extends:** `UnexpectedError`

```typescript
class PostgresLiveRowsRetrievalError extends UnexpectedError {
  constructor(rows: unknown);
}
```

---

## Module Errors
<!-- chunk: 09-module-errors | keywords: module, dependency, import | source: packages/@n8n/backend-common/src/modules/errors/ -->

### MissingModuleError
**File:** `packages/@n8n/backend-common/src/modules/errors/missing-module.error.ts:3`
**Extends:** `UserError`

```typescript
class MissingModuleError extends UserError {
  constructor(moduleName: string, errorMsg: string);
}
```

### ModuleConfusionError
**File:** `packages/@n8n/backend-common/src/modules/errors/module-confusion.error.ts:3`
**Extends:** `UserError`

```typescript
class ModuleConfusionError extends UserError {
  constructor(moduleNames: string[]);
}
```

### UnknownModuleError
**File:** `packages/@n8n/backend-common/src/modules/errors/unknown-module.error.ts:3`
**Extends:** `UnexpectedError`
**Level:** `'fatal'`

```typescript
class UnknownModuleError extends UnexpectedError {
  constructor(moduleName: string);
}
```

### DisallowedModuleError
**File:** `packages/@n8n/task-runner/src/js-task-runner/errors/disallowed-module.error.ts:3`
**Extends:** `UserError`

```typescript
class DisallowedModuleError extends UserError {
  constructor(moduleName: string);
}
```

---

## Expression Errors
<!-- chunk: 09-expression-errors | keywords: expression, evaluation, template | source: packages/workflow/src/errors/ -->

### ExpressionError
**File:** `packages/workflow/src/errors/expression.error.ts:31`
**Extends:** `ExecutionBaseError`
**Default Level:** `'warning'`

```typescript
class ExpressionError extends ExecutionBaseError {
  causeDetailed?: string;
  descriptionTemplate?: string;
  descriptionKey?: string;
  itemIndex?: number;
  messageTemplate?: string;
  nodeCause?: string;
  parameter?: string;
  runIndex?: number;
  type?: ExpressionErrorType;
  functionality?: FunctionalityType;

  constructor(message: string, options?: ExpressionErrorOptions);
}
```

### ExpressionExtensionError
**File:** `packages/workflow/src/errors/expression-extension.error.ts:3`
**Extends:** `ExpressionError`

```typescript
class ExpressionExtensionError extends ExpressionError {}
```

---

## File System Errors
<!-- chunk: 09-filesystem-errors | keywords: file, filesystem, binary | source: packages/core/src/errors/ -->

### FileSystemError (Abstract)
**File:** `packages/core/src/errors/abstract/filesystem.error.ts:3`
**Extends:** `ApplicationError`

```typescript
abstract class FileSystemError extends ApplicationError {
  constructor(message: string, filePath: string);
}
```

### FileNotFoundError
**File:** `packages/core/src/errors/file-not-found.error.ts:3`
**Extends:** `FileSystemError`

```typescript
class FileNotFoundError extends FileSystemError {
  constructor(filePath: string);
}
```

### FileTooLargeError
**File:** `packages/core/src/errors/file-too-large.error.ts:3`
**Extends:** `UserError`

```typescript
class FileTooLargeError extends UserError {
  constructor(options: {
    fileSizeMb: number;
    maxFileSizeMb: number;
    fileId: string;
    fileName?: string;
  });
}
```

### DisallowedFilepathError
**File:** `packages/core/src/errors/disallowed-filepath.error.ts:3`
**Extends:** `FileSystemError`

```typescript
class DisallowedFilepathError extends FileSystemError {
  constructor(filePath: string);
}
```

### BinaryDataFileNotFoundError
**File:** `packages/core/src/errors/binary-data-file-not-found.error.ts:3`
**Extends:** `UnexpectedError`

```typescript
class BinaryDataFileNotFoundError extends UnexpectedError {
  constructor(fileId: string);
}
```

---

## Database Errors
<!-- chunk: 09-database-errors | keywords: database, connection, timeout | source: packages/workflow/src/errors/ -->

### DbConnectionTimeoutError
**File:** `packages/workflow/src/errors/db-connection-timeout-error.ts:8`
**Extends:** `ApplicationError`

```typescript
class DbConnectionTimeoutError extends ApplicationError {
  constructor(opts: { configuredTimeoutInMs: number; cause: Error });
}
```

---

## MCP Errors
<!-- chunk: 09-mcp-errors | keywords: mcp, model, context, protocol | source: packages/cli/src/modules/mcp/ -->

### McpExecutionTimeoutError
**File:** `packages/cli/src/modules/mcp/mcp.errors.ts:9`
**Extends:** `UserError`

```typescript
class McpExecutionTimeoutError extends UserError {
  executionId: string | null;
  timeoutMs: number;

  constructor(executionId: string | null, timeoutMs: number);
}
```

### JWTVerificationError
**File:** `packages/cli/src/modules/mcp/mcp.errors.ts:23`
**Extends:** `AuthError`

```typescript
class JWTVerificationError extends AuthError {}
```
**Message:** "JWT Verification Failed"

### AccessTokenNotFoundError
**File:** `packages/cli/src/modules/mcp/mcp.errors.ts:30`
**Extends:** `AuthError`

```typescript
class AccessTokenNotFoundError extends AuthError {}
```
**Message:** "Access Token Not Found in Database"

---

## SAML/SSO Errors
<!-- chunk: 09-saml-errors | keywords: saml, sso, metadata, authentication | source: packages/cli/src/sso.ee/saml/errors/ -->

### InvalidSamlMetadataError
**File:** `packages/cli/src/sso.ee/saml/errors/invalid-saml-metadata.error.ts:3`
**Extends:** `UserError`

```typescript
class InvalidSamlMetadataError extends UserError {
  constructor(detail?: string);
}
```

### InvalidSamlMetadataUrlError
**File:** `packages/cli/src/sso.ee/saml/errors/invalid-saml-metadata-url.error.ts:3`
**Extends:** `UserError`

```typescript
class InvalidSamlMetadataUrlError extends UserError {
  constructor(url: string);
}
```

---

## IMAP Errors
<!-- chunk: 09-imap-errors | keywords: imap, email, connection | source: packages/@n8n/imap/src/ -->

### ImapError (Abstract)
**File:** `packages/@n8n/imap/src/errors.ts:1`
**Extends:** `Error`

```typescript
abstract class ImapError extends Error {}
```

### ConnectionTimeoutError
**File:** `packages/@n8n/imap/src/errors.ts:4`
**Extends:** `ImapError`

```typescript
class ConnectionTimeoutError extends ImapError {
  constructor(timeout?: number);
}
```

### ConnectionClosedError
**File:** `packages/@n8n/imap/src/errors.ts:17`
**Extends:** `ImapError`

```typescript
class ConnectionClosedError extends ImapError {}
```
**Message:** "Connection closed unexpectedly"

### ConnectionEndedError
**File:** `packages/@n8n/imap/src/errors.ts:23`
**Extends:** `ImapError`

```typescript
class ConnectionEndedError extends ImapError {}
```
**Message:** "Connection ended unexpectedly"

---

## Utility Functions
<!-- chunk: 09-utility-functions | keywords: utility, helper, ensure | source: packages/workflow/src/errors/ -->

### ensureError()
**File:** `packages/workflow/src/errors/ensure-error.ts:2`

```typescript
function ensureError(value: unknown): Error {
  if (value instanceof Error) return value;

  const error = new Error(
    typeof value === 'object' ? JSON.stringify(value) : String(value)
  );
  error.cause = value;
  return error;
}
```
**Purpose:** Ensures a value is an Error instance, wrapping non-Error values

---

## Error Hierarchy Summary
<!-- chunk: 09-hierarchy | keywords: hierarchy, inheritance, structure | source: analysis -->

```
Error (native)
├── BaseError (workflow)
│   ├── OperationalError
│   ├── UnexpectedError
│   └── UserError
├── ApplicationError (@n8n/errors) [DEPRECATED]
│   └── ExecutionBaseError
│       ├── NodeError (abstract)
│       │   ├── NodeApiError
│       │   └── NodeOperationError
│       │       ├── NodeCrashedError
│       │       └── WorkflowConfigurationError
│       ├── WorkflowOperationError
│       │   ├── SubworkflowOperationError
│       │   ├── WorkflowCrashedError
│       │   ├── WorkflowHasIssuesError
│       │   └── SubworkflowPolicyDenialError
│       ├── WorkflowActivationError
│       │   ├── WorkflowDeactivationError
│       │   └── WebhookPathTakenError
│       ├── ExpressionError
│       │   └── ExpressionExtensionError
│       ├── ExecutionCancelledError
│       │   ├── ManualExecutionCancelledError
│       │   ├── TimeoutExecutionCancelledError
│       │   └── SystemShutdownExecutionCancelledError
│       └── NodeSslError
└── ResponseError (cli)
    ├── BadRequestError (400)
    ├── AuthError (401)
    ├── UnauthenticatedError (401)
    ├── ForbiddenError (403)
    │   ├── InvalidMfaCodeError
    │   └── InvalidMfaRecoveryCodeError
    ├── NotFoundError (404)
    │   └── WebhookNotFoundError
    ├── ConflictError (409)
    ├── ContentTooLargeError (413)
    ├── UnprocessableRequestError (422)
    ├── TooManyRequestsError (429)
    ├── InternalServerError (500)
    ├── NotImplementedError (501)
    └── ServiceUnavailableError (503)
```

---

## Quick Reference
<!-- chunk: 09-quick-ref | keywords: quick, reference, lookup | source: analysis -->

### By HTTP Status Code

| Code | Error Class | Package |
|------|-------------|---------|
| 400 | BadRequestError | cli |
| 401 | AuthError, UnauthenticatedError | cli |
| 403 | ForbiddenError, InvalidMfaCodeError | cli |
| 404 | NotFoundError, WebhookNotFoundError | cli |
| 409 | ConflictError | cli |
| 413 | ContentTooLargeError | cli |
| 422 | UnprocessableRequestError | cli |
| 429 | TooManyRequestsError | cli |
| 500 | InternalServerError | cli |
| 501 | NotImplementedError | cli |
| 503 | ServiceUnavailableError | cli |

### By Category

| Category | Error Classes |
|----------|---------------|
| Authentication | AuthError, UnauthenticatedError, InvalidMfaCodeError, JWTVerificationError |
| Authorization | ForbiddenError, FeatureNotLicensedError, SubworkflowPolicyDenialError |
| Execution | ExecutionNotFoundError, ExecutionCancelledError, WorkflowCrashedError |
| Node | NodeApiError, NodeOperationError, NodeCrashedError, NodeSslError |
| Task Runner | TaskRunnerOomError, TaskRunnerDisconnectedError, TaskRequestTimeoutError |
| Data Table | DataTableNotFoundError, DataTableColumnNotFoundError, DataTableValidationError |
| Workflow | WorkflowOperationError, WorkflowActivationError, WorkflowHasIssuesError |
| Credentials | CredentialNotFoundError, UnrecognizedCredentialTypeError |
| File System | FileNotFoundError, FileTooLargeError, DisallowedFilepathError |

→ Debug Guide: [[06_DEBUG]]
→ API Reference: [[02_CORE_API]]
