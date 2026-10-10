# Structured Retrieval vs Hybrid RAG

These workflows intentionally use different retrieval patterns because the underlying data is different.

| | Structured SQL retrieval | Hybrid RAG |
|---|---|---|
| Data type | Relational / structured | Unstructured documents |
| Primary retrieval | SQL query | Vector similarity + full-text search |
| Embeddings | No | Yes |
| Vector database | No | PostgreSQL / PGVector |
| Chunking | No | Recursive chunks, 700 size / 100 overlap |
| Reranking | No | Cohere rerank |
| Main strength | Deterministic business lookup | Semantic document retrieval |
| Typical use | prices, catalog records, attributes | manuals, guides, knowledge documents |

## Structured SQL retrieval

The LLM does not receive the whole database. It receives a database tool and chooses a search term. PostgreSQL performs the actual lookup using a bounded query.

This pattern is appropriate when the answer should be sourced from explicit structured records and semantic similarity adds little value.

## Hybrid RAG

The document workflow creates embeddings for document chunks and stores them in PGVector with metadata.

At query time it combines two retrieval signals:

1. PostgreSQL full-text search for lexical matches.
2. PGVector cosine similarity for semantic matches.

The candidate sets are merged and deduplicated, then reranked before only the strongest context is passed to the answer model.

The answer stage is explicitly instructed to use the retrieved context and to report insufficient evidence instead of inventing an answer.

## Why hybrid retrieval

Vector search is useful when the user's wording differs from the document wording. Full-text search remains useful for exact terminology and strong lexical matches. Combining both creates a broader candidate set before reranking.

The workflow therefore demonstrates a retrieval pipeline rather than treating a vector store as the entire RAG system.