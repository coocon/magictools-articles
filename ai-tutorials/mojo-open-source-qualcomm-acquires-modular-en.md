---
title: "Mojo Went Open Source. Its Creator Now Works for Qualcomm."
slug: mojo-open-source-qualcomm-acquires-modular-en
summary: "Mojo 1.0 is fully open source, compiler and all. But two months before that, the man who wrote it — plus 150 colleagues — was bought by Qualcomm for nearly $4 billion."
category: ai-tutorials
tags: [Mojo, Modular, Qualcomm, CUDA, Chris Lattner, compilers, AI chips]
coverImage: ""
status: published
locale: en
source: authored
translationSlug: mojo-open-source-qualcomm-acquires-modular
---

# Mojo Went Open Source. Its Creator Now Works for Qualcomm.

> Sourced from Modular's own blog posts (the ModCon 2026 announcements, the Mojo open source post, and the Modular × Qualcomm engineering deep-dive) and WIRED's acquisition report of June 24, 2026. All quotes are verbatim.

---

On August 18, 2026, at ModCon, Modular announced that Mojo 1.0 is fully open source. Apache 2.0 with LLVM exceptions. The compiler and the entire toolchain, published to GitHub.

For a language whose whole reason to exist was breaking a monopoly, that is the most complete move it could make.

The footer of that same announcement reads: **Copyright © 2026 Modular Inc, A Qualcomm Company.**

In June, Qualcomm announced it would acquire Modular for nearly $4 billion. The deal closed before ModCon. By the time Mojo's compiler source actually went public, the person who wrote it was already a line on Qualcomm's org chart.

Chris Lattner spent four years trying to break one chip company's grip on the AI software stack — in a way he described as *structural*.

A different chip company bought him instead.

Here are the ten things about this story that actually deserve your attention.

---

## 1. The $4 Billion Bought Two Words: "No Rewrite"

> "We believe the future belongs to developer-friendly, horizontal platforms that can run across diverse compute environments and give customers real choice in how and where they deploy AI."
>
> — Cristiano Amon, President and CEO, Qualcomm

Start with the price. Qualcomm expects to issue up to 19.2 million shares of common stock, which works out to just under $4 billion at the prior closing price. Nine months earlier, Modular had raised $250 million at a $1.6 billion valuation. Nine months. 2.5x.

The headcount: two cofounders and roughly 150 employees, all expected to join Qualcomm. That's about $26 million per person.

Nobody pays that for a compiler. Qualcomm paid it for a promise: when your customers switch silicon, they don't rewrite their code.

Modular's engineering blog states the pitch more plainly than any press release: every time a new AI accelerator ships, developers ask how much of the stack they have to rewrite — and the answer Modular is selling is **none**.

**My take:** Qualcomm bought time, not technology. The historical answer to "how long does it take a chip company to build a software stack that can stand against CUDA" is five to ten years, and usually it fails anyway — AMD's ROCm is the standing evidence of how brutal that road is. In a market where inference silicon turns over every three years, $4 billion to skip five of them is cheap. The expensive part was never the chip. It's convincing people to write code for it.

---

## 2. The Word "Structural" Came Back for the Man Who Said It

> At the time, Lattner said he believed that he and Davis were tackling a software problem that had to be solved outside of a Big Tech environment because it was "structural." Ultimately, the structure of Qualcomm won out.
>
> — WIRED, June 24, 2026

That's the sharpest sentence in the entire WIRED piece, and it's the thesis of this whole story.

Lattner's original diagnosis: the fragmentation of the AI software stack is not a technical problem but a structural one. Every chip company has an incentive to weld its software to its own hardware, so the problem cannot be solved from inside any of them. To solve it, you have to stand outside all of them.

The diagnosis was right. What he underestimated was the economics of standing outside all of them.

**My take:** A neutral software layer has a birth defect — it's useful to everyone, which means nobody will pay real money for it. Chip vendors will fund software that sells their silicon. They will not fund software that also sells their competitor's silicon. Solving a structural problem requires structural resources, and in this industry, structural resources sit on exactly one kind of balance sheet. This isn't Modular's failure. It's the standing condition for every neutral-infrastructure startup.

