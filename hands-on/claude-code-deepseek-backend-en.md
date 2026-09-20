# Running Claude Code on DeepSeek: Everything Works, But the Cost Readout Lies by 38x

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-deepseek-backend-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-deepseek-backend-en?utm_source=github&utm_medium=referral)**

DeepSeek's API [speaks the Anthropic format](https://api-docs.deepseek.com/guides/anthropic_api) at `https://api.deepseek.com/anthropic`. Three environment variables and Claude Code talks to DeepSeek instead of Anthropic.

Plenty of posts tell you those three variables. Almost none tell you **what degrades**. That is the only question worth asking — an agentic coding tool is not a chat box, it lives or dies on tool calls, multi-turn state, and whether the numbers on your screen mean anything.

So I ran it. Claude Code **2.1.270**, macOS, in an isolated config directory that never touches my real setup. The Claude Code runs below all target `deepseek-v4-pro`; `deepseek-flash` appears in the raw-endpoint mapping tests and as the Haiku slot.

Short version: **the functionality is fine, the cost readout is not.** Claude Code told me I had spent $1.71. DeepSeek charged me ¥0.32.

## Background: why anyone wants this

Two reasons, and they pull in different directions.

The boring one is price. DeepSeek V4 Pro lists at **$1.32 / 1M input tokens** at peak and **$0.66 off-peak**, against Claude's per-million rates in a different bracket entirely. For a tool that re-sends a large system prompt on every single turn, that ratio compounds fast.

The interesting one is that Claude Code is, at this point, the most capable agentic harness in wide use — and it is the part you cannot easily rebuild. Being able to keep the harness and swap the engine underneath is genuinely useful, if the swap is clean.

Whether it is clean is an empirical question, so here is the measurement.

## What I tested, and what I deliberately did not

**Tested:** connectivity, the model-name mapping, the local tool matrix, subagent spawning, prompt caching across turns, and real billed cost against the actual account balance.

**Not tested (and why):**

- **MCP servers.** DeepSeek's compatibility table lists the API's `mcp_servers` field as *Ignored*, but that field is for Anthropic's **server-side** MCP. Claude Code's MCP is local stdio and never touches it. Testing the API field would prove nothing about the tool you actually use, and properly testing local MCP needs its own article.
- **Long-context behaviour at 1M.** DeepSeek advertises a 1M context window. Claude Code reports `contextWindow: 200000` for the model. Resolving which one wins needs a dedicated filling experiment; I am not going to guess from a metadata field.
- **Vision.** The pricing page states v4-pro does **not** support vision while flash does. I did not exercise image input.

...

---

**[👉 Continue reading: Running Claude Code on DeepSeek: Everything Works, But the Cost Readout Lies by 38x](https://tools.cooconsbit.com/en/articles/claude-code-deepseek-backend-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
