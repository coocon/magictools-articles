---
title: "Claude Code Skills 完全指南：用 SKILL.md 把重复工作流变成一条斜杠命令"
slug: claude-code-skills-guide
category: claude
locale: zh
translationSlug: claude-code-skills-guide-en
tags: [Claude Code, Skills, SKILL.md, 斜杠命令, 工作流自动化, 上下文管理]
summary: "Skills 是 Claude Code 里被低估最狠的功能：把一套操作步骤写进 SKILL.md，就能用 /命令 一键调用，还能让 Claude 在任务匹配时自动加载。本文讲清 Skills 的文件结构、frontmatter 写法、自动触发机制、与 CLAUDE.md/hooks/子代理的分工，以及三个可以直接抄的实战技能。"
status: published
---

如果你发现自己每周都在对 Claude Code 重复同一段长指令——"先拉数据、再按这个格式出报表、注意这三个坑"——那你需要的不是更好的提示词，而是一个 Skill。

Skill 是 Claude Code 的可复用指令包：把一套工作流写成 `SKILL.md` 文件放进约定目录，之后既可以用 `/技能名` 手动调用，也可以让 Claude 在识别到匹配任务时自动加载。它解决的是"专家经验没法沉淀"的问题：你调教好的工作流，从此跟着项目走，团队里每个人（以及每次新会话）都能直接用。

## Skill 的文件结构：一个目录、一个 SKILL.md

一个最小的 Skill 长这样：

```
.claude/skills/
└── deploy-check/
    └── SKILL.md
```

`SKILL.md` 由 frontmatter 和正文两部分组成：

```markdown
---
name: deploy-check
description: 部署前检查清单。当用户说"准备部署"、"发布前检查"或要求上线时使用。
---

按顺序执行以下检查，任何一步失败都停下来报告：

1. 运行 `npm run typecheck`，确认无类型错误
2. 运行 `npm run build`，确认构建通过
3. 检查 git status，列出未提交的文件
4. 对照 CHANGELOG 确认版本号已更新
```

两个存放位置，作用域不同：

- **项目级**：`.claude/skills/<名称>/SKILL.md`，跟着仓库走，团队共享
- **个人级**：`~/.claude/skills/<名称>/SKILL.md`，跨项目生效，只有你可见

## description 是灵魂：它决定 Skill 什么时候被自动加载

Skills 有两种触发方式。手动触发很直白：会话里输入 `/deploy-check` 即可。真正的巧思在自动触发——Claude 会在每轮对话里比对所有技能的 `description`，任务匹配就自动加载对应 SKILL.md。

所以 description 不要写成功能简介，要写成**触发条件清单**：

```yaml
# ❌ 弱 description：Claude 不知道什么时候该用
description: GA4 数据分析技能

# ✅ 强 description：把用户可能说的话写进去
description: 分析站点 GA4 流量数据时使用。触发词包括"看访问量"、
  "哪个页面最火"、"流量来源"、"改版效果"、pageviews、traffic。
```

这和给人写文档的思路相反：给人看的文档解释"是什么"，给 Claude 看的 description 要穷举"什么时候用"。

## 渐进式加载：Skills 是上下文管理利器

Skills 与 `CLAUDE.md` 的关键区别在加载时机。CLAUDE.md 每次会话全文进上下文，写太长会持续消耗 token；而 Skill 只有名称和 description 常驻，正文在被触发时才加载。

这带来一个实用的架构原则：

- **CLAUDE.md 放"永远适用"的规则**：代码风格、构建命令、红线约束
- **Skills 放"特定场景"的流程**：发布检查、数据分析、报表生成、某个子系统的操作手册

一个技能目录还可以带附属文件（参考文档、脚本），在 SKILL.md 里引用，Claude 需要时才去读——重文档的工作流也不会撑爆上下文。

## Skills、hooks、子代理：三兄弟怎么分工

三个机制经常被混淆，按"确定性"排一下就清楚了：

| 机制 | 本质 | 适用场景 |
|------|------|---------|
| Hooks | 确定性 shell 命令，在事件点必然执行 | 提交前 lint、写文件后自动格式化 |
| Skills | 按需加载的指令，Claude 照着做 | 多步骤流程、领域操作手册 |
| 子代理 | 独立上下文的分身，干完活汇报结果 | 大范围搜索、并行探索、隔离的重活 |

判断口诀：**必须每次都发生的用 hook，需要 Claude 理解和随机应变的用 skill，会污染主上下文的重活交给子代理**。三者可以组合——比如一个 skill 的正文里就可以写"把这一步派给子代理执行"。

## 三个可以直接抄的实战技能

**1. 发布检查（deploy-check）**：上文的例子。价值在于把"部署前该做什么"从某个人的脑子里搬进仓库。

**2. 数据报表（weekly-report）**：

```markdown
---
name: weekly-report
description: 生成周报时使用。触发词："周报"、"weekly report"、"这周数据"。
---

1. 运行 `npm run ga:report -- top-pages --days 7`
2. 与上周对比，标出涨跌超过 20% 的页面
3. 按"结论先行"的格式输出：先一句话总结，再放数据表
4. 数据延迟 24-48 小时，报表末尾必须注明统计截止日期
```

**3. 排障手册（debug-build）**：把项目特有的坑写进去——"构建失败先查 Node 版本是否 20+"、"Prisma 报错先跑 db:generate"。新人（和新会话）从此不用重新踩一遍坑。

## 上手建议

从你最近一周重复说过两次以上的指令开始：把它原样粘进一个 SKILL.md，写好触发条件，下次用 `/技能名` 调用。跑通一次后再回头打磨步骤描述。Skills 的回报是复利的——每沉淀一个，之后每次会话都在赚。

想系统了解 Claude Code 的其他进阶能力，可以接着读站内的《Claude Code 进阶实战：10 个官方技巧》和《Claude Code 的记忆与命令》。
