---
title: "One Prompt, 11 Models, a 200x Spread"
slug: one-prompt-11-models-en
summary: "Netlify ran the same coffee-shop prompt through 11 frontier models using its open-sourced AXIS framework. Credit spend ranged from 2.4 to 519 — and the price tag turned out to be the least interesting part."
category: ai-tutorials
tags: [model evaluation, AI coding, Netlify, OpenRouter, Claude, GPT, Gemini, DeepSeek, Kimi, GLM]
coverImage: ""
status: published
locale: en
source: authored
translationSlug: one-prompt-11-models
---

# One Prompt, 11 Models, a 200x Spread

> Source material: Netlify's blog post "More models, more choice: Comparing 11 different AI models" (2026-08-13). All block quotes are verbatim from the original.

---

Most model-selection advice is downstream of three things: leaderboards, launch-day slide decks, and someone on X saying "I tried it, it's insane."

Netlify just did something more useful. Fresh off a partnership with OpenRouter — which brought Kimi K3, GLM 5.2 and DeepSeek V4 into Agent Runners alongside the closed frontier models — they took **one prompt**, ran it through 11 models, three times each, using AXIS, the evaluation framework they recently open-sourced. Then they published every generated site and every credit total.

The spread: 519 credits at the top, 2.4 at the bottom. **A factor of 200.**

The number is the headline. The interesting part is everything underneath it — why the cheapest run looked surprisingly credible, why the most expensive model blew 1,055 credits on a single run, and why two models from the same vendor felt like they came from different decades.

Ten things worth taking away.

---

## 1. This Is a Receipt, Not a Leaderboard

> These checks are very much focused on _correct functionality of the generated site rather than its design_, e.g.: does it use a database when a user's needs call for it? Does it properly use Netlify Database in that case?

Netlify's internal AXIS runs are unglamorous and practical: did the model reach for a database when the brief called for one, did it wire it up correctly, and — just as important — did it *avoid* over-engineering when a static page would do. Models that score badly don't get offered in Agent Runners at all.

This post is a different exercise, and the author says so up front:

> Of course, this is going to be a much more subjective test than our internal test suites, but it's also going to be a very fun one.

It doesn't rank models. It shows you what a given amount of money buys.

**My take:** A leaderboard answers "which is best." A receipt answers "which should I put on the card." Only one of those is a decision I actually make.

---

## 2. The Real Test Is the Last Sentence of the Prompt

> Build a one-page site for a neighbourhood coffee shop: opening hours, the address, a short menu and a photo. Nothing on it changes unless I edit it myself.

The first half is the brief. The second half is the trap:

> The last sentence was added as a hint to the model that no fancy Content Management System is needed.

"Nothing on it changes unless I edit it myself" is a hint, not an instruction. It says: no CMS, no database, no admin panel. A model that misses it will happily architect you a content backend for a page with four facts on it.

**My take:** Capability isn't the differentiator anymore — restraint is. Over-building is a comprehension failure, exactly like under-building. It's just a more flattering one.

---

## 3. The Table Is the Argument

| Model | Average credits | Three runs |
|---|---|---|
| Claude Opus 5 | 519 | 253 · 249 · 1,055 |
| Claude Sonnet 5 | 143 | 81 · 245 · 103 |
| GPT 5.6 Sol (low effort by default) | 141 | 173 · 158 · 92 |
| Gemini 3.6 Flash | 103 | 109 · 91 · 111 |
| Kimi K3 | 102 | 125 · 95 · 86 |
| Gemini 3.1 Pro | 53 | 57 · 52 · 49 |
| GPT 5.6 Terra | 39 | 43 · 23 · 49 |
| DeepSeek V4 Pro | 37 | 47 · 30 · 33 |
| GLM 5.2 | 27 | 15 · 42 · 24 |
| Kimi K2.7 Code | 19 | 21 · 18 · 17 |
| DeepSeek V4 Flash (0731) | 2.4 | 3.4 · 1.3 · 2.5 |

> That's a pretty wide distribution, eh? Not only that: the Claude Opus average is heavily slanted upwards because one of its three runs spent a whopping 1,055 credits!

Netlify's plans put that in perspective: 300 credits on free, 1,000 on Personal, 3,000 on Pro, with top-ups at $10 per 1,500.

So one Opus run burned **three and a half times a free plan's entire monthly allowance**, or a third of a Pro plan. The same budget buys you roughly 400 runs of DeepSeek V4 Flash.

**My take:** Stop asking which model is strongest. Ask what your monthly credit ceiling is, then ask whether this particular task deserves a third of it.

