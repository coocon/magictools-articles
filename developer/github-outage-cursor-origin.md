# GitHub 宕机 7 个半小时那天，Cursor 发布了自己的代码托管平台

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/github-outage-cursor-origin?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/github-outage-cursor-origin?utm_source=github&utm_medium=referral)**

2026 年 8 月 17 日发生了两件事，单看哪件都算不上惊天动地，放在同一天就有了叙事的味道。

UTC 时间 13:40，GitHub 状态页开出编号 zkxwbgr0cnmx 的 incident，等级 critical。同一天下午，Cursor 发布了自家的代码托管平台 **Origin**，口号是 "designed for agent scale"。

一个平台的可靠性危机，和一个带着资本与 AI 叙事的挑战者，在同一个新闻周期里出现。这篇把两边的事实都摊开——包括几个已经在中文圈传歪了的细节。

---

## 先看宕机：7 小时 35 分，两个月内第 9 次 critical

根据 githubstatus 官方 API 的事故记录，8 月 17 日的时间线是这样的（全部 UTC）：

- **13:40** 开 incident；13:45 官方承认约 20% 错误率，波及 PR、Issues 等核心功能
- **14:04–16:16** 高峰期：Web/API 约 20% 错误率，归档下载和 raw 内容错误率约 50%，SAML/OIDC/SCIM 受影响；Actions、Webhooks、Pages、Git 操作、Copilot 陆续标记 degraded
- **16:59** 首轮缓解，之后余波反复——Git 操作、Issues、API 再次劣化，Copilot 认证间歇性失败
- **21:15** 解决。全程 **7 小时 35 分钟**，根因至今未公布，官方只承诺「会尽快分享详细的根因分析」

有个细节在 Hacker News 上被群嘲：有用户实测宕机从 13:35 左右就开始了，比官方开 incident 早 5 分钟——而那时状态页还是全绿的。更结构性的批评是：全站级故障被标成 "degraded performance"，而 degraded 不计入 uptime 统计，所以 Issues 的历史可用性显示 100%。

单次事故不算新闻，频率才是。我们拉了状态页 API 最近 50 条 incident 记录，**只覆盖了 6 月 16 日到 8 月 17 日约两个月**：6 月 9 起、7 月 26 起、8 月前 17 天 15 起。按等级分：critical 9 起、major 11 起、minor 28 起——其中 7 月一个月就有 6 起 critical。

这就是为什么这次 HN 的情绪不一样。主帖 529 分 888 条评论，有人写下「今天是 tipping point……希望已经死了」；单独的 Ask HN「GitHub 的替代品有哪些」拿了 491 分——抱怨帖变成了认真的选型帖。

## 再看 Origin：是什么，不是什么

Cursor 的 Origin 是真的，但网上流传的版本错得不少。以官方 changelog 和文档为准：

**它是什么**：Cursor 自家的 git forge，early beta，8 月 17 日起向所有**付费**计划分阶段开放。现有功能：仓库托管（标准 git clone/push/pull）、PR 的打开/审查/合并、网页端代码浏览与搜索、GitHub 双向镜像同步（GitHub 仍是 source of truth，PR 评论双向实时同步）、Origin CLI、第三方应用（Vercel 预览部署、Depot/Buildkite CI——后者可以直接跑你现有的 GitHub Actions workflow）。AI 原生的部分：每个仓库内置 agent，可以对着代码提问、改代码、更新 PR、推分支。

...

---

**[👉 继续阅读全文：GitHub 宕机 7 个半小时那天，Cursor 发布了自己的代码托管平台](https://tools.cooconsbit.com/zh/articles/github-outage-cursor-origin?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
