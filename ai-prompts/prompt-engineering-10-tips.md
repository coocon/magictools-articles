# Prompt 工程入门：写出高质量 AI 提示词的 10 个实用技巧

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/prompt-engineering-10-tips?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/prompt-engineering-10-tips?utm_source=github&utm_medium=referral)**

你是否有过这样的经历：用同样的 AI 工具，别人能得到精准、专业的输出，而你得到的却是废话连篇、不知所云？

区别不在于工具，在于提示词（Prompt）的质量。Prompt 工程不是什么神秘技术，它是一套可以学习、可以迭代的沟通方法论。这篇文章整理了 10 个经过验证的实用技巧，每个技巧都有反面教材和正确示例对比。

## 为什么 Prompt 质量决定 AI 输出质量

大型语言模型本质上是一个极其复杂的"补全机器"——它会根据你的输入，预测最合理的后续内容。你的 Prompt 质量直接决定了模型的"思考方向"。

一个模糊的 Prompt 会让模型自己猜测你的意图，而模型的猜测往往是最保守、最平庸的答案。一个精准的 Prompt 则像给模型装上了导航，让它直接驶向你需要的目的地。

## 10 个实用技巧

### 技巧 1：明确指定角色

给 AI 赋予一个具体角色，它会调用与该角色最匹配的知识库和表达风格。

❌ **错误示例**
```
帮我写一篇关于 Python 的文章。
```

✅ **正确示例**
```
你是一位有 10 年经验的 Python 工程师，擅长向非技术人员解释编程概念。
请为完全没有编程基础的产品经理写一篇 Python 入门文章，重点说明 Python 能解决哪些实际工作问题。
```

角色设定不需要夸张，要贴合实际需求。"世界顶级专家"不如"有 X 年经验的从业者"来得具体有效。

---

### 技巧 2：描述期望的输出格式

告诉 AI 你需要什么格式的输出，它会严格按照格式生成，省去你大量的二次整理工作。

❌ **错误示例**
```
列出 5 个热门 JavaScript 框架的对比。
```

✅ **正确示例**
```
请用 Markdown 表格对比以下 5 个 JavaScript 框架：React、Vue、Angular、Svelte、Solid.js
表格列包含：框架名称、发布年份、学习曲线（低/中/高）、适用场景、2026年 GitHub Star 数量级
```

常用的格式指令：`JSON 格式`、`Markdown 表格`、`编号列表`、`代码块`、`分点说明`。

---

### 技巧 3：提供上下文背景信息

AI 不了解你的具体情况，给它背景信息，它才能给出针对性建议而不是通用废话。

❌ **错误示例**
```
帮我优化这个 SQL 查询速度。
```

✅ **正确示例**
```
我有一个 MySQL 8.0 数据库，用户表有 500 万条记录。
以下查询每次执行需要 8 秒，已严重影响用户体验：

SELECT u.*, o.order_count
FROM users u
LEFT JOIN (SELECT user_id, COUNT(*) as order_count FROM orders GROUP BY user_id) o
ON u.id = o.user_id
WHERE u.created_at > '2025-01-01';

请分析性能瓶颈并给出优化方案，说明每个优化的原理。
```

上下文三要素：**环境**（技术栈/版本/规模）、**现状**（具体问题/数据/代码）、**目标**（期望达到什么效果）。

---

### 技巧 4：使用 Few-Shot 示例

给 AI 展示 2~3 个"你想要的输出"的例子，它会理解你的品味和标准，模仿该风格输出。

...

---

**[👉 继续阅读全文：Prompt 工程入门：写出高质量 AI 提示词的 10 个实用技巧](https://tools.cooconsbit.com/zh/articles/prompt-engineering-10-tips?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
