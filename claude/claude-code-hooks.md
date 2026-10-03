# Claude Code Hooks：自定义自动化工作流

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-hooks?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-hooks?utm_source=github&utm_medium=referral)**

这篇文章原本只讲原理。2026-09-19 我补了文末的「本机实测」一节：在 Claude Code 2.1.270 上真搭了一个隔离的 hook 实验室，8 类事件全挂同一个脚本，把 stdin JSON 原样落盘，再用「hook 标记文件 + 工具标记文件」双向确认工具到底跑没跑。先说结论：**上面这篇正文里有三处写错了——只有 `exit 2` 会拦住工具（`exit 1` 和 `exit 3` 一律放行）、单数 `"hook": {…}` 这个配置键已经被静默忽略、`$CLAUDE_FILE_PATH` / `$CLAUDE_TOOL_INPUT` 两个环境变量在 2.1.270 上根本不存在。** 原文保留在下面不动，实测一节逐条给出真实读数与修正写法。

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

`matcher` 字段支持正则表达式，用于匹配工具名称（如 `Edit`、`Write`、`Bash` 等）。

## 实战示例

### 文件保存后自动 Lint

```json
{
  "PostToolUse": [
    {
      "matcher": "Edit|Write",
      "hook": {
        "type": "command",
        "command": "npx eslint --fix $CLAUDE_FILE_PATH 2>/dev/null || true"
      }
    }
  ]
}
```

### 阻止危险命令

```json
{
  "PreToolUse": [
    {
      "matcher": "Bash",
      "hook": {
        "type": "command",
        "command": "if echo \"$CLAUDE_TOOL_INPUT\" | grep -qE 'rm\\s+-rf\\s+/|DROP\\s+DATABASE'; then echo 'BLOCKED: Dangerous command detected' >&2; exit 1; fi"
      }
    }
  ]
}
```

当 hook 脚本以非零状态退出时，对应的工具调用将被阻止。

### 完成时发送通知

```json
{
  "Stop": [
    {
      "matcher": "",
      "hook": {
        "type": "command",
        "command": "osascript -e 'display notification \"Claude 完成了任务\" with title \"Claude Code\"'"
      }
    }
  ]
}
```

## 最佳实践

- **保持脚本快速**：Hook 脚本会阻塞 Claude 的执行流程，避免长时间运行的任务
- **妥善处理错误**：使用 `|| true` 防止非关键脚本失败阻塞工作流
- **善用环境变量**：Claude 会注入 `$CLAUDE_FILE_PATH`、`$CLAUDE_TOOL_INPUT` 等上下文变量
- **团队共享**：将 hooks 配置放在项目级 `.claude/settings.json` 中，提交到仓库供团队共用

## 常见问题

### Hooks 和 MCP 有什么区别？

Hooks 是确定性的本地脚本，在特定事件时自动触发，适合自动化检查和格式化。MCP 则是让 Claude 访问外部工具和数据源的协议，Claude 可以主动调用 MCP 提供的工具。两者互补，不冲突。

### Hook 脚本执行失败会怎样？

如果 `PreToolUse` 的 hook 脚本以非零状态退出，对应的工具调用将被阻止。如果 `PostToolUse` 的脚本失败，Claude 会收到错误信息但会继续执行后续操作。

...

---

**[👉 继续阅读全文：Claude Code Hooks：自定义自动化工作流](https://tools.cooconsbit.com/zh/articles/claude-code-hooks?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
