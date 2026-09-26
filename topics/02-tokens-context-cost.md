---
title: "02. Tokens, context windows, cost"
layout: default
nav_order: 3
---

# Tokens, context windows, and cost
{: .no_toc }

*~7 min read*

**🎯 Interview frequent**

## Why it matters

Every LLM API call you make is billed and bounded in units of tokens, not words or characters. If you don't understand tokenization and context windows, you'll misjudge cost, hit mysterious truncation bugs, and write prompts that silently waste money. This is the topic that turns "just call the API" into "engineer a cost-aware system."

## Core concepts

- **Tokens are sub-word units**, produced by a tokenizer (commonly a byte-pair-encoding or similar scheme). A token is roughly ¾ of an English word on average, but code, non-English text, and rare words often tokenize much less efficiently (more tokens per "unit of meaning").
- **The context window is a hard ceiling** on total tokens — input plus output — the model can attend to in one call. Modern frontier models range from ~128K to 1M+ tokens, but bigger windows don't mean you should fill them: relevant information gets harder for the model to use well as irrelevant tokens pile up ("lost in the middle" effects).
- **Cost is (usually) per-token, and input/output are priced differently** — output tokens are typically several times more expensive than input tokens. Caching (prompt/context caching) can make repeated large inputs (e.g., a long system prompt or document) dramatically cheaper on subsequent calls.
- **Truncation is silent by default.** If you exceed the context window, most APIs error out or the client library truncates — either way, you can lose critical context (like the beginning of a long conversation) without an obvious symptom in the output.
- **Counting tokens before you send them** is a real engineering step — most providers ship a tokenizer library so you can measure prompt size ahead of time rather than discovering the limit at request time.
- **Latency scales with output length**, since decoding is sequential (one token at a time). A prompt that asks for a shorter, more targeted answer is often both cheaper and faster, not just cheaper.

## Mental model

```
[ system prompt ] + [ retrieved context / tools / history ] + [ user message ]  <-- input tokens ($)
                                        |
                                  context window (hard cap, input + output)
                                        |
                              [ model generates response ]  <-- output tokens ($$$)
```

Budget the context window like memory in an embedded system: every component (system prompt, few-shot examples, retrieved docs, chat history) is competing for a fixed, metered resource.

## Interview questions

1. **Why might a prompt with the same number of "words" cost very differently depending on the language or content type?**
   Answer: Tokenizers are trained on a specific data distribution (usually English-heavy). Code, non-Latin scripts, and rare/technical vocabulary often split into more tokens per word than common English text, so token count (and thus cost/latency) isn't a simple function of word or character count.

2. **Why doesn't simply using a model with a 1M-token context window solve your context problems?**
   Answer: Larger windows cost more (more input tokens billed) and latency grows with context size. Models also don't attend equally well to all parts of a huge context — relevant details in the middle of a very long input can get under-weighted ("lost in the middle"), so retrieval/filtering to relevant content usually beats just stuffing everything in.

3. **What's prompt/context caching and when is it worth using?**
   Answer: Many providers let you mark a prefix of your prompt (e.g., a long system prompt or reference document) as cacheable, so repeated calls that reuse that prefix are charged at a reduced rate and processed faster. It's worth it whenever you're sending the same large context repeatedly with only the tail (like the user's latest message) changing — common in agent loops and chat with a long system prompt.

4. **Why are output tokens typically priced higher than input tokens, and what design implication does that have?**
   Answer: Generation is sequential and requires a full forward pass per token, making it more compute-intensive than processing input tokens (which can be parallelized). Implication: asking for concise, structured output (rather than verbose prose) is a direct cost and latency lever, and techniques like requesting a short answer or capping `max_tokens` are legitimate optimizations, not just style preferences.

5. **You have a chat app where conversations keep growing. What are your options once the conversation approaches the context limit?**
   Answer: Truncate oldest turns, summarize/compact older history into a shorter representation (see [Context management & compaction](../07-context-management-compaction/)), retrieve only relevant past turns instead of resending everything, or move older context into external memory/RAG and re-fetch only what's relevant to the current turn.

6. **Why can two different models report different token counts for the exact same input text?**
   Answer: Each model family uses its own tokenizer with its own vocabulary, trained on its own data distribution — there's no universal tokenization standard. The same string can split into a different number of tokens depending on which model's tokenizer processes it, which is why you must count tokens with the specific provider/model's tokenizer, not a generic word-count heuristic, when predicting cost or checking you're under a context limit.

## Watch

- [Let's build the GPT Tokenizer](https://www.youtube.com/watch?v=zduSFxRajkE) — Andrej Karpathy. A from-scratch walkthrough of how BPE tokenization works, why tokens don't map cleanly to words, and how tokenization causes weird model behavior.
- [Deep Dive into LLMs like ChatGPT](https://www.youtube.com/watch?v=7xTGNNLPyMI) — Andrej Karpathy. Also covers context window limits and inference cost tradeoffs in practice (see the sections on inference).

## Further reading

- [OpenAI: Tokenizer](https://platform.openai.com/tokenizer) — interactive tool to see how text splits into tokens.
- [Anthropic docs: Prompt caching](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching) — how context caching works in practice.
