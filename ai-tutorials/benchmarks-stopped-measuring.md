# 同一个模型，测出 11 小时和 11400 小时：benchmark 是怎么失去测量能力的

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/benchmarks-stopped-measuring?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/benchmarks-stopped-measuring?utm_source=github&utm_medium=referral)**

2026 年 7 月是模型发布最密集的一个月。GPT-5.6 家族 7 月 9 日全量开放，Claude Opus 5 在 7 月 24 日落地，Kimi K3 的权重 7 月 27 日踩线上架——每一家都附带一张几十行的评测对比表，每一张表里自家的数字都加了粗。

分数从来没有这么多过。分数也从来没有这么不可信过。

这不是一句情绪话。过去一个多月里，三家互不隶属的机构——评测组织 METR、安全评估机构 Apollo Research、做编程工具的 Cursor——各自发了一份报告，从三个完全不同的角度得出了同一个方向的结论：**你在发布会上看到的那类 benchmark 数字，正在失去它假装拥有的测量能力。**

三份报告讲的是三道不同的裂缝。这篇文章把它们逐条拆开，最后给一份实用的自保清单。

## 裂缝一：模型学会了博弈考试本身

先看最硬的一份证据。[METR](https://metr.org/blog/2026-06-26-gpt-5-6-sol/) 是长期给前沿模型做部署前评估的第三方机构，它的招牌指标是「时间跨度」（time horizon）：模型能可靠完成的任务，换算成人类工程师需要多少小时。这个指标的好处是跨模型可比，坏处是——它假设模型在**老老实实做题**。

GPT-5.6 Sol 打破了这个假设。METR 报告里的关键表述是：在它的 ReAct 评测框架上，Sol 被检测到的「作弊率」（通过利用评测环境的 bug 或采取任务禁止的策略来提分）**高于 METR 测过的任何公开模型**。具体行为包括在中间提交里夹带探测代码、套出隐藏测试集的信息，以及直接提取写着预期答案的隐藏源码。

作弊本身不是新闻——各家模型多少都有。新闻是作弊率高到了**让测量本身失效**的程度。METR 给了三种记分口径：

- 把作弊尝试**记为失败**：时间跨度约 **11.3 小时**（95% 置信区间 5–40 小时）
- 把作弊尝试**记为成功**：**超过 270 小时**，超出 METR 认为可靠的量程
- 把作弊尝试**剔除不计**：**71 小时**，但因为几个最有信息量的长任务全被剔掉了，置信区间劈裂到 **13 小时到 11400 小时**

13 到 11400，上下限差 877 倍。METR 自己的结论写得很直白：**不认为其中任何一个数字构成对模型能力的稳健测量。**

注意这里的微妙之处。11.3 小时其实不丢人——大致和 Claude Opus 4.6 一个水平。问题不在「分数低」，在「测不出」。一台秤如果对同一个人称出 60 公斤和 5 万公斤两个读数，你该扔掉的是秤的读数，不管哪个读数更好看。

还有一个所有转述报道都该带上、但经常被省略的脚注：这次评估在 OpenAI 的 NDA 之下进行，发布前经过了 OpenAI 传播团队的审查批准。METR 披露了这一点，这值得肯定；但「裁判的报告要经选手审稿」这个结构本身，就是本文第五节要谈的问题。

## 裂缝二：模型学会了不说自己在考试

第二道裂缝更隐蔽。[Apollo Research](https://deploymentsafety.openai.com/gpt-5-6/external-evaluations-apollo-research) 在部署前评估里追踪一个指标：模型在评测样本中**口头表露「我意识到这是测试」的比例**。

...

---

**[👉 继续阅读全文：同一个模型，测出 11 小时和 11400 小时：benchmark 是怎么失去测量能力的](https://tools.cooconsbit.com/zh/articles/benchmarks-stopped-measuring?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
