---
title: "Debian 正在投票决定 AI 代码的命运：8 个选项讲清，附各大开源社区政策对照表"
slug: debian-llm-vote-2026-guide
summary: "2026 年 8 月 15 日至 28 日，Debian 开发者正在对「LLM 能不能用于 Debian 贡献」进行正式投票（GR 2026-002）。选票上有 8 个提案，从写进社会契约的全面禁止到不禁不倡，光谱完整；最严的选项需要 3:1 绝对多数。本文基于官方投票页、邮件列表原文和 LWN 报道，讲清 8 个选项的差别、两派核心论点、这场投票怎么走到今天，并附 Gentoo、Fedora、QEMU、curl、Linux 内核等十余个社区的 AI 贡献政策对照表。"
category: developer
tags: [Debian, 开源治理, AI 生成代码, LLM, 开源社区, DFSG, 投票, AI slop]
coverImage: ""
status: published
locale: zh
source: authored
---

Debian 正在进行的这场投票，可能是开源世界迄今最正式的一次「AI 代码表决」：General Resolution 2026-002《LLM usage in Debian》，投票期 **2026 年 8 月 15 日 00:00 UTC 至 8 月 28 日 23:59:59 UTC**（本文发布时投票刚开始，尚无结果）。只有 Debian Developer 有投票权，GPG 签名邮件投票，Condorcet 偏好排序，秘密计票。

选票上一共 9 项：8 个提案加"以上都不选"。这不是一次"禁或不禁"的二选一，而是把过去两年多社区里所有立场摊开来排序。下面按从严到松讲清每个选项，以及一个决定性的程序细节。

## 8 个选项：从写进社会契约的禁令到不禁不倡

**Choice 1：通过社会契约全面禁止**（提案人 Matthias Geiger）。往 Debian 社会契约里加一条："We will not allow direct contributions to Debian written with the use or assistance of large language models (LLMs) or other generative AI tools." 覆盖源码包、官方工具、网页、文档翻译、官方通信。执行方式是"善意遵守"——提案自己承认没有检测机制。提案文末还特意声明本文"无 LLM 协助、有机写成"。

**关键程序细节：这是唯一需要 3:1 绝对多数的选项**，因为它修改的是基础文件（社会契约），其余选项都只要简单多数。这个门槛极高，也是理解这场投票格局的钥匙。

**Choice 3：尽量拒绝，写入行为准则**（Ian Jackson）。承认全面禁止"currently impractical"，但划出一条日常红线：**人对人的消息——bug 报告、邮件列表、Salsa 讨论——必须由人来写**；任何 Debian 工作中用了 LLM 都必须声明；违规按行为准则处理。

**Choice 7：禁 AI 产出物，不禁 AI 工具**（Gard Spreemann，口号 "humans create Debian"）。最微妙的中间派：LLM 的输出不能直接成为贡献，但**不限制**你用 LLM 探索、分析、评审代码。这个"输出 vs 分析"的区分来自讨论期的跨阵营共识，GCC 新政策也是这个思路。

**Choice 2：允许，但附六个条件**（前 DPL Lucas Nussbaum）。工具条款兼容、许可证核验、贡献者全责、**强制声明**（Git trailer 如 `Generated-By:` / `Assisted-By:`）、批量自动化变更须事先讨论、保密信息不得喂给不可信服务。

**Choice 4：接受 + 责任制**（Pierre-Elliott Bécue，附议人包括 Russ Allbery）。理由直白："Rather than banning their use, which seems counter-productive and unenforceable, the project chooses to place responsibility on contributors."——标注是"应当"而非"必须"，因为 tab 补全类工具可能无感使用。

**Choice 5：不禁不倡**（Marc Haber，11 人附议）。"Debian neither endorses nor prohibits"——声明仅鼓励、不强制，最接近维持现状。

**Choice 6：鼓励避免**（Tobias Frost）。柔性版劝退：鼓励在可行处避免使用，声明是"对同伴的礼貌"。

**Choice 8：气候理由劝避**（Holger Levsen，**17 人附议，全场最多**）。一份立场声明而非可执行政策："we condemn LLM (resource) usage but not LLM users."

## 两派吵什么：论点浓缩

**反方（支持限制）**的四条主线：

