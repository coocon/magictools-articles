# One Prompt, 11 Models, a 200x Spread

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/one-prompt-11-models-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/one-prompt-11-models-en?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 Continue reading: One Prompt, 11 Models, a 200x Spread](https://tools.cooconsbit.com/en/articles/one-prompt-11-models-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
