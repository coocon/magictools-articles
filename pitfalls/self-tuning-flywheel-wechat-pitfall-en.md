# The Self-Reinforcing Flywheel Is Harder Than It Looks: 40 Days of an LLM Auto-Tuning Loop

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/self-tuning-flywheel-wechat-pitfall-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/self-tuning-flywheel-wechat-pitfall-en?utm_source=github&utm_medium=referral)**

## The idea

"Dev Breakfast" (码农早餐) is our daily tech newsletter on WeChat. One issue goes out at 8 a.m. every day, written by an LLM pipeline. On September 11 I added a self-optimizing loop on top of it. The plan:

1. Every day at 08:50, pull the official stats (reads, recommendation-feed reads, shares, follows, completion rate)
2. Score the "writing version" that produced yesterday's issue and compare it to its parent: below 0.8× rolls back, 1× or above is kept
3. An LLM reads the data and proposes changes to at most 3 of 25 typed knobs, which become the new version
4. The knobs cover topic-selection gates, title and opening hints, length, humor density, cover style, theme color, and follow-prompt copy

The verdict is code, not the LLM. The hard floors (fact-checking, real source links, no invented first-person experience) live outside the knobs, so no version can touch them. Better data → better writing → better data. The left foot steps on the right foot and you climb.

Looking back after 40 days, the flywheel never completed a real turn.

## The 40-day ledger

| | Number |
|---|---|
| Versions | 18 (12 proposed by the optimizer, 6 set manually) |
| Rolled back | 8, including 3 manual versions |
| Issues since the loop went live | 47, totaling 2,818 reads, 44 shares, 7 follows from the article |
| Issues with ≤ 50 reads | 28 (60%) |
| Share concentration | 21 of the 44 shares came from two issues (an eBPF agent cutting overhead by 90%, LiteLLM moving to a Rust core) |

Three details say more than the totals:

- **v10 had exactly the same knobs as its parent v6.** That day the optimizer produced no valid proposal and just reused the parent. It was rolled back anyway. That is an accidental A/A test: same configuration, two different days of articles, and the score gap crossed the rollback line.
- **Cover style went in circles**: dark → light (v3) → dark (v5) → flat (v9, rolled back) → light (v14, rolled back). The masthead tagline was proposed in v8 and v11 and rolled back both times. A human finally set it in v17.
- **"Drop the long deep-dive sections" was set three times by a human.** On 09-16, after reading 10 days of data, I cut them (v4). On 09-18 the optimizer rolled that back based on one issue's score. That issue ended at 387 reads, the highest since the loop launched. I set it again on 09-19 (v7), and it was rolled back again. The third attempt was v17 on 10-12.

## Why it never turned

### 1. Not enough signal: noise is bigger than the knob effect

...

---

**[👉 Continue reading: The Self-Reinforcing Flywheel Is Harder Than It Looks: 40 Days of an LLM Auto-Tuning Loop](https://tools.cooconsbit.com/en/articles/self-tuning-flywheel-wechat-pitfall-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
