---
title: "Stripe 70 亿收购 OpenRouter：买的是路由权"
summary: "Stripe 敲定超 70 亿美元收购 OpenRouter，这笔钱买的不是转发请求的代码，而是决定海量推理请求流向哪家供应商的权力。但它买到了亚马逊的位置，没买到亚马逊的锁定。"
tags:
  - stripe
  - openrouter
  - ai-infrastructure
  - llm
  - acquisition
slug: stripe-buys-openrouter
category: ai-tutorials
coverImage: ""
status: published
locale: zh
source: authored
translationSlug: stripe-buys-openrouter-en
---

# Stripe 70 亿收购 OpenRouter：买的是路由权

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

---

## 3. 默认路由是一套分销系统，不是一段转发逻辑

OpenRouter 官方文档白纸黑字写着默认策略：

> "Prioritize providers that have not seen significant outages in the last 30 seconds."
> "For the stable providers, look at the lowest-cost candidates and select one weighted by inverse square of the price (example below)."
> —— OpenRouter 官方路由文档

（优先选择最近 30 秒没有明显故障的供应商。在稳定的供应商里，看成本最低的候选，按价格的反平方加权选一个。）

官方自己举的例子：$1/M token 的 Provider A，比 $3/M 的 Provider C 高 9 倍概率拿到第一个请求（1/3² = 1/9）。

这是价格战的自动化。供应商降价，流量立刻涨；供应商抖一下，30 秒内被踢出候选。Lago 那句话说得更狠：

> "It can lose demand without a developer ever making an explicit decision to switch."
> —— Lago wiki

（一个供应商可能在没有任何开发者主动决定换供应商的情况下，丢掉需求。）

**My take：** 这就是"网关"这个词的欺骗性。网关听起来中立、被动、技术性。但一套决定 70 多家供应商谁吃饱谁挨饿的加权函数，本质是渠道。Google 的排序算法也从来没自称过渠道。

---

## 4. 网关变市场，靠的是聚客再控路

> "Amazon Marketplace did not become powerful because listing products online was difficult. It gathered buyers in one place, then controlled how merchants reached them. Search ranking and the Buy Box could matter as much as the seller's product."
> —— Lago wiki

（亚马逊市场之所以强大，不是因为把商品挂上网很难。它先把买家聚到一个地方，然后控制商家如何触达这些买家。搜索排名和 Buy Box 的重要性可以不亚于卖家的商品本身。）

Buy Box 是亚马逊那个"加入购物车"按钮背后的默认卖家资格。同一件商品十几个卖家，拿到 Buy Box 的那个吃掉绝大部分成交。亚马逊从不需要禁止别人卖，它只需要决定谁是默认。

OpenRouter 的默认路由就是推理界的 Buy Box。Lago 的判断很克制：**它还没走到那一步**（"OpenRouter is not there yet"），但准入、测评、分配需求这三件事它已经全在做了。

**My take：** 市场形成的临界点不是技术，是"多少人接受默认值"。这一点对做 SaaS 的人都是常识：默认值就是权力。区别在于，OpenRouter 的默认值直接换算成 70 多家推理公司的营收曲线。

---

## 5. Stripe 真正想缝上的，是成本侧和收入侧之间那道缝

> "Today Stripe can see a customer payment. OpenRouter can see which model was requested, which provider served it and what that inference cost. Put the two together and Stripe can connect the cost of producing an AI feature with the revenue earned from selling it."
> —— Lago wiki

（今天 Stripe 能看到一笔客户付款。OpenRouter 能看到请求了哪个模型、哪家供应商接的单、这次推理花了多少钱。把两者拼起来，Stripe 就能把生产一个 AI 功能的成本，和卖它赚到的收入连上。）

为什么这件事在 AI 时代突然值钱？因为 AI 产品的毛利极度不稳定。Lago 列了变量：模型、上下文长度、缓存命中、重试、选中的供应商——两个看起来一样的用户操作，成本可以差出好几倍。

