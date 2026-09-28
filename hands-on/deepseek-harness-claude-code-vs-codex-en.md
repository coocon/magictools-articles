# DeepSeek Harness Test: One Model, Three Harnesses — Claude Code 15/15, Codex CLI 15/15, Bare API 0/15 (and 5 Fake \"Done\"s)

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/deepseek-harness-claude-code-vs-codex-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/deepseek-harness-claude-code-vs-codex-en?utm_source=github&utm_medium=referral)**

People search for "DeepSeek harness" because they have noticed something odd: the *same* DeepSeek model feels brilliant in one coding tool and clumsy in another. The model didn't change. The **harness** did — everything wrapped around the model: the system prompt, the tool definitions and their names, the agent loop, the retry and timeout policy, and what happens when a tool's output is too big to fit.

So I held the model fixed and swapped the harness. One machine, one day (2026-09-28), one model — `deepseek-v4-pro` — and three ways of driving it:

| Group | Harness | DeepSeek endpoint |
|---|---|---|
| **A** | Claude Code 2.1.280 | `https://api.deepseek.com/anthropic` (Anthropic format) |
| **B** | Codex CLI 0.157.1 (`npx @openai/codex`) | `https://api.deepseek.com` with `wire_api = "responses"` |
| **C** | none — one bare `POST /chat/completions` with the task as the only user message | `https://api.deepseek.com/chat/completions` |

The short answer to "does the harness matter?" is: **it decides whether anything happens at all, it decides the bill by a factor of seven, and it decides what the model gets to see when things go wrong.**

![Same DeepSeek model, three harnesses: tool matrix, 3 rounds each](https://cdn.tools.cooconsbit.com/uploads/articles/2026-09-28-deepseek-harness/02-matrix.png)

## Background: why swap the harness and not the model

Most "DeepSeek vs X" comparisons change two things at once: the model *and* the tool around it. That makes the result uninterpretable. If DeepSeek in Codex is slower than Claude in Claude Code, is that DeepSeek or Codex?

DeepSeek is unusual in that it exposes **two** agent-friendly wire formats on the same account — an Anthropic-compatible endpoint (which is how you run [Claude Code on DeepSeek](/en/hands-on/claude-code-deepseek-backend-en)) and an OpenAI Responses-compatible endpoint. That makes it possible to put the two most widely used vendor harnesses, Claude Code and Codex CLI, on top of literally the same model and the same account, and measure the difference.

## The problem, broken down

"Harness" is fuzzy, so I split it into the pieces that could plausibly change an outcome, and measured each one:

1. **Tools** — which tools exist, what they are called, and whether the model uses them.
2. **Prompt weight** — how big the injected system prompt and tool schema are, and what that does to cost.
3. **Caching** — whether that weight gets re-billed on every run.
4. **Failure policy** — what happens on HTTP 500, 429, and a hung connection.
5. **Output truncation** — what the model is shown when a command prints 120KB.

...

---

**[👉 Continue reading: DeepSeek Harness Test: One Model, Three Harnesses — Claude Code 15/15, Codex CLI 15/15, Bare API 0/15 (and 5 Fake \"Done\"s)](https://tools.cooconsbit.com/en/articles/deepseek-harness-claude-code-vs-codex-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
