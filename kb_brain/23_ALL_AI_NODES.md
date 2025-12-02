# 23_ALL_AI_NODES.md
<!-- repo: n8n | version: 1.122.0 | generated: 2025-12-02 -->
<!-- tags: ai, langchain, llm, agents, memory, tools, vectors -->

## Contents
- [Overview](#overview)
- [Language Models](#language-models)
- [Chat Models](#chat-models)
- [Agents](#agents)
- [Memory](#memory)
- [Tools](#tools)
- [Vector Stores](#vector-stores)
- [Embeddings](#embeddings)
- [Document Loaders](#document-loaders)
- [Text Splitters](#text-splitters)
- [Retrievers](#retrievers)
- [Chains](#chains)
- [Output Parsers](#output-parsers)

---

## Overview
<!-- chunk: 23-overview | keywords: ai, langchain, count -->

### AI Node Statistics
- **Total AI Nodes:** 118
- **Package:** `@n8n/n8n-nodes-langchain`
- **Connection Types:** 13 AI-specific types

### AI Connection Types
| Type | Purpose | Max Connections |
|------|---------|-----------------|
| `ai_languageModel` | LLM provider | 1 |
| `ai_memory` | Conversation history | 1 |
| `ai_tool` | Agent tools | Unlimited |
| `ai_vectorStore` | Vector database | 1 |
| `ai_retriever` | Document retrieval | 1 |
| `ai_outputParser` | Output formatting | 1 |
| `ai_textSplitter` | Text chunking | 1 |
| `ai_embedding` | Embedding model | 1 |
| `ai_document` | Document loader | Unlimited |
| `ai_agent` | Sub-agents | Unlimited |
| `ai_chain` | LangChain chains | 1 |
| `ai_reranker` | Result reranking | 1 |

---

## Language Models
<!-- chunk: 23-llm | keywords: llm, models, providers -->

### OpenAI Chat Model
**Type:** `@n8n/n8n-nodes-langchain.lmChatOpenAi`
**Version:** 1.2

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
      "presencePenalty": 0,
      "timeout": 60000
    }
  },
  "credentials": {
    "openAiApi": { "id": "1", "name": "OpenAI" }
  }
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
    "options": {
      "temperature": 0.7,
      "maxTokensToSample": 4096,
      "topP": 1,
      "topK": 40
    }
  }
}
```

**Models:** `claude-sonnet-4-20250514`, `claude-3-5-sonnet-20241022`, `claude-3-opus-20240229`, `claude-3-haiku-20240307`

### Google Gemini Chat Model
**Type:** `@n8n/n8n-nodes-langchain.lmChatGoogleGemini`

```json
{
  "type": "@n8n/n8n-nodes-langchain.lmChatGoogleGemini",
  "parameters": {
    "model": "gemini-1.5-pro",
    "options": {
      "temperature": 0.7,
      "maxOutputTokens": 2048,
      "topP": 0.95,
      "topK": 40
    }
  }
}
```

### Google Vertex AI
**Type:** `@n8n/n8n-nodes-langchain.lmChatGoogleVertex`

### Azure OpenAI Chat Model
**Type:** `@n8n/n8n-nodes-langchain.lmChatAzureOpenAi`

```json
{
  "type": "@n8n/n8n-nodes-langchain.lmChatAzureOpenAi",
  "parameters": {
    "model": "gpt-4",
    "options": {
      "temperature": 0.7
    }
  },
  "credentials": {
    "azureOpenAiApi": { "id": "1", "name": "Azure OpenAI" }
  }
}
```

### Ollama Chat Model
**Type:** `@n8n/n8n-nodes-langchain.lmChatOllama`

```json
{
  "type": "@n8n/n8n-nodes-langchain.lmChatOllama",
  "parameters": {
    "model": "llama3.2",
    "baseUrl": "http://localhost:11434",
    "options": {
      "temperature": 0.7,
      "numCtx": 4096,
      "numPredict": 2048
    }
  }
}
```

### Other LLM Providers
- `lmChatGroq` - Groq (fast inference)
- `lmChatMistral` - Mistral AI
- `lmChatCohere` - Cohere
- `lmChatHuggingFaceInference` - Hugging Face
- `lmChatAwsBedrock` - AWS Bedrock
- `lmChatOpenRouter` - OpenRouter (multi-model)
- `lmChatDeepSeek` - DeepSeek
- `lmChatXAiGrok` - xAI Grok
- `lmChatLemonade` - Lemonade

---

## Agents
<!-- chunk: 23-agents | keywords: agent, tools, reasoning -->

### AI Agent
**Type:** `@n8n/n8n-nodes-langchain.agent`
**Version:** 1.7

```json
{
  "type": "@n8n/n8n-nodes-langchain.agent",
  "typeVersion": 1.7,
  "parameters": {
    "agent": "conversationalAgent",
    "promptType": "define",
    "text": "={{ $json.chatInput }}",
    "options": {
      "systemMessage": "You are a helpful assistant that can use tools to accomplish tasks.",
      "maxIterations": 10,
      "returnIntermediateSteps": false
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

**Inputs:**
- `main` (required) - Chat input
- `ai_languageModel` (required, max 1) - LLM
- `ai_memory` (optional, max 1) - Memory
- `ai_tool` (optional, unlimited) - Tools
- `ai_outputParser` (optional, max 1) - Parser

### OpenAI Assistant
**Type:** `@n8n/n8n-nodes-langchain.openAiAssistant`

```json
{
  "type": "@n8n/n8n-nodes-langchain.openAiAssistant",
  "parameters": {
    "assistantId": "asst_xxx",
    "options": {
      "baseURL": "",
      "maxRetries": 2
    }
  }
}
```

---

## Memory
<!-- chunk: 23-memory | keywords: memory, conversation, history -->

### Buffer Window Memory
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

**Session ID Types:**
- `fromInput` - From input data
- `customKey` - Custom key name
- `executionId` - Use execution ID

### Redis Memory
**Type:** `@n8n/n8n-nodes-langchain.memoryRedis`

```json
{
  "type": "@n8n/n8n-nodes-langchain.memoryRedis",
  "parameters": {
    "sessionIdType": "fromInput",
    "sessionKey": "sessionId",
    "sessionTTL": 3600
  },
  "credentials": {
    "redis": { "id": "1", "name": "Redis" }
  }
}
```

### PostgreSQL Memory
**Type:** `@n8n/n8n-nodes-langchain.memoryPostgres`

### MongoDB Memory
**Type:** `@n8n/n8n-nodes-langchain.memoryMongodb`

### Zep Memory
**Type:** `@n8n/n8n-nodes-langchain.memoryZep`

### Motorhead Memory
**Type:** `@n8n/n8n-nodes-langchain.memoryMotorhead`

### Xata Memory
**Type:** `@n8n/n8n-nodes-langchain.memoryXata`

### Memory Manager
**Type:** `@n8n/n8n-nodes-langchain.memoryManager`

```json
{
  "type": "@n8n/n8n-nodes-langchain.memoryManager",
  "parameters": {
    "operation": "get",
    "simplifyOutput": true
  }
}
```

**Operations:** `get`, `clear`

---

## Tools
<!-- chunk: 23-tools | keywords: tools, functions, capabilities -->

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
    "name": "custom_processor",
    "description": "Process data with custom logic",
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
    "description": "Call external API",
    "method": "GET",
    "url": "https://api.example.com/data",
    "authentication": "none",
    "placeholderDefinitions": {
      "parameters": [
        {
          "name": "query",
          "description": "Search query",
          "type": "string"
        }
      ]
    }
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
    "workflowId": "workflow-uuid",
    "fields": {
      "values": [
        { "name": "orderId", "description": "Order ID to process" }
      ]
    }
  }
}
```

### Wikipedia Tool
**Type:** `@n8n/n8n-nodes-langchain.toolWikipedia`

### SerpAPI Tool
**Type:** `@n8n/n8n-nodes-langchain.toolSerpApi`

### Wolfram Alpha Tool
**Type:** `@n8n/n8n-nodes-langchain.toolWolframAlpha`

### Vector Store Tool
**Type:** `@n8n/n8n-nodes-langchain.toolVectorStore`

```json
{
  "type": "@n8n/n8n-nodes-langchain.toolVectorStore",
  "parameters": {
    "name": "knowledge_search",
    "description": "Search the knowledge base for relevant information",
    "topK": 5
  }
}
```

### MCP Client Tool
**Type:** `@n8n/n8n-nodes-langchain.mcpClientTool`

---

## Vector Stores
<!-- chunk: 23-vectors | keywords: vector, store, embeddings -->

### Pinecone
**Type:** `@n8n/n8n-nodes-langchain.vectorStorePinecone`

```json
{
  "type": "@n8n/n8n-nodes-langchain.vectorStorePinecone",
  "parameters": {
    "mode": "retrieve",
    "pineconeIndex": "my-index",
    "pineconeNamespace": "default",
    "options": {
      "topK": 5
    }
  },
  "credentials": {
    "pineconeApi": { "id": "1", "name": "Pinecone" }
  }
}
```

**Modes:**
- `retrieve` - Query vectors
- `insert` - Add documents
- `load` - Load as retriever

### Qdrant
**Type:** `@n8n/n8n-nodes-langchain.vectorStoreQdrant`

```json
{
  "type": "@n8n/n8n-nodes-langchain.vectorStoreQdrant",
  "parameters": {
    "mode": "retrieve",
    "qdrantCollection": "my-collection",
    "options": {
      "topK": 5
    }
  }
}
```

### Supabase
**Type:** `@n8n/n8n-nodes-langchain.vectorStoreSupabase`

### MongoDB Atlas
**Type:** `@n8n/n8n-nodes-langchain.vectorStoreMongoDBAtlas`

### Milvus
**Type:** `@n8n/n8n-nodes-langchain.vectorStoreMilvus`

### Weaviate
**Type:** `@n8n/n8n-nodes-langchain.vectorStoreWeaviate`

### PGVector
**Type:** `@n8n/n8n-nodes-langchain.vectorStorePGVector`

### Redis (Vector)
**Type:** `@n8n/n8n-nodes-langchain.vectorStoreRedis`

### In-Memory
**Type:** `@n8n/n8n-nodes-langchain.vectorStoreInMemory`

```json
{
  "type": "@n8n/n8n-nodes-langchain.vectorStoreInMemory",
  "parameters": {
    "mode": "insert"
  }
}
```

### Zep
**Type:** `@n8n/n8n-nodes-langchain.vectorStoreZep`

### Azure AI Search
**Type:** `@n8n/n8n-nodes-langchain.vectorStoreAzureAISearch`

---

## Embeddings
<!-- chunk: 23-embeddings | keywords: embeddings, models -->

### OpenAI Embeddings
**Type:** `@n8n/n8n-nodes-langchain.embeddingsOpenAi`

```json
{
  "type": "@n8n/n8n-nodes-langchain.embeddingsOpenAi",
  "parameters": {
    "model": "text-embedding-3-small",
    "options": {
      "batchSize": 512,
      "stripNewLines": true
    }
  }
}
```

**Models:** `text-embedding-3-small`, `text-embedding-3-large`, `text-embedding-ada-002`

### Azure OpenAI Embeddings
**Type:** `@n8n/n8n-nodes-langchain.embeddingsAzureOpenAi`

### Cohere Embeddings
**Type:** `@n8n/n8n-nodes-langchain.embeddingsCohere`

### Google Embeddings
**Type:** `@n8n/n8n-nodes-langchain.embeddingsGoogle`

### AWS Bedrock Embeddings
**Type:** `@n8n/n8n-nodes-langchain.embeddingsAwsBedrock`

### Ollama Embeddings
**Type:** `@n8n/n8n-nodes-langchain.embeddingsOllama`

```json
{
  "type": "@n8n/n8n-nodes-langchain.embeddingsOllama",
  "parameters": {
    "model": "nomic-embed-text",
    "baseUrl": "http://localhost:11434"
  }
}
```

### Mistral Embeddings
**Type:** `@n8n/n8n-nodes-langchain.embeddingsMistral`

### Hugging Face Embeddings
**Type:** `@n8n/n8n-nodes-langchain.embeddingsHuggingFaceInference`

---

## Document Loaders
<!-- chunk: 23-loaders | keywords: document, loader, input -->

### Default Document Loader
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

**Data Types:** `json`, `binary`

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

### GitHub Document Loader
**Type:** `@n8n/n8n-nodes-langchain.documentGithubLoader`

```json
{
  "type": "@n8n/n8n-nodes-langchain.documentGithubLoader",
  "parameters": {
    "repository": "owner/repo",
    "branch": "main",
    "recursive": true,
    "ignorePaths": "node_modules/**,dist/**"
  }
}
```

### JSON Input Loader
**Type:** `@n8n/n8n-nodes-langchain.documentJsonInputLoader`

---

## Text Splitters
<!-- chunk: 23-splitters | keywords: text, splitter, chunk -->

### Recursive Character Text Splitter
**Type:** `@n8n/n8n-nodes-langchain.textSplitterRecursiveCharacterTextSplitter`

```json
{
  "type": "@n8n/n8n-nodes-langchain.textSplitterRecursiveCharacterTextSplitter",
  "parameters": {
    "chunkSize": 1000,
    "chunkOverlap": 200,
    "options": {
      "keepSeparator": false
    }
  }
}
```

### Character Text Splitter
**Type:** `@n8n/n8n-nodes-langchain.textSplitterCharacterTextSplitter`

```json
{
  "type": "@n8n/n8n-nodes-langchain.textSplitterCharacterTextSplitter",
  "parameters": {
    "chunkSize": 1000,
    "chunkOverlap": 200,
    "separator": "\n\n"
  }
}
```

### Token Text Splitter
**Type:** `@n8n/n8n-nodes-langchain.textSplitterTokenSplitter`

```json
{
  "type": "@n8n/n8n-nodes-langchain.textSplitterTokenSplitter",
  "parameters": {
    "chunkSize": 500,
    "chunkOverlap": 50,
    "encodingName": "gpt2"
  }
}
```

---

## Retrievers
<!-- chunk: 23-retrievers | keywords: retriever, search, query -->

### Vector Store Retriever
**Type:** `@n8n/n8n-nodes-langchain.retrieverVectorStore`

```json
{
  "type": "@n8n/n8n-nodes-langchain.retrieverVectorStore",
  "parameters": {
    "topK": 5
  }
}
```

### Multi-Query Retriever
**Type:** `@n8n/n8n-nodes-langchain.retrieverMultiQuery`

```json
{
  "type": "@n8n/n8n-nodes-langchain.retrieverMultiQuery",
  "parameters": {
    "queryCount": 3
  }
}
```

### Contextual Compression Retriever
**Type:** `@n8n/n8n-nodes-langchain.retrieverContextualCompression`

### Workflow Retriever
**Type:** `@n8n/n8n-nodes-langchain.retrieverWorkflow`

---

## Chains
<!-- chunk: 23-chains | keywords: chain, llm, qa -->

### Basic LLM Chain
**Type:** `@n8n/n8n-nodes-langchain.chainLlm`

```json
{
  "type": "@n8n/n8n-nodes-langchain.chainLlm",
  "parameters": {
    "prompt": "Summarize the following text:\n\n{{ $json.text }}",
    "options": {}
  }
}
```

### Summarization Chain
**Type:** `@n8n/n8n-nodes-langchain.chainSummarization`

```json
{
  "type": "@n8n/n8n-nodes-langchain.chainSummarization",
  "parameters": {
    "type": "map_reduce",
    "options": {
      "combineMapPrompt": "",
      "prompt": ""
    }
  }
}
```

**Types:** `stuff`, `map_reduce`, `refine`

### Retrieval QA Chain
**Type:** `@n8n/n8n-nodes-langchain.chainRetrievalQa`

```json
{
  "type": "@n8n/n8n-nodes-langchain.chainRetrievalQa",
  "parameters": {
    "query": "={{ $json.question }}",
    "options": {
      "sourceField": "source"
    }
  }
}
```

### Text Classifier
**Type:** `@n8n/n8n-nodes-langchain.textClassifier`

```json
{
  "type": "@n8n/n8n-nodes-langchain.textClassifier",
  "parameters": {
    "categories": "positive,negative,neutral",
    "inputText": "={{ $json.text }}",
    "options": {
      "multiLabel": false,
      "includeInputText": false
    }
  }
}
```

### Sentiment Analysis
**Type:** `@n8n/n8n-nodes-langchain.sentimentAnalysis`

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
        { "name": "email", "description": "Email address", "type": "string" },
        { "name": "age", "description": "Age in years", "type": "number" }
      ]
    }
  }
}
```

---

## Output Parsers
<!-- chunk: 23-parsers | keywords: output, parser, structured -->

### Structured Output Parser
**Type:** `@n8n/n8n-nodes-langchain.outputParserStructured`

```json
{
  "type": "@n8n/n8n-nodes-langchain.outputParserStructured",
  "parameters": {
    "schemaType": "manual",
    "schema": {
      "type": "object",
      "properties": {
        "name": { "type": "string" },
        "score": { "type": "number" },
        "tags": { "type": "array", "items": { "type": "string" } }
      },
      "required": ["name", "score"]
    }
  }
}
```

### Auto-Fixing Output Parser
**Type:** `@n8n/n8n-nodes-langchain.outputParserAutofixing`

### Item List Output Parser
**Type:** `@n8n/n8n-nodes-langchain.outputParserItemList`

```json
{
  "type": "@n8n/n8n-nodes-langchain.outputParserItemList",
  "parameters": {
    "options": {
      "numberOfItems": 5,
      "separator": ","
    }
  }
}
```

---

## Triggers
<!-- chunk: 23-triggers | keywords: trigger, chat, input -->

### Chat Trigger
**Type:** `@n8n/n8n-nodes-langchain.chatTrigger`
**Version:** 1.1

```json
{
  "type": "@n8n/n8n-nodes-langchain.chatTrigger",
  "typeVersion": 1.1,
  "webhookId": "chat-uuid",
  "parameters": {
    "mode": "hostedChat",
    "options": {
      "title": "AI Assistant",
      "subtitle": "How can I help you?",
      "inputPlaceholder": "Type your message...",
      "initialMessages": "Hello! How can I assist you today?",
      "responseMode": "lastNode"
    }
  }
}
```

**Modes:**
- `hostedChat` - n8n-hosted chat widget
- `webhook` - Webhook-based chat

### Manual Chat Trigger
**Type:** `@n8n/n8n-nodes-langchain.manualChatTrigger`

---

## Complete AI Workflow Example
<!-- chunk: 23-example | keywords: example, complete, workflow -->

```json
{
  "name": "AI Chat Agent with RAG",
  "nodes": [
    {
      "id": "1",
      "name": "Chat Trigger",
      "type": "@n8n/n8n-nodes-langchain.chatTrigger",
      "position": [250, 300],
      "parameters": {
        "mode": "hostedChat",
        "options": { "title": "Knowledge Assistant" }
      }
    },
    {
      "id": "2",
      "name": "AI Agent",
      "type": "@n8n/n8n-nodes-langchain.agent",
      "position": [700, 300],
      "parameters": {
        "agent": "conversationalAgent",
        "text": "={{ $json.chatInput }}",
        "options": {
          "systemMessage": "You are a helpful assistant with access to a knowledge base."
        }
      }
    },
    {
      "id": "3",
      "name": "OpenAI Chat",
      "type": "@n8n/n8n-nodes-langchain.lmChatOpenAi",
      "position": [450, 150],
      "parameters": {
        "model": "gpt-4o",
        "options": { "temperature": 0.7 }
      },
      "credentials": { "openAiApi": { "id": "1", "name": "OpenAI" } }
    },
    {
      "id": "4",
      "name": "Buffer Memory",
      "type": "@n8n/n8n-nodes-langchain.memoryBufferWindow",
      "position": [450, 450],
      "parameters": {
        "sessionKey": "={{ $json.sessionId }}",
        "contextWindowLength": 10
      }
    },
    {
      "id": "5",
      "name": "Vector Store Tool",
      "type": "@n8n/n8n-nodes-langchain.toolVectorStore",
      "position": [450, 550],
      "parameters": {
        "name": "knowledge_search",
        "description": "Search the knowledge base"
      }
    },
    {
      "id": "6",
      "name": "Pinecone",
      "type": "@n8n/n8n-nodes-langchain.vectorStorePinecone",
      "position": [200, 550],
      "parameters": {
        "mode": "load",
        "pineconeIndex": "knowledge-base"
      },
      "credentials": { "pineconeApi": { "id": "2", "name": "Pinecone" } }
    },
    {
      "id": "7",
      "name": "OpenAI Embeddings",
      "type": "@n8n/n8n-nodes-langchain.embeddingsOpenAi",
      "position": [200, 700],
      "parameters": { "model": "text-embedding-3-small" },
      "credentials": { "openAiApi": { "id": "1", "name": "OpenAI" } }
    }
  ],
  "connections": {
    "Chat Trigger": {
      "main": [[{ "node": "AI Agent", "type": "main", "index": 0 }]]
    },
    "OpenAI Chat": {
      "ai_languageModel": [[{ "node": "AI Agent", "type": "ai_languageModel", "index": 0 }]]
    },
    "Buffer Memory": {
      "ai_memory": [[{ "node": "AI Agent", "type": "ai_memory", "index": 0 }]]
    },
    "Vector Store Tool": {
      "ai_tool": [[{ "node": "AI Agent", "type": "ai_tool", "index": 0 }]]
    },
    "Pinecone": {
      "ai_vectorStore": [[{ "node": "Vector Store Tool", "type": "ai_vectorStore", "index": 0 }]]
    },
    "OpenAI Embeddings": {
      "ai_embedding": [[{ "node": "Pinecone", "type": "ai_embedding", "index": 0 }]]
    }
  }
}
```

---

*Source: packages/@n8n/nodes-langchain/nodes/*
