---
title: "Tailscale 挖出 SQLite 潜伏 16 年的 bug"
slug: sqlite-wal-reset-bug-tailscale
summary: "半年内 19 次数据库损坏，根因是 SQLite 里藏了约 16 年的 WAL-Reset 数据竞争。本文提炼 Tailscale 这场破案过程中的 10 个关键判断。"
category: ai-tutorials
tags: [SQLite, Tailscale, 数据库, WAL, 故障排查, 数据竞争, 工程复盘]
coverImage: ""
status: published
locale: zh
source: authored
translationSlug: sqlite-wal-reset-bug-tailscale-en
---

# Tailscale 挖出 SQLite 潜伏 16 年的 bug

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

事故间隔时而几小时，时而几周。中间那六周的空窗期，如果当时刚好上线过什么改动，几乎必然会被记成「修好了」。

这条伏笔在第 10 点会被引爆——正因为吃过一次「假平静」的亏，他们后来才拒绝用「没再出事」来结案。

**我的理解：** 间歇性故障最擅长的不是搞坏系统，是伪造证据。一段安静期加上一次巧合的发版，就能凭空生成一个错误的因果结论，然后团队带着这个错误结论继续往前跑几个月。**没有复现，不等于修好了**，只等于这次没抓到。

---

## 5. 破案转折点：一条 commit 了却「看不见」的写入

> "In two incidents, our transaction logs failed to replay cleanly. Upon closer inspection, we discovered that data written and committed by one transaction was inexplicably invisible to later transactions. A write had vanished into thin air without raising an error. That should be impossible!"
>
> —— 中文大意：有两次事故里，我们的事务日志无法干净地重放。细看之下我们发现，某个事务写入并提交的数据，莫名其妙地对后续事务不可见。一次写入凭空消失了，还没有报任何错。这本该是不可能的！

一边继续扛线上，一边他们搭了一条事务日志管线：**把每一条修改数据库的 SQL 语句都流式写进独立日志文件**。因为是单写者 + 可串行化事务，这份日志天然可以线性重放——重放不干净，就说明现实和日志对不上。

「commit 成功、无报错、但对后续事务不可见」，这个组合直接排除了一大片可能性：不是应用层逻辑错，不是并发写冲突，是**存储层把已经承诺过的数据弄丢了**。

**我的理解：** 这是整个调查里最漂亮的一步。它把问题从「数据库怎么坏的」精确成了「一次已提交的写去哪了」——一个范围小得多、也具体得多的问题。好的排查不是收集更多信息，是每一步都在削减搜索空间。

---

## 6. 从 10 页的 WAL 里拷出 20 页

> "If there are 10 pages in the WAL file and 20 pages get copied to the database, something is clearly wrong."
>
> —— 中文大意：如果 WAL 文件里有 10 页，却有 20 页被拷进数据库，那显然出问题了。

补一句 WAL 的原理：SQLite 数据库由一系列「页」组成，更新数据就要替换某些页。开启预写日志（Write-Ahead Logging）后，新页不直接写进主库文件，而是先写进 WAL 文件；WAL 不能无限增长，到某个时刻必须把页拷回主库——这个过程叫 **checkpoint（检查点）**。

他们的指标显示：损坏发生时，SQLite 报告的「从 WAL 拷回的页数」超过了 WAL 里实际存在的页数。数字对不上，checkpoint 有嫌疑。

于是 SQLite 官方开发者做了一个包在虚拟文件系统外面的追踪层——**tmstmpvfs shim**，专门吐额外的追踪信息和日志。Tailscale 把它部署进生产环境，然后等下一次损坏。

**我的理解：** 这一步的关键不是工具本身，是「先有假设，再造工具」的顺序。他们已经把嫌疑锁在 checkpoint 上，才去定制一个只观测 checkpoint 的探针。反过来做——先上一堆通用可观测性、指望从海量指标里浮出真相——通常只会得到一堆更贵的噪声。

---

## 7. 16 年没人踩中，罕见到要写代码故意触发

> "It could exist that long because it was rare — so rare, the SQLite developers had to add code to deliberately trigger it in their testing environments."
>
> —— 中文大意：它能存活这么久是因为太罕见了——罕见到 SQLite 开发者不得不在测试环境里加代码来故意触发它。

