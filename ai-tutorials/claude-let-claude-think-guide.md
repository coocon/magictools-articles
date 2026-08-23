# Claude 思考指南：什么时候应该让它一步一步推理

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-let-claude-think-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-let-claude-think-guide?utm_source=github&utm_medium=referral)**

当任务需要多步推理时，Claude 的表现通常会明显更好。Anthropic 的提示词工程文档把这种方法称为 chain-of-thought prompting，核心思路很直接：如果任务需要分析、取舍或校验，就不要一上来直接让 Claude 给结论。

这类方法尤其适合那些人也不可能一眼就答出来的任务。比如比较多个方案、分析一份文档、审查计划，或者处理逻辑较复杂的问题，先让 Claude 把思考过程展开，往往能得到更稳的结果。

## 什么时候适合让 Claude 思考

如果任务本身包含隐藏依赖，或者涉及多个判断点，就值得用一步一步推理：

- 比较多个方案的取舍
- 评估提案或文档
- 解决数学或逻辑问题
- 设计带约束的工作流
- 在最终定稿前先做自检

这种方式的价值不只是提高准确率，还能让你更容易看出 Claude 是在哪一步理解偏了。

## 什么时候不必使用

思考过程会增加输出长度，也会带来额外时间开销。这不是问题，而是取舍。对于简单查询、快速改写、短事实问答，没有必要强行要求很长的推理过程。

下面这些场景就可以用更轻的提示：

- 只需要简短答案
- 任务本身很直接
- 你更在意速度而不是解释过程

换句话说，只有当任务真的需要“想一想”时，才让 Claude 想一想。

## 三种常用写法

### 1. 基础推理提示

```text
请先一步一步思考，再回答。请分析选项、说明取舍，最后给出建议。
```

这是最快的方式，但它不会告诉 Claude 应该如何组织分析。

### 2. 带步骤的推理提示

```text
请分三步分析这个提案：
1. 用一句话总结提案。
2. 找出主要风险和假设。
3. 给出建议，并明确说明是支持还是反对。
```

这种写法更适合重复使用，因为它规定了推理路径。

### 3. 使用 XML 结构化推理

```text
<instructions>
请仔细审查方案，并先思考关键风险。
</instructions>

<thinking>
按顺序列出主要考虑因素。
</thinking>

<answer>
用 3 个要点给出最终建议。
</answer>
```

...

---

**[👉 继续阅读全文：Claude 思考指南：什么时候应该让它一步一步推理](https://tools.cooconsbit.com/zh/articles/claude-let-claude-think-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
