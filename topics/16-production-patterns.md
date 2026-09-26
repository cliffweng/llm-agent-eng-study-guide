---
title: "16. Production patterns"
layout: default
nav_order: 17
---

# Production patterns (caching, queues, human-in-the-loop)
{: .no_toc }

*~8 min read*

**🎯 Interview frequent**

## Why it matters

Everything else in this guide is about making an LLM system work correctly. This topic is about making it work reliably, affordably, and at scale — the unglamorous infrastructure patterns that separate a working demo from a system that survives real traffic, real costs, and real failure modes.

## Core concepts

- **Caching** is the single highest-leverage cost/latency optimization available. This includes exact-match caching (identical requests return a stored response instead of a new LLM call), semantic caching (near-duplicate requests, matched via embedding similarity, reuse a prior response), and prompt/context caching (providers charge less and respond faster for repeated large prompt prefixes — see [Tokens, context windows, cost](../02-tokens-context-cost/)). Caching is especially valuable for high-traffic, low-variance requests (e.g., common support questions).
- **Queues decouple request intake from LLM processing.** Instead of holding an HTTP connection open while a slow or rate-limited LLM call completes, requests go onto a queue and are processed asynchronously (with the result delivered via webhook, polling, or websocket). This smooths out traffic spikes, respects provider rate limits, and lets you retry failed calls without the original caller needing to resubmit.
- **Rate limiting and backpressure** are necessary because LLM providers impose their own rate/quota limits — your system needs to queue, throttle, or degrade gracefully rather than simply erroring out when you hit a provider's limit, especially under traffic spikes.
- **Retries need to be careful, not automatic-and-blind.** LLM calls can fail transiently (timeouts, rate limits) — retrying is often right, but retrying a call that has side effects (a tool call that sends an email or charges a card) without idempotency protection can cause duplicate side effects. Idempotency keys and side-effect-aware retry logic matter here the same way they do in any distributed system.
- **Human-in-the-loop (HITL)** inserts a person at specific points: approving a high-stakes action before it executes, reviewing/correcting low-confidence outputs before they're used, or providing feedback that improves the system over time. The design question is *where* to place the human checkpoint — too many gates kill the automation's value; too few risk unchecked errors on consequential actions (see [Safety, guardrails, trust](../15-safety-guardrails-trust/)).
- **Fallbacks and graceful degradation**: a secondary (cheaper/faster, or different provider's) model to fall back to if the primary is down, rate-limited, or too slow; a non-LLM deterministic fallback for critical paths where "the AI feature is temporarily unavailable" beats "the whole system is down."
- **Streaming responses** improve perceived latency (users see output as it's generated rather than waiting for the full response) and are table stakes for any user-facing chat-style interface — but streaming complicates things like output validation and structured-output parsing, since you often only have a complete, parseable object at the very end.
- **Cost observability** (tracking spend per user, per feature, per model) is a production necessity, not a nice-to-have — LLM costs scale with usage in a way that's easy to lose track of without deliberate tracking and alerting, and this ties directly into [Evaluation & observability](../13-evaluation-observability/)'s tracing infrastructure.

## Mental model

```
 request --> [ cache check ] --hit--> return cached response
                  | miss
                  v
           [ queue / rate limiter ]
                  |
                  v
         [ LLM call (with retry + fallback model) ]
                  |
                  v
   [ human approval gate, if high-stakes ] --> action / response
                  |
                  v
          [ cost + trace logging ]
```

Think of a production LLM system the way you'd think about any external, rate-limited, occasionally-slow third-party API dependency (because that's what it is) — cache aggressively, queue instead of blocking, retry carefully around side effects, and have a fallback for when it's unavailable.

## Interview questions

1. **What's the difference between exact-match caching and semantic caching for LLM responses, and when would you use each?**
   Answer: Exact-match caching returns a stored response only when the incoming request is identical (or normalized-identical) to a previous one — simple and safe, but only helps with literally repeated requests. Semantic caching matches near-duplicate requests via embedding similarity (e.g., "how do I reset my password" vs. "password reset help"), catching more cache hits at the cost of needing a similarity threshold tuned carefully to avoid returning a cached answer for a subtly different question.

2. **Why would you put LLM requests behind a queue instead of calling the provider synchronously from your request handler?**
   Answer: LLM calls can be slow and are subject to provider rate limits — holding a synchronous connection open for each risks timeouts and doesn't handle traffic spikes gracefully. A queue decouples intake from processing, lets you control throughput to respect rate limits, and enables retries without the original caller needing to resubmit the request.

3. **Why is blindly retrying a failed LLM call dangerous if that call included a tool call with a side effect?**
   Answer: If the side effect (e.g., sending an email, charging a payment) actually succeeded but the response indicating success was lost (a timeout after the action completed), a naive retry would repeat the side effect — duplicate email, duplicate charge. This needs idempotency keys or side-effect-aware retry logic, the same defensive pattern used for any distributed system with at-least-once delivery semantics.

4. **How would you decide where to place a human-in-the-loop approval gate in an agent workflow?**
   Answer: Based on the reversibility and blast radius of the action being gated — irreversible, high-impact, or hard-to-undo actions warrant a gate; frequent, low-stakes, easily-reversible actions shouldn't be gated, or the human approval step becomes a bottleneck that negates the value of automating the task at all. It's the same reversibility-based judgment call used for the "risky actions" guidance in agentic coding tools.

5. **Why does streaming a response complicate structured output validation, and how would you handle it?**
   Answer: Structured output (e.g., JSON matching a schema) usually can't be fully validated until the entire object has been streamed and is syntactically complete — you can't parse/validate a half-received JSON object mid-stream. Common approaches: stream raw text to the user for perceived responsiveness while buffering internally for validation, or use incremental/partial-JSON parsers that can validate as much of the structure as has arrived so far and finalize validation once the stream ends.

6. **If you fall back to a secondary model when the primary is down or rate-limited, what new problems can that introduce?**
   Answer: The fallback model may follow instructions differently, produce a different output format, or simply be lower quality — if your prompt or downstream parsing was tuned against the primary model's quirks, silently switching models mid-request can produce subtly wrong or malformed output rather than a clean failure. Mitigations: test your prompts against the fallback model ahead of time (not just the primary), and treat a fallback response as lower-confidence (e.g., flag it for review or a stricter validation pass) rather than assuming parity with the primary.

## Watch

- [Building LLM Applications for Production](https://www.youtube.com/watch?v=spamOhG7BOA) — Chip Huyen, LLMs in Production Conference. Covers the practical challenges of moving an LLM application from prototype to production, including cost, latency, and reliability tradeoffs.

## Further reading

- [Chip Huyen: Building LLM applications for production](https://huyenchip.com/2023/04/11/llm-engineering.html) — the widely-cited written companion covering these tradeoffs in more depth.
- [Anthropic docs: Prompt caching](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching) — caching mechanics referenced above.