---

## 4. Opus Overspending Isn't a Fluke — It's a Temperament

> But in all the tests I've done, Opus does have a tendency to run off with excessive credit usage (compared to its "typical" baseline) more than other models. It does not guarantee a worse or better outcome, though. It's something that just _happens_ pretty frequently.

Read that carefully. The claim isn't "Opus occasionally spikes." It's that Opus spikes *more than other models*, *fairly often*, and the 4x spend buys you no guarantee at all.

The 1,055-credit run genuinely was lovely — a stamp-like element with a coffee bean at the center, built as animatable text rather than an image; a custom map at the bottom; dark mode working out of the box. But the other two Opus runs cost 253 and 249, and they held up fine.

> As to whether the first result is truly "4x better" or not, opinions might vary.

**My take:** This is a variance problem, not a quality problem. You can't budget around a model whose spend has a long tail. Use Opus with a hand on the wheel, or accept that some runs are a raffle.

---

## 5. The Mid-Tier Collapse: A Throttled Flagship Beat a Mid-Tier One

At effectively identical cost — 143 vs 141 credits — the author's verdict is blunt:

> I think OpenAI's top-tier model in low effort mode wins over Anthropic's mid-tier model when it comes to basic design intuition, at least in this scenario.

Sonnet 5's problem wasn't ugliness. It was thinness:

> There's still some delightful detail in each of these, just _less so_ (and less content in general). The vector graphics is noticeably simpler and not really something you'd consider for a live site.

GPT 5.6 Sol, deliberately run at low effort, still produced richer content and none of the awkward vector shapes — at the cost of somewhat generic imagery.

**My take:** A flagship thinking less can beat a mid-tier model thinking hard. If you're cutting cost, try turning down effort before you turn down the model. At least for work like this.

---

## 6. Terra Isn't Sol-Lite — It's a Different Taste

