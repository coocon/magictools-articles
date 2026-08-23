# Claude Code Hooks: Custom Automation Workflows

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-hooks-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-hooks-en?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 Continue reading: Claude Code Hooks: Custom Automation Workflows](https://tools.cooconsbit.com/en/articles/claude-code-hooks-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
