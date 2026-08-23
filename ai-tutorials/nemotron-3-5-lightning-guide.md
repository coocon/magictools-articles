# Nemotron 3.5 Lightning 上手：30B 参数只激活 3B，单卡本地跑

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/nemotron-3-5-lightning-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/nemotron-3-5-lightning-guide?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：Nemotron 3.5 Lightning 上手：30B 参数只激活 3B，单卡本地跑](https://tools.cooconsbit.com/zh/articles/nemotron-3-5-lightning-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
