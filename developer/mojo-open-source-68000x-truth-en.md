# Mojo Open-Sourced Its Compiler — and Quietly Dropped the Phrase "Superset of Python"

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/mojo-open-source-68000x-truth-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/mojo-open-source-68000x-truth-en?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 Continue reading: Mojo Open-Sourced Its Compiler — and Quietly Dropped the Phrase \"Superset of Python\"](https://tools.cooconsbit.com/en/articles/mojo-open-source-68000x-truth-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
