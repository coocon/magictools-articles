---
title: "DiffusionGemma Doesn't Make Pictures: A 1,500 tokens/s Diffusion Language Model"
slug: diffusiongemma-diffusion-language-model-en
summary: "Seeing \"Diffusion\" and thinking image generation is the easiest way to misread this technical report. DiffusionGemma generates text — it refines blocks of 256 tokens in parallel and hits roughly 1,500 tokens/s on a single H100. The more interesting part is how it was built: not trained from scratch, but fine-tuned from Gemma 4's MoE model using under 10% of the original training token budget."
category: ai-tutorials
tags: [DiffusionGemma, diffusion-models, LLM, Gemma, inference-optimization, Google, open-weights]
coverImage: ""
status: published
locale: en
source: authored
translationSlug: diffusiongemma-diffusion-language-model
---

# DiffusionGemma Doesn't Make Pictures: A 1,500 tokens/s Diffusion Language Model

> "Rather than decoding one token at a time, DiffusionGemma iteratively refines blocks of 256 tokens in parallel, avoiding the sequential decoding bottleneck of conventional autoregressive large language models."
> — DiffusionGemma Technical Report

---

Let's clear up the easy mistake first: **DiffusionGemma generates text, not images.**

The confusion is understandable. For the past few years "diffusion" has been effectively synonymous with image generation in public conversation — Stable Diffusion, Midjourney, every text-to-image model. So when Google publishes something called DiffusionGemma, reading it as "the Gemma family got an image model" is a natural reaction. It's also completely wrong.

