---
title: "The Claude Code Hooks Stdin Trap: Python Heredocs Eat Your Hook JSON"
slug: claude-code-hooks-stdin-trap-en
category: claude
locale: en
translationSlug: claude-code-hooks-stdin-trap
tags: [Claude Code, hooks, PostToolUse, shell, python, troubleshooting, claude-code-lab]
summary: "The docs say hooks receive JSON via stdin — true. But parse it with a python heredoc (python3 - <<'EOF') and you hit a silent failure: the heredoc redirects python's stdin to the script itself, so json.load(sys.stdin) reads nothing. A best-practice hook then swallows the exception and exits 0 — no error anywhere, just a hook that 'somehow does not work'. The real debugging session, the two-line fix, and a lesson: silently fault-tolerant code is your enemy at debugging time."
status: published
lab:
  testedAt: "2026-08-05"
  ccVersion: "2.1.220"
  model: "claude-fable-5"
  platform: macOS
  status: reproducible
docsUrl: https://docs.claude.com/en/docs/claude-code/hooks
---

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

So python's stdin is fully occupied by the script text; once the program is read, the stream is at EOF. The `json.load(sys.stdin)` inside reads empty input and raises `JSONDecodeError` — which is then completely swallowed by `2>/dev/null` and the "hooks must never block" fault-tolerant design. On the Claude Code side, the hook exits 0. Everything looks "successful".

And the real hook JSON? It is still sitting on the outer sh process's stdin, but once the heredoc takes effect, python can never reach it.

What makes this trap nasty is **three layers of silence stacked together**: the heredoc is perfectly legal shell (no shell error), the python exception is discarded by the redirect (no runtime error), and the hook exits 0 per best practice (no Claude Code error). Each layer is correct design on its own; combined, they form a soundless black hole.

## The fix: spool to disk first, pass a path

The fix is two lines — **before** python starts, consume the outer process's stdin with `cat` into a temp file, then hand the file path in through an environment variable:

```sh
#!/bin/sh
TMP_IN="$(mktemp)" || exit 0
cat > "$TMP_IN" 2>/dev/null || true          # catch the hook JSON off stdin first
CL_HOOK_INPUT="$TMP_IN" python3 - <<'PYEOF' 2>/dev/null
import json, os, sys

try:
    data = json.load(open(os.environ["CL_HOOK_INPUT"], encoding="utf-8"))
except Exception:
    sys.exit(0)

if data.get("tool_name") != "Bash":
    sys.exit(0)
# ... failure detection, benign filtering, dedup by command hash, append to inbox ...
PYEOF
rm -f "$TMP_IN"
exit 0
```

It worked immediately: failed commands started landing in the inbox one by one, with dedup and filtering behaving as intended.

A few design points from the full version of this hook worth copying (all validated by real usage over time):

- **Never block**: every exceptional path ends in `exit 0`, including a failed `mktemp` — a broken hook must not drag down the main loop
- **Benign-failure filtering**: non-zero exits from `grep` / `rg` / `diff` / `test` are normal semantics, not worth recording
- **Dedup by command hash**: the same command failing repeatedly is recorded once, so the inbox never bloats

## Scope and boundaries

- This only affects the pattern of "feeding an interpreter its script via heredoc inside a hook" — `python3 - <<EOF` and `node - <<EOF` fail identically; if your python code lives in a separate file (`python3 hook.py`), stdin passes through untouched and there is no issue
- `python3 -c 'one-liner'` does not occupy stdin, so short logic can dodge the trap that way; beyond a few lines it becomes unmaintainable, and the spool-to-disk approach is sturdier
- Verified in the environment stated at the top of this page; hooks receiving JSON via stdin is a documented, stable contract, and this trap comes from shell semantics rather than Claude Code version behavior, so it should hold long-term

## The general lesson

Supporting-cast code like hooks is usually required to fail silently — rightly so, but **during development, take the `2>/dev/null` off first**. Had I seen `JSONDecodeError: Expecting value` early, locating the bug would have taken ten minutes; debugging through a full stack of silence cost an order of magnitude more. Fault tolerance is for production, not for troubleshooting.
