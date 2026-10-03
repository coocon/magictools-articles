# llama.cpp's llama-server or Ollama? Same GGUF on a 24GB Mac mini, Tested: Ollama Runs llama-server Under the Hood

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/llama-server-vs-ollama-mac-mini-24gb-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/llama-server-vs-ollama-mac-mini-24gb-en?utm_source=github&utm_medium=referral)**

## The problem

Running local models on a Mac comes down to a choice early on: use llama.cpp's `llama-server` directly, or install Ollama?

Most comparisons stop at "Ollama is simpler, llama.cpp is more flexible". Once you actually deploy, what decides the outcome is usually not which one is faster but a handful of defaults: how long the context is, what happens when a prompt is too long, how many requests run at once, how much memory gets used. When those defaults differ, the same model behaves completely differently on the two.

So on a Mac mini M4 with 24GB, I had both load **the same GGUF file**, with the same script and the same API (OpenAI-compatible `/v1/chat/completions`, streaming), and compared them item by item: deployment, API, speed, default context, long prompts, concurrency, memory, and how a 27B model behaves on a 24GB machine.

## Analysis

One look at the process list before benchmarking already gave the main answer: **Ollama 0.35.1's inference process is the `llama-server` shipped inside its own package**.

![The child process of ollama serve is bin/ollama/llama-server, launched with -c 4096 -np 1](https://cdn.tools.cooconsbit.com/uploads/articles/2026-10-03-llamacpp-vs-ollama/02-ollama-runner-is-llama-server.png)
*Figure: process list captured by the benchmark script while Ollama was running (paths shortened). Top: qwen3:4b from Ollama's library. Bottom: a Qwen3.8 GGUF I imported with `ollama create`. (Chinese comments: "Ollama 0.35.1 running qwen3:4b (library model)" / "the same Ollama running the imported Qwen3.8 GGUF".)*

The unpacked Ollama directory contains a `llama-server` (`--version` reports `0.5.0-dev (build 1, commit 6f767fe96)`) plus MLX files such as `mlx_metal_v3/v4`. When a GGUF model is loaded, `ollama serve` launches this llama-server as its runner with these key flags:

| Flag Ollama passes | Meaning |
|---|---|
| `-c 4096 -np 1` | 4096-token context, a single slot |
| `--no-jinja --chat-template chatml` | Ollama applies the template itself for library models (absent for imported GGUFs) |
| `--context-shift --keep 4` | when generation fills the context, drop old tokens and keep going |
| `-b 512 -ub 512` | logical batch 512; llama-server's default is `-b 2048 -ub 512` |
| `--no-webui` | turn off llama-server's built-in web UI |

So the real question isn't "which engine is faster". It's **which flags Ollama fills in for the same llama-server, and what management layer it wraps around it**. Almost everything below confirms this.

...

---

**[👉 Continue reading: llama.cpp's llama-server or Ollama? Same GGUF on a 24GB Mac mini, Tested: Ollama Runs llama-server Under the Hood](https://tools.cooconsbit.com/en/articles/llama-server-vs-ollama-mac-mini-24gb-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