This technical report ([arXiv:2608.00146](https://arxiv.org/abs/2608.00146), submitted July 31, 2026, credited to the DiffusionGemma Team and 43 authors) is about a **discrete diffusion language model**: applying the diffusion idea — repeatedly denoising toward a target — to token sequences instead of pixels.

The subject classifications confirm it. The paper is filed under cs.CL (Computation and Language) and cs.AI. Not cs.CV.

## 1. The problem it actually attacks

Every mainstream LLM today is autoregressive (AR): predict the next token, append it, predict the next. That loop has an unavoidable property — **it is strictly serial**. Generating 1,000 tokens takes 1,000 forward passes, and token 500 cannot be computed until token 499 exists.

That's the fundamental inference bottleneck. It isn't a compute shortage; it's a structure that forbids parallelism. Years of work routing around it has been clever but additive: KV cache eliminates redundant computation, speculative decoding has a small model guess a batch that a large model verifies in bulk, batching strategies raise throughput without improving single-request latency.

Diffusion language models take a different route: **don't generate one at a time — process a whole block and refine it repeatedly.**

DiffusionGemma refines **blocks of 256 tokens** in parallel. Each forward pass doesn't emit one token; it moves all 256 positions one step toward "more like the correct answer." After enough iterations, the block is final.

The key number in the abstract: **roughly 20 tokens produced per forward pass, averaged across the evaluation suite.**

That number is worth remembering more than "1,500 tokens/s," because it measures architectural efficiency directly and doesn't depend on hardware. For AR models this number is exactly 1. Speculative decoding reaches maybe 2–4, depending on draft model hit rate. 20 is an order-of-magnitude difference.

Translated to hardware: **roughly 1,500 output tokens/s on a single NVIDIA H100**, which the report says is substantially faster than AR models even with state-of-the-art speculative decoding.

## 2. The real engineering contribution: not training from scratch

If this report were just "we trained a diffusion language model and it's fast," it would carry much less weight. Diffusion language models have been an active research direction for years without reaching practical use, and the blocker was never really quality — it was **cost and ecosystem**. You'd have to train a brand-new foundation model from scratch, then rebuild every mature capability the AR ecosystem already has: instruction following, tool calling, long context, multimodality. Nobody wants to pay that for an uncertain architecture.

DiffusionGemma's approach sidesteps that entire cost:

**It wasn't trained from scratch. It was fine-tuned from Gemma 4.**

Specifically, the starting point is Gemma 4's mixture-of-experts model at **3.8B activated / 25.2B total parameters**. The whole conversion consumed **under 10% of the original AR model's total training token budget**.

Two stages:

1. **Supervised fine-tuning**, teaching bidirectional denoising — this is the critical capability shift. An AR model only ever looks left-to-right; a diffusion model has to use context on *both* sides of a position to decide what belongs there.
2. **Reinforcement learning plus sampler distillation**, jointly improving generation quality and inference efficiency. Sampler distillation is what teaches the model to reach the same quality in fewer iterations — directly producing that "20 tokens per forward pass" figure.

That "AR model → diffusion model" conversion path matters more than the model itself: it means **diffusion language models no longer need their own foundation model**. In principle any mature AR model could be converted into a high-speed diffusion variant for under 10% incremental cost. The cost structure shifts from "build a new one" to "add a layer on an existing asset."

## 3. What survives the conversion

The usual fear with a new architecture is "it got faster and lost everything else." The report's answer here is fairly counterintuitive — the converted model **keeps most of the starting model's capabilities**:

- **Thinking mode** — retained
- **Multimodal inputs** — retained
- **Long contexts** — retained

And one more interesting result: **it remains capable of AR generation, with only minor performance degradation.**

So this isn't a one-way architectural conversion. The model holds both generation modes at once, and the report points at a direction because of it: **hybrid diffusion-AR decoding**.

That's worth thinking through, because the two modes suit different content:

- **Parallel diffusion decoding** fits content that can be planned as a whole — structured output, code blocks, long text with a predictable shape. Seeing the entire block at once is an advantage there.
- **AR token-by-token decoding** fits content with strict sequential dependency — precise mathematical derivation, logic chains where each step builds on the last. Parallel revision can damage causal structure there.

A model that switches on demand mid-generation isn't trading quality for speed; it's allocating decoding strategy by content type. The report identifies the path without claiming to have implemented it.

## 4. Practical judgment calls

The abstract doesn't give per-benchmark scores — only that it establishes a new Pareto frontier for the speed/capability trade-off, averaged across the full evaluation suite. So what follows is reasoning from architectural properties, not measured results. Real selection decisions need the weights and your own benchmarks.

**Where it's worth attention:**

- **High-throughput batch text processing.** Translation, summarization, classification, data cleaning — predictable output length, steady quality requirements, no extreme reasoning demands. These capture the parallel decoding benefit best.
- **Latency-sensitive interactive applications.** 1,500 tokens/s makes a few-hundred-token reply feel near-instant. For conversational products the perceptual difference is large.
- **Local and edge deployment.** 3.8B activated parameters is a size you can seriously consider on a single GPU or even consumer hardware, and the MoE structure lets you trade memory for total parameter storage.

**Where to be careful:**

- **Strictly sequential reasoning.** Mechanically, parallel refinement means the model can revise already-filled positions within a block — different from AR's commit-and-move-on causal structure. Mathematical derivation and generation that needs a strict state machine require empirical validation.
- **Very short outputs.** The block size is 256 tokens. Generating a few-dozen-token reply doesn't exercise the parallelism and may not beat AR.
- **Existing inference stack compatibility.** vLLM, TensorRT-LLM and friends are optimized entirely around AR plus KV cache. Iterative refinement is a fundamentally different compute pattern, and toolchain support will take time.

One more practical consequence: **observability of generation changes.** AR streaming output is inherently final — a token emitted is a token committed. In a diffusion model, block content mutates during iteration. You either wait for the block to converge before emitting (sacrificing time-to-first-token) or design a new streaming strategy. If you're building product on this, that's an interaction problem to settle early.

## 5. Where this sits

By 2026 the inference competition has shifted from "how big is the model" to "how many milliseconds and how many cents per token." On that axis, DiffusionGemma doesn't offer a stronger model — it offers **a method for moving an existing model's inference efficiency up a tier**, at under 10% incremental training cost.

It's an open-weight, experimental model; the team uses the word "experimental" in the abstract themselves. So the pragmatic expectation is: not something replacing your production model tomorrow, but a demonstration that the path works. If this conversion method holds on other AR foundation models, the implications reach well past the Gemma family.

Independent research is already analyzing its token commitment behavior (["Neither Parallel Nor Sequential: How DiffusionGemma Actually Commits Tokens"](https://arxiv.org/abs/2606.14620)), which suggests real traction in the research community rather than just a launch post. If you want mechanism-level detail, that paper shows more of it than the technical report does.

---

**Sources**

- [DiffusionGemma Technical Report — arXiv:2608.00146](https://arxiv.org/abs/2608.00146)
- [DiffusionGemma Technical Report — Hugging Face Papers](https://huggingface.co/papers/2608.00146)
- [Neither Parallel Nor Sequential: How DiffusionGemma Actually Commits Tokens — arXiv:2606.14620](https://arxiv.org/abs/2606.14620)
