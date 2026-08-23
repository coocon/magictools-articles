# Claude Code Says No conversation found to continue — Your -p Sessions Are Being Filtered Out

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-no-conversation-found-to-continue-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-no-conversation-found-to-continue-en?utm_source=github&utm_medium=referral)**

## What the official docs say

The CLI reference gives `--continue` a very simple meaning: **continue the most recent conversation in the current directory**. No qualifiers — nothing about whether that session was started interactively or produced by a headless `claude -p "..."` run.

That is exactly the assumption many automation workflows are built on: scripts do the bulk work with `-p`, and when something needs a human, you drop into interactive mode with `--continue` to take over.

## What actually happened

macOS + Claude Code v2.1.220, a clean empty directory, four steps (terminal is a real TTY):

```console
$ claude --max-turns 1 -p 'The answer is 42. Reply only with: OK'
OK

$ claude --continue
No conversation found to continue        # ← BUG: the session clearly exists

$ claude -p --continue "What did I ask earlier?"
42                                        # ← headless continue works fine

$ CLAUDE_CODE_ENTRYPOINT=sdk-cli claude --continue
# ← interactive UI opens normally, with the -p session history fully loaded
```

![Terminal transcript: claude -p returns OK, ls shows the session .jsonl sitting in ~/.claude/projects, bare claude --continue answers "No conversation found to continue", yet claude -p --continue recalls 42 from that same session](https://cdn.tools.cooconsbit.com/uploads/hermes/2026-08-09/1786292779000-f1c28cd5.png)

*Typeset from the real run on v2.1.226 (2026-08-10) — commands and output are reproduced verbatim, only the colours and spacing are ours. The `ls` line is the point: the session file the interactive mode says it can't find is sitting right there on disk.*

...

---

**[👉 Continue reading: Claude Code Says No conversation found to continue — Your -p Sessions Are Being Filtered Out](https://tools.cooconsbit.com/en/articles/claude-code-no-conversation-found-to-continue-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
