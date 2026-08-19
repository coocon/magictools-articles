---
title: "不是 OpenAI 降价：GPT-5.6 Sol 的五折写着 9 月 18 日到期，还不包 BYOK"
slug: gpt-5-6-sol-openrouter-half-price-expiry
summary: "GPT-5.6 Sol 在 OpenRouter 上从 $5/$30 降到 $2.50/$15，「砍半」属实，但传播版本漏了三件事：这是 OpenRouter 和 Vercel AI Gateway 的平台侧促销，OpenAI 官方定价页至今仍是 $5/$30；促销 9 月 18 日到期；自带 key（BYOK）的请求不享受。SemiAnalysis 还提出了一个更尖的质疑——这两个平台占 OpenAI 用量微不足道，却恰好是外界估算模型市场份额的主要公开数据源。本文给出 17 个模型同口径价格表、272K 长上下文的加价陷阱，以及一份「要不要切」的决策清单。"
category: ai-tutorials
tags: [GPT-5.6, OpenRouter, API 成本, 模型选型, OpenAI, LLM 定价, Claude, Kimi K3]
coverImage: ""
status: published
locale: zh
source: authored
translationSlug: gpt-5-6-sol-openrouter-half-price-expiry-en
---

# 不是 OpenAI 降价：GPT-5.6 Sol 的五折写着 9 月 18 日到期，还不包 BYOK

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

在这两个平台上打五折，成本上几乎无关痛痒，却能让 Sol 在这些公开榜单上的成交量翻倍。而这些榜单正被拿来判断 OpenAI 和 Anthropic 的竞争格局。

这个质疑无法被证实（OpenAI 和两个平台都没有说明促销动机），我原样标注为第三方分析。但它给出的读法是可操作的：**接下来几周你会看到的「Sol 份额暴涨」类图表，需要打个折扣再读。** 促销期结束后回看同一条曲线会更有信息量。

我们写[Stripe 70 亿收购 OpenRouter](/articles/stripe-buys-openrouter) 时的判断，在这里得到了一个具体注脚：路由层的价值不只在于转发请求，还在于**它是这个行业为数不多的公开计分板**。谁在计分板上做促销，谁就在影响记分。

## 四、同口径横向价格表

全部来自 OpenRouter API 同一次快照（2026-08-19 00:50 UTC），单位 $/1M tokens。同口径比较，避免各家官网的计价方式差异：

| 模型 | Input | Output | 上下文 |
|------|-------|--------|--------|
| **GPT-5.6 Sol（促销）** | **2.50** | **15.00** | 1.05M |
| GPT-5.6 Sol（9-18 后） | 5.00 | 30.00 | 1.05M |
| GPT-5.6 Terra | 2.00 | 12.00 | 1.05M |
| GPT-5.6 Luna | 0.20 | 1.20 | 1.05M |
| Claude Opus 5 | 5.00 | 25.00 | 1M |
| Claude Sonnet 5 | 2.00 | 10.00 | 1M |
| Claude Haiku 4.5 | 1.00 | 5.00 | 200K |
| Gemini 3.1 Pro Preview | 2.00 | 12.00 | 1.05M |
| Gemini 3.7 Flash | 0.375 | 1.875 | 1.05M |
| DeepSeek V4 Pro | 0.66 | 1.98 | 1.05M |
| DeepSeek V4 Flash | 0.083 | 0.165 | 1.05M |
| Qwen3.5 Plus | 0.30 | 1.80 | 1M |
| Qwen3 Max | 0.78 | 3.90 | 262K |
| Kimi K3 | 3.00 | 15.00 | 1.05M |
| GLM-5.3 | 1.40 | 4.40 | 1.05M |
| Grok 4.6 | 2.00 | 6.00 | 500K |
| MiniMax M3 | 0.30 | 1.20 | 1.05M |

三个值得注意的位置关系：

**促销价把 Sol 放到了 Kimi K3 旁边。** $2.50/$15 对 $3/$15——输出单价完全一样。HN 上已经有人点破这个巧合。考虑到那条讨论串（614 分）的主线叙事就是「中国开源模型逼出美国大厂降价」，这个价位落点很难说是偶然。

**它仍然比 Claude Sonnet 5 的输出贵 50%。** $15 对 $10。促销价下的旗舰，输出仍比对手的中档贵一半——这是「五折」这个词容易盖住的事实。

**9 月 18 日之后，它回到 Opus 5 之上。** $5/$30 对 $5/$25。到期日一过，Sol 就是这张表上输出最贵的模型。

## 五、性价比佐证（注意来源）

OpenAI 发布页引用 Artificial Analysis 的数字（**这是官方引用，我没能独立复核 AA 原始图表，其数据在 JS 图表里**）：

- **Intelligence Index v4.1**：Sol 58.9 / Claude Fable 5 59.9 / Opus 4.8 55.7 / GPT-5.5 54.8 / Terra 55 / Luna 51.2 / Gemini 3.1 Pro Preview 46.5
- **Coding Agent Index v1.1**：Sol 80（超过 Fable 5 的 77.2），且 output token 用量不到一半
- Agents' Last Exam：Sol 53.6 —— 注意 OpenAI 同一页的正文和表格数字不一致（53.6 vs 52.7%），引用时留个心眼

简单说：Sol 比 Fable 5 低约 1 分，但 OpenAI 声称耗时少 61%、成本约一半。在 coding agent 这个细分上它是自称 SOTA。

