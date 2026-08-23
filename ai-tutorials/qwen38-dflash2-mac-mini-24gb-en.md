# Qwen3.8-27B + DFlash 2 on a 24GB Mac mini: a Measured 1.8x, and Why You Won't Get the Official 3x

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/qwen38-dflash2-mac-mini-24gb-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/qwen38-dflash2-mac-mini-24gb-en?utm_source=github&utm_medium=referral)**

August 2026 has been a good month for local-model people: Qwen3.8-27B's open weights landed on August 14, and on August 18 [Inco AI released DFlash 2](https://inco.ai/blog/dflash2/) along with a drafter checkpoint for that exact model. The headline number is striking: **2.7–3.4x** the throughput of autoregressive decoding on SGLang at batch size 1.

That number comes from data-center GPUs. What I have is a Mac mini with 24GB of RAM. This post answers three questions: **does it run? how much faster is it really? and where does the gap come from?**

The short version: it runs, at a stable **1.8–1.9x** (6.5 → 11.7–12.2 tok/s), across three controlled benchmark rounds with under 5% variance. That takes a 27B model from "barely tolerable" to "comfortable reading speed" — these two August releases genuinely changed what this machine is.

## The test machine (full disclosure)

| Item | Spec |
|---|---|
| Model | Mac mini (Mac16,10), 2024 base-tier |
| Chip | Apple M4 (base — not Pro, not Max) |
| CPU | 10 cores (4 performance + 6 efficiency) |
| GPU | 10 cores, Metal 4 |
| Memory | 24GB unified (base M4 is rated at 120GB/s bandwidth) |
| OS | macOS 26.3.1 |

The "base M4" part matters: local LLM decoding is memory-bandwidth-bound, so an M4 Pro (273GB/s) or M4 Max will post much higher absolute speeds. But the **speedup ratio** — the subject of this post — is mostly determined by the algorithm and quantization setup, so it should transfer.

## Why MLX, not vLLM or SGLang

DFlash 2 officially runs on four stacks: SGLang, vLLM, llama.cpp (still a PR), and **MLX**. The first two are CUDA-only — a non-starter on a Mac. But the official [z-lab/dflash](https://github.com/z-lab/dflash) repo ships an MLX backend for Apple Silicon, and its README explicitly lists DFlash 2 support for Qwen3.8-27B on MLX. This is a first-party path, not a community hack.

## Deployment: four commands

```bash
# 1. Environment (Python 3.12 — 3.14 is too fresh for the MLX ecosystem)
mkdir -p ~/llm/dflash && cd ~/llm/dflash
uv venv --python 3.12 .venv
uv pip install --python .venv/bin/python "dflash[local]"

# 2. Models (~19GB total download)
.venv/bin/hf download mlx-community/Qwen3.8-27B-4bit    # target, 15GB
.venv/bin/hf download z-lab/Qwen3.8-27B-DFlash2         # drafter, 3.85GB

# 3. Confirm MLX is on the GPU
.venv/bin/python -c "import mlx.core as mx; print(mx.default_device())"
# expected: Device(gpu, 0)

# 4. Generate
.venv/bin/dflash generate mlx \
    --model mlx-community/Qwen3.8-27B-4bit \
    --draft z-lab/Qwen3.8-27B-DFlash2 \
    --draft-bits 4 --block-size 5 --reasoning low \
    "your prompt here"
```

...

---

**[👉 Continue reading: Qwen3.8-27B + DFlash 2 on a 24GB Mac mini: a Measured 1.8x, and Why You Won't Get the Official 3x](https://tools.cooconsbit.com/en/articles/qwen38-dflash2-mac-mini-24gb-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
