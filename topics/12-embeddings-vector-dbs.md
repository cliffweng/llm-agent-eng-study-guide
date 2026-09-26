---
title: "12. Embeddings & vector DBs"
layout: default
nav_order: 13
---

# Embeddings & vector databases
{: .no_toc }

*~7 min read*

**🎯 Interview frequent**

## Why it matters

Embeddings are the mechanism that makes "semantic search" possible — finding text that means something similar, not just text that shares keywords — and they underpin RAG, memory retrieval, deduplication, clustering, and recommendation systems. Vector databases are the infrastructure that makes searching millions of embeddings fast. If you're building anything with RAG or semantic memory, this is the storage layer you're actually operating.

## Core concepts

- **An embedding is a fixed-length vector of numbers** that represents a piece of text (or image, audio, etc.) such that semantically similar inputs produce vectors that are close together in that vector space, by some distance measure (commonly cosine similarity or dot product).
- **Embeddings are produced by a model trained for this purpose** — a separate, usually much smaller/cheaper model than your main LLM, specifically trained so that similarity in vector space correlates with semantic similarity in meaning. Using your main chat model's internal representations directly is not how this normally works; you call a dedicated embedding model/endpoint.
- **Similarity search finds nearest neighbors in vector space.** Given a query embedding, you're looking for the stored embeddings closest to it — this is the mathematical operation underneath "semantic search."
- **Exact nearest-neighbor search doesn't scale.** Comparing a query vector against millions of stored vectors one-by-one is too slow for real-time use. Vector databases use approximate nearest neighbor (ANN) algorithms (e.g., HNSW graphs, IVF) that trade a small amount of accuracy for large speed gains — "approximate" here usually means you get the true nearest neighbors the vast majority of the time, not that results are unreliable.
- **A vector database is a vector index plus the operational features you actually need**: metadata storage and filtering (search only within a user's documents, or docs newer than a date), CRUD operations (update/delete embeddings as source data changes), persistence, and scaling. Purpose-built vector DBs (e.g., Pinecone, Weaviate, Qdrant, Milvus) and vector-search extensions on general databases (e.g., pgvector on Postgres) both exist; the right choice depends on whether you already have infra you'd rather extend versus need dedicated scale.
- **Embedding model choice matters and isn't interchangeable mid-project** — different embedding models produce vectors in different, incompatible spaces. If you switch embedding models, you generally need to re-embed and re-index your entire corpus, not just swap the model.
- **Dimensionality is a tradeoff.** Higher-dimensional embeddings can capture more nuance but cost more storage and compute per comparison; many providers offer configurable/truncatable embedding sizes for this reason.

## Mental model

```
"a golden retriever running" -----> embedding model -----> [0.12, -0.98, 0.34, ...]
"a dog playing in a park"    -----> embedding model -----> [0.15, -0.91, 0.29, ...]   <- close in vector space
"quarterly tax filing rules" -----> embedding model -----> [-0.87, 0.02, 0.55, ...]   <- far away
```

Think of an embedding as a coordinate in a very high-dimensional map of meaning, where "distance" is semantic difference — a vector database is just the spatial index that lets you quickly ask "what's near this point?" at scale, the way a geospatial index lets you ask "what's near this lat/long?"

## Interview questions

1. **Why can't you just compare raw text strings for semantic search instead of using embeddings?**
   Answer: String/keyword comparison only catches exact or near-exact term overlap, not conceptual similarity — "car" and "automobile" or "happy" and "pleased with the outcome" share little to no surface text but mean similar things. Embeddings map meaning into a geometric space where semantically related text ends up close together regardless of exact wording, which keyword matching can't do on its own (this is why hybrid search combines both, see [Advanced RAG](../11-advanced-rag/)).

2. **What does "approximate" mean in approximate nearest neighbor search, and why is that an acceptable tradeoff?**
   Answer: ANN algorithms don't guarantee finding the mathematically exact closest vectors every single time — they trade a small, usually negligible loss in retrieval accuracy for dramatically faster search over large datasets, since exact nearest-neighbor search doesn't scale to millions of vectors at real-time latency. It's acceptable because for search/recommendation-style use cases, retrieving a near-optimal set of relevant results fast beats retrieving the mathematically perfect set slowly.

3. **You switch your embedding model to a newer, better one. What has to happen to your existing vector index, and why?**
   Answer: You need to re-embed your entire corpus and rebuild the index — different embedding models produce vectors in different, incompatible spaces, so a query embedded with the new model can't be meaningfully compared against stored vectors from the old model. There's no in-place "upgrade" of existing vectors.

4. **What does a vector database give you beyond "a place to store vectors and compute cosine similarity"?**
   Answer: Metadata filtering (restricting search to a subset of vectors by structured attributes like user ID, date, or permission level), efficient ANN indexing at scale, CRUD support for updating/deleting entries as source data changes, and operational concerns like persistence, replication, and scaling — a naive in-memory list of vectors with brute-force comparison technically "works" but doesn't handle any of this.

5. **When would pgvector (a vector extension on Postgres) be a better choice than a dedicated vector database like Pinecone or Qdrant?**
   Answer: When you already run Postgres for your application data and want to keep vectors co-located with relational data (simpler ops, transactional consistency, one less system to run), and your scale/latency requirements don't demand a purpose-built ANN engine's extra performance — dedicated vector DBs earn their complexity at larger scale or when vector search is the primary, highest-QPS workload rather than a secondary feature of an existing relational system.

6. **Does it matter which distance metric (cosine similarity, dot product, Euclidean) you use to compare embeddings?**
   Answer: Yes — which metric is meaningful depends on how the embedding model was trained; many modern embedding models normalize vectors and are trained/evaluated using cosine similarity (or an equivalent normalized dot product), so using a different, untested metric can give inconsistent or degraded rankings. Always use the distance metric the embedding provider recommends/tested against, not just whichever your vector database defaults to.

## Watch

- [Transformers, the tech behind LLMs | Deep Learning Chapter 5](https://www.youtube.com/watch?v=wjZofJX0v4M) — 3Blue1Brown. Includes a rigorous, intuitive treatment of embeddings and the geometry of semantic similarity.
- [What is a Vector Database?](https://www.youtube.com/watch?v=t9IDoenf-lo) — IBM Technology. Clear explainer of vector DB mechanics, ANN indexing, and common use cases.

## Further reading

- [Pinecone: What is a Vector Database?](https://www.pinecone.io/learn/vector-database/) — practical overview of ANN algorithms (HNSW, IVF) and vector DB architecture.
- [pgvector](https://github.com/pgvector/pgvector) — open-source vector similarity search extension for Postgres.
