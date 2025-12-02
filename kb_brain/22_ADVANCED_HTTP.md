# 22_ADVANCED_HTTP.md
<!-- repo: n8n | version: 1.122.0 | generated: 2025-12-02 -->
<!-- tags: http, request, api, authentication, body -->

## Contents
- [Overview](#overview)
- [Complete Node Configuration](#complete-node-configuration)
- [Authentication Types](#authentication-types)
- [Body Types](#body-types)
- [Response Handling](#response-handling)
- [Advanced Options](#advanced-options)
- [Examples](#examples)

---

## Overview
<!-- chunk: 22-overview | keywords: http, request, methods -->

The HTTP Request node supports all HTTP methods and authentication types:

| Method | Use Case |
|--------|----------|
| GET | Retrieve data |
| POST | Create resources |
| PUT | Replace resources |
| PATCH | Update resources |
| DELETE | Remove resources |
| HEAD | Get headers only |
| OPTIONS | Check CORS/capabilities |

---

## Complete Node Configuration
<!-- chunk: 22-config | keywords: config, parameters, full -->

```json
{
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.2,
  "position": [500, 300],
  "parameters": {
    "method": "POST",
    "url": "https://api.example.com/endpoint",
    "authentication": "predefinedCredentialType",
    "nodeCredentialType": "httpHeaderAuth",
    "sendHeaders": true,
    "headerParameters": {
      "parameters": [
        { "name": "Content-Type", "value": "application/json" },
        { "name": "X-Custom-Header", "value": "custom-value" }
      ]
    },
    "sendQuery": true,
    "queryParameters": {
      "parameters": [
        { "name": "page", "value": "1" },
        { "name": "limit", "value": "100" }
      ]
    },
    "sendBody": true,
    "bodyParameters": {
      "parameters": [
        { "name": "field1", "value": "value1" },
        { "name": "field2", "value": "={{ $json.data }}" }
      ]
    },
    "options": {
      "allowUnauthorizedCerts": false,
      "batching": {
        "batch": {
          "batchSize": 10,
          "batchInterval": 1000
        }
      },
      "proxy": "",
      "timeout": 30000,
      "redirect": {
        "redirect": {
          "followRedirects": true,
          "maxRedirects": 21
        }
      },
      "response": {
        "response": {
          "responseFormat": "json",
          "fullResponse": false
        }
      }
    }
  },
  "credentials": {
    "httpHeaderAuth": {
      "id": "credential-id",
      "name": "API Key"
    }
  }
}
```

---

## Authentication Types
<!-- chunk: 22-auth | keywords: auth, oauth, basic, bearer -->

### None
```json
{
  "authentication": "none"
}
```

### Predefined Credential Type
```json
{
  "authentication": "predefinedCredentialType",
  "nodeCredentialType": "hubspotApi"
}
```

### Generic Credential Type

**Basic Auth:**
```json
{
  "authentication": "genericCredentialType",
  "genericAuthType": "httpBasicAuth"
}
```

**Header Auth:**
```json
{
  "authentication": "genericCredentialType",
  "genericAuthType": "httpHeaderAuth"
}
```

**Bearer Token:**
```json
{
  "authentication": "genericCredentialType",
  "genericAuthType": "httpBearerAuth"
}
```

**Query Auth:**
```json
{
  "authentication": "genericCredentialType",
  "genericAuthType": "httpQueryAuth"
}
```

**Digest Auth:**
```json
{
  "authentication": "genericCredentialType",
  "genericAuthType": "httpDigestAuth"
}
```

**OAuth2:**
```json
{
  "authentication": "genericCredentialType",
  "genericAuthType": "oAuth2Api"
}
```

**Custom Auth:**
```json
{
  "authentication": "genericCredentialType",
  "genericAuthType": "httpCustomAuth"
}
```

---

## Body Types
<!-- chunk: 22-body | keywords: body, json, form, multipart -->

### JSON Body
```json
{
  "sendBody": true,
  "specifyBody": "json",
  "jsonBody": "={{ JSON.stringify({ key: 'value', nested: { field: $json.data } }) }}"
}
```

### Form URL Encoded
```json
{
  "sendBody": true,
  "contentType": "form-urlencoded",
  "bodyParameters": {
    "parameters": [
      { "name": "username", "value": "user@example.com" },
      { "name": "password", "value": "secret" }
    ]
  }
}
```

### Multipart Form Data (File Upload)
```json
{
  "sendBody": true,
  "contentType": "multipart-form-data",
  "bodyParameters": {
    "parameters": [
      {
        "parameterType": "formBinaryData",
        "name": "file",
        "inputDataFieldName": "data"
      },
      {
        "parameterType": "formData",
        "name": "description",
        "value": "Uploaded file"
      }
    ]
  }
}
```

### Raw Body
```json
{
  "sendBody": true,
  "contentType": "raw",
  "rawContentType": "application/xml",
  "body": "<?xml version=\"1.0\"?><root><item>value</item></root>"
}
```

### n8n Binary Data
```json
{
  "sendBody": true,
  "contentType": "binaryData",
  "inputDataFieldName": "data"
}
```

---

## Response Handling
<!-- chunk: 22-response | keywords: response, format, binary -->

### Response Format Options

**Auto-Detect:**
```json
{
  "options": {
    "response": {
      "response": {
        "responseFormat": "autodetect"
      }
    }
  }
}
```

**JSON:**
```json
{
  "options": {
    "response": {
      "response": {
        "responseFormat": "json"
      }
    }
  }
}
```

**Text:**
```json
{
  "options": {
    "response": {
      "response": {
        "responseFormat": "text",
        "outputPropertyName": "response"
      }
    }
  }
}
```

**File/Binary:**
```json
{
  "options": {
    "response": {
      "response": {
        "responseFormat": "file",
        "outputPropertyName": "data"
      }
    }
  }
}
```

### Full Response (Headers + Status)
```json
{
  "options": {
    "response": {
      "response": {
        "fullResponse": true
      }
    }
  }
}
```

Output:
```json
{
  "body": { "data": "..." },
  "headers": {
    "content-type": "application/json",
    "x-rate-limit": "100"
  },
  "statusCode": 200,
  "statusMessage": "OK"
}
```

### Ignore HTTP Errors
```json
{
  "options": {
    "response": {
      "response": {
        "neverError": true
      }
    }
  }
}
```

---

## Advanced Options
<!-- chunk: 22-options | keywords: options, timeout, proxy -->

### Timeout
```json
{
  "options": {
    "timeout": 60000
  }
}
```

### Proxy
```json
{
  "options": {
    "proxy": "http://proxy.example.com:8080"
  }
}
```

### SSL Certificates
```json
{
  "options": {
    "allowUnauthorizedCerts": true
  }
}
```

### Redirects
```json
{
  "options": {
    "redirect": {
      "redirect": {
        "followRedirects": true,
        "maxRedirects": 10
      }
    }
  }
}
```

### Batching
```json
{
  "options": {
    "batching": {
      "batch": {
        "batchSize": 5,
        "batchInterval": 1000
      }
    }
  }
}
```

### Pagination

**Update Parameter:**
```json
{
  "options": {
    "pagination": {
      "pagination": {
        "paginationMode": "updateAParameterInEachRequest",
        "parameters": {
          "parameters": [
            {
              "name": "page",
              "type": "qs",
              "value": "={{ $pageCount + 1 }}"
            }
          ]
        },
        "paginationCompleteWhen": "responseIsEmpty",
        "maxRequests": 100
      }
    }
  }
}
```

**Response Contains Next URL:**
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

---

## Examples
<!-- chunk: 22-examples | keywords: examples, patterns -->

### Example 1: REST API with Bearer Token

```json
{
  "type": "n8n-nodes-base.httpRequest",
  "parameters": {
    "method": "GET",
    "url": "https://api.example.com/v1/users",
    "authentication": "genericCredentialType",
    "genericAuthType": "httpBearerAuth",
    "sendQuery": true,
    "queryParameters": {
      "parameters": [
        { "name": "status", "value": "active" },
        { "name": "limit", "value": "50" }
      ]
    }
  },
  "credentials": {
    "httpBearerAuth": { "id": "1", "name": "API Token" }
  }
}
```

### Example 2: POST JSON with Custom Headers

```json
{
  "type": "n8n-nodes-base.httpRequest",
  "parameters": {
    "method": "POST",
    "url": "https://api.example.com/v1/orders",
    "authentication": "genericCredentialType",
    "genericAuthType": "httpHeaderAuth",
    "sendHeaders": true,
    "headerParameters": {
      "parameters": [
        { "name": "X-Request-ID", "value": "={{ $execution.id }}" },
        { "name": "X-Timestamp", "value": "={{ $now.toISO() }}" }
      ]
    },
    "sendBody": true,
    "specifyBody": "json",
    "jsonBody": "={{ JSON.stringify({ orderId: $json.id, items: $json.items, total: $json.total }) }}"
  }
}
```

### Example 3: File Upload

```json
{
  "type": "n8n-nodes-base.httpRequest",
  "parameters": {
    "method": "POST",
    "url": "https://api.example.com/v1/upload",
    "authentication": "predefinedCredentialType",
    "nodeCredentialType": "dropboxApi",
    "sendBody": true,
    "contentType": "multipart-form-data",
    "bodyParameters": {
      "parameters": [
        {
          "parameterType": "formBinaryData",
          "name": "file",
          "inputDataFieldName": "data"
        },
        {
          "parameterType": "formData",
          "name": "folder",
          "value": "/uploads"
        }
      ]
    }
  }
}
```

### Example 4: GraphQL Query

```json
{
  "type": "n8n-nodes-base.httpRequest",
  "parameters": {
    "method": "POST",
    "url": "https://api.example.com/graphql",
    "authentication": "genericCredentialType",
    "genericAuthType": "httpHeaderAuth",
    "sendHeaders": true,
    "headerParameters": {
      "parameters": [
        { "name": "Content-Type", "value": "application/json" }
      ]
    },
    "sendBody": true,
    "specifyBody": "json",
    "jsonBody": "={{ JSON.stringify({ query: 'query GetUser($id: ID!) { user(id: $id) { name email } }', variables: { id: $json.userId } }) }}"
  }
}
```

### Example 5: OAuth2 with Refresh

```json
{
  "type": "n8n-nodes-base.httpRequest",
  "parameters": {
    "method": "GET",
    "url": "https://api.example.com/v1/me",
    "authentication": "genericCredentialType",
    "genericAuthType": "oAuth2Api"
  },
  "credentials": {
    "oAuth2Api": {
      "id": "oauth-cred-id",
      "name": "OAuth2 Connection"
    }
  }
}
```

### Example 6: Webhook Callback

```json
{
  "type": "n8n-nodes-base.httpRequest",
  "parameters": {
    "method": "POST",
    "url": "={{ $json.callbackUrl }}",
    "sendBody": true,
    "specifyBody": "json",
    "jsonBody": "={{ JSON.stringify({ status: 'completed', result: $json.processedData, executionId: $execution.id }) }}"
  }
}
```

### Example 7: Retry with Exponential Backoff

```json
{
  "type": "n8n-nodes-base.httpRequest",
  "retryOnFail": true,
  "maxTries": 5,
  "waitBetweenTries": 2000,
  "parameters": {
    "method": "POST",
    "url": "https://api.example.com/v1/process",
    "options": {
      "timeout": 30000,
      "response": {
        "response": {
          "neverError": false
        }
      }
    }
  }
}
```

### Example 8: Download Binary File

```json
{
  "type": "n8n-nodes-base.httpRequest",
  "parameters": {
    "method": "GET",
    "url": "https://example.com/files/report.pdf",
    "options": {
      "response": {
        "response": {
          "responseFormat": "file",
          "outputPropertyName": "document"
        }
      }
    }
  }
}
```

---

## IHttpRequestOptions Interface
<!-- chunk: 22-interface | keywords: interface, types -->

```typescript
interface IHttpRequestOptions {
  url: string;
  baseURL?: string;
  headers?: IDataObject;
  method?: 'DELETE' | 'GET' | 'HEAD' | 'PATCH' | 'POST' | 'PUT';
  body?: FormData | GenericValue | GenericValue[] | Buffer | URLSearchParams;
  qs?: IDataObject;
  arrayFormat?: 'indices' | 'brackets' | 'repeat' | 'comma';
  auth?: {
    username: string;
    password: string;
    sendImmediately?: boolean;
  };
  disableFollowRedirect?: boolean;
  encoding?: 'arraybuffer' | 'blob' | 'document' | 'json' | 'text' | 'stream';
  skipSslCertificateValidation?: boolean;
  returnFullResponse?: boolean;
  ignoreHttpStatusErrors?: boolean;
  proxy?: {
    host: string;
    port: number;
    auth?: { username: string; password: string };
    protocol?: string;
  };
  timeout?: number;
  json?: boolean;
  abortSignal?: GenericAbortSignal;
}
```

---

*Source: packages/nodes-base/nodes/HttpRequest/V3/*
