# Stripe 70 亿收购 OpenRouter：买的是路由权

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/stripe-buys-openrouter?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/stripe-buys-openrouter?utm_source=github&utm_medium=referral)**

> 素材来源：Bloomberg 独家（2026-08-16）、TechCrunch 同日跟进、Lago wiki 深度分析《With OpenRouter, is Stripe becoming the Amazon of AI?》、Hacker News 讨论串 #49323381、OpenRouter 官方路由文档。文中引语均为原文原句。

---

支付公司历来是卖水的。这次它把水源买了。

2026 年 8 月 16 日，Bloomberg 独家报道 Stripe 已敲定以超过 70 亿美元收购 OpenRouter，TechCrunch 当天跟进同一数字，Stripe 发言人拒绝置评。三个月前，OpenRouter 刚以 13 亿美元估值融完 1.13 亿美元 B 轮，投资方包括 Sequoia、a16z、Menlo Ventures 和 Alphabet 旗下 Capital G。三个月，5 倍多。

最好笑的是这段话的出处顺序。OpenRouter 的 CEO Alex Atallah 今年早些时候自称公司是"AI 界的 Stripe"——一个账号、一套 API 接 400 多个模型，宣称 800 万用户。然后 Stripe 把他买了。HN 上有人一句话点破了这个荒诞：

> "Earlier this year, Atallah described OpenRouter as the AI equivalent of Stripe. Not sure I understand how this is strategically aligned for Stripe but certainly an interesting comparison."
> —— HN 用户 zacharyozer

（今年早些时候，Atallah 把 OpenRouter 描述成 AI 界的 Stripe。我不太理解这在战略上对 Stripe 意味着什么，但这个类比确实有意思。）

下面 10 条，试着把这个"有意思"拆开。

---

## 1. 70 亿买的不是转发请求的代码

> "The code that forwards a request is not worth $7 billion. The right to decide where a large and growing pool of requests goes might be."
> —— Lago wiki

（转发一个请求的代码不值 70 亿美元。决定一大笔持续增长的请求流向哪里的权利，可能值。）

这是理解整笔交易的第一把钥匙。OpenRouter 的技术栈不神秘：接住请求、挑一个供应商、转发、把响应吐回来。真要复刻，几个工程师加几个月。

但代码从来不是标的物。标的物是"谁来分配需求"。800 万用户的生产流量每天从这里过一遍，每一次过都要做一个决定：这次请求给谁做。这个决定权是可以定价的，而且随着流量增长会越来越贵。

Lago 的原话是"the beginnings of Amazon Marketplace for AI inference"——AI 推理版亚马逊市场的雏形。注意 beginnings，不是已经是。

**My take：** 所有觉得"70 亿买个 API 代理疯了"的人，都在给代码估值。Stripe 在给流量的调度权估值。两个完全不同的东西。你可以自己写一个路由器，你写不出 800 万个已经把生产流量指过来的用户。

---

## 2. 开发者用它，不是因为"AI 太差需要人帮忙选模型"

HN 上最高赞的嘲讽和最扎实的反驳挨在一起，值得原样放：

> "I still find it hilarious that AI is so bad you need something to sit in front of it to pick models for you. And that's a normal, accepted thing."
> —— HN 用户 muppetman

（我还是觉得很好笑，AI 差到需要在前面架一个东西替你选模型，而且这居然成了正常、被接受的事。）

> "Lots of providers use an “OpenAI-ish” API, but many of them have subtle differences in things like tool calling or thinking blocks. OpenRouter normalizes the wire format."
> —— HN 用户 bensyverson

（很多供应商用的是"类 OpenAI"的 API，但在 tool calling 或 thinking block 这类地方有微妙差异。OpenRouter 把线上格式统一了。）

还有一条更实在的，来自真实付费用户：

> "On OpenRouter, I can switch dollars between models. But for the providers, after evaluating I'm stuck with some number of credits on ones I don't use."
> —— HN 用户 arjie

（在 OpenRouter 上，我可以把钱在模型之间挪。但直接找供应商，评测完之后我就卡着一堆用不掉的额度。）

三句话拼出真相：OpenRouter 卖的不是"智能选模型"，是**格式归一化 + 额度可迁移 + 一张账单**。simonw 在同一串里也说了，自动模型路由"我还没看到多少证据说明它已经被广泛使用，我认为现在还更像是实验性机制"。

**My take：** 把 OpenRouter 理解成"模型选择器"，就会得出"70 亿是泡沫"的结论。把它理解成"token 的货币兑换所"，估值逻辑立刻不一样了——兑换所赚的从来不是汇率，是流量。

...

---

**[👉 继续阅读全文：Stripe 70 亿收购 OpenRouter：买的是路由权](https://tools.cooconsbit.com/zh/articles/stripe-buys-openrouter?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
