---
title: "05. Agents & planning loops"
layout: default
nav_order: 6
---

# Agents & planning loops
{: .no_toc }

*~9 min read*

**🎯 Interview frequent**

## Why it matters

"Agent" is one of the most overloaded words in this field, but the underlying idea is simple: instead of one prompt → one response, the model runs in a loop — observing, deciding, acting, and reassessing — until it decides the task is done. Understanding this loop demystifies almost every "AI agent" product and lets you evaluate whether agentic behavior is actually the right tool for a given problem (often it isn't).

## Core concepts

- **The core loop is: observe → think → act → observe the result → repeat.** This is often called ReAct (Reason + Act): the model interleaves reasoning traces with tool calls, using each tool result to decide the next step, until it produces a final answer or hits a stop condition.
- **An agent is a loop wrapped around tool calling** (see [Tools / function calling](04-tools-function-calling.html)) plus a termination condition and, usually, some notion of "am I done / did that work." Without tools, an "agent" is just a chatbot that talks to itself.
- **Planning strategies vary in how much structure they impose:**
  - *ReAct-style*: interleave one reasoning step and one action at a time, reactively.
  - *Plan-and-execute*: produce an upfront multi-step plan, then execute steps (optionally replanning if a step fails or reveals new information).
  - *Tree/graph search* (e.g., tree-of-thought): explore multiple candidate reasoning paths and prune, useful for problems with a clear way to score partial progress.
- **Termination and loop control are the hard, unglamorous part.** Real systems need max-iteration caps, cost/time budgets, and explicit "done" detection — an ungoverned agent loop can spin forever, burn budget, or take irreversible actions repeatedly. This is where most production agent bugs live, not in the reasoning quality.
- **Error recovery matters more than the happy path.** A good agent notices when a tool call failed or returned something unexpected and adapts (retries with different arguments, tries an alternate tool, asks for clarification) rather than plowing forward on bad information.
- **Autonomy is a dial, not a switch.** You can constrain an agent heavily (fixed sequence of allowed actions, human approval before side effects) or loosely (open-ended tool access, self-directed planning). More autonomy generally trades reliability and auditability for flexibility — pick the least autonomy that solves the problem (see [Production patterns](16-production-patterns.html) for human-in-the-loop gating).
- **Agents are not always the right answer.** A single well-crafted prompt, or a deterministic pipeline with one or two LLM calls, is often more reliable, cheaper, and easier to debug than a multi-step agent loop. Reach for an agent loop when the task genuinely requires adaptive, multi-step interaction with unpredictable intermediate results.

## Mental model

```
        +-------------------------------------------+
        |                                            |
        v                                            |
   [ observe state / prior result ]                  |
        |                                            |
        v                                            |
   [ reason: what should I do next? ]                |
        |                                            |
        v                                            |
   [ act: call a tool, or answer ] ---- result -------+
        |
        v (if "done" condition met, or max iterations hit)
   [ final answer ]
```

Think of an agent loop as a while-loop around an LLM call, where the loop body is "decide next action," the exit condition is "task complete or budget exhausted," and every iteration's input includes everything learned so far.

## Interview questions

1. **What's the actual difference between "an agent" and "a chatbot with tools"? Is there one?**
   Answer: Not a sharp technical one — an agent is generally understood as a system that runs an iterative loop of reasoning and acting across multiple steps toward a goal, using tool results to decide subsequent actions, versus a single-turn tool-call-and-respond pattern. The distinguishing feature is the loop with adaptive multi-step behavior, not any specific architecture.

2. **What are the concrete risks of an under-governed agent loop in production, and how do you mitigate them?**
   Answer: Risks include infinite or excessively long loops (cost/latency blowup), repeated irreversible side effects (e.g., sending the same email multiple times), and silent failure to make progress. Mitigations: hard caps on iterations/time/cost, idempotency for side-effecting tools, explicit "done" detection, and human-in-the-loop approval gates before high-stakes actions.

3. **When would you choose a plan-and-execute architecture over a reactive (ReAct-style) loop?**
   Answer: When the task benefits from an upfront, reviewable plan (e.g., a user or reviewer can sanity-check the plan before execution begins) or when replanning is expensive/rare relative to execution — plan-and-execute front-loads reasoning and reduces redundant "what should I do next" calls, at the cost of being less adaptive if early assumptions turn out wrong mid-execution.

4. **Give an example of a task where an agent loop is overkill, and explain what you'd use instead.**
   Answer: E.g., "extract these five fields from this document" doesn't need multi-step planning or tool use — a single structured-output prompt (see [Prompting & structured outputs](03-prompting-structured-outputs.html)) is more reliable, cheaper, and easier to test than wrapping it in an agent loop that could take unpredictable paths.

5. **How should an agent handle a tool call that fails or returns unexpected data?**
   Answer: The failure/unexpected result should be fed back into the model's context as information, not hidden — a well-designed agent loop treats "tool failed" as a normal observation and lets the model's next reasoning step decide whether to retry, use a fallback tool, adjust its plan, or surface the problem to the user, rather than crashing the loop or silently continuing with bad data.

6. **An agent's process crashes midway through a 10-step task. What design choice determines whether you can resume cleanly versus having to start over?**
   Answer: Whether the agent's state (which step it's on, what's been produced/decided so far) is persisted externally and serializable, rather than living only in an in-memory loop. If state is checkpointed after each step, you can resume from the last completed step; if it's implicit in a single long-running process's memory, a crash loses all progress and you must restart from the beginning (see [State machines & workflows](08-state-machines-workflows.html) for how explicit state enables this).

## Watch

- [Building more effective AI agents](https://www.youtube.com/watch?v=uhJJgc-0iTQ) — Anthropic. Anthropic's own talk on agentic design patterns and planning loops, companion to their written guide below.

## Further reading

- [Anthropic: Building effective agents](https://www.anthropic.com/research/building-effective-agents) — the accompanying written guide, covering workflow vs. agent patterns.
- [ReAct paper](https://arxiv.org/abs/2210.03629) — the original ReAct paper.
