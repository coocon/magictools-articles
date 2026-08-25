# The Claude Code Hooks Stdin Trap: Python Heredocs Eat Your Hook JSON

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-hooks-stdin-trap-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-hooks-stdin-trap-en?utm_source=github&utm_medium=referral)**

## What the official docs say

The Claude Code hooks mechanism is straightforward: register a command in `settings.json`, and when an event fires (say `PostToolUse`), Claude Code launches that command and **passes the event data as JSON via stdin**. The docs are concise — a hook reads JSON from stdin, does its work, and communicates back through exit codes.

Standard Unix piping. Building a "log every failed command" hook should take ten minutes.

## What actually happened

The goal: whenever a Bash command fails, append it to a `tasks/lessons-inbox.md` inbox file (with dedup and benign-failure filtering). The logic is not complex, but it is verbose in shell, so the natural choice was to hand it to python. The first version of the hook looked like this:

```sh
#!/bin/sh
python3 - <<'PYEOF' 2>/dev/null
import json, sys

data = json.load(sys.stdin)   # read the hook JSON — or so you think
cmd = data.get("tool_input", {}).get("command", "")
# ... detect failure, append to inbox ...
PYEOF
exit 0
```

It ran without a single error. I then deliberately triggered several failing commands — the inbox file stayed empty. No error, no log output, the `settings.json` config triple-checked and fine. The hook simply "did not work".

## The trap: the heredoc redirects stdin

Trace the data flow of that command and the bug becomes obvious:

1. Claude Code launches the hook process with the event JSON attached to the process's **stdin**
2. `python3 -` means "read the **program itself** from stdin"
3. The `<<'PYEOF'` heredoc **redirects** python's stdin to the heredoc content — the python script

...

---

**[👉 Continue reading: The Claude Code Hooks Stdin Trap: Python Heredocs Eat Your Hook JSON](https://tools.cooconsbit.com/en/articles/claude-code-hooks-stdin-trap-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
