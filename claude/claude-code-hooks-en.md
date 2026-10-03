# Claude Code Hooks: Custom Automation Workflows

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-hooks-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-hooks-en?utm_source=github&utm_medium=referral)**

This article originally covered only the concepts. On 2026-09-19 I added the "Hands-On Lab" section at the end: a fully isolated hook laboratory on Claude Code 2.1.270, with all 8 events wired to the same capture script, every stdin JSON dumped to disk, and a hook-marker-file plus tool-marker-file pair to confirm independently whether the tool actually ran. The headline: **three things in the body below are wrong — only `exit 2` blocks a tool (`exit 1` and `exit 3` both let it through), the singular `"hook": {…}` config key is silently ignored, and neither `$CLAUDE_FILE_PATH` nor `$CLAUDE_TOOL_INPUT` exists in 2.1.270.** The original text is left untouched; the lab section below gives the real readings and the corrected patterns.

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

...

---

**[👉 Continue reading: Claude Code Hooks: Custom Automation Workflows](https://tools.cooconsbit.com/en/articles/claude-code-hooks-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
