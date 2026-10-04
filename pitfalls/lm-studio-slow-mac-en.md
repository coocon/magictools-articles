# LM Studio Slow on a Mac? Tested: the 39-Second Wait Is Prompt Prefill, Not GPU Offload

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/lm-studio-slow-mac-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/lm-studio-slow-mac-en?utm_source=github&utm_medium=referral)**

> **Short answers first**
>
> - **First decide: a long wait for the first token, or slow output?** LM Studio's API returns `time_to_first_token` and `tokens_per_second`, and the UI shows them too.
> - **Long wait for the first token**: usually a long prompt (long chat, big attached files, or LM Studio behind Claude Code / Cline, whose system prompts run to tens of thousands of tokens). A 13K-token prompt waited **39 seconds** before the first token. The next request with the same prefix took 0.1 s.
> - **Slow output**: check whether GPU offload was turned down — though on a Mac it matters less than you'd think: 29.5 tok/s at max, 26.4 tok/s fully off.
> - **Several tools sharing one LM Studio**: with `--parallel 4`, four simultaneous requests ran at 9.7 tok/s each; with 1 they queue and each runs at full speed.
> - **Still slow**: use a smaller model or lower quantization; on a Mac you can also try the MLX build of the same model (see Related).

## Background

"LM Studio is slow" covers at least two different problems:

1. **Slow start**: nothing for a long time, then reasonable output speed;
2. **Slow output**: tokens trickle out one by one.

The usual advice is "max out GPU offload", but on Apple Silicon that is often not the main cause. This article measures the common factors one at a time on a 24GB M4 Mac mini.

## Two phases

A request has two parts:

- **Prefill**: the model reads the whole prompt before the first token; time grows with prompt length — this is `time_to_first_token`.
- **Decode**: tokens are then generated one at a time — this is `tokens_per_second`.

Settings that affect them: GPU offload (`--gpu`), context length (`-c`), parallel predictions (`--parallel`), and whether LM Studio reuses the previous request's prefix. The `lms` CLI exposes all of these, and the REST API (`/api/v0/chat/completions`) returns `stats.tokens_per_second` and `stats.time_to_first_token`, which is what we measured.

## Method

| Approach | Verdict | Why |
|---|---|---|
| **Load with `lms`, read stats from the REST API** | ✅ used | Load options controlled one by one; speed figures are LM Studio's own |
| Try things in the chat UI | ❌ | Hard to record and reproduce |
| Download an MLX build to compare | ❌ not this time | Another ~6GB download; an MLX vs llama.cpp comparison already exists on this site (see Related) |

Constraints:

- LM Studio's state was recorded first (**no model loaded, server off**) and restored at the end; the server ran on a separate port, 1239.
- A memory guard ran `lms unload --all` if free memory fell below 35% (it fired once).
- Model `google/gemma-4-e4b` (7.5B params, GGUF Q4_K_M, 6.33GB, llama.cpp engine 2.13.0); 200 generated tokens per request, `temperature 0`; each case run twice.

...

---

**[👉 Continue reading: LM Studio Slow on a Mac? Tested: the 39-Second Wait Is Prompt Prefill, Not GPU Offload](https://tools.cooconsbit.com/en/articles/lm-studio-slow-mac-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
