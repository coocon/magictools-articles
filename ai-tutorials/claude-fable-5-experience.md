# 从 Opus 4.8 切到 Claude Fable 5：为什么我的工具调用不翻车了

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-fable-5-experience?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-fable-5-experience?utm_source=github&utm_medium=referral)**

先声明两件事：第一，这不是评测机构的 benchmark，是我在 Claude Code 里连续几天真实干活的体感，加上 Anthropic 官方文档的交叉验证；第二，这篇文章本身就是 Fable 5 写的、发布的——从读素材、写稿、到调用发布 API、再到线上验证，整条链十几次工具调用一次没断。这算是一种行为艺术式的自证。

## 我遇到的问题：Opus 4.8 的工具调用总"差口气"

先说清楚我说的"工具调用有问题"不是指报错。Opus 4.8 的工具调用格式基本不出错，问题出在**行为层**：

- **该调工具的时候不调**：明明该去搜索、该读文件、该派子任务，它选择靠已有上下文硬答
- **调之前反复确认**：一个明显该直接做的操作，它停下来问"要不要我……？"，一来一回浪费好几轮
- **长任务跑一半断链**：十几步的任务链，中间某一步之后突然开始总结陈词，剩下的活不干了

如果你也有类似体感，有个好消息：**这不是你的错觉，Anthropic 自己的迁移文档白纸黑字承认了**。Opus 4.8 的官方迁移指南里有专门的"行为偏移"章节，原话大意是：

- "更保守地使用需要显式决策的能力"——文件记忆、子代理委派、自定义工具，**不到很确定需要时不会主动去碰**（官方术语叫 under-utilization）
- "更加深思熟虑——更常停下来问"——官方承认它在小决策上也倾向暂停询问，甚至给出了压制这个行为的推荐提示词
- 前代 Opus 4.7 的文档里也写着"默认更少使用工具"

也就是说，Opus 家族近几代在往"谨慎、克制"的方向调，副作用就是 agentic 场景里的"差口气"感。官方给的补救方式是往系统提示词里加各种"何时该调工具"的显式指令——能用，但等于把模型该自己判断的事搬给了写提示词的人。

## Fable 5 是什么

先摆官方事实（来自 Anthropic 官方文档，不是二手消息）：

| 项目 | Fable 5 | Opus 4.8（对比） |
|------|---------|-----------------|
| 定位 | **最强的公开发布模型**，面向最难的推理和长程 agentic 工作 | 最强 Opus 档模型 |
| 上下文 | 1M tokens（默认即最大） | 1M tokens |
| 最大输出 | 128K | 128K |
| 价格 | **$10 / $50** 每百万 tokens（输入/输出） | $5 / $25 |
| Thinking | **永远开启**，不可关闭 | 可开可关（adaptive） |

两个数字值得停一秒：价格是 Opus 4.8 的**整整两倍**，以及 thinking **焊死常开**。这两件事其实是一体的——它被设计成"每一步都先想清楚再动手"的模型，你为这个买单。

## 官方文档里，Fable 5 针对性修了什么

对着 Opus 4.8 的三个痛点，Fable 5 的官方文档几乎是逐条回应：

**1. 长程执行是主打能力，不是附赠。** 官方描述是"长时间自主 agentic 工作的 SOTA"——复杂重构、通宵跑的编码任务，不需要人中途纠偏。单次请求跑十几分钟是设计内行为。这直接对应"长任务断链"问题。

...

---

**[👉 继续阅读全文：从 Opus 4.8 切到 Claude Fable 5：为什么我的工具调用不翻车了](https://tools.cooconsbit.com/zh/articles/claude-fable-5-experience?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
