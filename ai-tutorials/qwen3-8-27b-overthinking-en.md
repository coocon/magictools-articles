---
title: "Qwen3.8 27B: The New Local-Model Benchmark — Just Turn Off the Default Reasoning First"
slug: qwen3-8-27b-overthinking-en
summary: "A 17GB quantized file scores 52 on Artificial Analysis and draws the best local-model pelican ever — yet Simon Willison clocked the same task at 21 minutes on the default setting versus 137 seconds with reasoning off. Here's what Qwen3.8 27B can really do, the quantified evidence of its overthinking, and how to tune reasoning_effort."
category: ai-tutorials
tags: [Qwen, local models, LLM, open source models, Simon Willison, model evaluation]
coverImage: ""
status: published
locale: en
source: authored
translationSlug: qwen3-8-27b-overthinking
---

# Qwen3.8 27B: The New Local-Model Benchmark — Just Turn Off the Default Reasoning First

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

His verdict on the default: "This is a *hilarious* default. It's absolutely not a good way to run the model, especially on consumer hardware."

Remarkably, Artificial Analysis's data independently corroborates the verbosity: running the full Intelligence Index, Qwen3.8 27B generated **160 million tokens**, against a median of 43 million across all ranked models — **nearly 4x the median output**. The overthinking isn't a vibe; it has an itemized bill.

## But Reasoning Isn't Useless

After roasting the default, Simon offered the counterexample: he asked the model to produce bounding-box annotations and build an annotation tool. With reasoning off, the result "nearly works but shows the boxes in the wrong place"; with reasoning on, it got them right. His words: "this is a good example of how reasoning can make a difference."

So the correct conclusion isn't "reasoning is useless" — it's that **reasoning depth should be allocated per task, and this model ships with the most expensive tier as the factory default**.

## Hands-On: Tuning the Tiers, Rescuing the Speed

**Reasoning tiers.** `reasoning_effort` has three levels: `xhigh` (default) / `medium` / `low`, and reasoning can be disabled entirely. Simon's explicit advice: "My strong recommendation: ignore that default. Run Qwen 3.8 27B on low or even no reasoning levels at first." Use low/off for everyday Q&A, code completion, and format conversion; escalate to medium/xhigh only for genuinely multi-step tasks (complex debugging, spatial-understanding problems).

**Speed.** Dense architectures are memory-bandwidth-hungry — the structural reason this model isn't fast locally. Simon measured 15–30 tokens/s in LM Studio and admitted "it feels slow... it's going to be hard to win me away from hosted API models." But there's a comeback lever: the model ships with built-in **Multi-Token Prediction (MTP)**, and enabling it in llama.cpp with `--spec-type draft-mtp` gave Simon roughly a **72% speedup** on the DGX Spark (technique via a Georgi Gerganov tweet). If you run llama.cpp/llama-server, don't miss that flag.

## Why This Model Matters

Two lines from Simon's overall assessment are worth writing down.

One about the present: "The fact that a 17GB file can do all of this stuff on my home machines is a *miracle*." He also confirmed it produced "by far the best pelican SVG I've been able to generate with a model that runs on a local machine," that its visual bounding boxes were startlingly accurate, and that it could drive the Pi coding agent to read Datasette's source and build working tools.

One about the trend: "The most important thing about Qwen 3.8 27B is **what it demonstrates**." A year ago, this level of capability required flagship models weighing hundreds of gigabytes; now it's compressed into a 17GB, Apache 2.0 file anyone can download. Dense catching MoE, local catching cloud — faster than any vendor's roadmap.

Just remember: after the download finishes, the first thing to do is turn `reasoning_effort` down. A great model with the wrong default will happily waste 21 of your minutes.

---

*Sources:*
*Simon Willison: [Qwen 3.8 27B is excellent, but it defaults to wildly overthinking things](https://simonwillison.net/2026/Aug/16/qwen-38-27b/)*
*Artificial Analysis: [Qwen3.8 27B model page](https://artificialanalysis.ai/models/qwen3-8-27b)*
*Qwen official: [release blog](https://qwen.ai/blog?id=qwen3.8) / [Hugging Face model card](https://huggingface.co/Qwen/Qwen3.8-27B-FP8)*
*Hacker News: [launch thread (1,423 pts)](https://news.ycombinator.com/item?id=49299605) / [Simon's review thread (751 pts)](https://news.ycombinator.com/item?id=49324985) / [AA score thread (305 pts)](https://news.ycombinator.com/item?id=49334544)*
