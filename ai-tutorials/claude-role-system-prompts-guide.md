---
title: "Claude 角色提示词指南：什么时候该用 System Prompt"
slug: "claude-role-system-prompts-guide"
category: ai-tutorials
tags:
  - claude
  - anthropic
  - system prompts
  - prompt engineering
summary: "讲清楚 Claude 里的角色提示词和 system prompt 该如何使用，以及为什么 Anthropic 建议把角色定义和具体任务分开。"
coverImage: ""
status: published
scheduledAt: ""
---

在 Anthropic 的官方文档里，角色提示词是 system prompt 的一个高价值用法。它的核心逻辑并不复杂：先通过 `system` 参数告诉 Claude“你是谁、应该以什么视角工作”，再用用户消息描述当前这一轮任务。

很多用户会把角色、任务、格式要求、背景资料全部写进同一个提示词里。这样当然也能工作，但结构会越来越乱，后续维护和复用都很困难。Anthropic 官方建议把长期稳定的行为要求放到 system prompt，把会变化的任务要求留给 user prompt。

## system prompt 适合放什么

一个好的 system prompt，不是把所有项目背景一次性塞进去，而是定义那些**跨任务稳定存在**的行为规则，例如：

- Claude 要扮演什么角色
- 需要用什么专业视角思考
- 保持什么写作风格或判断框架
- 哪些边界条件需要持续生效

Anthropic 的建议非常明确：如果某条要求在多个任务之间都成立，它更适合进入 system prompt。

## 哪些场景特别适合角色提示词

角色提示词通常在下面这些场景里最有价值：

1. 你需要更符合某个专业领域的分析方式。
2. 你希望多轮输出始终保持一致风格。
3. 你希望 Claude 始终站在固定的业务视角上工作。

例如，“你是一名高级安全工程师，正在审查上线方案”通常会比“帮我看看这个上线方案”更容易得到聚焦、专业的反馈。

## 一个更清晰的结构写法

下面是一个符合 Anthropic 官方建议的写法：

### System prompt

```text
你是一名资深产品运营经理，擅长为跨部门管理层撰写简洁、可决策的状态更新。你会优先指出阻塞项、依赖关系和需要拍板的事项。
```

### User prompt

```text
请根据下面的会议记录，起草一份本周项目更新。

必须包含：
1. 总体状态
2. 关键风险
3. 本周需要决策的事项

约束：控制在 220 字以内。

会议记录：
[在这里粘贴内容]
```

这样的结构最大的好处，是角色定义可以长期复用，而当前任务随时可以替换。

## 为什么这种拆分方式更稳

把角色和任务分开之后，提示词更容易测试、修改和复用。你不需要每次都重复解释“请以产品运营视角回答”“请保持简洁、面向管理层”，只需要在 system prompt 里定义一次即可。

在 API 或重复性工作流里，这种拆分尤其重要。因为它让提示词架构更加清晰，也更容易排查到底是哪一层指令出了问题。

## 常见误区

角色提示词常见的误用方式主要有几种：

- 角色定义太空泛，比如只写“你是专家”
- 把经常变化的任务要求塞进 system prompt
- 以为角色本身就能弥补上下文缺失
- 把 role prompting 当成模糊任务说明的替代品

Anthropic 的文档其实说得很清楚：角色可以帮助 Claude 更聚焦，但它不能替代明确任务、材料和输出要求。

## 怎么判断一条要求该不该放进 system prompt

你可以问自己两个问题：

1. 这条要求在多个相关任务之间是否都成立？
2. 它是否真的改变 Claude 的思考方式或表达方式？

如果答案都是“是”，那它大概率应该放进 system prompt。否则，它更可能属于当前任务说明的一部分。

## 官方参考资料

- [Giving Claude a role with a system prompt](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/system-prompts)
- [Be clear, direct, and detailed](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/be-clear-and-direct)
- [Prompt engineering overview](https://docs.anthropic.com/en/docs/prompt-engineering)

以上资料检索于 2026年3月29日。功能可用性、套餐限制和界面细节可能会变化，发布前请以链接中的 Anthropic 官方资料为准。
