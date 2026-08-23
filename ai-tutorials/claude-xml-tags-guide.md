# Claude XML 标签指南：让提示词结构更清晰

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-xml-tags-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-xml-tags-guide?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：Claude XML 标签指南：让提示词结构更清晰](https://tools.cooconsbit.com/zh/articles/claude-xml-tags-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
