# Claude Prompt Caching 指南：降低重复输入、成本和延迟

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-prompt-caching-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-prompt-caching-guide?utm_source=github&utm_medium=referral)**

Prompt Caching 是让 Claude 重复请求更便宜、更快的实用功能之一。Anthropic 官方文档将它描述为一种复用固定前缀的方法，例如系统指令、工具定义、示例和背景资料，而不用在每次请求里反复发送同一批内容。

这在你的工作流会不断重复同一套上下文时尤其有价值。如果你反复运行同一个 agent、同一套评分标准，或者同一组参考材料，Prompt Caching 可以减少大量无意义的输入 token，而且不会改变最终答案。

这篇文章原本只讲原理。2026-09-10 我补了文末的「本机实测」一节：真调 API，同一段 28K token 的 system 前缀连打 21 次，把 `usage` 里的三个 cache 字段、端到端延迟、首 token 延迟和按官方定价换算的账单全部读出来。先说结论：**成本收益是量级的（命中后单次 $0.0719 → $0.0058），延迟收益在这个规模下只有一百多毫秒。**

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

## 需要注意的限制

Prompt Caching 并不是所有请求都能直接套用的快捷方式。Anthropic 明确提到了一些限制：

- 提示词必须足够长，才能进入可缓存范围。
- 命中的前缀必须完全一致。
- 缓存按组织隔离。
- 空文本块不能缓存。
- Thinking blocks 不能直接缓存。

还有 TTL 的问题。Anthropic 支持默认的短缓存，以及部分场景下的一小时缓存。如果你要做生产流程，最好在依赖某个模型或缓存时长之前，先确认当前支持范围。

## 最适合缓存的工作流

Prompt Caching 在这些场景里通常效果最好：

- 多步骤 agent，反复复用同一套说明
- 固定评分标准的内部工具
- 大段上下文分析，而且参考材料变化不频繁
- 使用相同设置的批处理任务

如果每次请求都完全不同，缓存价值就很有限。系统指令、示例和上下文都变来变去时，能复用的部分很少。

## 常见错误

最常见的错误很简单：

- 缓存变化频繁的内容
- 以为改一点提示词措辞还能命中同一缓存
- 只缓存了一小段真正可复用的内容
- 把 Prompt Caching 当成提示词设计的替代品

只有“稳定内容真的稳定”时，缓存才有意义。提示词本身如果很乱，缓存也只是让混乱变便宜。

## 一个简单判断

如果你发现自己在很多 Claude 请求里反复复制同样的说明、示例或参考材料，那这部分内容大概率就适合缓存。

## 本机实测：同一段 63KB 的 system 前缀，连打 21 次

### 问题背景

上面的原理我写了半年，但一直没亲眼看过 `usage` 里的 `cache_read_input_tokens` 到底长什么样，也没算过「省 90%」落到一次真实请求上是几分钱。更要紧的是官方文档把「更便宜」和「更快」并列，这两个收益在我的环境里是不是同一个量级，没人替我测。

### 问题分析

要把「缓存生效」坐实，三件事必须分开量：

1. **命中证据**只有一处：响应 `usage` 里的 `input_tokens` / `cache_creation_input_tokens` / `cache_read_input_tokens`。第一次应该是 creation 有数、read 为 0；第二次反过来。没有这两个字段，其他一切都是猜。
2. **延迟**要剥离。端到端时间里混着网络往返、中间代理、排队和生成，缓存只影响 prefill 那一段。所以除了端到端，还得用流式请求量首 token 时间，并且拿一个没有长前缀的极小请求当基线。
3. **成本**按官方定价页换算：Claude Sonnet 5 输入 $2 / MTok，5 分钟缓存写入 $2.50（1.25 倍），缓存命中 $0.20（0.1 倍），输出 $10（[定价页](https://platform.claude.com/docs/en/about-claude/pricing)，2026-09-10 核对）。

...

---

**[👉 继续阅读全文：Claude Prompt Caching 指南：降低重复输入、成本和延迟](https://tools.cooconsbit.com/zh/articles/claude-prompt-caching-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
