# DeepSeek 标称 1M、Claude Code 只认 200K：两个都测了，真正卡住你的是第三个数

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-deepseek-1m-context?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-deepseek-1m-context?utm_source=github&utm_medium=referral)**

DeepSeek [价格页](https://api-docs.deepseek.com/quick_start/pricing)给 `deepseek-flash` 和 `deepseek-v4-pro` 都标了 **1M 上下文**。把 Claude Code 接到 DeepSeek 的 Anthropic 兼容端点上（三个环境变量，接法和[它的计费坑](/zh/hands-on/claude-code-deepseek-backend)我另写了一篇），Claude Code 对同一个模型报告的是：

```json
"contextWindow": 200000
```

两个数字差 5 倍，描述同一个模型。谁说了算？

我测了。答案是**两个都不算**，而且失效的方式比这两个数字本身更有意思。

## 测了哪三层

三个经常被混为一谈的问题：

1. **端点收不收？**——API 的硬上限。
2. **模型用不用得上？**——能检索到，不只是没报错。
3. **Claude Code 让不让你到那儿？**——没人写过的客户端闸门。

全部条件：Claude Code **2.1.270**、`deepseek-flash`、macOS、独立 `CLAUDE_CONFIG_DIR`。大海捞针的针放在**第 0 位**（海量上下文里最难捞的位置），问题放在最末尾。每个请求带唯一 nonce 破缓存——这个实验的第一版没加，结果两个体量差一倍的负载报出了几乎相同的 token 数。

## 第一层：DeepSeek 的真实上限是 2^20，不是「一百万」

裸端点阶梯，针在第 0 位：

| 目标 | 真实 input token | HTTP | 耗时 | 捞到针 |
|---|---|---|---|---|
| 200K | 399,954 | 200 | 8.6s | ✅ |
| 400K | 799,823 | 200 | 15.8s | ✅ |
| 950K | 949,775 | 200 | 19.8s | ✅ |
| 1040K | **1,039,744** | 200 | 19.4s | ✅ |
| 1048K | — | **400** | 3.7s | — |

**1,039,744 token** 时，模型仍能把开头第一行的字符串准确取出，用时不到 20 秒。这个长上下文是真的，不是参数表上的数字。

被拒那次直接给出了上限：

```
This model's maximum context length is 1048576 tokens.
```

**1,048,576 = 2^20。** 所以这个「1M」是二进制的，比字面一百万多出 48,576 个 token。

![裸端点阶梯：1,039,744 token 通过且捞到针，1,048,576 是硬上限](https://cdn.tools.cooconsbit.com/uploads/articles/2026-09-20-deepseek-1m-context/01-ladder.png)

### 这个额度包含你的输出预算

下单之前得知道。固定输入，只改 `max_tokens`：

| 输入 token | `max_tokens` | 结果 |
|---|---|---|
| 1,045,710 | 1,000 | ✅ 200 |
| 1,045,707 | 8,000 | ❌ 400 |

报错自己把账算给你看了：

> you requested 1053707 tokens (1045707 in the messages, 8000 in the completion)

所以预算是 **输入 + 输出 ≤ 1,048,576**。在接近满窗时给一个慷慨的 `max_tokens`，会让本来没问题的输入被拒。

![同输入只改 max_tokens：1,048,576 的额度含输入与输出之和](https://cdn.tools.cooconsbit.com/uploads/articles/2026-09-20-deepseek-1m-context/02-budget.png)

...

---

**[👉 继续阅读全文：DeepSeek 标称 1M、Claude Code 只认 200K：两个都测了，真正卡住你的是第三个数](https://tools.cooconsbit.com/zh/articles/claude-code-deepseek-1m-context?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
