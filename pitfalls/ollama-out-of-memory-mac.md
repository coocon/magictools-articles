# Ollama 显存不足怎么办：24GB Mac 实测 num_ctx、并发、KV 量化各吃多少内存，以及它不会拦你的那个坑

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/ollama-out-of-memory-mac?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/ollama-out-of-memory-mac?utm_source=github&utm_medium=referral)**

> **先回答你搜的问题**
>
> - **先看 `ollama ps`**：`PROCESSOR` 列显示 `100% GPU` 就没有「显存不足」；显示 `48%/52% CPU/GPU` 这类，说明放不下，有一部分在 CPU 上跑。`SIZE` 列是这个模型实际占的内存。
> - **最常见的原因是上下文开太大**：qwen3:4b 在 4k 上下文时占 3.73GB，32k 时占 9.89GB，大头是 KV cache，不是模型本身。
> - **最有效的一招**：启动 Ollama 前设 `OLLAMA_FLASH_ATTENTION=1` 和 `OLLAMA_KV_CACHE_TYPE=q8_0`（或 `q4_0`）。实测 32k 上下文从 9.89GB 降到 5.51GB（q8_0）/ 4.31GB（q4_0），生成速度不变。
> - **检查并发数**：`OLLAMA_NUM_PARALLEL=4` 会让内存按 4 倍上下文算，但 `ollama ps` 的 `CONTEXT` 列看不出来。不需要并发就设成 1。
> - **别指望 Ollama 拦你**：上下文设得远超内存时，它不会报「内存不够」，而是直接开始分配，系统会被拖进疯狂换页。设之前先算一下。
> - **Mac 上「挪到 CPU」没那么可怕**：实测 qwen3:4b 全部放 CPU 只比全 GPU 慢 25%。

## 问题背景

本地跑大模型，「显存不足」几乎是绕不过去的一关。在 Mac 上它有两个特殊之处：

1. Apple Silicon 是统一内存，没有单独的「显存」，但 Ollama 会按 Metal 报告的可用量当显存算；
2. 你看到的往往不是一句报错，而是变慢、卡顿，或者整台机器开始转风扇、换页。

所以这篇不讲概念，直接在一台 24GB 的 M4 Mac mini 上量：每个设置到底吃多少内存，以及超过之后会发生什么。

## 问题分析

先看 Ollama 怎么理解这台机器。启动日志原文：

```
msg="inference compute" library=Metal name=Metal description="Apple M4" total="17.8 GiB" available="17.8 GiB"
msg="vram-based default context" total_vram="17.8 GiB" default_num_ctx=4096
```

**24GB 内存的 Mac，Ollama 认到的显存是 17.8 GiB**。按 [官方文档](https://github.com/ollama/ollama/blob/v0.19.0/docs/context-length.mdx)，默认上下文按显存分档：小于 24 GiB 给 4k，24~48 GiB 给 32k，48 GiB 及以上给 256k。所以这台机器默认只有 **4096**。而同一份文档也写着：*Tasks which require large context like web search, agents, and coding tools should be set to at least 64000 tokens.*

于是大家会去调大上下文，「显存不足」就从这里开始。文档里和内存直接相关的几条：

- 内存需求按 `OLLAMA_NUM_PARALLEL` × `OLLAMA_CONTEXT_LENGTH` 计算（[FAQ](https://github.com/ollama/ollama/blob/v0.19.0/docs/faq.mdx)）
- KV cache 可以量化：`q8_0` 约为 `f16` 的 1/2，`q4_0` 约为 1/4，**前提是开 Flash Attention**
- `ollama ps` 的 `PROCESSOR` 列：`100% GPU` / `100% CPU` / `48%/52% CPU/GPU`

## 技术方案与选型

| 方案 | 结论 | 理由 |
|---|---|---|
| **单独起一个 Ollama 实例逐项测** | ✅ 采用 | 在 `127.0.0.1:11500` 起独立实例，只读复用本机已下载的模型，每测一项就卸载，`/api/ps` 读占用、`/api/generate` 读速度 |
| 用 27B 模型直接撞内存上限 | ❌ 放弃 | 实验时这台机器上还跑着另一个占 7GB 的回测任务，27B（约 17GB）会把它挤进换页 |
| Docker 里跑 | ❌ 排除 | 官方 FAQ：*GPU acceleration is not available for Docker Desktop in macOS*，测出来全是 CPU 数据 |

...

---

**[👉 继续阅读全文：Ollama 显存不足怎么办：24GB Mac 实测 num_ctx、并发、KV 量化各吃多少内存，以及它不会拦你的那个坑](https://tools.cooconsbit.com/zh/articles/ollama-out-of-memory-mac?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
