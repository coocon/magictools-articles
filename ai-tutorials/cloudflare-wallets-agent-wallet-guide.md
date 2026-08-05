---
title: "Cloudflare Wallets 上线：给 AI Agent 发身份证和钱包，cloudflare.pay 账户名开抢"
slug: cloudflare-wallets-agent-wallet-guide
summary: "2026 年 8 月 4 日，Cloudflare 在 Agents Week 上发布 Cloudflare Wallets 和 cloudflare.pay：给 AI Agent 一个人类可读的稳定身份，加一个由主人设定额度的稳定币钱包。本文讲清它解决什么问题、Account Wallet 与 Virtual Wallet 怎么分工、x402 微支付如何凑齐 Agent 经济的双边市场，以及现在唯一能做的事——抢注你的 Wallet 账户名。"
category: ai-tutorials
tags: [Cloudflare, AI Agent, 稳定币, x402, Agentic Commerce, 智能体支付]
status: published
locale: zh
source: authored
translationSlug: cloudflare-wallets-agent-wallet-guide-en
---

# Cloudflare Wallets 上线：给 AI Agent 发身份证和钱包，cloudflare.pay 账户名开抢

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

技术上，这层身份建立在 Cloudflare 已有的 Bot Management、Turnstile 和 Web Bot Auth（密钥对注册身份）之上——handle 的作用是把密钥对变成**人类可读**的名字。

### 2.2 Account Wallet 与 Virtual Wallet：人管钱，Agent 花钱

钱包分两级，权责分明：

| | Account Wallet（主钱包） | Virtual Wallet（虚拟子钱包） |
|---|---|---|
| 给谁用 | 人类（Cloudflare 账户所有者） | AI Agent |
| 怎么操作 | Dashboard 管理 | API key |
| 能干什么 | 充值、提现、持有稳定币、给子钱包分配额度 | 在授权范围内消费 |
| 护栏 | 制定所有策略 | 消费上限、商家白名单、单笔交易限额 |

入金方式：银行转账后自动转换为美元稳定币；符合条件的用户也可以直接用稳定币自充。

官方博客里有一个反直觉但很精彩的论点：**给 Agent 更少的钱，反而给了它更多自由**。如果一个 Agent 只掌管 10 美元，你可以放心让它自主探索几十上百个 API（每个只花几美分）；如果它掌管 1000 美元，你就得盯着它。低额度 + 白名单 + 单笔上限，让"放手让 Agent 自己跑"从冒险变成可控实验。

官方给的落地场景也很实际：想给每个员工每周 100 美元的 AI 推理预算？开一个 Account Wallet，给每人发一个带此规则的 Virtual Wallet。超了额度，员工找有权限的管理员人工审批放行——异常快速消费会触发人工复核。

### 2.3 x402 + Monetization Gateway：双边市场闭环

支付协议走 **x402**——一个把付款直接附在 HTTP 请求上的微支付协议。这让"按请求付费"成为可能：AI 推理、数据、内容，都可以细到单次请求收费。

时间线值得注意：一个月前（2026 年 7 月 1 日），Cloudflare 刚发布了 **Monetization Gateway**，让网站可以按请求向 Agent 收稳定币——这是**卖方**基建。这次的 Wallets 补上了**买方**基建。买卖双方凑齐，Agent 经济的双边市场才算闭环。

## 三、现在能做什么：只有抢名字

务必分清当前状态：

- ✅ **现在可用**：去 [cloudflare.pay](https://cloudflare.pay) 认领你的 Wallet handle（免费，先到先得，需要 Cloudflare 账户）。Cloudflare 保留以任何理由拒绝预订的权利，所以别指望抢注商标名转卖。
- ⏳ **未来几个月**：完整钱包功能——充值/提现、发行 Virtual Wallet、实际支付。

抢注步骤很简单：打开 cloudflare.pay → 登录 Cloudflare 账户 → 输入想要的名字 → 确认预订。功能开放时会收到通知。

**为什么值得现在就抢**：域名时代抢的是网站门牌，Agent 时代抢的是支付身份。这个 handle 未来既是你的收款地址，也是你旗下所有 AI Agent 对外的"姓氏"。好名字的稀缺性逻辑和域名、社交媒体用户名一模一样——发布当天 X 上已经有不少人在抱怨心仪的名字被抢了。

## 四、三个值得保持清醒的点

1. **功能还没上线，别当成能用的钱包**。现在的 cloudflare.pay 只是一个名字预订页。任何声称"现在就能往 Cloudflare Wallet 充值"的链接都是骗局。
2. **警惕仿冒钓鱼站**。这种抢注热潮历来是钓鱼站的温床。认准官方域名 `cloudflare.pay`，从 Cloudflare 官方博客或官方 X 账号的链接进入，注意域名拼写。
3. **稳定币合规是变量**。入金/出金"在受支持的地区"分批开放——这暗示了不同司法辖区的合规进度会不同，国内用户能否完整使用存疑，抢个名字倒是无妨。

## 五、为什么这件事重要

Cloudflare 引用了自家数据：**网络流量已经过半来自 bot**。互联网正在从"人类浏览"切换到"Agent 交易"，但信任和支付的基础设施还停留在人类时代。

Cloudflare 的独特位置在于它坐在全球约 20% 网站的流量路径上——它做 Agent 身份层，商家侧几乎零改造成本。这和 Visa、Mastercard、OpenAI 各自推进的 Agent 支付方案形成了有趣的竞争格局：卡组织从支付网络往上做，Cloudflare 从流量网络往下做。

值不值 all in 还早，但花两分钟抢个名字的性价比，怎么算都是划算的。

## 参考资料

- [Cloudflare 官方新闻稿：Cloudflare Gives AI Agents an Identity and a Wallet](https://www.cloudflare.com/press/press-releases/2026/cloudflare-gives-ai-agents-an-identity-and-a-wallet/)
- [Cloudflare 官方博客：Announcing Cloudflare Wallets: the programmable wallet for the agentic Internet](https://blog.cloudflare.com/wallets/)
- [cloudflare.pay 官方预订页](https://cloudflare.pay)
- [Fortune：Cloudflare just launched a permanent ID tool and wallet for AI shopping](https://fortune.com/2026/08/04/cloudflare-ai-agents-wallets-id/)
