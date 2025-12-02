# 21_SUBWORKFLOWS.md
<!-- repo: n8n | version: 1.122.0 | generated: 2025-12-02 -->
<!-- tags: subworkflow, execute, workflow, call, nested -->

## Contents
- [Overview](#overview)
- [Execute Workflow Node](#execute-workflow-node)
- [Workflow Sources](#workflow-sources)
- [Data Passing](#data-passing)
- [Patterns](#patterns)
- [Best Practices](#best-practices)

---

## Overview
<!-- chunk: 21-overview | keywords: subworkflow, execute, modular -->

Sub-workflows allow you to:
- **Reuse** common logic across multiple workflows
- **Organize** complex workflows into smaller, manageable pieces
- **Encapsulate** functionality for maintainability
- **Run** workflows programmatically with custom inputs

---

## Execute Workflow Node
<!-- chunk: 21-node | keywords: executeWorkflow, node, config -->

### Basic Configuration

```json
{
  "type": "n8n-nodes-base.executeWorkflow",
  "typeVersion": 1.2,
  "position": [500, 300],
  "parameters": {
    "source": "database",
    "workflowId": "workflow-uuid-here",
    "mode": "each",
    "options": {}
  }
}
```

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `source` | string | Where to load workflow: `database`, `parameter`, `url` |
| `workflowId` | string | ID of workflow (for `database` source) |
| `workflowJson` | json | Inline workflow JSON (for `parameter` source) |
| `workflowUrl` | string | URL to fetch workflow (for `url` source) |
| `mode` | string | Execution mode: `each` (per item) or `once` (all items) |

### Mode Options

**Mode: each (Run Once for Each Item)**
```json
{
  "mode": "each",
  "options": {
    "waitForSubWorkflow": true
  }
}
```
- Executes sub-workflow once per input item
- Each execution receives single item
- Results collected and returned

**Mode: once (Run Once for All Items)**
```json
{
  "mode": "once",
  "options": {}
}
```
- Executes sub-workflow once with all items
- Sub-workflow receives entire array
- Single execution result returned

---

## Workflow Sources
<!-- chunk: 21-sources | keywords: database, parameter, url -->

### Source: Database (By ID)

Load workflow from n8n database by ID.

```json
{
  "source": "database",
  "workflowId": "wf_abc123",
  "options": {}
}
```

**Dynamic ID:**
```json
{
  "source": "database",
  "workflowId": "={{ $json.workflowToRun }}"
}
```

### Source: Parameter (Inline JSON)

Pass workflow definition directly as JSON.

```json
{
  "source": "parameter",
  "workflowJson": {
    "name": "Inline Workflow",
    "nodes": [
      {
        "name": "Start",
        "type": "n8n-nodes-base.executeWorkflowTrigger",
        "position": [250, 300]
      },
      {
        "name": "Process",
        "type": "n8n-nodes-base.set",
        "position": [500, 300],
        "parameters": {
          "assignments": {
            "assignments": [
              { "name": "processed", "value": true }
            ]
          }
        }
      }
    ],
    "connections": {
      "Start": {
        "main": [[{ "node": "Process", "type": "main", "index": 0 }]]
      }
    }
  }
}
```

### Source: URL

Fetch workflow JSON from external URL.

```json
{
  "source": "url",
  "workflowUrl": "https://example.com/workflows/process.json",
  "options": {}
}
```

---

## Data Passing
<!-- chunk: 21-data | keywords: input, output, passing -->

### Execute Workflow Trigger

Sub-workflows must start with Execute Workflow Trigger to receive data.

```json
{
  "type": "n8n-nodes-base.executeWorkflowTrigger",
  "typeVersion": 1.1,
  "position": [250, 300],
  "parameters": {}
}
```

**Accessing Input Data in Sub-Workflow:**
```javascript
// The input from parent workflow is available as normal $json
{{ $json.field }}

// All items from parent
{{ $input.all() }}
```

### Returning Data to Parent

Sub-workflow output (last node's data) is returned to parent workflow.

**Parent Workflow:**
```json
{
  "nodes": [
    {
      "name": "Get Data",
      "type": "n8n-nodes-base.set",
      "parameters": {
        "assignments": {
          "assignments": [
            { "name": "input", "value": "test data" }
          ]
        }
      }
    },
    {
      "name": "Execute Sub",
      "type": "n8n-nodes-base.executeWorkflow",
      "parameters": {
        "source": "database",
        "workflowId": "sub-workflow-id"
      }
    },
    {
      "name": "Use Result",
      "type": "n8n-nodes-base.set",
      "parameters": {
        "assignments": {
          "assignments": [
            { "name": "result", "value": "={{ $json.processedData }}" }
          ]
        }
      }
    }
  ]
}
```

**Sub-Workflow:**
```json
{
  "nodes": [
    {
      "name": "Trigger",
      "type": "n8n-nodes-base.executeWorkflowTrigger"
    },
    {
      "name": "Process",
      "type": "n8n-nodes-base.set",
      "parameters": {
        "assignments": {
          "assignments": [
            { "name": "processedData", "value": "={{ $json.input + ' - processed' }}" }
          ]
        }
      }
    }
  ]
}
```

### Passing Complex Data

```javascript
// Parent workflow - pass structured data
{
  "assignments": [
    { "name": "config", "value": "={{ { apiKey: 'xxx', mode: 'test' } }}", "type": "object" },
    { "name": "items", "value": "={{ $input.all().map(i => i.json) }}", "type": "array" }
  ]
}

// Sub-workflow - access structured data
{{ $json.config.apiKey }}
{{ $json.items[0].name }}
```

---

## Patterns
<!-- chunk: 21-patterns | keywords: patterns, examples -->

### Pattern 1: Reusable API Handler

**Parent Workflow:**
```json
{
  "nodes": [
    {
      "name": "Call API Handler",
      "type": "n8n-nodes-base.executeWorkflow",
      "parameters": {
        "source": "database",
        "workflowId": "api-handler-workflow",
        "mode": "once"
      }
    }
  ]
}
```

**Sub-Workflow (API Handler):**
```json
{
  "name": "API Handler",
  "nodes": [
    {
      "name": "Trigger",
      "type": "n8n-nodes-base.executeWorkflowTrigger"
    },
    {
      "name": "HTTP Request",
      "type": "n8n-nodes-base.httpRequest",
      "continueOnFail": true,
      "retryOnFail": true,
      "maxTries": 3,
      "parameters": {
        "url": "={{ $json.url }}",
        "method": "={{ $json.method || 'GET' }}",
        "body": "={{ $json.body }}"
      }
    },
    {
      "name": "Format Response",
      "type": "n8n-nodes-base.set",
      "parameters": {
        "assignments": {
          "assignments": [
            { "name": "success", "value": "={{ !$json.error }}" },
            { "name": "data", "value": "={{ $json.error ? null : $json }}" },
            { "name": "error", "value": "={{ $json.error?.message }}" }
          ]
        }
      }
    }
  ]
}
```

### Pattern 2: Fan-Out Processing

Process items in parallel sub-workflows.

```json
{
  "nodes": [
    {
      "name": "Get Items",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": { "url": "https://api.example.com/items" }
    },
    {
      "name": "Process Each",
      "type": "n8n-nodes-base.executeWorkflow",
      "parameters": {
        "source": "database",
        "workflowId": "item-processor",
        "mode": "each",
        "options": {
          "waitForSubWorkflow": true
        }
      }
    },
    {
      "name": "Aggregate Results",
      "type": "n8n-nodes-base.aggregate",
      "parameters": {
        "aggregate": "aggregateAllItemData",
        "destinationFieldName": "results"
      }
    }
  ]
}
```

### Pattern 3: Conditional Workflow Selection

```json
{
  "nodes": [
    {
      "name": "Determine Workflow",
      "type": "n8n-nodes-base.switch",
      "parameters": {
        "mode": "rules",
        "rules": {
          "values": [
            {
              "outputKey": "typeA",
              "conditions": {
                "conditions": [{ "leftValue": "={{ $json.type }}", "rightValue": "A" }]
              }
            },
            {
              "outputKey": "typeB",
              "conditions": {
                "conditions": [{ "leftValue": "={{ $json.type }}", "rightValue": "B" }]
              }
            }
          ]
        }
      }
    },
    {
      "name": "Run Type A Handler",
      "type": "n8n-nodes-base.executeWorkflow",
      "parameters": {
        "source": "database",
        "workflowId": "type-a-handler"
      }
    },
    {
      "name": "Run Type B Handler",
      "type": "n8n-nodes-base.executeWorkflow",
      "parameters": {
        "source": "database",
        "workflowId": "type-b-handler"
      }
    }
  ]
}
```

### Pattern 4: Recursive Processing

Sub-workflow can call itself for tree/graph processing.

```json
{
  "name": "Recursive Processor",
  "nodes": [
    {
      "name": "Trigger",
      "type": "n8n-nodes-base.executeWorkflowTrigger"
    },
    {
      "name": "Process Current",
      "type": "n8n-nodes-base.set",
      "parameters": {
        "assignments": {
          "assignments": [
            { "name": "processed", "value": "={{ $json.data }}" },
            { "name": "depth", "value": "={{ ($json.depth || 0) + 1 }}" }
          ]
        }
      }
    },
    {
      "name": "Check Children",
      "type": "n8n-nodes-base.if",
      "parameters": {
        "conditions": {
          "conditions": [
            {
              "leftValue": "={{ $json.children?.length > 0 && $json.depth < 10 }}",
              "rightValue": true
            }
          ]
        }
      }
    },
    {
      "name": "Process Children",
      "type": "n8n-nodes-base.executeWorkflow",
      "parameters": {
        "source": "database",
        "workflowId": "{{ $workflow.id }}",
        "mode": "each"
      }
    }
  ]
}
```

### Pattern 5: Error Isolation

Isolate risky operations in sub-workflow.

```json
{
  "nodes": [
    {
      "name": "Safe Processing",
      "type": "n8n-nodes-base.set"
    },
    {
      "name": "Risky Operation",
      "type": "n8n-nodes-base.executeWorkflow",
      "continueOnFail": true,
      "parameters": {
        "source": "database",
        "workflowId": "risky-processor"
      }
    },
    {
      "name": "Check Result",
      "type": "n8n-nodes-base.if",
      "parameters": {
        "conditions": {
          "conditions": [{
            "leftValue": "={{ $json.error }}",
            "operator": { "operation": "exists" }
          }]
        }
      }
    },
    {
      "name": "Handle Failure",
      "type": "n8n-nodes-base.set"
    },
    {
      "name": "Continue Success",
      "type": "n8n-nodes-base.set"
    }
  ]
}
```

---

## Best Practices
<!-- chunk: 21-best-practices | keywords: best, practices -->

### 1. Use Descriptive Names
```
- "Process Order" instead of "Sub-Workflow 1"
- "Validate User Input" instead of "Helper"
```

### 2. Document Input/Output
```json
// Sub-workflow trigger with notes
{
  "name": "Input: { orderId, items[] }",
  "type": "n8n-nodes-base.executeWorkflowTrigger",
  "notes": "Input: { orderId: string, items: Array<{sku, qty}> }\nOutput: { success: boolean, processedItems: number }"
}
```

### 3. Limit Nesting Depth
- Maximum 3-4 levels of nesting
- Use horizontal composition over deep nesting

### 4. Handle Errors Appropriately
```json
{
  "continueOnFail": true,
  "parameters": {
    "options": {
      "waitForSubWorkflow": true
    }
  }
}
```

### 5. Consider Memory
- `mode: "each"` uses more memory for large datasets
- `mode: "once"` is more efficient for bulk processing

### 6. Use Static Data for State
```javascript
// In sub-workflow Code node
const staticData = $getWorkflowStaticData('global');
staticData.lastRun = new Date().toISOString();
staticData.runCount = (staticData.runCount || 0) + 1;
```

---

## API Interface
<!-- chunk: 21-api | keywords: api, interface -->

### executeWorkflow Method

```typescript
async executeWorkflow(
  workflowInfo: IExecuteWorkflowInfo,
  inputData?: INodeExecutionData[],
  parentCallbackManager?: CallbackManager,
  options?: {
    doNotWaitToFinish?: boolean;
    parentExecution?: RelatedExecution;
  }
): Promise<ExecuteWorkflowData>

interface IExecuteWorkflowInfo {
  code?: IWorkflowBase;  // Inline workflow definition
  id?: string;           // Workflow ID reference
}

interface ExecuteWorkflowData {
  executionId: string;
  data: INodeExecutionData[][];
}
```

---

*Source: packages/nodes-base/nodes/ExecuteWorkflow/*
