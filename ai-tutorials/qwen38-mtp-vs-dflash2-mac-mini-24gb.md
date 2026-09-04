# 24G Mac mini 实测 Qwen3.8 原生 MTP：内存省了 3GB，速度反而降了 24%——与 DFlash 2 同机对照

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/qwen38-mtp-vs-dflash2-mac-mini-24gb?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/qwen38-mtp-vs-dflash2-mac-mini-24gb?utm_source=github&utm_medium=referral)**

## 问题背景

[上一篇 DFlash 2 实测](/zh/articles/qwen38-dflash2-mac-mini-24gb)在这台 24G Mac mini M4 上跑出了 1.8–1.9x 的投机解码加速（6.5 → 11.7–12.2 tok/s），文章结尾留了一条待跟路线：

> Qwen3.8 原生 MTP 头：模型自带训练好的多 token 预测头，llama.cpp 已支持挂载，不需要额外 2B 草稿模型，是 24GB 机器上更抠内存的方案——代价是加速比路线不同，值得单独对照实测一次。

这篇就是那次对照。先把结论放前面，因为它和预期相反：**在这台机器上，原生 MTP 不是"加速比低一点的省内存方案"，而是净减速**——散文生成从 6.0 掉到 4.5 tok/s（-24%），代码生成也没赚（-4%）。省内存倒是真的：峰值 16.0GB，比 DFlash 路线的 19.4GB 少 3.4GB。

同一台机器、同一个提示词、同样的 greedy 400 token 方法，两条投机解码路线一个 1.9x、一个 0.76x。差距在哪，正文拆开讲。

## 问题分析

两条路线的原理同源：都是"便宜地猜几个 token，让大模型一次前向批量验证"。区别在猜的人是谁：

- **DFlash 2**：外挂一个约 2B 参数的独立草稿模型（下载 3.85GB BF16，载入后现场压成 4-bit 约占 1GB 内存），跑在 MLX 后端。
- **原生 MTP**：Qwen3.8 训练时就带了多 token 预测层（`blk.*.nextn.*` 张量），量化 GGUF 里原样保留。llama.cpp 在 [PR #22673](https://github.com/ggml-org/llama.cpp/pull/22673)（2026 年 7 月）加入了 `draft-mtp` 投机解码：不加参数时这些张量加载后被忽略，加一个 flag 就用它当草稿头。

投机解码赚钱的前提是一个近似：**验证 n+1 个 token 的耗时 ≈ 验证 1 个**——权重只读一遍，摊给每一行。这个近似在不同后端上成立的程度天差地别，而它恰恰是本次实测的胜负手。

动手前先查了社区数据。[sudoingX/qwen38-mtp](https://github.com/sudoingX/qwen38-mtp) 收集了 53 份配置的 A/B 实测：CUDA/ROCm 卡上这个 flag 普遍 +33% 到 +145%；但表里唯一一行 Apple M4 24GB（Metal）写着 **5.8 → 5.8，持平**，附带的 [Apple Silicon 深测](https://github.com/sudoingX/qwen38-mtp/blob/master/sweeps/apple-silicon.md)给出了更细的分裂：代码 +9~10%、散文 -22~24%，两头相抵。该仓库把"MLX 与 llama.cpp 同机对照"列为头号未解课题——我这台跑过 DFlash 的机器正好能补上这块。

## 技术方案与选型

**为什么走 llama.cpp 而不是 MLX**：原生 MTP 头的第一方支持在 llama.cpp（PR #22673 已合并进 master），`mlx_lm` 和 dflash CLI 目前都没有挂载 Qwen3.8 nextn 头的现成路径。也就是说这次对照实际是"llama.cpp + 原生 MTP"对"MLX + DFlash 草稿模型"的**整条技术栈对照**，不是单变量实验——所以两条路线各自的纯自回归基线都要单独测，加速比各自算各自的。

**GGUF 选择**：两个来源都能用——[ggml-org/Qwen3.8-27B-GGUF](https://huggingface.co/ggml-org/Qwen3.8-27B-GGUF) 提供独立的 `mtp-*.gguf` 草稿文件（Q4_0 1.68GB，配 `--spec-draft-model` 挂载）；[unsloth/Qwen3.8-27B-GGUF](https://huggingface.co/unsloth/Qwen3.8-27B-GGUF) 的主模型 GGUF **直接内嵌** nextn 张量，一个文件全搞定。我选了 unsloth 的 **UD-Q4_K_S（15.36GB）**：一是内嵌省事，二是体积与上一篇 MLX 4-bit（约 15GB）最接近，内存账好对齐。ggml-org 自家最小量化是 Q4_K_M 18.97GB，24G 机器上偏紧，放弃。

**排除项**：Q8_0（29GB）和 BF16（54GB）装不下，不试；社区已在同款 M4 上实测过 `--spec-draft-n-max` 更深的档位更糟，我只对照 2 和 4 两档验证方向。

## 实测过程

环境：Mac mini M4 基础版 24GB，macOS 26.3.1。llama.cpp 当天 master（b96806d）源码构建：

...

---

**[👉 继续阅读全文：24G Mac mini 实测 Qwen3.8 原生 MTP：内存省了 3GB，速度反而降了 24%——与 DFlash 2 同机对照](https://tools.cooconsbit.com/zh/articles/qwen38-mtp-vs-dflash2-mac-mini-24gb?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
