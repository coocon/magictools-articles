---
title: "DiffusionGemma 不是画图的：每秒 1500 token 的扩散语言模型"
slug: diffusiongemma-diffusion-language-model
summary: "看到 Diffusion 就想到画图，是这篇技术报告最容易被误读的地方。DiffusionGemma 生成的是文本——它一次并行精修 256 个 token 的块，单张 H100 上跑到约 1500 tokens/s。更值得注意的是它怎么来的：不是从零训练，而是拿 Gemma 4 的 MoE 模型微调，只花了原模型不到 10% 的训练 token 预算。"
category: ai-tutorials
tags: [DiffusionGemma, 扩散模型, 大模型, Gemma, 推理加速, Google, 开放权重]
coverImage: ""
status: published
locale: zh
source: authored
translationSlug: diffusiongemma-diffusion-language-model-en
---

# DiffusionGemma 不是画图的：每秒 1500 token 的扩散语言模型

> "Rather than decoding one token at a time, DiffusionGemma iteratively refines blocks of 256 tokens in parallel, avoiding the sequential decoding bottleneck of conventional autoregressive large language models."
> —— DiffusionGemma Technical Report

---

先把最容易搞错的一点讲清楚：**DiffusionGemma 生成的是文本，不是图像。**

这个误会太容易发生了。过去几年，"diffusion"这个词在公众语境里几乎和图像生成绑定——Stable Diffusion、Midjourney、各种文生图模型。所以看到 Google 发了个叫 DiffusionGemma 的东西，第一反应是"Gemma 家族出图像模型了"，非常自然，也完全错误。

