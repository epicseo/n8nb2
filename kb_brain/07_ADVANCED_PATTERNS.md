# 07_ADVANCED_PATTERNS.md
<!-- repo: n8n | version: 1.122.0 | generated: 2025-12-02 -->
<!-- tags: advanced, binary, http, files, pagination, errors -->

## Contents
- [Binary Data Handling](#binary-data-handling)
- [HTTP Request Advanced](#http-request-advanced)
- [Error Handling Patterns](#error-handling-patterns)
- [Pagination Patterns](#pagination-patterns)
- [Subworkflows](#subworkflows)

---

## Binary Data Handling
<!-- chunk: 07-binary | keywords: binary, files, upload, download -->

### IBinaryData Interface
```typescript
interface IBinaryData {
  data: string;           // Base64 or reference ID
  mimeType: string;       // e.g., "application/pdf"
  fileType?: 'text' | 'json' | 'image' | 'audio' | 'video' | 'pdf' | 'html';
  fileName?: string;      // "document.pdf"
  fileExtension?: string; // "pdf"
  fileSize?: string;      // "1.2 MB"
  id?: string;            // Reference ID
}
```

### Binary in Output
```json
{
  "json": { "name": "document.pdf" },
  "binary": {
    "data": {
      "data": "base64-content",
      "mimeType": "application/pdf",
      "fileName": "document.pdf",
      "fileExtension": "pdf",
      "fileSize": "1.2 MB"
    }
  }
}
```

### Download File
```json
{
  "type": "n8n-nodes-base.httpRequest",
  "parameters": {
    "method": "GET",
    "url": "https://example.com/file.pdf",
    "options": {
      "response": {
        "response": { "responseFormat": "file" }
      }
    }
  }
}
```

### Upload File (Multipart)
```json
{
  "type": "n8n-nodes-base.httpRequest",
  "parameters": {
    "method": "POST",
    "url": "https://api.example.com/upload",
    "sendBody": true,
    "contentType": "multipart-form-data",
    "bodyParameters": {
      "parameters": [
        {
          "parameterType": "formBinaryData",
          "name": "file",
          "inputDataFieldName": "data"
        }
      ]
    }
  }
}
```

### Read Binary File
```json
{
  "type": "n8n-nodes-base.readBinaryFile",
  "parameters": { "filePath": "/path/to/file.pdf" }
}
```

### Write Binary File
```json
{
  "type": "n8n-nodes-base.writeBinaryFile",
  "parameters": {
    "fileName": "/output/result.pdf",
    "dataPropertyName": "data"
  }
}
```

### Spreadsheet File (Excel/CSV)
```json
{
  "type": "n8n-nodes-base.spreadsheetFile",
  "parameters": {
    "operation": "toJson",
    "fileFormat": "autoDetect",
    "options": { "headerRow": true, "sheetName": "Sheet1" }
  }
}
```

### Access Binary in Expressions
```javascript
{{ $binary.data.fileName }}    // Filename
{{ $binary.data.mimeType }}    // MIME type
{{ $binary.data.fileSize }}    // File size
```

### Common MIME Types
| Extension | MIME Type |
|-----------|-----------|
| .json | application/json |
| .pdf | application/pdf |
| .xlsx | application/vnd.openxmlformats-officedocument.spreadsheetml.sheet |
| .csv | text/csv |
| .png | image/png |
| .jpg | image/jpeg |

---

## HTTP Request Advanced
<!-- chunk: 07-http | keywords: http, request, authentication -->

### Complete Configuration
```json
{
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.2,
  "parameters": {
    "method": "POST",
    "url": "https://api.example.com/endpoint",
    "authentication": "predefinedCredentialType",
    "nodeCredentialType": "httpHeaderAuth",
    "sendHeaders": true,
    "headerParameters": {
      "parameters": [
        { "name": "Content-Type", "value": "application/json" }
      ]
    },
    "sendQuery": true,
    "queryParameters": {
      "parameters": [
        { "name": "page", "value": "1" }
      ]
    },
    "sendBody": true,
    "bodyParameters": {
      "parameters": [
        { "name": "field", "value": "={{ $json.data }}" }
      ]
    },
    "options": {
      "timeout": 30000,
      "redirect": { "redirect": { "followRedirects": true, "maxRedirects": 10 } },
      "response": { "response": { "responseFormat": "json" } }
    }
  },
  "credentials": { "httpHeaderAuth": { "id": "1", "name": "API Key" } }
}
```

### Authentication Types
| Type | Configuration |
|------|---------------|
| None | `"authentication": "none"` |
| Basic Auth | `"genericAuthType": "httpBasicAuth"` |
| Header Auth | `"genericAuthType": "httpHeaderAuth"` |
| Bearer Token | `"genericAuthType": "httpBearerAuth"` |
| Query Auth | `"genericAuthType": "httpQueryAuth"` |
| OAuth2 | `"genericAuthType": "oAuth2Api"` |

### Body Types

**JSON Body:**
```json
{
  "sendBody": true,
  "specifyBody": "json",
  "jsonBody": "={{ JSON.stringify({ key: $json.value }) }}"
}
```

**Form URL Encoded:**
```json
{
  "sendBody": true,
  "contentType": "form-urlencoded",
  "bodyParameters": {
    "parameters": [{ "name": "field", "value": "value" }]
  }
}
```

**Raw Body (XML):**
```json
{
  "sendBody": true,
  "contentType": "raw",
  "rawContentType": "application/xml",
  "body": "<?xml version=\"1.0\"?><root><item>value</item></root>"
}
```

### Full Response (Headers + Status)
```json
{
  "options": {
    "response": { "response": { "fullResponse": true } }
  }
}
```

Output:
```json
{
  "body": { "data": "..." },
  "headers": { "content-type": "application/json" },
  "statusCode": 200
}
```

### Batching
```json
{
  "options": {
    "batching": {
      "batch": { "batchSize": 5, "batchInterval": 1000 }
    }
  }
}
```

---

## Error Handling Patterns
<!-- chunk: 07-errors | keywords: error, retry, continueOnFail -->

### Node-Level Options
```json
{
  "name": "HTTP Request",
  "type": "n8n-nodes-base.httpRequest",
  "continueOnFail": true,
  "retryOnFail": true,
  "maxTries": 3,
  "waitBetweenTries": 1000,
  "onError": "continueErrorOutput",
  "parameters": { "url": "https://api.example.com" }
}
```

### OnError Options
| Value | Behavior |
|-------|----------|
| `continueErrorOutput` | Continue to error output |
| `continueRegularOutput` | Continue to regular output |
| `stopWorkflow` | Stop execution |

### Error Output Structure
When `continueOnFail` is enabled:
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

### Error Trigger Workflow
```json
{
  "type": "n8n-nodes-base.errorTrigger",
  "typeVersion": 1,
  "parameters": {}
}
```

Error data:
```json
{
  "execution": {
    "id": "12345",
    "error": { "message": "Error message" },
    "lastNodeExecuted": "HTTP Request"
  },
  "workflow": { "id": "1", "name": "My Workflow" }
}
```

### Set Error Workflow
```json
{
  "settings": { "errorWorkflow": "error-handler-workflow-id" }
}
```

### Stop and Error Node
```json
{
  "type": "n8n-nodes-base.stopAndError",
  "parameters": {
    "errorType": "errorMessage",
    "errorMessage": "Validation failed: missing required field"
  }
}
```

### Try/Catch Pattern
```json
{
  "nodes": [
    {
      "name": "HTTP Request",
      "type": "n8n-nodes-base.httpRequest",
      "continueOnFail": true,
      "parameters": { "url": "https://api.example.com" }
    },
    {
      "name": "Check Error",
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
      "name": "Handle Error", "type": "n8n-nodes-base.set",
      "parameters": { "assignments": { "assignments": [{ "name": "status", "value": "failed" }] } }
    },
    {
      "name": "Success", "type": "n8n-nodes-base.set",
      "parameters": { "assignments": { "assignments": [{ "name": "status", "value": "success" }] } }
    }
  ],
  "connections": {
    "HTTP Request": { "main": [[{ "node": "Check Error" }]] },
    "Check Error": {
      "main": [[{ "node": "Handle Error" }], [{ "node": "Success" }]]
    }
  }
}
```

---

## Pagination Patterns
<!-- chunk: 07-pagination | keywords: pagination, pages, cursor -->

### Update Parameter Method
```json
{
  "options": {
    "pagination": {
      "pagination": {
        "paginationMode": "updateAParameterInEachRequest",
        "parameters": {
          "parameters": [{
            "name": "page",
            "type": "qs",
            "value": "={{ $pageCount + 1 }}"
          }]
        },
        "paginationCompleteWhen": "responseIsEmpty",
        "maxRequests": 100
      }
    }
  }
}
```

### Response Contains Next URL
```json
{
  "options": {
    "pagination": {
      "pagination": {
        "paginationMode": "responseContainsNextURL",
        "nextURL": "={{ $response.body.pagination.next_url }}",
        "paginationCompleteWhen": "receiveSpecificStatusCodes",
        "statusCodesWhenComplete": "404",
        "maxRequests": 100
      }
    }
  }
}
```

### Offset-Based Pagination
```json
{
  "options": {
    "pagination": {
      "pagination": {
        "paginationMode": "updateAParameterInEachRequest",
        "parameters": {
          "parameters": [{
            "name": "offset",
            "type": "qs",
            "value": "={{ $pageCount * 100 }}"
          }]
        },
        "paginationCompleteWhen": "responseLengthLessThan",
        "responseLengthValue": 100,
        "maxRequests": 50
      }
    }
  }
}
```

### Cursor-Based Pagination
```json
{
  "options": {
    "pagination": {
      "pagination": {
        "paginationMode": "updateAParameterInEachRequest",
        "parameters": {
          "parameters": [{
            "name": "cursor",
            "type": "qs",
            "value": "={{ $response.body.next_cursor }}"
          }]
        },
        "paginationCompleteWhen": "other",
        "completeExpression": "={{ $response.body.has_more === false }}",
        "maxRequests": 100
      }
    }
  }
}
```

### Pagination Variables
| Variable | Description |
|----------|-------------|
| `$pageCount` | Current page number (0-based) |
| `$response` | Current response object |
| `$response.body` | Response body |
| `$response.headers` | Response headers |

---

## Subworkflows
<!-- chunk: 07-subworkflows | keywords: subworkflow, execute, call -->

### Execute Workflow Node
```json
{
  "type": "n8n-nodes-base.executeWorkflow",
  "typeVersion": 1.1,
  "parameters": {
    "source": "database",
    "workflowId": "workflow-uuid",
    "mode": "each",
    "options": {}
  }
}
```

### Source Options
| Source | Description |
|--------|-------------|
| `database` | Select workflow by ID |
| `parameter` | Specify workflow ID in expression |
| `localFile` | Read from local file |

### Mode Options
| Mode | Description |
|------|-------------|
| `each` | Execute for each input item |
| `once` | Execute once with all items |

### Pass Data to Subworkflow
```json
{
  "parameters": {
    "source": "database",
    "workflowId": "sub-workflow-id",
    "options": {
      "waitForSubWorkflow": true
    }
  }
}
```

### Workflow Trigger Node (in subworkflow)
```json
{
  "type": "n8n-nodes-base.executeWorkflowTrigger",
  "typeVersion": 1,
  "parameters": {}
}
```

### Caller Policy Settings
```json
{
  "settings": {
    "callerPolicy": "workflowsFromSameOwner",
    "callerIds": "workflow-id-1,workflow-id-2"
  }
}
```

| Policy | Description |
|--------|-------------|
| `any` | Any workflow can call |
| `none` | Cannot be called |
| `workflowsFromAList` | Only listed workflows |
| `workflowsFromSameOwner` | Same owner only |

### AI Workflow Tool
```json
{
  "type": "@n8n/n8n-nodes-langchain.toolWorkflow",
  "parameters": {
    "name": "process_order",
    "description": "Process an order",
    "workflowId": "workflow-uuid"
  }
}
```

---

## Best Practices

### Error Handling
1. Use `continueOnFail` for optional operations
2. Set up error workflow for critical flows
3. Implement retry with exponential backoff
4. Log errors to database for analysis

### HTTP Requests
1. Always set appropriate timeout
2. Use batching for rate-limited APIs
3. Implement pagination for large datasets
4. Handle 429 (rate limit) with retry

### Binary Data
1. Use streaming for large files
2. Set correct MIME types
3. Clean up temporary files
4. Validate file types before processing

### Subworkflows
1. Use for reusable logic
2. Set appropriate caller policies
3. Handle timeouts appropriately
4. Pass only necessary data

---

*Source: packages/nodes-base/nodes/*
