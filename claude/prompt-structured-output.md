---
title: "结构化输出与多模态：格式化响应与图文理解"
slug: prompt-structured-output
category: claude
tags: [prompt-engineering, structured-output, multimodal, vision, advanced]
summary: "学习如何让 Claude 输出 JSON、XML 等结构化格式，以及如何利用 Claude 的视觉能力理解图片、文档和图表。"
status: published
---

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

XML 标签的优势在于支持嵌套和属性，比纯 JSON 更灵活地表达层级关系。

## 处理边界情况

结构化输出时需要预防几种常见问题：

```
输出要求：
- 始终返回有效 JSON，即使输入数据异常
- 如果无法分析某个字段，使用 null 而非空字符串
- 数组字段在没有数据时返回 []，不要省略
- 不要在 JSON 外添加任何解释文字
```

这些约束能让你的程序稳定解析 Claude 的输出。

## 多模态：图片理解

Claude 具备强大的视觉能力。通过 API 发送图片时，可以结合文字提示进行分析：

```
请分析这张截图中的 UI 界面：
1. 列出所有可见的 UI 组件
2. 指出不符合设计规范的地方
3. 给出改进建议

以 JSON 数组格式输出，每项包含 component、issue、suggestion 三个字段。
```

### 常见图片分析场景

| 场景 | 提示词示例 |
|------|-----------|
| OCR 文字提取 | "提取图片中的所有文字，保持原始排版" |
| 图表数据读取 | "读取柱状图中的数据，以表格形式输出" |
| UI 描述 | "描述这个界面的布局结构和交互元素" |
| 文档解析 | "提取这张发票中的关键信息：日期、金额、供应商" |

## 视觉 + 结构化输出的组合

将视觉能力和结构化输出结合是最强大的应用模式：

```
请分析这张产品截图，以 JSON 格式输出：
{
  "product_name": "产品名称",
  "price": "价格",
  "rating": "评分",
  "key_features": ["特征列表"],
  "visible_issues": ["界面问题"]
}
```

这种组合非常适合构建自动化数据提取管线。

## 常见问题

### Claude 能 100% 保证输出有效 JSON 吗？

在大多数情况下可以，尤其是使用预填充技巧时。但在极端情况下（非常长的输出被截断），JSON 可能不完整。建议在代码中始终使用 try-catch 包裹 JSON 解析，并设置重试机制。

### Claude 能处理哪些类型的图片？

Claude 支持 JPEG、PNG、GIF 和 WebP 格式的图片。它可以理解照片、截图、图表、文档扫描件、手写笔记等。但对于极小的文字、模糊的图片或高度抽象的艺术作品，准确率会下降。

### 结构化输出和自由格式输出哪个更好？

取决于用途。如果输出需要被程序解析（API 集成、数据管线），一定使用结构化输出。如果是给人阅读的内容（文章、邮件、报告），自由格式通常更自然。两者也可以结合——让 Claude 在 JSON 的特定字段中放入自由格式文本。

### 发送图片时有什么限制？

通过 API 单次请求最多可以发送 20 张图片。图片会消耗 token 配额，高分辨率图片消耗更多。建议在发送前适当压缩图片，保持关键细节清晰即可，不需要超高分辨率。
