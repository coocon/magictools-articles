# When API Tokens Sell at 3% of List Price, You Are Not Buying Tokens

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/llm-token-relay-market-anatomy-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/llm-token-relay-market-anatomy-en?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 Continue reading: When API Tokens Sell at 3% of List Price, You Are Not Buying Tokens](https://tools.cooconsbit.com/en/articles/llm-token-relay-market-anatomy-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
