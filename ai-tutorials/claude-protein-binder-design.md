# Claude 从零设计蛋白质：成功率翻倍

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-protein-binder-design?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-protein-binder-design?utm_source=github&utm_medium=referral)**

> 本文基于 Anthropic 2026 年研究博客《How Claude is accelerating protein design and analytical chemistry》整理，提炼 10 个观点并做解读。引语均为原文英文原句。
> 原文链接：https://www.anthropic.com/research/Claude-accelerates-protein-design

---

过去几个月，AI 在数学上刷成绩已经不新鲜了——因为数学的验证很便宜，跑个证明检查器就知道对不对。

真正难的是那些「验证很贵」的领域：生命科学。你想知道一个蛋白质设计对不对，得合成出来、送进湿实验室、等几周。

这次 Anthropic 干的就是这件事。他们让 Claude 独立跑完一场真实的蛋白质结合剂（minibinder）设计战役，把设计稿寄给两家外部实验室做物理验证，然后把结果、提示词、全部数据一次性公开。

以下是 10 个值得注意的点。

---

## 1. 22%–35% 对 10%–15%：这不是提升，是翻倍

> "Claude (Mythos Preview and Opus 4.8) designed protein binders against 15 targets, and succeeded against 14 of them. Between 22% and 35% of its individual designs bound successfully, depending on the setup, compared to the 10-15% that is typical in protein design campaigns today."

命中率（hit rate）是这行的硬通货：你设计 100 个候选，最后真能结合上靶点的有几个。

行业今天的典型水平是 10%–15%。Claude 拿到的是：多靶点 48 小时会话里，Mythos Preview 26.7%、Opus 4.8 22.6%；改成单靶点模式（每个靶点独立开 24 小时会话），Mythos Preview 冲到 35.1%。

15 个靶点里成功了 14 个，总共产出 354 个确认结合剂，来自 1320 份设计。

这个数字的分量在于：它不是把 10% 推到 12%，而是直接翻倍。在一个「候选合成成本按个计价、湿实验室排期按周计价」的行业里，命中率翻倍意味着同样的钱能验证的有效假设翻倍。

**My take：** 大多数 AI-for-Science 的成绩单要么是基准测试分数，要么是「协助专家完成了 X」。这次给的是产业界每天都在看的那个指标，而且是在真实实验里量出来的。指标选对了，说服力就完全不一样。

---

## 2. 端到端自主：人类只负责点「同意」

> "After giving Claude the prompt, we left the model to execute autonomously. We provided no additional scientific, technical, or operational guidance after we initiated the campaigns."

这句话是整篇文章最狠的一句。

人类的全部参与是：批准网络访问请求、盯着基础设施别崩、把生成的设计寄去实验室下单。科学上、技术上、操作上的指导，零。

Claude 自己做完了这些事：选择在靶点蛋白的哪个位置下手；编排多个结构设计、序列设计和共折叠模型生成候选；跑多轮计算机模拟优化；筛选出既新颖、又能表达、又能保持可溶、又能结合的候选。

> "Claude conducted all of the work that goes into designing a binder, which can take a human operator weeks."

资源配置也公开了：多靶点模式 48 小时 wall time + 最多 12500 个 H100 小时；单靶点模式每个靶点 24 小时 + 最多 2500 个 H100 小时。token 和子智能体预算不设限，fast mode 开着。

**My take：** 注意「orchestrating」这个词——Claude 用的是这个领域已经在用的公开专业模型，它没有发明新算法。它做的是编排：决定用哪个模型、怎么串、什么时候该重来一轮。这恰好是过去需要一个计算生物学专家花几周手工做的事。AI 在科研里的第一个真实位置，可能不是「发现新原理」，而是「替掉那个编排流水线的人」。

---

## 3. 验证方是外人，不是自己

> "Our external evaluators, Adaptyv Bio and Twist Bioscience, independently produced and tested Claude's designs in the lab, finding that of the 15 targets we designed against, Claude successfully designed binders against 14 of them."

Adaptyv Bio 和 Twist Bioscience 独立生产并测试了这些设计。Anthropic 不碰湿实验室那一段。

...

---

**[👉 继续阅读全文：Claude 从零设计蛋白质：成功率翻倍](https://tools.cooconsbit.com/zh/articles/claude-protein-binder-design?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
