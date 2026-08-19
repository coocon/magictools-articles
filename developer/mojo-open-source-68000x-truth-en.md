---
title: "Mojo Open-Sourced Its Compiler — and Quietly Dropped the Phrase \"Superset of Python\""
slug: mojo-open-source-68000x-truth-en
summary: "On 2026-08-18, Modular open-sourced the Mojo compiler and full toolchain under Apache 2.0 with LLVM exceptions. But the two most-repeated selling points both need correcting: the \"68,000x faster than Python\" figure comes from a 2023 Mandelbrot blog series whose baseline is a single-threaded pure-CPython loop — Modular itself explained that 35,000x became 68,000x only because they switched to an 88-core machine — and the \"superset of Python\" positioning now reads, in the official roadmap, \"may or may not... and it's okay if it doesn't.\" There's also something the announcement didn't headline: Qualcomm completed its acquisition of Modular on July 29. Here's the exact scope of the release, the conditions the benchmark holds under, the real cost of interop, and whether to pick it up now."
category: developer
tags: [Mojo, Modular, Chris Lattner, programming languages, open source, Python, performance, Qualcomm, MLIR]
coverImage: ""
status: published
locale: en
source: authored
translationSlug: mojo-open-source-68000x-truth
---

# Mojo Open-Sourced Its Compiler — and Quietly Dropped the Phrase "Superset of Python"

On August 18, at its own ModCon conference, Modular announced that the Mojo compiler and its entire toolchain are now open source under Apache 2.0 with LLVM exceptions.

The version of this news that traveled furthest carried two selling points: **"a superset of Python"** and **"68,000x faster than Python."**

Neither is something this announcement said. One is a three-year-old number; the other has been edited out of the official roadmap.

Put both back in their original context and you get a judgment that is less exciting than the headline — and considerably more worth acting on.

---

## 1. Which Part Actually Got Open-Sourced

Start with the boundaries, because "open source" is doing a lot of work in this story.

Modular's wording:

> "We are happy to announce that the Mojo🔥 language is now fully open source under the Apache 2.0 license (with LLVM exceptions)! The source code for the Mojo compiler, tooling, and everything else you need to build the language are now available in our modular GitHub repository."

And their own candid description of what came before:

> "For the last four years, Mojo has been developed with an open community, but a closed compiler."

So what's genuinely new is the **compiler itself and the toolchain** (LSP, debugger, formatter, REPL), living in the `KGEN` directory of the `modular/modular` repo. The path here took three steps: the standard library opened in March 2024, then 450,000 lines of Mojo-written MAX kernel code in the 25.3 release, and now the last piece.

One note on the license: I checked the first line of the repo's LICENSE file, and it is indeed **Apache License v2.0 with LLVM Exceptions**, with no CLA-style commercial riders. But GitHub's sidebar reads it as "Other" because of a custom header block — **don't copy the sidebar**. Modular's stated reasoning is blunt: "The Apache 2.0 license is the gold standard for programming languages and compilers."

Two qualifiers are easy to skip past, and both matter for judging how "open" this is:

**First, the compiler isn't taking outside contributions yet.** Official wording:

> "we aren't ready to take contributions to the compiler and tooling. We aim to accept contributions to the compiler and tooling by the end of this year."

**Second, the MAX engine is not covered by this Apache release.** The repo README states plainly: "MAX usage and distribution are licensed under the Modular Community License." The same-day ModCon announcement says MAX takes a different road — device usage restrictions removed, then "**source-available** with an open alliance program." Note that the official term for MAX is *source-available*, not *open source*. That distinction is deliberate.

The Hacker News thread on whether this counts as open source laid out both defensible positions:

> Lichtso: "Technically source available now with the promise of accepting contributions (thus becoming fully open source) early next year... For many the closed source nature of the compiler was a knock-out criterion. We will see if Mojo can gain traction now or if it has missed its window of opportunity."

> cube2222, in response: "It's open source under the Apache 2 license, not source available. Accepting contributions is not required to be open source. SQLite doesn't accept contributions from random people either."

cube2222 is right on the definition — accepting contributions has never been part of what makes something open source, and SQLite is the standing counterexample. But Lichtso's closing question, whether the window has already closed, is the one that recurs most in that thread.

An incidental delight: someone dug through `KGEN/docs/` and found the compiler design documents still carrying their original classification markings. HN user ModernMech: "'Modular Confidential (obviously), May 14, 2022' — lol, feels like espionage." They opened it thoroughly enough to ship the 2022 internal docs along with it.

## 2. The 68,000x Is From 2023, and the Baseline Is a Single-Core Interpreter

**Neither the open-source announcement nor the 1.0 announcement mentions 68,000x.** It comes from a three-part official blog series in 2023 about the Mandelbrot set (not matrix multiplication):

| Stage | What was done | Speedup |
|-------|--------------|---------|
| Part 1 | Port Python to Mojo + type annotations + algebraic simplification | 89x |
| Part 2 | SIMD vectorization + multicore parallelism | 26,000x |
| Part 3 | Over-partitioning for load balancing | 68,000x |

