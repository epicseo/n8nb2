# 04_NODE_REFERENCE.md
<!-- repo: n8n | version: 1.122.0 | generated: 2025-12-02 -->
<!-- tags: nodes, reference, integrations, triggers, actions, ai, langchain -->

## Contents
- [Overview](#overview)
- [Core Flow Control](#core-flow-control)
- [Data Transform](#data-transform)
- [Triggers](#triggers)
- [Communication](#communication)
- [Project Management](#project-management)
- [CRM Sales](#crm-sales)
- [Database](#database)
- [File Storage](#file-storage)
- [AI Language Models](#ai-language-models)
- [AI Agents](#ai-agents)
- [AI Memory](#ai-memory)
- [AI Tools](#ai-tools)
- [AI Vector Stores](#ai-vector-stores)
- [AI Embeddings](#ai-embeddings)
- [AI Document Loaders](#ai-document-loaders)
- [AI Chains](#ai-chains)
- [Utility Nodes](#utility-nodes)
- [Node JSON Structure](#node-json-structure)

---

## Overview
<!-- chunk: 04-overview | keywords: nodes, statistics, categories -->

### Key Statistics
- **Total Nodes:** 649 (531 nodes-base + 118 nodes-langchain)
- **Trigger Nodes:** 112 (22%)
- **Action Nodes:** 391 (78%)
- **AI Nodes:** 118
- **Credential Types:** 389

### AI Connection Types
| Type | Purpose | Max |
|------|---------|-----|
| `ai_languageModel` | LLM provider | 1 |
| `ai_memory` | Conversation history | 1 |
| `ai_tool` | Agent tools | Unlimited |
| `ai_vectorStore` | Vector database | 1 |
| `ai_retriever` | Document retrieval | 1 |
| `ai_outputParser` | Output formatting | 1 |
| `ai_textSplitter` | Text chunking | 1 |
| `ai_embedding` | Embedding model | 1 |
| `ai_document` | Document loader | Unlimited |

---

## Core Flow Control
<!-- chunk: 04-flow | keywords: if, switch, merge, split, loop, wait -->

### If Node
**Type:** `n8n-nodes-base.if` | **Version:** 2.2

```json
{
  "type": "n8n-nodes-base.if",
  "typeVersion": 2.2,
  "parameters": {
    "conditions": {
      "conditions": [{
        "leftValue": "={{ $json.status }}",
        "rightValue": "active",
        "operator": { "type": "string", "operation": "equals" }
      }],
      "combinator": "and"
    }
  }
}
```

**Operators:**
| Type | Operations |
|------|------------|
| String | equals, notEquals, contains, notContains, startsWith, endsWith, regex |
| Number | equals, notEquals, gt, gte, lt, lte |
| Boolean | true, false |
| Array | contains, notContains, lengthEquals, empty, notEmpty |
| DateTime | after, before, equals |

**Outputs:** 2 branches (true=0, false=1)

### Switch Node
**Type:** `n8n-nodes-base.switch` | **Version:** 3.3

```json
{
  "type": "n8n-nodes-base.switch",
  "typeVersion": 3.3,
  "parameters": {
    "mode": "rules",
    "rules": {
      "values": [
        {
          "outputKey": "high",
          "conditions": {
            "conditions": [{
              "leftValue": "={{ $json.priority }}",
              "rightValue": "high",
              "operator": { "type": "string", "operation": "equals" }
            }]
          }
        }
      ]
    },
    "options": { "fallbackOutput": "extra" }
  }
}
```

**Modes:** `rules`, `expression`

### Merge Node
**Type:** `n8n-nodes-base.merge` | **Version:** 3.2

```json
{
  "type": "n8n-nodes-base.merge",
  "typeVersion": 3.2,
  "parameters": {
    "mode": "combine",
    "mergeByFields": {
      "values": [{ "field1": "id", "field2": "userId" }]
    },
    "joinMode": "keepMatches"
  }
}
```

**Modes:** `append`, `combine`, `chooseBranch`, `multiplex`
**Join Modes:** `keepMatches`, `keepNonMatches`, `keepEverything`, `enrichInput1`, `enrichInput2`

### Split In Batches
**Type:** `n8n-nodes-base.splitInBatches` | **Version:** 3

```json
{
  "type": "n8n-nodes-base.splitInBatches",
  "typeVersion": 3,
  "parameters": {
    "batchSize": 10,
    "options": { "reset": false }
  }
}
```

**Outputs:** Index 0 (done), Index 1 (loop)

### Wait Node
**Type:** `n8n-nodes-base.wait` | **Version:** 1.1

```json
{
  "type": "n8n-nodes-base.wait",
  "typeVersion": 1.1,
  "parameters": {
    "resume": "timeInterval",
    "amount": 5,
    "unit": "seconds"
  }
}
```

**Resume Options:** `timeInterval`, `specificTime`, `webhook`, `form`

---

## Data Transform
<!-- chunk: 04-transform | keywords: set, code, filter, aggregate, sort -->

### Set Node (Edit Fields)
**Type:** `n8n-nodes-base.set` | **Version:** 3.4

```json
{
  "type": "n8n-nodes-base.set",
  "typeVersion": 3.4,
  "parameters": {
    "mode": "manual",
    "assignments": {
      "assignments": [
        {
          "name": "fullName",
          "value": "={{ $json.firstName }} {{ $json.lastName }}",
          "type": "string"
        }
      ]
    },
    "includeOtherFields": true
  }
}
```

### Code Node
**Type:** `n8n-nodes-base.code` | **Version:** 2

```json
{
  "type": "n8n-nodes-base.code",
  "typeVersion": 2,
  "parameters": {
    "mode": "runOnceForAllItems",
    "language": "javaScript",
    "jsCode": "const results = [];\nfor (const item of $input.all()) {\n  results.push({ json: { ...item.json, processed: true } });\n}\nreturn results;"
  }
}
```

**Available APIs:**
```javascript
$input.all()              // All items
$input.first()            // First item
$('NodeName').all()       // Items from node
$workflow.id              // Workflow ID
$execution.id             // Execution ID
$env.VAR                  // Environment variable
```

### Filter Node
**Type:** `n8n-nodes-base.filter` | **Version:** 2.2

```json
{
  "type": "n8n-nodes-base.filter",
  "typeVersion": 2.2,
  "parameters": {
    "conditions": {
      "conditions": [{
        "leftValue": "={{ $json.status }}",
        "rightValue": "active",
        "operator": { "type": "string", "operation": "equals" }
      }],
      "combinator": "and"
    }
  }
}
```

### Aggregate Node
**Type:** `n8n-nodes-base.aggregate` | **Version:** 1

```json
{
  "type": "n8n-nodes-base.aggregate",
  "typeVersion": 1,
  "parameters": {
    "aggregate": "aggregateAllItemData",
    "destinationFieldName": "data"
  }
}
```

### Sort Node
**Type:** `n8n-nodes-base.sort` | **Version:** 1

```json
{
  "type": "n8n-nodes-base.sort",
  "typeVersion": 1,
  "parameters": {
    "sortFieldsUi": {
      "sortField": [
        { "fieldName": "createdAt", "order": "descending" }
      ]
    }
  }
}
```

---

## Triggers
<!-- chunk: 04-triggers | keywords: webhook, schedule, cron, manual -->

### Manual Trigger
**Type:** `n8n-nodes-base.manualTrigger` | **Version:** 1

```json
{
  "type": "n8n-nodes-base.manualTrigger",
  "typeVersion": 1,
  "parameters": {}
}
```

### Webhook Trigger
**Type:** `n8n-nodes-base.webhook` | **Version:** 2.1

```json
{
  "type": "n8n-nodes-base.webhook",
  "typeVersion": 2.1,
  "webhookId": "unique-webhook-id",
  "parameters": {
    "httpMethod": "POST",
    "path": "my-webhook",
    "authentication": "none",
    "responseMode": "onReceived"
  }
}
```

**HTTP Methods:** `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`
**Auth Options:** `none`, `basicAuth`, `headerAuth`, `jwtAuth`
**Response Modes:** `onReceived`, `lastNode`, `responseNode`

### Schedule Trigger
**Type:** `n8n-nodes-base.scheduleTrigger` | **Version:** 1.2

```json
{
  "type": "n8n-nodes-base.scheduleTrigger",
  "typeVersion": 1.2,
  "parameters": {
    "rule": {
      "interval": [{
        "field": "cronExpression",
        "expression": "0 9 * * 1-5"
      }]
    }
  }
}
```

**Cron Patterns:**
```
0 9 * * 1-5      # Weekdays 9 AM
0 */4 * * *      # Every 4 hours
0 0 1 * *        # First of month
*/15 * * * *     # Every 15 minutes
```

### Chat Trigger (AI)
**Type:** `@n8n/n8n-nodes-langchain.chatTrigger` | **Version:** 1.1

```json
{
  "type": "@n8n/n8n-nodes-langchain.chatTrigger",
  "typeVersion": 1.1,
  "webhookId": "chat-uuid",
  "parameters": {
    "mode": "hostedChat",
    "options": {
      "title": "AI Assistant",
      "inputPlaceholder": "Type message..."
    }
  }
}
```

---

## Communication
<!-- chunk: 04-communication | keywords: slack, email, discord, telegram -->

### Slack
**Type:** `n8n-nodes-base.slack` | **Version:** 2.4

```json
{
  "type": "n8n-nodes-base.slack",
  "typeVersion": 2.4,
  "parameters": {
    "resource": "message",
    "operation": "post",
    "channel": { "mode": "id", "value": "C0123456789" },
    "text": "Hello from n8n!"
  },
  "credentials": {
    "slackOAuth2Api": { "id": "1", "name": "Slack" }
  }
}
```

**Resources:** `message`, `channel`, `reaction`, `star`, `file`, `user`, `userGroup`

### Email Send
**Type:** `n8n-nodes-base.emailSend` | **Version:** 2.1

```json
{
  "type": "n8n-nodes-base.emailSend",
  "typeVersion": 2.1,
  "parameters": {
    "fromEmail": "sender@example.com",
    "toEmail": "recipient@example.com",
    "subject": "Subject",
    "emailFormat": "html",
    "html": "<h1>Hello</h1>"
  },
  "credentials": { "smtp": { "id": "1", "name": "SMTP" } }
}
```

### Discord
**Type:** `n8n-nodes-base.discord` | **Version:** 2

```json
{
  "type": "n8n-nodes-base.discord",
  "typeVersion": 2,
  "parameters": {
    "resource": "message",
    "operation": "send",
    "content": "Hello from n8n!"
  }
}
```

---

## Project Management
<!-- chunk: 04-pm | keywords: jira, linear, asana, trello -->

### Jira
**Type:** `n8n-nodes-base.jira` | **Version:** 1

```json
{
  "type": "n8n-nodes-base.jira",
  "typeVersion": 1,
  "parameters": {
    "resource": "issue",
    "operation": "create",
    "project": "PROJ",
    "issueType": "Task",
    "summary": "Issue title"
  },
  "credentials": { "jiraSoftwareCloudApi": { "id": "1", "name": "Jira" } }
}
```

**Resources:** `issue`, `issueAttachment`, `issueComment`, `user`

### Linear
**Type:** `n8n-nodes-base.linear` | **Version:** 1.1

```json
{
  "type": "n8n-nodes-base.linear",
  "typeVersion": 1.1,
  "parameters": {
    "resource": "issue",
    "operation": "create",
    "teamId": "team-uuid",
    "title": "Issue title"
  },
  "credentials": { "linearApi": { "id": "1", "name": "Linear" } }
}
```

---

## CRM Sales
<!-- chunk: 04-crm | keywords: salesforce, hubspot, pipedrive -->

### Salesforce
**Type:** `n8n-nodes-base.salesforce` | **Version:** 1

```json
{
  "type": "n8n-nodes-base.salesforce",
  "typeVersion": 1,
  "parameters": {
    "resource": "contact",
    "operation": "create",
    "lastname": "Smith",
    "additionalFields": {
      "firstName": "John",
      "email": "john@example.com"
    }
  },
  "credentials": { "salesforceOAuth2Api": { "id": "1", "name": "Salesforce" } }
}
```

**Resources:** `account`, `attachment`, `case`, `contact`, `lead`, `opportunity`, `task`

### HubSpot
**Type:** `n8n-nodes-base.hubspot` | **Version:** 2.1

```json
{
  "type": "n8n-nodes-base.hubspot",
  "typeVersion": 2.1,
  "parameters": {
    "resource": "contact",
    "operation": "create",
    "additionalFields": {
      "email": "contact@example.com",
      "firstName": "John"
    }
  },
  "credentials": { "hubspotOAuth2Api": { "id": "1", "name": "HubSpot" } }
}
```

---

## Database
<!-- chunk: 04-database | keywords: postgres, mysql, mongodb, redis -->

### PostgreSQL
**Type:** `n8n-nodes-base.postgres` | **Version:** 2.5

```json
{
  "type": "n8n-nodes-base.postgres",
  "typeVersion": 2.5,
  "parameters": {
    "operation": "executeQuery",
    "query": "SELECT * FROM users WHERE status = $1",
    "options": { "queryParams": "active" }
  },
  "credentials": { "postgres": { "id": "1", "name": "PostgreSQL" } }
}
```

**Operations:** `executeQuery`, `insert`, `update`, `upsert`, `delete`, `select`

### MongoDB
**Type:** `n8n-nodes-base.mongoDb` | **Version:** 1.2

```json
{
  "type": "n8n-nodes-base.mongoDb",
  "typeVersion": 1.2,
  "parameters": {
    "operation": "find",
    "collection": "users",
    "query": "{ \"status\": \"active\" }"
  },
  "credentials": { "mongoDb": { "id": "1", "name": "MongoDB" } }
}
```

**Operations:** `aggregate`, `delete`, `find`, `insert`, `update`

### Redis
**Type:** `n8n-nodes-base.redis` | **Version:** 1

```json
{
  "type": "n8n-nodes-base.redis",
  "typeVersion": 1,
  "parameters": {
    "operation": "get",
    "key": "myKey"
  },
  "credentials": { "redis": { "id": "1", "name": "Redis" } }
}
```

**Operations:** `delete`, `get`, `incr`, `keys`, `pop`, `publish`, `push`, `set`

---

## File Storage
<!-- chunk: 04-storage | keywords: sheets, s3, drive, dropbox -->

### Google Sheets
**Type:** `n8n-nodes-base.googleSheets` | **Version:** 4.5

```json
{
  "type": "n8n-nodes-base.googleSheets",
  "typeVersion": 4.5,
  "parameters": {
    "resource": "sheet",
    "operation": "appendOrUpdate",
    "documentId": { "mode": "id", "value": "spreadsheet-id" },
    "sheetName": { "mode": "name", "value": "Sheet1" },
    "columns": {
      "mappingMode": "defineBelow",
      "value": {
        "Name": "={{ $json.name }}",
        "Email": "={{ $json.email }}"
      }
    }
  },
  "credentials": { "googleSheetsOAuth2Api": { "id": "1", "name": "Google Sheets" } }
}
```

**Operations:** `append`, `appendOrUpdate`, `clear`, `create`, `delete`, `read`, `update`

### AWS S3
**Type:** `n8n-nodes-base.awsS3` | **Version:** 2

```json
{
  "type": "n8n-nodes-base.awsS3",
  "typeVersion": 2,
  "parameters": {
    "resource": "file",
    "operation": "upload",
    "bucketName": "my-bucket",
    "fileName": "path/to/file.json",
    "binaryData": true
  },
  "credentials": { "aws": { "id": "1", "name": "AWS" } }
}
```

---

## AI Language Models
<!-- chunk: 04-llm | keywords: openai, anthropic, gemini, ollama -->

### OpenAI Chat Model
**Type:** `@n8n/n8n-nodes-langchain.lmChatOpenAi` | **Version:** 1.2

```json
{
  "type": "@n8n/n8n-nodes-langchain.lmChatOpenAi",
  "typeVersion": 1.2,
  "parameters": {
    "model": "gpt-4o",
    "options": {
      "temperature": 0.7,
      "maxTokens": 2048
    }
  },
  "credentials": { "openAiApi": { "id": "1", "name": "OpenAI" } }
}
```

**Models:** `gpt-4o`, `gpt-4o-mini`, `gpt-4-turbo`, `gpt-4`, `gpt-3.5-turbo`, `o1-preview`, `o1-mini`

### Anthropic Chat Model
**Type:** `@n8n/n8n-nodes-langchain.lmChatAnthropic`

```json
{
  "type": "@n8n/n8n-nodes-langchain.lmChatAnthropic",
  "parameters": {
    "model": "claude-sonnet-4-20250514",
    "options": { "temperature": 0.7, "maxTokensToSample": 4096 }
  }
}
```

**Models:** `claude-sonnet-4-20250514`, `claude-3-5-sonnet-20241022`, `claude-3-opus-20240229`, `claude-3-haiku-20240307`

### Google Gemini
**Type:** `@n8n/n8n-nodes-langchain.lmChatGoogleGemini`

```json
{
  "type": "@n8n/n8n-nodes-langchain.lmChatGoogleGemini",
  "parameters": {
    "model": "gemini-1.5-pro",
    "options": { "temperature": 0.7 }
  }
}
```

### Ollama (Local)
**Type:** `@n8n/n8n-nodes-langchain.lmChatOllama`

```json
{
  "type": "@n8n/n8n-nodes-langchain.lmChatOllama",
  "parameters": {
    "model": "llama3.2",
    "baseUrl": "http://localhost:11434",
    "options": { "temperature": 0.7 }
  }
}
```

### Other LLM Providers
| Type | Provider |
|------|----------|
| `lmChatAzureOpenAi` | Azure OpenAI |
| `lmChatGroq` | Groq |
| `lmChatMistral` | Mistral AI |
| `lmChatCohere` | Cohere |
| `lmChatAwsBedrock` | AWS Bedrock |
| `lmChatOpenRouter` | OpenRouter |
| `lmChatDeepSeek` | DeepSeek |
| `lmChatXAiGrok` | xAI Grok |

---

## AI Agents
<!-- chunk: 04-agents | keywords: agent, tools, reasoning -->

### AI Agent
**Type:** `@n8n/n8n-nodes-langchain.agent` | **Version:** 1.7

```json
{
  "type": "@n8n/n8n-nodes-langchain.agent",
  "typeVersion": 1.7,
  "parameters": {
    "agent": "conversationalAgent",
    "text": "={{ $json.chatInput }}",
    "options": {
      "systemMessage": "You are a helpful assistant.",
      "maxIterations": 10
    }
  }
}
```

**Agent Types:**
| Type | Description |
|------|-------------|
| `conversationalAgent` | Chat with memory and tools |
| `openAiFunctionsAgent` | OpenAI function calling |
| `reActAgent` | Reasoning and Acting |
| `toolsAgent` | Generic tools agent |

**Inputs:** `main` (required), `ai_languageModel` (required), `ai_memory`, `ai_tool` (multiple), `ai_outputParser`

### OpenAI Assistant
**Type:** `@n8n/n8n-nodes-langchain.openAiAssistant`

```json
{
  "type": "@n8n/n8n-nodes-langchain.openAiAssistant",
  "parameters": { "assistantId": "asst_xxx" }
}
```

---

## AI Memory
<!-- chunk: 04-memory | keywords: memory, conversation, history -->

### Buffer Window Memory
**Type:** `@n8n/n8n-nodes-langchain.memoryBufferWindow` | **Version:** 1.3

```json
{
  "type": "@n8n/n8n-nodes-langchain.memoryBufferWindow",
  "typeVersion": 1.3,
  "parameters": {
    "sessionIdType": "fromInput",
    "sessionKey": "={{ $json.sessionId }}",
    "contextWindowLength": 10
  }
}
```

### Redis Memory
**Type:** `@n8n/n8n-nodes-langchain.memoryRedis`

```json
{
  "type": "@n8n/n8n-nodes-langchain.memoryRedis",
  "parameters": {
    "sessionKey": "sessionId",
    "sessionTTL": 3600
  },
  "credentials": { "redis": { "id": "1", "name": "Redis" } }
}
```

### Other Memory Types
- `memoryPostgres` - PostgreSQL Memory
- `memoryMongodb` - MongoDB Memory
- `memoryZep` - Zep Memory
- `memoryXata` - Xata Memory

---

## AI Tools
<!-- chunk: 04-tools | keywords: tools, functions, capabilities -->

### Calculator Tool
**Type:** `@n8n/n8n-nodes-langchain.toolCalculator`

```json
{
  "type": "@n8n/n8n-nodes-langchain.toolCalculator",
  "parameters": {}
}
```

### Code Tool
**Type:** `@n8n/n8n-nodes-langchain.toolCode`

```json
{
  "type": "@n8n/n8n-nodes-langchain.toolCode",
  "parameters": {
    "name": "processor",
    "description": "Process data",
    "language": "javaScript",
    "jsCode": "return { result: query.toUpperCase() };"
  }
}
```

### HTTP Request Tool
**Type:** `@n8n/n8n-nodes-langchain.toolHttpRequest`

```json
{
  "type": "@n8n/n8n-nodes-langchain.toolHttpRequest",
  "parameters": {
    "name": "api_call",
    "description": "Call API",
    "method": "GET",
    "url": "https://api.example.com/data"
  }
}
```

### Workflow Tool
**Type:** `@n8n/n8n-nodes-langchain.toolWorkflow`

```json
{
  "type": "@n8n/n8n-nodes-langchain.toolWorkflow",
  "parameters": {
    "name": "order_processor",
    "description": "Process an order",
    "workflowId": "workflow-uuid"
  }
}
```

### Vector Store Tool
**Type:** `@n8n/n8n-nodes-langchain.toolVectorStore`

```json
{
  "type": "@n8n/n8n-nodes-langchain.toolVectorStore",
  "parameters": {
    "name": "knowledge_search",
    "description": "Search knowledge base",
    "topK": 5
  }
}
```

### Other Tools
- `toolWikipedia` - Wikipedia search
- `toolSerpApi` - Web search
- `toolWolframAlpha` - Computation

---

## AI Vector Stores
<!-- chunk: 04-vectors | keywords: vector, store, embeddings -->

### Pinecone
**Type:** `@n8n/n8n-nodes-langchain.vectorStorePinecone`

```json
{
  "type": "@n8n/n8n-nodes-langchain.vectorStorePinecone",
  "parameters": {
    "mode": "retrieve",
    "pineconeIndex": "my-index",
    "options": { "topK": 5 }
  },
  "credentials": { "pineconeApi": { "id": "1", "name": "Pinecone" } }
}
```

**Modes:** `retrieve`, `insert`, `load`

### Qdrant
**Type:** `@n8n/n8n-nodes-langchain.vectorStoreQdrant`

```json
{
  "type": "@n8n/n8n-nodes-langchain.vectorStoreQdrant",
  "parameters": {
    "mode": "retrieve",
    "qdrantCollection": "collection",
    "options": { "topK": 5 }
  }
}
```

### Other Vector Stores
| Type | Provider |
|------|----------|
| `vectorStoreSupabase` | Supabase |
| `vectorStoreMongoDBAtlas` | MongoDB Atlas |
| `vectorStoreMilvus` | Milvus |
| `vectorStoreWeaviate` | Weaviate |
| `vectorStorePGVector` | PGVector |
| `vectorStoreRedis` | Redis |
| `vectorStoreInMemory` | In-Memory |
| `vectorStoreAzureAISearch` | Azure AI Search |

---

## AI Embeddings
<!-- chunk: 04-embeddings | keywords: embeddings, models -->

### OpenAI Embeddings
**Type:** `@n8n/n8n-nodes-langchain.embeddingsOpenAi`

```json
{
  "type": "@n8n/n8n-nodes-langchain.embeddingsOpenAi",
  "parameters": {
    "model": "text-embedding-3-small",
    "options": { "batchSize": 512 }
  }
}
```

**Models:** `text-embedding-3-small`, `text-embedding-3-large`, `text-embedding-ada-002`

### Other Embedding Providers
- `embeddingsAzureOpenAi` - Azure OpenAI
- `embeddingsCohere` - Cohere
- `embeddingsGoogle` - Google
- `embeddingsOllama` - Ollama (local)
- `embeddingsMistral` - Mistral
- `embeddingsAwsBedrock` - AWS Bedrock

---

## AI Document Loaders
<!-- chunk: 04-loaders | keywords: document, loader, input -->

### Default Data Loader
**Type:** `@n8n/n8n-nodes-langchain.documentDefaultDataLoader`

```json
{
  "type": "@n8n/n8n-nodes-langchain.documentDefaultDataLoader",
  "parameters": {
    "dataType": "json",
    "jsonData": "={{ $json }}"
  }
}
```

### Binary Document Loader
**Type:** `@n8n/n8n-nodes-langchain.documentBinaryInputLoader`

```json
{
  "type": "@n8n/n8n-nodes-langchain.documentBinaryInputLoader",
  "parameters": {
    "binaryDataKey": "data",
    "loader": "auto"
  }
}
```

**Loaders:** `auto`, `csvLoader`, `docxLoader`, `epubLoader`, `jsonLoader`, `pdfLoader`, `textLoader`

### Text Splitters
| Type | Description |
|------|-------------|
| `textSplitterRecursiveCharacterTextSplitter` | Recursive splitting |
| `textSplitterCharacterTextSplitter` | Character-based |
| `textSplitterTokenSplitter` | Token-based |

```json
{
  "type": "@n8n/n8n-nodes-langchain.textSplitterRecursiveCharacterTextSplitter",
  "parameters": {
    "chunkSize": 1000,
    "chunkOverlap": 200
  }
}
```

---

## AI Chains
<!-- chunk: 04-chains | keywords: chain, llm, qa -->

### Basic LLM Chain
**Type:** `@n8n/n8n-nodes-langchain.chainLlm`

```json
{
  "type": "@n8n/n8n-nodes-langchain.chainLlm",
  "parameters": {
    "prompt": "Summarize: {{ $json.text }}"
  }
}
```

### Summarization Chain
**Type:** `@n8n/n8n-nodes-langchain.chainSummarization`

```json
{
  "type": "@n8n/n8n-nodes-langchain.chainSummarization",
  "parameters": { "type": "map_reduce" }
}
```

**Types:** `stuff`, `map_reduce`, `refine`

### Retrieval QA Chain
**Type:** `@n8n/n8n-nodes-langchain.chainRetrievalQa`

```json
{
  "type": "@n8n/n8n-nodes-langchain.chainRetrievalQa",
  "parameters": { "query": "={{ $json.question }}" }
}
```

### Text Classifier
**Type:** `@n8n/n8n-nodes-langchain.textClassifier`

```json
{
  "type": "@n8n/n8n-nodes-langchain.textClassifier",
  "parameters": {
    "categories": "positive,negative,neutral",
    "inputText": "={{ $json.text }}"
  }
}
```

### Information Extractor
**Type:** `@n8n/n8n-nodes-langchain.informationExtractor`

```json
{
  "type": "@n8n/n8n-nodes-langchain.informationExtractor",
  "parameters": {
    "text": "={{ $json.text }}",
    "attributes": {
      "attributes": [
        { "name": "name", "description": "Person's name", "type": "string" },
        { "name": "email", "description": "Email address", "type": "string" }
      ]
    }
  }
}
```

---

## Utility Nodes
<!-- chunk: 04-utility | keywords: http, request, respond -->

### HTTP Request
**Type:** `n8n-nodes-base.httpRequest` | **Version:** 4.2

```json
{
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.2,
  "parameters": {
    "method": "POST",
    "url": "https://api.example.com/endpoint",
    "authentication": "genericCredentialType",
    "sendBody": true,
    "bodyParameters": {
      "parameters": [{ "name": "key", "value": "={{ $json.data }}" }]
    }
  }
}
```

**Auth Options:** `none`, `predefinedCredentialType`, `genericCredentialType`
**Body Types:** `json`, `form-urlencoded`, `multipart-form-data`, `raw`

### Respond to Webhook
**Type:** `n8n-nodes-base.respondToWebhook` | **Version:** 1.1

```json
{
  "type": "n8n-nodes-base.respondToWebhook",
  "typeVersion": 1.1,
  "parameters": {
    "respondWith": "json",
    "responseBody": "={{ $json }}",
    "options": { "responseCode": 200 }
  }
}
```

---

## Node JSON Structure
<!-- chunk: 04-structure | keywords: structure, format, template -->

### Complete Node Definition
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
  "webhookId": "optional-webhook-uuid",
  "disabled": false,
  "continueOnFail": false,
  "retryOnFail": false,
  "maxTries": 3
}
```

### Position Guidelines
- Start: `[250, 300]`
- Horizontal spacing: +250px
- Vertical branches: +/-150px
- Keep Y: 0-600

---

## Statistics Summary

| Category | Count | Examples |
|----------|-------|----------|
| Core Flow | 29 | If, Switch, Merge, Split, Wait |
| Transform | 132 | Set, Code, Filter, Aggregate |
| Triggers | 112 | Webhook, Schedule, Manual |
| Communication | 11 | Slack, Email, Discord |
| Database | 6 | PostgreSQL, MongoDB, Redis |
| File Storage | 18 | Google Sheets, S3, Drive |
| AI/LangChain | 118 | Agent, OpenAI, Memory, Tools |
| CRM/PM | 16 | Salesforce, Jira, Linear |

---

*Source: packages/nodes-base/nodes/, packages/@n8n/nodes-langchain/nodes/*
