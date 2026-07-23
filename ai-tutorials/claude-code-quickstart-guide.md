---
title: "Claude Code 快速上手指南：你的第一次实战会话"
slug: "claude-code-quickstart-guide"
category: ai-tutorials
tags:
  - claude code
  - anthropic
  - 终端
  - 编程
summary: "从安装、登录到第一次提问与修改代码，帮助你用最短路径上手 Claude Code。"
coverImage: ""
status: published
scheduledAt: ""
---

Claude Code 是 Anthropic 推出的终端式编程助手。想快速看到效果，不需要先把所有命令背下来。更有效的方式是先完成一次干净的首次会话，让 Claude 先读懂代码库，再去执行一个明确的改动任务。

Anthropic 的快速上手与概览文档给出的路径很直接：安装 CLI、使用 Claude.ai 或 Anthropic Console 账号登录、进入项目目录，然后运行 `claude` 开始会话。之后，Claude 可以帮助你理解代码库、提出修改方案、运行测试，并在同一个对话里继续推进。

## 开始前准备什么

在打开 Claude Code 之前，先准备好下面几项：

1. 一个可用的终端窗口
2. 一个真实的项目仓库
3. Claude.ai 或 Anthropic Console 账号
4. 如果打算用 npm 安装，需要 Node.js 18 或更高版本

这些就够了。第一次体验不需要复杂的配置流程，也不需要先准备额外的项目模板。

## 安装并启动 Claude Code

Anthropic 文档提供了两种安装方式：

```bash
npm install -g @anthropic-ai/claude-code
```

或者使用新的原生安装方式：

```bash
curl -fsSL claude.ai/install.sh | bash
```

安装完成后，进入项目目录并启动：

```bash
cd /path/to/your/project
claude
```

如果需要登录，Claude 会在会话中提示你完成认证。认证成功后，就可以直接开始使用。

## 先问代码库本身的问题

第一次提问最好是“描述性”的，而不是直接要求修改。可以先问 Claude：

- 这个项目是做什么的？
- 主入口文件在哪里？
- 这个仓库用了哪些技术栈？
- 请解释一下文件夹结构

这样做的好处是让 Claude 先读取项目上下文，再进入修改阶段。它会更像一个同事，而不是只能靠猜测回答的工具。

## 做第一次修改

等 Claude 理解了代码库，再给它一个具体任务：

```text
这里有一个 bug：用户可以提交空表单。请找出原因并修复。
```

Anthropic 对典型工作流的描述很清楚：Claude 会定位相关代码、理解上下文、实现修复，并在可用时运行测试。这就是你应该建立的使用预期。

## 用后续追问控制节奏

Claude Code 最适合“边做边调整”：

- 大改动前先让它列计划
- 对风险较高的改动要求先解释
- 输出范围太大时让它收缩
- 让它用测试或更小的复现来验证结果

第一周真正要养成的习惯，不是立刻得到完美答案，而是把工作拆成可检查的小步骤。

## 常用命令

Anthropic 的快速上手文档里列了几个最常用的命令：

```bash
claude
claude "fix the build error"
claude -p "explain this function"
claude -c
claude -r
```

在会话中，`/help` 可以查看命令列表，`/clear` 可以重置对话历史，方便重新开始。

## 第一次会话检查清单

建议按下面的顺序尝试：

1. 安装 CLI
2. 登录
3. 打开一个真实项目
4. 让 Claude 解释仓库
5. 交给它一个小而明确的任务
6. 先审查结果，再决定是否扩大范围

这个流程刻意保持简单，就是为了让你先了解 Claude 在你的环境里是怎么工作的，再逐步把它用到更大的改动中。

## 官方参考资料

- [Quickstart](https://docs.anthropic.com/en/docs/claude-code/quickstart)
- [Claude Code overview](https://docs.anthropic.com/en/docs/claude-code/overview)
- [Slash commands](https://docs.anthropic.com/en/docs/claude-code/slash-commands)

以上资料检索于 2026年3月29日。功能可用性、套餐限制和界面细节可能会变化，发布前请以链接中的 Anthropic 官方资料为准。
