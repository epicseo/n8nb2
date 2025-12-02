# 14_WORKFLOW_TEMPLATES.md
<!-- repo: n8n | version: 1.122.0 | generated: 2025-12-02 -->
<!-- tags: templates, workflows, patterns, examples, automation -->

## Contents
- [Trigger Patterns](#trigger-patterns)
- [Data Processing Patterns](#data-processing-patterns)
- [Integration Patterns](#integration-patterns)
- [AI Workflow Patterns](#ai-workflow-patterns)
- [Error Handling Patterns](#error-handling-patterns)
- [Complete Workflow Examples](#complete-workflow-examples)

---

## Trigger Patterns
<!-- chunk: 14-triggers | keywords: trigger, webhook, schedule, polling | source: analysis -->

### Manual Trigger (Testing)
```json
{
  "id": "manual-1",
  "name": "Manual Trigger",
  "type": "n8n-nodes-base.manualTrigger",
  "typeVersion": 1,
  "position": [0, 0],
  "parameters": {}
}
```

### Webhook Trigger (HTTP Endpoint)
```json
{
  "id": "webhook-1",
  "name": "Webhook",
  "type": "n8n-nodes-base.webhook",
  "typeVersion": 2,
  "position": [0, 0],
  "parameters": {
    "path": "my-endpoint",
    "httpMethod": "POST",
    "responseMode": "onReceived",
    "responseCode": 200,
    "options": {}
  },
  "webhookId": "unique-webhook-id"
}
```

### Schedule Trigger (Cron)
```json
{
  "id": "schedule-1",
  "name": "Schedule Trigger",
  "type": "n8n-nodes-base.scheduleTrigger",
  "typeVersion": 1.2,
  "position": [0, 0],
  "parameters": {
    "rule": {
      "interval": [
        {
          "field": "cronExpression",
          "expression": "0 9 * * 1-5"
        }
      ]
    }
  }
}
```
Common cron expressions:
- `0 * * * *` - Every hour
- `0 9 * * *` - Daily at 9 AM
- `0 9 * * 1-5` - Weekdays at 9 AM
- `0 0 * * 0` - Weekly on Sunday
- `0 0 1 * *` - Monthly on 1st

### Polling Trigger (Check for Updates)
```json
{
  "id": "poll-1",
  "name": "Email Trigger (IMAP)",
  "type": "n8n-nodes-base.emailReadImap",
  "typeVersion": 2.1,
  "position": [0, 0],
  "parameters": {
    "mailbox": "INBOX",
    "options": {
      "markAsRead": true
    }
  },
  "credentials": {
    "imap": { "id": "imap-cred", "name": "Email IMAP" }
  }
}
```

---

## Data Processing Patterns
<!-- chunk: 14-data-processing | keywords: transform, filter, merge, split | source: analysis -->

### Transform Data (Set Node)
```json
{
  "id": "set-1",
  "name": "Transform Data",
  "type": "n8n-nodes-base.set",
  "typeVersion": 3.4,
  "position": [220, 0],
  "parameters": {
    "mode": "manual",
    "duplicateItem": false,
    "assignments": {
      "assignments": [
        {
          "id": "a1",
          "name": "fullName",
          "value": "={{ $json.firstName }} {{ $json.lastName }}",
          "type": "string"
        },
        {
          "id": "a2",
          "name": "processedAt",
          "value": "={{ $now.toISO() }}",
          "type": "string"
        }
      ]
    },
    "includeOtherFields": true
  }
}
```

### Filter Items
```json
{
  "id": "filter-1",
  "name": "Filter Active",
  "type": "n8n-nodes-base.filter",
  "typeVersion": 2,
  "position": [220, 0],
  "parameters": {
    "conditions": {
      "options": {
        "caseSensitive": true,
        "typeValidation": "strict"
      },
      "conditions": [
        {
          "id": "c1",
          "leftValue": "={{ $json.status }}",
          "rightValue": "active",
          "operator": {
            "type": "string",
            "operation": "equals"
          }
        }
      ],
      "combinator": "and"
    }
  }
}
```

### Conditional Branching (If)
```json
{
  "id": "if-1",
  "name": "Check Condition",
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
          "id": "c1",
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
}
```
**Outputs:** `[0]` = true branch, `[1]` = false branch

### Multi-way Branching (Switch)
```json
{
  "id": "switch-1",
  "name": "Route by Type",
  "type": "n8n-nodes-base.switch",
  "typeVersion": 3.2,
  "position": [220, 0],
  "parameters": {
    "mode": "rules",
    "rules": {
      "values": [
        {
          "outputKey": "order",
          "conditions": {
            "conditions": [
              {
                "leftValue": "={{ $json.type }}",
                "rightValue": "order",
                "operator": { "type": "string", "operation": "equals" }
              }
            ]
          }
        },
        {
          "outputKey": "refund",
          "conditions": {
            "conditions": [
              {
                "leftValue": "={{ $json.type }}",
                "rightValue": "refund",
                "operator": { "type": "string", "operation": "equals" }
              }
            ]
          }
        }
      ]
    },
    "options": {
      "fallbackOutput": "extra"
    }
  }
}
```

### Merge Multiple Inputs
```json
{
  "id": "merge-1",
  "name": "Merge Data",
  "type": "n8n-nodes-base.merge",
  "typeVersion": 3,
  "position": [440, 0],
  "parameters": {
    "mode": "combine",
    "combinationMode": "mergeByPosition",
    "options": {}
  }
}
```

### Split Array to Items
```json
{
  "id": "split-1",
  "name": "Split Out",
  "type": "n8n-nodes-base.splitOut",
  "typeVersion": 1,
  "position": [220, 0],
  "parameters": {
    "fieldToSplitOut": "items",
    "options": {}
  }
}
```

### Aggregate Items to Array
```json
{
  "id": "agg-1",
  "name": "Aggregate",
  "type": "n8n-nodes-base.aggregate",
  "typeVersion": 1,
  "position": [440, 0],
  "parameters": {
    "aggregate": "aggregateAllItemData",
    "options": {}
  }
}
```

### Loop Over Items
```json
{
  "id": "loop-1",
  "name": "Loop Over Items",
  "type": "n8n-nodes-base.splitInBatches",
  "typeVersion": 3,
  "position": [220, 0],
  "parameters": {
    "batchSize": 1,
    "options": {}
  }
}
```

---

## Integration Patterns
<!-- chunk: 14-integrations | keywords: http, api, database, email | source: analysis -->

### HTTP GET Request
```json
{
  "id": "http-1",
  "name": "Fetch Data",
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.2,
  "position": [220, 0],
  "parameters": {
    "url": "https://api.example.com/data",
    "method": "GET",
    "authentication": "none",
    "options": {}
  }
}
```

### HTTP POST with JSON Body
```json
{
  "id": "http-2",
  "name": "Send Data",
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.2,
  "position": [220, 0],
  "parameters": {
    "url": "https://api.example.com/webhook",
    "method": "POST",
    "authentication": "none",
    "sendBody": true,
    "bodyParameters": {
      "parameters": [
        { "name": "event", "value": "={{ $json.event }}" },
        { "name": "data", "value": "={{ $json }}" }
      ]
    },
    "options": {}
  }
}
```

### HTTP with OAuth2
```json
{
  "id": "http-3",
  "name": "OAuth Request",
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.2,
  "position": [220, 0],
  "parameters": {
    "url": "https://api.service.com/resource",
    "method": "GET",
    "authentication": "predefinedCredentialType",
    "nodeCredentialType": "oAuth2Api",
    "options": {}
  },
  "credentials": {
    "oAuth2Api": { "id": "oauth-cred", "name": "Service OAuth2" }
  }
}
```

### Database Query (Postgres)
```json
{
  "id": "db-1",
  "name": "Query Database",
  "type": "n8n-nodes-base.postgres",
  "typeVersion": 2.5,
  "position": [220, 0],
  "parameters": {
    "operation": "executeQuery",
    "query": "SELECT * FROM users WHERE status = $1",
    "options": {
      "queryParams": "active"
    }
  },
  "credentials": {
    "postgres": { "id": "pg-cred", "name": "Production DB" }
  }
}
```

### Send Email (SMTP)
```json
{
  "id": "email-1",
  "name": "Send Email",
  "type": "n8n-nodes-base.emailSend",
  "typeVersion": 2.1,
  "position": [440, 0],
  "parameters": {
    "fromEmail": "noreply@company.com",
    "toEmail": "={{ $json.email }}",
    "subject": "Your Order Confirmation",
    "emailType": "html",
    "html": "<h1>Thank you!</h1><p>Order #{{ $json.orderId }}</p>",
    "options": {}
  },
  "credentials": {
    "smtp": { "id": "smtp-cred", "name": "Company SMTP" }
  }
}
```

### Slack Message
```json
{
  "id": "slack-1",
  "name": "Send Slack",
  "type": "n8n-nodes-base.slack",
  "typeVersion": 2.2,
  "position": [440, 0],
  "parameters": {
    "resource": "message",
    "operation": "send",
    "channel": { "__rl": true, "mode": "id", "value": "C0123456789" },
    "text": "New order received: {{ $json.orderId }}",
    "options": {}
  },
  "credentials": {
    "slackApi": { "id": "slack-cred", "name": "Slack Bot" }
  }
}
```

### Google Sheets Append
```json
{
  "id": "sheets-1",
  "name": "Log to Sheets",
  "type": "n8n-nodes-base.googleSheets",
  "typeVersion": 4.5,
  "position": [440, 0],
  "parameters": {
    "resource": "sheet",
    "operation": "append",
    "documentId": {
      "__rl": true,
      "mode": "list",
      "value": "spreadsheet-id-here"
    },
    "sheetName": {
      "__rl": true,
      "mode": "list",
      "value": "Sheet1"
    },
    "columns": {
      "mappingMode": "autoMapInputData",
      "value": {}
    },
    "options": {}
  },
  "credentials": {
    "googleSheetsOAuth2Api": { "id": "gsheets-cred", "name": "Google Sheets" }
  }
}
```

---

## AI Workflow Patterns
<!-- chunk: 14-ai | keywords: ai, agent, langchain, llm, chat | source: analysis -->

### Basic Chat Completion
```json
{
  "nodes": [
    {
      "id": "chat-1",
      "name": "Chat Trigger",
      "type": "@n8n/n8n-nodes-langchain.chatTrigger",
      "typeVersion": 1.1,
      "position": [0, 0],
      "parameters": { "options": {} },
      "webhookId": "chat-webhook"
    },
    {
      "id": "llm-1",
      "name": "OpenAI",
      "type": "@n8n/n8n-nodes-langchain.lmChatOpenAi",
      "typeVersion": 1.2,
      "position": [100, 200],
      "parameters": {
        "model": "gpt-4o",
        "options": { "temperature": 0.7 }
      },
      "credentials": {
        "openAiApi": { "id": "openai-1", "name": "OpenAI" }
      }
    },
    {
      "id": "chain-1",
      "name": "Basic LLM Chain",
      "type": "@n8n/n8n-nodes-langchain.chainLlm",
      "typeVersion": 1.4,
      "position": [300, 0],
      "parameters": {
        "prompt": "={{ $json.chatInput }}"
      }
    }
  ],
  "connections": {
    "Chat Trigger": {
      "main": [[{ "node": "Basic LLM Chain", "type": "main", "index": 0 }]]
    },
    "OpenAI": {
      "ai_languageModel": [[{ "node": "Basic LLM Chain", "type": "ai_languageModel", "index": 0 }]]
    }
  }
}
```

### AI Agent with Tools
```json
{
  "nodes": [
    {
      "id": "trigger-1",
      "name": "Chat Trigger",
      "type": "@n8n/n8n-nodes-langchain.chatTrigger",
      "typeVersion": 1.1,
      "position": [0, 0],
      "parameters": {},
      "webhookId": "agent-chat"
    },
    {
      "id": "agent-1",
      "name": "AI Agent",
      "type": "@n8n/n8n-nodes-langchain.agent",
      "typeVersion": 1.7,
      "position": [400, 0],
      "parameters": {
        "options": {
          "systemMessage": "You are a helpful assistant. Use the tools available to help answer questions."
        }
      }
    },
    {
      "id": "llm-1",
      "name": "OpenAI GPT-4",
      "type": "@n8n/n8n-nodes-langchain.lmChatOpenAi",
      "typeVersion": 1.2,
      "position": [100, 200],
      "parameters": {
        "model": "gpt-4o",
        "options": {}
      },
      "credentials": {
        "openAiApi": { "id": "openai-1", "name": "OpenAI API" }
      }
    },
    {
      "id": "memory-1",
      "name": "Buffer Memory",
      "type": "@n8n/n8n-nodes-langchain.memoryBufferWindow",
      "typeVersion": 1.3,
      "position": [100, 350],
      "parameters": {
        "sessionIdType": "fromInput",
        "sessionKey": "={{ $json.sessionId }}",
        "contextWindowLength": 10
      }
    },
    {
      "id": "tool-1",
      "name": "Calculator",
      "type": "@n8n/n8n-nodes-langchain.toolCalculator",
      "typeVersion": 1,
      "position": [100, 500],
      "parameters": {}
    },
    {
      "id": "tool-2",
      "name": "HTTP Tool",
      "type": "@n8n/n8n-nodes-langchain.toolHttpRequest",
      "typeVersion": 1.1,
      "position": [250, 500],
      "parameters": {
        "url": "https://api.example.com/search",
        "method": "GET",
        "description": "Search the knowledge base"
      }
    }
  ],
  "connections": {
    "Chat Trigger": {
      "main": [[{ "node": "AI Agent", "type": "main", "index": 0 }]]
    },
    "OpenAI GPT-4": {
      "ai_languageModel": [[{ "node": "AI Agent", "type": "ai_languageModel", "index": 0 }]]
    },
    "Buffer Memory": {
      "ai_memory": [[{ "node": "AI Agent", "type": "ai_memory", "index": 0 }]]
    },
    "Calculator": {
      "ai_tool": [[{ "node": "AI Agent", "type": "ai_tool", "index": 0 }]]
    },
    "HTTP Tool": {
      "ai_tool": [[{ "node": "AI Agent", "type": "ai_tool", "index": 0 }]]
    }
  }
}
```

### RAG Pipeline (Retrieval Augmented Generation)
```json
{
  "nodes": [
    {
      "id": "trigger-1",
      "name": "Chat Trigger",
      "type": "@n8n/n8n-nodes-langchain.chatTrigger",
      "typeVersion": 1.1,
      "position": [0, 0],
      "parameters": {},
      "webhookId": "rag-chat"
    },
    {
      "id": "qa-1",
      "name": "Question and Answer Chain",
      "type": "@n8n/n8n-nodes-langchain.chainRetrievalQa",
      "typeVersion": 1.3,
      "position": [400, 0],
      "parameters": {
        "options": {}
      }
    },
    {
      "id": "llm-1",
      "name": "OpenAI",
      "type": "@n8n/n8n-nodes-langchain.lmChatOpenAi",
      "typeVersion": 1.2,
      "position": [100, 200],
      "parameters": { "model": "gpt-4o" },
      "credentials": {
        "openAiApi": { "id": "openai-1", "name": "OpenAI" }
      }
    },
    {
      "id": "retriever-1",
      "name": "Vector Store Retriever",
      "type": "@n8n/n8n-nodes-langchain.retrieverVectorStore",
      "typeVersion": 1,
      "position": [100, 350],
      "parameters": {
        "topK": 4
      }
    },
    {
      "id": "vectorstore-1",
      "name": "Pinecone Vector Store",
      "type": "@n8n/n8n-nodes-langchain.vectorStorePinecone",
      "typeVersion": 1,
      "position": [100, 500],
      "parameters": {
        "mode": "retrieve",
        "pineconeIndex": "my-index"
      },
      "credentials": {
        "pineconeApi": { "id": "pinecone-1", "name": "Pinecone" }
      }
    },
    {
      "id": "embeddings-1",
      "name": "OpenAI Embeddings",
      "type": "@n8n/n8n-nodes-langchain.embeddingsOpenAi",
      "typeVersion": 1.1,
      "position": [100, 650],
      "parameters": {},
      "credentials": {
        "openAiApi": { "id": "openai-1", "name": "OpenAI" }
      }
    }
  ],
  "connections": {
    "Chat Trigger": {
      "main": [[{ "node": "Question and Answer Chain", "type": "main", "index": 0 }]]
    },
    "OpenAI": {
      "ai_languageModel": [[{ "node": "Question and Answer Chain", "type": "ai_languageModel", "index": 0 }]]
    },
    "Vector Store Retriever": {
      "ai_retriever": [[{ "node": "Question and Answer Chain", "type": "ai_retriever", "index": 0 }]]
    },
    "Pinecone Vector Store": {
      "ai_vectorStore": [[{ "node": "Vector Store Retriever", "type": "ai_vectorStore", "index": 0 }]]
    },
    "OpenAI Embeddings": {
      "ai_embedding": [[{ "node": "Pinecone Vector Store", "type": "ai_embedding", "index": 0 }]]
    }
  }
}
```

---

## Error Handling Patterns
<!-- chunk: 14-errors | keywords: error, handling, retry, fallback | source: analysis -->

### Node-Level Error Handling
```json
{
  "id": "risky-1",
  "name": "External API",
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.2,
  "position": [220, 0],
  "parameters": {
    "url": "https://unreliable-api.com/data"
  },
  "onError": "continueErrorOutput",
  "retryOnFail": true,
  "maxTries": 3,
  "waitBetweenTries": 1000
}
```

### Error Workflow Trigger
```json
{
  "id": "error-trigger-1",
  "name": "Error Trigger",
  "type": "n8n-nodes-base.errorTrigger",
  "typeVersion": 1,
  "position": [0, 0],
  "parameters": {}
}
```

### Try-Catch Pattern with If Node
```json
{
  "nodes": [
    {
      "id": "try-1",
      "name": "Try Operation",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.2,
      "position": [220, 0],
      "parameters": { "url": "https://api.example.com" },
      "onError": "continueErrorOutput"
    },
    {
      "id": "check-1",
      "name": "Check Error",
      "type": "n8n-nodes-base.if",
      "typeVersion": 2.2,
      "position": [440, 0],
      "parameters": {
        "conditions": {
          "conditions": [
            {
              "leftValue": "={{ $json.error }}",
              "rightValue": "",
              "operator": { "type": "string", "operation": "notEquals" }
            }
          ]
        }
      }
    },
    {
      "id": "success-1",
      "name": "Handle Success",
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [660, -100],
      "parameters": {}
    },
    {
      "id": "error-1",
      "name": "Handle Error",
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [660, 100],
      "parameters": {}
    }
  ],
  "connections": {
    "Try Operation": {
      "main": [[{ "node": "Check Error", "type": "main", "index": 0 }]]
    },
    "Check Error": {
      "main": [
        [{ "node": "Handle Error", "type": "main", "index": 0 }],
        [{ "node": "Handle Success", "type": "main", "index": 0 }]
      ]
    }
  }
}
```

---

## Complete Workflow Examples
<!-- chunk: 14-complete-examples | keywords: complete, workflow, full | source: analysis -->

### Webhook to Slack Notification
```json
{
  "name": "Webhook to Slack",
  "nodes": [
    {
      "id": "1",
      "name": "Webhook",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 2,
      "position": [0, 0],
      "parameters": {
        "path": "alert",
        "httpMethod": "POST",
        "responseMode": "onReceived"
      },
      "webhookId": "alert-webhook"
    },
    {
      "id": "2",
      "name": "Format Message",
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [220, 0],
      "parameters": {
        "mode": "manual",
        "assignments": {
          "assignments": [
            {
              "id": "a1",
              "name": "message",
              "value": "=🚨 *Alert*: {{ $json.title }}\n\n{{ $json.description }}\n\nSeverity: {{ $json.severity }}",
              "type": "string"
            }
          ]
        }
      }
    },
    {
      "id": "3",
      "name": "Send to Slack",
      "type": "n8n-nodes-base.slack",
      "typeVersion": 2.2,
      "position": [440, 0],
      "parameters": {
        "resource": "message",
        "operation": "send",
        "channel": { "__rl": true, "mode": "id", "value": "C0123456789" },
        "text": "={{ $json.message }}",
        "options": {}
      },
      "credentials": {
        "slackApi": { "id": "slack-1", "name": "Slack" }
      }
    }
  ],
  "connections": {
    "Webhook": {
      "main": [[{ "node": "Format Message", "type": "main", "index": 0 }]]
    },
    "Format Message": {
      "main": [[{ "node": "Send to Slack", "type": "main", "index": 0 }]]
    }
  },
  "active": true
}
```

### Daily Report Email
```json
{
  "name": "Daily Report",
  "nodes": [
    {
      "id": "1",
      "name": "Schedule",
      "type": "n8n-nodes-base.scheduleTrigger",
      "typeVersion": 1.2,
      "position": [0, 0],
      "parameters": {
        "rule": {
          "interval": [{ "field": "cronExpression", "expression": "0 8 * * 1-5" }]
        }
      }
    },
    {
      "id": "2",
      "name": "Fetch Metrics",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.2,
      "position": [220, 0],
      "parameters": {
        "url": "https://api.metrics.com/daily",
        "method": "GET"
      },
      "credentials": {
        "httpHeaderAuth": { "id": "api-1", "name": "Metrics API" }
      }
    },
    {
      "id": "3",
      "name": "Format Report",
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [440, 0],
      "parameters": {
        "mode": "manual",
        "assignments": {
          "assignments": [
            {
              "id": "a1",
              "name": "html",
              "value": "=<h1>Daily Report - {{ $now.toFormat('MMMM d, yyyy') }}</h1><ul><li>Users: {{ $json.users }}</li><li>Revenue: ${{ $json.revenue }}</li><li>Orders: {{ $json.orders }}</li></ul>",
              "type": "string"
            }
          ]
        }
      }
    },
    {
      "id": "4",
      "name": "Send Email",
      "type": "n8n-nodes-base.emailSend",
      "typeVersion": 2.1,
      "position": [660, 0],
      "parameters": {
        "fromEmail": "reports@company.com",
        "toEmail": "team@company.com",
        "subject": "=Daily Report - {{ $now.toFormat('yyyy-MM-dd') }}",
        "emailType": "html",
        "html": "={{ $json.html }}"
      },
      "credentials": {
        "smtp": { "id": "smtp-1", "name": "Company SMTP" }
      }
    }
  ],
  "connections": {
    "Schedule": {
      "main": [[{ "node": "Fetch Metrics", "type": "main", "index": 0 }]]
    },
    "Fetch Metrics": {
      "main": [[{ "node": "Format Report", "type": "main", "index": 0 }]]
    },
    "Format Report": {
      "main": [[{ "node": "Send Email", "type": "main", "index": 0 }]]
    }
  },
  "active": true
}
```

---

## Quick Reference
<!-- chunk: 14-quick-ref | keywords: quick, reference, patterns | source: analysis -->

### Common Node Patterns

| Pattern | Nodes Used | Purpose |
|---------|------------|---------|
| HTTP API Call | HTTP Request | External API integration |
| Data Transform | Set | Modify/add fields |
| Conditional | If | Branch based on condition |
| Multi-branch | Switch | Route to multiple paths |
| Combine Data | Merge | Join multiple inputs |
| Split Array | Split Out | Array items to separate |
| Aggregate | Aggregate | Items to single array |
| Loop | Split In Batches | Process one at a time |
| Wait | Wait | Pause execution |
| Subworkflow | Execute Workflow | Call another workflow |

### Trigger Types

| Trigger | Use Case |
|---------|----------|
| Manual Trigger | Testing, manual runs |
| Webhook | HTTP requests, API endpoints |
| Schedule Trigger | Cron jobs, recurring tasks |
| Email Trigger (IMAP) | Email-based automation |
| Chat Trigger | AI conversations |
| Error Trigger | Error handling workflows |

→ JSON Schema: [[12_WORKFLOW_JSON_SCHEMA]]
→ Expressions: [[13_EXPRESSION_GUIDE]]
→ Node Reference: [[13_NODE_REFERENCE]]
