---
title: "Three Clocks: Why Kimi K3's Weights Still Are Not Out"
slug: kimi-k3-weights-three-clocks-en
summary: "The widely-reported release moment has passed and moonshotai's newest Hugging Face repo is still last month's K2.7-Code. Includes my four API measurements across seven hours — but the point is not that it is late. The release is caught between three clocks: Moonshot's engineering schedule, Beijing's draft export controls on weights (already-downloaded weights are unaffected), and Washington's distillation accusation. An open-weight release date is no longer set by training progress."
category: ai-tutorials
tags: [Kimi K3, open weights, Moonshot AI, export controls, model distillation, AI regulation]
coverImage: ""
status: published
locale: en
source: authored
translationSlug: kimi-k3-weights-three-clocks
---

# Three Clocks: Why Kimi K3's Weights Still Are Not Out

Facts first, interpretation second.

Over the past seven-plus hours I queried the Hugging Face API four times. The record, all times UTC:

| Time | Newest repo under `author=moonshotai` | Official K3 repo |
|---|---|---|
| 07-26 17:45 | `Kimi-K2.7-Code` (2026-06-15) | Does not exist |
| 07-26 19:37 | Same | Does not exist |
| 07-26 21:37 | Same | Does not exist |
| **07-27 01:08** | Same | **Still does not exist** |

The release moment reported almost everywhere was **00:00 UTC on July 27, 2026**. As I write this, that moment passed 68 minutes ago.

## First, calibrate the word "late"

Before writing "Moonshot missed its deadline," it is worth checking the wording — because this is precisely the class of error I criticized in an earlier piece: **a number lifted out of its context and treated as a conclusion.**

Moonshot's own technical blog says the full weights will be released **by July 27, 2026**.

In secondary coverage that became "shipping at 00:00 UTC on July 27."

Those are not the same sentence. By Moonshot's own wording, the deadline is the **end of July 27** — 23:59 UTC. So at the moment I write this they have nearly 23 hours left and **have not missed anything**.

This is therefore not a gotcha piece. What is actually worth writing is something else: **inside this waiting window you can see more than the release itself would show you.**

Because the timing of this release has stopped being an engineering question.

## Clock one: Moonshot's own engineering schedule

First, rule out "something happened to the company."

I checked the MoonshotAI organization on GitHub: the `kimi-code` repository was last pushed at **2026-07-26 16:38 UTC** — an hour before my first query — at 5,192 stars. The org is not dark. People are working.

The delivery difficulty is also real. This is roughly a **1.4TB** weight release (2.8 trillion parameters stored natively at MXFP4 4-bit), alongside a promised technical report. And the KDA prefill-cache implementation Moonshot contributed to vLLM is expected to land with the weights — KDA's hybrid attention is incompatible with conventional prefix caching, so shipping weights without the matching inference path means shipping a file most people cannot run.

All legitimate reasons for delay. None of them explain the next two clocks.

## Clock two: Beijing — draft export controls on weights

On July 21, the *Financial Times* reported that China's Ministry of Commerce has been consulting **Alibaba, ByteDance, Zhipu** and other domestic AI and chip companies on possible controls covering **model weights, key training data, and chip designs**. Reuters followed the same day but said it could not immediately verify the report; MOFCOM and the named companies did not respond to requests for comment.

This gets summarized as "China is going to ban open-source models." That summary is wrong, and it is wrong in the place that matters most.

**The real distinction is possession versus access.**

Per the reporting, two things sit at the center: the transfer of training data abroad, and **whether foreign users should continue to be able to freely download the weights of Chinese AI systems**. Access via APIs and cloud offerings **would remain**. Overseas customers could still use Chinese AI services and models remotely, and Chinese companies could still monetize foreign demand.

In other words: **what would be restricted is not usage. It is possession.**

What that means for a developer: if rules land in this shape, you can still call Kimi's API, but you **cannot download the weights to self-host, cannot fine-tune, and cannot redistribute**.