The thing that matters is **what the baseline is**: a single-threaded, pure-CPython implementation. Not NumPy. Certainly not C.

So 68,000x means: vectorized, compiled Mojo running on 88 cores, versus **a single-core interpreted Python for-loop**.

The most revealing passage is Modular's own explanation in Part 3 of why the number went from the 35,000x they advertised at launch to 68,000x:

> "Why are we getting a 68,000x speedup instead of the advertised 35,000x speedup? In short, the benchmarking system is different. During launch, we evaluated on an AWS r7iz.metal-16xl 32-Core Intel Xeon Gold 6455B, but in this blog we evaluated on a GCP h3-standard-88 which uses an 88-Core Intel Xeon Platinum 8481C."

**They moved to a machine with more cores and the multiplier nearly doubled.** That's an honest technical footnote, but it also tells you that a large share of what this number measures is *how many cores you rented*, not how fast the language is.

On the third-party side: the Julia community ran a comparison in September 2023, and sundarurfriend's summary on HN was "the version of code in [1] is already a few times faster than the Mojo code - because that's pretty basic Julia code that anyone with a little Julia experience could write" — ordinary Julia beats the implementation in Mojo's own docs. There was also a dedicated HN thread that year titled "Is Mojo really 35000x faster than Python?"

To be straight about the limits of this: **Modular has never published a figure for "how many times faster with a NumPy or C baseline," and I found no authoritative third-party number either.** All that's established is the direction — change the baseline and the multiplier collapses by orders of magnitude.

This isn't a Mojo-specific flaw; it's the standard shape of performance marketing. We used the same method taking apart ["Codex made it 232x faster"](/articles/codex-autoresearch-gpu-kernel-232x): **ask what the baseline is before you look at the multiplier.** Reverse the order and any number can impress you.

## 3. "Superset of Python" Has Been Walked Back

When Mojo launched in 2023, the hook was that it would be a *superset of Python* — your existing Python runs as-is, and you add type annotations incrementally to buy performance.

Here is Phase 3 of the current official roadmap:

> "Mojo may or may not evolve into a full superset of Python, and it's okay if it doesn't."

And in the current official FAQ, the superset entry is gone entirely. This was a reversal with no announcement attached. On HN, spprashant's read: "They moved the goalposts." pansa2 added the more substantive point: "It absolutely was [part of the appeal], but IMO was never going to happen... Python has a reputation as a simple language, but it really isn't."

So what is the interop story today?

**You can import arbitrary CPython packages — that part is true.** From the docs:

> "You can import existing Python modules and use them in a Mojo program. This is 100% compatible because we use the CPython runtime without modification for full compatibility with existing Python libraries."

But read the second half — **it uses the unmodified CPython runtime.** Which means those Python libraries still run at CPython speed, plus a layer of cross-language call overhead. MohamedMabrouk, a developer in the NuMojo orbit, put it most clearly on HN: "you can but through a python interpreter in the mojo process so you get the same numpy speed with mojo<->python interop overhead."

**The NumPy you import does not get faster by virtue of running inside Mojo.** The speedup happens only in the parts you rewrite in Mojo.

Core Python features still unsupported, all parked in Phase 3 of the roadmap (i.e. "later"):

- **Classes and inheritance** — "Eventually, we want Mojo to support the core dynamic features that make Python great, including untyped variables, classes, inheritance, etc."
- **Untyped dynamic variables** — "For now, Mojo requires explicit `PythonObject` type annotations."
- **A full async model** — the 1.0 announcement files it under future work: "major capabilities ahead including a robust asynchronous programming model, pattern matching and unions"
- Lambdas only arrived in 1.0 (before that, there were no lambdas at all)

A more accurate description: **Mojo is a systems language with Python-flavored syntax that can call Python. It is not Python.** Its actual lineage got a good one-line summary from totalperspectiv on HN: "It's got an ownership system adjacent to Rust, comptime similar to Zig, and a first class dependent type system." Rust's ownership, Zig's compile-time evaluation, dependent types on top. That is a very different animal from "superset of Python."

## 4. What the Announcement Didn't Headline: Modular Belongs to Qualcomm Now

This is the piece of context I'd have put in the headline, and almost nobody mentioned it.

- 2026-06-21 definitive agreement signed; announced 06-24
- **2026-07-29 acquisition completed**, all-stock, roughly **$3.9 billion**
- Chris Lattner is now EVP, Advanced AI Software and Platforms at Qualcomm Technologies
- The website footer now reads "Modular Inc, A Qualcomm Company"

One uncertainty to flag: **Qualcomm's official announcement did not disclose the price.** The $3.9B figure comes from press analysis of the SEC filing (up to 19.2 million new Qualcomm shares).

Put the acquisition back on the timeline and the motive for open-sourcing gets much clearer. Modular raised $380M across three rounds ($30M seed in 2022, $100M in August 2023, $250M in September 2025 at a $1.6B valuation). HN argued hard about what kind of deal this was:

