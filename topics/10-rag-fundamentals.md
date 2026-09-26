---
title: "10. RAG fundamentals"
layout: default
nav_order: 11
---

# RAG fundamentals
{: .no_toc }

*~8 min read*

**🎯 Interview frequent**

## Why it matters

A model's training data is frozen at some point in the past and doesn't include your company's private documents. Retrieval-Augmented Generation (RAG) is the standard pattern for giving an LLM access to specific, current, or proprietary information at answer time — by fetching relevant text and putting it directly in the prompt — instead of hoping the model already "knows" it or trying to retrain the model on your data.

## Core concepts

- **The basic RAG pipeline**: chunk your documents into pieces → embed each chunk into a vector (see [Embeddings & vector DBs](../12-embeddings-vector-dbs/)) → store vectors in a searchable index → at query time, embed the user's question, retrieve the most similar chunks → stuff those chunks into the prompt as context → the model answers grounded in that retrieved text.
- **RAG exists to reduce hallucination and add current/private knowledge**, not to replace prompting or fine-tuning. It's the right tool when the problem is "the model doesn't have access to this information," not when the problem is "the model doesn't understand the task" (that's a prompting problem) or "the model doesn't behave in the right style" (that's closer to fine-tuning).
- **Retrieval quality bounds answer quality.** If the retrieval step returns irrelevant or incomplete chunks, no amount of prompt engineering downstream fixes it — the model can only ground its answer in what it was actually given. This is why RAG debugging usually starts by inspecting *what was retrieved*, not the final answer.
- **Chunking strategy matters a lot** (covered in depth in [Advanced RAG](../11-advanced-rag/)) — chunks too large dilute relevance and waste context; chunks too small lose surrounding context needed to make sense of the content.
- **Similarity search is approximate, not exact.** Vector similarity finds chunks that are semantically close to the query embedding, which isn't the same as "the objectively correct answer" — a chunk can be a close semantic match while still being the wrong piece of information for the question asked.
- **RAG isn't just for unstructured documents.** The same "retrieve relevant context, then generate" pattern applies to structured data (SQL results), code (retrieve relevant functions/files), and conversation history (retrieving relevant past turns — see [Memory](../06-memory/)).
- **Citations/attribution** (showing which retrieved chunk supported which part of the answer) are both a UX feature (lets users verify) and a debugging tool (lets you trace a wrong answer back to a specific bad retrieval or a model error on top of good retrieval).

## Mental model

```
 [ documents ] -> chunk -> embed -> [ vector index ]
                                          ^
                                          | similarity search
 [ user question ] -> embed -------------+
                                          |
                                          v
                              [ top-k relevant chunks ]
                                          |
                                          v
        prompt = instructions + retrieved chunks + question -> LLM -> grounded answer
```

Think of RAG as an open-book exam: instead of expecting the model to have memorized everything, you hand it the specific pages of the book relevant to the question right before it answers.

## Interview questions

1. **When would you reach for RAG instead of fine-tuning to give a model access to specific knowledge?**
   Answer: RAG fits when the underlying information changes frequently, is too large to bake into weights, needs to be attributable/traceable (you can point to the source chunk), or needs per-user/per-tenant access control at query time. Fine-tuning fits better for teaching a model a consistent style, format, or specialized skill rather than injecting facts — and doesn't naturally support easy updates or per-request access control the way retrieval does.

2. **A RAG system gives a wrong answer. What's the first thing to check, and why?**
   Answer: What was actually retrieved for that query — the model can only ground its answer in the chunks it was given, so if retrieval returned irrelevant or incomplete chunks, the generation step was never going to produce a correct answer regardless of prompt quality. Only after confirming retrieval was good do you look at whether the model mishandled correct context.

3. **Why isn't "the most semantically similar chunk" always "the most relevant chunk" for answering a question?**
   Answer: Vector similarity captures topical/semantic closeness, not logical relevance to answering a specific question — e.g., a chunk that discusses the same entity but the wrong attribute, or a near-duplicate chunk that happens to omit the one relevant detail, can score highly similar while being unhelpful or misleading for the actual question.

4. **What's the risk of retrieving too many chunks "just to be safe"?**
   Answer: More chunks means more tokens (cost, latency) and more irrelevant content competing for the model's attention alongside what's actually needed, which can dilute answer quality rather than improve it — retrieval should aim for the smallest set of chunks that reliably contains the needed information, not the largest set that might.

5. **How would you debug a RAG system where retrieval looks correct (right chunks) but the final answer is still wrong?**
   Answer: At that point it's a generation/prompting problem, not a retrieval problem — check whether the prompt clearly instructs the model to use only the provided context, whether the chunks are presented in a way the model can parse (clear boundaries, not mashed together), and whether the model is being asked something the retrieved context doesn't actually fully answer (a scope mismatch between the question and what was retrieved).

6. **Your source documents get updated daily, but the RAG system keeps citing outdated information. What's the most likely cause?**
   Answer: The vector index wasn't re-embedded/re-indexed after the source documents changed — retrieval only ever returns what's in the index, so if the ingestion pipeline that chunks and embeds new/changed documents isn't running (or isn't running often enough), the index silently drifts out of sync with the source of truth. Fixing it means treating the index as a derived artifact with its own freshness/update pipeline, not a one-time build step.

## Watch

- [What is Retrieval-Augmented Generation (RAG)?](https://www.youtube.com/watch?v=T-D1OfcDW1M) — IBM Technology. Clear, widely-referenced explainer of the core RAG concept and pipeline.
- [Learn RAG From Scratch – Python AI Tutorial from a LangChain Engineer](https://www.youtube.com/watch?v=sVcwVQRHIc8) — freeCodeCamp.org (taught by LangChain engineer Lance Martin). Full practical walkthrough of building a RAG pipeline.

## Further reading

- [Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks](https://arxiv.org/abs/2005.11401) — the original RAG paper.
- [LangChain: RAG concepts](https://python.langchain.com/docs/concepts/rag/) — practical taxonomy of RAG components and variations.