SQLite 官方把它命名为 **WAL-Reset bug**，并估计它在 SQLite 里至少存在了 16 年。bug 本体是 **checkpoint 与写事务之间的一个罕见数据竞争**：

> "If a write occurs at a specific time during a checkpoint, the checkpointing process gets confused — it thinks some of the pages have been copied from the WAL into the main database file, but they haven't. Those pages never get written to the database file, and that data is permanently lost."
>
> —— 中文大意：如果一次写入恰好发生在 checkpoint 期间的某个特定时刻，checkpoint 过程就会犯迷糊——它以为某些页已经从 WAL 拷进主库文件，但其实没有。这些页永远不会被写进数据库文件，那部分数据就永久丢失了。

库为什么会「损坏」而不只是「丢数据」？因为**其他引用这些页的页（比如索引）被正常写进去了**——指针还在，被指向的内容没了，结构就断了。官方的修复是在 checkpoint 函数里加一道检查，识别 WAL 是否已被另一个线程重置。

**我的理解：** 「这段代码跑了 16 年没出过事」是工程里最有欺骗性的一句安全感。它证明的只是这 16 年里没人凑齐触发条件，不是条件不存在。SQLite 是全世界测试最狠的开源项目之一，它都能藏住这个 bug 16 年——你那份「一直没出过问题」的祖传代码，凭什么更干净？

---

## 8. 为什么偏偏是 Tailscale 中招

> "They also explained why we were more likely to hit the bug than other SQLite users: we take manual control of the checkpointing process, and we checkpoint very aggressively."
>
> —— 中文大意：他们还解释了为什么我们比其他 SQLite 用户更容易撞上这个 bug：我们手动接管了 checkpoint 过程，并且 checkpoint 得非常激进。

他们为什么要手动接管？为了**跑快速且一致的备份**——每隔几分钟对数据库做一次完整快照，把整个 SQLite 文件传到 S3。要保证快照一致，就得自己控制 checkpoint 的时机。

这是一个非常典型的工程权衡：为了备份的确定性，放弃了默认路径的确定性。文章里那句自评很克制——"This non-standard approach seemed suspicious."（这种非标准做法看起来很可疑。）

**我的理解：** 数据竞争是概率问题，而调用频率就是采样次数。别人一天 checkpoint 几次，你一分钟几次，那么在同一个 bug 面前，你的曝光量可能是别人的几千倍。**把某个操作的频率拉高一两个数量级，本质上等于替全世界跑压力测试**——好处是你会先发现问题，坏处是用你的生产环境发现。

---

## 9. 修好一个 bug，炸出另一个：3.52.0 被撤回

> "Because this change caused false corruption warnings, the SQLite developers withdrew the 3.52.0 release and instead published 3.51.3, which only contained a fix for the WAL-Reset bug."
>
> —— 中文大意：由于这个改动导致了虚假的损坏告警，SQLite 开发者撤回了 3.52.0 版本，改为发布只包含 WAL-Reset bug 修复的 3.51.3。

修复过程本身也值得记一笔。他们先灰度到几个 canary shard，再推全量——然后备份监控立刻变红，报告 **13 个数据库损坏**。

虚惊一场：这些库并没有真的损坏，而是撞上了另一个跟 stale expression index（陈旧表达式索引）有关的问题。他们把高精度时间戳以文本存储，再用 VIRTUAL 生成列转成浮点数；而 3.52.0 在修数据竞争的同时做了一个优化，微妙地改变了「文本转浮点」的舍入行为。索引里存的是旧舍入结果，重新计算得到新结果，校验就判定为损坏。

所以官方撤回 3.52.0，只发 WAL-Reset 修复的 3.51.3。Tailscale 自己这边把时间戳精度降到整数秒；SQLite 那边在 3.53.0 里做了自动自愈索引的能力。

**我的理解：** 两个信号。第一，**灰度不是形式主义**——如果他们一把梭全量，13 个库同时报损坏的场面会让人第一时间怀疑是修复本身把数据搞坏了，误判成本极高。第二，版本号不代表安全，3.52.0 比 3.51.3 新，但正确的选择是后者。升级前看 release note 和撤回公告，比看数字大小有用得多。

---

