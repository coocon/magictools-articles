# Claude Code 快速上手指南：你的第一次实战会话

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-quickstart-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-quickstart-guide?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：Claude Code 快速上手指南：你的第一次实战会话](https://tools.cooconsbit.com/zh/articles/claude-code-quickstart-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
