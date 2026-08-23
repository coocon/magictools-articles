# DiffusionGemma 不是画图的：每秒 1500 token 的扩散语言模型

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/diffusiongemma-diffusion-language-model?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/diffusiongemma-diffusion-language-model?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：DiffusionGemma 不是画图的：每秒 1500 token 的扩散语言模型](https://tools.cooconsbit.com/zh/articles/diffusiongemma-diffusion-language-model?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
