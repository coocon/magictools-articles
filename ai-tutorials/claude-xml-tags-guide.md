---
title: "Claude XML 标签指南：让提示词结构更清晰"
slug: "claude-xml-tags-guide"
category: ai-tutorials
tags:
  - claude
  - anthropic
  - xml
  - 提示词工程
summary: "介绍如何在 Claude 提示词中使用 XML 标签，把上下文、指令、示例和输出要求清晰分离。"
coverImage: ""
status: published
scheduledAt: ""
---

XML 标签是 Anthropic 提示词工程里非常实用的一种结构化工具。它能帮助 Claude 区分指令、示例、上下文和输出格式，避免这些内容混在一起。

它最重要的价值不是“高级”，而是稳定。当提示词里包含多个部分时，结构清晰会让 Claude 更容易正确理解你的需求。

## 为什么 XML 标签有用

当提示词包含多个部分，或者你希望 Claude 更稳定地解析内容时，就适合用 XML 标签：

- 把指令和原始材料分开
- 清楚标记示例
- 单独说明输出格式
- 让复杂提示词更容易维护

标签名不需要花哨，只要和内容对应，并且保持一致就行。

## 一个简单写法

```text
<instructions>
请把这段笔记改写成适合管理层阅读的版本。
</instructions>

<context>
这是每周项目更新，阅读者是高管。
</context>

<output_format>
请输出 3 个要点：状态、风险、下一步。
</output_format>
```

这样的结构会让 Claude 更容易看清任务边界，也方便你以后回头修改提示词。

## 最佳实践

Anthropic 官方文档强调了几个实用习惯：

1. 标签名保持一致。
2. 用标签区分不同类型的内容。
3. 当内容有层级时，可以嵌套标签。

例如：

```text
<contract>
  <section>...</section>
  <section>...</section>
</contract>
```

这种方式特别适合长文本输入，也更利于 Claude 在输出时保持和原文一致的结构。

## 什么时候搭配其他技巧一起用

XML 标签和示例提示、推理提示都很适合搭配使用。Anthropic 还明确建议，当任务更复杂时，可以把标签和多示例提示或链式思考提示结合起来，让结构更清楚。

常见组合方式：

- 用 `<examples>` 放示例输出
- 用 `<thinking>` 放中间推理
- 用 `<answer>` 放最终答案

这种分离方式尤其适合后续只提取某一部分结果的场景。

## 实用示例

```text
<instructions>
请把下面的客户反馈总结给产品团队。
</instructions>

<feedback>
[在这里粘贴原始反馈]
</feedback>

<output_format>
请返回：
1. 总体情绪
2. 主要投诉点
3. 一个建议动作
</output_format>
```

比起把所有说明写成一大段文字，这种结构更容易让 Claude 把每个部分分别处理好。

## 常见错误

- 标签名太模糊
- 所有内容都塞进一个标签，失去结构价值
- 指令和数据混在一起，没有边界
- 以为加了标签就不需要明确任务了

XML 标签只能提升清晰度，不能替代好的提示词设计。你仍然需要写清楚任务、背景和最终目标。

## 总结

如果你的 Claude 提示词越来越长、越来越复杂，XML 标签是保持结构清楚的最好方法之一。它能减少歧义，让提示词更容易维护，也很适合和示例、推理提示一起使用。

## 官方参考资料

- [Use XML tags to structure your prompts](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-xml-tags)
- [Chain complex prompts for stronger performance](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/chain-prompts)
- [Let Claude think (chain of thought prompting) to increase performance](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/chain-of-thought)
- [Prompt engineering overview](https://docs.anthropic.com/en/docs/prompt-engineering)

以上资料检索于 2026年3月29日。提示行为和最佳实践可能会更新，发布前请以链接中的 Anthropic 官方页面为准。
