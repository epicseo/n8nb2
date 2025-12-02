# 06_WORKFLOW_BUILDER.md
<!-- repo: n8n | version: 1.122.0 | generated: 2025-12-02 -->
<!-- tags: workflow, json, schema, expressions, templates -->

## Contents
- [Workflow JSON Structure](#workflow-json-structure)
- [Node Structure](#node-structure)
- [Connections Format](#connections-format)
- [Expression Syntax](#expression-syntax)
- [Built-in Variables](#built-in-variables)
- [Extension Methods](#extension-methods)
- [Workflow Templates](#workflow-templates)
- [AI Workflow Patterns](#ai-workflow-patterns)

---

## Workflow JSON Structure
<!-- chunk: 06-structure | keywords: workflow, root, json, schema -->

### Root Object
```json
{
  "name": "Workflow Name",
  "nodes": [],
  "connections": {},
  "settings": { "executionOrder": "v1" },
  "active": false
}
```

### Required Fields
| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Display name |
| `nodes` | INode[] | Array of nodes |
| `connections` | IConnections | Connection map |
| `active` | boolean | Is workflow activated |

### Optional Fields
| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique workflow ID |
| `description` | string | Workflow description |
| `settings` | object | Workflow settings |
| `pinData` | object | Test data per node |
| `tags` | string[] | Tag IDs |

### Settings Object
```json
{
  "settings": {
    "timezone": "America/New_York",
    "executionOrder": "v1",
    "executionTimeout": 300000,
    "saveDataSuccessExecution": "all",
    "saveDataErrorExecution": "all",
    "callerPolicy": "workflowsFromSameOwner"
  }
}
```

---

## Node Structure
<!-- chunk: 06-node | keywords: node, inode, parameters -->

### Complete Node
```json
{
  "id": "unique-uuid",
  "name": "Node Display Name",
  "type": "n8n-nodes-base.nodetype",
  "typeVersion": 1.0,
  "position": [250, 300],
  "parameters": {},
  "credentials": {
    "credentialType": { "id": "cred-uuid", "name": "Cred Name" }
  },
  "webhookId": "optional-for-webhooks",
  "disabled": false,
  "onError": "continueErrorOutput",
  "retryOnFail": false,
  "maxTries": 3,
  "continueOnFail": false
}
```

### Node Type Format
`{package}.{nodeName}` or `@{scope}/{package}.{nodeName}`

Examples:
- `n8n-nodes-base.httpRequest`
- `n8n-nodes-base.webhook`
- `n8n-nodes-base.if`
- `@n8n/n8n-nodes-langchain.agent`
- `@n8n/n8n-nodes-langchain.lmChatOpenAi`

### Position Guidelines
- Start: `[250, 300]`
- Horizontal spacing: +250px
- Vertical branches: +/-150px
- Keep Y: 0-600

### Common Parameter Patterns

**Expression Value:**
```json
{ "field": "={{ $json.value }}" }
```

**Resource Locator:**
```json
{
  "channel": { "mode": "id", "value": "C0123456789" }
}
```

**Fixed Collection:**
```json
{
  "assignments": {
    "assignments": [
      { "id": "uuid", "name": "field", "value": "text", "type": "string" }
    ]
  }
}
```

**Filter Conditions:**
```json
{
  "conditions": {
    "conditions": [{
      "leftValue": "={{ $json.status }}",
      "rightValue": "active",
      "operator": { "type": "string", "operation": "equals" }
    }],
    "combinator": "and"
  }
}
```

---

## Connections Format
<!-- chunk: 06-connections | keywords: connections, wiring, links -->

### Basic Connection
```json
{
  "SourceNode": {
    "main": [[{ "node": "TargetNode", "type": "main", "index": 0 }]]
  }
}
```

### Multiple Targets (Fan-out)
```json
{
  "Trigger": {
    "main": [[
      { "node": "Process1", "type": "main", "index": 0 },
      { "node": "Process2", "type": "main", "index": 0 }
    ]]
  }
}
```

### Multiple Outputs (If/Switch)
```json
{
  "If Node": {
    "main": [
      [{ "node": "TrueBranch", "type": "main", "index": 0 }],
      [{ "node": "FalseBranch", "type": "main", "index": 0 }]
    ]
  }
}
```

### Merge Node Inputs
```json
{
  "Branch1": {
    "main": [[{ "node": "Merge", "type": "main", "index": 0 }]]
  },
  "Branch2": {
    "main": [[{ "node": "Merge", "type": "main", "index": 1 }]]
  }
}
```

### AI Connections
| Type | Purpose | Max |
|------|---------|-----|
| `ai_languageModel` | LLM provider | 1 |
| `ai_memory` | Conversation history | 1 |
| `ai_tool` | Agent tools | Unlimited |
| `ai_vectorStore` | Vector database | 1 |
| `ai_retriever` | Document retrieval | 1 |
| `ai_embedding` | Embedding model | 1 |

```json
{
  "OpenAI Chat": {
    "ai_languageModel": [[{ "node": "AI Agent", "type": "ai_languageModel", "index": 0 }]]
  },
  "Buffer Memory": {
    "ai_memory": [[{ "node": "AI Agent", "type": "ai_memory", "index": 0 }]]
  }
}
```

---

## Expression Syntax
<!-- chunk: 06-expressions | keywords: expression, syntax, javascript -->

### Basic Syntax
```javascript
={{ expression }}
```

### Expression Types
| Type | Example |
|------|---------|
| Field access | `={{ $json.field }}` |
| Nested access | `={{ $json.user.email }}` |
| Array access | `={{ $json.items[0] }}` |
| Method call | `={{ $json.name.toUpperCase() }}` |
| Arithmetic | `={{ $json.price * 1.1 }}` |
| Concatenation | `={{ "Hello " + $json.name }}` |
| Ternary | `={{ $json.age >= 18 ? "adult" : "minor" }}` |
| Optional chaining | `={{ $json.user?.email }}` |
| Nullish coalescing | `={{ $json.name ?? "Unknown" }}` |

---

## Built-in Variables
<!-- chunk: 06-variables | keywords: variables, json, input, node -->

### $json - Current Item
```javascript
{{ $json }}                    // Entire item
{{ $json.fieldName }}          // Specific field
{{ $json.user.email }}         // Nested field
{{ $json.items[0].name }}      // Array access
```

### $input - Input Methods
```javascript
{{ $input.first() }}           // First input item
{{ $input.last() }}            // Last input item
{{ $input.all() }}             // All items as array
{{ $input.all().length }}      // Item count
```

### $node - Other Node Data
```javascript
{{ $('NodeName').item.json }}           // Node output
{{ $('HTTP Request').first().json }}    // First item from node
{{ $node["Node Name"].json.data }}      // Alternative syntax
```

### $workflow - Metadata
```javascript
{{ $workflow.id }}             // Workflow ID
{{ $workflow.name }}           // Workflow name
{{ $workflow.active }}         // Is active
```

### $env - Environment
```javascript
{{ $env.API_KEY }}             // Environment variable
{{ $env.DATABASE_URL }}        // Database URL
```

### $now / $today - DateTime
```javascript
{{ $now }}                     // Current DateTime
{{ $now.toISO() }}             // ISO string
{{ $now.toFormat('yyyy-MM-dd') }}  // Formatted
{{ $today }}                   // Today at midnight
```

### $execution - Execution Info
```javascript
{{ $execution.id }}            // Execution ID
{{ $execution.mode }}          // Execution mode
{{ $itemIndex }}               // Current item index
{{ $runIndex }}                // Current run index
```

---

## Extension Methods
<!-- chunk: 06-methods | keywords: string, array, date, methods -->

### String Methods
```javascript
{{ "hello".toUpperCase() }}        // "HELLO"
{{ "HELLO".toLowerCase() }}        // "hello"
{{ "hello world".toTitleCase() }}  // "Hello World"
{{ "test@email.com".isEmail() }}   // true
{{ "https://n8n.io".isUrl() }}     // true
{{ "text".extractEmail() }}        // Extract email
{{ "hello".base64Encode() }}       // "aGVsbG8="
{{ "hello".hash("sha256") }}       // SHA-256 hash
{{ "42".toNumber() }}              // 42
{{ "<p>text</p>".removeTags() }}   // "text"
```

### Array Methods
```javascript
{{ [1, 2, 3].first() }}            // 1
{{ [1, 2, 3].last() }}             // 3
{{ [1, 2, 3, 4, 5].sum() }}        // 15
{{ [1, 2, 3, 4, 5].average() }}    // 3
{{ [1, 2, 3, 4, 5].min() }}        // 1
{{ [1, 2, 3, 4, 5].max() }}        // 5
{{ [1, 1, 2, 2, 3].unique() }}     // [1, 2, 3]
{{ [1, 2, 3].chunk(2) }}           // [[1, 2], [3]]
{{ [{name: "a"}].pluck("name") }}  // ["a"]
```

### Date Methods (Luxon)
```javascript
{{ $now.toFormat("yyyy-MM-dd") }}      // "2024-01-15"
{{ $now.toFormat("HH:mm:ss") }}        // "10:30:00"
{{ $now.plus(7, "days") }}             // Add 7 days
{{ $now.minus(1, "month") }}           // Subtract 1 month
{{ $now.startOf("day") }}              // Start of today
{{ $now.endOf("month") }}              // End of month
{{ $now.year }}                        // 2024
{{ $now.month }}                       // 1
{{ $now.day }}                         // 15
{{ $now.weekday }}                     // 1 (Monday)
{{ "2024-01-15".toDateTime() }}        // Parse string
```

### Format Tokens
| Token | Output | Example |
|-------|--------|---------|
| `yyyy` | 4-digit year | 2024 |
| `MM` | 2-digit month | 01 |
| `dd` | 2-digit day | 15 |
| `HH` | 24-hour hour | 14 |
| `mm` | Minutes | 30 |
| `ss` | Seconds | 45 |
| `cccc` | Full weekday | Monday |
| `MMMM` | Full month | January |

### Number Methods
```javascript
{{ (3.7).floor() }}                // 3
{{ (3.2).ceil() }}                 // 4
{{ (3.456).round(2) }}             // 3.46
{{ (1234567.89).format() }}        // "1,234,567.89"
```

---

## Workflow Templates
<!-- chunk: 06-templates | keywords: templates, patterns, examples -->

### Webhook to Slack
```json
{
  "name": "Webhook to Slack",
  "nodes": [
    {
      "id": "1", "name": "Webhook",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 2.1,
      "position": [250, 300],
      "webhookId": "webhook-uuid",
      "parameters": {
        "httpMethod": "POST",
        "path": "incoming",
        "responseMode": "onReceived"
      }
    },
    {
      "id": "2", "name": "Slack",
      "type": "n8n-nodes-base.slack",
      "typeVersion": 2.4,
      "position": [500, 300],
      "parameters": {
        "resource": "message",
        "operation": "post",
        "channel": { "mode": "id", "value": "C0123456789" },
        "text": "={{ 'New: ' + $json.message }}"
      },
      "credentials": {
        "slackOAuth2Api": { "id": "1", "name": "Slack" }
      }
    }
  ],
  "connections": {
    "Webhook": {
      "main": [[{ "node": "Slack", "type": "main", "index": 0 }]]
    }
  },
  "settings": { "executionOrder": "v1" }
}
```

### Scheduled Database Report
```json
{
  "name": "Daily Report",
  "nodes": [
    {
      "id": "1", "name": "Schedule",
      "type": "n8n-nodes-base.scheduleTrigger",
      "typeVersion": 1.2,
      "position": [250, 300],
      "parameters": {
        "rule": {
          "interval": [{ "field": "cronExpression", "expression": "0 9 * * 1-5" }]
        }
      }
    },
    {
      "id": "2", "name": "PostgreSQL",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2.5,
      "position": [500, 300],
      "parameters": {
        "operation": "executeQuery",
        "query": "SELECT * FROM orders WHERE created_at >= NOW() - INTERVAL '1 day'"
      },
      "credentials": { "postgres": { "id": "1", "name": "DB" } }
    },
    {
      "id": "3", "name": "Email",
      "type": "n8n-nodes-base.emailSend",
      "typeVersion": 2.1,
      "position": [750, 300],
      "parameters": {
        "fromEmail": "reports@company.com",
        "toEmail": "team@company.com",
        "subject": "={{ 'Daily Report - ' + $now.toFormat('yyyy-MM-dd') }}",
        "html": "={{ '<h1>Orders</h1><p>Total: ' + $json.length + '</p>' }}"
      },
      "credentials": { "smtp": { "id": "1", "name": "SMTP" } }
    }
  ],
  "connections": {
    "Schedule": { "main": [[{ "node": "PostgreSQL", "type": "main", "index": 0 }]] },
    "PostgreSQL": { "main": [[{ "node": "Email", "type": "main", "index": 0 }]] }
  }
}
```

### Conditional Processing
```json
{
  "name": "Conditional Workflow",
  "nodes": [
    {
      "id": "1", "name": "Webhook",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 2.1,
      "position": [250, 300],
      "webhookId": "hook-uuid",
      "parameters": { "httpMethod": "POST", "path": "process" }
    },
    {
      "id": "2", "name": "If",
      "type": "n8n-nodes-base.if",
      "typeVersion": 2.2,
      "position": [500, 300],
      "parameters": {
        "conditions": {
          "conditions": [{
            "leftValue": "={{ $json.priority }}",
            "rightValue": "high",
            "operator": { "type": "string", "operation": "equals" }
          }],
          "combinator": "and"
        }
      }
    },
    {
      "id": "3", "name": "High Priority",
      "type": "n8n-nodes-base.slack",
      "typeVersion": 2.4,
      "position": [750, 150],
      "parameters": {
        "resource": "message", "operation": "post",
        "channel": { "mode": "id", "value": "C-URGENT" },
        "text": "={{ 'URGENT: ' + $json.message }}"
      }
    },
    {
      "id": "4", "name": "Normal Priority",
      "type": "n8n-nodes-base.emailSend",
      "typeVersion": 2.1,
      "position": [750, 450],
      "parameters": {
        "toEmail": "team@company.com",
        "subject": "={{ $json.subject }}",
        "text": "={{ $json.message }}"
      }
    }
  ],
  "connections": {
    "Webhook": { "main": [[{ "node": "If", "type": "main", "index": 0 }]] },
    "If": {
      "main": [
        [{ "node": "High Priority", "type": "main", "index": 0 }],
        [{ "node": "Normal Priority", "type": "main", "index": 0 }]
      ]
    }
  }
}
```

---

## AI Workflow Patterns
<!-- chunk: 06-ai | keywords: ai, agent, langchain, chat -->

### Basic AI Chat
```json
{
  "name": "AI Chat",
  "nodes": [
    {
      "id": "1", "name": "Chat Trigger",
      "type": "@n8n/n8n-nodes-langchain.chatTrigger",
      "typeVersion": 1.1,
      "position": [250, 300],
      "webhookId": "chat-uuid",
      "parameters": {
        "mode": "hostedChat",
        "options": { "title": "AI Assistant" }
      }
    },
    {
      "id": "2", "name": "AI Agent",
      "type": "@n8n/n8n-nodes-langchain.agent",
      "typeVersion": 1.7,
      "position": [700, 300],
      "parameters": {
        "agent": "conversationalAgent",
        "text": "={{ $json.chatInput }}",
        "options": { "systemMessage": "You are a helpful assistant." }
      }
    },
    {
      "id": "3", "name": "OpenAI",
      "type": "@n8n/n8n-nodes-langchain.lmChatOpenAi",
      "typeVersion": 1.2,
      "position": [450, 150],
      "parameters": { "model": "gpt-4o", "options": { "temperature": 0.7 } },
      "credentials": { "openAiApi": { "id": "1", "name": "OpenAI" } }
    },
    {
      "id": "4", "name": "Memory",
      "type": "@n8n/n8n-nodes-langchain.memoryBufferWindow",
      "typeVersion": 1.3,
      "position": [450, 450],
      "parameters": {
        "sessionIdType": "fromInput",
        "sessionKey": "={{ $json.sessionId }}",
        "contextWindowLength": 10
      }
    }
  ],
  "connections": {
    "Chat Trigger": { "main": [[{ "node": "AI Agent", "type": "main", "index": 0 }]] },
    "OpenAI": { "ai_languageModel": [[{ "node": "AI Agent", "type": "ai_languageModel", "index": 0 }]] },
    "Memory": { "ai_memory": [[{ "node": "AI Agent", "type": "ai_memory", "index": 0 }]] }
  }
}
```

### AI Agent with Tools
```json
{
  "name": "AI Agent with Tools",
  "nodes": [
    {
      "id": "1", "name": "Chat Trigger",
      "type": "@n8n/n8n-nodes-langchain.chatTrigger",
      "typeVersion": 1.1,
      "position": [250, 300],
      "webhookId": "chat-uuid",
      "parameters": { "mode": "hostedChat" }
    },
    {
      "id": "2", "name": "AI Agent",
      "type": "@n8n/n8n-nodes-langchain.agent",
      "typeVersion": 1.7,
      "position": [700, 300],
      "parameters": {
        "agent": "conversationalAgent",
        "text": "={{ $json.chatInput }}",
        "options": {
          "systemMessage": "You are a helpful assistant with access to tools.",
          "maxIterations": 10
        }
      }
    },
    {
      "id": "3", "name": "OpenAI",
      "type": "@n8n/n8n-nodes-langchain.lmChatOpenAi",
      "typeVersion": 1.2,
      "position": [450, 100],
      "parameters": { "model": "gpt-4o" },
      "credentials": { "openAiApi": { "id": "1", "name": "OpenAI" } }
    },
    {
      "id": "4", "name": "Calculator",
      "type": "@n8n/n8n-nodes-langchain.toolCalculator",
      "typeVersion": 1,
      "position": [450, 500],
      "parameters": {}
    },
    {
      "id": "5", "name": "HTTP Tool",
      "type": "@n8n/n8n-nodes-langchain.toolHttpRequest",
      "typeVersion": 1.1,
      "position": [600, 500],
      "parameters": {
        "name": "search_api",
        "description": "Search the web",
        "method": "GET",
        "url": "https://api.search.com/search"
      }
    }
  ],
  "connections": {
    "Chat Trigger": { "main": [[{ "node": "AI Agent", "type": "main", "index": 0 }]] },
    "OpenAI": { "ai_languageModel": [[{ "node": "AI Agent", "type": "ai_languageModel", "index": 0 }]] },
    "Calculator": { "ai_tool": [[{ "node": "AI Agent", "type": "ai_tool", "index": 0 }]] },
    "HTTP Tool": { "ai_tool": [[{ "node": "AI Agent", "type": "ai_tool", "index": 0 }]] }
  }
}
```

### RAG Pipeline
```json
{
  "name": "RAG Chat",
  "nodes": [
    {
      "id": "1", "name": "Chat Trigger",
      "type": "@n8n/n8n-nodes-langchain.chatTrigger",
      "typeVersion": 1.1,
      "position": [250, 300],
      "webhookId": "rag-uuid",
      "parameters": { "mode": "hostedChat" }
    },
    {
      "id": "2", "name": "QA Chain",
      "type": "@n8n/n8n-nodes-langchain.chainRetrievalQa",
      "typeVersion": 1.3,
      "position": [700, 300],
      "parameters": { "query": "={{ $json.chatInput }}" }
    },
    {
      "id": "3", "name": "OpenAI",
      "type": "@n8n/n8n-nodes-langchain.lmChatOpenAi",
      "typeVersion": 1.2,
      "position": [450, 100],
      "parameters": { "model": "gpt-4o" },
      "credentials": { "openAiApi": { "id": "1", "name": "OpenAI" } }
    },
    {
      "id": "4", "name": "Retriever",
      "type": "@n8n/n8n-nodes-langchain.retrieverVectorStore",
      "typeVersion": 1,
      "position": [450, 500],
      "parameters": { "topK": 5 }
    },
    {
      "id": "5", "name": "Pinecone",
      "type": "@n8n/n8n-nodes-langchain.vectorStorePinecone",
      "typeVersion": 1,
      "position": [200, 600],
      "parameters": { "mode": "load", "pineconeIndex": "knowledge" },
      "credentials": { "pineconeApi": { "id": "1", "name": "Pinecone" } }
    },
    {
      "id": "6", "name": "Embeddings",
      "type": "@n8n/n8n-nodes-langchain.embeddingsOpenAi",
      "typeVersion": 1.1,
      "position": [200, 750],
      "parameters": { "model": "text-embedding-3-small" },
      "credentials": { "openAiApi": { "id": "1", "name": "OpenAI" } }
    }
  ],
  "connections": {
    "Chat Trigger": { "main": [[{ "node": "QA Chain", "type": "main", "index": 0 }]] },
    "OpenAI": { "ai_languageModel": [[{ "node": "QA Chain", "type": "ai_languageModel", "index": 0 }]] },
    "Retriever": { "ai_retriever": [[{ "node": "QA Chain", "type": "ai_retriever", "index": 0 }]] },
    "Pinecone": { "ai_vectorStore": [[{ "node": "Retriever", "type": "ai_vectorStore", "index": 0 }]] },
    "Embeddings": { "ai_embedding": [[{ "node": "Pinecone", "type": "ai_embedding", "index": 0 }]] }
  }
}
```

---

## Validation Checklist

Before creating workflows:
- [ ] All nodes have unique `id` and `name`
- [ ] All nodes have correct `type` and `typeVersion`
- [ ] Positions don't overlap
- [ ] All connections reference existing nodes
- [ ] Credential types match node requirements
- [ ] Expressions use `={{ }}` syntax
- [ ] If/Switch outputs correctly indexed
- [ ] AI connections use correct types

---

*Source: packages/workflow/src/, generated templates*
