# Claude Designed Proteins From Scratch — And the Prompt Is Public

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-protein-binder-design-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-protein-binder-design-en?utm_source=github&utm_medium=referral)**

> Based on Anthropic's 2026 research post, *How Claude is accelerating protein design and analytical chemistry*. All block quotes are verbatim from the original.
> Source: https://www.anthropic.com/research/Claude-accelerates-protein-design

---

AI beating benchmarks in math stopped being interesting a while ago. Math verifies cheaply — run the checker, get an answer in seconds.

The hard frontier is the domains where verification is slow and expensive. You cannot grade a protein design with a script. You have to synthesize it, ship it to a wet lab, and wait weeks.

That's what this result is about. Anthropic gave Claude a protein binder design campaign, let it run unsupervised, sent the output to two outside labs for physical testing, and then published the prompt and every data point.

Ten things worth pulling out.

---

## 1. 22–35% Against a 10–15% Baseline

> "Claude (Mythos Preview and Opus 4.8) designed protein binders against 15 targets, and succeeded against 14 of them. Between 22% and 35% of its individual designs bound successfully, depending on the setup, compared to the 10-15% that is typical in protein design campaigns today."

Hit rate is the currency of this field. Design 100 candidates, count how many actually bind.

The field runs at 10–15%. Claude got 26.7% (Mythos Preview) and 22.6% (Opus 4.8) designing against all targets at once in a 48-hour session. Switch to single-target mode — one 24-hour session per target, closer to how a human protein engineer actually works — and Mythos Preview hits 35.1%.

Final tally: 354 confirmed binders against 14 of 15 targets, out of 1,320 designs.

Note what that isn't. It isn't nudging 10% to 12%. It's a doubling, in a field where every candidate costs real money to synthesize and every validation round costs weeks of lab time.

**My take:** Most AI-for-science announcements report either a benchmark number or "assisted an expert with X." This one reports the metric the industry already argues about, measured in a physical experiment. Picking the right metric is half the credibility.

---

## 2. Nobody Was Steering

> "After giving Claude the prompt, we left the model to execute autonomously. We provided no additional scientific, technical, or operational guidance after we initiated the campaigns."

This is the sentence that matters most.

Human involvement consisted of: approving network access requests, keeping the infrastructure alive, and placing the orders for lab validation. Scientific guidance: none. Technical guidance: none.

...

---

**[👉 Continue reading: Claude Designed Proteins From Scratch — And the Prompt Is Public](https://tools.cooconsbit.com/en/articles/claude-protein-binder-design-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
