---
title: "Kimi K3's Weights Landed Right on Deadline. The Same Day, Anthropic's CEO Published a Denial"
slug: kimi-k3-weights-land-dario-denial-en
summary: "On July 27, two things happened within hours of each other: Moonshot shipped Kimi K3's 2.8-trillion-parameter weights to Hugging Face just inside its 'by July 27' deadline (1,314 points on Hacker News), and Dario Amodei personally published 'Our position on open-weights models,' opening with a denial: Anthropic has never advocated a ban. This piece verifies what actually shipped, reads the benchmark footnotes, and lays out why 665 HN comments largely refused to take yes for an answer."
category: ai-tutorials
tags: [Kimi K3, open-weights models, Anthropic, Dario Amodei, Moonshot AI, AI policy, model distillation, MoE, open source AI]
coverImage: ""
status: published
locale: en
source: authored
translationSlug: kimi-k3-weights-land-dario-denial
---

# Kimi K3's Weights Landed Right on Deadline. The Same Day, Anthropic's CEO Published a Denial

First, let's close out the cliffhanger.

Two days ago, in [Three Clocks](/articles/kimi-k3-weights-three-clocks-en), I logged the state of play: as of 01:08 UTC on July 27, the K3 repo under `moonshotai` on Hugging Face still did not exist — but Moonshot's own wording had always been that weights would ship *by* July 27, which gave them another 23 hours before anyone could fairly say "late."

They made it. **Right on the wire.**

On July 27, `moonshotai/Kimi-K3` went live, and the Hacker News post climbed to **1,314 points and 516 comments** — the top story of the weekend. People in the thread were literally sharing a countdown page, waiting for the gates to open like it was New Year's Eve.

And then, the same day, the other thing happened: **Dario Amodei published a personally signed post on Anthropic's site** titled *Our position on open-weights models*. The first section gets straight to the point:

> "Anthropic has never advocated for a ban on open-weights models."

The CEO of the industry's most safety-forward frontier lab, publishing a denial letter on the very day of the largest open-weights release in history — those two events, side by side, are the whole plot of this weekend in AI.

This piece does two jobs: verify what actually shipped and how to read K3's benchmark table, then faithfully report what Dario actually said — and why 665 HN comments largely refused to take yes for an answer.

## 1. What you actually get in the download

Per the model card and technical report, this is more than a weights file:

- **Scale**: a 2.8-trillion-total-parameter MoE with 104B activated per token; 896 experts with 16 selected (plus 2 shared). Moonshot calls it "the world's first open 3T-class model."
- **Architecture**: 93 layers — 69 running Kimi Delta Attention (KDA) and 24 running gated MLA — plus Attention Residuals; natively multimodal via a 401M-parameter MoonViT-V2 vision encoder; a 1-million-token context window.
- **Precision**: MXFP4 weights / MXFP8 activations from quantization-aware training — native, not a post-hoc squeeze. The full download is roughly 1.5TB.
- **Infrastructure shipped alongside**: MoonEP (expert parallelism), FlashKDA (KDA kernels), and AgentEnv. *Three Clocks* noted that KDA is incompatible with ordinary prefix caching, and that shipping weights without the serving stack would mean shipping a file most people can't run. They shipped the stack too.

**The license has one clause that is not MIT**, and hosts should read it in the original: if you operate a model-as-a-service business on K3 and your aggregate revenue exceeds **$20 million** over any consecutive 12 months, you need a separate agreement with Moonshot; products over 100M MAU or $20M revenue must display "Kimi" attribution. Irrelevant to almost every researcher and small team. Not irrelevant to inference providers.

As for "it's open, so anyone can run it" — no. Native MXFP4 still means ~1.5TB of VRAM. One infrastructure-minded HN commenter put the floor at 8×B200, with 16 being the realistic configuration once you account for context and throughput. **The direct beneficiaries of this release are hosting providers and clouds, not your workstation.**

## 2. Reading the benchmark table: wins, losses, and what's in the footnotes

