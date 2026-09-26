# LLM & Agent Engineering Study Guide

A practical study guide for software engineers learning LLM and agent engineering — for building real systems, and for interview prep.

**Live site:** https://cliffweng.github.io/llm-agent-eng-study-guide/

## Roadmap

16 topics, one file each under [`topics/`](topics/), ordered foundational → advanced:

1. LLM basics for SWEs
2. Tokens, context windows, cost
3. Prompting & structured outputs
4. Tools / function calling
5. Agents & planning loops
6. Memory (short / long / episodic)
7. Context management & compaction
8. State machines & workflows
9. Multi-agent / inter-agent communication
10. RAG fundamentals
11. Advanced RAG (chunking, hybrid, eval)
12. Embeddings & vector DBs
13. Evaluation & observability
14. Post-training (SFT, RLHF/RLAIF, preference tuning)
15. Safety, guardrails, trust
16. Production patterns (caching, queues, human-in-the-loop)

## Interview hotspots

Every topic page carries a badge (🎯 Interview frequent / Interview occasional / Background) so you know where to spend prep time. If you're short on time, prioritize these:

- **Tokens, context windows, cost** — every system design or "how would you build X" question runs into cost/latency tradeoffs; interviewers use it to check you think past "just call the API."
- **Prompting & structured outputs** — the most hands-on skill in the guide; commonly probed via "how would you get reliable JSON out of a model" or prompt-injection scenarios.
- **Tools / function calling** — the prerequisite for every agent question; interviewers check you know the model never executes anything itself.
- **Agents & planning loops** — the single most-asked conceptual topic right now; expect "what's an agent, really" and loop-governance questions.
- **Memory** — frequently paired with context management to see if you understand the across-session vs. within-session distinction.
- **Context management & compaction** — long-running agent sessions are a hot production pain point; expect questions on compaction risk and what to keep verbatim.
- **Multi-agent / inter-agent communication** — orchestrator-worker and swarm patterns come up whenever "how would you scale this agent" is asked.
- **RAG fundamentals & Advanced RAG** — still the most common way companies actually ship LLM features, so expect concrete pipeline and debugging questions.
- **Embeddings & vector DBs** — the infrastructure layer under RAG/memory; interviewers use it to gauge how deep your understanding actually goes.
- **Evaluation & observability** — the "how do you know it's not just vibes" question; increasingly a signal of production maturity, not just academic rigor.
- **Production patterns** — caching, retries, and human-in-the-loop gating are where "have you actually shipped this" gets tested.

**Occasional** (still worth knowing, less likely to anchor a whole interview): LLM basics, state machines & workflows, safety/guardrails/trust.

**Background** (good to understand conceptually, rarely probed in depth): post-training (SFT/RLHF/DPO) — know the shape of it, not the math.

This split is a judgment call based on what shows up in practice today, not a guarantee for any specific interview loop — adjust your prep if a role is unusually research- or safety-focused.

## How to use this guide

Each topic page is designed to be read in **~10 minutes** and follows the same structure: why it matters, core concepts, a mental model, 3-5 interview questions with brief answer keys, and a short list of verified YouTube videos. Read them in order, or jump straight to what you need. No backend, no auth, no sign-up — just read the pages.

## How to contribute

See [CONTRIBUTING.md](CONTRIBUTING.md). In short: one topic per file, keep it under ~10 minutes to read, and only link to sources (especially YouTube videos) you've personally verified exist. Open an issue before proposing new topics or restructuring the curriculum.

## Decisions

These are the product locks this guide was built against — echoed here so future contributors don't accidentally relitigate them:

- **Audience**: software engineers, learning + interview prep. Not a general AI-for-beginners course.
- **Time-boxed**: every topic is readable in 10 minutes or less. Depth is sacrificed for scannability; "further reading" links are where depth lives.
- **Learning + interview prep in one page**: each topic pairs core concepts with interview questions, rather than splitting them into separate tracks.
- **Real links only**: every YouTube link is verified to exist before being added. No invented URLs, ever.
- **Static site, GitHub Pages, Just the Docs**: no backend, no auth, no Vercel. Cheap to host, cheap to maintain, easy to contribute to via plain Markdown + front matter.

## Enabling GitHub Pages

Just the Docs is configured via `_config.yml` (remote theme). After this lands on `main`:

1. Repo **Settings → Pages**.
2. **Build and deployment → Source**: **Deploy from a branch**.
3. Branch: `main` / folder: `/ (root)`.
4. Save. Site publishes to https://cliffweng.github.io/llm-agent-eng-study-guide/


## Local preview

```bash
bundle install
bundle exec jekyll serve
```

Then open `http://localhost:4000`.

## License

[MIT](LICENSE)
