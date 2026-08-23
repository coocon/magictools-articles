# Debian 正在投票决定 AI 代码的命运：8 个选项讲清，附各大开源社区政策对照表

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/debian-llm-vote-2026-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/debian-llm-vote-2026-guide?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：Debian 正在投票决定 AI 代码的命运：8 个选项讲清，附各大开源社区政策对照表](https://tools.cooconsbit.com/zh/articles/debian-llm-vote-2026-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
