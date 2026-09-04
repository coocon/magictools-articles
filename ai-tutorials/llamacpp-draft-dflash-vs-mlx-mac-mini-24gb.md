# 同一台 24G Mac mini、同一个 DFlash 2：MLX 上加速 1.9x，llama.cpp 上减速 50% 还 OOM

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/llamacpp-draft-dflash-vs-mlx-mac-mini-24gb?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/llamacpp-draft-dflash-vs-mlx-mac-mini-24gb?utm_source=github&utm_medium=referral)**

## 问题背景

这是这台 24G Mac mini M4 上投机解码对照的第三篇。前情两篇：

- [DFlash 2 实测](/zh/articles/qwen38-dflash2-mac-mini-24gb)：MLX 栈外挂 2B 草稿模型，6.5 → 11.7–12.2 tok/s，稳定 **1.8–1.9x**；
- [Qwen3.8 原生 MTP 实测](/zh/articles/qwen38-mtp-vs-dflash2-mac-mini-24gb)：llama.cpp 栈挂模型自带的 MTP 头，结果**净减速**（散文 -24%），根因是 llama-batched-bench 实测出的 Metal 批量解码摊薄只有 1.13x。

MTP 那篇留了个自然的问题：llama.cpp 在 [PR #27342](https://github.com/ggml-org/llama.cpp/pull/27342) 里也合并了 DFlash 2 支持（`--spec-type draft-dflash`），z-lab 官方还发布了配套的 [GGUF 草稿模型](https://huggingface.co/z-lab/Qwen3.8-27B-DFlash2-GGUF)。**同一个 DFlash 2 算法，换到 llama.cpp 栈上跑，是复刻 MLX 的 1.9x，还是重蹈 MTP 的净减速？**

这个问题值得测，因为两个假设只能活一个：如果 MTP 输在"草稿质量"，那 DFlash 2 这个更强的草稿器（官方 CUDA 评测接受长度 5.1–5.4）应该能赢；如果输在"Metal 验证批量不摊薄"这个后端税，那换什么草稿器都没用。

先说结论：**后端税赢了，而且比预想的更狠**。官方 README 推荐的 `--spec-draft-n-max 7` 在这台机器上直接 Metal OOM（可复现）；能稳定跑完的 n-max 3 散文从 6.0 掉到 3.0 tok/s（-50%）；代码提示词接受率高达 83.8%、平均一步接住 3.5 个 token——依然亏 23%。同一台机器上 MLX 栈是 1.8–1.9x。

## 问题分析

三篇对照下来，这台机器上的投机解码格局是一张 2×2 的表：

| | MLX 栈 | llama.cpp 栈（Metal） |
|---|---|---|
| **DFlash 2 外挂草稿** | **1.8–1.9x**（上上篇） | **本篇：0.5x，官方推荐配置 OOM** |
| **原生 MTP 头** | 无现成挂载路径 | 0.76x（上篇） |

上篇已经量化了 llama.cpp Metal 的病根：批量验证不摊薄。`llama-batched-bench` 实测 batch 8 聚合吞吐 6.90 tok/s，对比单流 6.10，摊薄比只有 **1.13x**（CUDA 同场景是 3.34x）。投机解码的收益公式里，"验证 n 个草稿 ≈ 验证 1 个 token"这个近似在这个后端上不成立。

所以动手前的预判是：DFlash 2 的草稿质量再好，也要过验证这道税关。但实测出来的账比"验证税"还多一层——**草稿本身在这条栈上也贵得离谱**，后面用数字算。

## 技术方案与选型

**目标模型**：继续用 unsloth **UD-Q4_K_S（15.36GB）**，和上篇 MTP 实测完全同一个文件、同一套 server 参数（`-c 8192 -ngl 999 -fa on -b 512 -ub 512`）。这样昨天的 baseline 直接可比，今天实测 baseline 也确实精确复现了（6.00 vs 6.00 tok/s）。

**草稿模型**：z-lab 官方 GGUF 三个量化里选 **Q4_K_M（1.14GB）**——一是对齐 MLX 实测用的 `--draft-bits 4`，二是官方自己的评测里 Q4_K_M 接受长度（5.39）反而比 BF16（5.28）还高，三是上上篇复测已经证明 8-bit 草稿在这台机器上不如 4-bit。排除 Q8_0（2.06GB）和 BF16（3.86GB）：24G 机器上主模型加草稿已经贴着 Metal 工作集上限（后面 OOM 一节就是证据），没有理由用更大的。

**对照臂**：baseline / n-max 7（官方 README 推荐值）/ n-max 3（llama.cpp 默认值）/ n-max 12（探上限）。方法继续用净速度法：greedy（temperature 0）、固定 400 token、读 `timings.predicted_per_second`，散文和代码两种提示词各两轮。

...

---

**[👉 继续阅读全文：同一台 24G Mac mini、同一个 DFlash 2：MLX 上加速 1.9x，llama.cpp 上减速 50% 还 OOM](https://tools.cooconsbit.com/zh/articles/llamacpp-draft-dflash-vs-mlx-mac-mini-24gb?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
