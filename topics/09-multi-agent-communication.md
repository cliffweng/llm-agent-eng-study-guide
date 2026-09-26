---
title: "09. Multi-agent / inter-agent communication"
layout: default
nav_order: 10
---

# Multi-agent / inter-agent communication
{: .no_toc }

*~8 min read*

**🎯 Interview frequent**

## Why it matters

A single agent with one context window and one set of tools hits real limits: context bloat, conflicting responsibilities, and no natural way to parallelize independent work. Multi-agent systems split a problem across multiple LLM-driven "agents," each with a narrower job — but that introduces a new hard problem: how do these agents communicate, hand off work, and stay coordinated without duplicating effort or contradicting each other.

## Core concepts

- **Why split into multiple agents at all**: context isolation (each sub-agent only needs the context relevant to its sub-task, not the whole system's history), specialization (a "researcher" agent and a "writer" agent can have very different prompts/tools tuned to their job), and parallelism (independent sub-tasks can run concurrently instead of serially in one agent's loop).
- **Orchestrator-worker (a.k.a. supervisor) pattern**: a coordinating agent (or plain code) decomposes a task, dispatches sub-tasks to worker agents, and assembles their results. The orchestrator owns the overall plan; workers don't need to know about each other.
- **Swarm/handoff pattern**: agents hand off control directly to one another based on the conversation's needs (e.g., a "billing agent" hands off to a "technical support agent" mid-conversation), without a central orchestrator making every decision.
- **Communication needs a shared protocol/format**, not just "pass a string." Structured messages (who's asking, what's the sub-task, what format is the expected result) reduce the same ambiguity problems that plain prompting has (see [Prompting & structured outputs](../03-prompting-structured-outputs/)) — this gets harder, not easier, once multiple agents are involved.
- **Shared vs. isolated state.** Agents can share a common memory/blackboard (simpler coordination, but risk of interference/race conditions) or keep fully isolated context and only exchange explicit messages (cleaner boundaries, but risk of duplicated work or missing context one agent has that another needs).
- **Coordination failure modes are the main risk**, not any individual agent's reasoning quality: duplicated work (two agents solve the same sub-problem), contradictory outputs (two agents produce incompatible answers with no reconciliation step), and cost/latency multiplication (N agents means roughly N times the LLM calls for a task that might have fit in one well-structured prompt).
- **More agents isn't automatically better.** Multi-agent architectures add real overhead (more calls, more moving parts, harder to debug end-to-end) — they earn their cost when a task genuinely benefits from specialization or parallelism, not by default for anything "complex-sounding."

## Mental model

```
                     +------------------+
                     |   orchestrator   |
                     +------------------+
                      /       |        \
             sub-task A  sub-task B  sub-task C
                /             |             \
        [ worker agent ] [ worker agent ] [ worker agent ]
                \             |             /
                     results merged back
                     +------------------+
                     |   orchestrator   | --> final answer
                     +------------------+
```

Think of multi-agent systems like a small engineering team: it works when the orchestrator writes clear tickets (sub-tasks) and each worker's output is checked/integrated by someone — it fails the same way a badly-run team fails, with duplicated work and nobody reconciling conflicting outputs.

## Interview questions

1. **What's the actual engineering justification for splitting a task across multiple agents instead of using one agent with more tools?**
   Answer: Context isolation (each sub-agent's context stays focused on its sub-task rather than accumulating the whole system's history), specialization (different prompts/tools tuned per role), and parallelism (independent sub-tasks execute concurrently) — not "it sounds more sophisticated." If a single well-scoped agent or workflow can do the job, multi-agent adds cost and debugging surface for no benefit.

2. **Compare the orchestrator-worker pattern to a swarm/handoff pattern. When would you pick one over the other?**
   Answer: Orchestrator-worker centralizes planning and result assembly in one place, which is easier to reason about and debug but creates a bottleneck/single point of coordination. Swarm/handoff lets agents transfer control directly based on context, which fits naturally conversational scenarios (e.g., support routing) but makes overall system behavior harder to trace/predict since no single component owns the full plan.

3. **What's a concrete failure mode of giving multiple agents shared, unstructured memory?**
   Answer: Race conditions/interference — one agent's in-progress or incorrect intermediate write can be read and acted on by another agent before it's finalized or corrected, and there's no inherent mechanism to detect that two agents are working on conflicting assumptions. Isolated context with explicit, structured message-passing avoids this at the cost of needing to design what gets communicated.

4. **Why does inter-agent communication need more structure ("passing a string") than a single agent's internal reasoning?**
   Answer: A receiving agent has no shared context/history with the sending agent the way a single agent has continuity with its own prior turns — an unstructured string handoff can be ambiguous about what's being asked, what format is expected back, or what assumptions were already made, so structured messages (explicit sub-task, expected output format, relevant context) reduce that ambiguity the same way a schema does for tool calls.

5. **A multi-agent system is producing contradictory outputs from two of its sub-agents on the same underlying question. What's the actual root cause to look for, and how would you fix it architecturally (not just "add another prompt telling them to agree")?**
   Answer: Root cause is usually a coordination gap — either both agents were given the same ambiguous sub-task with no reconciliation step, or they had access to different/inconsistent context and reached different conclusions from it. Fix: add an explicit reconciliation/merge step (owned by the orchestrator or a dedicated verifier) that compares outputs and resolves conflicts using defined rules, rather than hoping independent agents converge.

6. **A single agent could technically handle a task, but a colleague proposes splitting it into three specialized agents "for scalability." What should you push back on?**
   Answer: Push back on cost and complexity without a concrete justification — each additional agent roughly multiplies the number of LLM calls (and thus cost/latency) for a task that might fit in one well-scoped prompt or workflow, and every agent boundary adds a place where coordination can fail (duplicated work, contradictory outputs). "Scalability" is only a real justification if the sub-tasks genuinely need context isolation, specialization, or parallelism — not because more agents sounds more sophisticated.

## Watch

- [Multi-agent swarms with LangGraph](https://www.youtube.com/watch?v=JeyDrn1dSUQ) — LangChain. Teaches the swarm/handoff pattern for coordinating multiple agents.
- [Building more effective AI agents](https://www.youtube.com/watch?v=uhJJgc-0iTQ) — Anthropic. Covers orchestrator-worker and other multi-step agent design patterns.

## Further reading

- [Anthropic: Building effective agents](https://www.anthropic.com/research/building-effective-agents) — the orchestrator-workers pattern section.
- [LangGraph multi-agent concepts](https://langchain-ai.github.io/langgraph/concepts/multi_agent/) — supervisor, swarm, and network topologies.
