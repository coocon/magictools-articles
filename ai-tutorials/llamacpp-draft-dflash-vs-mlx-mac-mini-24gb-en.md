# Same 24GB Mac mini, Same DFlash 2: 1.9x Faster on MLX, 50% Slower (and OOM) on llama.cpp

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/llamacpp-draft-dflash-vs-mlx-mac-mini-24gb-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/llamacpp-draft-dflash-vs-mlx-mac-mini-24gb-en?utm_source=github&utm_medium=referral)**

## Background

This is the third speculative-decoding benchmark on the same base Mac mini M4 with 24GB. Previously:

- [DFlash 2 on MLX](/en/articles/qwen38-dflash2-mac-mini-24gb-en): external 2B draft model, 6.5 → 11.7–12.2 tok/s, a stable **1.8–1.9x**;
- [Qwen3.8 native MTP on llama.cpp](/en/articles/qwen38-mtp-vs-dflash2-mac-mini-24gb-en): a **net slowdown** (prose -24%), root-caused with llama-batched-bench to Metal's batched-decode amortization being only 1.13x.

The MTP article left an obvious open question. llama.cpp merged DFlash 2 support in [PR #27342](https://github.com/ggml-org/llama.cpp/pull/27342) (`--spec-type draft-dflash`), and z-lab published official [GGUF draft models](https://huggingface.co/z-lab/Qwen3.8-27B-DFlash2-GGUF) for it. **Same DFlash 2 algorithm, but on the llama.cpp stack — does it replicate MLX's 1.9x, or repeat MTP's net loss?**

The experiment discriminates between two hypotheses, and only one can survive. If MTP lost because of draft quality, then DFlash 2 — a much stronger drafter (official CUDA eval: acceptance length 5.1–5.4) — should win. If MTP lost to the backend tax of unamortized batch verification on Metal, no drafter can help.

Spoiler: **the backend tax won, and by more than expected**. The README-recommended `--spec-draft-n-max 7` hits a reproducible Metal OOM on this machine. The only configuration that runs stably, n-max 3, drops prose from 6.0 to 3.0 tok/s (-50%). On the code prompt the acceptance rate reaches 83.8% — 3.5 tokens landed per step — and it *still* loses 23%. On the same machine, MLX delivers 1.8–1.9x.

## Analysis

Three articles in, the speculative-decoding landscape on this machine is a 2×2 grid:

| | MLX stack | llama.cpp stack (Metal) |
|---|---|---|
| **DFlash 2 external draft** | **1.8–1.9x** (part one) | **This article: 0.5x, recommended config OOMs** |
| **Native MTP head** | no mounting path available | 0.76x (part two) |

Part two already quantified llama.cpp Metal's core problem: batch verification doesn't amortize. `llama-batched-bench` measured batch-8 aggregate throughput at 6.90 tok/s versus 6.10 single-stream — an amortization ratio of just **1.13x** (the same scenario on CUDA gives 3.34x). Speculative decoding's profit formula assumes "verifying n draft tokens costs about the same as verifying one." On this backend, that assumption is false.

So the pre-test prediction was: DFlash 2's draft quality must still pass through the verification toll booth. The measured bill turned out to have a second line item — **drafting itself is startlingly expensive on this stack**. Numbers below.

...

---

**[👉 Continue reading: Same 24GB Mac mini, Same DFlash 2: 1.9x Faster on MLX, 50% Slower (and OOM) on llama.cpp](https://tools.cooconsbit.com/en/articles/llamacpp-draft-dflash-vs-mlx-mac-mini-24gb-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
