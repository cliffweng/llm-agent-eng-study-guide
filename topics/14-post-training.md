---
title: "14. Post-training"
layout: default
nav_order: 15
---

# Post-training (SFT, RLHF/RLAIF, preference tuning)
{: .no_toc }

*~9 min read*

**Background**

## Why it matters

Pretraining gives you a model that can predict plausible next tokens over internet-scale text — it doesn't give you a model that reliably follows instructions, refuses harmful requests, or produces the format you want. Post-training is the set of techniques that turns a raw completion engine into "an assistant," and understanding it explains both why models behave the way they do and what your options are (fine-tuning vs. prompting vs. RAG) when you want to change that behavior.

## Core concepts

- **Supervised Fine-Tuning (SFT)** trains the pretrained model further on a curated dataset of (prompt, ideal response) pairs, usually written or curated by humans. This is what first teaches a base model to behave like it's following instructions and responding conversationally, rather than just continuing text.
- **RLHF (Reinforcement Learning from Human Feedback)** goes a step further: humans rank or compare multiple model outputs for the same prompt, that preference data trains a separate *reward model* to predict which outputs humans would prefer, and then the LLM is fine-tuned (traditionally via an RL algorithm like PPO) to produce outputs the reward model scores highly. This is what shapes qualities like helpfulness, tone, and harmlessness beyond what SFT alone achieves.
- **RLAIF (RL from AI Feedback)** replaces (or supplements) human preference labels with preference judgments from another AI model, making preference data much cheaper and faster to produce at scale — at the cost of inheriting whatever biases or blind spots the judging AI model has.
- **DPO (Direct Preference Optimization)** is a newer, more direct technique that skips training a separate reward model and running RL entirely — it directly optimizes the model on preference pairs (chosen vs. rejected response) using a simpler, more stable training objective, achieving similar goals to RLHF with a simpler pipeline. This is why DPO-style methods have become common alongside or instead of full RLHF.
- **Why this matters even if you never train a model yourself**: it explains real, observable behavior — why base (non-instruction-tuned) models behave so differently from chat models, why a model might be evasive on certain topics (RLHF-shaped refusal behavior), and why "the model was trained to prefer X" is a real, architecturally-grounded explanation rather than hand-waving.
- **Post-training is also where you decide between fine-tuning your own model vs. using prompting/RAG.** Fine-tuning (including lightweight approaches like LoRA) is worth considering when you need a consistent behavior/format/style change across huge volumes of requests where prompting is too costly or unreliable per-call — but it's a heavier, slower-to-iterate lever than prompting, and doesn't solve knowledge-freshness problems the way RAG does (see [RAG fundamentals](10-rag-fundamentals.html)).
- **Post-training can also cause regressions** ("alignment tax") — optimizing hard for one preference signal (e.g., being agreeable, or being cautious) can measurably hurt other qualities (e.g., raw capability on certain tasks, willingness to give direct answers). This tradeoff is an active, ongoing area of the field, not a solved problem.

## Mental model

```
[ pretrained base model ]  (predicts plausible text, no notion of "assistant")
            |
     supervised fine-tuning (SFT) on (prompt, ideal response) pairs
            |
            v
[ instruction-following model ]
            |
   preference data: humans (RLHF) or AI (RLAIF) rank/compare outputs
            |
   train via reward model + RL (classic RLHF), or directly via DPO
            |
            v
[ aligned assistant model ]  (helpful, follows instructions, refuses per policy)
```

Think of pretraining as teaching someone to read and write fluently from a huge library, SFT as showing them worked examples of how to actually answer questions helpfully, and RLHF/DPO as repeatedly giving them feedback on which of their answers people preferred until their instincts match what's wanted.

## Interview questions

1. **What's the practical difference in what SFT teaches a model versus what RLHF/preference tuning teaches it?**
   Answer: SFT primarily teaches the model *how to follow instructions and format responses* (via direct examples of ideal behavior); RLHF/preference tuning further shapes *which of several plausible responses is preferred* (tone, helpfulness, safety tradeoffs, subtle quality differences) using comparative feedback rather than single "correct" examples — it's a finer-grained shaping signal than SFT's example-based training.

2. **Why does a base (pretrained-only) model behave so differently from a chat-tuned model, given they're "the same" model architecturally?**
   Answer: A base model has only been trained to predict plausible continuations of internet-scale text — it has no post-training shaping it toward instruction-following, conversational format, or refusal behavior, so prompting it like a chat assistant often produces text that continues/completes the prompt rather than "answering" it. Post-training (SFT + RLHF/DPO) is specifically what installs assistant-like behavior on top of the same underlying pretrained weights.

3. **A model refuses a request you think is reasonable, or is oddly cautious. Is that a bug you can "prompt around," and what's actually happening underneath?**
   Answer: It's typically a learned behavior from post-training (RLHF/RLAIF shaping the model to be cautious around certain patterns), not a hardcoded rule — so it's sometimes possible to get a different response with different phrasing/context, but it reflects the alignment tradeoffs baked into training, not a simple bug. Understanding it as a trained preference (with an associated "alignment tax" tradeoff) explains why the behavior can be inconsistent and why providers actively tune this balance over time.

## Watch

- [Deep Dive into LLMs like ChatGPT](https://www.youtube.com/watch?v=7xTGNNLPyMI) — Andrej Karpathy. Covers the full training stack — pretraining, SFT, and RLHF — in one accessible walkthrough.
- [Aligning LLMs with Direct Preference Optimization](https://www.youtube.com/watch?v=QXVCqtAZAn4) — DeepLearning.AI, with Hugging Face's Lewis Tunstall and Edward Beeching. A focused deep dive specifically on DPO.

## Further reading

- [InstructGPT paper (Training language models to follow instructions with human feedback)](https://arxiv.org/abs/2203.02155) — the paper that established the modern SFT + RLHF recipe.
- [Direct Preference Optimization paper](https://arxiv.org/abs/2305.18290) — the original DPO paper.
