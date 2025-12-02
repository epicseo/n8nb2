# 17_CONNECTION_PATTERNS.md
<!-- repo: n8n | version: 1.122.0 | generated: 2025-12-02 -->
<!-- tags: connections, patterns, routing, ai, workflow -->

## Contents
- [Overview](#overview)
- [Connection Types](#connection-types)
- [Connection JSON Structure](#connection-json-structure)
- [Main Connection Patterns](#main-connection-patterns)
- [AI Connection Patterns](#ai-connection-patterns)
- [Complex Workflow Patterns](#complex-workflow-patterns)
- [Best Practices](#best-practices)

---

## Overview
<!-- chunk: 17-overview | keywords: connections, types, routing -->

### Connection System
n8n workflows are directed graphs where nodes connect via typed connections. Each connection has:
- **Source node** - The node producing output
- **Source output type** - The connection type (e.g., `main`, `ai_languageModel`)
- **Source output index** - Which output port (0-based)
- **Target node** - The node receiving input
- **Target input type** - Must match source output type
- **Target input index** - Which input port (0-based)

### Connection Types Summary
| Type | Purpose | Max Connections |
|------|---------|-----------------|
| `main` | Standard data flow | Unlimited |
| `ai_languageModel` | LLM provider | 1 |
| `ai_memory` | Conversation memory | 1 |
| `ai_tool` | Agent tools | Unlimited |
| `ai_vectorStore` | Vector database | 1 |
| `ai_retriever` | Document retrieval | 1 |
| `ai_outputParser` | Output formatting | 1 |
| `ai_textSplitter` | Text chunking | 1 |
| `ai_embedding` | Embedding model | 1 |
| `ai_document` | Document loader | Unlimited |
| `ai_agent` | Sub-agents | Unlimited |
| `ai_chain` | LangChain chains | 1 |

---

## Connection Types
<!-- chunk: 17-connection-types | keywords: main, ai, types -->

### NodeConnectionTypes Enum
From `packages/workflow/src/interfaces.ts`:

```typescript
export const NodeConnectionTypes = {
  // Standard data flow
  Main: 'main',

  // AI/LangChain connections
  AiAgent: 'ai_agent',
  AiChain: 'ai_chain',
  AiDocument: 'ai_document',
  AiEmbedding: 'ai_embedding',
  AiLanguageModel: 'ai_languageModel',
  AiMemory: 'ai_memory',
  AiOutputParser: 'ai_outputParser',
  AiReranker: 'ai_reranker',
  AiRetriever: 'ai_retriever',
  AiTextSplitter: 'ai_textSplitter',
  AiTool: 'ai_tool',
  AiVectorStore: 'ai_vectorStore',
} as const;
```

### Main Connection (`main`)
Standard data flow between nodes. Carries `INodeExecutionData[]` items.

**Characteristics:**
- Most common connection type
- Supports multiple connections from same output
- Supports multiple outputs per node (If, Switch)
- Items flow as arrays of JSON objects

### AI Language Model (`ai_languageModel`)
Connects LLM providers to consumers.

**Source Nodes:**
- `@n8n/n8n-nodes-langchain.lmChatOpenAi`
- `@n8n/n8n-nodes-langchain.lmChatAnthropic`
- `@n8n/n8n-nodes-langchain.lmChatGoogleGemini`
- `@n8n/n8n-nodes-langchain.lmChatOllama`

**Target Nodes:**
- `@n8n/n8n-nodes-langchain.agent`
- `@n8n/n8n-nodes-langchain.chainLlm`
- `@n8n/n8n-nodes-langchain.chainSummarization`

### AI Memory (`ai_memory`)
Provides conversation history storage.

**Source Nodes:**
- `@n8n/n8n-nodes-langchain.memoryBufferWindow`
- `@n8n/n8n-nodes-langchain.memoryRedis`
- `@n8n/n8n-nodes-langchain.memoryPostgres`

**Target Nodes:**
- `@n8n/n8n-nodes-langchain.agent`
- `@n8n/n8n-nodes-langchain.chatTrigger`

### AI Tool (`ai_tool`)
Provides tools for AI agents to use.

**Source Nodes:**
- `@n8n/n8n-nodes-langchain.toolCalculator`
- `@n8n/n8n-nodes-langchain.toolCode`
- `@n8n/n8n-nodes-langchain.toolHttpRequest`
- `@n8n/n8n-nodes-langchain.toolWorkflow`
- `@n8n/n8n-nodes-langchain.toolWikipedia`

**Target Nodes:**
- `@n8n/n8n-nodes-langchain.agent`

### AI Vector Store (`ai_vectorStore`)
Vector database for RAG applications.

**Source Nodes:**
- `@n8n/n8n-nodes-langchain.vectorStorePinecone`
- `@n8n/n8n-nodes-langchain.vectorStoreQdrant`
- `@n8n/n8n-nodes-langchain.vectorStoreSupabase`
- `@n8n/n8n-nodes-langchain.vectorStoreInMemory`

**Target Nodes:**
- `@n8n/n8n-nodes-langchain.retrieverVectorStore`

---

## Connection JSON Structure
<!-- chunk: 17-json-structure | keywords: json, format, connections -->

### IConnections Interface
```typescript
interface IConnections {
  [sourceNodeName: string]: {
    [connectionType: string]: Array<Array<IConnection>>;
  };
}

interface IConnection {
  node: string;      // Target node name
  type: string;      // Target input type
  index: number;     // Target input index
}
```

### Basic Connection Structure
```json
{
  "connections": {
    "Source Node Name": {
      "main": [
        [
          {
            "node": "Target Node Name",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  }
}
```

### Understanding the Array Structure
```json
{
  "connections": {
    "NodeA": {
      "main": [           // Connection type
        [                 // Output index 0
          {               // Connection 1 from output 0
            "node": "NodeB",
            "type": "main",
            "index": 0
          },
          {               // Connection 2 from output 0 (parallel)
            "node": "NodeC",
            "type": "main",
            "index": 0
          }
        ],
        [                 // Output index 1 (for nodes like If, Switch)
          {
            "node": "NodeD",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  }
}
```

---

## Main Connection Patterns
<!-- chunk: 17-main-patterns | keywords: linear, branch, merge, loop -->

### Pattern 1: Linear Flow
Simple sequential execution.

```
Trigger → Node1 → Node2 → Node3
```

```json
{
  "connections": {
    "Trigger": {
      "main": [[{ "node": "Node1", "type": "main", "index": 0 }]]
    },
    "Node1": {
      "main": [[{ "node": "Node2", "type": "main", "index": 0 }]]
    },
    "Node2": {
      "main": [[{ "node": "Node3", "type": "main", "index": 0 }]]
    }
  }
}
```

### Pattern 2: Parallel Branches (Fan-out)
One node sends to multiple nodes.

```
         ┌→ Branch A
Trigger ─┼→ Branch B
         └→ Branch C
```

```json
{
  "connections": {
    "Trigger": {
      "main": [
        [
          { "node": "Branch A", "type": "main", "index": 0 },
          { "node": "Branch B", "type": "main", "index": 0 },
          { "node": "Branch C", "type": "main", "index": 0 }
        ]
      ]
    }
  }
}
```

### Pattern 3: Conditional Branching (If Node)
If node has two outputs: true (index 0) and false (index 1).

```
           true→ Success Path
Trigger → If
           false→ Failure Path
```

```json
{
  "connections": {
    "Trigger": {
      "main": [[{ "node": "If", "type": "main", "index": 0 }]]
    },
    "If": {
      "main": [
        [{ "node": "Success Path", "type": "main", "index": 0 }],
        [{ "node": "Failure Path", "type": "main", "index": 0 }]
      ]
    }
  }
}
```

### Pattern 4: Multi-way Routing (Switch Node)
Switch node has dynamic outputs based on rules.

```
              case1→ Handler 1
Trigger → Switch─case2→ Handler 2
              fallback→ Default Handler
```

```json
{
  "connections": {
    "Trigger": {
      "main": [[{ "node": "Switch", "type": "main", "index": 0 }]]
    },
    "Switch": {
      "main": [
        [{ "node": "Handler 1", "type": "main", "index": 0 }],
        [{ "node": "Handler 2", "type": "main", "index": 0 }],
        [{ "node": "Default Handler", "type": "main", "index": 0 }]
      ]
    }
  }
}
```

### Pattern 5: Merge (Fan-in)
Multiple branches converge to one node.

```
Branch A ─┐
Branch B ─┼→ Merge → Continue
Branch C ─┘
```

```json
{
  "connections": {
    "Branch A": {
      "main": [[{ "node": "Merge", "type": "main", "index": 0 }]]
    },
    "Branch B": {
      "main": [[{ "node": "Merge", "type": "main", "index": 1 }]]
    },
    "Branch C": {
      "main": [[{ "node": "Merge", "type": "main", "index": 2 }]]
    },
    "Merge": {
      "main": [[{ "node": "Continue", "type": "main", "index": 0 }]]
    }
  }
}
```

### Pattern 6: Loop (Split In Batches)
Process items in batches with loop-back.

```
                 done→ Complete
Trigger → Split ─┐
            ↑    loop→ Process → (back to Split)
            └────────────────────┘
```

```json
{
  "connections": {
    "Trigger": {
      "main": [[{ "node": "Split In Batches", "type": "main", "index": 0 }]]
    },
    "Split In Batches": {
      "main": [
        [{ "node": "Complete", "type": "main", "index": 0 }],
        [{ "node": "Process", "type": "main", "index": 0 }]
      ]
    },
    "Process": {
      "main": [[{ "node": "Split In Batches", "type": "main", "index": 0 }]]
    }
  }
}
```

### Pattern 7: Diamond (Branch and Merge)
Branch out and merge back.

```
           ┌→ Transform A ─┐
Trigger → If              Merge → Output
           └→ Transform B ─┘
```

```json
{
  "connections": {
    "Trigger": {
      "main": [[{ "node": "If", "type": "main", "index": 0 }]]
    },
    "If": {
      "main": [
        [{ "node": "Transform A", "type": "main", "index": 0 }],
        [{ "node": "Transform B", "type": "main", "index": 0 }]
      ]
    },
    "Transform A": {
      "main": [[{ "node": "Merge", "type": "main", "index": 0 }]]
    },
    "Transform B": {
      "main": [[{ "node": "Merge", "type": "main", "index": 1 }]]
    },
    "Merge": {
      "main": [[{ "node": "Output", "type": "main", "index": 0 }]]
    }
  }
}
```

---

## AI Connection Patterns
<!-- chunk: 17-ai-patterns | keywords: agent, rag, langchain -->

### Pattern 1: Basic AI Agent
Agent with LLM and memory.

```
Chat Trigger ────────→ AI Agent → Response
                          ↑
OpenAI Chat Model ────────┤ (ai_languageModel)
                          │
Buffer Memory ────────────┘ (ai_memory)
```

```json
{
  "connections": {
    "Chat Trigger": {
      "main": [[{ "node": "AI Agent", "type": "main", "index": 0 }]]
    },
    "OpenAI Chat Model": {
      "ai_languageModel": [[{ "node": "AI Agent", "type": "ai_languageModel", "index": 0 }]]
    },
    "Buffer Memory": {
      "ai_memory": [[{ "node": "AI Agent", "type": "ai_memory", "index": 0 }]]
    },
    "AI Agent": {
      "main": [[{ "node": "Response", "type": "main", "index": 0 }]]
    }
  }
}
```

### Pattern 2: Agent with Tools
Agent with multiple tools.

```
Chat Trigger ────────→ AI Agent → Response
                          ↑
OpenAI Chat Model ────────┤ (ai_languageModel)
                          │
Calculator Tool ──────────┤ (ai_tool)
                          │
HTTP Request Tool ────────┤ (ai_tool)
                          │
Wikipedia Tool ───────────┘ (ai_tool)
```

```json
{
  "connections": {
    "Chat Trigger": {
      "main": [[{ "node": "AI Agent", "type": "main", "index": 0 }]]
    },
    "OpenAI Chat Model": {
      "ai_languageModel": [[{ "node": "AI Agent", "type": "ai_languageModel", "index": 0 }]]
    },
    "Calculator Tool": {
      "ai_tool": [[{ "node": "AI Agent", "type": "ai_tool", "index": 0 }]]
    },
    "HTTP Request Tool": {
      "ai_tool": [[{ "node": "AI Agent", "type": "ai_tool", "index": 0 }]]
    },
    "Wikipedia Tool": {
      "ai_tool": [[{ "node": "AI Agent", "type": "ai_tool", "index": 0 }]]
    }
  }
}
```

### Pattern 3: RAG Pipeline
Retrieval-Augmented Generation workflow.

```
Document Loader → Vector Store Insert
                       ↑
Text Splitter ─────────┤ (ai_textSplitter)
                       │
Embedding Model ───────┘ (ai_embedding)

Query Trigger → Vector Store Retriever → AI Agent → Response
                       ↑                     ↑
Vector Store ──────────┘ (ai_vectorStore)    │
                                             │
OpenAI Chat Model ───────────────────────────┘ (ai_languageModel)
```

**Document Indexing:**
```json
{
  "connections": {
    "Document Loader": {
      "ai_document": [[{ "node": "Vector Store Insert", "type": "ai_document", "index": 0 }]]
    },
    "Text Splitter": {
      "ai_textSplitter": [[{ "node": "Vector Store Insert", "type": "ai_textSplitter", "index": 0 }]]
    },
    "Embedding Model": {
      "ai_embedding": [[{ "node": "Vector Store Insert", "type": "ai_embedding", "index": 0 }]]
    }
  }
}
```

**Query Pipeline:**
```json
{
  "connections": {
    "Query Trigger": {
      "main": [[{ "node": "Vector Store Retriever", "type": "main", "index": 0 }]]
    },
    "Vector Store": {
      "ai_vectorStore": [[{ "node": "Vector Store Retriever", "type": "ai_vectorStore", "index": 0 }]]
    },
    "Vector Store Retriever": {
      "main": [[{ "node": "AI Agent", "type": "main", "index": 0 }]]
    },
    "OpenAI Chat Model": {
      "ai_languageModel": [[{ "node": "AI Agent", "type": "ai_languageModel", "index": 0 }]]
    }
  }
}
```

### Pattern 4: Summarization Chain
Chain for summarizing documents.

```
Document Input → Summarization Chain → Summary Output
                       ↑
OpenAI Chat Model ─────┘ (ai_languageModel)
```

```json
{
  "connections": {
    "Document Input": {
      "main": [[{ "node": "Summarization Chain", "type": "main", "index": 0 }]]
    },
    "OpenAI Chat Model": {
      "ai_languageModel": [[{ "node": "Summarization Chain", "type": "ai_languageModel", "index": 0 }]]
    },
    "Summarization Chain": {
      "main": [[{ "node": "Summary Output", "type": "main", "index": 0 }]]
    }
  }
}
```

---

## Complex Workflow Patterns
<!-- chunk: 17-complex-patterns | keywords: webhook, error, retry -->

### Pattern 1: Webhook with Response
Receive webhook, process, respond.

```
Webhook ─→ Process ─→ Respond to Webhook
```

```json
{
  "nodes": [
    {
      "name": "Webhook",
      "type": "n8n-nodes-base.webhook",
      "parameters": {
        "httpMethod": "POST",
        "path": "my-webhook",
        "responseMode": "responseNode"
      }
    },
    {
      "name": "Respond to Webhook",
      "type": "n8n-nodes-base.respondToWebhook",
      "parameters": {
        "respondWith": "json",
        "responseBody": "={{ $json }}"
      }
    }
  ],
  "connections": {
    "Webhook": {
      "main": [[{ "node": "Process", "type": "main", "index": 0 }]]
    },
    "Process": {
      "main": [[{ "node": "Respond to Webhook", "type": "main", "index": 0 }]]
    }
  }
}
```

### Pattern 2: Error Handling with Retry
Handle errors with retry logic.

```
Trigger → HTTP Request ─success→ Process
               │
               error→ Wait → (back to HTTP Request)
```

Using node settings:
```json
{
  "name": "HTTP Request",
  "type": "n8n-nodes-base.httpRequest",
  "retryOnFail": true,
  "maxTries": 3,
  "waitBetweenTries": 2000,
  "continueOnFail": false
}
```

### Pattern 3: Parallel Processing with Aggregation
Process items in parallel and aggregate results.

```
Trigger → Split ─→ Process 1 ─┐
               ─→ Process 2 ─┼→ Merge → Aggregate
               ─→ Process 3 ─┘
```

```json
{
  "connections": {
    "Trigger": {
      "main": [[{ "node": "Split", "type": "main", "index": 0 }]]
    },
    "Split": {
      "main": [
        [
          { "node": "Process 1", "type": "main", "index": 0 },
          { "node": "Process 2", "type": "main", "index": 0 },
          { "node": "Process 3", "type": "main", "index": 0 }
        ]
      ]
    },
    "Process 1": {
      "main": [[{ "node": "Merge", "type": "main", "index": 0 }]]
    },
    "Process 2": {
      "main": [[{ "node": "Merge", "type": "main", "index": 1 }]]
    },
    "Process 3": {
      "main": [[{ "node": "Merge", "type": "main", "index": 2 }]]
    },
    "Merge": {
      "main": [[{ "node": "Aggregate", "type": "main", "index": 0 }]]
    }
  }
}
```

### Pattern 4: Daily Report Generator
Scheduled workflow with data aggregation.

```
Schedule → Fetch Data → Transform → Generate Report → Send Email
```

```json
{
  "connections": {
    "Schedule Trigger": {
      "main": [[{ "node": "Fetch Data", "type": "main", "index": 0 }]]
    },
    "Fetch Data": {
      "main": [[{ "node": "Transform", "type": "main", "index": 0 }]]
    },
    "Transform": {
      "main": [[{ "node": "Generate Report", "type": "main", "index": 0 }]]
    },
    "Generate Report": {
      "main": [[{ "node": "Send Email", "type": "main", "index": 0 }]]
    }
  }
}
```

---

## Best Practices
<!-- chunk: 17-best-practices | keywords: tips, optimization, design -->

### Connection Design

1. **Keep flows readable**
   - Use NoOp nodes for organization
   - Add node notes for documentation
   - Position nodes left-to-right

2. **Minimize connection crossings**
   - Arrange parallel branches vertically
   - Use merge nodes to consolidate

3. **Handle errors appropriately**
   - Set `continueOnFail` for non-critical nodes
   - Use try/catch patterns with If nodes

### Performance Considerations

1. **Avoid unnecessary branches**
   - Each branch creates execution overhead
   - Combine operations when possible

2. **Use batch processing**
   - Split In Batches for large datasets
   - Prevents memory issues

3. **AI connection constraints**
   - Most AI inputs allow only 1 connection (LLM, Memory, Output Parser)
   - Tools allow multiple connections

### Node Positioning Guidelines

```
Standard spacing:
- X increment: 250 pixels
- Y increment: 150 pixels (for branches)

Example positions:
- Trigger: [250, 300]
- Node 1: [500, 300]
- Node 2: [750, 300]
- Branch A: [750, 150]
- Branch B: [750, 450]
- Merge: [1000, 300]
```

### Connection Validation

1. **Type matching** - Source and target types must match
2. **Max connections** - Respect `maxConnections` limits
3. **Required inputs** - Ensure required inputs are connected
4. **Cycle detection** - Avoid infinite loops (except SplitInBatches)

---

*Generated from n8n v1.122.0 - packages/workflow/src*
