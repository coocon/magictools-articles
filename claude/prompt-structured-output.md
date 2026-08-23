# 结构化输出与多模态：格式化响应与图文理解

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/prompt-structured-output?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/prompt-structured-output?utm_source=github&utm_medium=referral)**

## 获取 JSON 格式输出

让 Claude 输出结构化 JSON 的最可靠方法是提供明确的 Schema 定义：

```
请分析以下用户评论的情感，以 JSON 格式输出：
{
  "sentiment": "positive | negative | neutral",
  "confidence": 0.0-1.0,
  "keywords": ["关键词数组"],
  "summary": "一句话总结"
}

评论内容：这家餐厅的牛排非常好吃，但是等位时间太长了，服务员态度也一般。
```

Claude 会精确按照 Schema 格式返回结果。

## 预填充保证格式

通过 API 使用时，在 assistant 回复中预填充开头内容可以 100% 保证输出格式：

```python
messages = [
    {"role": "user", "content": "分析这段文本的情感"},
    {"role": "assistant", "content": "{"}  # 预填充
]
```

Claude 会从 `{` 继续，确保输出纯 JSON，不会添加"好的，以下是分析结果："这类前缀。

## XML 标签分区

对于包含多个部分的复杂输出，XML 标签是极好的结构化工具：

```
请用以下 XML 格式输出代码审查结果：

<review>
  <issues>
    <issue severity="high|medium|low">
      <description>问题描述</description>
      <location>文件和行号</location>
      <fix>修复建议</fix>
    </issue>
  </issues>
  <summary>整体评价</summary>
  <score>1-10</score>
</review>
```

...

---

**[👉 继续阅读全文：结构化输出与多模态：格式化响应与图文理解](https://tools.cooconsbit.com/zh/articles/prompt-structured-output?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