The model card stacks K3 against Claude Fable 5, GPT-5.6 Sol, Claude Opus 4.8, GPT-5.5, and GLM-5.2 across dozens of rows. Following the habit from [Four Numbers Everyone Is Misreading](/articles/kimi-k3-open-weights-four-numbers-en): footnotes first, numbers second.

**Clear wins**: BrowseComp 91.2 (vs. Fable 5's 88.0), MCPMark-Verified 94.5 (vs. 87.4), OmniDocBench 91.1, τ³-Banking. The most striking row is **SWE-Marathon: 42.0 against Fable 5's 35.0** — the ~400-hour-task, compound-reward benchmark discussed in [our lights-off software factory piece](/articles/lights-off-software-factory-postmortem-en). If that score holds up, it lands precisely on the objection that open models only sprint short tasks.

**Clear losses**: FrontierSWE 81.2 vs. Fable 5's 86.6; a GDPval-AA v2 Elo of 1686 vs. 1747; OSWorld 2.0 trailing by about 8 points. On the long tail of real engineering and general computer use, the closed frontier still leads.

**The footnoted conditions**: K3's coding scores were produced with Moonshot's own Kimi Code harness, while competitors get their *best available* harness — Terminus 2 for Claude on Terminal-Bench, Codex for GPT. To their credit, the footnotes also report a neutral-harness reference: with mini-SWE-agent, K3 scores 67.3 on DeepSWE versus 67.5 with its own harness — nearly no gap. Honest disclosure. But the conclusion still needs its qualifier: **"an open-weights model has entered the top tier" is now true; "open weights have fully caught the closed frontier" is not.**

## 3. The other letter that day: what Dario actually said

Now the Anthropic side. Context first: over the past week, reports said US officials were weighing a **ban on US companies using Chinese open-weights models**; NVIDIA, Microsoft, and many others signed an open letter opposing it (background in [Open-Weight AI's Kubernetes Moment](/articles/open-weight-ai-kubernetes-moment-en)); and parts of the discourse accused Anthropic of quietly wanting that ban to protect its business.

Dario's response deserves faithful summary, because it is more nuanced than either camp's cartoon of it. Three layers:

**Layer one: he opposes the ban, explicitly.** "Open-weights models that don't have dangerous capabilities are a public good." Banning their use by US businesses "does nothing to address" his security concerns, and "protecting US AI companies from competition has never been my goal." Note what this means: the ban is being drafted by officials in Washington, and the company accused of pushing it just publicly removed itself from the ban camp. That is, structurally, an assist for open weights.

**Layer two: two nightmare scenarios.** First, authoritarian governments — he names the CCP as "clearly the most capable threat" — building models stronger than America's for military superiority and domestic repression; he stresses this has nothing to do with whether weights are open ("the most dangerous model may be one trained in secret and handed only to the People's Liberation Army"). Second, misuse of powerful models for cyber and biological attacks — where open weights do carry higher risk, because guardrails can be stripped and released weights can never be withdrawn.

**Layer three: three concrete asks.** Keep advanced chips and chipmaking equipment out of China and crush smuggling; crack down on "industrial-scale distillation operations"; and subject all sufficiently capable models — open and closed alike — to mandatory safety testing, applied globally, "which means even the CCP would need to be on board."

## 4. Why 665 comments didn't buy it

The comment section on this denial was among the most hostile I've seen aimed at Anthropic. The strongest objections deserve to be stated properly — and so do the fair defenses.

**Objection one: "mandatory testing" is a ban's enforcement mechanism.** The most-discussed comment, from cogman10: who administers the test? What if it's too costly, or the administrator simply refuses to certify certain parties? "This is exactly how the US has banned goods in the past — by requiring a stamp and then refusing to issue it." Commenter modeless pressed the adjacent point: what happens when a model *fails*? Dario never says — and if he won't name the consequence, it is hard to imagine it being anything other than a ban.

**Objection two: the distillation clause is ladder-pulling.** Multiple commenters noted that Anthropic trained on millions of books — litigation about which concluded barely a year ago — and now wants "industrial-scale distillation" cracked down upon. "No fair stealing the value we already stole fair and square," as one put it. High emotional voltage; but for accuracy's sake, Dario's stated objection to distillation is geopolitical (it lets China's frontier ride "within a few months" of America's despite chip limits), not an intellectual-property claim. The analogy lands emotionally; it doesn't quite land logically.

**Objection three: the chip premise.** "China cannot build more powerful models than the US without US chips" was challenged directly in the thread — and K3 going live *the same day* did the challenging more eloquently than any comment. In fairness: what silicon K3 was trained on is not publicly established, so on this point both sides are asserting, not proving.

**The fair defenses exist too.** From arjie: "This seems like the maximal position he could take compatible with his expressed principles. There's no way to allow bioweapon-grade models to be open-weights if one doesn't want widespread human damage. It's only a matter of degree, and whether we're already there." Another commenter put it more simply: the position benefiting Anthropic doesn't make the risks fabricated.

## 5. My take: the coordinate system of the debate moved in one day

Put the week on a single timeline: July 21, Beijing reported to be drafting weight export controls. July 25, Washington reported to be weighing a use-ban; the industry's open letter lands. July 27, the weights land — and so does the denial.

Three readings:

**First, "can open weights reach the top tier" went from prediction to fact, and the debate was forced upstairs.** Last week the argument was whether to ban; any ban proposal now has to reckon with an irrevocable 1.5TB artifact already distributed worldwide. Note the symmetry: *both sides cite irrevocability*. Dario lists it as the core risk — safeguards strippable, weights unrecallable. The open-weights camp holds it up as the core guarantee — no government can ever take this back. **The same physical fact is a risk or a guarantee depending on whose hands you trust more.**

**Second, the next battle is over who issues the stamp.** If mandatory safety testing becomes the consensus — and Dario notes both recent White House direction and industry proposals pointing that way — then the real power sits with whoever administers the test and writes the pass criteria. The "stamp ban" worry isn't trolling; it's a standard chapter of US trade history. For the open-weights camp, fighting for the neutrality of the testing regime will matter more than chanting against regulation.

**Third, for working developers, the real change shows up on price sheets.** K3's hosting floor (8–16×B200) means the market is about to produce its first honest answer to "what should a million tokens from a 3T-class model cost." One HN reference point: GLM-5.2's third-party hosting prices reportedly fell ~45% in the six weeks after its release — a number disputed in the same thread, because the cheapest hosts serve lower-precision quantizations, an apples-to-oranges trap we covered in [the token relay market piece](/articles/llm-token-relay-market-anatomy-en). If you build agent products: over the coming weeks, put K3's third-party quotes next to the official API, *with quantization labeled*, and watch. That table is where this release actually lands.

One closing detail. The same day Dario's post went up, another story took 256 points on HN: the Financial Times reporting that AI companies' Washington lobbying spend has hit record highs. One commenter suggested that, as a gesture of good faith, the lobbying budget could pause for a couple of years.

Unkind — but it names the real problem: **however clearly the position paper is written, the industry's trust deficit about who is shaping the rules is not something one blog post can repay.**

---

**References**

- [moonshotai/Kimi-K3 · Hugging Face](https://huggingface.co/moonshotai/Kimi-K3)
- [Kimi K3 Technical Report (PDF)](https://github.com/MoonshotAI/Kimi-K3/blob/main/k3_tech_report.pdf)
- [Anthropic: Our position on open-weights models](https://www.anthropic.com/news/position-open-weights-models)
- [HN discussion: Kimi-K3 on HuggingFace (1,314 points)](https://news.ycombinator.com/item?id=49065752)
- [HN discussion: Anthropic's position (476 points)](https://news.ycombinator.com/item?id=49076057)
- [FT: AI companies spend record sums on Washington lobbying](https://www.ft.com/content/d8a5f95e-3b6d-463a-a848-c9ef8e2394db)