- **版权与 DFSG**：LLM 输出的法律地位不明，而"Debian Policy 和 DFSG 要求许可与版权绝对清晰"，凭什么给 LLM 输出开特例
- **打包质量**：提案 A 说得很具体——LLM 生成的打包文件会混杂各年代语法，"不能工作的 watch 文件、脱离上下文的 overrides、想象出来的 copyright"
- **新人管线**：最坏情况是"新贡献者只是在 AI 和维护者之间当传话筒"（Simon Richter 的 "onboarding problem"）
- **爬虫之怨**：提案 A 直接把 AI 爬虫称为对 Debian 基础设施"大规模且永续的拒绝服务攻击"。这条有现场证据——lists.debian.org 现在挂着工作量证明反爬验证，本文引用的邮件原文都得先算一段 SHA-256 才抓得到

**正方（反对禁令）**的回击同样锋利：

- **不可执行**：Russ Allbery 两年前评 Gentoo 禁令时就说过"它（如他们自己承认的）无法执行"，"我们从不对人们本地用什么工具立法"
- **质量论点站不住**：还是 Allbery，2026 年的名句——"Writing meaningless slop requires no creativity; writing really bad code requires human ingenuity."（写无意义的烂浆糊不需要创造力；写真正的烂代码才需要人类的聪明才智）
- **守门自伤**：Ted Ts'o："你现在是在说，我们应该把可能在用 AI 的贡献者拒之门外？我说这更加自毁。"
- **"修改的首选形式"难题**：Bdale Garbee 问，chat prompt 生成的代码，它的 preferred form of modification 是什么？Nussbaum 答"是工具的输入"，但 LWN 指出 LLM 输出不可复现、模型会退役，这个回答并不圆满——这可能是整场讨论里最深的法理坑

有个细节把两派的方法论对撞浓缩得很好：提案 A 声明自己"有机写成"，而 Nussbaum 给全体开发者做的选项对比表，公然标注"由 AI 协助生成"。

## 这场投票是怎么走到今天的

- **2024-05**：受 Gentoo 禁令启发，Debian 社区首次正式讨论 AI 贡献政策，无果而终
- **2025-04**：Mo Zhou 提出"无训练数据的 AI 模型不算 DFSG 合规"的 GR，讨论后撤回
- **2026-02**：Lucas Nussbaum 草拟"允许 + 条件"的 GR——他自述动机是回应"针对在 Debian 语境下使用 AI 的人的各种攻击"——最终没有正式提交，LWN 标题精准："Debian decides not to decide"
- **2026-06/07**：外部环境骤变——Godot 禁 vibe coding（6 月 30 日）、Codeberg 社区 358:144 投票禁止托管"主要由 AI 生成"的仓库（7 月 22 日截止）、GCC AI 政策落地（7 月 30 日通报）
- **2026-07-22**：Matthias Geiger 直接甩出禁令提案："It is time for Debian to make a statement." 两周内滚出 8 个选项，8 月 15 日开票

## 各社区政策对照表

Debian 不是在真空里投票。主流社区的现行政策：

| 社区 | 政策 | 生效时间 |
|------|------|---------|
| Gentoo | 全面禁止（版权/质量/伦理三条理由） | 2024-04 |
| NetBSD | LLM 代码默认视为 tainted，须 core 书面批准 | 2024 |
| Asahi Linux | 最激烈：统称 "Slop Generators"，使用即违规，一次警告后永久封禁 | 现行 |
| QEMU | 默认拒收 AI 生成贡献（法理：DCO 无法满足），可申请例外 | 现行 |
| GCC | 拒绝"法律上重要"的 LLM 贡献；测试用例例外；小贡献须标 Assisted-by | 2026-07 |
| Godot | 禁自主 agent / vibe coding，AI 仅限杂务且须声明 | 2026-06 |
| Codeberg | 社区投票 358:144 禁止托管"主要由 AI 生成"的仓库 | 2026-07 |
| Fedora | 允许 + 强制声明（Assisted-by 标签），贡献者全责；AI 可辅助评审但不得终审 | 2025-10 |
| Linux 内核 | 允许 + `Assisted-by:` 标签；只有人能签 Signed-off-by（DCO） | 2025-12 合入文档 |
| curl | 不禁但强制声明，slop 即封号；2025 年约 20% 提交为 slop，2026 年 1 月底关停漏洞赏金后问题缓解 | 2024 起 |
| FreeBSD | 无正式政策，core team 因许可顾虑不用 AI 生成代码，政策制定中 | — |

