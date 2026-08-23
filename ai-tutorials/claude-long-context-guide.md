# Claude 长上下文指南：处理大输入而不丢失重点

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-long-context-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-long-context-guide?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：Claude 长上下文指南：处理大输入而不丢失重点](https://tools.cooconsbit.com/zh/articles/claude-long-context-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