传统 SaaS 不需要这层。一个用户点一次按钮的边际成本约等于零，账单是纯收入侧的事。AI SaaS 每一次点击都在真实烧钱，收入侧和成本侧必须对齐，否则你不知道自己在卖亏还是卖赚。

**My take：** 这才是 Stripe 视角里最硬的那块逻辑。它做了十几年"收入侧"，现在 AI 把"成本侧"变成了一个需要实时计量的东西。谁能同时看到两边，谁就能卖计量、卖信用额度、卖账单、卖税务、卖风控——在成本发生的那一刻就地分发。

---

## 6. "为什么 Stripe 不自己写一个"

HN 上问得最直接的一条：

> "What I'm not so sure about is their moat. They have the brand recognition, but it seems rather easy for someone else to build what they've built. I'm a little confused about why Stripe didn't just build the same thing internally."
> —— HN 用户 Aurornis

（我不太确定他们的护城河在哪。他们有品牌认知，但别人要造出他们造的东西似乎相当容易。我有点困惑为什么 Stripe 不干脆内部自己做一个。）

Lago 的回答是这篇分析里最锋利的一句：

> "Stripe could build an AI router for far less than $7 billion. It could not quickly reproduce OpenRouter's provider relationships or persuade 8 million users to move production traffic through a new one."
> —— Lago wiki

（Stripe 花远少于 70 亿的钱就能造一个 AI 路由器。但它没法快速复制 OpenRouter 的供应商关系，也没法说服 800 万用户把生产流量搬到一个新的上面去。）

**My take：** 自建 vs 收购的判断标准从来不是"能不能造出来"，是"造出来之后能不能让人搬家"。前者是工程问题，后者是时间问题，而时间在 AI 这条曲线上是唯一买不到的东西。溢价买的是那两年。

---

## 7. Metronome、PayPal、OpenRouter，是同一张图的三块

> "PayPal would bring merchants and consumers. OpenRouter brings developers and inference providers. Both are networks in which several parties need money moved, reconciled and monetized."
> —— Lago wiki

（PayPal 会带来商家和消费者。OpenRouter 带来开发者和推理供应商。两者都是多方需要转账、对账和变现的网络。）

再加上第三块：Stripe 此前以约 10 亿美元收购了 Metronome，做高频用量计量和复杂定价。Lago 的注脚很有意思——Metronome 现在挂着 Stripe 的牌子卖，但它仍然有自己独立的应用和产品界面，和 Stripe Billing 并存。原话是：

> "Buying the missing layer was faster than rebuilding it. Making two products feel like one is slower."
> —— Lago wiki

（买下缺的那一层比重建它更快。让两个产品用起来像一个，则更慢。）

**My take：** 别把这三笔单独看。计量（Metronome）+ 路由（OpenRouter）+ 消费者与商家网络（PayPal 报价 530 亿）拼起来，是一句话：**AI 时代的每一笔钱，从产生成本到收到收入，都从 Stripe 过一遍。** 至于整合痛苦——Lago 已经提醒了，收购快，融合慢。

---

## 8. 5.5% 的充值手续费，实质是一道 token 税

OpenRouter 向用户收 5.5% 的充值手续费，不对 token 价格加价。HN 上有人算得很直白：

> "What's the angle for stripe , electrify over tokens exchange is the new money flow , and stripe wants to monetize it. 5% tax on any llm token is an amazing deal"
> —— HN 用户 maxdo

（Stripe 的角度是什么？token 交换成了新的资金流，Stripe 想把它变现。对任何 LLM token 抽 5% 的税，这买卖太划算了。）

但 Lago 提醒，明面上的费率反而是最不重要的部分：

> "The visible fee is simple. The more interesting asset is the demand sitting behind it."
> —— Lago wiki

（看得见的费率很简单。更有意思的资产是它背后那些需求。）

顺带一提，HN 上还有人引了 Stripe 自己的宣传语并评论：

