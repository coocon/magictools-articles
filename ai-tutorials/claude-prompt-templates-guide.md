# Claude 提示词模板指南：复用好提示词而不丢质量

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-prompt-templates-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-prompt-templates-guide?utm_source=github&utm_medium=referral)**

提示词模板是把“临时提问”推进到“可复用工作流”的关键一步。Anthropic 的官方文档把模板描述成固定内容和变量内容的组合，这正好适合那些会反复出现、但每次输入不同的任务。

如果你希望 Claude 的行为稳定，又不想每次都重写完整 prompt，模板就是最合适的工具。你把稳定不变的部分写进模板，再把变化的内容放进变量里即可。

## 模板由什么组成

Anthropic 把模板内容分成两类：

1. 固定内容：每次请求都保持不变
2. 变量内容：每次请求都会变化

常见变量包括用户输入、RAG 检索内容、对话上下文，以及从工具调用返回的结果。这样的结构让 prompt 更容易阅读，也更容易测试。

## 什么时候该用模板

只要 prompt 中有一部分会在后续调用中重复出现，就适合使用模板。Anthropic 明确说明，这类能力主要用于 API 或 Anthropic Console，不是 claude.ai 的功能。

因此，模板特别适合这些场景：

- 客服工作流
- 数据抽取流程
- 内部助手的重复指令
- 需要固定评分标准的多步骤任务
- 需要统一输出格式的研究或摘要任务

## 一个实用的模板结构

一个清晰的模板，通常会把指令和变量分开。

```text
你正在帮助产品团队生成周报摘要。

任务：
把下面的输入整理成适合管理层阅读的摘要。

受众：
{{audience}}

输入：
{{source_text}}

输出格式：
1. 关键决策
2. 风险
3. 下一步
4. 待解决问题
```

这种结构好维护，因为 prompt 的整体形状保持稳定，变化的只是源文本。

## 为什么变量很重要

变量可以减少 prompt 漂移。如果指令部分稳定，你就不容易在更新内容时不小心改了任务本身。

这在下面这些场景里尤其有用：

- 测试 prompt 行为
- 对比不同版本
- 在不同用户或文档之间复用同一流程
- 从其他系统动态传入数据给 Claude

...

---

**[👉 继续阅读全文：Claude 提示词模板指南：复用好提示词而不丢质量](https://tools.cooconsbit.com/zh/articles/claude-prompt-templates-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
