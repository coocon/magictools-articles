---
title: "OpenAI Didn't Cut the Price: GPT-5.6 Sol's 50% Off Expires September 18 and Excludes BYOK"
slug: gpt-5-6-sol-openrouter-half-price-expiry-en
summary: "GPT-5.6 Sol dropped from $5/$30 to $2.50/$15 on OpenRouter. The 50% is real, but three qualifiers went missing in transit: this is a platform-side promotion from OpenRouter and Vercel AI Gateway, with OpenAI's own pricing page still showing $5/$30; it expires September 18; and BYOK requests don't get it. SemiAnalysis raised a sharper question — these two platforms are a negligible share of OpenAI's volume but happen to be the main public data source for estimating model market share. Here's a same-source price table for 17 models, the 272K long-context pricing cliff, and a checklist for deciding whether to switch."
category: ai-tutorials
tags: [GPT-5.6, OpenRouter, API costs, model selection, OpenAI, LLM pricing, Claude, Kimi K3]
coverImage: ""
status: published
locale: en
source: authored
translationSlug: gpt-5-6-sol-openrouter-half-price-expiry
---

# OpenAI Didn't Cut the Price: GPT-5.6 Sol's 50% Off Expires September 18 and Excludes BYOK

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

For contrast, the July 30 cut *was* an official OpenAI price change: Luna down 80% ($1/$6 → $0.20/$1.20), Terra down 20% ($2.50/$15 → $2/$12). **Sol didn't move at all that time.** These are categorically different events: one is a vendor repricing, the other is a channel running a promotion.

While we're correcting things: **there is no "GPT-5.6 standard."** This generation's lineup is three tiers — Sol / Terra / Luna. In OpenAI's framing, the number denotes the generation while Sol/Terra/Luna are persistent capability tiers that evolve independently. Sol is the flagship, Terra the balanced tier, Luna the cheapest and fastest. Latin for sun, earth, moon.

## 3. SemiAnalysis's Question: This Promotion May Not Be For You

This deserves its own section.

SemiAnalysis argued on X that OpenRouter and Vercel are a negligible share of OpenAI's total volume — **but they happen to be the main public data source the industry uses to estimate model market share**, because OpenAI publishes no such numbers itself.

Halving prices on those two platforms costs almost nothing while potentially doubling Sol's visible volume on exactly the scoreboards people use to judge how OpenAI is faring against Anthropic.

The claim is unverifiable — neither OpenAI nor either platform has explained the motivation — and I'm flagging it as third-party analysis. But the reading it suggests is actionable: **discount any "Sol's share is surging" chart you see over the next few weeks.** The same curve will be far more informative after the promotion ends.

The judgment we made writing about [Stripe's $7B acquisition of OpenRouter](/articles/stripe-buys-openrouter-en) picks up a concrete footnote here: the routing layer's value isn't only in forwarding requests, it's that **it's one of the few public scoreboards this industry has.** Whoever runs a promotion on the scoreboard is influencing the score.

## 4. Same-Source Price Comparison

All from one OpenRouter API snapshot (2026-08-19 00:50 UTC), in $/1M tokens. Same source throughout, to avoid differences in how each vendor presents pricing:

| Model | Input | Output | Context |
|-------|-------|--------|---------|
| **GPT-5.6 Sol (promo)** | **2.50** | **15.00** | 1.05M |
| GPT-5.6 Sol (after 9-18) | 5.00 | 30.00 | 1.05M |
| GPT-5.6 Terra | 2.00 | 12.00 | 1.05M |
| GPT-5.6 Luna | 0.20 | 1.20 | 1.05M |
| Claude Opus 5 | 5.00 | 25.00 | 1M |
| Claude Sonnet 5 | 2.00 | 10.00 | 1M |
| Claude Haiku 4.5 | 1.00 | 5.00 | 200K |
| Gemini 3.1 Pro Preview | 2.00 | 12.00 | 1.05M |
| Gemini 3.7 Flash | 0.375 | 1.875 | 1.05M |
| DeepSeek V4 Pro | 0.66 | 1.98 | 1.05M |
| DeepSeek V4 Flash | 0.083 | 0.165 | 1.05M |
| Qwen3.5 Plus | 0.30 | 1.80 | 1M |
| Qwen3 Max | 0.78 | 3.90 | 262K |
| Kimi K3 | 3.00 | 15.00 | 1.05M |
| GLM-5.3 | 1.40 | 4.40 | 1.05M |
| Grok 4.6 | 2.00 | 6.00 | 500K |
| MiniMax M3 | 0.30 | 1.20 | 1.05M |

Three positions worth noticing:

**The promo puts Sol right next to Kimi K3.** $2.50/$15 against $3/$15 — identical output pricing. Someone on HN already caught the coincidence. Given that the dominant narrative in that 614-point thread was "Chinese open models are forcing American labs to cut prices," this landing spot is hard to read as accidental.

**It's still 50% more expensive than Claude Sonnet 5 on output.** $15 against $10. A flagship at half price still costs half again as much as a competitor's mid-tier on output — a fact the phrase "50% off" tends to obscure.

**After September 18, it sits above Opus 5.** $5/$30 against $5/$25. The day the promo ends, Sol becomes the most expensive output on this table.

## 5. Quality Evidence (Mind the Sources)

OpenAI's launch page cites Artificial Analysis (**these are official citations; I could not independently verify AA's raw charts, whose values live inside JS**):