Dropping from Sol to Terra (141 → 39 credits; OpenAI's ladder is Sol→Terra→Luna) did *not* reproduce the cliff we just saw going from Opus to Sonnet:

> here it seems like Terra has a _different visual language_, and not a necessarily worse one.

Terra's output was simpler in content and had visible rough edges — a missing image in one run, low-contrast text over a photo in another — but nothing structurally broken. It reads as a different aesthetic, not a degraded one.

Which leads to the single most actionable line in the post:

> Up to this point, if I had a very vague idea of what design & language I'd like for a project, my personal inclination would be to run the same prompt with Opus 5 and GPT 5.6 Terra, and get two very different but worthwhile takes.

Opus 5 plus Terra is 519 + 39 ≈ 558 credits — cheaper than running Opus twice, and instead of two near-duplicates you get two genuinely different opinions.

**My take:** When the brief is vague, don't ask one model for the right answer. Ask two models with different taste for their answer, and arbitrate. Diversity of output beats depth of output at the ideation stage.

---

## 7. Generation Beats Tier, and It Isn't Close

Gemini 3.1 Pro spent 53 credits, and the author didn't even bother linking the results:

> Here is what Gemini 3.1 Pro generated for 53 credits on average. I'm not even putting the links to the live site here, because there's really nothing to see.

The three runs looked nearly identical, and:

> Yes, these are wholly separate runs. It did what we asked in the prompt, and really nothing more.

Gemini 3.6 Flash — a *cheaper tier*, a newer generation — spent 103 credits and, in the author's words, `seems like a whole new generation`, working noticeably harder on content. Its one weakness is repetition:

> All models repeat themselves, but it seems like Gemini might repeat itself even more.

**My take:** "Pro beats Flash" is a stale heuristic. The newer cheap tier out-produced the older expensive tier, and cost *more* doing it — which is exactly what doing more work looks like. Check the release date before you check the tier name.

---

## 8. Open Weights: Real Positions, Wrong Exam

Kimi K3 came in at 102 credits and didn't stand out. The author volunteers the caveat himself:

> To be clear, Kimi K3 is marketed mostly as a frontier model for long-horizon agentic tasks, and various benchmarks and reviews confirm its prowess in that field. It was built to take on Fable 5 more than Opus 5. But in this narrow design-led task, it does not particularly shine among others. **To really do this model justice, we'd need a wholly different set of prompts engineered for a complex web app,**

Kimi K2.7 Code came in at just 19 credits, and the verdict is short:

> Despite some hype about Kimi's visual capabilities from around the K2.6 model launch, in terms of design or content there's really not much to see here.

GLM 5.2 is the most interesting entry. It averaged 27 credits (15, 42, 24), but the variance wasn't in the price — it was in the output:

> Surprisingly, these runs are - maple aside - very different, as if coming from a few different models. For the relatively low credit cost of GLM, it's probably worthwhile to run it a few times before settling on what this model can do for you.

(GLM also has an unexplained fondness for maple leaves.)

And one hard capability wall:

> Note that being a text-only model that does not receive image inputs, GLM in its current 5.2 iteration cannot do something that Kimi models can: get screenshots from the user for inspiration, as in "this is the kind of design I'm looking for".

**My take:** With cheap models, the right move is to roll several times and pick — at 20-odd credits a run, sampling is basically free. But text-only is not a quality gap you can sample your way out of. If the workflow involves "make it look like this screenshot," that model was never a candidate.

---

## 9. One Broken Image, and the Gap It Exposes

DeepSeek V4 Pro averaged 37 credits — nearly identical to GPT 5.6 Terra — and lost the comparison. The damning detail wasn't aesthetic:

> The middle run also has a broken image: the HTML file points to an image file that does not actually exist in the project, which is a lot less likely to occur nowadays with any of the commercial models from OpenAI, Anthropic, or Google.

The HTML referenced a file that wasn't in the project. That's not taste — that's a missing self-check. The model never went back to confirm its own references resolved.

Then V4 Flash 0731 set the floor at 2.4 credits and delivered a genuine surprise:

> Interestingly, the middle one doesn't just look the most like what a mid-tier closed model might give you, but also feels the same in terms of language, and has actually consumed the least credits among all runs.

That run cost 1.3 credits. **The cheapest single run in the entire test produced the most closed-model-looking result.**

**My take:** The risk with dirt-cheap models isn't that the output looks bad — you'd notice that. It's silent omission. Nothing tells you what didn't get done. Which means cheap models are affordable only in proportion to your ability to inspect the output.

---

## 10. Past a Static Page, the Whole Rubric Changes

> First, for anything beyond a simple website or the initial ideation phase for a project, the question shifts from how nice the model design & copy is to:
>
> *   Does it know which platform features to use, when and how, to get the functionality you want? Can it store user data, use AI in your web app, and handle authentication and security?
> *   Does it rigorously validate its own work? Can it validate the frontend aspect of your project (that's where image inputs become crucial)? Can it reliably find and fix issues based on feedback from you, and tell you when your own input is misleading or you've overlooked an important concern?

This is the real point of the post. A coffee shop page can only be judged on looks and copy. A real project is judged on three things: does it know the platform, does it check its own work, and can it reliably fix what's broken.

Note the last clause especially — can it tell you *your* request is the problem. That's a bar most models are nowhere near.

The final read on Opus lands here too:

> Currently, Opus will probably provide the most clever word games and sleekest design, but you don't necessarily need it to. Of course, Opus will also perform relentless self-validation of its own work (it does not bill itself on good looks alone). But remember there's certainly a higher-than-average credit cost attached to that.

Part of that 519-credit average isn't buying polish. It's buying the relentless self-validation. Whether you need it depends entirely on the task.

And on budget, the author refuses to hand you an answer:

> Given a limited budget, would you prefer a _turnkey solution_ that attempts to pre-plan and handle everything for you, or should you go with a simpler model and a more iterative approach, where you guide the model with follow-up prompts towards what you want? No option here is necessarily wrong.

**My take:** Turnkey is expensive because you don't have to steer. Iterative is cheap because you do. The deciding variable isn't the model — it's whether you can tell when the output is wrong. **The better you are at reviewing, the cheaper a model you can afford.**

---

## Closing Thought

The report is honest about its own limits, repeatedly. This is one **narrow, design-led task** — a static page for a coffee shop. Kimi K3 was handed an exam it wasn't built for, and the author says so unprompted. The whole thing is more subjective than Netlify's internal suite, and he says that too.

It's still worth more than a stack of leaderboards, because it pins two normally separate questions to the same table: **what you spent, and what you got.**

If you take one thing: **there is no best model, only one that fits your budget and your ability to review its work.** Use two different-tasting models when the brief is vague. Roll a cheap one several times when the task is narrow and well-specified. Save the 519-credit question for when databases, auth and AI gateways enter the picture — which is exactly where Netlify says the follow-up posts are headed.

---

*Original: More models, more choice: Comparing 11 different AI models (Netlify Blog, 2026-08-13)*
*Link: https://www.netlify.com/blog/one-prompt-11-models-very-different-results/*
*Full test results: https://the-coffee-shop-brief.netlify.app/*
