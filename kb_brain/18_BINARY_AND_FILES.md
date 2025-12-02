# 18_BINARY_AND_FILES.md
<!-- repo: n8n | version: 1.122.0 | generated: 2025-12-02 -->
<!-- tags: binary, files, upload, download, streams, base64 -->

## Contents
- [Overview](#overview)
- [IBinaryData Interface](#ibinarydata-interface)
- [Binary Helper Functions](#binary-helper-functions)
- [File Operations Patterns](#file-operations-patterns)
- [Node Examples](#node-examples)
- [JSON Structure](#json-structure)

---

## Overview
<!-- chunk: 18-overview | keywords: binary, files, data -->

n8n handles binary data (files, images, documents) through the `IBinaryData` interface. Binary data is:
- Stored separately from JSON data
- Referenced by ID in the data store
- Streamed for large files
- Base64 encoded for small transfers

---

## IBinaryData Interface
<!-- chunk: 18-interface | keywords: IBinaryData, structure, properties -->

```typescript
interface IBinaryData {
  // Base64 encoded data (for small files) or reference
  data: string;

  // MIME type of the file
  mimeType: string;

  // File type classification
  fileType?: 'text' | 'json' | 'image' | 'audio' | 'video' | 'pdf' | 'html';

  // Original filename
  fileName?: string;

  // Directory path (if applicable)
  directory?: string;

  // File extension without dot
  fileExtension?: string;

  // Human-readable file size
  fileSize?: string;

  // Reference ID for stored binary data
  id?: string;
}
```

### Binary Data in Items

```json
{
  "json": {
    "name": "document.pdf",
    "processedAt": "2024-01-01T00:00:00Z"
  },
  "binary": {
    "data": {
      "data": "base64-encoded-content-or-reference",
      "mimeType": "application/pdf",
      "fileType": "pdf",
      "fileName": "document.pdf",
      "fileExtension": "pdf",
      "fileSize": "1.2 MB",
      "id": "binary-data-uuid"
    },
    "attachment": {
      "data": "another-binary-reference",
      "mimeType": "image/png",
      "fileName": "image.png"
    }
  }
}
```

---

## Binary Helper Functions
<!-- chunk: 18-helpers | keywords: prepareBinaryData, getBinaryStream, buffer -->

### Available in Node Execution Context

```typescript
// Prepare binary data from buffer
const binaryData = await this.helpers.prepareBinaryData(
  buffer: Buffer,
  fileName?: string,
  mimeType?: string
): Promise<IBinaryData>;

// Example
const pdfBuffer = Buffer.from(pdfContent, 'base64');
const binary = await this.helpers.prepareBinaryData(
  pdfBuffer,
  'report.pdf',
  'application/pdf'
);

// Set binary data buffer (update existing)
const updatedBinary = await this.helpers.setBinaryDataBuffer(
  binaryData: IBinaryData,
  newBuffer: Buffer
): Promise<IBinaryData>;

// Get binary as readable stream
const stream = await this.helpers.getBinaryStream(
  binaryDataId: string
): Promise<Readable>;

// Convert stream to buffer
const buffer = await this.helpers.binaryToBuffer(
  stream: Readable
): Promise<Buffer>;

// Convert binary to string
const text = await this.helpers.binaryToString(
  stream: Readable,
  encoding?: BufferEncoding  // 'utf8', 'base64', etc.
): Promise<string>;

// Check if file path is blocked
const isBlocked = await this.helpers.isFilePathBlocked(
  filePath: string
): Promise<boolean>;

// Create read stream from file
const readStream = await this.helpers.createReadStream(
  filePath: string
): Promise<Readable>;

// Get storage path for workflow
const storagePath = this.helpers.getStoragePath(): string;

// Write content to file
await this.helpers.writeContentToFile(
  filePath: string,
  content: string | Buffer,
  flag?: string  // 'w' for write, 'a' for append
): Promise<void>;
```

---

## File Operations Patterns
<!-- chunk: 18-patterns | keywords: upload, download, convert -->

### Pattern 1: Download File from URL

```json
{
  "type": "n8n-nodes-base.httpRequest",
  "parameters": {
    "method": "GET",
    "url": "https://example.com/file.pdf",
    "options": {
      "response": {
        "response": {
          "responseFormat": "file"
        }
      }
    }
  }
}
```

Output includes binary data:
```json
{
  "json": {},
  "binary": {
    "data": {
      "mimeType": "application/pdf",
      "fileName": "file.pdf",
      "data": "..."
    }
  }
}
```

### Pattern 2: Upload File (Multipart)

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

### Pattern 3: Read Binary from Previous Node

```javascript
// In Code node
const binaryData = $input.item.binary.data;

// Get as buffer
const buffer = await this.helpers.getBinaryDataBuffer(
  $input.itemIndex,
  'data'  // binary property name
);

// Get as string
const content = buffer.toString('utf8');

return [{ json: { content } }];
```

### Pattern 4: Create Binary from Text/JSON

```javascript
// In Code node
const jsonContent = JSON.stringify({ key: 'value' }, null, 2);
const buffer = Buffer.from(jsonContent, 'utf8');

const binaryData = await this.helpers.prepareBinaryData(
  buffer,
  'data.json',
  'application/json'
);

return [{
  json: { created: true },
  binary: { file: binaryData }
}];
```

### Pattern 5: Convert Between Formats

```json
{
  "type": "n8n-nodes-base.spreadsheetFile",
  "parameters": {
    "operation": "fromJson",
    "fileFormat": "xlsx",
    "options": {
      "fileName": "export.xlsx",
      "sheetName": "Data"
    }
  }
}
```

### Pattern 6: Compress/Decompress

```json
{
  "type": "n8n-nodes-base.compression",
  "parameters": {
    "operation": "compress",
    "binaryPropertyName": "data",
    "outputFormat": "zip",
    "fileName": "archive.zip"
  }
}
```

---

## Node Examples
<!-- chunk: 18-nodes | keywords: nodes, examples -->

### Read Binary File

```json
{
  "type": "n8n-nodes-base.readBinaryFile",
  "parameters": {
    "filePath": "/path/to/file.pdf",
    "options": {}
  }
}
```

### Write Binary File

```json
{
  "type": "n8n-nodes-base.writeBinaryFile",
  "parameters": {
    "fileName": "/output/result.pdf",
    "dataPropertyName": "data",
    "options": {
      "append": false
    }
  }
}
```

### Move Binary File

```json
{
  "type": "n8n-nodes-base.moveBinaryData",
  "parameters": {
    "mode": "binaryToJson",
    "sourceKey": "data",
    "options": {
      "encoding": "base64",
      "fileName": true,
      "mimeType": true
    }
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
    "options": {
      "headerRow": true,
      "sheetName": "Sheet1",
      "range": "A1:Z100"
    }
  }
}
```

### Extract from PDF

```json
{
  "type": "n8n-nodes-base.extractFromFile",
  "parameters": {
    "operation": "pdf",
    "binaryPropertyName": "data",
    "options": {
      "joinPages": true,
      "maxPages": 10
    }
  }
}
```

---

## JSON Structure
<!-- chunk: 18-json | keywords: json, workflow, binary -->

### Binary in Workflow JSON

```json
{
  "nodes": [
    {
      "name": "HTTP Request",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "url": "https://example.com/image.png",
        "options": {
          "response": {
            "response": {
              "responseFormat": "file",
              "outputPropertyName": "image"
            }
          }
        }
      }
    },
    {
      "name": "Write File",
      "type": "n8n-nodes-base.writeBinaryFile",
      "parameters": {
        "fileName": "={{ $json.fileName || 'output.png' }}",
        "dataPropertyName": "image"
      }
    }
  ],
  "connections": {
    "HTTP Request": {
      "main": [[{ "node": "Write File", "type": "main", "index": 0 }]]
    }
  }
}
```

### Accessing Binary in Expressions

```javascript
// Get binary property names
{{ Object.keys($binary) }}

// Check if binary exists
{{ $binary.data ? 'Has file' : 'No file' }}

// Get filename
{{ $binary.data.fileName }}

// Get MIME type
{{ $binary.data.mimeType }}

// Get file size
{{ $binary.data.fileSize }}
```

---

## Common MIME Types

| Extension | MIME Type |
|-----------|-----------|
| .json | application/json |
| .pdf | application/pdf |
| .xml | application/xml |
| .zip | application/zip |
| .xlsx | application/vnd.openxmlformats-officedocument.spreadsheetml.sheet |
| .docx | application/vnd.openxmlformats-officedocument.wordprocessingml.document |
| .csv | text/csv |
| .txt | text/plain |
| .html | text/html |
| .png | image/png |
| .jpg | image/jpeg |
| .gif | image/gif |
| .svg | image/svg+xml |
| .mp3 | audio/mpeg |
| .mp4 | video/mp4 |

---

*Source: packages/workflow/src/interfaces.ts, packages/nodes-base/utils/binary.ts*