---

## 3. He Never Actually Left the Hardware Companies

> "What makes this team truly exceptional is the complementary partnership between Chris and Tim. Chris is an N-of-1 human, in that he's bold, visionary, and technically uncompromising."
>
> — Dave Munichiello, managing partner at GV, an early Modular investor

Lay out Lattner's résumé and a different pattern shows up.

LLVM started as a master's thesis at the University of Illinois, but it only became industrial infrastructure after Apple hired him in 2005 to make it production-grade. Clang, Swift, the Xcode toolchain — all built inside Apple, over twelve years. MLIR was born at Google, in service of TPUs. In between there was Tesla, where he ran Autopilot software for under five months before Andrej Karpathy eventually took the role. Then SiFive, a RISC-V chip company.

Now Qualcomm.

**My take:** The history of "open infrastructure" is really the history of which rich company was willing to underwrite it. LLVM became the industry's foundation not because it was open source, but because Apple fed it for over a decade. Lattner's rarest skill was never compiler design alone — it was convincing a giant that funding open infrastructure served the giant's interest. Founding a startup was, in one reading, an attempt to skip that step. Four years later, he's back at it.

Worth noting: Mojo compiles down to Qualcomm's Hexagon LLVM NPU backend. The bridge that carried him into Qualcomm is one he built himself, twenty years ago.

---

## 4. The Compiler Is Open. Your Pull Request Is Not Welcome Yet.

> "One learning (particularly in today's era of AI coding) is that we need to be deliberate about how we handle contributions. As such, we aren't ready to take contributions to the compiler and tooling. We aim to accept contributions to the compiler and tooling by the end of this year."
>
> — Modular, Mojo open source announcement

The source is all there. Apache 2.0 with LLVM exceptions. Read it, modify it, fork it, ship commercial products on it.

You just can't commit to it yet.

The stated reason — that the era of AI-generated code demands more deliberate contribution governance — is entirely credible. Anyone who has maintained a popular repository in the last two years knows exactly what drowning in machine-written PRs feels like.

**My take:** Open source and open governance are two different things, and conflating them is amateur hour. What Apache 2.0 gives you is the **right to exit** — fork it if you hate where it's going. It does not give you a **vote** on where it goes. That's consistent with Lattner's stated philosophy, which the announcement spells out: the "soul" of a language comes from small, tight-knit design teams, not committees. Swift ran the same playbook in its early years.

So don't read "Mojo is open source" as "Mojo is now community-governed." The first statement is true. The second isn't. The signal actually worth watching is whether that end-of-year commitment lands, and whether the first outside compiler PR gets merged.

---

## 5. The Real Asset Isn't Mojo. It's MAX — And MAX Isn't Open Source.

> "The MAX license no longer contains device usage restrictions, and MAX will be source-available with an open alliance program, so the broader ecosystem can build the platform with us."
>
> — Modular, ModCon 2026 announcements

This line got buried under the Mojo headline, and commercially it matters more.

Mojo is **open source** (Apache 2.0). MAX is **source-available**. Those terms are not synonyms: one means you can use and redistribute freely, the other means you can read the code under a license that constrains what you do with it.

And MAX is the part that makes money — the graph compiler, the runtime, the serving framework, the whole multi-backend inference stack. It's what Modular Cloud runs on. It's what serves MiniMax's production traffic at billions of tokens per minute.

**My take:** Open the language to win the ecosystem, keep the platform to win the revenue. This is textbook moat construction — Java, .NET, and Swift all ran versions of it. There's nothing to condemn here, but be clear-eyed: open-sourcing Mojo cost far less commercially than it earned in goodwill.

The real test of Modular's independence under Qualcomm won't be Mojo's license today. It'll be MAX's license a year from now. "Source-available" and "open alliance program" leave an enormous amount of room for interpretation.

---

