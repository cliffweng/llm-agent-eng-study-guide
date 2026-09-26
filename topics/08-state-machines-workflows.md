---
title: "08. State machines & workflows"
layout: default
nav_order: 9
---

# State machines & workflows
{: .no_toc }

*~8 min read*

**Interview occasional**

## Why it matters

Fully autonomous agent loops (see [Agents & planning loops](05-agents-planning-loops.html)) are flexible but unpredictable — the exact same input can take a different path through the loop on different runs. A lot of production LLM systems don't need that flexibility; they need a small, fixed set of steps executed reliably, with LLM calls plugged in at specific points. That's a workflow, and modeling it explicitly as a state machine or graph makes the system testable, debuggable, and far easier to reason about than an open-ended loop.

## Core concepts

- **Workflow vs. agent is a spectrum, not a binary.** A workflow has a predetermined control flow (the code decides what step runs next); an agent has the model decide the control flow dynamically at each step. Most reliable production systems are workflows with agentic sub-steps, not fully agentic end-to-end.
- **Modeling as an explicit graph/state machine** (nodes = steps, edges = transitions, sometimes conditional) makes the possible execution paths enumerable and inspectable — you can draw it, test each node in isolation, and reason about what states are reachable, unlike a freeform loop.
- **Common workflow patterns**: prompt chaining (output of step N feeds step N+1), routing (classify the input, then dispatch to a specialized path), parallelization (fan out independent sub-tasks, fan back in), and orchestrator-worker (a coordinating step delegates to specialized steps and assembles results).
- **Explicit state makes retries and resumability possible.** If a workflow's state (what step it's on, what's been produced so far) is persisted and inspectable, a failure at step 4 can resume from step 4 rather than restarting the whole thing — critical for anything long-running or expensive.
- **Conditional edges let you keep most of the determinism of a workflow while allowing an LLM call to decide *which* branch to take** (e.g., classify a support ticket, then route to a different fixed sub-flow per category) — this is usually a better trade-off than making the whole flow agentic.
- **Frameworks like LangGraph** formalize this as a graph of nodes and edges with persisted state, giving you built-in support for conditional routing, checkpointing/resumability, and human-in-the-loop interrupts — but the underlying concept (explicit state machine around LLM calls) applies whether or not you use a framework.

## Mental model

```
        +-------------+       classify        +----------------+
        |   START     | ---------------------> |  route: type A |----+
        +-------------+                        +----------------+    |
                                                +----------------+    |
                                                |  route: type B |----+---> [ merge ] -> END
                                                +----------------+    |
                                                +----------------+    |
                                                |  route: type C |----+
                                                +----------------+
```

Think of a workflow as a flowchart you'd draw on a whiteboard before writing any code — each box is a step (which may or may not call an LLM), each arrow is a transition, and some arrows are conditional on what a step produced.

## Interview questions

1. **What's the practical difference between "a workflow with LLM steps" and "an agent," and why would you pick one over the other?**
   Answer: In a workflow, the application code determines the sequence of steps (possibly with conditional branches decided by an LLM classification); in an agent, the model itself decides what to do next at each iteration. Workflows are more predictable, testable, and debuggable — pick them when the task has a knowable structure. Agents are more flexible for open-ended tasks where the right sequence of actions can't be predetermined.

2. **Why does modeling a system as an explicit state machine make it easier to add retries?**
   Answer: If the state machine's current node/step and accumulated outputs are persisted, a failure can be handled by resuming from the last successful state rather than restarting the whole process — this requires the state to be externally visible and serializable, which an implicit code-only control flow (e.g., a deeply nested function call chain) doesn't naturally give you.

3. **Describe the "routing" workflow pattern and give an example of when you'd use it instead of a single generic prompt.**
   Answer: Routing uses an initial (often cheap/fast) classification step to determine which of several specialized downstream paths to run — e.g., classify a support ticket as billing/technical/account and send it down a different prompt/tool chain per category. This beats one generic prompt when categories need meaningfully different handling (different tools, different tone, different data sources) and a single prompt trying to do all of it becomes unwieldy or error-prone.

4. **What's the benefit of a human-in-the-loop "interrupt" node in a workflow graph, compared to just asking the LLM to ask the user a question?**
   Answer: An explicit interrupt node pauses execution at a known state and hands control back to a human/external system in a structured, resumable way (the workflow's state is preserved and can resume exactly where it left off) — versus relying on the model to "remember" to ask and the surrounding code to correctly detect and handle that as a special case, which is more fragile and harder to guarantee.

## Watch

- [LangChain Academy: Introduction to LangGraph](https://www.youtube.com/watch?v=29XE10U6ooc) — LangChain. Introduces the graph/state-machine model for building LLM workflows, including conditional routing and persisted state.

## Further reading

- [Anthropic: Building effective agents](https://www.anthropic.com/research/building-effective-agents) — the workflow patterns section (prompt chaining, routing, parallelization, orchestrator-worker) this topic draws from.
- [LangGraph documentation](https://langchain-ai.github.io/langgraph/) — graphs, state, checkpointing, and human-in-the-loop interrupts.
