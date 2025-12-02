# 19_PAGINATION_PATTERNS.md
<!-- repo: n8n | version: 1.122.0 | generated: 2025-12-02 -->
<!-- tags: pagination, cursor, offset, api, batch -->

## Contents
- [Overview](#overview)
- [Pagination Types](#pagination-types)
- [HTTP Request Pagination](#http-request-pagination)
- [Declarative API Pagination](#declarative-api-pagination)
- [Custom Pagination Patterns](#custom-pagination-patterns)
- [Node Examples](#node-examples)

---

## Overview
<!-- chunk: 19-overview | keywords: pagination, returnAll, limit -->

n8n supports multiple pagination strategies for fetching large datasets from APIs:

| Strategy | Use Case | Parameters |
|----------|----------|------------|
| **Offset** | Page-based APIs | offset, limit, page |
| **Cursor** | Token-based APIs | nextCursor, after |
| **Link Header** | REST APIs with Link header | Link: <url>; rel="next" |
| **Response Path** | Next URL in response body | pagination.next_url |

### Common Parameters

```json
{
  "returnAll": true,        // Fetch all pages automatically
  "limit": 100,             // Max items to return (when returnAll=false)
  "maxResults": 1000        // Safety limit for returnAll
}
```

---

## Pagination Types
<!-- chunk: 19-types | keywords: offset, cursor, types -->

### Type 1: Offset Pagination

API uses `offset` and `limit` parameters.

```
Page 1: GET /items?limit=100&offset=0
Page 2: GET /items?limit=100&offset=100
Page 3: GET /items?limit=100&offset=200
```

**Configuration:**
```typescript
{
  type: 'offset',
  properties: {
    limitParameter: 'limit',
    offsetParameter: 'offset',
    pageSize: 100,
    rootProperty: 'data.items',
    type: 'query'  // 'query' or 'body'
  }
}
```

### Type 2: Cursor Pagination

API returns a cursor/token for next page.

```
Page 1: GET /items?limit=100
Response: { items: [...], nextCursor: "abc123" }

Page 2: GET /items?limit=100&cursor=abc123
Response: { items: [...], nextCursor: "def456" }

Page 3: GET /items?limit=100&cursor=def456
Response: { items: [...], nextCursor: null }  // Last page
```

**Configuration:**
```typescript
{
  type: 'cursor',
  properties: {
    cursorProperty: 'nextCursor',    // Response path to cursor
    cursorParameter: 'cursor',        // Query param name
    limitParameter: 'limit',
    pageSize: 100
  }
}
```

### Type 3: Generic Pagination (Custom)

Custom pagination logic using `IExecutePaginationFunctions`.

```typescript
async function customPagination(
  this: IExecutePaginationFunctions,
  requestOptions: DeclarativeRestApiSettings.ResultOptions
): Promise<INodeExecutionData[]> {
  let allItems: INodeExecutionData[] = [];
  let nextPage: string | undefined;

  do {
    if (nextPage) {
      requestOptions.options.qs = { ...requestOptions.options.qs, page: nextPage };
    }

    const response = await this.helpers.httpRequest(requestOptions.options);
    allItems.push(...response.data);
    nextPage = response.pagination?.next_page;

  } while (nextPage && allItems.length < requestOptions.maxResults);

  return allItems.map(item => ({ json: item }));
}
```

---

## HTTP Request Pagination
<!-- chunk: 19-http | keywords: http, request, pagination -->

### HTTP Request Node Configuration

```json
{
  "type": "n8n-nodes-base.httpRequest",
  "parameters": {
    "method": "GET",
    "url": "https://api.example.com/items",
    "authentication": "predefinedCredentialType",
    "nodeCredentialType": "exampleApi",
    "sendQuery": true,
    "queryParameters": {
      "parameters": [
        { "name": "per_page", "value": "100" }
      ]
    },
    "options": {
      "pagination": {
        "pagination": {
          "paginationMode": "off"
        }
      }
    }
  }
}
```

### Pagination Modes

**Mode: Off (Manual)**
```json
{
  "paginationMode": "off"
}
```
No automatic pagination - handle manually with Loop.

**Mode: Update a Parameter in Each Request**
```json
{
  "paginationMode": "updateAParameterInEachRequest",
  "parameters": {
    "parameters": [
      {
        "name": "page",
        "type": "qs",
        "valueSource": "expression",
        "value": "={{ $pageCount + 1 }}"
      }
    ]
  },
  "paginationCompleteWhen": "responseIsEmpty",
  "maxRequests": 100
}
```

**Mode: Response Contains Next URL**
```json
{
  "paginationMode": "responseContainsNextURL",
  "nextURL": "={{ $response.body.links.next }}",
  "paginationCompleteWhen": "receiveSpecificStatusCodes",
  "statusCodesWhenComplete": "404",
  "maxRequests": 100
}
```

### Pagination Complete Conditions

| Condition | Description |
|-----------|-------------|
| `responseIsEmpty` | Stop when response has no items |
| `receiveSpecificStatusCodes` | Stop on specific HTTP codes |
| `other` | Custom expression evaluation |

### Available Pagination Variables

```javascript
$pageCount      // Current page number (0-indexed)
$response       // Last response object
$response.body  // Response body
$response.headers  // Response headers
```

---

## Declarative API Pagination
<!-- chunk: 19-declarative | keywords: declarative, api, nodes -->

### IN8nRequestOperationPaginationOffset

```typescript
interface IN8nRequestOperationPaginationOffset {
  type: 'offset';
  properties: {
    // Parameter name for limit
    limitParameter: string;

    // Parameter name for offset
    offsetParameter: string;

    // Items per page
    pageSize: number;

    // Where to put parameters
    type: 'body' | 'query';

    // Path to items array in response
    rootProperty?: string;
  };
}
```

**Example:**
```json
{
  "type": "offset",
  "properties": {
    "limitParameter": "limit",
    "offsetParameter": "skip",
    "pageSize": 50,
    "type": "query",
    "rootProperty": "results"
  }
}
```

### IN8nRequestOperationPaginationGeneric

```typescript
interface IN8nRequestOperationPaginationGeneric {
  type: 'generic';
  properties: {
    // Continue while this expression is true
    continue: string | boolean;

    // Request configuration for subsequent pages
    request: {
      [key: string]: string | IDataObject;
    };
  };
}
```

**Example:**
```json
{
  "type": "generic",
  "properties": {
    "continue": "={{ $response.body.hasMore }}",
    "request": {
      "qs": {
        "cursor": "={{ $response.body.nextCursor }}"
      }
    }
  }
}
```

---

## Custom Pagination Patterns
<!-- chunk: 19-custom | keywords: custom, patterns -->

### Pattern 1: Cursor-Based Paginator Function

```typescript
export const getCursorPaginator = () => {
  return async function cursorPagination(
    this: IExecutePaginationFunctions,
    requestOptions: DeclarativeRestApiSettings.ResultOptions,
  ): Promise<INodeExecutionData[]> {
    let executions: INodeExecutionData[] = [];
    let nextCursor: string | undefined;
    const returnAll = this.getNodeParameter('returnAll', true) as boolean;

    do {
      if (nextCursor) {
        requestOptions.options.qs = {
          ...requestOptions.options.qs,
          cursor: nextCursor,
        };
      }

      const response = await this.helpers.httpRequest(requestOptions.options);
      const items = response.data || [];
      executions.push(...items.map((item: IDataObject) => ({ json: item })));
      nextCursor = response.nextCursor;

      if (!returnAll) {
        const limit = this.getNodeParameter('limit', 100) as number;
        if (executions.length >= limit) {
          return executions.slice(0, limit);
        }
      }

    } while (nextCursor);

    return executions;
  };
};
```

### Pattern 2: apiRequestAllItems Helper

```typescript
export async function apiRequestAllItems(
  this: IExecuteFunctions | ILoadOptionsFunctions | IHookFunctions,
  method: IHttpRequestMethods,
  endpoint: string,
  body: IDataObject = {},
  query: IDataObject = {}
): Promise<IDataObject[]> {
  const returnData: IDataObject[] = [];
  let responseData: IDataObject;

  query.limit = query.limit || 100;

  do {
    responseData = await apiRequest.call(this, method, endpoint, body, query);
    returnData.push(...(responseData.data as IDataObject[]));

    if (responseData.nextCursor) {
      query.cursor = responseData.nextCursor;
    }
  } while (responseData.nextCursor);

  return returnData;
}
```

### Pattern 3: Loop-Based Pagination (Workflow)

```json
{
  "nodes": [
    {
      "name": "Initialize",
      "type": "n8n-nodes-base.set",
      "parameters": {
        "assignments": {
          "assignments": [
            { "name": "page", "value": 1, "type": "number" },
            { "name": "hasMore", "value": true, "type": "boolean" },
            { "name": "allItems", "value": "={{ [] }}", "type": "array" }
          ]
        }
      }
    },
    {
      "name": "Fetch Page",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "url": "https://api.example.com/items",
        "qs": {
          "page": "={{ $json.page }}",
          "per_page": 100
        }
      }
    },
    {
      "name": "Check More",
      "type": "n8n-nodes-base.if",
      "parameters": {
        "conditions": {
          "conditions": [{
            "leftValue": "={{ $json.data.length }}",
            "rightValue": 0,
            "operator": { "type": "number", "operation": "gt" }
          }]
        }
      }
    },
    {
      "name": "Increment Page",
      "type": "n8n-nodes-base.set",
      "parameters": {
        "assignments": {
          "assignments": [
            { "name": "page", "value": "={{ $json.page + 1 }}", "type": "number" }
          ]
        }
      }
    }
  ],
  "connections": {
    "Initialize": { "main": [[{ "node": "Fetch Page" }]] },
    "Fetch Page": { "main": [[{ "node": "Check More" }]] },
    "Check More": {
      "main": [
        [{ "node": "Increment Page" }],
        [{ "node": "Done" }]
      ]
    },
    "Increment Page": { "main": [[{ "node": "Fetch Page" }]] }
  }
}
```

---

## Node Examples
<!-- chunk: 19-nodes | keywords: nodes, examples -->

### Google Sheets - Return All

```json
{
  "type": "n8n-nodes-base.googleSheets",
  "parameters": {
    "operation": "read",
    "documentId": "...",
    "sheetName": "Sheet1",
    "options": {
      "returnAll": true
    }
  }
}
```

### Airtable - With Limit

```json
{
  "type": "n8n-nodes-base.airtable",
  "parameters": {
    "operation": "list",
    "application": "...",
    "table": "...",
    "returnAll": false,
    "limit": 50,
    "options": {
      "sort": {
        "sort": [
          { "field": "Created", "direction": "desc" }
        ]
      }
    }
  }
}
```

### PostgreSQL - Offset Pagination

```json
{
  "type": "n8n-nodes-base.postgres",
  "parameters": {
    "operation": "executeQuery",
    "query": "SELECT * FROM users ORDER BY id LIMIT $1 OFFSET $2",
    "options": {
      "queryParams": "100, {{ $json.offset || 0 }}"
    }
  }
}
```

### HubSpot - Cursor Pagination (Built-in)

```json
{
  "type": "n8n-nodes-base.hubspot",
  "parameters": {
    "resource": "contact",
    "operation": "getAll",
    "returnAll": true,
    "options": {
      "properties": ["email", "firstname", "lastname"]
    }
  }
}
```

---

## Best Practices
<!-- chunk: 19-best-practices | keywords: best, practices, tips -->

1. **Use `returnAll` with caution** - Large datasets can cause memory issues
2. **Set reasonable `maxResults`** - Safety limit for runaway pagination
3. **Implement rate limiting** - Add delays between requests if needed
4. **Handle empty pages** - Some APIs return empty arrays before null cursor
5. **Cache cursor state** - For resumable pagination across executions
6. **Monitor execution time** - Long pagination can timeout

### Rate-Limited Pagination

```javascript
// In Code node
const items = [];
let cursor = null;
const delay = 200; // ms between requests

do {
  const response = await this.helpers.httpRequest({
    method: 'GET',
    url: 'https://api.example.com/items',
    qs: { cursor, limit: 100 }
  });

  items.push(...response.data);
  cursor = response.nextCursor;

  if (cursor) {
    await new Promise(resolve => setTimeout(resolve, delay));
  }
} while (cursor && items.length < 10000);

return items.map(item => ({ json: item }));
```

---

*Source: packages/workflow/src/interfaces.ts, packages/nodes-base/nodes/N8n/GenericFunctions.ts*
