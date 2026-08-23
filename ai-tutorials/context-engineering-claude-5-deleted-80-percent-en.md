# Anthropic Deleted 80% of Claude Code's System Prompt. The Evals Didn't Move.

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/context-engineering-claude-5-deleted-80-percent-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/context-engineering-claude-5-deleted-80-percent-en?utm_source=github&utm_medium=referral)**

On July 24, Anthropic published a post by Claude Code team member Thariq Shihipar whose core claim is one sentence:

> For models like Claude Opus 5 and Claude Fable 5, we removed **over 80%** of Claude Code's system prompt **with no measurable loss** on our coding evaluations.

It hit 434 points and 343 comments on Hacker News. The reading that spread fastest was "see, prompts should be shorter."

That reading is wrong, and expensively so. **What got deleted was not information. It was constraints.** Those two look alike, and deleting the wrong one makes your agent worse.

This piece does three things: clarify what Anthropic actually removed; supply the **API-layer mechanics the official post omits** — why "progressive disclosure" is not merely a stylistic recommendation but sits on a hard cost structure; and surface the two unresolved disagreements from those 343 comments.

## 1. What they actually deleted

The post lists six then/now pairs. Put them together, because **the dividing line only emerges once you see all six**:

| Then | Now |
|---|---|
| Give Claude rules | Let Claude use judgement |
| Give Claude examples | Design interfaces |
| Put it all upfront | Use progressive disclosure |
| Repeat yourself | Simple tool descriptions |
| Memory in CLAUDE.md | Auto-memory |
| Simple specs | Rich references |

A concrete deletion. The old system prompt contained this:

> In code: default to writing no comments. Never write multi-paragraph docstrings or multi-line comment blocks — one short line max. Don't create planning, decision, or analysis documents unless the user asks for them — work from conversation context, not intermediate files.

In the new prompt, that entire block became one line:

> **Write code that reads like the surrounding code: match its comment density, naming, and idiom.**

See the difference? The old version is **a stack of hard rules**. The new one is **a basis for judgement**.

The post is candid about why the deletion works: those constraints **were once necessary**. Without them, older models wrote comments that were incorrect in many cases, so the team had to accept the tradeoff of a blanket ban. And for some subset of prompts, the rule was simply wrong — a user may have their own preferences, and specific parts of very complex code genuinely need multi-line comment blocks.

**In other words, a substantial fraction of past prompt text was not describing the task. It was compensating for a capability gap.** Close the gap and that compensation layer stops being protection and becomes noise.

...

---

**[👉 Continue reading: Anthropic Deleted 80% of Claude Code's System Prompt. The Evals Didn't Move.](https://tools.cooconsbit.com/en/articles/context-engineering-claude-5-deleted-80-percent-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
