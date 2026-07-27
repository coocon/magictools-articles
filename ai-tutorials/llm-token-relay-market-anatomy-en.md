---
title: "When API Tokens Sell at 3% of List Price, You Are Not Buying Tokens"
slug: llm-token-relay-market-anatomy-en
summary: "425 RMB buys $3,333 of official Anthropic credit — a 97.8% discount. That is not a promotion, it is the retail price of a four-layer supply chain. This is an anatomy of the relay market: who sits at each layer, which two open-source projects it runs on, where the money comes from, and the part most product teams miss — if your product exposes model capability, you are already inventory in somebody's account pool."
category: ai-tutorials
tags: [AI security, LLM, API abuse, relay market, model distillation, one-api, fraud, Anthropic]
coverImage: ""
status: published
locale: en
source: authored
translationSlug: llm-token-relay-market-anatomy
---

# When API Tokens Sell at 3% of List Price, You Are Not Buying Tokens

Start with a price.

A relay operator's comparison page lists a package: **425 RMB for roughly $3,333 of official Anthropic credit**. Every dollar you spend buys $7.70 of official usage — **$0.13 per $1 of list price**, a 97.8% discount.

This is not a subsidy or a group buy. Anyone who has ever modeled cloud costs should react to that number immediately: **a 97.8% discount cannot come from negotiating leverage.** No reseller, at any volume, buys wholesale at 2% of retail.

When something sells far below its cost, there is only one explanation: **it is not the product.** It is a giveaway attached to some other business, or bait for one.

This piece takes apart that business — its four layers, the software it runs on, where the money actually comes from, and, more important for most readers, **where you sit in the chain if you build AI products.**

The primary material comes from threat research published by Matt Lenhard on June 28, 2026, whose own source is a Chinese-language industry thread on V2EX that ran from March 5 to June 23, drawing roughly 35,000 views and 190 replies. Live measurements in this article are my own.

## This is not a crew of hackers, it is a supply chain

The vocabulary people reach for — "black market," "stolen keys" — implies something scattered and opportunistic. The actual structure is far more industrialized, with **four cleanly separated layers**:

**Upstream: card and account merchants (卡商 / 号商).** They sell two things — virtual credit cards engineered to pass US and European billing checks, and bulk-registered accounts. This is the raw materials layer.

**Midstream: account pools (账号池).** A pool aggregates dozens to hundreds of upstream accounts, manages their auth tokens and rate limits, fails over when an account gets flagged, and exposes **one** API surface. This is the critical layer — it converts "a pile of dirty accounts that die constantly" into "a service that looks stable."

**Downstream: relays, or transfer stations (中转站).** They wrap the pool's API in a clean Chinese-language product with billing, invoicing and WeChat support groups, then compete on price. This is the layer buyers actually see.

**End users:** individual developers, startups and mid-sized SaaS companies hunting cheap inference — plus commercial buyers running model distillation on the same pipes.

In practice the middle two layers are often the same operators, and the forum uses "pool" and "transfer station" interchangeably. But the four-layer structure tells you something important: **this market is well past its frontier phase.** It has price-comparison sites, affiliate programs, gateway products and support SLAs. The ten highest-traffic relays the researcher tracks pull **3.6 million visits a month combined**.

## It runs on two perfectly legitimate open-source projects

Nearly every relay is built on one of two open-source projects: **one-api** or **new-api**.

Both are OpenAI-compatible gateways. An operator deploys the panel and adds a set of **channels (渠道)**, where each channel is a provider plus a pool of API keys. The panel exposes a single endpoint matching the OpenAI API, so a buyer changes one base URL in an existing SDK and is done. Each request pulls a key from the pool, forwards it upstream, returns the response, and deducts quota priced at usage times a **multiplier (倍率)**. Users, tokens, price tiers, logs and billing all ship in the box.

**To be clear: both projects are entirely legitimate.** Plenty of ordinary companies self-host one-api to put their own several accounts behind a single gateway with team quotas and spend tracking. That is what it was built for.

**The line is crossed at the inventory, not the software.** A gateway becomes a relay when its channels are stocked with stolen, leaked or pooled keys rather than the operator's own accounts, and when it resells that access against provider terms. The tool is neutral. The sourcing is not.

I checked the current state of both projects and found a detail the original research does not mention:

| Project | Stars | Last commit |
|---|---|---|
| `songquanpeng/one-api` | 35,953 | **2026-01-09** |
| `QuantumNous/new-api` | 43,459 | **2026-07-26 (same day)** |