> jillesvangurp: "If I read between the lines here what happened is the VC money ran out and they arranged some kind of acquihire. That obviously raises a lot of questions about what will happen to Mojo"

> willseth, in response: "$3.9 billion is not an acquihire!"

$3.9B is plainly not an acquihire. But the second half of jillesvangurp's comment is a fair worry: what happens to Mojo now?

From public information the answer is unambiguous: **the language is free, and the money is collected on the three layers above it.**

1. **Modular Cloud** — announced GA at ModCon: shared endpoints billed per token plus dedicated deployments, OpenAI-compatible API
2. **MAX** — stays under the Modular Community License, going source-available with an alliance program; enterprise licensing
3. **Strategic value to Qualcomm** — a cross-hardware AI software stack that doesn't depend on CUDA. ModCon simultaneously announced support for AWS Trainium, Google TPU, and Qualcomm Cloud AI 100

Item 3 is what justifies $3.9B. **Open-sourcing the compiler isn't generosity inside this framework; it's customer acquisition cost** — the more the language gets adopted, the better the business on the two layers above it, and the more credibly Qualcomm can claim it has an anti-CUDA path. That's genuinely good news for users (free, Apache 2.0, irrevocable). Just don't read it as idealism.

## 5. Should You Pick It Up Now

The production-readiness facts, laid out:

| Dimension | Status |
|-----------|--------|
| Version | Mojo 1.0, released 2026-08-11 (with Modular 26.5) |
| Stability promise | "During the 1.x timeframe, changes should primarily be additive... **Breaking changes may still be made**, but will be managed with care" |
| Platforms | macOS + Linux natively; **Windows only via WSL** (native support announced with Microsoft, not shipped) |
| Install | `uv pip install mojo` (distributed as a Python wheel); community packages via `modular/modular-community` |
| Community | Modular reports nearly 200 contributors, 1,100+ PRs, 200K lines changed since the stdlib opened; 27,056 stars on the repo (measured) |
| Production use | Modular itself (the foundation of MAX and Modular Cloud); MiniMax is a flagship Modular Cloud customer. **I could not find a single independent company writing large-scale production code in the Mojo language itself** |

Note the gap between the last two rows: 27K stars on GitHub, and the only language-level production users Modular can name are essentially itself. That's not a knock — it's the normal state of any new language at this stage — but it determines the nature of what you're doing if you invest now. You'd be placing **a bet**, not making **a selection**.

A 1.0 without native Windows support did draw fire, too (HN user vovavili: "A bit odd to have a 1.0 release without Windows support.").

My recommendation, split by who you are:

**Worth your time right now:** people writing GPU kernels, building inference engines, or writing operators for heterogeneous hardware. This is the one scenario where Mojo has no substitute — Python-like syntax for kernels, compile-time metaprogramming, one codebase across CPU/GPU/TPU/Trainium. With Qualcomm behind it, the continuity of this line is arguably more credible than most startup languages.

**Fine to watch until year-end:** people hoping to speed up ordinary Python business code. Wait for two things to land — the compiler actually accepting outside contributions (Modular says end of this year) and native Windows. Until then, your NumPy won't get faster; rewriting will.

**Safe to ignore entirely for now:** anyone shopping for a painless silver bullet to speed up an existing Python project. That promise has been removed from the official roadmap, so stop planning around it.

One filter to keep, usable on any language or framework performance story:

1. **What's the baseline?** An equally optimized competitor, or an unoptimized control group?
2. **Did this announcement produce that number, or is it three-year-old material being re-attached?**
3. **Has the official positioning quietly changed?** Go read the roadmap and FAQ history — walk-backs rarely come with a press release.

Today's story goes three for three.

---

**References**

- [Modular: Mojo is now open source](https://www.modular.com/blog/mojo-open-source)
- [Modular: Mojo 1.0 release announcement](https://www.modular.com/blog/modular-26-5-mojo-1-0-is-here)
- [Modular: ModCon 2026 announcements](https://www.modular.com/blog/modcon-announcements)
- [GitHub: modular/modular (compiler under KGEN)](https://github.com/modular/modular)
- [Mojo Roadmap: the superset walk-back](https://mojolang.org/docs/roadmap/)
- [Mojo docs: Python interoperability](https://docs.modular.com/mojo/manual/python/)
- [The original 68,000x Part 3 (Wayback archive)](https://web.archive.org/web/2024/https://www.modular.com/blog/mojo-a-journey-to-68-000x-speedup-over-python-part-3)
- [HN: Mojo 1.0 (428 points)](https://news.ycombinator.com/item?id=49261128)
- [HN: Mojo is now open source (125 points)](https://news.ycombinator.com/item?id=49348079)
- [Qualcomm completes acquisition of Modular](https://www.modular.com/blog/qualcomm-completes-acquisition-of-modular)
- Related on this site: [The truth about "Codex made it 232x faster"](/articles/codex-autoresearch-gpu-kernel-232x) · [How benchmarks stopped measuring](/articles/benchmarks-stopped-measuring-en)