这篇技术报告（[arXiv:2608.00146](https://arxiv.org/abs/2608.00146)，2026 年 7 月 31 日提交，DiffusionGemma 团队 43 位作者署名）讲的是**离散扩散（discrete diffusion）语言模型**：把扩散这套"从噪声里反复去噪、逐步逼近目标"的思路，用在 token 序列上，而不是像素上。

分类标签也说明了这一点——论文归在 cs.CL（计算语言学）和 cs.AI 下，不是 cs.CV。

## 一、它到底在解决什么问题

今天所有主流大模型都是自回归（autoregressive, AR）的：预测下一个 token，把它接到序列末尾，再预测下一个。这个循环有个无法回避的性质——**它是严格串行的**。生成 1000 个 token 就要走 1000 次前向传播，第 500 个 token 必须等第 499 个出来才能算。

这就是推理速度的根本瓶颈。它不是算力不够，是结构上不允许并行。这些年绕这个瓶颈的努力都很聪明但也都是补丁：KV cache 省掉重复计算、speculative decoding 用小模型猜一批再用大模型批量验证、各种 batching 策略提高吞吐但不改善单请求延迟。

扩散语言模型走的是另一条路：**不逐个生成，而是一次处理一整块，反复精修。**

DiffusionGemma 的做法是一次并行精修 **256 个 token 的块**。每一次前向传播不是产出 1 个 token，而是让这 256 个位置整体向"更像正确答案"的方向移动一步。迭代若干次之后，整块定稿。

摘要里给的关键数字是：**平均每次前向传播产出约 20 个 token**。

这个数字比"1500 tokens/s"更值得记住，因为它是架构效率的直接度量，不受硬件影响。AR 模型这个数字恒等于 1，speculative decoding 能做到 2-4 左右（取决于草稿模型的命中率）。20 是一个数量级上的差别。

落到硬件上的结果：**单张 NVIDIA H100 上约 1500 output tokens/s**，报告称这显著快于配备了最先进 speculative decoding 的 AR 模型。

## 二、真正的工程贡献：不从零训练

如果这篇报告只是"我们训了个扩散语言模型，很快"，它的分量会小很多。扩散语言模型这个方向学术界研究好几年了，一直没进入实用，核心障碍不是效果，是**成本和生态**：你要从零训一个全新架构的基座模型，还要把 AR 生态里那些成熟的能力（指令跟随、工具调用、长上下文、多模态）全部重新做一遍。没人愿意为一个不确定的架构付这个价。

DiffusionGemma 的解法绕过了这一整块成本：

**它不是从零训的，是拿 Gemma 4 微调出来的。**

具体来说，起点是 Gemma 4 的混合专家（MoE）模型，**38 亿激活参数 / 252 亿总参数**。整个转换流程用掉的训练 token，**不到原 AR 模型总预算的 10%**。

两个阶段：

1. **监督微调（SFT）**，教模型做双向去噪——这是关键的能力转换。AR 模型只会从左往右看，扩散模型必须能同时利用一个位置左右两边的上下文来判断这里该填什么。
2. **强化学习 + 采样器蒸馏（sampler distillation）**，同时优化生成质量和推理效率。sampler distillation 这一步是让模型学会用更少的迭代步数达到同样质量——直接对应前面那个"每次前向 20 个 token"的指标。

这个"AR 模型 → 扩散模型"的转换路径，含义比模型本身大：它意味着**扩散语言模型不必再自己养一个基座**。任何一个成熟的 AR 模型，理论上都可以用不到 10% 的增量成本转换成一个高速扩散版本。成本结构从"重新造一个"变成了"在现有资产上加一层"。

## 三、转换之后还剩下什么

新架构最怕的是"快了，但别的都没了"。报告在这一点上给的答案相当反直觉——转换后的模型**保留了起点模型的大部分能力**：

- **thinking mode**（推理模式）保留
- **多模态输入**保留
- **长上下文**保留

还有一条更有意思：**它仍然能做 AR 生成，且性能只有轻微退化。**

也就是说这不是一个单向的架构改造，模型同时具备两种生成模式。报告据此提出了一个方向：**混合 diffusion-AR 解码**（hybrid diffusion-AR decoding）。

这个方向值得展开想一下。两种模式的适用场景是错开的：

- 扩散并行解码擅长**能整体规划的内容**——结构化输出、代码块、格式确定的长文本。这些地方"一次看一整块"是优势。
- AR 逐 token 解码擅长**需要严格前后依赖的内容**——精确的数学推导、必须一步扣一步的逻辑链。这些地方并行修改容易破坏因果结构。

如果一个模型能在生成过程中按需切换，那就不是"用速度换质量"的取舍，而是按内容类型分配解码策略。报告只是指出了这条路径，没说已经实现。

## 四、几条使用上的判断

摘要没有给逐项 benchmark 分数，只说"在完整评测套件上平均"建立了速度与能力权衡的新帕累托前沿。所以下面这些是基于架构特性的判断，不是实测结论——真要选型，得等权重到手自己跑。

**什么场景值得关注：**

- **高吞吐的批量文本处理**。翻译、摘要、分类、数据清洗这类任务，输出长度可预期、质量要求稳定但不极端，最能吃到并行解码的收益。
- **延迟敏感的交互式应用**。1500 tokens/s 意味着一段几百 token 的回复接近瞬时。对话产品的体感差距会非常明显。
- **本地/边缘部署**。38 亿激活参数是可以在单卡甚至消费级硬件上认真考虑的规模，MoE 结构又让总参数的存储成本可以用内存换。

**什么场景要谨慎：**

- **严格逐步的推理任务**。并行精修的机制上，模型在块内是可以"反悔"和修改已填位置的，这和 AR 那种"落子无悔"的因果结构不同。数学推导、需要严格状态机的生成，得实测验证。
- **超短输出**。块大小是 256 token，生成一个几十 token 的短回复，并行的优势发挥不出来，可能还不如 AR。
- **现有推理栈的兼容性**。vLLM、TensorRT-LLM 这些成熟推理框架的优化全是围绕 AR + KV cache 建的。扩散模型的迭代精修是完全不同的计算模式，工具链适配需要时间。

最后一条最实际：**generation 的可观测性会变**。AR 模型流式输出天然就是最终结果，token 出来就是定的。扩散模型在迭代过程中，块内内容是会变的——你要么等整块收敛再吐出来（牺牲首字延迟），要么设计一套新的流式策略。做产品的话这是个需要提前想清楚的交互问题。

## 五、这件事放在哪个坐标上看

2026 年推理成本的竞争已经从"模型多大"转向"每 token 多少钱、多少毫秒"。在这个坐标下，DiffusionGemma 提供的不是一个更强的模型，而是**一个把已有模型的推理效率整体挪一个档位的方法**，代价是不到 10% 的增量训练。

它是开放权重（open-weight）的实验性模型，团队自己在摘要里用了 "experimental" 这个词。所以务实的期待是：这不是一个明天就替换你生产模型的东西，而是一个证明了路径可行的样本——如果这套转换方法在其他 AR 基座上同样成立，那影响的就不只是 Gemma 家族。

已经有独立研究在分析它的 token 提交行为（["Neither Parallel Nor Sequential: How DiffusionGemma Actually Commits Tokens"](https://arxiv.org/abs/2606.14620)），说明这个模型在研究社区里已经有实际的关注度，而不只是一篇发布报告。想深入的话，那篇是比技术报告本身更能看出机制细节的材料。

---

**参考来源**

- [DiffusionGemma Technical Report — arXiv:2608.00146](https://arxiv.org/abs/2608.00146)
- [DiffusionGemma Technical Report — Hugging Face Papers](https://huggingface.co/papers/2608.00146)
- [Neither Parallel Nor Sequential: How DiffusionGemma Actually Commits Tokens — arXiv:2606.14620](https://arxiv.org/abs/2606.14620)
