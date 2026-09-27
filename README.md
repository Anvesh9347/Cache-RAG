# Cache-Based RAG with Semantic Caching

## Overview

This project implements a **Cache-Based Retrieval-Augmented Generation (RAG)** system that reduces unnecessary LLM calls by caching previously generated answers.

Instead of sending every user query directly to the LLM, the system first checks a **semantic cache** using vector similarity. If a similar question has already been answered, the system retrieves the cached response instead of making another LLM call.

This helps reduce **LLM API costs and response latency** while maintaining a smooth question-answering experience.

## How It Works

The system follows this workflow:

**User Query → Normalize Query → Semantic Cache Lookup → Cache Hit / Cache Miss**

* **Cache Hit:** A semantically similar question is found → return the cached answer directly.
* **Cache Miss:** No relevant cached answer is found → retrieve relevant documents → generate an answer using the LLM → store the generated answer in the cache for future queries.

The project uses **FAISS vector search** and **OpenAI embeddings** to identify semantically similar questions rather than relying only on exact keyword matching.

For example:

> First query: `What is LangGraph?`

The system generates the answer using the RAG pipeline and stores the response in the cache.

If a user later asks:

> `Explain about LangGraph?`

the semantic cache can recognize that the questions are related and reuse the previously generated response instead of calling the LLM again.

## Key Technologies

* Python
* LangGraph
* LangChain
* OpenAI Embeddings
* GPT-4o-mini
* FAISS Vector Store
* Retrieval-Augmented Generation (RAG)
* Semantic Caching

## Cost & Performance Benefits

### Reduced LLM Cost

LLM API calls are typically the most expensive part of a RAG pipeline. By returning cached responses for repeated or semantically similar queries, the system can avoid unnecessary LLM calls.

For example, if the same type of question is asked 100 times and 90 requests can be served from the cache, the system only needs to generate responses for the cache misses rather than making 100 LLM calls.

### Faster Response Time

A cache hit avoids the complete RAG pipeline:

**Cache Hit:**
`Query → Embedding → Vector Search → Cached Answer`

Instead of:

**Cache Miss:**
`Query → Embedding → Vector Search → Retrieve Documents → LLM → Generate Answer`

This can significantly reduce response time for frequently repeated questions.

### Scalable Architecture

As the cache grows, commonly asked questions can be served directly from the vector cache, reducing unnecessary workload on the LLM layer.

## Architecture

```text
                User Query
                    │
                    ▼
            Normalize Question
                    │
                    ▼
          Semantic Cache Lookup
                    │
           ┌────────┴────────┐
           │                 │
        Cache Hit          Cache Miss
           │                 │
           ▼                 ▼
    Return Cached       Retrieve Documents
       Answer                 │
                             ▼
                        LLM Generation
                             │
                             ▼
                       Store Answer
                       in Cache
                             │
                             ▼
                        Return Answer
```

## Key Implementation

The project maintains a separate FAISS-based vector store for cached Q&A pairs. Generated answers are stored as documents and embedded into the cache.

When a new query arrives, the system performs a semantic similarity search against the cache before executing the main RAG pipeline.

The workflow is orchestrated using **LangGraph**, allowing the application to conditionally route requests based on whether a cache hit occurs.

## Future Improvements

* Add configurable similarity thresholds for cache validation
* Add cache expiration / TTL
* Track cache hit and miss rates
* Add cache persistence using Redis, PostgreSQL, or another production database
* Add response freshness validation
* Add monitoring for cost and latency savings
* Support multi-user and production-scale caching

## Project Goal

The main goal of this project is to demonstrate how **semantic caching can be combined with RAG to reduce redundant LLM calls, lower inference costs, and improve response latency for repeated or semantically similar user queries.**