The research notes that across tracked relays, one-api appears roughly four times as often as new-api. Put that observation next to the table and the picture resolves: **one-api holds the installed base, new-api holds the growth.** The one-api main repository has been dormant for seven months while its deployment count coasts on inertia. Meanwhile the fork — whose differentiating features over one-api are precisely self-service payment, account top-up, and image, video and audio models — has passed it in stars and was still shipping commits on the day I checked.

When a fork's headline feature is a self-service checkout, that tells you who it is mainly serving.

## Where the money is actually manufactured

That 97.8% gap is produced by five methods. What they share is that **the cost gets transferred to somebody else.**

**Free-credit abuse.** Automate account creation at scale, claim the free credits, proxy that traffic back to paying end users. The marketing budget a lab spends on acquisition becomes a relay's cost of goods.

**Chargeback attacks.** Reverse the charges after the usage period closes to recoup the spend — or start with stolen cards and skip the pretense.

**Prepaid cards.** Fund accounts with capped prepaid cards, run them to the limit, discard.

**Open inference.** This is the one product teams should stare at: **any support chatbot without strict guardrails can be used to proxy traffic.** You think you shipped a pre-sales assistant. What you actually shipped is a free LLM endpoint that you are paying for.

**Denial of wallet.** Strictly speaking not a relay technique but an emerging abuse pattern: attackers fire floods of concurrent requests **purely to burn a provider's spend**. The other four at least have a profit motive. This one does not — the goal is simply to bankrupt you. Which means the standard "model fraud by attacker profit" approach fails here, because the attacker is not optimizing for profit.

## Who buys, and the three reasons they buy

The forum discussion is unusually candid about motive. Three categories: **cheap tokens, bypassing geographic restrictions, and model distillation.**

The first two are self-explanatory. The third is where the actual profit sits. From the thread, translated:

> "Distillation uses Claude/CodeX models to train domestic models. There are intermediaries that specialize in distillation and can provide relevant evidence, but I can't name specific domestic companies. Anyway, companies with strong programming capabilities are all distilling Claude; it's a multi-billion RMB industry chain, and many big players earn hundreds of thousands a day."

> "It's not just true — many distillers in the industry have made millions."

And on scale: "I got 20TB on my first day online."

String those together and the 97.8% discount finally makes sense: **cheap tokens are not this business's product, they are its customer acquisition channel.** The product is the **content** flowing through the gateway — your prompts, your code context, your multi-turn reasoning traces. To a downstream distillation buyer that is high-quality training data, and crucially it is **pre-labeled**, because the upstream model's response is the label.

So the old line holds here literally: when you are not paying for the product, you are the product. Except here you did pay — just far too little to cover cost, which guarantees the difference is being made up somewhere else.

## The part most people miss: the application layer is inventory too

If you take one thing from this article, take this.

One passage in the research is easy to skim past: the pools' inventory is **not only lab API keys**. Alongside direct OpenAI, Anthropic and Google credentials sit large numbers of accounts harvested from the **application layer** — much of the forum's activity centers on "reverse-engineered" access to tools like Kiro and antigravity, which are **consumer products**, not lab APIs. Apps other people built.

To a pool, it makes no difference whether a token came from a lab or from an app built on one. **Anything that resells or exposes a model is a target.**

The implication is blunt:

> **If your product exposes model capability to users, you are already a token supplier — whether or not you agreed to be one.**

Your free trial allowance is inventory. Your support chatbot is inventory. That `/api/chat` you forgot to rate-limit is inventory. The frictionless "try it without signing up" flow you built for conversion is the highest-grade inventory of all.

Most teams building AI products model cost as "expected users times average calls per user." There is no line item in that model called "the share consumed by being somebody's upstream." And by the structure above, the better your product works, the looser your guardrails, and the easier your signup, the higher your grade in the pool's catalog.

## A detail worth sitting with: the fairness theater

One detail in the research is, to me, the most revealing thing in it.

A relay-comparison site — which publicly positions itself as a relay authenticity checker and price aggregator — **gives away fifty $100 API keys in a daily lottery**. You earn entry credits from a daily check-in, spend 20 credits per ticket, up to three tickets a round. In one round, 1,150 tickets competed for the 50 keys.

The mechanism is the interesting part. The draw is **provably fair**. The random seed is the hash of the latest Bitcoin block, winners are selected with a Partial Fisher-Yates shuffle, and the full entry list is published as a snapshot before the draw — the same cryptographic scheme legitimate crypto-gambling sites use to prove they did not rig the outcome.

Sit with the absurdity: **a market distributing credentials of unclear provenance imported blockchain-grade verifiable fairness so participants would believe the raffle was not fixed.**