可以看到光谱两端都有人站，而**声明义务（disclosure）是目前最大的公约数**——Debian 选票上 8 个选项里有 5 个包含某种形式的声明要求。

## 对普通开发者的实际影响

- **上游不受管**。全部 8 个选项一致把 upstream 排除在外：你自己项目怎么用 AI，Debian 不管，Debian 也照样打包含 AI 代码的上游软件。连最严的 Choice 1 都写明"不包括上游项目用 LLM 开发"
- **如果"允许 + 声明"类选项胜出**（目前看程序门槛上最可能）：给 Debian 提补丁时需要加 `Generated-By:` / `Assisted-By:` 之类的 Git trailer；批量自动化贡献要先上邮件列表讨论
- **如果 Choice 3 胜出**：影响最日常的一条——给 Debian 报 bug、发邮件不能让 AI 代写
- **通用红线（几乎所有选项共有）**：处于 embargo 的安全信息、debian-private 的内容，不得喂给第三方 AI 服务；贡献者对提交内容负全责，且要能自己解释它

结果 8 月 28 日之后见分晓。无论哪个选项胜出，这场投票的 8 份提案文本本身，已经是各开源社区制定自己 AI 政策时最完整的一份参考素材。

## 常见问题 FAQ

### 投票什么时候出结果？

投票期是 2026 年 8 月 15 日至 28 日（UTC），结果将在结束后由 Debian 项目秘书公布在官方投票页（vote_002）。采用 Condorcet 偏好排序，开发者对 9 个选项排偏好序。

### 最严的禁令选项通过的可能性大吗？

程序上门槛极高：Choice 1 要修改社会契约，需要 3:1 绝对多数，其余选项只需简单多数。历史上修改基础文件的 GR 通过者寥寥；且讨论期中"允许 + 声明"派聚集了多位前 DPL 背书。但 Condorcet 投票的排序效应难以预测，一切以 8 月 28 日后的官方结果为准。

### 这会影响我用 AI 写自己的开源项目吗？

不会。所有选项都只管"直接给 Debian 的贡献"（打包、官方工具、文档翻译、官方通信），明确排除上游项目。但如果你的项目会被 Debian 打包、或者你要给 Debian 提补丁，声明义务可能落到你头上。

### 为什么这么多社区都在 2024–2026 年集中立规？

两个直接压力源：一是版权/DFSG 层面 LLM 输出法律地位不明，DCO 签字制度无法覆盖（QEMU 政策的核心法理）；二是 AI slop 的运营成本——curl 的数据是 2025 年约 20% 的提交为 AI 垃圾报告，7 人安全团队每份要耗 3-4 人各 0.5-3 小时处理，最终在 2026 年 1 月关停了漏洞赏金计划。

## 参考链接

- [GR 2026-002 官方投票页（8 提案全文 + 时间线）— debian.org](https://www.debian.org/vote/2026/vote_002)
- [第一次投票召集（修正选票）— debian-devel-announce](https://lists.debian.org/debian-devel-announce/2026/08/msg00002.html)
- [Geiger 的原始提案 — debian-vote](https://lists.debian.org/debian-vote/2026/07/msg00000.html)
- [Debian decides not to decide on AI models — LWN（2026-03）](https://lwn.net/Articles/1061544/)
- [Debian dismisses AI-contributions policy — LWN（2024-05）](https://lwn.net/Articles/972331/)
- [Gentoo AI 政策原文](https://wiki.gentoo.org/wiki/Project:Council/AI_policy)
- [QEMU 代码来源政策（AI 贡献部分）](https://www.qemu.org/docs/master/devel/code-provenance.html)
- [GCC AI 政策](https://gcc.gnu.org/ai-policy.html)
- [Asahi Linux 关于 "Slop Generators" 的政策](https://asahilinux.org/docs/project/policies/slop/)
- [Death by a thousand slops — Daniel Stenberg（curl）](https://daniel.haxx.se/blog/2025/07/14/death-by-a-thousand-slops/)
