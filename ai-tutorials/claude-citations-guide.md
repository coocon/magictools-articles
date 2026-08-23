# Claude 引用指南：让回答更容易核对

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-citations-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-citations-guide?utm_source=github&utm_medium=referral)**

Claude 的引用功能只有一个核心目标：让答案更容易核对。你不再只是得到一段流畅的回答，而是可以把它和你提供的资料一一对照。对于政策文本、研究材料、合同、内部笔记这类内容，这一点尤其重要。

Anthropic 的官方文档也讲得很直接：引用不是自动发生的。你仍然需要提供合适的源材料、开启引用功能，并明确告诉 Claude 你想要什么样的答案。设置正确时，引用能把 Claude 从“会解释”变成“可审计”。

## 引用功能适合做什么

当读者需要核对答案来源时，就应该使用引用功能。适合的场景包括：

- 总结一份文档，并保留可追踪的依据
- 比较同一份政策的两个版本
- 从报告中提取证据
- 回答准确性比文风更重要的问题

它的意义不是让每个回答都更长，而是让重要结论更容易核查。

## 引用功能是怎么工作的

Anthropic 的引用文档描述了一个简单流程：

1. 提供受支持格式的文档。
2. 为这些文档启用引用。
3. 要求 Claude 使用这些资料回答问题。
4. Claude 的回答中会出现指向源位置的引用。

支持的文档类型包括纯文本、PDF 和自定义内容文档。Anthropic 还说明，目前引用是文本引用，不是图片引用。

这意味着，引用功能最适合“源文本分析”类任务。如果源材料是 PDF，Claude 可以引用页面级内容；如果是纯文本，引用则会映射到字符位置。

## 一个适合引用功能的提示词结构

提示词里要同时说明“回答什么”和“如何锚定证据”。

```text
请使用提供的文档回答下面的问题。

任务：[你需要什么]
证据要求：每个重要结论都要给出引用。
输出格式：[简短摘要、项目符号、表格等]
约束：不要添加无法从原文中支持的结论。

问题：[你的问题]
```

如果问题有多个子项，建议明确分点。这样 Claude 更容易把每个答案对齐到对应来源。

...

---

**[👉 继续阅读全文：Claude 引用指南：让回答更容易核对](https://tools.cooconsbit.com/zh/articles/claude-citations-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