Familiar? In [my analysis of K3 three days ago](/en/articles/kimi-k3-open-weights-four-numbers-en) I argued that between the 1.5TB+ HBM requirement and Moonshot's 64-accelerator serving guidance, "open weights" and "usable by an individual developer" had **already decoupled in practice**, and that for most people K3 going open meant "more vendors will serve a K3 API at lower prices."

The rules now being drafted would take that **de facto split and write it into law**.

And there is one detail that turns this from a regulatory story into a countdown:

> **Weights already downloaded are unaffected. Any controls would apply prospectively.**

That single sentence makes publication a **one-way door**.

Once weights are public they cannot be recalled — the files are on thousands of machines regardless of what rules follow. So for a lab that has publicly committed to releasing, every day before rules land is a closing window. **Release earlier and it is more irreversible; wait longer and the rules may catch up.**

Which is also why "an hour late" is worth watching at all. In an ordinary engineering context an hour is nothing. In a context with a possible regulatory window it is a signal — though to be explicit, **there is no evidence connecting this delay to regulation**. MOFCOM has made no public statement and announced no timetable.

One counterintuitive detail from the coverage: **industry pushback has been significant**. Several companies told regulators that tighter rules would slow their own development and weaken China's chances in the global technology race. Open weights are exactly what made Qwen, DeepSeek and Kimi the default choice for developers worldwide — closing that path closes their own distribution channel.

## Clock three: Washington — distillation accusation and sanctions threat

On July 22, **Michael Kratsios** — Assistant to the President and Director of the White House Office of Science and Technology Policy — posted on X accusing Moonshot AI of **distilling Anthropic's recently released Fable model** to develop K3, alleging Moonshot "developed a sophisticated internal platform to conduct large scale distillation against U.S. models, allowing them to quickly switch between multiple methods of access to avoid detection."

He further alleged Moonshot had acquired Nvidia GB300-equipped servers and accessed GB300s **in Thailand**, likely for training — GB300s are Blackwell generation, banned from sale to Chinese companies.

Treasury Secretary **Scott Bessent** escalated, posting: "**Open source is not open season on American IP**," and adding that when Chinese firms conduct covert, industrial-scale distillation that crosses into IP theft, sanctions and Entity List designations are on the table.

### But there is a 15-day evidence gap

The accusation deserves to be taken seriously, and it deserves to be doubted. The grounds for doubt are a **specific, checkable timeline**:

- **Fable 5 returns to public availability**: July 1
- **Moonshot launches K3**: July 16

Fifteen days.

Running large-scale distillation against a model that has been public for fifteen days, then training a 2.8-trillion-parameter model and completing an evaluated launch, is difficult to make work as engineering. Experts have disputed on those grounds that K3 was primarily developed through Fable distillation.

More importantly: **neither the White House nor Anthropic has published technical evidence.** No detected prompts, no account infrastructure, no model fingerprints, no watermarks, no training records. As of July 24 there was also **no enforcement action** — every consequence in circulation is floated, threatened, or under investigation.

In fairness to the other side: Anthropic **has** documented an earlier large-scale Moonshot distillation campaign involving millions of Claude exchanges. So the base rate is not zero. What lacks evidence is **this specific claim** (Fable 5 → K3), not the question of whether distillation has ever happened.

### A piece of corroboration I reported myself — and what it does not prove

One phrase in Kratsios's accusation is worth pausing on:

> "…allowing them to **quickly switch between multiple methods of access to avoid detection**."

That is, word for word, the functional definition of an **account pool (账号池)**.

[My previous piece](/en/articles/llm-token-relay-market-anatomy-en) took apart that supply chain's four layers: card and account merchants supplying virtual cards engineered to pass US and European billing checks plus bulk-registered accounts → account pools aggregating dozens to hundreds of upstream accounts, managing auth and rate limits and **failing over when an account gets flagged** → relays wrapping it in a consumer product and competing on price → buyers. The primary source there was a V2EX industry thread running over three months with 35,000 views, in which an operator wrote: "companies with strong programming capabilities are all distilling Claude; it's a multi-billion RMB industry chain."

