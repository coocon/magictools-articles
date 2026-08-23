# Claude Prompt Chaining 指南：把复杂任务拆成可靠步骤

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-prompt-chaining-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-prompt-chaining-guide?utm_source=github&utm_medium=referral)**

当一个任务不止一个关键步骤时，prompt chaining 往往是让 Claude 表现更稳定的最好办法。Anthropic 的官方建议很直接：如果工作天然能拆成多个阶段，就不要强行把所有内容塞进一个 prompt 里。

原因也很现实。复杂 prompt 经常会出现跳步、混淆指令、或者看起来“差不多对了”但漏掉关键转换的问题。把任务拆开之后，每一步都只需要处理一个明确目标，整体流程更容易控制。

## 为什么 prompt chaining 有效

Anthropic 建议在任务包含多个独立步骤时使用 chaining。它的好处主要有三点：

1. 准确率更高，因为每个子任务都能获得完整注意力。
2. 结构更清晰，因为每一步都可以写得更简单、更具体。
3. 更容易排查，因为你可以快速定位是哪一步出了问题。

这是一种很实用的取舍。单个大 prompt 写起来更快，但一条链式流程通常更容易调试，也更适合重复使用。

## 什么时候适合用

下面这些场景非常适合 prompt chaining：

- 研究资料整理
- 文档分析
- 逐步内容创作
- 信息抽取、转换和汇总
- 生成后还需要复核的任务

如果你能把任务描述成“先收集，再整理，再分析，最后总结”，那它通常就是 chaining 的候选场景。

## 一个简单的链式结构

最稳妥的方式，是让每一步都产出下一步可以直接使用的结果。

```text
Prompt 1：从原始材料中提取关键信息。
Prompt 2：把这些信息整理成结构化大纲。
Prompt 3：根据大纲写出最终答案。
Prompt 4：检查草稿是否遗漏要点，并润色语言。
```

这种方式能保持每一步职责单一。最后如果结果不理想，你也能很快看出是哪一环掉了质量。

## 用 XML 标签做交接

Anthropic 的 chaining 指南建议用 XML 标签在 prompts 之间传递结果。这样做的好处是，模型更容易把“指令”和“数据”分开。

...

---

**[👉 继续阅读全文：Claude Prompt Chaining 指南：把复杂任务拆成可靠步骤](https://tools.cooconsbit.com/zh/articles/claude-prompt-chaining-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
