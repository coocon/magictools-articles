# Claude 研究指南：如何提出更好的研究问题

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-research-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-research-guide?utm_source=github&utm_medium=referral)**

当你需要的不是一个“看起来像答案”的回复，而是一份真的经过调查的结果时，Claude Research 很有用。Anthropic 的官方说明把 Research 描述为一个 beta 功能，它会结合网页和已连接的上下文进行多轮检索，再把结果整理成答案。

最常见的错误，是拿一个模糊问题去指望 Research 自动补全。Research 更适合已有明确决策目标、明确范围和明确证据要求的任务。

## 什么情况下适合用 Research

如果任务需要跨来源综合判断，Research 就很合适：

- 比较产品、供应商或框架
- 总结市场、政策或新闻主题
- 整理网页信息和内部上下文
- 核对应该带引用的事实

如果任务本来就很小、很局部，或者范围已经很清楚，普通对话通常就够了。Research 更强在“探索”，不是替代精确提问。

## 怎样写出更好的 Research 提示词

效果好的 Research 提示词，通常都包含四部分：

1. 你需要的决策或输出。
2. 问题范围，比如时间、地区、公司或受众。
3. 证据标准，比如引用、来源多样性或直接引文。
4. 你希望返回的格式。

例如：

```text
请使用 Research 比较 Claude、ChatGPT 和 Gemini 在内部知识工作中的表现。
优先参考公开文档和最近的官方产品页面。
我最关心长上下文处理、引用质量和工作区功能。
先输出一个简短对比表，再给出带引用的建议。
```

这比直接问“哪个 AI 助手最好”有效得多，因为它给了 Research 明确目标。

## 如何引导 Research

Anthropic 提到，如果 Research 没有自动沿着你想要的角度展开，你可以主动引导。这个能力很重要，因为 Research 是 agentic 的：它会自己决定下一步搜索什么，但你依然可以纠正方向。

...

---

**[👉 继续阅读全文：Claude 研究指南：如何提出更好的研究问题](https://tools.cooconsbit.com/zh/articles/claude-research-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
