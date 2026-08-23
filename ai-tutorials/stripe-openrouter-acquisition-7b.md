# Stripe 70 亿美元买下 OpenRouter：买的不是模型路由，是 AI 经济的收银台

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/stripe-openrouter-acquisition-7b?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/stripe-openrouter-acquisition-7b?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：Stripe 70 亿美元买下 OpenRouter：买的不是模型路由，是 AI 经济的收银台](https://tools.cooconsbit.com/zh/articles/stripe-openrouter-acquisition-7b?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
