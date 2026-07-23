---
title: "Claude Few-Shot 提示词指南：用示例锁定输出格式"
slug: "claude-few-shot-prompting-guide"
category: ai-tutorials
tags:
  - claude
  - anthropic
  - few-shot prompting
  - prompt engineering
summary: "讲清楚如何在 Claude 中使用 few-shot / multishot 提示词，通过示例提高输出格式、分类规则和风格的一致性。"
coverImage: ""
status: published
scheduledAt: ""
---

Anthropic 在官方提示词工程文档里把“示例”视为一个非常高价值的技巧。它常被叫做 few-shot 或 multishot prompting。简单来说，就是你不要只告诉 Claude 结果应该长什么样，而是直接给它看几个高质量样例。

这招特别适合那些“你脑子里很清楚，但很难用一句话解释清楚”的任务。比如固定输出格式、分类标准、风格要求，或者边界条件复杂的判断题。相比写一大段抽象说明，几个好例子往往更有效。

## 哪些场景最适合 few-shot

示例驱动的提示词在下面几类任务里尤其有用：

- 你需要稳定一致的输出结构
- 你在做文本分类、标签判断、内容审核
- 你希望 Claude 模仿某种固定风格
- 你担心边界情况处理不一致

Anthropic 官方建议，在比较复杂的任务里可以考虑提供 3 到 5 个相关且有差异性的例子。

## 什么样的示例才算好

根据 Anthropic 的官方说明，好的示例通常有三个特点：

1. **相关**：和你真正要做的任务足够接近。
2. **多样**：能够覆盖不同常见情况，而不是只重复一种模式。
3. **清晰**：示例之间边界明显，Claude 很容易解析。

官方文档还建议使用 XML 风格标签，比如 `<example>`、`<examples>`，来显式划分示例块。这会让 Claude 更容易识别结构。

## 一个实用的 few-shot 模板

```text
请把每条客户消息分类到以下标签之一：
- 计费问题
- 技术故障
- 取消服务
- 功能建议

请参考示例中的判断方式。

<examples>
<example>
消息："我这个月被重复扣费了。"
标签：计费问题
</example>

<example>
消息："导出 PDF 时应用会崩溃。"
标签：技术故障
</example>

<example>
消息："请在当前账期结束后关闭我的账号。"
标签：取消服务
</example>
</examples>

现在请分类：
消息："[在这里插入新的客户消息]"
```

这个写法的价值在于，Claude 不仅知道“要分类”，还知道“你希望按照什么判断边界去分类”。

## 为什么 few-shot 往往比长篇解释更有效

如果你只写任务说明，Claude 需要自己推断什么才算正确格式或正确判断。示例的作用，是把这种推断空间大幅压缩。你不是在解释一种抽象规则，而是在提供可以模仿和归纳的证据。

这也是 why few-shot 常常能快速提升稳定性。你把模糊要求换成了具体样本。

## 常见错误

few-shot 提示词最常见的问题通常有这几类：

- 示例和真实任务不够接近
- 示例之间过于重复，没有覆盖变化
- 不小心用低质量样例教错了 Claude
- 不明确区分示例区和真正要处理的新输入

Anthropic 的文档甚至建议你反过来让 Claude 评估你的示例集是否“相关、充分、多样”。如果你在做长期复用的工作流，这一步很值得加上。

## 一个实用判断标准

如果你发现自己正在用很长一段话解释格式、风格或判断逻辑，不妨停下来问自己一句：这件事是不是用 2 到 5 个例子就能更快说明白？

在很多情况下，答案都是“能”。few-shot 不是对清晰指令的替代，而是对清晰指令的强化。和明确要求结合起来时，效果通常最好。

## 官方参考资料

- [Use examples (multishot prompting) to guide Claude's behavior](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/multishot-prompting)
- [Prompt engineering overview](https://docs.anthropic.com/en/docs/prompt-engineering)
- [Be clear, direct, and detailed](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/be-clear-and-direct)

以上资料检索于 2026年3月29日。功能可用性、套餐限制和界面细节可能会变化，发布前请以链接中的 Anthropic 官方资料为准。
