# Tailscale 挖出 SQLite 潜伏 16 年的 bug

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/sqlite-wal-reset-bug-tailscale?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/sqlite-wal-reset-bug-tailscale?utm_source=github&utm_medium=referral)**

> 本文基于 Tailscale 官方博客《How we tracked down a 16-year-old SQLite bug》（2026-08-12）整理，提炼 10 个核心观点并做解读。引文均为英文原文，附中文大意。原文链接见文末。

---

先交代背景，不然后面的线索看不懂。

Tailscale 的控制平面对外是一个域名（controlplane.tailscale.com），对内被切成一组协调服务器，官方叫 shard。每个 tailnet 同一时刻只落在一个 shard 上，可以无缝迁移。每个 shard 配一个 SQLite 数据库，由**单个 Go 进程独占访问**——这是 SQLite 设计上最推崇的用法。他们从 2022 年就这么跑，2023 年初起没出过事。

然后，去年 8 月，读 S3 备份的数据管线报了错。`PRAGMA integrity_check` 跑下来，库确实损坏了。修完、查完，没找到原因。

接下来半年，这件事重复了 19 次。

---

## 1. 规模会把「极罕见」变成「每月都有」

> "When operating at scale, even rare events can occur with some frequency. In total, we faced 19 separate instances of database corruption over six months before we finally resolved the underlying bug."
>
> —— 中文大意：在足够大的规模下，即使是罕见事件也会以一定频率发生。在最终定位根因之前，我们在六个月里遭遇了 19 次独立的数据库损坏。

SQLite 损坏是有可能的，但极不寻常。单看任何一次，都会被归进「宇宙射线」那一类无法解释的意外。19 次之后，它就不再是意外，而是一个必须被找出来的确定性缺陷。

值得注意的是他们对影响面的坦诚：这些库里存的是 tailnet 和设备的元数据，从来不含私钥和网络流量；早期几次事故，恢复的代价是「少数刚添加的设备或配置变更没有持久化」。而且**每次损坏都要停掉那个 shard 的控制平面进程**才能修复或恢复，最初一次停机超过一小时。

他们还提到一句常被工程团队忽略的话：即使只影响少数 tailnet，他们也会在状态页发全局事故公告——**重复的停机会侵蚀信任，无论你是不是当事人**。

**我的理解：** 「概率极低」不是一个可以拿来结案的答案，它只是一个还没被计算的分母。你的实例数、请求量、运行时长每涨一个数量级，罕见事件就往前挪一格。规模本身就是一台把长尾事件从「理论上存在」推成「本周又来了」的机器。

---

## 2. 教科书式的正确用法，照样会中招

> "A single Go process exclusively accesses that database, and serves the control plane for those tailnets. This single-writer design is exactly how SQLite is meant to be used."
>
> —— 中文大意：单个 Go 进程独占访问该数据库，并为这些 tailnet 提供控制平面服务。这种单写者设计正是 SQLite 被设计出来的用法。

这句话在故事开头是一句自证清白：我们没有多进程抢写，没有 NFS，没有把 SQLite 当分布式数据库用。恰恰因为架构是干净的，所有「你用错了」的解释才被提前排除掉。

但这句话真正的价值要到第 5 点才兑现——正因为是单写者、可串行化事务，**事务历史是完全线性、完全确定的**。这一点在 Postgres 或 MySQL 那种多写者数据库里根本不成立。

**我的理解：** 干净的架构不保证不出事，但它保证出事之后你查得动。可复现性、可重放性、可归因性，都是架构在事前就已经决定好的东西。等到出事再想补，来不及。

---

## 3. 所有「共同点」全部落空，就只能被动取证

> "It wasn't tied to a single shard, or customer, or tailnet feature, or time of day, or load level."
>
> —— 中文大意：它跟单个 shard、客户、tailnet 功能、时间段、负载水平，都对不上。

排查的常规套路是找相关性：哪台机器？哪个客户？哪个新功能？哪个时段？高负载吗？Tailscale 把这些维度挨个查了一遍，**全部落空**。他们也复查了最近的变更、拿放大镜过了一遍所有底层 SQLite 代码，同样一无所获。

没有可靠的触发条件，就意味着无法合成复现。于是策略被迫切换：

> "Instead, we had to rely on deploying passive, forensic telemetry in our live environment to catch the corruption red-handed."
>
> —— 中文大意：我们只能在生产环境部署被动的、取证式的遥测，等着把损坏当场抓获。

**我的理解：** 无法复现的 bug 有两种打法：拼命造复现环境，或者把生产环境改造成案发现场的证据链。前者在触发条件未知时是无底洞，后者才是正解。区别在于，第二种打法要求你敢在生产环境加诊断代码——这需要工程能力，更需要组织上的许可。

---

## 4. 六周的平静，是这个故事里最危险的部分

> "We had a six-week period between October and December when there were no corruption incidents, before they returned as an unwelcome Christmas present."
>
> —— 中文大意：10 月到 12 月之间有六周没有发生任何损坏事故，然后它们作为一份不受欢迎的圣诞礼物回来了。

...

---

**[👉 继续阅读全文：Tailscale 挖出 SQLite 潜伏 16 年的 bug](https://tools.cooconsbit.com/zh/articles/sqlite-wal-reset-bug-tailscale?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
