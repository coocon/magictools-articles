# 系统提示词设计：让 Claude 精准理解你的需求

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/prompt-system-prompts?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/prompt-system-prompts?utm_source=github&utm_medium=referral)**

## 系统提示词与用户消息的区别

在 Claude 的 API 中，消息分为两个层级：系统提示词（System Prompt）和用户消息（User Message）。系统提示词定义了 Claude 的"身份"和"行为准则"，它在整个对话过程中持续生效。用户消息则是具体的任务请求。

将固定的指令放入系统提示词，变化的内容放入用户消息，是高效使用 Claude 的关键模式。

## 角色定义模式

角色定义是系统提示词中最强大的技巧。它不仅设定了 Claude 的专业领域，还暗含了回答的深度和风格。

**基础角色定义：**

```
你是一位有 10 年经验的数据分析师，擅长用通俗语言向非技术人员解释数据洞察。
```

**进阶角色定义（加入行为约束）：**

```
你是一位严谨的法律顾问。
- 回答始终基于中国现行法律法规
- 如果问题超出你的确定范围，明确告知用户寻求专业律师帮助
- 不做任何法律承诺或保证
- 引用具体法条时注明出处
```

## 输出格式控制

系统提示词非常适合定义固定的输出格式。以下是一个 JSON 输出的实际示例：

```
所有回答必须使用以下 JSON 格式输出，不要包含其他文本：
{
  "answer": "直接回答",
  "confidence": "high/medium/low",
  "sources": ["参考来源"],
  "caveats": ["注意事项"]
}
```

## 预填充技巧

通过预填充 Claude 的回复开头，可以强制输出特定格式。在 API 调用中，你可以在 assistant 消息中预填充：

```
// 用户消息：分析这段代码的性能问题
// 预填充 assistant 消息为：
```json
```

这样 Claude 会直接从 JSON 格式开始输出，避免添加额外的解释文字。

## 约束条件设计

好的系统提示词要同时告诉 Claude "做什么"和"不做什么"：

```
## 必须做的
- 每个回答都附带一个实际操作步骤
- 使用表格对比多个选项
- 不确定时说明置信度

...

---

**[👉 继续阅读全文：系统提示词设计：让 Claude 精准理解你的需求](https://tools.cooconsbit.com/zh/articles/prompt-system-prompts?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
