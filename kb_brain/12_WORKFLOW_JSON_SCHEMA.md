# 12_WORKFLOW_JSON_SCHEMA.md
<!-- repo: n8n | version: 1.122.0 | generated: 2025-12-02 -->
<!-- tags: workflow, json, schema, structure, format -->

## Contents
- [Workflow Structure](#workflow-structure)
- [Node Structure](#node-structure)
- [Connections Format](#connections-format)
- [Settings Object](#settings-object)
- [Pin Data Format](#pin-data-format)
- [Complete Examples](#complete-examples)

---

## Workflow Structure
<!-- chunk: 12-workflow-structure | keywords: workflow, root, base, structure | source: packages/workflow/src/interfaces.ts -->

### Root Workflow Object

```json
{
  "id": "string (unique identifier)",
  "name": "string (workflow name)",
  "description": "string | null (optional description)",
  "active": "boolean (is workflow active)",
  "nodes": "INode[] (array of nodes)",
  "connections": "IConnections (connection map)",
  "settings": "IWorkflowSettings (optional)",
  "staticData": "IDataObject (optional, persisted data)",
  "pinData": "IPinData (optional, test data)",
  "versionId": "string (optional, version identifier)",
  "meta": "WorkflowFEMeta (optional, frontend metadata)",
  "tags": "string[] (optional, tag IDs)"
}
```

### Required Fields
| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Display name of the workflow |
| `nodes` | INode[] | Array of all nodes in the workflow |
| `connections` | IConnections | Map of node connections |
| `active` | boolean | Whether workflow is activated |

### Optional Fields
| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `id` | string | auto-generated | Unique workflow identifier |
| `description` | string | null | Workflow description |
| `settings` | object | {} | Workflow-level settings |
| `staticData` | object | {} | Persistent data across executions |
| `pinData` | object | {} | Pinned test data per node |
| `versionId` | string | null | Version tracking ID |
| `meta` | object | {} | Frontend metadata |
| `tags` | string[] | [] | Associated tag IDs |

---

## Node Structure
<!-- chunk: 12-node-structure | keywords: node, inode, parameters | source: packages/workflow/src/interfaces.ts -->

### INode Object

```json
{
  "id": "string (UUID)",
  "name": "string (unique display name)",
  "type": "string (node type identifier)",
  "typeVersion": "number (node version)",
  "position": "[number, number] (x, y coordinates)",
  "parameters": "INodeParameters (node configuration)",
  "credentials": "INodeCredentials (optional)",
  "disabled": "boolean (optional, default false)",
  "notes": "string (optional)",
  "notesInFlow": "boolean (optional)",
  "retryOnFail": "boolean (optional)",
  "maxTries": "number (optional)",
  "waitBetweenTries": "number (optional, milliseconds)",
  "alwaysOutputData": "boolean (optional)",
  "executeOnce": "boolean (optional)",
  "onError": "OnError (optional)",
  "continueOnFail": "boolean (optional, legacy)",
  "webhookId": "string (optional, for webhook nodes)"
}
```

### Node Type Identifiers
Format: `{package}.{nodeName}`

| Category | Example Type | Description |
|----------|-------------|-------------|
| Core | `n8n-nodes-base.httpRequest` | HTTP Request node |
| Trigger | `n8n-nodes-base.webhook` | Webhook trigger |
| Transform | `n8n-nodes-base.set` | Set/Edit Fields |
| Flow | `n8n-nodes-base.if` | Conditional branching |
| AI | `@n8n/n8n-nodes-langchain.agent` | AI Agent |

### Node Parameters (INodeParameters)
Parameters vary by node type. Common patterns:

```json
{
  "stringParam": "plain string value",
  "numberParam": 42,
  "booleanParam": true,
  "expressionParam": "={{ $json.field }}",
  "optionsParam": "selectedOption",

  "resourceLocator": {
    "__rl": true,
    "mode": "id",
    "value": "resource-id-here"
  },

  "fixedCollection": {
    "values": [
      { "name": "field1", "value": "value1" },
      { "name": "field2", "value": "value2" }
    ]
  },

  "filterConditions": {
    "options": {
      "caseSensitive": true,
      "typeValidation": "strict"
    },
    "conditions": [
      {
        "id": "uuid-here",
        "leftValue": "={{ $json.field }}",
        "operator": {
          "type": "string",
          "operation": "equals"
        },
        "rightValue": "expectedValue"
      }
    ],
    "combinator": "and"
  }
}
```

### Node Credentials (INodeCredentials)

```json
{
  "credentialTypeName": {
    "id": "credential-id-or-null",
    "name": "Credential Display Name"
  }
}
```

Example:
```json
{
  "googleSheetsOAuth2Api": {
    "id": "cred123",
    "name": "Google Sheets Account"
  },
  "slackApi": {
    "id": "cred456",
    "name": "Slack Bot Token"
  }
}
```

### OnError Options
```typescript
type OnError =
  | 'continueErrorOutput'    // Continue to error output
  | 'continueRegularOutput'  // Continue to regular output
  | 'stopWorkflow';          // Stop execution
```

---

## Connections Format
<!-- chunk: 12-connections | keywords: connections, wiring, links | source: packages/workflow/src/interfaces.ts -->

### IConnections Structure

```json
{
  "SourceNodeName": {
    "connectionType": [
      [
        { "node": "TargetNode1", "type": "main", "index": 0 },
        { "node": "TargetNode2", "type": "main", "index": 0 }
      ],
      [
        { "node": "SecondOutputTarget", "type": "main", "index": 0 }
      ]
    ]
  }
}
```

### Connection Types (NodeConnectionType)

| Type | Description | Use Case |
|------|-------------|----------|
| `main` | Standard data flow | Most nodes |
| `ai_languageModel` | LLM connection | AI nodes |
| `ai_memory` | Memory connection | AI agents |
| `ai_tool` | Tool connection | AI agents |
| `ai_agent` | Agent connection | Agent chains |
| `ai_chain` | Chain connection | LangChain |
| `ai_document` | Document loader | RAG |
| `ai_embedding` | Embedding model | Vector stores |
| `ai_vectorStore` | Vector store | RAG retrieval |
| `ai_retriever` | Retriever | RAG |
| `ai_outputParser` | Output parser | Structured output |
| `ai_textSplitter` | Text splitter | Document processing |
| `ai_reranker` | Reranker | Search results |

### Single Output to Single Input
```json
{
  "Trigger": {
    "main": [
      [
        { "node": "Process", "type": "main", "index": 0 }
      ]
    ]
  }
}
```

### Single Output to Multiple Inputs
```json
{
  "Trigger": {
    "main": [
      [
        { "node": "Process1", "type": "main", "index": 0 },
        { "node": "Process2", "type": "main", "index": 0 }
      ]
    ]
  }
}
```

### Multiple Outputs (If/Switch)
```json
{
  "If": {
    "main": [
      [
        { "node": "TrueBranch", "type": "main", "index": 0 }
      ],
      [
        { "node": "FalseBranch", "type": "main", "index": 0 }
      ]
    ]
  }
}
```

### AI Connections
```json
{
  "OpenAI Chat Model": {
    "ai_languageModel": [
      [
        { "node": "AI Agent", "type": "ai_languageModel", "index": 0 }
      ]
    ]
  },
  "Window Buffer Memory": {
    "ai_memory": [
      [
        { "node": "AI Agent", "type": "ai_memory", "index": 0 }
      ]
    ]
  }
}
```

### Merge Node Inputs
```json
{
  "Branch1": {
    "main": [
      [
        { "node": "Merge", "type": "main", "index": 0 }
      ]
    ]
  },
  "Branch2": {
    "main": [
      [
        { "node": "Merge", "type": "main", "index": 1 }
      ]
    ]
  }
}
```

---

## Settings Object
<!-- chunk: 12-settings | keywords: settings, configuration, workflow | source: packages/workflow/src/interfaces.ts -->

### IWorkflowSettings

```json
{
  "timezone": "string (e.g., 'America/New_York', 'UTC')",
  "errorWorkflow": "string (workflow ID for error handling)",
  "callerIds": "string (comma-separated workflow IDs)",
  "callerPolicy": "WorkflowCallerPolicy",
  "saveDataErrorExecution": "SaveDataExecution",
  "saveDataSuccessExecution": "SaveDataExecution",
  "saveManualExecutions": "boolean | 'DEFAULT'",
  "saveExecutionProgress": "boolean | 'DEFAULT'",
  "executionTimeout": "number (milliseconds)",
  "executionOrder": "'v0' | 'v1'"
}
```

### WorkflowCallerPolicy
```typescript
type WorkflowCallerPolicy =
  | 'any'                    // Any workflow can call
  | 'none'                   // Cannot be called
  | 'workflowsFromAList'     // Only listed workflows
  | 'workflowsFromSameOwner'; // Same owner's workflows
```

### SaveDataExecution
```typescript
type SaveDataExecution =
  | 'DEFAULT'  // Use instance setting
  | 'all'      // Always save
  | 'none';    // Never save
```

### Example Settings
```json
{
  "settings": {
    "timezone": "America/New_York",
    "executionTimeout": 300000,
    "executionOrder": "v1",
    "saveDataSuccessExecution": "all",
    "saveDataErrorExecution": "all",
    "callerPolicy": "workflowsFromSameOwner"
  }
}
```

---

## Pin Data Format
<!-- chunk: 12-pindata | keywords: pindata, test, mock | source: packages/workflow/src/interfaces.ts -->

### IPinData Structure

Pin data stores test/mock output for nodes:

```json
{
  "NodeName": [
    {
      "json": {
        "field1": "value1",
        "nested": { "key": "value" }
      },
      "binary": {
        "data": {
          "data": "base64-encoded-content",
          "mimeType": "image/png",
          "fileName": "image.png",
          "fileExtension": "png"
        }
      }
    },
    {
      "json": {
        "field1": "value2"
      }
    }
  ]
}
```

### INodeExecutionData Item

```json
{
  "json": "IDataObject (required, JSON data)",
  "binary": "IBinaryKeyData (optional, binary files)",
  "pairedItem": "IPairedItemData (optional, item tracking)",
  "error": "NodeError (optional, execution error)",
  "metadata": "IItemMetadata (optional)"
}
```

### Binary Data Format

```json
{
  "propertyName": {
    "data": "base64-encoded-string",
    "mimeType": "application/pdf",
    "fileName": "document.pdf",
    "fileExtension": "pdf",
    "fileSize": "102400",
    "fileType": "pdf"
  }
}
```

### File Types
```typescript
type FileType =
  | 'text' | 'json' | 'image'
  | 'audio' | 'video' | 'pdf' | 'html';
```

---

## Complete Examples
<!-- chunk: 12-examples | keywords: examples, complete, workflow | source: analysis -->

### Minimal Workflow

```json
{
  "name": "My First Workflow",
  "nodes": [
    {
      "id": "1",
      "name": "Manual Trigger",
      "type": "n8n-nodes-base.manualTrigger",
      "typeVersion": 1,
      "position": [0, 0],
      "parameters": {}
    }
  ],
  "connections": {},
  "active": false
}
```

### HTTP Request Workflow

```json
{
  "name": "Fetch API Data",
  "nodes": [
    {
      "id": "trigger-1",
      "name": "Schedule Trigger",
      "type": "n8n-nodes-base.scheduleTrigger",
      "typeVersion": 1.2,
      "position": [0, 0],
      "parameters": {
        "rule": {
          "interval": [
            { "field": "hours", "hoursInterval": 1 }
          ]
        }
      }
    },
    {
      "id": "http-1",
      "name": "HTTP Request",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.2,
      "position": [220, 0],
      "parameters": {
        "url": "https://api.example.com/data",
        "method": "GET",
        "authentication": "predefinedCredentialType",
        "nodeCredentialType": "httpHeaderAuth"
      },
      "credentials": {
        "httpHeaderAuth": {
          "id": "1",
          "name": "API Key"
        }
      }
    },
    {
      "id": "set-1",
      "name": "Transform Data",
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [440, 0],
      "parameters": {
        "mode": "manual",
        "duplicateItem": false,
        "assignments": {
          "assignments": [
            {
              "id": "a1",
              "name": "processedAt",
              "value": "={{ $now.toISO() }}",
              "type": "string"
            },
            {
              "id": "a2",
              "name": "data",
              "value": "={{ $json }}",
              "type": "object"
            }
          ]
        }
      }
    }
  ],
  "connections": {
    "Schedule Trigger": {
      "main": [
        [
          { "node": "HTTP Request", "type": "main", "index": 0 }
        ]
      ]
    },
    "HTTP Request": {
      "main": [
        [
          { "node": "Transform Data", "type": "main", "index": 0 }
        ]
      ]
    }
  },
  "settings": {
    "executionOrder": "v1",
    "timezone": "UTC"
  },
  "active": true
}
```

### Conditional Workflow with If Node

```json
{
  "name": "Conditional Processing",
  "nodes": [
    {
      "id": "webhook-1",
      "name": "Webhook",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 2,
      "position": [0, 0],
      "parameters": {
        "path": "process-order",
        "httpMethod": "POST",
        "responseMode": "lastNode"
      },
      "webhookId": "abc123"
    },
    {
      "id": "if-1",
      "name": "Check Amount",
      "type": "n8n-nodes-base.if",
      "typeVersion": 2.2,
      "position": [220, 0],
      "parameters": {
        "conditions": {
          "options": {
            "caseSensitive": true,
            "typeValidation": "strict"
          },
          "conditions": [
            {
              "id": "cond-1",
              "leftValue": "={{ $json.amount }}",
              "rightValue": 100,
              "operator": {
                "type": "number",
                "operation": "gt"
              }
            }
          ],
          "combinator": "and"
        }
      }
    },
    {
      "id": "high-1",
      "name": "High Value",
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [440, -100],
      "parameters": {
        "mode": "manual",
        "assignments": {
          "assignments": [
            {
              "id": "h1",
              "name": "priority",
              "value": "high",
              "type": "string"
            }
          ]
        },
        "includeOtherFields": true
      }
    },
    {
      "id": "low-1",
      "name": "Low Value",
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [440, 100],
      "parameters": {
        "mode": "manual",
        "assignments": {
          "assignments": [
            {
              "id": "l1",
              "name": "priority",
              "value": "standard",
              "type": "string"
            }
          ]
        },
        "includeOtherFields": true
      }
    }
  ],
  "connections": {
    "Webhook": {
      "main": [
        [
          { "node": "Check Amount", "type": "main", "index": 0 }
        ]
      ]
    },
    "Check Amount": {
      "main": [
        [
          { "node": "High Value", "type": "main", "index": 0 }
        ],
        [
          { "node": "Low Value", "type": "main", "index": 0 }
        ]
      ]
    }
  },
  "active": true
}
```

### AI Agent Workflow

```json
{
  "name": "AI Customer Support",
  "nodes": [
    {
      "id": "chat-1",
      "name": "Chat Trigger",
      "type": "@n8n/n8n-nodes-langchain.chatTrigger",
      "typeVersion": 1.1,
      "position": [0, 0],
      "parameters": {
        "mode": "webhook",
        "options": {}
      },
      "webhookId": "chat-webhook"
    },
    {
      "id": "agent-1",
      "name": "AI Agent",
      "type": "@n8n/n8n-nodes-langchain.agent",
      "typeVersion": 1.7,
      "position": [300, 0],
      "parameters": {
        "options": {
          "systemMessage": "You are a helpful customer support agent."
        }
      }
    },
    {
      "id": "llm-1",
      "name": "OpenAI Chat Model",
      "type": "@n8n/n8n-nodes-langchain.lmChatOpenAi",
      "typeVersion": 1.2,
      "position": [100, 200],
      "parameters": {
        "model": "gpt-4o",
        "options": {
          "temperature": 0.7
        }
      },
      "credentials": {
        "openAiApi": {
          "id": "openai-1",
          "name": "OpenAI API"
        }
      }
    },
    {
      "id": "memory-1",
      "name": "Window Buffer Memory",
      "type": "@n8n/n8n-nodes-langchain.memoryBufferWindow",
      "typeVersion": 1.3,
      "position": [100, 350],
      "parameters": {
        "sessionIdType": "fromInput",
        "sessionKey": "={{ $json.sessionId }}",
        "contextWindowLength": 10
      }
    }
  ],
  "connections": {
    "Chat Trigger": {
      "main": [
        [
          { "node": "AI Agent", "type": "main", "index": 0 }
        ]
      ]
    },
    "OpenAI Chat Model": {
      "ai_languageModel": [
        [
          { "node": "AI Agent", "type": "ai_languageModel", "index": 0 }
        ]
      ]
    },
    "Window Buffer Memory": {
      "ai_memory": [
        [
          { "node": "AI Agent", "type": "ai_memory", "index": 0 }
        ]
      ]
    }
  },
  "active": true
}
```

---

## Validation Rules
<!-- chunk: 12-validation | keywords: validation, rules, requirements | source: analysis -->

### Node Validation
1. **Node names** must be unique within the workflow
2. **Node IDs** should be UUIDs (generated automatically)
3. **Position** must be `[x, y]` number array
4. **Type** must match a registered node type
5. **TypeVersion** must be a valid version for that node type

### Connection Validation
1. **Source nodes** must exist in the nodes array
2. **Target nodes** must exist in the nodes array
3. **Connection types** must be valid NodeConnectionType
4. **Index** must be non-negative integer

### Credential Validation
1. **Credential ID** can be null (user assigns at runtime)
2. **Credential name** must be a non-empty string
3. **Credential type** must be registered

---

## Quick Reference
<!-- chunk: 12-quick-ref | keywords: quick, reference, summary | source: analysis -->

### Workflow JSON Template
```json
{
  "name": "",
  "nodes": [],
  "connections": {},
  "settings": {},
  "active": false
}
```

### Node Template
```json
{
  "id": "uuid",
  "name": "Unique Name",
  "type": "n8n-nodes-base.nodeName",
  "typeVersion": 1,
  "position": [0, 0],
  "parameters": {}
}
```

### Connection Template
```json
{
  "SourceNode": {
    "main": [[{ "node": "TargetNode", "type": "main", "index": 0 }]]
  }
}
```

→ Node Reference: [[13_NODE_REFERENCE]]
→ Expression Guide: [[14_EXPRESSION_GUIDE]]
→ Workflow Types: [[11_WORKFLOW_TYPES]]
