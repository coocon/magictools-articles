# DeepSeek Says 1M, Claude Code Says 200K: I Measured Both and Neither Number Is the Real Limit

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-deepseek-1m-context-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-deepseek-1m-context-en?utm_source=github&utm_medium=referral)**

DeepSeek's [pricing page](https://api-docs.deepseek.com/quick_start/pricing) lists a **1M context length** for both `deepseek-flash` and `deepseek-v4-pro`. Point Claude Code at DeepSeek's Anthropic-compatible endpoint (three env vars — I covered the setup and [what it costs you](/en/hands-on/claude-code-deepseek-backend-en) separately) and Claude Code reports this for the same model:

```json
"contextWindow": 200000
```

Two numbers, 5x apart, describing the same model. Which one governs?

I measured it. The answer is **neither**, and the way it fails is more interesting than either number.

## What I measured

Three separate questions that get conflated:

1. **Does the endpoint accept it?** — a hard API limit.
2. **Does the model actually use it?** — retrieval, not just acceptance.
3. **Does Claude Code let you get there?** — the client-side gate nobody documents.

All runs: Claude Code **2.1.270**, `deepseek-flash`, macOS, isolated `CLAUDE_CONFIG_DIR`. Needle-in-a-haystack uses a unique string planted at position zero (hardest position to retrieve when the haystack is huge) with the question at the very end. Every prompt carries a unique nonce so prompt caching can't contaminate the token counts — the first version of this experiment did not, and produced two rows with identical token counts for payloads differing 2x in size.

## Layer 1: DeepSeek's real ceiling is 2^20, not "1M"

Ladder on the raw endpoint, needle at position zero:

| Target | Real input tokens | HTTP | Wall | Needle found |
|---|---|---|---|---|
| 200K | 399,954 | 200 | 8.6s | ✅ |
| 400K | 799,823 | 200 | 15.8s | ✅ |
| 950K | 949,775 | 200 | 19.8s | ✅ |
| 1040K | **1,039,744** | 200 | 19.4s | ✅ |
| 1048K | — | **400** | 3.7s | — |

At **1,039,744 tokens** the model still pulled a string planted at the very beginning, in under 20 seconds. The long context is real, not a spec-sheet number.

The rejection tells you the exact limit:

```
This model's maximum context length is 1048576 tokens.
```

**1,048,576 = 2^20.** So "1M" is binary, not decimal — you get 48,576 tokens more than a literal million.

![Context ladder on the raw endpoint: 1,039,744 tokens accepted with the needle retrieved; 1,048,576 is the hard ceiling](https://cdn.tools.cooconsbit.com/uploads/articles/2026-09-20-deepseek-1m-context/01-ladder.png)

### The limit includes your output budget

Worth knowing before you size a request. Same input, only `max_tokens` changed:

| Input tokens | `max_tokens` | Result |
|---|---|---|
| 1,045,710 | 1,000 | ✅ 200 |
| 1,045,707 | 8,000 | ❌ 400 |

...

---

**[👉 Continue reading: DeepSeek Says 1M, Claude Code Says 200K: I Measured Both and Neither Number Is the Real Limit](https://tools.cooconsbit.com/en/articles/claude-code-deepseek-1m-context-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