- **Intelligence Index v4.1**: Sol 58.9 / Claude Fable 5 59.9 / Opus 4.8 55.7 / GPT-5.5 54.8 / Terra 55 / Luna 51.2 / Gemini 3.1 Pro Preview 46.5
- **Coding Agent Index v1.1**: Sol 80 (above Fable 5's 77.2), using under half the output tokens
- Agents' Last Exam: Sol 53.6 — note that OpenAI's own page disagrees with itself here (53.6 in prose, 52.7% in the table), so cite carefully

In short: Sol scores about a point below Fable 5, while OpenAI claims it takes 61% less time at roughly half the cost. On the coding-agent slice specifically, it claims SOTA.

On speed, OpenRouter's platform-side monitoring (captured 2026-08-19 — this part is third-party measured):

| Provider | P50 first response | Throughput | Price |
|----------|-------------------|------------|-------|
| OpenAI | 4.31s | 41 tps | $2.50/$15 (promo) |
| Azure | 3.46s | 42 tps | $5.00/$30 |
| **Bedrock** | **2.58s** | **100 tps** | $5.50/$33 |

Bedrock is 2.4× faster and 2.2× more expensive. **If your workload is interactive — a user is waiting — run that tradeoff explicitly.** Not every scenario should default to the cheapest route.

## 6. Should You Switch: A Checklist

**If you do switch, get these right:**

1. **Pin the provider.** To get the discount, requests must route to the OpenAI provider without BYOK. The default Balanced routing usually lands on the discounted tier, but the safe move is to specify `provider: {order: ["openai"]}` explicitly. This one model has 3 providers and 7 endpoints on OpenRouter, priced anywhere from $2.50 to $5.50.
2. **Don't enable BYOK.** BYOK requests bill at official list price, which means forfeiting the discount outright. (BYOK has its own free allowance — $25,000/month at list, 5% beyond that — but that's a separate ledger; don't mix them.)
3. **Budget the 5.5%.** OpenRouter adds no inference markup, but charges 5.5% on credit purchases ($0.80 minimum), or 5% for crypto. During the promo, going through OpenRouter nets roughly 47% savings versus direct OpenAI — not 50%.
4. **Put September 18 in your calendar.** After expiry the two are priced identically and OpenRouter's remaining value is multi-provider failover and a unified interface. **Don't write the promo price into any long-term cost model.**
5. **Check your prompt length distribution.** Any request over 272K tokens gets no discount.

**You'll probably use more after a price cut, and that isn't automatically good.** TD Cowen measured the aftermath of the July 30 official cut: Luna dropped 80% in price, volume rose roughly 14×, and revenue actually *increased* about 34%; Terra rose ~5× in volume with ~45% more revenue. Textbook Jevons paradox. The data window is only about two weeks, so don't treat it as settled — but the direction is clear. **Cheap induces new usage; your bill will not necessarily halve just because the unit price did.**

**Community read** (from that 614-point HN thread, for what it's worth): Sol is widely considered strong at coding and agentic work with fewer refusals — several users migrated after Claude's safety filters caught them in false positives. Others think Fable 5 still edges it on long tasks and that Sol "gets stuck fixating on details." As of capture, **no reports of rate limiting or quality degradation following the discount.**

## 7. What I Take From This

**First, treat "who cut the price" and "for how long" as required fields on any pricing story.** Miss any of this one's three qualifiers — platform-side, expiring, BYOK-excluded — and your cost model is wrong. Channel promotions and vendor price cuts are different animals: the former can end at any time, the latter usually means the cost structure genuinely changed. July 30's Luna cut was the latter. This is the former.

**Second, the thing to optimize isn't which model you use — it's how cheap it is to change your mind.** Is a 30-day promo window worth a code change? If model selection is hardcoded into your business logic, basically no — you'll just change it back on September 18. If it's one line of config, the promo is free money. We ran the numbers on selection in [One prompt, 11 models, a 200× spread](/articles/one-prompt-11-models-en), and the conclusion holds: **in a market where prices move monthly, the cost of switching models is more worth optimizing than the price of any model.**

**Third, run a real bill before you trust a marketing number.** If you have batch work, move some traffic over during the promo and measure the savings against your own token distribution and reasoning tier. Benchmark scores and "roughly half the cost" claims are built on somebody else's workload. How long your prompts are, how long your outputs are, and whether you turn reasoning up affect your bill far more than leaderboard position does.

**Fourth, don't read "same price as Kimi K3" as coincidence.** The most informative thing on that table isn't any single row — it's how fast the distance between rows is collapsing. An open-weight flagship has anchored closed-weight flagship pricing to the $15-output line, and that line didn't exist three months ago. The promotion will end. That pressure won't.

---

**References**

- [OpenRouter: GPT-5.6 Sol model page](https://openrouter.ai/openai/gpt-5.6-sol)
- [OpenRouter Models API (price data source for this piece)](https://openrouter.ai/api/v1/models)
- [OpenAI launch page ($5/$30 pricing, naming, benchmarks)](https://openai.com/index/gpt-5-6/)
- [OpenRouter announcement tweet (2026-08-17)](https://x.com/OpenRouter/status/2089406144297214339)
- [OpenRouter fees FAQ (5.5% credit fee, BYOK rules)](https://openrouter.ai/docs/faq)
- [BigGo Finance analysis (promo window, SemiAnalysis, TD Cowen data)](https://finance.biggo.com/news/c4170767-79dc-4f18-862a-95107ffe6fd5)
- [HN discussion (614 points)](https://news.ycombinator.com/item?id=49337602)
- Related on this site: [Stripe's $7B OpenRouter acquisition: buying the routing layer](/articles/stripe-buys-openrouter-en) · [One prompt, 11 models, a 200× spread](/articles/one-prompt-11-models-en) · [When API tokens sell for 3% of list price](/articles/llm-token-relay-market-anatomy-en)
