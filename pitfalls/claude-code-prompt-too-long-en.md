# Claude Code \"Prompt is too long\" vs \"maximum context length\": Why Auto-Compact Works for One and Not the Other (Tested)

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-prompt-too-long-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-prompt-too-long-en?utm_source=github&utm_medium=referral)**

> **Short answers first**
>
> - **`Prompt is too long · …`**: Claude Code recognized a context overflow. In a multi-turn conversation it **auto-compacts older turns and retries**; usually you do nothing. If it says *single exchange cannot be compacted*, the system prompt, tool definitions or attachments themselves are too big — cut MCP tools or attachments, or `/clear`.
> - **`API Error: 400 This model's maximum context length is …`** (common with DeepSeek and gateways): Claude Code **doesn't recognize this text**, won't auto-compact, and every following turn fails the same way. Type `/compact` — tested, it recovers.
> - **Permanent fix**: tell Claude Code the model's real window so it compacts before hitting it:
>   - Model ID not starting with `claude-` (e.g. `glm-4.6`, `deepseek-…`): `export CLAUDE_CODE_MAX_CONTEXT_TOKENS=131072` (use your model's real limit)
>   - Gateway model ID starting with `claude-`: `export CLAUDE_CODE_AUTO_COMPACT_WINDOW=131072` (minimum 100000)
> - **An overflow about `max_tokens`**: nothing to do; Claude Code lowers `max_tokens` and retries automatically.

## Background

As a conversation grows, or when you attach many files, you eventually hit the context limit. In Claude Code that shows up in two very different ways:

```
Prompt is too long · the request is ~210000 tokens (limit 200000) but this conversation is only ~12807 tokens — …
```

```
API Error: 400 This model's maximum context length is 1048576 tokens. However, you requested 1053707 tokens (1045707 in the messages, 8000 in the completion).
```

Many people never see the first one because Claude Code compacts in the background. People on DeepSeek or a gateway see the second one over and over until they `/clear`. The difference isn't the model — it's **the error text the backend returns**.

## Why: matching on error text

The [errors docs](https://code.claude.com/docs/en/errors#prompt-is-too-long) mention that Bedrock's overflow text is `Input is too long for requested model.`, and *Before v2.1.217, Claude Code didn't recognize the Bedrock wording, so auto-compact never triggered on it*. Auto-compaction is triggered by **recognizing the text**.

The recognizers in the 2.1.285 binary, verbatim:

```js
function BWn(e){let n=e.toLowerCase();return n.includes("prompt is too long")||n.includes("input is too long for requested model")}
function jWn(e){return e.toLowerCase().includes("context window")}          // used for 413 only
function M0r(e){return e.toLowerCase().includes("input length and `max_tokens` exceed context limit")}
function kA(e){ … return BWn(e.message)||bI(e.message,"prompt_too_long")}
```

...

---

**[👉 Continue reading: Claude Code \"Prompt is too long\" vs \"maximum context length\": Why Auto-Compact Works for One and Not the Other (Tested)](https://tools.cooconsbit.com/en/articles/claude-code-prompt-too-long-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
