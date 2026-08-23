# Qwen3.8 27B: The New Local-Model Benchmark — Just Turn Off the Default Reasoning First

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/qwen3-8-27b-overthinking-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/qwen3-8-27b-overthinking-en?utm_source=github&utm_medium=referral)**

The same model earned two opposite headlines in the same week.

On one side, Artificial Analysis's independent evaluation: an **Intelligence Index of 52** — up from 38 for the previous-generation Qwen3.6 27B, a 14-point generational jump that puts a 27B dense model within striking distance of flagship MoEs (Qwen3.8-Max itself scores 58).

On the other, Simon Willison's August 16 write-up, whose title pulls no punches: "Qwen 3.8 27B is excellent, but it defaults to **wildly** overthinking things."

Both are true, and each explains the other. Three Hacker News waves totaled over 2,400 points (launch thread 1,423, Simon's post 751, the AA score thread 305) — likely the local-model scene's loudest week this year.

---

## The Model Itself, First

Qwen3.8 27B was released by Alibaba's Qwen (Tongyi Lab) on August 14: 27.78B parameters, **dense architecture** (not MoE), multimodal input (text/image/video), native 262,144-token context (extendable to 1M via YaRN), Apache 2.0 license. The number that matters most to local users: the Q4_K_M quant is **just 17GB** — it runs on a 24GB-VRAM GPU or a big-memory Mac.

The self-reported benchmarks are aggressive: SWE-Bench Pro 61.7%, LiveCodeBench 90.3%, GPQA Diamond 89.2%. Standard caveat: these are vendor-reported, some on internally modified benchmarks, with no independent reproduction yet — the number with third-party backing is that AA 52.

## 21 Minutes vs. 137 Seconds: What Overthinking Looks Like

Simon tested on a 128GB M5 Max MacBook Pro and an NVIDIA DGX Spark. The problem is the factory default: `reasoning_effort` ships set to **xhigh**.

The consequences? His signature "draw a pelican riding a bicycle as SVG" test took **21 minutes** on the default setting, burning 22,276 reasoning tokens; with reasoning off, the same task finished in **137 seconds**. Even more absurd: asked to "draw an SVG of a circle," the model spiraled into a self-invented Bauhaus-style animated ring design study — in Simon's words, "absolutely beautiful... which was entirely not what I had asked for!"

...

---

**[👉 Continue reading: Qwen3.8 27B: The New Local-Model Benchmark — Just Turn Off the Default Reasoning First](https://tools.cooconsbit.com/en/articles/qwen3-8-27b-overthinking-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
