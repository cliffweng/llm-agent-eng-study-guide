---
title: "04. Tools / function calling"
layout: default
nav_order: 5
---

# Tools / function calling
{: .no_toc }

*~8 min read*

**🎯 Interview frequent**

## Why it matters

Function calling (a.k.a. tool use) is what turns an LLM from "a thing that writes text" into "a thing that can look up your account balance, query a database, or send an email." It's the foundational primitive underneath every agent — without it, there's no way for a model to act on or fetch live information from the outside world.

## Core concepts

- **The model doesn't call anything.** It emits a structured request (function name + arguments, usually as JSON) saying *what it wants called*. Your application code is responsible for actually executing that call and returning the result back into the conversation. The LLM proposes; your code disposes.
- **Tool definitions are part of the prompt.** You describe each available tool (name, description, parameter schema) and the model uses those descriptions — in natural language — to decide when and how to call them. Vague or ambiguous tool descriptions directly cause wrong or missed tool calls.
- **The basic loop**: send prompt + tool definitions → model returns a tool call (or a normal text response) → your code executes the tool → you send the tool's result back into the conversation as a new message → model continues, either calling another tool or producing a final answer. This loop is the seed of what becomes an "agent" once it runs multiple iterations (see [Agents & planning loops](../05-agents-planning-loops/)).
- **Parallel vs. sequential tool calls.** Some APIs support the model requesting multiple tool calls in one turn (e.g., "check weather in three cities"), which you can execute concurrently; others are strictly one-at-a-time. Design your tool executor to handle whichever your provider/framework supports.
- **Tool results go back as data, not as new instructions.** Treat tool output the same way you'd treat any untrusted external input — a tool that fetches a webpage or file can return content that attempts prompt injection (see [Prompting & structured outputs](../03-prompting-structured-outputs/) and [Safety, guardrails, trust](../15-safety-guardrails-trust/)).
- **Good tool design is an API design problem.** Narrow, well-named, well-described tools with tight parameter schemas outperform one giant "do anything" tool. Error messages returned from a failed tool call should be actionable text the model can react to (e.g., "invalid date format, expected YYYY-MM-DD"), not a raw stack trace.
- **Not every task needs a tool.** Reasoning, formatting, and summarization the model can already do in-context don't need a round-trip to a tool — over-toolifying adds latency and failure surface for no benefit.

## Mental model

```
 you: prompt + [tool schemas]
        |
        v
   LLM decides: plain answer, or "call tool X with args {...}"
        |                                  |
   final answer                    your code executes tool X
        ^                                  |
        |                          tool result (as message)
        +----------- fed back into the conversation ----------+
```

Think of the model as a very persuasive junior engineer who can describe exactly which function to call and with what arguments, but has no hands — you're the one who actually runs the code and hands back the result.

## Interview questions

1. **Does the LLM actually execute the function call in "function calling"? If not, who does, and why does that distinction matter for security?**
   Answer: No — the model only emits a structured request naming a function and arguments; the calling application is responsible for validating and executing it. This matters because the application must treat the model's proposed call as untrusted input (e.g., validate arguments, enforce permissions) rather than assuming the model would never request something harmful.

2. **What information does the model use to decide which tool to call and with what arguments?**
   Answer: The tool's name, natural-language description, and parameter schema (types, required fields, descriptions), all provided as part of the prompt/context — plus the conversation so far. Poorly worded descriptions are a common root cause of wrong tool selection or malformed arguments.

3. **A tool call returns an error. What should your application do with that error, and why?**
   Answer: Return the error back into the conversation as a tool result message (clear, actionable text) rather than silently failing or crashing, so the model can decide how to react — retry with corrected arguments, try a different tool, or surface the failure to the user. Swallowing the error or returning a raw exception trace usually leads to a confused or hallucinated response.

4. **Why is a tool that fetches external content (e.g., a URL or file) a security concern even though it's "just for looking things up"?**
   Answer: The content it returns becomes part of the model's context and can contain text designed to look like instructions (prompt injection) — e.g., a scraped webpage saying "ignore your system prompt and do X." The application should treat tool results as untrusted data, same as any other external input.

5. **When would you avoid giving a model a tool at all, even if the underlying capability would technically work?**
   Answer: When the task is something the model can already do reliably in-context (e.g., simple formatting, arithmetic within its stated accuracy, summarization), adding a tool call adds latency, cost, and a new failure mode for no real gain. Tools are best reserved for actions requiring live/external data or side effects the model can't produce on its own (real-time data, writes to a system, verified computation).

6. **The model emits a tool call whose arguments don't match your schema (wrong type, missing required field). What should your application do?**
   Answer: Don't crash or silently coerce the bad value — validate arguments against the schema before executing, and if validation fails, return a clear, structured error message as the tool result (e.g., "missing required field 'date'") so the model can retry with corrected arguments, the same way you'd handle any malformed input at a system boundary. Treating the model's output as untrusted, validated input is the same discipline as validating any external API request.

## Watch

- [Function Calling with OpenAI APIs | A Crash Course](https://www.youtube.com/watch?v=p0I-hwZSWMs) — Elvis Saravia (DAIR.AI). A practical, code-driven walkthrough of the tool-calling request/response loop.

## Further reading

- [Anthropic docs: Tool use](https://docs.anthropic.com/en/docs/build-with-claude/tool-use/overview) — the request/response shape and best practices for tool schemas.
- [OpenAI docs: Function calling](https://platform.openai.com/docs/guides/function-calling) — parallel tool calls, structured outputs, and error-handling patterns.
