# Cloudflare Wallets 上线：给 AI Agent 发身份证和钱包，cloudflare.pay 账户名开抢

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/cloudflare-wallets-agent-wallet-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/cloudflare-wallets-agent-wallet-guide?utm_source=github&utm_medium=referral)**

2026 年 8 月 4 日，Cloudflare（NYSE: NET）在自家 Agents Week 上扔出一个重磅发布：**Cloudflare Wallets** 和 **cloudflare.pay**。一句话概括——给部署在 Cloudflare 上的 AI Agent 一个稳定的、人类可读的身份，再配一个由人类主人设定消费额度的稳定币钱包。

CEO Matthew Prince 本人第一时间抢注了 `eastdakota.cloudflare.pay` 并发推带节奏，X 上随即刮起一阵"晒 handle"风潮。如果你刷到过 *"I just reserved my Cloudflare Wallet tag"* 这种模板推文，没错，那就是这次发布的官方病毒式传播设计。

这篇文章讲清楚三件事：它解决什么问题、机制怎么设计的、以及你现在能做什么（提示：目前唯一能做的就是抢名字）。

## 一、AI Agent 上网干活的两个死结

Cloudflare 在新闻稿里把问题说得很直白：**互联网是为人类设计的，不是为 Agent 设计的**。一个 AI Agent 想试用一个新 API，今天的真实流程是这样的：

1. 撞上一个为人类设计的注册页（可能还有验证码）；
2. 找它的人类主人要一张信用卡绑支付方式；
3. 生成 API key，再琢磨怎么调用。

结果就是 Agent 往往直接放弃，把注册、绑卡、生成 key 这些活全部踢回给人。这背后是两个结构性缺陷：

- **没有身份**：Agent 访问网站时，商家没有可靠手段区分它是"真实客户的助手"还是"薅羊毛的恶意脚本"。老一代反爬虫工具是为搜索引擎爬虫设计的，不适用于替真人交易的 Agent。商家只剩两个坏选项：要么全封，要么裸奔。
- **没法付钱**：Agent 没有原生的支付手段。想让它自主花几美分试用一个 API？现有支付体系里没有这条路。

Prince 的原话值得引用："当一个 Agent 出现在你门口时，你需要知道是谁派它来的。Cloudflare 可以给 Agent 一张脸——一条指向拥有它的人或组织的链接——这样信任、问责和真正的商业才能随之而来。"

## 二、机制拆解：Handle + 两级钱包 + x402

### 2.1 Wallet Handle：Agent 世界的"姓氏"

每个 Cloudflare 账户可以在 [cloudflare.pay](https://cloudflare.pay) 认领一个唯一的 handle，形如 `yourname.cloudflare.pay`。这个网址型 ID 就是你在 Agent 经济里的稳定身份。

更关键的是身份可以**向下委托**：你可以把身份延伸给具体的 Agent。比如一个研究型 Agent 可以住在 `research.yourname.cloudflare.pay`——商家一眼就能看出"这是哪个组织授权的哪个代理"。Agent 声明身份是完全可选的，但商家也有权决定是否优先与"有名有姓"的 Agent 做生意。Cloudflare 的类比是 VPN：匿名不等于不可信，但匿名者需要付出更多证明成本。

...

---

**[👉 继续阅读全文：Cloudflare Wallets 上线：给 AI Agent 发身份证和钱包，cloudflare.pay 账户名开抢](https://tools.cooconsbit.com/zh/articles/cloudflare-wallets-agent-wallet-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
