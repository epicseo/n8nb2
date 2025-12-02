# 15_NODE_REFERENCE.md
<!-- repo: n8n | version: 1.122.0 | generated: 2025-12-02 -->
<!-- tags: nodes, reference, integrations, triggers, actions, parameters -->

## Contents
- [Overview](#overview)
- [Core Flow Control Nodes](#core-flow-control-nodes)
- [Data Transform Nodes](#data-transform-nodes)
- [Trigger Nodes](#trigger-nodes)
- [Communication Nodes](#communication-nodes)
- [Project Management Nodes](#project-management-nodes)
- [CRM Sales Nodes](#crm-sales-nodes)
- [Database Nodes](#database-nodes)
- [File Storage Nodes](#file-storage-nodes)
- [AI ML Nodes](#ai-ml-nodes)
- [Utility Nodes](#utility-nodes)
- [Node JSON Structure](#node-json-structure)

---

## Overview
<!-- chunk: 15-overview | keywords: nodes, statistics, categories -->

### Key Statistics
- **Total Nodes:** 503
- **Trigger Nodes:** 112 (22%)
- **Action Nodes:** 391 (78%)
- **Credential Types:** 389

### Distribution by Category
```
Transform Nodes:     132 (26%)
Input Nodes:         113 (22%)
Trigger Nodes:       112 (22%)
Output Nodes:        107 (21%)
Organization:          8 (2%)
Other:                31 (6%)
```

---

## Core Flow Control Nodes
<!-- chunk: 15-flow-control | keywords: if, switch, merge, split, loop, wait -->

### If Node
**Type:** `n8n-nodes-base.if`
**Version:** 2.2
**Purpose:** Route items based on conditions (true/false branches)

```json
{
  "type": "n8n-nodes-base.if",
  "typeVersion": 2.2,
  "parameters": {
    "conditions": {
      "options": {
        "caseSensitive": true,
        "leftValue": "",
        "typeValidation": "strict"
      },
      "conditions": [
        {
          "id": "condition-uuid",
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

**Operators:**
| Type | Operations |
|------|------------|
| String | equals, notEquals, contains, notContains, startsWith, endsWith, regex, notRegex |
| Number | equals, notEquals, gt, gte, lt, lte |
| Boolean | true, false |
| Array | contains, notContains, lengthEquals, lengthGt, lengthLt, empty, notEmpty |
| Object | empty, notEmpty |
| DateTime | after, before, equals |

**Outputs:** 2 branches (true at index 0, false at index 1)

---

### Switch Node
**Type:** `n8n-nodes-base.switch`
**Version:** 3.3
**Purpose:** Multi-way conditional routing

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
            "conditions": [
              {
                "leftValue": "={{ $json.priority }}",
                "rightValue": "high",
                "operator": { "type": "string", "operation": "equals" }
              }
            ]
          }
        },
        {
          "outputKey": "medium",
          "conditions": {
            "conditions": [
              {
                "leftValue": "={{ $json.priority }}",
                "rightValue": "medium",
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

**Modes:**
- `rules` - Match conditions to route
- `expression` - Use expression result as output index

**Outputs:** Dynamic (one per rule + optional fallback)

---

### Merge Node
**Type:** `n8n-nodes-base.merge`
**Version:** 3.2
**Purpose:** Combine multiple input streams

```json
{
  "type": "n8n-nodes-base.merge",
  "typeVersion": 3.2,
  "parameters": {
    "mode": "combine",
    "mergeByFields": {
      "values": [
        { "field1": "id", "field2": "userId" }
      ]
    },
    "joinMode": "keepMatches",
    "outputDataFrom": "both",
    "options": {}
  }
}
```

**Modes:**
| Mode | Description |
|------|-------------|
| `append` | Append all items from all inputs |
| `combine` | Combine by position or key field |
| `chooseBranch` | Pass through items from selected branch |
| `multiplex` | Cross-join all combinations |

**Join Modes (for combine):**
- `keepMatches` - Only matching items
- `keepNonMatches` - Only non-matching items
- `keepEverything` - All items (outer join)
- `enrichInput1` - Left join
- `enrichInput2` - Right join

---

### Split In Batches Node
**Type:** `n8n-nodes-base.splitInBatches`
**Version:** 3
**Purpose:** Process items in batches with loop support

```json
{
  "type": "n8n-nodes-base.splitInBatches",
  "typeVersion": 3,
  "parameters": {
    "batchSize": 10,
    "options": {
      "reset": false
    }
  }
}
```

**Outputs:**
- Index 0 (`done`): Emitted once when all batches complete
- Index 1 (`loop`): Emitted for each batch

---

### Wait Node
**Type:** `n8n-nodes-base.wait`
**Version:** 1.1
**Purpose:** Pause workflow execution

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

**Resume Options:**
- `timeInterval` - Wait for specified duration
- `specificTime` - Wait until specific datetime
- `webhook` - Wait for external webhook call
- `form` - Wait for form submission

**Units:** `seconds`, `minutes`, `hours`, `days`

---

### No Operation Node
**Type:** `n8n-nodes-base.noOp`
**Version:** 1
**Purpose:** Pass-through node (useful for organization)

```json
{
  "type": "n8n-nodes-base.noOp",
  "typeVersion": 1,
  "parameters": {}
}
```

---

## Data Transform Nodes
<!-- chunk: 15-transform | keywords: set, code, function, aggregate, filter -->

### Set Node (Edit Fields)
**Type:** `n8n-nodes-base.set`
**Version:** 3.4
**Purpose:** Modify, add, or remove fields from items

```json
{
  "type": "n8n-nodes-base.set",
  "typeVersion": 3.4,
  "parameters": {
    "mode": "manual",
    "duplicateItem": false,
    "assignments": {
      "assignments": [
        {
          "id": "uuid",
          "name": "fullName",
          "value": "={{ $json.firstName }} {{ $json.lastName }}",
          "type": "string"
        },
        {
          "id": "uuid2",
          "name": "processed",
          "value": true,
          "type": "boolean"
        }
      ]
    },
    "includeOtherFields": true,
    "options": {}
  }
}
```

**Modes:**
- `manual` - Define fields manually
- `raw` - Use raw JSON

**Field Types:** `string`, `number`, `boolean`, `array`, `object`

---

### Code Node
**Type:** `n8n-nodes-base.code`
**Version:** 2
**Purpose:** Execute custom JavaScript or Python code

```json
{
  "type": "n8n-nodes-base.code",
  "typeVersion": 2,
  "parameters": {
    "mode": "runOnceForAllItems",
    "language": "javaScript",
    "jsCode": "// Process all items\nconst results = [];\nfor (const item of $input.all()) {\n  results.push({\n    json: {\n      ...item.json,\n      processed: true,\n      timestamp: new Date().toISOString()\n    }\n  });\n}\nreturn results;"
  }
}
```

**Modes:**
- `runOnceForAllItems` - Code runs once with access to all items
- `runOnceForEachItem` - Code runs once per item

**Languages:** `javaScript`, `python`

**Available in JavaScript:**
```javascript
// Access input data
$input.all()           // All input items
$input.first()         // First item
$input.last()          // Last item
$input.item            // Current item (in runOnceForEachItem mode)

// Access other nodes
$('NodeName').all()    // All items from node
$('NodeName').first()  // First item from node

// Workflow data
$workflow.id           // Workflow ID
$workflow.name         // Workflow name
$execution.id          // Execution ID

// Environment
$env.MY_VAR            // Environment variable

// Return format
return [{ json: { key: 'value' } }];
```

---

### Filter Node
**Type:** `n8n-nodes-base.filter`
**Version:** 2.2
**Purpose:** Keep or remove items based on conditions

```json
{
  "type": "n8n-nodes-base.filter",
  "typeVersion": 2.2,
  "parameters": {
    "conditions": {
      "options": {
        "caseSensitive": true,
        "leftValue": ""
      },
      "conditions": [
        {
          "leftValue": "={{ $json.status }}",
          "rightValue": "active",
          "operator": { "type": "string", "operation": "equals" }
        }
      ],
      "combinator": "and"
    }
  }
}
```

---

### Aggregate Node
**Type:** `n8n-nodes-base.aggregate`
**Version:** 1
**Purpose:** Group and summarize data

```json
{
  "type": "n8n-nodes-base.aggregate",
  "typeVersion": 1,
  "parameters": {
    "aggregate": "aggregateAllItemData",
    "destinationFieldName": "data",
    "include": "allFieldsExcept",
    "fieldsToExclude": "internalId",
    "options": {}
  }
}
```

**Aggregate Options:**
- `aggregateAllItemData` - Collect all items into array
- `aggregateIndividualFields` - Aggregate specific fields

---

### Sort Node
**Type:** `n8n-nodes-base.sort`
**Version:** 1
**Purpose:** Sort items by field values

```json
{
  "type": "n8n-nodes-base.sort",
  "typeVersion": 1,
  "parameters": {
    "sortFieldsUi": {
      "sortField": [
        { "fieldName": "createdAt", "order": "descending" },
        { "fieldName": "name", "order": "ascending" }
      ]
    },
    "options": {}
  }
}
```

---

### Limit Node
**Type:** `n8n-nodes-base.limit`
**Version:** 1
**Purpose:** Limit number of items passed through

```json
{
  "type": "n8n-nodes-base.limit",
  "typeVersion": 1,
  "parameters": {
    "maxItems": 10,
    "keep": "firstItems"
  }
}
```

**Keep Options:** `firstItems`, `lastItems`

---

## Trigger Nodes
<!-- chunk: 15-triggers | keywords: webhook, cron, schedule, manual, trigger -->

### Manual Trigger
**Type:** `n8n-nodes-base.manualTrigger`
**Version:** 1
**Purpose:** Start workflow manually

```json
{
  "type": "n8n-nodes-base.manualTrigger",
  "typeVersion": 1,
  "parameters": {}
}
```

---

### Webhook Trigger
**Type:** `n8n-nodes-base.webhook`
**Version:** 2.1
**Purpose:** Listen for incoming HTTP requests

```json
{
  "type": "n8n-nodes-base.webhook",
  "typeVersion": 2.1,
  "webhookId": "unique-webhook-id",
  "parameters": {
    "httpMethod": "POST",
    "path": "my-webhook-path",
    "authentication": "none",
    "responseMode": "onReceived",
    "responseData": "allEntries",
    "options": {
      "rawBody": false,
      "responseHeaders": {}
    }
  }
}
```

**HTTP Methods:** `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`

**Authentication Options:**
- `none` - No authentication
- `basicAuth` - Basic HTTP authentication
- `headerAuth` - Header-based authentication
- `jwtAuth` - JWT token validation

**Response Modes:**
- `onReceived` - Respond immediately when webhook received
- `lastNode` - Respond with data from last node
- `responseNode` - Use specific "Respond to Webhook" node

---

### Schedule Trigger (Cron)
**Type:** `n8n-nodes-base.scheduleTrigger`
**Version:** 1.2
**Purpose:** Execute workflow on schedule

```json
{
  "type": "n8n-nodes-base.scheduleTrigger",
  "typeVersion": 1.2,
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

**Interval Fields:**
- `seconds` - Every X seconds
- `minutes` - Every X minutes
- `hours` - Every X hours
- `days` - Every X days
- `weeks` - Every X weeks
- `months` - Every X months
- `cronExpression` - Custom cron expression

**Cron Expression Format:** `minute hour dayOfMonth month dayOfWeek`

Common Patterns:
```
0 9 * * 1-5      # Weekdays at 9 AM
0 */4 * * *      # Every 4 hours
0 0 1 * *        # First of each month at midnight
*/15 * * * *     # Every 15 minutes
0 0 * * 0        # Every Sunday at midnight
```

---

### Interval Trigger
**Type:** `n8n-nodes-base.interval`
**Version:** 1
**Purpose:** Trigger at regular intervals

```json
{
  "type": "n8n-nodes-base.interval",
  "typeVersion": 1,
  "parameters": {
    "interval": 5,
    "unit": "minutes"
  }
}
```

---

### Chat Trigger (AI)
**Type:** `@n8n/n8n-nodes-langchain.chatTrigger`
**Version:** 1.1
**Purpose:** Receive chat messages for AI workflows

```json
{
  "type": "@n8n/n8n-nodes-langchain.chatTrigger",
  "typeVersion": 1.1,
  "parameters": {
    "mode": "hostedChat",
    "options": {
      "title": "My AI Assistant",
      "subtitle": "How can I help you?",
      "inputPlaceholder": "Type your message..."
    }
  }
}
```

---

## Communication Nodes
<!-- chunk: 15-communication | keywords: slack, email, discord, telegram -->

### Slack Node
**Type:** `n8n-nodes-base.slack`
**Version:** 2.4
**Credentials:** `slackOAuth2Api` or `slackApi`

```json
{
  "type": "n8n-nodes-base.slack",
  "typeVersion": 2.4,
  "parameters": {
    "resource": "message",
    "operation": "post",
    "channel": { "mode": "id", "value": "C0123456789" },
    "messageType": "text",
    "text": "Hello from n8n!",
    "otherOptions": {
      "includeLinkToWorkflow": false
    }
  },
  "credentials": {
    "slackOAuth2Api": { "id": "cred-id", "name": "Slack OAuth2" }
  }
}
```

**Resources & Operations:**
| Resource | Operations |
|----------|------------|
| `message` | post, update, delete, get, getPermalink |
| `channel` | archive, close, create, get, getAll, history, invite, join, kick, leave, member, open, rename, replies, setPurpose, setTopic, unarchive |
| `reaction` | add, get, remove |
| `star` | add, delete, getAll |
| `file` | getAll, upload |
| `user` | get, getAll, getPresence, updateProfile |
| `userGroup` | create, disable, enable, getAll, update |

---

### Email Send Node
**Type:** `n8n-nodes-base.emailSend`
**Version:** 2.1
**Credentials:** `smtp`

```json
{
  "type": "n8n-nodes-base.emailSend",
  "typeVersion": 2.1,
  "parameters": {
    "fromEmail": "sender@example.com",
    "toEmail": "recipient@example.com",
    "subject": "Email Subject",
    "emailFormat": "html",
    "html": "<h1>Hello</h1><p>This is the email body.</p>",
    "options": {
      "ccEmail": "cc@example.com",
      "bccEmail": "bcc@example.com",
      "replyTo": "reply@example.com",
      "attachments": "data"
    }
  },
  "credentials": {
    "smtp": { "id": "cred-id", "name": "SMTP" }
  }
}
```

---

### Discord Node
**Type:** `n8n-nodes-base.discord`
**Version:** 2
**Credentials:** `discordWebhookApi` or `discordOAuth2Api`

```json
{
  "type": "n8n-nodes-base.discord",
  "typeVersion": 2,
  "parameters": {
    "resource": "message",
    "operation": "send",
    "webhookUri": "https://discord.com/api/webhooks/...",
    "content": "Hello from n8n!",
    "options": {
      "username": "n8n Bot",
      "avatarUrl": ""
    }
  }
}
```

---

## Project Management Nodes
<!-- chunk: 15-project-mgmt | keywords: jira, asana, linear, trello, clickup -->

### Jira Node
**Type:** `n8n-nodes-base.jira`
**Version:** 1
**Credentials:** `jiraSoftwareCloudApi`

```json
{
  "type": "n8n-nodes-base.jira",
  "typeVersion": 1,
  "parameters": {
    "resource": "issue",
    "operation": "create",
    "project": "PROJ",
    "issueType": "Task",
    "summary": "Issue title",
    "additionalFields": {
      "description": "Issue description",
      "priority": "Medium",
      "labels": ["bug", "urgent"],
      "assignee": "user-id"
    }
  },
  "credentials": {
    "jiraSoftwareCloudApi": { "id": "cred-id", "name": "Jira" }
  }
}
```

**Resources & Operations:**
| Resource | Operations |
|----------|------------|
| `issue` | create, update, get, getAll, delete, changelog, notify, transitions |
| `issueAttachment` | add, get, getAll, remove |
| `issueComment` | add, get, getAll, remove, update |
| `user` | create, delete, get |

---

### Linear Node
**Type:** `n8n-nodes-base.linear`
**Version:** 1.1
**Credentials:** `linearApi`

```json
{
  "type": "n8n-nodes-base.linear",
  "typeVersion": 1.1,
  "parameters": {
    "resource": "issue",
    "operation": "create",
    "teamId": "team-uuid",
    "title": "Issue title",
    "additionalFields": {
      "description": "Issue description",
      "priority": 2,
      "stateId": "state-uuid"
    }
  },
  "credentials": {
    "linearApi": { "id": "cred-id", "name": "Linear" }
  }
}
```

---

## CRM Sales Nodes
<!-- chunk: 15-crm | keywords: salesforce, hubspot, pipedrive -->

### Salesforce Node
**Type:** `n8n-nodes-base.salesforce`
**Version:** 1
**Credentials:** `salesforceOAuth2Api`

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
      "email": "john.smith@example.com",
      "phone": "+1234567890",
      "accountId": "001..."
    }
  },
  "credentials": {
    "salesforceOAuth2Api": { "id": "cred-id", "name": "Salesforce" }
  }
}
```

**Resources:** `account`, `attachment`, `case`, `contact`, `customObject`, `document`, `flow`, `lead`, `opportunity`, `search`, `task`, `user`

---

### HubSpot Node
**Type:** `n8n-nodes-base.hubspot`
**Version:** 2.1
**Credentials:** `hubspotOAuth2Api` or `hubspotApi`

```json
{
  "type": "n8n-nodes-base.hubspot",
  "typeVersion": 2.1,
  "parameters": {
    "resource": "contact",
    "operation": "create",
    "additionalFields": {
      "email": "contact@example.com",
      "firstName": "John",
      "lastName": "Doe",
      "phone": "+1234567890"
    }
  },
  "credentials": {
    "hubspotOAuth2Api": { "id": "cred-id", "name": "HubSpot" }
  }
}
```

**Resources:** `contact`, `company`, `deal`, `engagement`, `form`, `ticket`, `contactList`

---

## Database Nodes
<!-- chunk: 15-database | keywords: postgres, mysql, mongodb, redis -->

### PostgreSQL Node
**Type:** `n8n-nodes-base.postgres`
**Version:** 2.5
**Credentials:** `postgres`

```json
{
  "type": "n8n-nodes-base.postgres",
  "typeVersion": 2.5,
  "parameters": {
    "operation": "executeQuery",
    "query": "SELECT * FROM users WHERE status = $1",
    "options": {
      "queryParams": "active"
    }
  },
  "credentials": {
    "postgres": { "id": "cred-id", "name": "PostgreSQL" }
  }
}
```

**Operations:**
- `executeQuery` - Run custom SQL query
- `insert` - Insert rows
- `update` - Update rows
- `upsert` - Insert or update
- `delete` - Delete rows
- `select` - Select rows with filters

---

### MongoDB Node
**Type:** `n8n-nodes-base.mongoDb`
**Version:** 1.2
**Credentials:** `mongoDb`

```json
{
  "type": "n8n-nodes-base.mongoDb",
  "typeVersion": 1.2,
  "parameters": {
    "operation": "find",
    "collection": "users",
    "query": "{ \"status\": \"active\" }",
    "options": {
      "limit": 100,
      "sort": "{ \"createdAt\": -1 }"
    }
  },
  "credentials": {
    "mongoDb": { "id": "cred-id", "name": "MongoDB" }
  }
}
```

**Operations:** `aggregate`, `delete`, `find`, `findOneAndReplace`, `findOneAndUpdate`, `insert`, `update`

---

### Redis Node
**Type:** `n8n-nodes-base.redis`
**Version:** 1
**Credentials:** `redis`

```json
{
  "type": "n8n-nodes-base.redis",
  "typeVersion": 1,
  "parameters": {
    "operation": "get",
    "key": "myKey",
    "options": {
      "dotNotation": true
    }
  },
  "credentials": {
    "redis": { "id": "cred-id", "name": "Redis" }
  }
}
```

**Operations:** `delete`, `get`, `incr`, `info`, `keys`, `pop`, `publish`, `push`, `set`

---

## File Storage Nodes
<!-- chunk: 15-file-storage | keywords: google-drive, s3, dropbox, sheets -->

### Google Sheets Node
**Type:** `n8n-nodes-base.googleSheets`
**Version:** 4.5
**Credentials:** `googleSheetsOAuth2Api`

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
    },
    "options": {
      "cellFormat": "USER_ENTERED"
    }
  },
  "credentials": {
    "googleSheetsOAuth2Api": { "id": "cred-id", "name": "Google Sheets" }
  }
}
```

**Operations:** `append`, `appendOrUpdate`, `clear`, `create`, `delete`, `read`, `update`

---

### AWS S3 Node
**Type:** `n8n-nodes-base.awsS3`
**Version:** 2
**Credentials:** `aws`

```json
{
  "type": "n8n-nodes-base.awsS3",
  "typeVersion": 2,
  "parameters": {
    "resource": "file",
    "operation": "upload",
    "bucketName": "my-bucket",
    "fileName": "path/to/file.json",
    "binaryData": true,
    "binaryPropertyName": "data",
    "options": {
      "acl": "private",
      "storageClass": "STANDARD"
    }
  },
  "credentials": {
    "aws": { "id": "cred-id", "name": "AWS" }
  }
}
```

**Resources & Operations:**
| Resource | Operations |
|----------|------------|
| `bucket` | create, delete, getAll, search |
| `file` | copy, delete, download, getAll, upload |
| `folder` | create, delete, getAll |

---

## AI ML Nodes
<!-- chunk: 15-ai | keywords: openai, langchain, agent, llm, chat -->

### AI Agent Node
**Type:** `@n8n/n8n-nodes-langchain.agent`
**Version:** 1.7
**Purpose:** Create AI agents with tools and memory

```json
{
  "type": "@n8n/n8n-nodes-langchain.agent",
  "typeVersion": 1.7,
  "parameters": {
    "agent": "conversationalAgent",
    "promptType": "define",
    "text": "={{ $json.chatInput }}",
    "options": {
      "systemMessage": "You are a helpful assistant.",
      "maxIterations": 10,
      "returnIntermediateSteps": false
    }
  }
}
```

**Inputs:**
- `main` (required) - Chat input data
- `ai_languageModel` (required) - LLM connection
- `ai_memory` (optional) - Conversation memory
- `ai_tool` (optional, multiple) - Tools for agent
- `ai_outputParser` (optional) - Output formatting

---

### OpenAI Chat Model
**Type:** `@n8n/n8n-nodes-langchain.lmChatOpenAi`
**Version:** 1.2
**Credentials:** `openAiApi`

```json
{
  "type": "@n8n/n8n-nodes-langchain.lmChatOpenAi",
  "typeVersion": 1.2,
  "parameters": {
    "model": "gpt-4o",
    "options": {
      "temperature": 0.7,
      "maxTokens": 2048,
      "topP": 1,
      "frequencyPenalty": 0,
      "presencePenalty": 0
    }
  },
  "credentials": {
    "openAiApi": { "id": "cred-id", "name": "OpenAI" }
  }
}
```

**Models:** `gpt-4o`, `gpt-4o-mini`, `gpt-4-turbo`, `gpt-4`, `gpt-3.5-turbo`

---

### Buffer Memory Node
**Type:** `@n8n/n8n-nodes-langchain.memoryBufferWindow`
**Version:** 1.3

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

---

## Utility Nodes
<!-- chunk: 15-utility | keywords: http, request, respond, execute -->

### HTTP Request Node
**Type:** `n8n-nodes-base.httpRequest`
**Version:** 4.2
**Purpose:** Make HTTP requests to any API

```json
{
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.2,
  "parameters": {
    "method": "POST",
    "url": "https://api.example.com/endpoint",
    "authentication": "genericCredentialType",
    "genericAuthType": "httpHeaderAuth",
    "sendHeaders": true,
    "headerParameters": {
      "parameters": [
        { "name": "Content-Type", "value": "application/json" }
      ]
    },
    "sendBody": true,
    "bodyParameters": {
      "parameters": [
        { "name": "key", "value": "={{ $json.data }}" }
      ]
    },
    "options": {
      "response": { "response": { "responseFormat": "json" } },
      "timeout": 30000,
      "allowUnauthorizedCerts": false
    }
  }
}
```

**Authentication Options:**
- `none` - No authentication
- `predefinedCredentialType` - Use existing credential
- `genericCredentialType` - Generic auth (Basic, Header, OAuth2, etc.)

**Body Content Types:**
- `json` - JSON body
- `form-urlencoded` - Form data
- `multipart-form-data` - File uploads
- `raw` - Raw content

---

### Respond to Webhook Node
**Type:** `n8n-nodes-base.respondToWebhook`
**Version:** 1.1
**Purpose:** Send custom response to webhook caller

```json
{
  "type": "n8n-nodes-base.respondToWebhook",
  "typeVersion": 1.1,
  "parameters": {
    "respondWith": "json",
    "responseBody": "={{ $json }}",
    "options": {
      "responseCode": 200,
      "responseHeaders": {
        "entries": [
          { "name": "X-Custom-Header", "value": "custom-value" }
        ]
      }
    }
  }
}
```

---

## Node JSON Structure
<!-- chunk: 15-json-structure | keywords: structure, format, template -->

### Complete Node Definition
```json
{
  "id": "unique-uuid",
  "name": "Node Display Name",
  "type": "n8n-nodes-base.nodetype",
  "typeVersion": 1.0,
  "position": [250, 300],
  "parameters": {
    "param1": "value1",
    "param2": "={{ $json.field }}"
  },
  "credentials": {
    "credentialType": {
      "id": "credential-uuid",
      "name": "Credential Display Name"
    }
  },
  "webhookId": "optional-webhook-uuid",
  "disabled": false,
  "notes": "Optional node notes",
  "notesInFlow": false,
  "executeOnce": false,
  "retryOnFail": false,
  "maxTries": 3,
  "waitBetweenTries": 1000,
  "continueOnFail": false,
  "pairedItem": { "item": 0 },
  "alwaysOutputData": false,
  "color": "#ff6d5a"
}
```

### Position Guidelines
- Start nodes: `[250, 300]`
- Subsequent nodes: Increment X by 250 (`[500, 300]`, `[750, 300]`)
- Parallel branches: Offset Y by 150 (`[500, 150]`, `[500, 450]`)
- Keep Y between 0-600 for visibility

---

## Statistics Summary

| Category | Node Count | Common Examples |
|----------|------------|-----------------|
| Core/Flow | 29 | If, Switch, Merge, Split, Wait |
| Transform | 132 | Set, Code, Filter, Aggregate |
| Triggers | 112 | Webhook, Schedule, Slack, GitHub |
| Communication | 11 | Slack, Email, Discord, Teams |
| Project Mgmt | 8 | Jira, Asana, Linear, Trello |
| CRM/Sales | 8 | Salesforce, HubSpot, Pipedrive |
| Database | 6 | PostgreSQL, MySQL, MongoDB, Redis |
| File Storage | 18 | Google Drive, Sheets, S3, Dropbox |
| AI/ML | 50+ | Agent, OpenAI, Memory, Tools |
| Utility | 5 | HTTP Request, Webhook, Wait |

---

*Generated from n8n v1.122.0 - packages/nodes-base/nodes*
