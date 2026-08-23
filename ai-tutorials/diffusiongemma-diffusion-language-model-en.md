# DiffusionGemma Doesn't Make Pictures: A 1,500 tokens/s Diffusion Language Model

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/diffusiongemma-diffusion-language-model-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/diffusiongemma-diffusion-language-model-en?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 Continue reading: DiffusionGemma Doesn't Make Pictures: A 1,500 tokens/s Diffusion Language Model](https://tools.cooconsbit.com/en/articles/diffusiongemma-diffusion-language-model-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
