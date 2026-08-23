# Claude 提示词基础：如何写出清晰直接的请求

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-clear-direct-prompts-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-clear-direct-prompts-guide?utm_source=github&utm_medium=referral)**

Anthropic 在官方提示词工程文档里给出的一个最重要建议，其实也是最容易被忽略的建议：要清晰、直接、具体。很多人觉得 Claude 回答得“太泛”，并不是模型本身不行，而是输入给它的任务边界太模糊。

Anthropic 常用一个很好理解的类比：把 Claude 想成一位非常聪明、但刚加入团队的新员工。它不会自动知道你的业务背景、表达偏好或结果标准，除非你把这些信息写出来。

## 为什么模糊提示词容易翻车

像“帮我把这段话写得更好一点”这种提示词，看起来很自然，但实际上留了太多空白：

- 是写给客户、老板，还是朋友？
- “更好”是更短、更正式，还是更有说服力？
- 输出是邮件、汇报、文案，还是报告？
- 有没有字数限制、结构要求或语气限制？

Claude 当然还是会给出一个答案，但它必须替你做很多猜测。而 Anthropic 官方文档的核心思路，就是尽量减少这种猜测空间。

## 一个清晰提示词通常包含什么

按照 Anthropic 的官方建议，效果更稳定的提示词通常包含三个部分：

1. **上下文**：这个任务处在什么场景里，给谁看，为什么要做。
2. **具体动作**：你希望 Claude 具体完成什么。
3. **输出约束**：格式、长度、风格、步骤或必须包含的部分。

少了这些内容，Claude 往往会给出“看起来合理但不够贴合场景”的答案。

## 一个更好的改写示例

下面是一个很弱的版本：

```text
帮我把这段周报改写得更好。
```

下面是一个更强的版本：

```text
请把下面这段周报改写给公司管理层阅读。

目标：语气简洁、明确、事实导向。
背景：这是发给高层的项目更新，他们最关心风险、交付时间和需要拍板的事项。
输出格式：
1. 一句话总体状态
2. 三个主要风险
3. 接下来一周的动作
约束：总字数不超过 180 字。

原始内容：
[在这里粘贴草稿]
```

...

---

**[👉 继续阅读全文：Claude 提示词基础：如何写出清晰直接的请求](https://tools.cooconsbit.com/zh/articles/claude-clear-direct-prompts-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
