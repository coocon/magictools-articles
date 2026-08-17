---
title: "Stripe 70 亿美元买下 OpenRouter：买的不是模型路由，是 AI 经济的收银台"
slug: stripe-openrouter-acquisition-7b
summary: "据 Bloomberg 报道，Stripe 已就超 70 亿美元收购 OpenRouter 达成协议——三个月前它的估值还是 13 亿。本文基于 Bloomberg/WSJ/官方文档交叉核实：交易到底进行到哪一步、为什么说 Stripe 是在收购自己的计费客户、从 Bridge 到 Tempo 再到 OpenRouter 的收购叙事线，以及开发者现在最该做的一件事——把路由层抽象出来。附 BYOK 规则与五个替代方案对比。"
category: ai-tutorials
tags: [Stripe, OpenRouter, AI 收购, LLM API, 模型路由, BYOK, LiteLLM, AI Gateway]
coverImage: ""
status: published
locale: zh
source: authored
---

Stripe 收购 OpenRouter 的消息这两天刷屏，流传版本里有两处失真，先纠正：

1. **交易没有"完成"，是"据报道达成协议"**。Bloomberg 的原文措辞是 "has finalized an agreement...according to people familiar with the matter"（据知情人士，已敲定收购协议），并明确写了"最终价格仍可能变动"。Stripe 发言人对 TechCrunch 的回应是"不评论传闻与猜测"，OpenRouter 同样拒绝置评——**双方都没有官宣**。
2. **"此前估值不过数亿美元"是错的**。OpenRouter 在 2026 年 5 月刚完成 1.13 亿美元 B 轮（CapitalG 领投，NVIDIA、Snowflake、Databricks 等战投参与），媒体报道的投后估值约 13 亿美元。所以准确的说法是：**三个月，估值从 13 亿到 70 亿+，约 5.4 倍**。更有意思的反差是，《华尔街日报》7 月 23 日报道双方洽购时，谈的价码是约 100 亿美元——最终敲定的数字反而是谈低了的。

数字澄清完，说这笔交易真正值得开发者注意的地方。

## OpenRouter 是什么体量

如果你没用过：OpenRouter 用一个 OpenAI 兼容的统一 API 聚合了 400 多个模型，开发者接一次就能在 GPT、Claude、Gemini、DeepSeek、开源模型之间自由切换和比价。CEO Alex Atallah（OpenSea 联合创始人）在《纽约时报》DealBook 的采访里，把自家产品描述为 "the equivalent of Stripe for AI"——这句话在收购传出后被反复引用，成了最省事的注脚。

规模数据（Menlo Ventures 与官方口径）：800 万以上开发者用户；周处理 token 量 6 个月内从 5 万亿涨到 25 万亿；年化 token 运行速率约 1500 万亿。商业模式很简单：充值时抽平台费（非加密支付 5.5%、最低 0.8 美元），模型调用本身按厂商价格转售。