**The evidentiary status of these two things is not the same, and the distinction has to be explicit:**

- "Such a supply chain exists and is commoditized" — primary-sourced, checkable.
- "Moonshot used it to build K3" — a **political accusation** with no published technical evidence.

The first does not imply the second. I do not think it can.

But the first does explain two things: **why this class of accusation sounds credible** (the infrastructure it describes genuinely exists and is available off the shelf), and **why it is nearly impossible to falsify** (that infrastructure's entire architecture is built to evade detection — failover, account pooling, multi-path access are all designed to scatter the attribution trail).

That is a doubly unfavorable position: the accuser cannot produce clean evidence, and the accused cannot clear their name. **When the object of forensics is a system engineered to be untraceable, "absence of evidence" favors neither party.**

## A second observation from the waiting window: the squatter repo is gaining likes

One small detail I logged across the four queries.

A full-text search for `Kimi-K3` on Hugging Face returns no official repository and three third-party repos holding the name. One of them, `audnai/penclaw-Kimi-K3.0-abliterated-GGUF` (created July 18), moved like this:

| Time | Likes | Downloads |
|---|---|---|
| 07-26 17:45 | 80 | **0** |
| 07-27 01:08 | **82** | **0** |

An "abliterated GGUF quantization of K3," gaining bookmarks while the official weights **do not yet exist**, with downloads pinned at zero throughout.

The combination is informative: **rising likes mean the demand is real; zero downloads mean there is nothing usable inside.** It is a thermometer for a demand vacuum — and live evidence for the "download only from `huggingface.co/moonshotai`" item on my earlier checklist. The moment the real weights land is when repos like this become most dangerous: that is when "K3 GGUF" listings proliferate and the window for telling real from fake is narrowest.

## What to verify when the weights do land

I published a pre-release checklist in [the first K3 analysis](/en/articles/kimi-k3-open-weights-four-numbers-en). Since the release has not happened, it still stands — and can now be sharpened:

1. **Whether the arithmetic contradiction resolves.** Reported download size is roughly 594GB, but 2.8 trillion parameters at 4 bits should be about 1.4TB. Those do not reconcile. Size from the actual files in the repository — this is the fastest thing to check.
2. **The actual LICENSE text.** Modified MIT is an **inference** from the K2 line's precedent, not a commitment. Open the file and read how the attribution clause is actually worded.
3. **Repository ownership.** Download only from `huggingface.co/moonshotai`.
4. **Any regional restrictions.** This is a new item — if export controls land in the reported shape, future releases may carry geographic access limits. Worth a look at whether any trace appears at launch.
5. **Whether vLLM's KDA support merged alongside.** Weights without the matching inference path are a file self-hosters can download and cannot run.

## The one-line version

If one sentence survives:

> **For the first time, an open-weight release date is not set by training progress.**

Whether and when a Chinese lab publishes its own model weights now depends simultaneously on its engineering schedule, on export controls Beijing is drafting but has not enacted, and on accusations and sanctions threats from Washington. The three clocks are not synchronized, and no engineer controls any of them.

The practical implication for developers is a single plain sentence: **if your architecture assumes weights can be downloaded and self-hosted at any time, that assumption is turning from a technical question into a legal one.**

Weights already downloaded are unaffected. In regulatory coverage that reads as a technical footnote. In an architecture decision it reads as an instruction.

---

*Sources: Hugging Face and GitHub API results were measured by the author across four checks between 17:45 UTC on 2026-07-26 and 01:08 UTC on 2026-07-27. The export-control consultation is from the Financial Times (July 21, 2026) and a same-day Reuters follow-up (Reuters said it could not immediately verify). The distillation accusation is from White House OSTP Director Michael Kratsios's July 22 public statements, Treasury Secretary Scott Bessent's remarks, and reporting by TechCrunch and CyberScoop. Moonshot's release commitment is from its official technical blog. As of publication MOFCOM has issued no public statement, no US enforcement action has been taken, and no technical evidence for the distillation claim has been published. Treat final official texts as authoritative on all policy matters.*
