# 20_ERROR_HANDLING.md
<!-- repo: n8n | version: 1.122.0 | generated: 2025-12-02 -->
<!-- tags: errors, handling, retry, continueOnFail, workflows -->

## Contents
- [Overview](#overview)
- [Node-Level Error Handling](#node-level-error-handling)
- [Error Trigger Workflows](#error-trigger-workflows)
- [Retry Logic](#retry-logic)
- [Stop and Error Node](#stop-and-error-node)
- [Patterns](#patterns)

---

## Overview
<!-- chunk: 20-overview | keywords: errors, handling, types -->

n8n provides multiple error handling mechanisms:

| Mechanism | Level | Purpose |
|-----------|-------|---------|
| `continueOnFail` | Node | Skip errors, continue workflow |
| `retryOnFail` | Node | Automatic retry with backoff |
| Error Trigger | Workflow | Catch failures in dedicated workflow |
| Stop and Error | Node | Explicitly throw errors |
| Try/Catch pattern | Workflow | If-based error branching |

---

## Node-Level Error Handling
<!-- chunk: 20-node-level | keywords: continueOnFail, retryOnFail, options -->

### continueOnFail

Allows workflow to continue even if node fails.

**Node Configuration:**
```json
{
  "name": "HTTP Request",
  "type": "n8n-nodes-base.httpRequest",
  "continueOnFail": true,
  "parameters": {
    "url": "https://api.example.com/endpoint"
  }
}
```

**In Code:**
```javascript
// Check if continueOnFail is enabled
if (this.continueOnFail()) {
  // Return input data unchanged on error
  return [this.getInputData()];
}
```

**Error Output:**
When `continueOnFail` is enabled, failed items include error info:
```json
{
  "json": {
    "error": {
      "message": "Request failed with status 404",
      "name": "Error"
    }
  }
}
```

### retryOnFail

Automatic retry with configurable attempts and delay.

**Node Configuration:**
```json
{
  "name": "HTTP Request",
  "type": "n8n-nodes-base.httpRequest",
  "retryOnFail": true,
  "maxTries": 3,
  "waitBetweenTries": 1000,
  "parameters": {
    "url": "https://api.example.com/endpoint"
  }
}
```

**Parameters:**
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `retryOnFail` | boolean | false | Enable retry |
| `maxTries` | number | 3 | Maximum retry attempts |
| `waitBetweenTries` | number | 1000 | Delay in ms between retries |

### alwaysOutputData

Ensure node always outputs data, even on error.

```json
{
  "name": "Process",
  "type": "n8n-nodes-base.code",
  "alwaysOutputData": true,
  "parameters": {
    "jsCode": "// code that might fail"
  }
}
```

---

## Error Trigger Workflows
<!-- chunk: 20-error-trigger | keywords: errorTrigger, workflows, catch -->

### Error Trigger Node

Creates a workflow that runs when another workflow fails.

**Error Trigger Configuration:**
```json
{
  "type": "n8n-nodes-base.errorTrigger",
  "typeVersion": 1,
  "position": [250, 300],
  "parameters": {}
}
```

**Error Data Structure:**
```json
{
  "execution": {
    "id": "12345",
    "url": "https://n8n.example.com/execution/12345",
    "retryOf": "",
    "error": {
      "message": "Error message here",
      "stack": "Error stack trace..."
    },
    "lastNodeExecuted": "HTTP Request",
    "mode": "trigger"
  },
  "workflow": {
    "id": "1",
    "name": "My Workflow"
  }
}
```

### Setting Error Workflow

In workflow settings, specify error handler:
```json
{
  "settings": {
    "errorWorkflow": "error-handler-workflow-id"
  }
}
```

### Error Handler Workflow Example

```json
{
  "name": "Error Handler",
  "nodes": [
    {
      "name": "Error Trigger",
      "type": "n8n-nodes-base.errorTrigger",
      "position": [250, 300]
    },
    {
      "name": "Slack Alert",
      "type": "n8n-nodes-base.slack",
      "position": [500, 300],
      "parameters": {
        "resource": "message",
        "operation": "post",
        "channel": { "value": "C-ALERTS" },
        "text": "={{ 'Workflow Failed: ' + $json.workflow.name + '\\nError: ' + $json.execution.error.message }}"
      }
    },
    {
      "name": "Log to DB",
      "type": "n8n-nodes-base.postgres",
      "position": [750, 300],
      "parameters": {
        "operation": "insert",
        "table": "workflow_errors",
        "columns": "workflow_id,workflow_name,error_message,execution_id,timestamp",
        "values": "={{ $json.workflow.id }}, {{ $json.workflow.name }}, {{ $json.execution.error.message }}, {{ $json.execution.id }}, {{ $now.toISO() }}"
      }
    }
  ],
  "connections": {
    "Error Trigger": {
      "main": [[
        { "node": "Slack Alert", "type": "main", "index": 0 },
        { "node": "Log to DB", "type": "main", "index": 0 }
      ]]
    }
  }
}
```

---

## Retry Logic
<!-- chunk: 20-retry | keywords: retry, backoff, exponential -->

### Exponential Backoff Pattern

```typescript
async function requestWithRetries(
  requestFn: () => Promise<any>,
  retryCount: number = 0,
  maxRetries: number = 10
): Promise<any> {
  try {
    return await requestFn();
  } catch (error) {
    if (retryCount >= maxRetries) {
      throw error;
    }

    // Exponential backoff: 1s, 2s, 4s, 8s, ...
    const delay = 1000 * Math.pow(2, retryCount);

    // Retry on rate limit (429) or server error (5xx)
    if (error.statusCode === 429 || error.statusCode >= 500) {
      console.log(`Retrying in ${delay}ms (attempt ${retryCount + 1}/${maxRetries})`);
      await sleep(delay);
      return requestWithRetries(requestFn, retryCount + 1, maxRetries);
    }

    throw error;
  }
}
```

### Rate Limit Handling

```javascript
// In Code node
const maxRetries = 5;
let retries = 0;

while (retries < maxRetries) {
  try {
    const response = await $http.request({
      method: 'GET',
      url: 'https://api.example.com/data'
    });
    return [{ json: response }];
  } catch (error) {
    if (error.response?.status === 429) {
      const retryAfter = error.response.headers['retry-after'] || Math.pow(2, retries);
      await new Promise(r => setTimeout(r, retryAfter * 1000));
      retries++;
    } else {
      throw error;
    }
  }
}

throw new Error('Max retries exceeded');
```

---

## Stop and Error Node
<!-- chunk: 20-stop-error | keywords: stop, error, throw -->

### Explicitly Throw Error

```json
{
  "type": "n8n-nodes-base.stopAndError",
  "typeVersion": 1,
  "position": [500, 300],
  "parameters": {
    "errorType": "errorMessage",
    "errorMessage": "Validation failed: missing required field"
  }
}
```

### Error with Details

```json
{
  "type": "n8n-nodes-base.stopAndError",
  "typeVersion": 1,
  "parameters": {
    "errorType": "errorObject",
    "errorObject": {
      "code": "VALIDATION_ERROR",
      "message": "Invalid input data",
      "details": {
        "field": "email",
        "reason": "Invalid email format"
      }
    }
  }
}
```

### Dynamic Error Message

```json
{
  "parameters": {
    "errorType": "errorMessage",
    "errorMessage": "={{ 'Failed to process: ' + $json.id + ' - Reason: ' + $json.failureReason }}"
  }
}
```

---

## Patterns
<!-- chunk: 20-patterns | keywords: patterns, try, catch -->

### Pattern 1: Try/Catch with If Node

```json
{
  "nodes": [
    {
      "name": "HTTP Request",
      "type": "n8n-nodes-base.httpRequest",
      "continueOnFail": true,
      "parameters": { "url": "https://api.example.com/data" }
    },
    {
      "name": "Check Error",
      "type": "n8n-nodes-base.if",
      "parameters": {
        "conditions": {
          "conditions": [{
            "leftValue": "={{ $json.error }}",
            "rightValue": "",
            "operator": { "type": "string", "operation": "notEquals" }
          }]
        }
      }
    },
    {
      "name": "Handle Error",
      "type": "n8n-nodes-base.set",
      "parameters": {
        "assignments": {
          "assignments": [
            { "name": "status", "value": "failed" },
            { "name": "error", "value": "={{ $json.error.message }}" }
          ]
        }
      }
    },
    {
      "name": "Process Success",
      "type": "n8n-nodes-base.set",
      "parameters": {
        "assignments": {
          "assignments": [
            { "name": "status", "value": "success" }
          ]
        }
      }
    }
  ],
  "connections": {
    "HTTP Request": { "main": [[{ "node": "Check Error" }]] },
    "Check Error": {
      "main": [
        [{ "node": "Handle Error" }],
        [{ "node": "Process Success" }]
      ]
    }
  }
}
```

### Pattern 2: Fallback on Error

```json
{
  "nodes": [
    {
      "name": "Primary API",
      "type": "n8n-nodes-base.httpRequest",
      "continueOnFail": true,
      "parameters": { "url": "https://primary-api.com/data" }
    },
    {
      "name": "Check Primary",
      "type": "n8n-nodes-base.if",
      "parameters": {
        "conditions": {
          "conditions": [{
            "leftValue": "={{ $json.error }}",
            "rightValue": "",
            "operator": { "type": "string", "operation": "exists" }
          }]
        }
      }
    },
    {
      "name": "Fallback API",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": { "url": "https://fallback-api.com/data" }
    },
    {
      "name": "Merge Results",
      "type": "n8n-nodes-base.merge",
      "parameters": { "mode": "chooseBranch" }
    }
  ]
}
```

### Pattern 3: Batch Processing with Error Collection

```json
{
  "nodes": [
    {
      "name": "Split In Batches",
      "type": "n8n-nodes-base.splitInBatches",
      "parameters": { "batchSize": 10 }
    },
    {
      "name": "Process Item",
      "type": "n8n-nodes-base.httpRequest",
      "continueOnFail": true,
      "parameters": { "url": "https://api.example.com/process/{{ $json.id }}" }
    },
    {
      "name": "Categorize",
      "type": "n8n-nodes-base.if",
      "parameters": {
        "conditions": {
          "conditions": [{
            "leftValue": "={{ $json.error }}",
            "rightValue": "",
            "operator": { "type": "string", "operation": "exists" }
          }]
        }
      }
    },
    {
      "name": "Collect Errors",
      "type": "n8n-nodes-base.set",
      "parameters": {
        "assignments": {
          "assignments": [
            { "name": "status", "value": "error" },
            { "name": "itemId", "value": "={{ $('Split In Batches').item.json.id }}" },
            { "name": "errorMessage", "value": "={{ $json.error.message }}" }
          ]
        }
      }
    },
    {
      "name": "Collect Success",
      "type": "n8n-nodes-base.set",
      "parameters": {
        "assignments": {
          "assignments": [
            { "name": "status", "value": "success" },
            { "name": "itemId", "value": "={{ $('Split In Batches').item.json.id }}" }
          ]
        }
      }
    }
  ]
}
```

### Pattern 4: Circuit Breaker

```javascript
// In Code node - Circuit Breaker pattern
const FAILURE_THRESHOLD = 5;
const RECOVERY_TIME = 60000; // 1 minute

// Get circuit state from static data
const staticData = $getWorkflowStaticData('global');
const circuit = staticData.circuit || { failures: 0, lastFailure: 0, open: false };

// Check if circuit is open
if (circuit.open) {
  const timeSinceFailure = Date.now() - circuit.lastFailure;
  if (timeSinceFailure < RECOVERY_TIME) {
    // Circuit is open, skip request
    return [{ json: { skipped: true, reason: 'Circuit breaker open' } }];
  }
  // Recovery time passed, try again
  circuit.open = false;
}

try {
  const response = await $http.request({
    method: 'GET',
    url: 'https://api.example.com/data'
  });

  // Success - reset failures
  circuit.failures = 0;
  staticData.circuit = circuit;

  return [{ json: response }];

} catch (error) {
  circuit.failures++;
  circuit.lastFailure = Date.now();

  if (circuit.failures >= FAILURE_THRESHOLD) {
    circuit.open = true;
  }

  staticData.circuit = circuit;
  throw error;
}
```

---

## Error Types Reference
<!-- chunk: 20-error-types | keywords: types, classes -->

| Error Class | Use Case |
|-------------|----------|
| `NodeOperationError` | Operation-specific errors |
| `NodeApiError` | API response errors |
| `NodeConnectionError` | Connection failures |
| `ExpressionError` | Expression evaluation errors |
| `WorkflowActivationError` | Workflow activation failures |

### Custom Error Throwing

```javascript
// In Code node
const { NodeOperationError } = require('n8n-workflow');

if (!$json.requiredField) {
  throw new NodeOperationError(
    $node,
    'Required field is missing',
    {
      description: 'The input must contain a requiredField property',
      itemIndex: $itemIndex
    }
  );
}
```

---

*Source: packages/nodes-base/nodes/StopAndError/, packages/nodes-base/nodes/ErrorTrigger/*
