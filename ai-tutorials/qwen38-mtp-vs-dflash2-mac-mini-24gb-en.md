# Qwen3.8 Native MTP on a 24GB Mac mini: 3GB Less Memory, 24% Slower — Head-to-Head With DFlash 2

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/qwen38-mtp-vs-dflash2-mac-mini-24gb-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/qwen38-mtp-vs-dflash2-mac-mini-24gb-en?utm_source=github&utm_medium=referral)**

## Background

[My DFlash 2 write-up](/en/articles/qwen38-dflash2-mac-mini-24gb-en) got speculative decoding running on this 24GB Mac mini M4 at 1.8–1.9x (6.5 → 11.7–12.2 tok/s), and ended with a promised follow-up:

> Qwen3.8's native MTP head: the model ships with a trained multi-token-prediction head, llama.cpp already supports mounting it, no extra 2B draft model needed — the more memory-frugal option on a 24GB machine. The trade-off is a different speedup profile, worth a separate head-to-head.

This is that head-to-head, and the headline is the opposite of what I expected: **on this machine, native MTP is not "a bit less speedup for less memory" — it is a net slowdown.** Prose generation drops from 6.0 to 4.5 tok/s (-24%), and code generation doesn't pay either (-4%). The memory saving is real: 16.0GB peak versus 19.4GB on the DFlash route, 3.4GB less.

Same machine, same prompt, same greedy-400-token method. Two speculative decoding routes: one 1.9x, the other 0.76x. The gap has a specific, measurable cause.

## The analysis

Both routes share the same principle: guess a few tokens cheaply, then have the big model verify the whole block in one forward pass. The difference is who guesses:

- **DFlash 2**: an external ~2B-parameter draft model (a 3.85GB BF16 download, quantized to 4-bit at load, ~1GB resident), running on the MLX backend.
- **Native MTP**: Qwen3.8 was trained with multi-token-prediction layers (`blk.*.nextn.*` tensors), and quantized GGUFs keep them. llama.cpp added `draft-mtp` speculative decoding in [PR #22673](https://github.com/ggml-org/llama.cpp/pull/22673) (July 2026): without the flag those tensors load and sit idle; with it, they become the draft head.

Speculative decoding only pays under one approximation: **verifying n+1 tokens costs about the same as verifying 1** — the weights are read once and shared across rows. How well that holds varies wildly by backend, and it turns out to be the whole story here.

Before touching anything I checked community data. [sudoingX/qwen38-mtp](https://github.com/sudoingX/qwen38-mtp) collects 53 A/B configs: on CUDA/ROCm cards the flag is worth +33% to +145%. The single Apple M4 24GB (Metal) row reads **5.8 → 5.8, a wash**, and the accompanying [Apple Silicon deep-dive](https://github.com/sudoingX/qwen38-mtp/blob/master/sweeps/apple-silicon.md) shows the split underneath: code +9–10%, prose -22–24%, cancelling out. That repo lists "MLX versus llama.cpp on the same Mac" as its top open thread — and my machine, having already produced the DFlash numbers, is positioned to answer it.

...

---

**[👉 Continue reading: Qwen3.8 Native MTP on a 24GB Mac mini: 3GB Less Memory, 24% Slower — Head-to-Head With DFlash 2](https://tools.cooconsbit.com/en/articles/qwen38-mtp-vs-dflash2-mac-mini-24gb-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
