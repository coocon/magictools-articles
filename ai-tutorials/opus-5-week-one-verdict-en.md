---
title: "One Week With Claude Opus 5: The Data Says More Precise, the Vibes Say More Annoying — and It's the Same Thing"
slug: opus-5-week-one-verdict-en
summary: "A week in, the verdict on Claude Opus 5 is unusually split. CodeRabbit's 96-real-bug benchmark found the highest precision it has ever measured — alongside lower recall and 4x the nitpicks. Claire Vo coined 'neurotic AF' and 'Claudeslop' to complain about it, then ranked it first in her own blind test. Read together: precision up, recall down, verbosity, timidity, and refusing to touch someone's branch are five readings of one knob. Plus a practical guide to choosing failure modes."
category: ai-tutorials
tags: [Claude, Opus 5, Anthropic, code review, CodeRabbit, reasoning effort, LLM, AI coding]
coverImage: ""
status: published
locale: en
source: authored
translationSlug: opus-5-week-one-verdict
---

# One Week With Claude Opus 5: The Data Says More Precise, the Vibes Say More Annoying — and It's the Same Thing

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

## The vibes thread: neurotic AF, and the obnoxious colleague who won the blind test

Vo contributed two terms to this model that may stick in the industry lexicon.

The first is **"neurotic AF"**: "It's so timid, so apologetic, and so scared." Her signature example: a one-line merge conflict the model **refused to touch** — on the grounds that the branch belonged to someone else and it didn't want to disturb their local work. She also ran a personality test: asked whether it thought it had passed, Opus 5 answered that it *hoped* it had. Her description: "a sad, self-deprecating little neurotic agent that needs to heal its inner child" — while the control, GPT-5.6 Sol, was "all vibes."

The second is **"Claudeslop"**: her name for the model's verbal tics — the padding, the strings of non-sequiturs. Verbosity, for the first time, has a brand name.

Then the twist. In her How I AI blind benchmark (seven models, multiple tasks, scores revealed live on camera — she didn't know the results in advance), **Opus 5 took first place**, with standout scores on front-end design and prototyping. Her summary is a line built to be quoted: **"My most loathed colleague does the best work."** (Footnote: this is her personal blind-test result; the full leaderboard exists only in her video and has not been independently reproduced.)

Dan Shipper of Every reported the same experience plus one crucial addendum: his team initially found the model "argued with instructions and stopped before work was finished" — but **after deleting every workflow tuned for previous models and rewriting prompts from scratch, it became "dramatically better."** Hold onto that detail; the practical guide needs it.

## Twisting the threads together: five symptoms, one knob

Now put the evidence from both sides in one row:

1. Highest precision ever measured (data)
2. Lower known-bug recall (data)
3. Nitpicks up 4x (data)
4. Timid, apologetic, "hoped it passed" (vibes)
5. Refuses to touch someone else's branch (vibes)

These five facts are five exposures of a single training disposition: **this model has been tuned to prefer not acting over acting wrongly.** Precision is high because it only speaks when confident. Recall drops because when unsure it stays silent — missed bugs are the direct price of "never be wrong." Nitpicks quadruple because uncertainty needs an outlet, and judgments it won't bet on leak out as small complaints. The apologies and self-doubt are the same posture projected into language; the merge-conflict refusal is the same posture projected into action — caution executed all the way to recusing itself over ownership.

So "the data says more precise" and "the vibes say more annoying" don't conflict. They are **the reading and the noise of the same knob.** Once you see that, every apparently contradictory review of the past week snaps into place: winning the blind test (output quality is the knob's front side) and "I hate working with it" (interaction friction is its back side) were always going to be true simultaneously. You've met this person at work.

## The practical guide: pick failure modes, not "higher is better"

Five rules for daily use:

1. **Treat the effort setting as a failure-mode selector.** For final-gate review where false positives are expensive (say, a CI check that blocks PRs), use x-high — it misses more, but what it says is right. For first-pass sweeps where over-flagging is fine (a security pass over an unfamiliar codebase), use default — noisy, but wide. "Two failure modes" is a far more useful frame than "more expensive = better."

2. **Pair it; don't solo it.** Strip the product self-interest from CodeRabbit's advice and it still stands: let a wide-net model generate candidates and let Opus 5 verify and adjudicate. That puts the knob's position exactly where it pays.

3. **Delete the prompts you tuned for the previous generation.** Dan Shipper's finding is the week's most valuable single tip: defensive instructions written for older models' personalities ("don't be lazy," "be proactive") push an already over-cautious model in exactly the wrong direction. Start clean; explicit authorization beats exhortation.

4. **Write ownership into the context.** "This branch is mine; you have permission to modify it" — one sentence dissolves the merge-conflict-style refusal. It's not a capability problem; it's a permission-imagination problem. Nail down the permissions and it moves.

5. **Re-run your cost math.** $5/$25 pricing plus a 1M-token default context plus per-turn adjustable effort makes long-session cost structures quite different from the previous generation. Our [LLM API price calculator](/tools/llm-pricing) lets you plug in your own cache-hit rate and throughput before committing to a daily driver.

## Coda

Last week's benchmark survival checklist included this rule: build 20 private tasks of your own and stop trusting leaderboards. Vo's blind test is that advice in its completed form — she walked in carrying the full weight of "I hate this model," and it still came out first. **Her emotions didn't contaminate the measurement, because the measurement was designed to precede the emotions.**

That may be the most generalizable thing Opus 5's first week leaves behind: a model's "personality" is its interface; a model's output is its deliverable. The two can diverge — and they will diverge more often from here on. People who learn to grade them separately will reach the productivity before people who wait for a model that is both great to use and great to be around.

The obnoxious colleague does the best work. Learn to collaborate first. Like them later.
