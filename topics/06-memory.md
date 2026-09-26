---
title: "06. Memory (short / long / episodic)"
layout: default
nav_order: 7
---

# Memory (short / long / episodic)
{: .no_toc }

*~8 min read*

**🎯 Interview frequent**

## Why it matters

LLMs are stateless (see [LLM basics](01-llm-basics.html)), so any "memory" an agent appears to have is something you built. Choosing the wrong kind of memory for the wrong kind of information is a common source of both bad UX (the assistant "forgets" things it should know) and bloated cost (re-sending everything, always, forever). This topic is about picking the right storage and retrieval strategy for different lifespans of information.

## Core concepts

- **Short-term (working) memory** is just the current context window — the conversation history and any scratch state passed along within a single session. It's fast and simple but bounded by the context window and disappears when the session ends unless explicitly persisted.
- **Long-term memory** is information persisted across sessions — user preferences, facts learned once and expected to be recalled later, prior decisions. This typically lives outside the model, in a database, key-value store, or vector store, and is explicitly retrieved and injected into context when relevant (this is functionally a form of RAG — see [RAG fundamentals](10-rag-fundamentals.html)).
- **Episodic memory** stores specific past experiences/events ("what happened last time," "what did we discuss in session 3") as opposed to distilled facts. Useful for agents that need to recall specific prior interactions or outcomes, not just general knowledge about the user.
- **Semantic memory** (a common companion term) stores generalized facts/knowledge distilled from experience, decoupled from *when* they were learned — e.g., "the user prefers metric units," extracted once from an episodic interaction and stored as a standalone fact.
- **Memory requires a write policy and a read policy, not just storage.** *Write*: what gets saved, when, and how it's summarized/extracted (naive approach: dump every message; better: extract salient facts/decisions). *Read*: what gets retrieved and injected into context for a given turn (naive: dump everything; better: retrieve only what's relevant, ranked by recency/similarity/importance).
- **Memory can go stale or contradict itself.** A long-term memory store that never updates or reconciles conflicting facts (e.g., "user works at Company A" vs. a later "user now works at Company B") will confidently feed the model wrong context. Real systems need update/invalidation logic, not just append-only storage.
- **More memory isn't automatically better.** Injecting irrelevant "remembered" facts into every prompt wastes tokens and can distract the model from the current task — memory retrieval should be relevance-gated, the same way RAG retrieval should be.

## Mental model

```
                 session ends
[ short-term: current context ] ------------> [ extraction: what's worth keeping? ]
        ^                                                   |
        | injected each turn                                v
        |                                    [ long-term store: facts / episodes ]
        +---------------- relevant recall ---------------------+
                     (retrieved based on current turn)
```

Think of short-term memory as RAM (fast, session-scoped, disappears on restart) and long-term memory as a database you deliberately write to and query — the model itself is neither; it's just the CPU that reads whatever you load into its registers (the context window) this turn.

## Interview questions

1. **Why can't you just keep appending every message to the context forever and call that "memory"?**
   Answer: The context window is finite, and cost/latency grow with context size, so an ever-growing transcript eventually breaks (truncation or unaffordable cost). You need a policy for what gets persisted outside the context (long-term store) and what gets retrieved back in only when relevant, rather than treating the raw transcript as infinite memory.

2. **What's the difference between episodic and semantic memory, and why might an agent want both?**
   Answer: Episodic memory stores specific past events/interactions ("in our last session, you asked about X and I answered Y"); semantic memory stores generalized, time-independent facts distilled from those events ("user prefers concise answers"). An agent might want both because some tasks need "what exactly happened last time" while others just need the distilled preference/fact, without needing to re-derive it from raw history every time.

3. **How would you handle a long-term memory store where two stored facts about the user contradict each other?**
   Answer: You need explicit reconciliation logic — e.g., timestamp facts and prefer the most recent, detect contradictions at write time and either overwrite or flag for review, or store facts with provenance/confidence so retrieval can pick the most trustworthy one. Naive append-only storage without reconciliation will eventually feed contradictory context to the model.

4. **Why is memory retrieval basically a RAG problem?**
   Answer: Both involve selecting a relevant subset of stored information to inject into a bounded context window based on the current query/turn, typically via similarity search or ranking over an external store — the same techniques (embeddings, hybrid search, reranking) that apply to document RAG apply to memory recall (see [RAG fundamentals](10-rag-fundamentals.html)).

5. **What's a concrete failure mode of injecting too much "remembered" context into every prompt?**
   Answer: It wastes tokens/cost on irrelevant information, can push relevant current-turn context further from where the model attends most reliably, and can even confuse the model by surfacing stale or tangential facts as if they matter to the current request — relevance-gated retrieval (only pull in memory likely to matter for this turn) avoids this.

6. **How is "memory" different from "context management" (see [Context management & compaction](07-context-management-compaction.html)), given both decide what information the model sees?**
   Answer: Memory is about persisting and later recalling information *across* sessions — user preferences, past decisions, facts that should survive after the current conversation ends. Context management is about keeping a *single* long-running session coherent as it approaches the context window limit, via compaction, summarization, or offloading. They share techniques (summarize, store externally, retrieve selectively), but memory's failure mode is "a future session doesn't know something it should," while context management's failure mode is "this session breaks or degrades before the task is done."

## Watch

- [The Four Types of Memory Every AI Agent Needs](https://www.youtube.com/watch?v=BacJ6sEhqMo) — IBM Technology. Clear breakdown of short-term/working, long-term, episodic, and semantic memory for agents.

## Further reading

- [LangChain: Memory concepts](https://python.langchain.com/docs/concepts/memory/) — a practical taxonomy of memory types used in agent frameworks.
