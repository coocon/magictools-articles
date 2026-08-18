---
title: "GitHub 宕机 7 个半小时那天，Cursor 发布了自己的代码托管平台"
slug: github-outage-cursor-origin
summary: "8 月 17 日 GitHub 经历了 7 小时 35 分的 critical 级事故——这是它两个月内的第 9 次 critical。同一天，Cursor 发布代码托管平台 Origin。一边是耐心耗尽的 Hacker News 认真讨论搬家，一边是带着 SpaceX 600 亿美元资本入场的挑战者。护城河开始松动了吗？我们把两件事的数据都核了一遍。"
category: developer
tags: [GitHub, Cursor, 代码托管, Git, 宕机, 开发者工具]
coverImage: ""
status: published
locale: zh
source: authored
translationSlug: github-outage-cursor-origin-en
---

# GitHub 宕机 7 个半小时那天，Cursor 发布了自己的代码托管平台

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

**它不是什么**：没有 Issue 跟踪，没有 Wiki，没有独立定价，也没有免费层——官方文档明确写着 "It is not available on free plans"。流传的「每月 20 美元起」是把 Cursor Pro 的订阅价安到了 Origin 头上（Origin 是付费计划附带功能，不单卖）；「免费版 3 个私有仓库」则纯属编造。

**背景两条**：Cursor 在 2025 年 12 月收购了代码审查公司 Graphite，其创始人 Tomas Reimers 就是 Origin 的开发者之一，本人在 HN 发布帖里现身认领。更大的背景是，就在 Origin 发布 3 天前（8 月 14 日），SpaceX 完成了对 Cursor 母公司 Anysphere 的 **600 亿美元**全股票收购（SEC 8-K 已确认）。

还有一个写剧本都不敢这么写的细节：**Origin 发布当天就被 GitHub 宕机波及了**——它的 GitHub 同步功能依赖 GitHub，Cursor 自己也挂出了 incident。挑战者在出道首日，被它想挑战的对象连带击穿。

## HN 在吵什么：归因、搬家、和搬不动的理由

**归因之争**。一派认为是 LLM 时代的流量把传统 forge 压垮了——「用量涨了几十倍，agent 在推垃圾代码」（这条评论下有 130 条回复）；另一派反驳：「LLM 代码涌入不是问题，微软管理不善才是」。还有人问了个很实际的问题：为什么不用价格和限流解决？

**搬家实录**。声量最大的目的地是 Forgejo/Codeberg：「我们弃了 GitHub 换 Forgejo，迁移只花几小时，当 hackathon 玩」「几个月前全迁 Codeberg 并设了年度捐赠，压垮我的是 GitHub 硬塞 Copilot」。有人给出决策框架：想要 GitHub 体验选 Forgejo/Gitea，只要省心托管选 GitLab/Codeberg，有自己基础设施就 gitolite+CGit。也有联邦化新势力 tangled.org 的创始人现场招揽。

**搬不动的理由**，可能是全场最清醒的部分。自托管 GitLab 六年的用户作证：「并不总是一帆风顺」，升级回滚、维护成本都是真的。更根本的反方论点：GitHub 的价值在于**全开源世界的统一性**——跨项目搜索、共享 Actions、单一贡献面板；forge 巴尔干化会摧毁「所有人都在同一处」的公共品价值。重度绑定 GitHub Actions 生态的组织，实际上没有选项——「Actions 的替代品才是真问题」。

## 判断：松动的开端，不是松动已成

把证据摆在一起看：

**支撑「护城河松动」的**：两个月 50 起 incident、9 起 critical 是官方 API 可查的硬数据，不是体感；状态页粉饰实锤了信任侵蚀；社区讨论第一次从「抱怨完继续用」变成有真实迁移案例的认真选型；而 Cursor 挑的时机极准——「agent-scale」的定位，打的正是「传统 forge 扛不住 agent 流量」这个 HN 吵了一整天的痛点，背后还有 SpaceX 的资源。

**反方证据同样硬**：Origin 现阶段是付费墙内的 early beta，没有 Issues、没有免费层，对 GitHub 的开源生态构不成任何替代——HN 上有人一句话点破：「这是付费计划的 beta，不是 GitHub alternative」。它今天还是 GitHub 的**寄生层**（靠 GitHub 同步活着），而不是替代品。且「编辑器 + 托管 + 模型」三链归于一家 Musk 系公司，已经有开发者以信任理由明确拒绝。

所以最准确的表述是：GitHub 的可靠性危机，第一次和一个有资本、有 AI 叙事的挑战者出现在了同一天。但挑战者离能承接迁移还很远，而 GitHub 最深的护城河从来不是可用性，是生态的网络效应。

给普通开发者的可操作建议只有一条，也是这次宕机里代价最低的保险：花十分钟给关键仓库加一个第二 remote（GitLab/Codeberg/自建都行），`git remote set-url --add --push` 一条命令的事。平台之争你可以旁观，push 不上去的下午你不想再经历第二次。

---

*资料来源：*
*GitHub 状态页事故记录：[Incident zkxwbgr0cnmx](https://www.githubstatus.com/incidents/zkxwbgr0cnmx)（频率统计来自 [githubstatus API](https://www.githubstatus.com/api/v2/incidents.json)）*
*Cursor 官方：[Origin changelog](https://cursor.com/changelog/origin-code-hosting) / [Origin 文档](https://cursor.com/docs/origin)*
*Hacker News：[宕机主帖（529 分）](https://news.ycombinator.com/item?id=49330597) / [Ask HN: GitHub 替代品（491 分）](https://news.ycombinator.com/item?id=49331033) / [Origin 发布帖](https://news.ycombinator.com/item?id=49334209)*
*SpaceX 收购 Cursor：[Seeking Alpha（SEC 8-K）](https://seekingalpha.com/news/4633335-spacex-completes-60b-acquisition-of-cursor-as-musk-led-firm-tries-to-gain-edge-in-ai-coding)*
*观点文：[GitHub has an availability problem](https://dhruv2038.bearblog.dev/github-has-an-availability-problem-is-it-time-to-look-elsewhere/)*