But the absurdity is exactly the signal. **Cryptographic trust gets imported to fill a vacuum of institutional trust.** Everyone in this market knows the counterparty may vanish, dilute, or ship fakes, so it has to bootstrap a mechanism that requires no trust at all. Reaching that point means internal fraud got severe enough to need an engineering fix. **Even the scammers are being scammed.**

For what it is worth, my own measurements: the V2EX thread still returns 200 and reads normally. The comparison site returns a 301 redirect. The lottery page returns **403** to my request — it filters by origin. What kind of traffic a publicly operated site chooses to block is not a hard question.

## So what should you actually do

Two roles, because most developers occupy both.

### If you are a buyer

**Do not buy.** This is a risk calculation, not a moral appeal — your real cost far exceeds the 97% you save:

1. **Model substitution.** You pay for Opus and may receive something far cheaper. Every evaluation baseline you build afterward sits on sand, and you will not know.
2. **Unexplainable data flow.** Your prompts and code context traversed a gateway operated by someone you cannot name, and were most likely retained and resold. In any compliance review, that question has no acceptable answer.
3. **The availability is fake.** Batch enforcement against pooled accounts is routine; failover masks only part of it. Your production SLA is effectively a function of how long a set of fraudulently funded accounts survives.
4. **Guilt by traffic pattern.** When upstream providers hunt abuse, what they fingerprint is traffic shape.

**There are legitimate ways to be cheap.** Aggregator gateways like OpenRouter front hundreds of models behind one key at a thin markup. AWS Bedrock and Azure Foundry resell first-party models with compliance certifications and consolidated billing. And the plainest option of all — **move down a model tier**. Output pricing for models like DeepSeek sits two orders of magnitude below the closed frontier. **Dropping one tier usually saves more than a relay's discount, without the tail risk.**

(One related correction to a common instinct: if the goal is saving money, compute **cost per task**, not price per token. I worked through that in [the previous piece on Kimi K3](/en/articles/kimi-k3-open-weights-four-numbers-en) — halving unit price while doubling token consumption saves nothing.)

### If you build AI products

You are upstream inventory. Design as though you will be hit. In the order abuse travels:

**1. Raise the cost of entry.** Make bulk account creation hard and cap what a fresh account can spend. Check for browser-automation signals. Any client-side detection can be bypassed, but every layer of friction raises the attacker's unit cost — and this market is exquisitely cost-sensitive, since cost arbitrage is the entire business model.

**2. Watch the money.** Prepaid cards, virtual cards, mismatched billing details, small card-testing charges. These signals have high precision.

**3. Watch the behavior.** Look for patterns no real user produces: time from registration to first token, model selected, prompt relevance to your product, account age, IP signals (proxy, VPN, country). **Prompt relevance is especially effective** — a request that came to borrow your compute usually has nothing to do with your business domain.

**4. Cluster the accounts.** Hunt IP sybils and shared device fingerprints to collapse nominally separate accounts back onto one operator.

**5. Monitor and alert on AI spend.** This is a failsafe, not an optimization. A denial-of-wallet run can burn a quarter's budget in hours; what you need is the ability to detect and cut it in minutes.

**6. Assume some gets through, and bound the damage.** Enforce spend caps, spend locks and concurrency limits per account. **Reserve budget for every in-flight request**, or concurrent calls will sail past your limit. Start new accounts low and let them earn higher limits through age and a verified card. When risk rises mid-session, insert a CAPTCHA or a second identity check.

**7. When you catch someone, throttle quietly.** A clean error message tells the attacker exactly which signal to fix before returning.

## The one-line version

Anthropic tightened access rules in September 2025, required photo identity verification for some users in April 2026, and added government ID and biometrics in a July 8, 2026 policy. Plenty of developers have complained about the friction.

**That friction was not a product manager's idea. This supply chain forced it.** The signup cost legitimate developers pay is this gray market's externality, and the bill gets split across everyone.

I also agree with the researcher's forecast: as KYC tightens at the lab layer, **abuse will not disappear, it will relocate**. To where? To the layer with the weakest defenses and the least awareness that it is a target — **the application layer, which is where you and I work.**

So the real conclusion is not "don't buy cheap tokens." It is:

> **In an AI product's cost model, "the portion consumed by abuse" is a line item you must write down explicitly. It is not an accident.**

Now go check whether your `/api/chat` has a rate limit.

---

*Primary material: "An Inside Look at the Relay Market Powering Token Resellers and Fraud" by Matt Lenhard (Vectoral), published June 28, 2026, whose own source is V2EX thread t/1196011 (March 5 to June 23, 2026). The price conversion originates in reply #50 of that thread. GitHub project statistics and site reachability were measured by the author on July 26, 2026. This is analytical reporting. It provides no access instructions for any relay service and does not recommend using one.*
