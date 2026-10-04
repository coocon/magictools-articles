# Ollama Out of Memory on a Mac: How Much num_ctx, Parallelism and KV Quantization Really Cost on 24GB (Tested) — and the Trap It Won't Stop You From

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/ollama-out-of-memory-mac-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/ollama-out-of-memory-mac-en?utm_source=github&utm_medium=referral)**

> **Short answers first**
>
> - **Start with `ollama ps`**: `100% GPU` under `PROCESSOR` means it fits; something like `48%/52% CPU/GPU` means part runs on the CPU. `SIZE` is the real memory used.
> - **The usual cause is too much context**: qwen3:4b uses 3.73GB at 4k context and 9.89GB at 32k. Most of that is KV cache, not the model.
> - **The most effective fix**: set `OLLAMA_FLASH_ATTENTION=1` and `OLLAMA_KV_CACHE_TYPE=q8_0` (or `q4_0`) before starting Ollama. Measured: 32k context went from 9.89GB to 5.51GB (q8_0) / 4.31GB (q4_0) at the same speed.
> - **Check parallelism**: `OLLAMA_NUM_PARALLEL=4` sizes memory for 4× the context, but the `CONTEXT` column in `ollama ps` doesn't show it. Set it to 1 if you don't need concurrency.
> - **Don't count on Ollama to stop you**: with a context far larger than your RAM it doesn't say "not enough memory" — it starts allocating and drags the system into swapping. Do the math first.
> - **On a Mac, "offloaded to CPU" isn't a disaster**: qwen3:4b fully on CPU was only 25% slower than fully on GPU.

## Background

Running models locally, "out of memory" is almost unavoidable. On a Mac it has two twists:

1. Apple Silicon has unified memory with no separate VRAM, but Ollama treats whatever Metal reports as VRAM;
2. You often don't get a clean error — you get slowness, stalls, or the whole machine swapping.

So instead of theory, this article measures on a 24GB M4 Mac mini what each setting costs and what happens past the limit.

## How Ollama sees this machine

From the startup log:

```
msg="inference compute" library=Metal name=Metal description="Apple M4" total="17.8 GiB" available="17.8 GiB"
msg="vram-based default context" total_vram="17.8 GiB" default_num_ctx=4096
```

**On a 24GB Mac, Ollama sees 17.8 GiB of VRAM.** Per the [docs](https://github.com/ollama/ollama/blob/v0.19.0/docs/context-length.mdx), the default context depends on VRAM: under 24 GiB gets 4k, 24–48 GiB gets 32k, 48 GiB and up gets 256k. So this machine defaults to **4096**, while the same page says *Tasks which require large context like web search, agents, and coding tools should be set to at least 64000 tokens.*

So people raise the context, and that's where out-of-memory starts. Relevant doc points:

- Memory scales with `OLLAMA_NUM_PARALLEL` × `OLLAMA_CONTEXT_LENGTH` ([FAQ](https://github.com/ollama/ollama/blob/v0.19.0/docs/faq.mdx))
- The KV cache can be quantized: `q8_0` ≈ 1/2 of `f16`, `q4_0` ≈ 1/4, **only with Flash Attention on**
- `ollama ps` `PROCESSOR`: `100% GPU` / `100% CPU` / `48%/52% CPU/GPU`

...

---

**[👉 Continue reading: Ollama Out of Memory on a Mac: How Much num_ctx, Parallelism and KV Quantization Really Cost on 24GB (Tested) — and the Trap It Won't Stop You From](https://tools.cooconsbit.com/en/articles/ollama-out-of-memory-mac-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
