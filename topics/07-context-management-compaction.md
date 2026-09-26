---
title: "07. Context management & compaction"
layout: default
nav_order: 8
---

# Context management & compaction
{: .no_toc }

*~8 min read*

**🎯 Interview frequent**

## Why it matters

Long-running agent sessions — the kind that call tools dozens of times, read large files, or run for hours — will eventually exceed any context window, no matter how large. Context management is the engineering discipline of deciding what stays, what gets summarized, and what gets dropped, so the agent keeps working coherently instead of hitting a wall or degrading silently as context fills up.

## Core concepts

- **The problem isn't just "running out of space."** Even well before hitting the hard token limit, very long contexts can degrade output quality (irrelevant/stale content competes for attention with what's actually needed right now) and always cost more per call. Managing context is as much about quality as about avoiding truncation.
- **Compaction/summarization** replaces older, low-value parts of the context (e.g., early tool call outputs, resolved sub-tasks, verbose intermediate reasoning) with a condensed summary that preserves the information needed going forward. This is typically done by an explicit LLM call whose only job is "summarize what's happened so far, keep what matters for continuing the task."
- **Selective retention** keeps some things verbatim (the original task/goal, key decisions, unresolved errors) while aggressively compacting or dropping others (raw tool output that's already been acted on, exploratory dead ends). Not all context is equally worth preserving.
- **Sliding windows** keep only the last N turns/tokens verbatim, dropping or summarizing anything older — simple, but risks losing information from early in a session that turns out to matter later.
- **External offloading**: move information out of the context window entirely into a file, scratchpad, or memory store, and have the agent explicitly re-fetch it only when needed (this overlaps heavily with [Memory](06-memory.html) — the practical difference is memory is about *across sessions*, compaction is often about managing a *single long session*).
- **Compaction has real risk.** A summarization step can drop a detail that turns out to matter three steps later, and there's no free way to know in advance which details those are. Production systems mitigate this by keeping critical anchors (goal statement, hard constraints, unresolved errors) always verbatim and only compacting the "middle" of a session.
- **Triggering compaction** is usually threshold-based (e.g., compact when context usage crosses some percentage of the window) rather than after every turn, since summarization itself costs a call and adds latency.

## Mental model

```
[ system / goal ]  <-- always kept verbatim
[ oldest turns ]  ---->  [ compacted summary ]  ----+
[ middle turns  ]  ---->  [ compacted summary ]     |--> fits in context window
[ recent turns  ]  <-- kept verbatim (high relevance)
[ current turn  ]  <-- always kept verbatim
```

Think of compaction like a video game autosave that periodically compresses your play history into "here's the state that matters now" — you don't replay every keystroke, you resume from a summary that's faithful enough to continue correctly.

## Interview questions

1. **Why would you compact context well before actually hitting the model's context window limit?**
   Answer: Quality can degrade before the hard limit is reached — long, noisy contexts make it harder for the model to attend to what's currently relevant, and every extra token costs money/latency regardless of whether the limit is reached. Proactive compaction (e.g., at 70-80% of window usage) avoids both the quality degradation and the risk of an abrupt failure right at the limit.

2. **What's the risk of summarizing away "old" tool outputs in a long agent session, and how do production systems mitigate it?**
   Answer: A detail that seemed irrelevant when summarized might turn out to matter later (e.g., an error code from step 3 becomes relevant when debugging step 12). Mitigation: keep certain categories of information (the original goal/constraints, unresolved errors, key decisions) always verbatim/pinned, and only aggressively compact clearly-resolved or purely exploratory content.

3. **What's the difference between context compaction and long-term memory, given that both involve summarizing/storing information outside the immediate prompt?**
   Answer: Compaction is typically scoped to keeping a single long-running session coherent as it exceeds context limits; long-term memory is about persisting and later recalling information *across* sessions. In practice they use similar techniques (summarization, selective retention, external storage + retrieval), but compaction's failure mode is "this session breaks," while memory's failure mode is "future sessions don't know something they should."

4. **Why is a simple sliding window (keep only the last N turns) often insufficient for agent sessions, as opposed to chat UIs?**
   Answer: Agent sessions often depend on information established much earlier (the original task specification, a constraint discovered mid-task, an ID or file path referenced once) that a fixed window would silently drop, even though it's still needed. Sliding windows work better for casual chat where old turns are usually genuinely less relevant.

5. **How would you decide when to trigger a compaction step in an agent loop — after every turn, or something else?**
   Answer: Usually threshold-based (e.g., trigger when context usage crosses a percentage of the window) rather than every turn, because compaction itself costs an LLM call and adds latency — running it too eagerly wastes resources, while running it too late risks hitting the limit mid-task.

6. **Besides summarization, what's another way to reclaim context in a long agent session, and what's its tradeoff?**
   Answer: External offloading — move information (e.g., a large file's full contents, verbose tool output already acted on) out of the context window entirely into a scratchpad, file, or memory store, and have the agent explicitly re-fetch it only if needed later. The tradeoff versus summarization: you don't lose any detail (nothing is lossily compressed), but the agent needs an explicit, reliable mechanism to know *when* to re-fetch — if it doesn't realize it needs the offloaded detail, it's effectively gone anyway.

## Watch

- [Context Engineering for AI Agents with LangChain and Manus](https://www.youtube.com/watch?v=6_BcCthVvb8) — LangChain. Direct treatment of context window management and compaction strategy for long agent sessions.

## Further reading

- [Anthropic docs: Context editing](https://docs.anthropic.com/en/docs/build-with-claude/context-editing) — automatic context management and compaction for the Claude API.
- [Anthropic: Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — engineering guidance on what to keep, compact, or offload.
