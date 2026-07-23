---
title: "Claude Code 的记忆与命令：CLAUDE.md、斜杠命令和工作流快捷方式"
slug: "claude-code-memory-and-commands-guide"
category: ai-tutorials
tags:
  - claude code
  - anthropic
  - 记忆
  - 斜杠命令
summary: "了解 Claude Code 的记忆系统与斜杠命令，帮助你把项目规则、个人偏好和常用工作流组织起来。"
coverImage: ""
status: published
scheduledAt: ""
---

当你不再在每次会话里重复同样的指令时，Claude Code 才会真正变得高效。Anthropic 的记忆系统和斜杠命令正好解决了这个问题。前者负责跨会话保留上下文，后者负责在会话中快速执行常见操作。

这样一来，工作流会稳定很多：项目规则放在 `CLAUDE.md` 里，个人偏好放在用户记忆里，常用命令则可以在会话中快速调用，而不用每次重新写长提示词。

## Claude Code 的记忆是怎么工作的

Anthropic 把记忆分成了多个层级：

1. 企业策略记忆，用于组织级规则
2. 项目记忆，放在 `./CLAUDE.md`
3. 用户记忆，放在 `~/.claude/CLAUDE.md`

这种结构的意义在于：团队规范和个人习惯可以分开管理。你打开项目时，Claude 就能自动读取合适的说明。

Claude Code 还会从当前工作目录向上递归查找记忆文件，这对大型仓库特别有用。你也可以用 `/memory` 查看当前加载了哪些记忆。

## 项目记忆里应该放什么

`CLAUDE.md` 适合放那些应该跟着代码库一起走的说明：

- 构建和测试命令
- 命名规范
- 架构说明
- 团队约定
- 常见工作流提示

目标很简单：把你每次开会话都要重复的说明保存下来，减少重复沟通，也减少输出不一致。

## 如何快速添加或编辑记忆

Anthropic 提供了两种很方便的方式：

- 在消息前面加 `#`，快速添加一条记忆
- 用 `/memory` 打开对应的记忆文件并直接编辑

示例：

```text
# Always prefer small, reviewable changes
```

这样就可以在不中断工作流的情况下记录一个简单偏好。

## 最常用的斜杠命令

Anthropic 官方特别强调的内置命令，在日常使用中非常实用：

- `/init`：为项目生成 `CLAUDE.md` 指南
- `/clear`：清空当前对话历史
- `/compact`：当会话太长时压缩上下文
- `/config`：查看或修改配置
- `/mcp`：管理 MCP 连接
- `/model`：切换模型
- `/permissions`：查看或调整权限
- `/help`：查看可用命令

这些命令能让你不离开当前会话就完成调整，不必每次都重新输入长提示词。

## 面向重复任务的自定义命令

Anthropic 也支持用 Markdown 文件定义自定义斜杠命令。对于经常重复的工作，这非常有用。

项目命令放在 `.claude/commands/`，个人命令放在 `~/.claude/commands/`。你可以把它们用在这些场景里：

- 代码审查清单
- 安全检查提示
- 重构说明
- 文档更新流程

如果你经常让 Claude 做同类任务，把它做成命令通常比写一大段提示词更好，因为更容易复用，也更容易保持一致。

## 一个简单的记忆策略

可以用三层规则来管理：

1. 团队规则放项目记忆
2. 个人默认设置放用户记忆
3. 一次性要求放当前会话

这样 Claude Code 会更稳定，也能避免你把所有内容都堆进一个提示词里。

## 官方参考资料

- [Manage Claude's memory](https://docs.anthropic.com/en/docs/claude-code/memory)
- [Slash commands](https://docs.anthropic.com/en/docs/claude-code/slash-commands)
- [Quickstart](https://docs.anthropic.com/en/docs/claude-code/quickstart)

以上资料检索于 2026年3月29日。功能可用性、套餐限制和界面细节可能会变化，发布前请以链接中的 Anthropic 官方资料为准。
