---
title: "Claude Code Hooks: Custom Automation Workflows"
slug: "claude-code-hooks-en"
category: "claude"
tags: [claude-code, hooks, automation, advanced]
summary: "Deep dive into Claude Code's Hooks system. Learn to auto-run custom scripts before and after tool calls to build automation workflows that match your team's standards."
status: "published"
---

## What Are Hooks

Hooks are Claude Code's automation extension mechanism. They let you run custom scripts automatically when specific events occur — before or after Claude calls a tool. Use hooks to auto-format code, block dangerous commands, or send notifications.

Hooks execute deterministically on your local machine without consuming LLM tokens, making them ideal for building reliable automation workflows.

## Hook Event Types

| Event | When It Fires |
|-------|--------------|
| `PreToolUse` | Before a tool call executes |
| `PostToolUse` | After a tool call completes |
| `Notification` | When Claude sends a notification |
| `Stop` | When Claude finishes a response |

## Configuration

Configure hooks in `.claude/settings.json`:

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

The `matcher` field supports regex to match tool names like `Edit`, `Write`, or `Bash`.

## Practical Examples

### Auto-Lint After File Save

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

### Block Dangerous Commands

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

When a hook script exits with a non-zero status, the corresponding tool call is blocked.

### Send Notification on Completion

```json
{
  "Stop": [
    {
      "matcher": "",
      "hook": {
        "type": "command",
        "command": "osascript -e 'display notification \"Claude finished the task\" with title \"Claude Code\"'"
      }
    }
  ]
}
```

## Best Practices

- **Keep scripts fast**: Hooks block Claude's execution, so avoid long-running tasks
- **Handle errors gracefully**: Use `|| true` to prevent non-critical failures from blocking the workflow
- **Leverage environment variables**: Claude injects `$CLAUDE_FILE_PATH`, `$CLAUDE_TOOL_INPUT`, and other context variables
- **Share with your team**: Put hooks in the project-level `.claude/settings.json` and commit to the repository

## FAQ

### What is the difference between Hooks and MCP?

Hooks are deterministic local scripts that trigger on specific events, ideal for automated checks and formatting. MCP is a protocol that lets Claude access external tools and data sources on demand. They are complementary — hooks for automation rules, MCP for extended capabilities.

### What happens if a hook script fails?

If a `PreToolUse` hook exits with a non-zero status, the tool call is blocked. If a `PostToolUse` hook fails, Claude receives the error but continues with subsequent operations.

### What environment variables are available in hooks?

Claude Code automatically injects context variables including `CLAUDE_FILE_PATH` (the file being operated on) and `CLAUDE_TOOL_INPUT` (the tool's input parameters). Available variables depend on the event type and tool being used.
