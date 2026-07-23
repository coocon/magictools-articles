---
title: "Claude Prompt Chaining 指南：把复杂任务拆成可靠步骤"
slug: "claude-prompt-chaining-guide"
category: ai-tutorials
tags:
  - claude
  - anthropic
  - prompt chaining
  - prompt engineering
summary: "基于 Anthropic 官方文档，介绍如何用 prompt chaining 把复杂任务拆成更容易控制的多个步骤。"
coverImage: ""
status: published
scheduledAt: ""
---

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

```text
<source>
[把原始材料放在这里]
</source>

<task>
只提取决策和待解决问题。
</task>
```

这样一来，每一步都有清晰的输入输出边界，不会把说明文字和实际内容混在一起。

## 适合落地的工作流

处理重要任务时，可以按这个顺序来：

1. 把任务拆成几个明确的子步骤。
2. 给每一步只设一个目标。
3. 用结构化格式把结果传给下一步。
4. 只修有问题的那一环，不要动不动整条链重来。

最后一点尤其重要。如果第三步质量差，就先改第三步，而不是整条流程反复重跑。

## 自我修正链

Anthropic 还提到，Claude 可以检查自己的输出。这在高风险任务里很有用，因为第二遍往往能发现遗漏、逻辑不顺或表达不清。

一个简单版本是：

1. 先生成第一版。
2. 再让 Claude 按目标检查草稿并指出缺口。
3. 最后只修改薄弱部分。

这通常比完全重写更稳，因为可以保留好的部分，只修复有问题的地方。

## 常见错误

最常见的 chaining 错误很简单：

- 每一步都太宽泛
- 一个子任务里混了多个目标
- 没有说明下一步应该接收什么结果
- 只有一环出问题，却把整条链全部重跑

如果某一步没法用一句话说明白，它通常还是太大了。

## 核心原则

prompt chaining 不是把 prompt 写得更长，而是把每一步写得更小、更干净、更容易评估。对于需要结构、准确性和可追踪性的任务，这通常比一个超长 prompt 更可靠。

## 官方参考资料

- [Chain complex prompts for stronger performance](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/chain-prompts)
- [Prompt engineering overview](https://docs.anthropic.com/en/docs/prompt-engineering)
- [Be clear, direct, and detailed](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/be-clear-and-direct)

以上资料检索于 2026年3月29日。功能可用性、套餐限制和界面细节可能会变化，发布前请以链接中的 Anthropic 官方资料为准。
