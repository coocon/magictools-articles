# 不是 OpenAI 降价：GPT-5.6 Sol 的五折写着 9 月 18 日到期，还不包 BYOK

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/gpt-5-6-sol-openrouter-half-price-expiry?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/gpt-5-6-sol-openrouter-half-price-expiry?utm_source=github&utm_medium=referral)**

先把好消息说完：**降价是真的，幅度也确实是五折。**

GPT-5.6 Sol 在 OpenRouter 上的价格，从 $5 / $30（每百万 token 输入 / 输出）降到了 **$2.50 / $15**。我用 OpenRouter 的公开 API 拉了原始数据，不是看官网截图，数字对得上。

但这条消息在传播过程中掉了三个限定词，每一个都会直接影响你的账单：

1. **降价的不是 OpenAI**，是 OpenRouter 和 Vercel AI Gateway 两个平台
2. **有到期日**：9 月 18 日
3. **自带 key（BYOK）不享受**

如果你正准备把生产流量切过去，这三条比「砍半」本身重要。

---

## 一、准确价格：一张表，含两个容易踩的坑

OpenRouter API 快照，抓取时间 **2026-08-19 00:50 UTC**：

| 计费项 | 促销价 | 原价 |
|--------|--------|------|
| Input | **$2.50** | $5.00 |
| Output | **$15.00** | $30.00 |
| Cache read | $0.25 | $0.50 |
| Cache write | $3.125 | — |
| Batch / Flex 档 | $1.25 / $7.50 | $2.50 / $15 |
| **长上下文档（prompt > 272K tokens）** | **$5.00 / $22.50** | — |
| Web search | $10 / 1000 次调用 | 未降价 |

**坑一：272K 的价格断点。** prompt 超过 272,000 tokens 之后，input 单价直接翻倍到 $5、output 涨到 $22.50。也就是说，**在长上下文档位上，你付的是没打折的原价**。Sol 的上下文窗口有 105 万 token，如果你的用法是「把整个代码库塞进去」，这一档才是你的真实单价，促销基本与你无关。做成本模型时务必把这个断点算进去。

**坑二：它是推理模型，reasoning token 按 output 计费。** Sol 支持 `reasoning_effort` 参数，档位从 medium 到 ultra（ultra 默认跑 4 个 agent 并行）。输出单价是输入的 6 倍，而推理 token 全部按输出计价——**档位调高的成本代价是非线性的**。这一点和我们之前写[Qwen3.8 27B 默认档跑 21 分钟](/articles/qwen3-8-27b-overthinking)是同一类问题，只不过那边烧的是你的时间，这边烧的是你的钱。

原价 $5/$30 有三个独立佐证：OpenAI 官方发布页明文写着 "Sol is $5 input / $30 output"；OpenRouter 模型页上是划线价；同一页面上 Azure provider 至今仍挂 $5/$30 没降。

## 二、降的是谁的价

这是最关键的更正。

- **生效时间**：2026-08-17（OpenRouter 公告推文时间戳 17:36 UTC）
- **有效期至**：**2026-09-18**，之后恢复原价
- **范围**：仅 OpenRouter + Vercel AI Gateway；仅路由到 OpenAI provider 的请求；**仅非 BYOK 请求**
- **OpenAI 官方 API 价格没动**，至今仍是 $5/$30。Azure、Bedrock 都没跟进

作为对照，7 月 30 日那次才是 OpenAI 的**官方**降价：Luna 降 80%（$1/$6 → $0.20/$1.20）、Terra 降 20%（$2.50/$15 → $2/$12）。**那次 Sol 一分没降。** 两件事性质完全不同：一次是厂商调整定价，一次是渠道方做促销。

顺带纠正一个常见误写：**不存在「GPT-5.6 标准版」**。这一代的产品线就是 Sol / Terra / Luna 三档——按 OpenAI 的说法，"数字代表世代，Sol/Terra/Luna 是可独立演进的持久能力层级"。Sol 是旗舰，Terra 均衡，Luna 最便宜最快。拉丁语的太阳、大地、月亮。

## 三、SemiAnalysis 的质疑：这次促销可能不是给你的

这是我认为最值得单独说的一段。

SemiAnalysis 在 X 上提出：OpenRouter 和 Vercel 在 OpenAI 总用量里占比微不足道，**但它们恰恰是业界估算各家模型市场份额的主要公开数据源**——因为 OpenAI 自己不公布这类数字。

...

---

**[👉 继续阅读全文：不是 OpenAI 降价：GPT-5.6 Sol 的五折写着 9 月 18 日到期，还不包 BYOK](https://tools.cooconsbit.com/zh/articles/gpt-5-6-sol-openrouter-half-price-expiry?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