## 6. "We Are Not Copying CUDA"

> "We are not copying CUDA; we are building a custom path into a portable stack."
>
> — Modular × Qualcomm engineering blog

Every team building a "CUDA alternative" should tape that sentence to the wall.

The numbers behind it: Modular and Qualcomm engineers brought the Cloud AI 100 Ultra into MAX and Mojo. The first end-to-end GPT-2 run came in at **1.6× slower** than the PyTorch baseline. Roughly three weeks of optimization later, they hit **6.7× the baseline**. From the first Gemma 4 kernel to a live serving endpoint took **under six months**. The bring-up went in stages: get the Mojo compiler running on the device, use GPT-2 as a "Hello World" to validate the full pipeline, scale to Gemma 4 31B across four chips in production serving, then compress new models and features from weeks to days.

**My take:** For a decade, nearly every CUDA challenger picked the same strategy — build a compatibility layer, run CUDA code unchanged on someone else's silicon. That approach has a structural flaw: you are permanently chasing a moving target whose owner controls the movement.

Modular refused to compete at that layer. Instead it raised the abstraction — developers write model definitions and Mojo kernels, and hardware differences get absorbed by the compiler. That's the only posture with a real chance: don't imitate the monopolist's interface, make the interface layer stop mattering.

And of those numbers, 6.7× is the least interesting one. "Three weeks" and "under six months" are the story. **CUDA's moat was never performance. It's switching cost.** What this port actually measured is how far switching cost can be driven down.

---

## 7. An NPU Is Not a GPU — That's Where the Proof Is

There's a block of hardware detail in the engineering post that's easy to skip past. It's also what determines whether any of this generalizes.

The Qualcomm Cloud AI 100 Ultra is four SoCs on one card, 32 GB each, with each SoC made of 16 NSPs. Its differences from a GPU aren't differences of degree — they're differences of paradigm:

- It's **SIMD, not SIMT**. The kernel programming model is fundamentally different.
- There's **no unified memory**. Data moves between on-chip VTCM and off-chip DDR through explicit DMA. You cannot lean on a cache hierarchy to hide it for you.
- The HMX matrix unit demands a specific "crouton" data layout, and the conversion costs something. So for batch-size-1 decode, the team routes around HMX entirely and runs a GEMV kernel on the HVX vector unit — if the conversion doesn't pay, don't pay it.
- Gemma 4 31B needs roughly 60 GB in fp16 for weights alone, nearly double a single SoC's 32 GB. It has to be sharded across four devices with tensor parallelism.

This is the first ASIC (NPU) to enter the Modular stack.

**My take:** "Portability" is a badly abused word. Porting from NVIDIA to AMD isn't a real test — both are SIMT, both have unified memory, the programming models are isomorphic. That's translating between dialects. The real test is porting to an ASIC with a completely different execution model *without* forking the model definition, the graph compiler, or the serving layer.

Modular claims all hardware targets live in one repository and share the vast majority of the code. If that holds, it proves something far bigger than a 6.7× number: **the hardware dependency in an AI software stack can be isolated into one thin abstraction.** That is precisely the proposition NVIDIA would prefer stayed unproven.

---

## 8. Qualcomm Isn't Betting on Silicon. It's Betting on a Position.

String together Qualcomm's last two years and this acquisition stops looking surprising.

Late last year it acquired Ventana Micro Systems, a startup building RISC-V server CPUs. It's working on custom ASIC designs for data centers, with ByteDance reported as an early customer. Amon has said the company has been working on **40 different chip designs** for AI gadgets — smart glasses, jewelry, earbuds, pins, watches. In the data center: the Cloud AI 100 line and Dragonfly AI 200/250/300.

Now, a full software stack.

**My take:** Qualcomm's position rhymes with ARM's a decade ago. It cannot beat NVIDIA in a head-on fight — process node, packaging, HBM supply, the CUDA ecosystem, each is a ten-year accumulation. So the only option is to change what the fight is about.

