# Retrieval & RAG Patterns with n8n

A compact portfolio project showing two retrieval patterns for LLM applications built with n8n and PostgreSQL:

1. **Structured SQL retrieval** for deterministic business data.
2. **Hybrid RAG** for unstructured documents using PostgreSQL/PGVector, full-text search, embeddings and reranking.

The point of the project is not to treat every lookup as RAG. It demonstrates choosing the retrieval mechanism that matches the data.

## Architecture

```text
                         USER QUERY
                             │
                ┌────────────┴────────────┐
                │                         │
        Structured data            Unstructured docs
                │                         │
        PostgreSQL tool             Hybrid retrieval
                │                  ┌──────┴──────┐
          SQL filtering            │             │
                │              PostgreSQL      PGVector
                │                 FTS          cosine search
                │                  │             │
                │                  └──────┬──────┘
                │                         │
                │                    deduplicate
                │                         │
                │                   Cohere rerank
                │                         │
                └─────────────────────────┴─────► grounded answer
```

## Workflow 1 — Structured SQL Retrieval

`workflows/01_structured_sql_retrieval.json`

A Telegram-connected AI agent uses a PostgreSQL tool when the user asks about structured service data such as price or description.

```text
Telegram
   ↓
AI Agent
   ↓
PostgreSQL tool
   ↓
parameterized service lookup
   ↓
grounded response
```

The agent is instructed not to invent prices. The database tool performs a bounded parameterized lookup against a normal relational table.

**Why this pattern exists:** exact or semi-structured business data usually does not need embeddings or vector similarity. SQL is simpler, explainable and deterministic.

## Workflow 2 — Hybrid PGVector RAG

`workflows/02_hybrid_pgvector_rag.json`

This workflow demonstrates both document ingestion and retrieval.

### Document ingestion

```text
Telegram PDF
   ↓
duplicate / hash checks
   ↓
PDF text extraction
   ↓
recursive chunking
chunk size: 700
chunk overlap: 100
   ↓
OpenAI embeddings
1536 dimensions
   ↓
PostgreSQL / PGVector
```

The ingestion flow stores metadata such as source file, title, page, document key, Telegram file identifier and SHA-256 file hash. Existing document chunks can be detected or replaced before re-indexing.

### Retrieval

```text
User question
     │
     ├────────────► PostgreSQL full-text search
     │
     └────────────► PGVector semantic search
                         cosine distance
                         top K = 10
     │
     ▼
merge candidates
     ↓
deduplicate
     ↓
Cohere rerank
     ↓
top 3 documents
     ↓
build RAG context
     ↓
LLM grounded answer
     ↓
source + page references
```

The workflow also contains an alternative agent-tool retrieval path using PGVector as `retrieve-as-tool` with reranking.

## What this project demonstrates

- Retrieval-Augmented Generation (RAG)
- PostgreSQL / PGVector vector storage
- OpenAI embeddings
- recursive document chunking
- document metadata design
- semantic similarity search
- PostgreSQL full-text search
- hybrid retrieval
- candidate deduplication
- Cohere reranking
- grounded generation
- source attribution
- document deduplication with file identifiers and SHA-256 hashes
- structured SQL tool calling
- n8n AI-agent orchestration

## Why SQL retrieval and RAG are separated

The first workflow is **not labeled RAG**. It is structured retrieval through an LLM tool.

The second workflow is RAG because it contains the core retrieval pipeline for unstructured knowledge: document ingestion, chunking, embeddings, vector storage, retrieval and grounded generation.

This distinction is intentional. See [Retrieval Comparison](docs/retrieval-comparison.md).

## Repository structure

```text
workflows/
├── 01_structured_sql_retrieval.json
└── 02_hybrid_pgvector_rag.json

docs/
└── retrieval-comparison.md
```

## Public export notes

The workflows are sanitized portfolio exports and are intentionally inactive.

Removed before publication:

- n8n credential bindings and credential names;
- generated webhook IDs;
- workflow/version/instance metadata;
- production database credential references;
- production Telegram credential references;
- OpenAI and Cohere credential references;
- internal production schema names.

The public workflows use the demonstration schema names:

```text
retrieval_demo.price_list
retrieval_demo.knowledge_base
```

Credentials must be reconnected manually after importing the workflows into n8n.

## Stack

- n8n
- PostgreSQL
- PGVector
- OpenAI chat model + embeddings
- Cohere Rerank
- Telegram
- JavaScript / SQL inside n8n workflows

## Scope

This repository is a portfolio demonstration of retrieval architecture and n8n implementation patterns. It is not packaged as a one-click production deployment and does not include the original database DDL or production credentials.