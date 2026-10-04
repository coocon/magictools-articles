# LM Studio 速度慢怎么办：Mac 实测首字等 39 秒的真正原因是 prompt 预填充，不是 GPU 没开满

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/lm-studio-slow-mac?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/lm-studio-slow-mac?utm_source=github&utm_medium=referral)**

> **先回答你搜的问题**
>
> - **先分清是「等很久才出第一个字」还是「出字本身慢」**。LM Studio 的 API 返回里有 `time_to_first_token` 和 `tokens_per_second`，界面上也会显示，看一眼就知道。
> - **等很久才出第一个字**：多半是 prompt 太长（长对话、塞了大文件、或者把 LM Studio 接给了 Claude Code / Cline 这类系统提示上万 token 的工具）。实测 1.3 万 token 的 prompt 要等 **39 秒**才出第一个字。好消息是前缀相同的下一次请求只要 0.1 秒。
> - **出字本身慢**：先看 GPU offload 是不是被调低了。不过在 Mac 上影响没想象中大：全开 29.5 tok/s，全关 26.4 tok/s。
> - **多个工具同时用一个 LM Studio**：如果加载时设了 `--parallel 4`，同时来 4 个请求，每个只剩 9.7 tok/s；设成 1 则排队，每个都是满速。
> - **还是嫌慢**：模型太大就换小一号或更低的量化；Mac 上也可以试同一模型的 MLX 版本（站内 Qwen3.8-27B 实测 MLX 6.4~6.5 tok/s，GGUF 5.9~6.0 tok/s）。

## 问题背景

「LM Studio 很慢」是个很模糊的抱怨，背后至少有两种完全不同的慢：

1. **首字慢**：发出去之后半天没反应，一旦开始出字速度还行；
2. **出字慢**：一个字一个字往外蹦。

网上的建议大多是「把 GPU offload 拉满」，但在 Apple Silicon 上这往往不是主因。这篇在一台 24GB 的 M4 Mac mini 上把几个常见因素逐个量出来。

## 问题分析

一次请求的耗时由两段组成：

- **预填充（prefill）**：模型先把整段 prompt 读一遍，算完才能出第一个字。耗时和 prompt 长度成正比，决定「首字等多久」（`time_to_first_token`）。
- **解码（decode）**：之后一个 token 一个 token 地生成，决定「出字速度」（`tokens_per_second`）。

能影响这两段的设置主要有：GPU offload 比例（`--gpu`）、上下文长度（`-c`）、并发预测数（`--parallel`），以及 LM Studio 会不会复用上一次请求的前缀。LM Studio 的 `lms` 命令行工具把这些都暴露了出来，REST API（`/api/v0/chat/completions`）的返回里直接带 `stats.tokens_per_second` 和 `stats.time_to_first_token`，正好拿来测。

## 技术方案与选型

| 方案 | 结论 | 理由 |
|---|---|---|
| **`lms` 加载 + REST API 读 stats** | ✅ 采用 | 加载参数可以逐项控制；速度数据是 LM Studio 自己统计的 |
| 在聊天界面里手动试 | ❌ 排除 | 参数改动难以记录，没法复现 |
| 下载 MLX 版本做对比 | ❌ 本次不做 | 要再下载约 6GB 模型；MLX 与 llama.cpp 的对比站内已有实测（见相关阅读） |

实验约束：

- 开始前记录 LM Studio 的状态（**没有加载模型、本地服务未开**），结束后恢复成同样的状态；本地服务开在单独的端口 1239。
- 内存守护脚本：系统可用内存低于 35% 时执行 `lms unload --all`（后面触发了一次）。
- 模型 `google/gemma-4-e4b`（7.5B 参数，GGUF Q4_K_M，6.33GB，llama.cpp 引擎 2.13.0）；每次生成 200 个 token，`temperature 0`；每组跑 2 次。

## 实测过程

### 1. 默认设置：4k 上下文，29.5 tok/s

不加任何参数加载：上下文 **4096**，GPU offload 自动（结果与 `--gpu max` 相同），生成速度 **29.62 / 29.46 tok/s**，短 prompt 首字 0.3 秒。

...

---

**[👉 继续阅读全文：LM Studio 速度慢怎么办：Mac 实测首字等 39 秒的真正原因是 prompt 预填充，不是 GPU 没开满](https://tools.cooconsbit.com/zh/articles/lm-studio-slow-mac?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
