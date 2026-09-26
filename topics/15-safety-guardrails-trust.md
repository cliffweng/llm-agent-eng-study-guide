---
title: "15. Safety, guardrails, trust"
layout: default
nav_order: 16
---

# Safety, guardrails, and trust
{: .no_toc }

*~8 min read*

**Interview occasional**

## Why it matters

An LLM system that's occasionally wrong is a quality problem; an LLM system that can be manipulated into leaking data, taking unauthorized actions, or producing harmful content is a trust and liability problem. As soon as a model handles untrusted input, calls tools, or acts with any autonomy, safety stops being someone else's job (the model provider's) and becomes part of your system design.

## Core concepts

- **Prompt injection is the core new attack class LLM systems introduce.** Any untrusted text that enters the model's context — user input, retrieved documents, tool output, scraped web content — can contain instructions crafted to hijack the model's behavior (see [Prompting & structured outputs](03-prompting-structured-outputs.html) and [Tools / function calling](04-tools-function-calling.html)). There's no complete fix at the model level; mitigation is a system design problem.
- **Guardrails operate at multiple layers**: input filtering (block or flag clearly malicious/out-of-scope requests before they reach the model), output filtering (check the model's response before it's shown or acted on — for policy violations, PII leakage, or unsafe content), and structural constraints (limiting what tools/actions are even reachable, so a manipulated model has less it can do damage with).
- **Least privilege applies to agents just like it applies to services.** An agent should only have access to the tools, data, and permissions actually required for its task — not broad access "in case it's useful." This bounds the blast radius of a prompt injection or a model just being wrong.
- **Human-in-the-loop for high-stakes or irreversible actions** (sending money, deleting data, sending external communications) is a design pattern, not an admission of failure — the right level of autonomy depends on the reversibility and blast radius of the action, not on how capable the model seems (see [Production patterns](16-production-patterns.html)).
- **Red-teaming** means deliberately trying to break your own system — crafting adversarial inputs designed to trigger unsafe behavior, jailbreak restrictions, or extract system prompts/training data — before an actual attacker does. This should be an ongoing practice, not a one-time pre-launch check, since new jailbreak techniques keep emerging.
- **Trust also means calibrated confidence, not just "safe" content.** A system that states hallucinated facts with total confidence is a trust failure even without any adversarial input involved — citations, confidence signals, and clear "I don't know" behavior are part of building a system users can actually rely on.
- **Data privacy and PII handling** are safety concerns too: what user data ends up in prompts, what gets logged/traced (see [Evaluation & observability](13-evaluation-observability.html)), and whether that data could be exposed through a model's output to a different user or session.

## Mental model

```
 untrusted input (user, retrieved doc, tool output, scraped page)
            |
    [ input guardrail: filter/flag ]
            |
            v
      LLM (with least-privilege tools)
            |
    [ output guardrail: filter/check ]
            |
            v
  [ human approval gate, if high-stakes/irreversible ] --> action / response
```

Think of this the way you'd think about a service that accepts untrusted user input and has database access: you don't trust the input, you scope permissions tightly, you validate output before it has effects, and you gate anything destructive behind confirmation — none of this is new security thinking, it's the same discipline applied to a new kind of "user input" (natural language that can manipulate the processor itself).

## Interview questions

1. **What makes prompt injection fundamentally different from traditional injection attacks (like SQL injection), and why is there no complete fix at the model level?**
   Answer: In SQL injection, there's a clean boundary between code and data that can be enforced with parameterized queries; in an LLM, instructions and data share the same channel (natural language in the context window), and the model has no perfectly reliable way to distinguish "this is an instruction I should follow" from "this is data I'm processing" when both look like plain text. Mitigation has to happen at the system level (least privilege, output validation, structural constraints) because the model alone can't fully solve it.

2. **Why does "least privilege" matter more, not less, once a system has an LLM agent making tool-calling decisions?**
   Answer: An agent's tool calls are influenced by model reasoning that can be wrong or manipulated (via prompt injection or just an ordinary mistake) — if the agent only has access to the narrow set of tools/data/permissions its task actually requires, the worst-case impact of a bad decision is bounded. Broad access "just in case" turns an ordinary model mistake or a successful injection into a much bigger incident.

3. **Why should red-teaming be an ongoing practice rather than a one-time pre-launch check?**
   Answer: Jailbreak and injection techniques continue to evolve after launch, and a system that was robust against known attack patterns at launch time can become vulnerable to newly discovered techniques — treating red-teaming as continuous (like ongoing security testing/monitoring) rather than a one-time gate catches drift that a single pre-launch review can't.

4. **Give an example of a "trust" failure that has nothing to do with adversarial attacks.**
   Answer: A model confidently stating a hallucinated fact with no hedge or citation, or a RAG system presenting an answer without indicating which source it came from — a user has no way to distinguish a well-grounded answer from a confidently-stated wrong one, which erodes trust even when no one was trying to attack the system.

## Watch

- [AI Safety in Practice – Red Teaming, Guardrails, and Monitoring for LLM Systems](https://www.youtube.com/watch?v=HrZMrT1KEkU) — GenAI Summit. Practical coverage of red-teaming, guardrail layers, and monitoring for deployed LLM systems.

## Further reading

- [OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/) — the standard reference for LLM-specific vulnerability classes, including prompt injection.
- [Anthropic: Responsible Scaling Policy](https://www.anthropic.com/news/anthropics-responsible-scaling-policy) — how a frontier lab thinks about safety guardrails at the model-provider level.
