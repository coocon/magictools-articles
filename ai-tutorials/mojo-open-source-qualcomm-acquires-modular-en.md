# Mojo Went Open Source. Its Creator Now Works for Qualcomm.

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/mojo-open-source-qualcomm-acquires-modular-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/mojo-open-source-qualcomm-acquires-modular-en?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 Continue reading: Mojo Went Open Source. Its Creator Now Works for Qualcomm.](https://tools.cooconsbit.com/en/articles/mojo-open-source-qualcomm-acquires-modular-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
