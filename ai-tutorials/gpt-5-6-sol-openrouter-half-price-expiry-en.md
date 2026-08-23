# OpenAI Didn't Cut the Price: GPT-5.6 Sol's 50% Off Expires September 18 and Excludes BYOK

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/gpt-5-6-sol-openrouter-half-price-expiry-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/gpt-5-6-sol-openrouter-half-price-expiry-en?utm_source=github&utm_medium=referral)**

Good news first: **the discount is real, and it is genuinely half off.**

GPT-5.6 Sol on OpenRouter went from $5 / $30 (per million input / output tokens) to **$2.50 / $15**. I pulled the raw data from OpenRouter's public API rather than reading a screenshot, and the numbers check out.

But three qualifiers fell off in transit, and every one of them hits your bill:

1. **OpenAI didn't cut the price** — OpenRouter and Vercel AI Gateway did
2. **It expires** on September 18
3. **Bring-your-own-key (BYOK) requests don't qualify**

If you're about to move production traffic, those three matter more than the headline number.

---

## 1. The Real Prices, and Two Easy Traps

OpenRouter API snapshot, taken **2026-08-19 00:50 UTC**:

| Line item | Promo | List |
|-----------|-------|------|
| Input | **$2.50** | $5.00 |
| Output | **$15.00** | $30.00 |
| Cache read | $0.25 | $0.50 |
| Cache write | $3.125 | — |
| Batch / Flex tier | $1.25 / $7.50 | $2.50 / $15 |
| **Long-context tier (prompt > 272K tokens)** | **$5.00 / $22.50** | — |
| Web search | $10 / 1000 calls | not discounted |

**Trap one: the 272K pricing cliff.** Once a prompt exceeds 272,000 tokens, input doubles to $5 and output rises to $22.50. In other words, **on the long-context tier you're paying undiscounted list price.** Sol's window is 1.05M tokens, so if your pattern is "stuff the whole codebase in," that tier is your actual unit price and the promotion is largely irrelevant to you. Build this cliff into any cost model.

**Trap two: it's a reasoning model, and reasoning tokens bill as output.** Sol supports `reasoning_effort` from medium up to ultra (ultra runs 4 agents in parallel by default). Output costs 6× input, and every reasoning token bills at the output rate — **so raising the effort tier costs you non-linearly.** This is the same class of problem we covered in [Qwen3.8 27B taking 21 minutes on its default setting](/articles/qwen3-8-27b-overthinking-en), except there it burned your time and here it burns your money.

The $5/$30 list price has three independent confirmations: OpenAI's launch page states "Sol is $5 input / $30 output"; OpenRouter's model page shows it struck through; and the Azure provider on that same page is still listed at $5/$30, undiscounted.

## 2. Whose Price Actually Changed

This is the correction that matters most.

- **Effective**: 2026-08-17 (OpenRouter's announcement tweet is timestamped 17:36 UTC)
- **Expires**: **2026-09-18**, reverting to list
- **Scope**: OpenRouter + Vercel AI Gateway only; requests routed to the OpenAI provider only; **non-BYOK requests only**
- **OpenAI's own API pricing is unchanged** at $5/$30. Neither Azure nor Bedrock followed

...

---

**[👉 Continue reading: OpenAI Didn't Cut the Price: GPT-5.6 Sol's 50% Off Expires September 18 and Excludes BYOK](https://tools.cooconsbit.com/en/articles/gpt-5-6-sol-openrouter-half-price-expiry-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