> "> Every company in the Forbes AI 50 that monetizes does so on Stripe. That's chilling."
> —— HN 用户 bix6

（"福布斯 AI 50 里所有做变现的公司，都在 Stripe 上做。" 这话读着发凉。）

**My take：** 5.5% 是收银台费率，谁都能算。真正的资产是那个可以随时改写的加权函数。费率写在定价页上，路由权不写在任何地方。

---

## 9. 但这门生意的门没锁

Lago 自己把反方论点摆得最完整，这是这篇分析比一般吹票文可信的地方：

> "Developers can pin a provider, set a maximum price, sort for latency or throughput, bring their own provider keys, move to another gateway or integrate directly. Large teams can run open-source routing software themselves."
> —— Lago wiki

（开发者可以锁定某个供应商、设定最高价、按延迟或吞吐排序、自带供应商密钥、换一个网关，或者直接对接。大团队可以自己跑开源路由软件。）

> "As long as switching remains cheap, Stripe cannot squeeze either side very hard."
> —— Lago wiki

（只要切换成本保持低廉，Stripe 就没法对任何一边榨得太狠。）

这是和亚马逊类比最大的裂缝。亚马逊的商家离不开，是因为买家只在亚马逊。OpenRouter 的用户离得开，因为需求在他们自己手里——他们本来就是开发者，改一个 base_url 的成本约等于零。

**My take：** 这条决定了 70 亿是贵还是便宜。市场的价值 = 流量规模 × 抽成能力。流量 OpenRouter 有了，抽成能力目前基本没有。Stripe 赌的是前者足够大、且后者会随时间长出来。

---

## 10. 真正的风险资产不是费率，是中立性

> "The risk is not that OpenRouter becomes commercially motivated. It already is. The risk is that developers begin to suspect routing, pricing or product decisions are being made for Stripe before they are being made for them."
> —— Lago wiki

（风险不在于 OpenRouter 变得有商业动机，它本来就有。风险在于开发者开始怀疑，路由、定价和产品决策是先为 Stripe 服务，再为他们服务的。）

Lago 承认 OpenRouter 从来就不是完全中立的——"Its defaults already encode a view of what a good route is. That is the product."（它的默认值本身就编码了一套关于"什么是好路由"的观点。这就是产品。）开发者接受它，是因为规则写在文档里，而且通常符合他们的利益。

这两个条件，一个是透明，一个是利益对齐。Stripe 接手之后，第一个条件容易维持，第二个条件从此永远存疑。

**My take：** 这类中间层的资产负债表上，最大的一项写着"信任"，且从不出现在财报里。它的特点是：损耗时无声，归零时一次性。而 Stripe 买它的理由（把自家产品塞进路由链路）和维持它的条件（看起来不偏心），天然对着干。

---

## 结论

Lago 全文最后一句就是最好的结论，不用改写：

> "Stripe may be buying its way into the Amazon position. It has not bought Amazon's lock-in."
> —— Lago wiki

（Stripe 也许正在花钱买到亚马逊的位置。但它没有买到亚马逊的锁定。）

我把这句话再往前推一步：**它买到的是一个默认值，而不是一个必需品。**

默认值是有价值的，非常有价值——互联网的历史基本上就是一部默认值变现史。但默认值的价值有个上限，上限由"改掉它有多难"决定。对普通消费者，改默认值难于登天；对开发者，改默认值是一行配置。

Stripe 花 70 亿买下了一群全世界最擅长绕过默认值的人。这笔账要成立，它得让这些人在被明显偏心之前，先离不开。

Bloomberg 说交易已经敲定，Stripe 说不评论传闻。剩下的答案在未来两年的路由日志里。

---

*信源：Bloomberg（2026-08-16 独家）、TechCrunch（Anthony Ha，2026-08-16）、Lago wiki《With OpenRouter, is Stripe becoming the Amazon of AI?》、Hacker News 讨论串 #49323381、OpenRouter 官方 Provider Routing 文档。*