速度方面，OpenRouter 平台侧的监测数据（2026-08-19 抓取，这个是第三方实测）：

| Provider | P50 首响 | 吞吐 | 价格 |
|----------|---------|------|------|
| OpenAI | 4.31s | 41 tps | $2.50/$15（促销） |
| Azure | 3.46s | 42 tps | $5.00/$30 |
| **Bedrock** | **2.58s** | **100 tps** | $5.50/$33 |

Bedrock 快 2.4 倍，贵 2.2 倍。**如果你的场景是交互式的（用户在等），这个取舍值得单独算一次**——不是所有场景都该无脑选便宜的那条路由。

## 六、要不要切：一份操作清单

**如果决定切，这几件事必须做对：**

1. **把 provider 钉死。** 想吃到折扣，请求必须路由到 OpenAI provider 且不用 BYOK。默认的 Balanced 路由通常会命中折扣档，但保险做法是显式指定 `provider: {order: ["openai"]}`。同一个模型在 OpenRouter 上有 3 家 provider、7 个 endpoint，价格从 $2.50 到 $5.50 都有。
2. **BYOK 不要开。** 自带 key 的请求按官方原价计费，等于白白放弃折扣。（BYOK 另有每月 $25,000 列表价的免费额度，超出收 5%——这是另一套账，别混算。）
3. **把 5.5% 算进去。** OpenRouter 推理价格零加价，但充值收 5.5% 手续费（最低 $0.80），加密货币充值 5%。促销期内走 OpenRouter 相比直连 OpenAI 净省约 47%，不是 50%。
4. **9 月 18 日设个日历提醒。** 到期后两边价格持平，OpenRouter 只剩多 provider 容灾和统一接口的价值。**别把促销价写进任何长期成本模型。**
5. **检查你的 prompt 长度分布。** 超过 272K 的那部分请求，享受不到促销。

**降价之后你大概率会用得更多，这不一定是好事。** TD Cowen 统计了 7 月 30 日那次官方降价的后果：Luna 降价 80% 之后用量涨了约 14 倍、收入反而增长约 34%；Terra 用量涨约 5 倍、收入增约 45%。典型的杰文斯悖论。数据窗口只有两周左右，别当定论，但方向很清楚——**便宜会诱发新增用量，你的账单未必因为单价减半就减半。**

**社区口碑**（HN 614 分那条讨论串，仅供参考）：Sol 被普遍认为编码和 agentic 任务强、拒答少（有多位用户因为 Claude 的安全拦截误伤而迁移过来）；也有人认为长任务上 Fable 5 仍略好，Sol 「容易在细节上钻牛角尖」。截至抓取时，**没有发现降价后限流或质量下降的报告**。

## 七、我的解读

**第一，把「谁在降价」和「降多久」当成价格新闻的两个必填字段。** 这次的三个限定词（平台侧、有到期日、排除 BYOK）任何一个漏掉，你的成本模型都会算错。渠道促销和厂商降价是两件事：前者随时可以结束，后者通常意味着成本结构真的变了。7 月 30 日 Luna 降 80% 是后者，这次是前者。

**第二，真正该做的不是切模型，是让切模型变便宜。** 一个 30 天的促销窗口值不值得你改代码？如果模型选择是硬编码在业务逻辑里的，答案基本是不值得——你会在 9 月 18 日再改一次。如果它是一行配置，那这次促销就是白捡的。我们在[同一个提示词，11 个模型差出 200 倍](/articles/one-prompt-11-models)里算过选型的账，结论到今天没变：**在这个价格每月都在动的市场里，「换模型的成本」比「模型的价格」更值得你优化。**

**第三，先跑真实账单，再看宣传数字。** 如果手头有批量任务，趁促销期切一部分流量跑两天，用你自己的 token 分布和 reasoning 档位实测省了多少。官方的 benchmark 分数和「成本约一半」的说法，都建立在别人的负载上。你的 prompt 有多长、输出有多长、要不要开高档推理，这三件事对账单的影响远大于榜单排名。

**第四，别把「和 Kimi K3 同价」读成巧合。** 这张表上最有信息量的不是某一行，是行与行之间的距离在快速收敛。开源旗舰把闭源旗舰的价格锚在了 $15 输出这条线上，而这条线三个月前还不存在。促销会结束，这个压力不会。

---

**参考链接**

- [OpenRouter: GPT-5.6 Sol 模型页](https://openrouter.ai/openai/gpt-5.6-sol)
- [OpenRouter Models API（本文价格数据源）](https://openrouter.ai/api/v1/models)
- [OpenAI 官方发布页（$5/$30 定价、命名解释、benchmark）](https://openai.com/index/gpt-5-6/)
- [OpenRouter 公告推文（2026-08-17）](https://x.com/OpenRouter/status/2089406144297214339)
- [OpenRouter 费用 FAQ（5.5% 充值费、BYOK 规则）](https://openrouter.ai/docs/faq)
- [BigGo Finance 分析（促销期限、SemiAnalysis 质疑、TD Cowen 数据）](https://finance.biggo.com/news/c4170767-79dc-4f18-862a-95107ffe6fd5)
- [HN 讨论（614 分）](https://news.ycombinator.com/item?id=49337602)
- 站内相关：[Stripe 70 亿收购 OpenRouter：买的是路由权](/articles/stripe-buys-openrouter) · [同一个提示词，11 个模型差出 200 倍](/articles/one-prompt-11-models) · [当 API token 卖到官方价的 3%](/articles/llm-token-relay-market-anatomy)
