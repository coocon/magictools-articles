# One Week With Claude Opus 5: The Data Says More Precise, the Vibes Say More Annoying — and It's the Same Thing

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/opus-5-week-one-verdict-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/opus-5-week-one-verdict-en?utm_source=github&utm_medium=referral)**

Claude Opus 5 launched on July 24 ($5/$25 per million tokens, scoring 61 on the Artificial Analysis Intelligence Index, edging out Fable 5 at 60 and GPT-5.6 Sol at 59). One week of honeymoon later, the community has moved into verdict season — and this time the verdict split cleanly in two.

On the data side, [CodeRabbit's code review benchmark](https://www.coderabbit.ai/blog/opus-5-model-review) delivered a coolly worded judgment: "a precision specialist, which pairs well with a recall model." On the vibes side, ChatPRD founder and How I AI host Claire Vo [opened her review](https://www.chatprd.ai/how-i-ai/my-surprising-verdict-on-claude-opus-5) with: "I *hate* working with it. And yet in a blind taste test, I ranked it above every other model — even Fable and my beloved GPT-5.6."

"More precise" and "more annoying" look like two stories. Lay out the numbers and you'll find they are **one**.

## The data thread: most precise ever measured — also leakier and louder

Start with CodeRabbit's methodology, because it's sturdier than most vendor benchmarks: 96 error patterns drawn from verified issues in real open-source pull requests (not synthetic bugs), each configuration run three times against three runs of the current production model mix.

The result is a textbook trade-off:

- **Actionable-comment precision 39.3% vs. the baseline's 35.2%** — the highest CodeRabbit has ever measured
- **Known-bug recall 55.2% vs. 61.1%** — more precise, yet it missed more real bugs
- **Nitpicks (low-value comments): roughly 92 vs. the baseline's 23** — four times as many
- Full-stream precision (all comment classes counted) **28.6%, below the baseline's 32.8%**

The effort-level experiment is even more interesting: dropped to the default reasoning effort, the model found the most issues overall — but full-stream precision sank to 26.4% and nitpicks hit 110. CodeRabbit's conclusion belongs in your notebook: **"More reasoning did not consistently produce a better review. Treat effort as a choice between failure modes."**

(The honest footnote: these are CodeRabbit's numbers from CodeRabbit's harness, and CodeRabbit sells code review — the "pair it with another model" recommendation happens to lead straight to its product shape. As we wrote in [the benchmarks piece](/articles/benchmarks-stopped-measuring-en): before the score, check who measured it. This data's value is its transparent methodology, not universal rankings.)

...

---

**[👉 Continue reading: One Week With Claude Opus 5: The Data Says More Precise, the Vibes Say More Annoying — and It's the Same Thing](https://tools.cooconsbit.com/en/articles/opus-5-week-one-verdict-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
