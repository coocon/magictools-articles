# Claude Code Hooks：自定义自动化工作流

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-hooks?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-hooks?utm_source=github&utm_medium=referral)**

## 什么是 Hooks

Hooks 是 Claude Code 提供的自动化扩展机制，允许你在特定事件发生时自动执行自定义脚本。通过 Hooks，你可以在 Claude 调用工具之前或之后插入自己的逻辑，比如自动格式化代码、阻止危险命令或发送通知。

Hooks 在你的本地机器上以确定性方式执行，不消耗 LLM 的 token，是构建可靠自动化工作流的理想方式。

## Hook 事件类型

| 事件 | 触发时机 |
|------|---------|
| `PreToolUse` | 工具调用执行之前 |
| `PostToolUse` | 工具调用执行之后 |
| `Notification` | Claude 发送通知时 |
| `Stop` | Claude 完成响应时 |

## 配置方法

Hooks 在 `.claude/settings.json` 中配置：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hook": {
          "type": "command",
          "command": "echo 'File is about to be modified'"
        }
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hook": {
          "type": "command",
          "command": "npx prettier --write $CLAUDE_FILE_PATH"
        }
      }
    ]
  }
}
```

...

---

**[👉 继续阅读全文：Claude Code Hooks：自定义自动化工作流](https://tools.cooconsbit.com/zh/articles/claude-code-hooks?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
