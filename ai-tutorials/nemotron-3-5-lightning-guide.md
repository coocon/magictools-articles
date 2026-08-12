---
title: "Nemotron 3.5 Lightning 上手：30B 参数只激活 3B，单卡本地跑"
slug: nemotron-3-5-lightning-guide
category: ai-tutorials
locale: zh
source: authored
translationSlug: nemotron-3-5-lightning-guide-en
tags: [nvidia, nemotron, llm, local-inference, open-weights]
summary: "Nvidia Nemotron 3.5 Lightning 关键参数逐条核实：30B 总参 3B 激活、1M 上下文、NVFP4 量化几乎无损、OpenMDW 许可可商用。本文讲清它能跑在哪些卡上、需要多少显存、以及三种上手方式。"
status: published
---

2026 年 8 月 11 日，Nvidia 发布了 **Nemotron 3.5 Lightning**——Nemotron 3 开源模型家族的新成员，定位是「长期运行的 Agent 工作流里的轻量主力」。同场还开源了模型路由库 **NeMo Switchyard**。

这篇文章不复述新闻稿。下面所有参数都逐条核对过 [Hugging Face 模型卡](https://huggingface.co/nvidia/NVIDIA-Nemotron-3.5-Lightning-30B-A3B-NVFP4)和 [Nvidia 官方博客](https://blogs.nvidia.com/blog/nemotron-lightning-switchyard-rtx-dgx/)，重点回答三个问题：**它是什么、你的卡能不能跑、怎么最快试到**。

## 关键参数速览（全部来自官方模型卡）

| 项目 | 值 |
|------|-----|
| 总参数 / 激活参数 | 30B / **3B**（MoE） |
| 架构 | Mamba-2 + MoE + Attention 混合 |
| 上下文长度 | 最长 **1M tokens** |
| 官方量化 | NVFP4（4-bit，另有 BF16 版） |
| 单卡部署 | 1× DGX Spark（GB10）或 1× H100 |
| 支持硬件 | Blackwell（DGX Spark、GB200、**GeForce RTX 5090**）、Hopper（H100/H200）、Ampere（经 W4A16） |
| 许可证 | OpenMDW-1.1，**可商用** |
| 语言 | 英语 + 西/法/德/意/日（中文不在官方支持列表） |
| 推荐采样参数 | temperature 1.0，top_p 0.95 |
| 数据截止 | 预训练 2025-09，后训练 2026-05 |

两个容易被新闻稿带偏的点，先纠正：

1. **官方硬件列表里没有 RTX 40 系（Ada 架构）**。列表只写了 Blackwell、Hopper、Ampere（W4A16 路径）。4090 用户别看到「RTX 可跑」就直接冲，等社区实测或 GGUF 转换。
2. 它的强项是**低激活量带来的吞吐和显存优势**，不是 benchmark 全面碾压——官方自己的 HLE 分数只有 10.47，定位就是「Agent 系统里的高频便宜劳动力」，复杂规划还是要交给大模型。

## NVFP4 量化掉了多少精度？答案是几乎没掉

官方在同一评测框架（NeMo Gym / NeMo Evaluator）下给出了 BF16 与 NVFP4 的对照，摘几行关键的：

| Benchmark | BF16 | NVFP4 |
|-----------|------|-------|
| MMLU Pro | 81.94 | 81.62 |
| GPQA Diamond | 75.44 | 75.57 |
| SWE-bench Verified | 51.56 | **52.80** |
| Terminal-Bench 2.1 | 24.58 | 23.46 |
| IFBench (loose) | 71.88 | 72.88 |

SWE-bench Verified 上量化版反而更高（52.80 vs 51.56）。这组数字的实际含义是：**直接用 NVFP4 官方 checkpoint 就行，不用纠结精度损失**。

## 显存要多少？

官方没给显存数字，但可以算权重体积：30B 参数 × 4 bit ≈ **15GB 权重**，加上 KV cache 和运行时开销，24-32GB 显存档位是合理起点——这正好落在 RTX 5090（32GB）的官方支持范围内。1M 上下文跑满是另一回事，KV cache 会随上下文线性膨胀，官方 vLLM 配方里也默认开了 `--kv-cache-dtype fp8`。

想按自己的卡和上下文长度精确估算，可以用我们的 [LLM 显存计算器](https://tools.cooconsbit.com/tools/llm-vram-calculator)。

## 三种上手方式，按成本从低到高

**1. 零成本在线试（不装任何东西）**

- [build.nvidia.com](https://build.nvidia.com/nvidia/nemotron-3.5-lightning-30b-a3b) 官方 playground 直接聊
- [OpenRouter 有 free 档位](https://openrouter.ai/nvidia/nemotron-3.5-lightning:free)（`nvidia/nemotron-3.5-lightning:free`），可以直接接 API 测

**2. vLLM 本地部署（官方配方）**

模型卡给出的 DGX Spark 配方，要点是 marlin MoE backend + fp8 KV cache + DSpark 投机解码：

```shell
export MODEL_CKPT=nvidia/NVIDIA-Nemotron-3.5-Lightning-30B-A3B-NVFP4
export DSPARK_CKPT=nvidia/NVIDIA-Nemotron-3.5-Lightning-30B-A3B-NVFP4-DSpark

vllm serve --model $MODEL_CKPT \
  --moe-backend marlin \
  --kv-cache-dtype fp8 \
  --enable-prefix-caching \
  --speculative_config.num_speculative_tokens 3
```

（vLLM 需要较新版本，官方标注的是 `vllm/vllm-openai:v0.27.1`。）

**3. NIM 微服务 / 云上**

企业路径走 NVIDIA NIM，HF、ModelScope 也都有权重镜像。

## 顺带发布的 NeMo Switchyard 值得多看一眼

Switchyard 是个[开源模型路由库](https://github.com/NVIDIA-NeMo/Switchyard)：把 Agent 工作流里的每个请求自动路由到「够用的最便宜模型」。合作方给出的数字比较有说服力——LangChain 在 145 个多轮 Deep Agents 任务里只把 7% 的调用发给旗舰模型，成本降 74%（精度损失 6%）；Ramp 在自家 SWE-Bench 上砍掉 58% 成本、33% 运行时长。

这和 Lightning 是配套打法：**路由器把高频简单活分给 3B 激活的便宜模型，难题才上旗舰**。如果你在做多 Agent 系统，这套「系统级省钱」思路比单纯换模型更值得评估。各家模型单价可以在 [LLM API 价格对比](https://tools.cooconsbit.com/tools/llm-pricing)里横向比。

## 常见问题 FAQ

### Nemotron 3.5 Lightning 需要多少显存？

NVFP4 权重约 15GB（30B × 4 bit），加上 KV cache 建议 24GB 以上显存起步。官方给出的单卡参考是 1× DGX Spark（GB10）或 1× H100；消费级卡里 RTX 5090（32GB）在官方支持列表中。

### 可以商用吗？

可以。许可证是 OpenMDW-1.1，模型卡明确标注 "ready for commercial use"。

### RTX 4090 能跑吗？

官方支持列表只包含 Blackwell、Hopper 和 Ampere（W4A16 路径），**没有列出 Ada 架构（RTX 40 系）**。建议等社区实测结果再做决定。

### 支持中文吗？

官方语言列表是英语、西班牙语、法语、德语、意大利语、日语，中文不在其中。代码语言支持 43 种。

### 思考模式可以关掉吗？

可以。推理控制通过 chat-template kwargs 暴露：默认开启 thinking，可切换为直接回答模式。

## 参考链接

- [Nvidia 官方博客：Nemotron 3.5 Lightning 与 NeMo Switchyard 发布](https://blogs.nvidia.com/blog/nemotron-lightning-switchyard-rtx-dgx/)
- [Hugging Face 模型卡（含完整 benchmark 与部署配方）](https://huggingface.co/nvidia/NVIDIA-Nemotron-3.5-Lightning-30B-A3B-NVFP4)
- [NeMo Switchyard（GitHub）](https://github.com/NVIDIA-NeMo/Switchyard)
- [OpenRouter 免费档位](https://openrouter.ai/nvidia/nemotron-3.5-lightning:free)
