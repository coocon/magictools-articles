# Claude Prompt Caching 指南：降低重复输入、成本和延迟

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-prompt-caching-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-prompt-caching-guide?utm_source=github&utm_medium=referral)**

Prompt Caching 是让 Claude 重复请求更便宜、更快的实用功能之一。Anthropic 官方文档将它描述为一种复用固定前缀的方法，例如系统指令、工具定义、示例和背景资料，而不用在每次请求里反复发送同一批内容。

这在你的工作流会不断重复同一套上下文时尤其有价值。如果你反复运行同一个 agent、同一套评分标准，或者同一组参考材料，Prompt Caching 可以减少大量无意义的输入 token，而且不会改变最终答案。

## Prompt Caching 适合什么场景

Prompt Caching 最适合那些前半段几乎不变的请求：

- 很少变化的系统指令
- 长期固定的工具定义
- 大段背景资料
- Few-shot 示例
- 可复用的参考文档

Anthropic 的功能总览里把 Prompt Caching 和批量处理、引用、文件支持放在一起说明，这本身就很能说明问题：它主要是面向 API 和生产流程的优化能力，而不是给普通聊天用户准备的小技巧。

## 基本原理

工作方式其实很直接：

1. 把稳定内容放在请求开头。
2. 用 `cache_control` 标记可复用部分的结束位置。
3. 后续请求沿用同样的前缀，让 Claude 复用缓存内容。

Anthropic 建议按固定顺序组织静态内容：`tools`、`system`、`messages`。系统会自动匹配最长前缀，所以通常不需要到处放缓存断点。

## 实际使用模式

你可以把请求拆成两部分：

1. 一部分是可复用的基础内容，比如角色说明、评分标准、示例或参考资料。
2. 另一部分是每次变化的任务，比如新的用户问题或新的文档。

例如，一个客服分类 agent 可以把角色指令、升级标准和输出格式放进缓存，而每次只替换工单内容。文档分析流程也可以先缓存参考资料，再针对不同问题反复查询。

...

---

**[👉 继续阅读全文：Claude Prompt Caching 指南：降低重复输入、成本和延迟](https://tools.cooconsbit.com/zh/articles/claude-prompt-caching-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
