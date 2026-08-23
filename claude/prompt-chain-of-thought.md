# 思维链提示：让 Claude 一步步解决复杂问题

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/prompt-chain-of-thought?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/prompt-chain-of-thought?utm_source=github&utm_medium=referral)**

## 什么是思维链提示？

思维链（Chain of Thought，简称 CoT）是一种让 AI 展示推理过程的提示技巧。与直接给出答案不同，CoT 让 Claude 把解题步骤一步步写出来，大幅提高复杂问题的准确率。

研究表明，对于需要多步推理的任务（数学、逻辑、代码调试等），思维链提示可以显著减少错误。

## 最简单的用法

只需在提示词中添加一句话：

> 请一步步思考，然后给出最终答案。

**示例 — 无思维链：**
> 一个水池有两个水管，A 管单独注满需要 6 小时，B 管单独注满需要 3 小时。同时打开两个管，多久注满？

Claude 可能直接给出"2 小时"，但偶尔会犯错。

**示例 — 加入思维链：**
> 一个水池有两个水管，A 管单独注满需要 6 小时，B 管单独注满需要 3 小时。同时打开两个管，多久注满？请一步步推理。

Claude 会输出：
1. A 管每小时注入 1/6 池
2. B 管每小时注入 1/3 池
3. 两管合计每小时注入 1/6 + 1/3 = 1/6 + 2/6 = 3/6 = 1/2 池
4. 注满需要 1 ÷ 1/2 = 2 小时

展示推理过程让 Claude 自我检查，减少计算错误。

## Extended Thinking：内置深度推理

Claude 提供了 Extended Thinking 功能。开启后，Claude 会在回答前进行内部深度推理（用户可以看到思考过程）。通过 API 使用时，你可以设置 `budget_tokens` 参数来控制思考的深度：

```json
{
  "thinking": {
    "type": "enabled",
    "budget_tokens": 5000
  }
}
```

适合需要极高准确率的场景，如数学证明、复杂代码逻辑分析等。

## Few-Shot CoT：用示例教 Claude 推理

...

---

**[👉 继续阅读全文：思维链提示：让 Claude 一步步解决复杂问题](https://tools.cooconsbit.com/zh/articles/prompt-chain-of-thought?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
