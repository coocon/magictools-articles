# Claude Code Skills 完全指南：用 SKILL.md 把重复工作流变成一条斜杠命令

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-skills-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-skills-guide?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：Claude Code Skills 完全指南：用 SKILL.md 把重复工作流变成一条斜杠命令](https://tools.cooconsbit.com/zh/articles/claude-code-skills-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
