---
title: "11. Advanced RAG"
layout: default
nav_order: 12
---

# Advanced RAG (chunking, hybrid search, eval)
{: .no_toc }

*~9 min read*

**🎯 Interview frequent**

## Why it matters

Basic RAG (chunk, embed, retrieve top-k, generate) works as a demo and breaks down in production: relevant information gets split across chunk boundaries, pure vector similarity misses exact keyword matches, and without evaluation you can't tell whether a bad answer came from retrieval or generation. This topic covers the practical techniques that separate a RAG demo from a RAG system people can trust.

## Core concepts

- **Chunking strategy is a real design decision, not an afterthought.** Fixed-size chunking (e.g., every 500 tokens) is simple but can cut sentences or ideas in half. Semantic/structure-aware chunking (splitting on headings, paragraphs, or logical document sections) tends to produce chunks that are more self-contained and coherent. Overlap between adjacent chunks (e.g., 10-20%) helps avoid losing context that straddles a boundary.
- **Hybrid search combines dense (embedding/vector) retrieval with sparse (keyword, e.g., BM25) retrieval.** Vector search is great at semantic/conceptual matches but can miss exact terms (product codes, names, acronyms) that keyword search catches directly; combining both and merging/reranking the results tends to outperform either alone.
- **Reranking adds a second, more expensive pass** over an initial larger candidate set (e.g., top-50 from vector + keyword search) using a more precise (often cross-encoder) model to reorder by actual relevance to the query, then keeps only the top few for the final prompt. This trades some latency for meaningfully better precision.
- **Query transformation** improves retrieval before it even happens: rewriting a vague user query into a more specific search query, decomposing a complex multi-part question into sub-queries retrieved separately, or generating a hypothetical answer first and embedding *that* to search (HyDE) — all aimed at closing the gap between how users ask questions and how relevant content is actually phrased.
- **RAG evaluation needs to separate retrieval quality from generation quality.** Common metrics: *context precision/recall* (did retrieval get the right chunks, and only the right chunks), *faithfulness/groundedness* (is the generated answer actually supported by the retrieved context, or did the model add unsupported claims), and *answer relevance* (does the final answer actually address the question). Conflating these makes it impossible to tell what to fix.
- **The "RAG triad"** (a common framing): evaluate context relevance, groundedness, and answer relevance as three separate checks, often using an LLM-as-judge for each (see [Evaluation & observability](13-evaluation-observability.html)) rather than a single end-to-end "was this a good answer" score.
- **Metadata filtering** (filtering candidates by structured attributes — date, source, document type, permission level — before or alongside vector search) is often as important as the similarity search itself, especially for access control and recency.

## Mental model

```
query --> [ query rewrite / decomposition ] --> multiple search queries
                                                        |
                        +-------------------------------+-------------------------------+
                        |                                                               |
                 dense (vector) search                                        sparse (keyword/BM25) search
                        |                                                               |
                        +-------------------- merge candidates ------------------------+
                                                        |
                                              [ rerank (cross-encoder) ]
                                                        |
                                              top-k chunks -> prompt -> LLM
                                                        |
                          eval: context precision/recall | faithfulness | answer relevance
```

Think of advanced RAG as tuning a search engine, because that's what it is — chunking is indexing strategy, hybrid search is combining ranking signals, reranking is a second-pass relevance model, and eval is exactly the kind of precision/recall measurement search teams have used for decades, just applied to feeding an LLM instead of a results page.

## Interview questions

1. **Why would you use hybrid search instead of pure vector similarity search?**
   Answer: Vector search captures semantic similarity but can miss retrieval where an exact term (a product SKU, an error code, a proper noun) matters more than conceptual meaning — keyword/BM25 search catches those exact matches directly. Combining both and merging results (often with reranking) covers both failure modes better than either alone.

2. **What's the purpose of a reranking step if you've already retrieved with vector + keyword search?**
   Answer: Initial retrieval (especially at reasonable latency/cost) typically optimizes for recall over a larger candidate set using cheaper similarity methods; a reranker is a more expensive, more precise model (often a cross-encoder that looks at the query and document together, rather than comparing precomputed independent embeddings) applied to that smaller candidate set to get a more accurate final ordering before deciding what actually goes in the prompt.

3. **How do you tell whether a bad RAG answer is a retrieval problem or a generation problem?**
   Answer: Check context precision/recall (did retrieval return the right supporting chunks) separately from faithfulness (did the generated answer actually stick to what was in the retrieved context) and answer relevance (does the answer address the question asked). If the right chunks were retrieved but the answer contains unsupported claims, that's a faithfulness/generation problem; if the chunks themselves were wrong or missing, that's a retrieval problem — conflating the two metrics into one "was this answer good" score hides which one to fix.

4. **Why might fixed-size chunking (e.g., every 500 tokens) hurt retrieval quality compared to structure-aware chunking?**
   Answer: Fixed-size splitting can cut a sentence, table, or logical unit in half at an arbitrary boundary, producing chunks that are semantically incomplete or misleading in isolation — a chunk that's half of one idea and half of an unrelated next idea embeds poorly and retrieves poorly. Structure-aware chunking (by heading/paragraph/section) tends to produce more self-contained, coherently embeddable chunks, sometimes at the cost of more variable chunk sizes.

5. **What's query decomposition and when is it worth the extra latency/cost of multiple retrieval passes?**
   Answer: Query decomposition breaks a complex, multi-part question into separate sub-queries that are each retrieved independently, then combined — worth it when a single query would otherwise need to match multiple, disparate pieces of information that no single similarity search is likely to retrieve together (e.g., "compare X and Y" needs separate retrieval for X's info and Y's info). Not worth it for simple, single-fact questions where it just adds latency for no retrieval benefit.

6. **Why is metadata filtering often as important as the similarity search itself in an enterprise RAG deployment?**
   Answer: Similarity search alone has no concept of who's allowed to see what or what's still current — without filtering by attributes like permission level, tenant/user ID, or date, a semantically relevant chunk from a document the requesting user shouldn't access (or an outdated version of a policy) can get retrieved and surfaced. Metadata filtering enforces access control and recency constraints that similarity ranking has no way to express on its own.

## Watch

- [Building, Evaluating, and Optimizing your RAG App for Production](https://www.youtube.com/watch?v=KLfYPwVP0qs) — Jerry Liu (LlamaIndex CEO), via H2O.ai. Covers advanced retrieval techniques and the RAG evaluation triad for production systems.
- [The Complete Guide to Hybrid Search in RAG (BM25 + Embeddings + Reranker)](https://www.youtube.com/watch?v=XvKiTfd6Xvo) — Dave Ebbelaar. Hands-on walkthrough of combining sparse and dense retrieval with a reranking stage.

## Further reading

- [Pinecone: Chunking strategies for LLM applications](https://www.pinecone.io/learn/chunking-strategies/) — practical comparison of chunking approaches.
- [RAGAS documentation](https://docs.ragas.io/) — open-source framework implementing the context precision/recall, faithfulness, and answer relevance metrics.