Software is the only lever that changes the dimension of competition. If "write once, run on any accelerator" actually holds, hardware procurement stops being decided by "whose software ecosystem is better" and starts being decided by performance per dollar per watt — which happens to be the exact battlefield Qualcomm has been winning on mobile for decades.

This wasn't a tool purchase. It was an attempt to rewrite the scorecard.

---

## 9. The Neutrality Pledge, and Its Shelf Life

> "Modular Platform will continue supporting and optimizing for a broad range of hardware, including hardware that competes directly with Qualcomm Technologies' platforms. The opportunity in front of the ecosystem is much bigger than any single vendor's roadmap, and a foundation only works if everyone can stand on it."
>
> — Modular, ModCon 2026 announcements

Well written, and necessary. Without it, AWS, Google, and d-Matrix would all be re-evaluating their partnerships this week.

The current evidence isn't bad either. Modular Platform already supports AWS Trainium, Google TPUs, and NVIDIA and AMD GPUs. The Google TPU support was actually brought up by engineers at HTEC, an outside firm, in a few months with a few engineers, with Modular in a supporting role. That's the strongest proof available that the foundation can be extended by people who don't work there.

**My take:** On the day of the acquisition, that pledge is one hundred percent sincere. The problem isn't sincerity. It's scheduling.

Three years from now, when Dragonfly AI 300 support and a competitor's new accelerator land in the same quarter, which one goes first? No CEO answers that question in the competitor's favor. It requires no conspiracy — only finite engineering capacity and a roadmap owned by Qualcomm.

So ignore the blog posts and watch the schedule. **The concrete test: does production support for Trainium, TPU, and d-Matrix advance in step with the Dragonfly line?** If a year from now the non-Qualcomm targets are still "coming to production over the coming months," the pledge has already expired — not torn up, just quietly starved of priority.

---

## 10. So Should You Actually Learn Mojo?

Trace the three-year arc and Modular's open-sourcing has been remarkably disciplined: the standard library in 2024, hundreds of thousands of lines of MAX kernels in 2025, the whole compiler in 2026. Each step balanced ecosystem pressure against commercial reserve.

Add the rest of the ModCon news: native Windows support is coming to Mojo, built with the Microsoft Windows team — previously Windows developers had to go through WSL. Modular Cloud is generally available, after quietly serving OpenRouter traffic for months under the name ModelRun, consistently ranking at or near the top of the platform on latency and throughput.

None of that is slideware. It's already running.

**My take:** This question needs two answers, because the acquisition pushed the two risks in opposite directions.

**Betting on the language got safer.** Mojo 1.0 ships a source stability guarantee. Apache 2.0 with LLVM exceptions gives the most complete exit right available. A fully open compiler means that in the worst case, the community can fork. At the language level, code you write today doesn't die because the company changed owners — which is what open-sourcing actually buys at this particular moment, and plausibly one reason it happened *after* the deal closed rather than before.

**Betting on the platform got riskier.** MAX and Modular Cloud now belong to a company with an explicit hardware position. If your inference business is going to live on this stack for years, you need an exit plan and a habit of checking whether the neutrality pledge is being honored.

Concretely: if you work on high-performance kernels, heterogeneous compute, or compilers, Mojo is worth your time now. With the compiler open, it stopped being a product and became a readable implementation of some genuinely new ideas. If you're choosing a platform for production inference, evaluate it as an excellent piece of technology with a known owner — not as neutral infrastructure.

---

## The Last Word

Four and a half years ago, Modular made a bet: AI would not run on one kind of silicon forever, and the software stack would have to be rearchitected for heterogeneous hardware.

The bet was right. Heterogeneous compute arrived. Trainium, TPUs, NPUs, and a parade of ASICs are all pushing into production, and they genuinely need a common software foundation.

The foundation just didn't end up standing outside all of them. It ended up on one of their balance sheets.

Whether CUDA's monopoly gets broken is still an open question. What's settled is the method: it won't be broken by an independent company winning on neutrality.

That sentence is worth reading twice: **Ultimately, the structure of Qualcomm won out.**
