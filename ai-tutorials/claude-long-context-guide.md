---
title: "Claude 长上下文指南：处理大输入而不丢失重点"
slug: "claude-long-context-guide"
category: ai-tutorials
tags:
  - claude
  - anthropic
  - 长上下文
  - 提示词工程
summary: "学习如何更有效地使用 Claude 的长上下文能力，包括文档排序、XML 标签拆分和先引用后分析的写法。"
coverImage: ""
status: published
scheduledAt: ""
---

Claude 在面对大段输入时，最需要的不是你把所有内容一股脑丢进去，而是你帮它把结构理清。Anthropic 的长上下文建议很实用：把长文档放在前面，用 XML 标签区分来源，并在正式分析前先让 Claude 提取引用内容。

这不只是为了“装得下更多文本”，而是为了减少噪音、保留来源边界，并让 Claude 更容易在多页文档、多文件资料或大量背景信息中找到真正重要的部分。

## Anthropic 对长上下文的建议

官方建议主要集中在三点：

1. 把长文本放在提示词前面。
2. 用 XML 标签包裹每个文档，方便模型区分来源。
3. 先让 Claude 引用相关片段，再做总结、比较或建议。

这些做法有效，是因为 Claude 并不会对长提示词中的每一部分等价处理。结构越清晰，可检索的信息就越容易被抓住。

## 实用工作流

当你需要 Claude 分析一大批材料时，可以按这个顺序来：

1. 先放来源文档。
2. 用统一标签分隔，比如 `<document>`、`<source>`、`<document_content>`。
3. 在提示词末尾写清楚任务。
4. 先要求 Claude 提取最相关的引用，再进行总结、比较或判断。

这个顺序很重要。Anthropic 明确提到，把问题放在末尾，在多文档长输入场景下通常能提升回答质量。

## 提示词示例

```text
<document>
  <source>政策备忘录</source>
  <document_content>
  [在这里粘贴备忘录]
  </document_content>
</document>

<document>
  <source>会议纪要</source>
  <document_content>
  [在这里粘贴纪要]
  </document_content>
</document>

任务：
1. 先引用与问题最相关的片段。
2. 比较两份材料。
3. 给出简洁建议。

问题：这些文档在哪些地方对最终决定达成一致，哪些地方存在冲突？
```

这种写法把“检索”和“推理”分开了，Claude 更容易说明结论是从哪里来的。

## 什么场景最适合长上下文

长上下文最适合这些需要跨文档关联的任务：

- 政策审查
- 合同或需求对比
- 研究综述
- 大段代码或设计审核
- 会议记录和项目历史整理

如果问题本身很窄，长上下文反而不一定更好。输入很大不代表一定更有价值，必要时先裁剪内容再提问。

## 常见错误

最常见的问题其实都很典型：

- 把所有材料和问题混成一大段
- 还没提取证据就直接要求结论
- 多份文档混在一起却没有分隔符
- 把长上下文当成“自动理解一切”的替代品

长提示词仍然需要任务设计。内容更多，不等于结果更好。

## 一个实用原则

如果 Claude 需要比较或综合多个来源，先让它提取相关引用，再让它解释这些引用之间的关系。这个两步法通常比一步直接要最终答案更稳定。

## 官方参考资料

- [Long context prompting tips](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/long-context-tips)
- [Prompt engineering overview](https://docs.anthropic.com/en/docs/prompt-engineering)
- [Getting started with Claude](https://support.anthropic.com/en/articles/8114491-how-do-i-get-started-with-claude-ai)

以上资料检索于 2026年3月29日。功能可用性、套餐限制和界面细节可能会变化，发布前请以链接中的 Anthropic 官方资料为准。
