---
title: "Claude Code Hooks：自定义自动化工作流"
slug: "claude-code-hooks"
category: "claude"
tags: [claude-code, hooks, automation, advanced]
summary: "深入了解 Claude Code 的 Hooks 系统，学习如何在工具调用前后自动执行自定义脚本，构建符合团队规范的自动化工作流。"
status: "published"
---

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

### 可以在 Hook 中使用哪些环境变量？

Claude Code 会自动注入上下文相关的环境变量，包括 `CLAUDE_FILE_PATH`（当前操作的文件路径）、`CLAUDE_TOOL_INPUT`（工具的输入参数）等。具体可用变量取决于触发的事件和工具类型。