想直观感受"在 400 个模型里比价"是什么概念，可以看我们的 [LLM 价格对比工具](https://tools.cooconsbit.com/tools/llm-pricing)——OpenRouter 做的就是把这张表变成一个可编程的 API。

## 关键事实：Stripe 收购的是自己的计费客户

这是多数快讯没提的一层：**Stripe 本来就是 OpenRouter 的支付和计费服务商**。Stripe 官网新闻室有专门的客户案例页，OpenRouter 在用 Stripe Invoicing（账单）、Stripe Tax（全球税务）和 Radar（风控），法币收单全部走 Stripe（加密货币走 Coinbase）。

所以这不是一次陌生并购，是"供应商买下客户"：Stripe 看着 OpenRouter 的真实流水做的决策。HN 上有用户指出另一个时间巧合——OpenAI 近期把支付服务商从 Stripe 换成了 Adyen。AI 相关的支付量正在成为支付公司必争的增量盘子，把 token 结算的源头买下来，比只做通道更彻底。

把 Stripe 近 18 个月的动作连起来看，叙事线非常完整：

- **2025-02**：11 亿美元收购稳定币基础设施公司 Bridge（当时是 Stripe 史上最大收购）
- **2025-06**：收购嵌入式加密钱包公司 Privy
- **2025-09**：与 OpenAI 发布 Agentic Commerce Protocol，给 ChatGPT 的 Instant Checkout 提供支付
- **2026-03**：支付专用区块链 Tempo 主网上线，同步发布 Machine Payments Protocol，让 AI agent 无需逐笔人工批准即可自主付款
- **2026 年初**：收购用量计费公司 Metronome（现已是 Stripe 官网正式产品线）
- **2026-08**：据报道 70 亿+ 美元收购 OpenRouter

钱的轨道（Bridge）→ 钱包（Privy）→ agent 怎么付钱（ACP/Tempo/MPP）→ 按用量怎么算钱（Metronome）→ **token 本身的计量、路由与结算（OpenRouter）**。Stripe 在把"AI 经济的金融基础设施"整条买齐，OpenRouter 是这一串里最贵的一笔。

## HN 吵成了两派

主帖 246 条评论，观点大致两派：

**看多派**认为这是天作之合：Stripe 是全球最好的 API 公司之一，抽象过支付轨道，现在来抽象 LLM 轨道；token 本质上是一种轻量的高价值计费资产，Stripe 最懂在特性差异巨大的多条"轨道"之间做路由和清结算。收购后，任何按用量计费的 AI 产品都可能拿到"计量 + 计费 + 抽成"的一站式基础设施。

**看空派**集中在两点。一是估值：一个 API 中间商凭什么值 70 亿——比 Lyft、Dolby 的市值还高，且"OpenRouter 的技术远比 Stripe 的核心资产（风控 + 全球银行集成）容易复制"。二是开发者切身利益：涨价、免费额度消失（"RIP free DeepSeek access"）、审查收紧、以及把最敏感的 prompt 数据交给一家支付巨头的顾虑。

值得注意的是，两派对一个事实没有分歧：**OpenRouter 的 API 形态决定了迁移成本很低**。这既是它护城河薄的原因，也是开发者的自救通道。

## 开发者现在该做什么

**第一件事：把路由层抽象出来。** 收购后现有 API 和价格是否维持，目前没有任何公开承诺（Bloomberg/TechCrunch/Fortune 均未提及，双方拒绝评论）。历史经验反复证明，"免费好用的 API 被收购"之后的剧本大多不好看。如果你的代码里到处硬编码 `openrouter.ai/api/v1`，现在就值得包一层。

**了解 BYOK 规则，它是你的对冲工具。** OpenRouter 支持自带厂商 API key：走 BYOK 的请求按"该请求正常价格的 5%"收平台费，Pay-as-you-go 档每月含 2.5 万美元的 BYOK 免费额度。真到了要迁移的那天，BYOK 意味着你的厂商侧账户和限额是现成的，切换只剩改 endpoint。

**替代方案心里有数即可，不必今天就换**：

| 方案 | 一句话定位 |
|------|-----------|
| LiteLLM | 开源可自托管的统一网关，100+ 模型 OpenAI 格式（注意近期曝过重大安全漏洞，自托管要盯更新） |
| Portkey | 商业 AI Gateway，主打可观测性、guardrails 和企业治理 |
| Cloudflare AI Gateway | 边缘代理层：缓存、限流、分析、回退，不做计费市场 |
| Vercel AI Gateway | 绑定 Vercel 生态的模型路由 + 统一计费 |
| AWS Bedrock / GCP Vertex | 云厂商托管路线，合规强，但模型上新慢、限流问题多（HN 实测吐槽不少） |

另外，如果你比价的目的是决定自托管还是走 API，算清显存账再做决定——[LLM 显存计算器](https://tools.cooconsbit.com/tools/llm-vram-calculator) 可以帮你估算目标模型在本地跑起来要多少卡。

## 常见问题 FAQ

### Stripe 收购 OpenRouter 已经完成了吗？

没有。截至 2026 年 8 月 17 日，状态是"据知情人士已敲定协议"（Bloomberg 报道），最终价格可能变动，Stripe 与 OpenRouter 双方均拒绝评论，未官宣、更未交割。

### 收购后 OpenRouter 的 API 会涨价或停用吗？

目前没有任何公开承诺或声明。短期内 API 兼容大概率维持（迁移成本低反而是约束双方的因素），但长期定价、免费额度、分成模式都存在调整可能。建议把路由层抽象出来，保留多平台切换能力。

### OpenRouter 值 70 亿美元吗？

这正是 HN 上争议最大的点：三个月前 B 轮估值约 13 亿美元，70 亿+ 是 5.4 倍溢价；但《华尔街日报》7 月报道的洽购价码是约 100 亿，最终数字低于早期传闻。看多的逻辑是买年化 1500 万亿 token 的结算入口，看空的逻辑是路由技术本身护城河很薄。

### 有哪些 OpenRouter 的替代品？

开源自托管首选 LiteLLM；要企业治理看 Portkey；已在 Cloudflare/Vercel 生态的可以用各自的 AI Gateway；合规优先走 AWS Bedrock 或 GCP Vertex。OpenRouter 的 OpenAI 兼容 API 形态决定了迁移成本不高。

## 参考链接

- [Stripe Nears Deal to Buy AI Firm OpenRouter for Over $7 Billion — Bloomberg](https://www.bloomberg.com/news/articles/2026-08-16/stripe-nears-deal-to-buy-ai-firm-openrouter-for-over-7-billion)
- [Stripe will reportedly acquire AI gateway startup OpenRouter for $7B+ — TechCrunch](https://techcrunch.com/2026/08/16/stripe-will-reportedly-acquire-ai-gateway-startup-openrouter-for-7b/)
- [Stripe in Talks to Buy Buzzy AI Model Marketplace OpenRouter — WSJ（2026-07-23）](https://www.wsj.com/tech/ai/stripe-in-talks-to-buy-buzzy-ai-model-marketplace-openrouter-decc6a74)
- [OpenRouter Series B 官方公告](https://openrouter.ai/announcements/series-b)
- [Stripe powers OpenRouter's global AI model access — Stripe Newsroom](https://stripe.com/newsroom/news/openrouter-and-stripe)
- [OpenRouter now processes more than a quadrillion tokens a year — Menlo Ventures](https://menlovc.com/perspective/openrouter-now-processes-more-than-a-quadrillion-tokens-a-year/)
- [OpenRouter BYOK 官方文档](https://openrouter.ai/docs/use-cases/byok)
- [Hacker News 讨论（246 评论）](https://news.ycombinator.com/item?id=49323381)