## 10. 不出事不算修好，要的是「正证明」

> "An absence of corruption incidents doesn't mean things are fixed — we'd already had one six-week period of deceptive calm. We wanted positive proof that this data race was actively occurring."
>
> —— 中文大意：没有损坏事故并不代表问题修好了——我们已经经历过一次长达六周的欺骗性平静。我们想要正面证据，证明这个数据竞争确实在真实发生。

于是他们做了一件反直觉的事：**给已经打过补丁的 SQLite driver 再加一条日志**，专门在「checkpoint 与写事务重叠」这个条件命中时打 warning。部署，然后等。

两个月后，告警终于响了："SQLite attempted corruption... in party mode, but the system prevented it."（SQLite 试图触发损坏……在 party 模式下，但系统阻止了它。）

这条告警的意义是：**触发条件确实在他们的生产环境中发生过**，补丁不是安慰剂，是真的挡住了。从那次告警之后，又跑了四个月，零数据库事故。

**我的理解：** 「没有证据表明还在出问题」和「有证据表明问题已被拦住」，是两个完全不同的结论。前者只能安慰团队，后者才能结案。绝大多数团队修完 bug 就把监控撤了——正好把最有价值的那个证据丢掉。**修复上线之后，留一条专门证明修复生效的日志，成本极低，回报极高。**

---

## 结语：boring technology 的边界在哪

整篇文章最重的一句在结尾：

> "This investigation is a useful reminder: running boring technology in a non-standard way is a risk. The common paths and standard configurations are incredibly well-tested and reliable. Everything we were doing was a public, documented, supported configuration — but by taking manual control of the checkpointing process and running at our own aggressive pace, we stepped off the well-trodden operational path."
>
> —— 中文大意：这次调查是一个有用的提醒：用非标准的方式运行「无聊的技术」本身就是风险。常见路径和标准配置经过了极其充分的测试，非常可靠。我们所做的一切都是公开的、有文档的、受支持的配置——但通过手动接管 checkpoint 过程、按我们自己激进的节奏运行，我们踏出了那条被踩实的运维路径。

注意这里的分寸：他们**没有**说「不要用 SQLite」，也没说「别选无聊的技术」。他们说的是——**你选了一项久经考验的技术，不代表你选了一条久经考验的路径**。技术的可靠性来自被无数人重复走过的那条主路；一旦你为了某个正当理由（这里是一致性备份）拐进支路，可靠性的存量就不再自动继承给你了。

「公开、有文档、受支持」这三个词特别值得琢磨。这是很多技术选型评审能给出的最高分，但它依然不等于「被大量真实生产环境验证过」。文档写了这个配置合法，和这个配置每天被十万个用户跑，是两回事。

这次事件里还有一条容易被忽略的价值：Tailscale 找 SQLite 开发者签了**专业支持合同**，并**出资资助了那个用来隔离竞态的开源 VFS shim**。结果是——bug 修了，全世界的 SQLite 用户都受益，包括那些一分钱没出的。这是开源协作里少见的、账算得清清楚楚的正向循环。

**给普通开发者的三条实操建议：**

1. **查一下自己的 SQLite 版本。** 如果你用 WAL 模式而且手动控制 checkpoint，升级到 3.51.3 或更高（注意不是 3.52.0，它已被撤回）。绝大多数默认配置的用户风险很低，但确认一下不费事。
2. **备份要演练，不是要存在。** Tailscale 在这半年里把恢复流程实战演练了十几次，把停机从一小时以上压下来。没被真正跑过的恢复流程，等同于没有。
3. **偏离默认路径时，把这件事写下来。** 每一处「我们和标准用法不一样」的地方，都应该在文档里留一行，注明为什么这么做、放弃了什么。等到出事那天，这份清单就是排查的第一份地图。

最后照抄他们的结论：这半年很难熬，但结束时他们比开始时更强了——bug 修了，顺手修掉了几十个排查过程中发现的其他问题，备份恢复流程被真刀真枪验证了十几次。

**排查一个够狠的 bug，附赠品往往比修复本身更值钱。**

---

*原文：Tailscale 官方博客《How we tracked down a 16-year-old SQLite bug》*
*原文链接：https://tailscale.com/blog/sqlite-wal-reset-bug*
