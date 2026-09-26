---
title: "13. Evaluation & observability"
layout: default
nav_order: 14
---

# Evaluation & observability
{: .no_toc }

*~8 min read*

**🎯 Interview frequent**

## Why it matters

"It seemed to work when I tried it" is not an engineering practice. LLM outputs are non-deterministic and prompt/model changes can silently regress cases that used to work. Evaluation and observability are what turn "vibes-based" LLM development into something you can actually trust, ship, and improve with confidence — the same way tests and monitoring do for traditional software.

## Core concepts

- **Evals are tests for probabilistic systems.** Instead of a single assert-equal, an eval runs your system against a representative set of inputs and scores the outputs against some criteria — exact match, a rubric, or a judge model — so you can detect regressions and measure improvement over time, not just anecdotally "try a few examples."
- **Three broad ways to score an output**: *exact/structural match* (does the output match a known correct answer or valid schema — cheap, precise, but only works for tasks with a clear ground truth), *human evaluation* (a person rates outputs — high quality signal, expensive and slow to scale), and *LLM-as-judge* (another LLM call scores the output against a rubric — scalable and fast, but introduces its own noise and biases and needs to be validated against human judgment before you trust it).
- **Online vs. offline evaluation.** Offline evals run against a fixed dataset before you ship a change (like CI). Online evaluation monitors real production traffic (e.g., sampling live outputs for quality scoring, tracking user feedback signals like thumbs up/down or edit rates) — you need both: offline to catch regressions before shipping, online to catch issues offline data didn't anticipate.
- **Observability/tracing** means capturing the full execution path of a request — every prompt, tool call, retrieved context, intermediate model output, and final response — so that when something goes wrong, you can inspect exactly what happened rather than guessing from the final output alone. This is especially critical for agents and RAG, where the final answer is downstream of several opaque intermediate steps.
- **A good eval set is built from real failures, not invented ones.** The most valuable eval cases usually come from production traces where something actually went wrong (a real user complaint, a real bad output you caught) — invented edge cases are useful too, but they don't replace grounding your eval set in what actually breaks.
- **Regression testing matters as much as absolute quality scores.** A single "quality: 8/10" number is less actionable than "this prompt change improved case type A but regressed case type B" — evals should be granular enough to tell you *what* changed, not just whether an aggregate score moved.
- **Cost and latency are eval dimensions too**, not just correctness — a change that improves accuracy by 2% but doubles latency or cost may not be a net win depending on the product.

## Mental model

```
              production traffic
                     |
                     v
   [ tracing: capture prompts, tool calls, context, outputs ]
                     |
        sample interesting/failing cases
                     |
                     v
        [ eval dataset: real inputs + expected/graded outputs ]
                     |
         run before shipping any prompt/model change
                     |
                     v
      exact match | human review | LLM-as-judge  --> pass/fail, scores, diffs vs. baseline
```

Think of tracing as your system's logs/APM equivalent, and evals as your test suite — except the "tests" here are graded on a spectrum rather than pass/fail, and the thing under test can behave differently on the same input from one run to the next.

## Interview questions

1. **Why is LLM-as-judge useful, and what's the catch?**
   Answer: It scales far better than human review — you can grade thousands of outputs against a rubric automatically and quickly. The catch: the judge model has its own biases and failure modes (e.g., favoring longer or more confident-sounding answers, being fooled by superficially plausible but wrong content), so judge scores need to be validated/calibrated against human judgment on a sample before you trust them as a proxy.

2. **Why do you need both offline and online evaluation — isn't a good offline eval set enough?**
   Answer: An offline eval set, however good, is necessarily a fixed snapshot of anticipated inputs — production traffic will surface novel inputs, edge cases, and failure patterns the eval set didn't anticipate. Online evaluation (sampling live traffic, tracking user feedback signals) catches what offline testing structurally can't, the same way monitoring in production catches things unit tests don't.

3. **A prompt change improves your aggregate eval score from 82% to 85%. Is that good news? What else do you need to know?**
   Answer: Not necessarily sufficient on its own — you need to know whether the improvement is broad or concentrated, and critically whether any previously-passing cases regressed even as the aggregate improved. A change that fixes 5 cases and breaks 2 different ones can raise the average while introducing a new, possibly worse-for-users failure mode; granular diffing against the previous run matters more than the single number.

4. **Why is tracing especially important for agents and RAG systems, more so than for a single-prompt LLM call?**
   Answer: Both involve multiple opaque intermediate steps (tool calls and their results, retrieved chunks, intermediate reasoning) between the input and final output — without capturing each step, a wrong final answer gives you no way to tell whether the failure was in retrieval, a tool call, an intermediate reasoning step, or the final generation. A single-prompt call has much less hidden machinery to go wrong in between.

5. **Where should your eval set's hardest cases come from, and why is that better than only writing cases you imagine in advance?**
   Answer: From real production failures/traces — actual inputs that broke the system or drew a user complaint — because these represent the failure distribution that actually occurs, which is often different (and weirder) than what you'd think to write by hand. Imagined edge cases are a useful supplement but shouldn't be the primary source, since they reflect your assumptions about failure rather than reality.

6. **How does evaluating a multi-step agent differ from evaluating a single-turn LLM call?**
   Answer: A single-turn call is graded mostly on its final output; an agent's eval needs to check the *trajectory*, not just the destination — did it call the right tools, in a reasonable order, did it recover from a failed tool call, did it stop within budget, and did intermediate steps stay grounded. Two agent runs can reach the same correct final answer via very different paths (one efficient, one wasteful or lucky), so tracing (see the Observability section above) is a prerequisite for agent eval, not just a debugging nice-to-have.

## Watch

- [How to Construct Domain Specific LLM Evaluation Systems](https://www.youtube.com/watch?v=eLXF0VojuSs) — Hamel Husain & Emil Sedgh, via AI Engineer. A practitioner-level walkthrough of building real eval systems for LLM applications, from one of the most respected voices in this space.

## Further reading

- [Hamel Husain: Your AI Product Needs Evals](https://hamel.dev/blog/posts/evals/) — widely-cited practical guide to building LLM evals.
- [RAGAS documentation](https://docs.ragas.io/) — open-source RAG-specific evaluation framework (context precision/recall, faithfulness, answer relevance).
