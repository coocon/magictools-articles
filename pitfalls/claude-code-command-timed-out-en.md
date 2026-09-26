# Claude Code \"Command timed out after 2m 0s\": Two Timeout Paths, BASH_DEFAULT_TIMEOUT_MS and run_in_background Tested

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-command-timed-out-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-command-timed-out-en?utm_source=github&utm_medium=referral)**

## Background

Ask Claude Code to run something slow — a cold `npm install`, a full test suite, `docker build`, a `sleep`-based poll — and after two minutes the tool result turns into:

```
Exit code 143
Command timed out after 2m 0s
```

That is the Bash tool's default timeout of 120000 ms. The usual advice is "raise the timeout", but that leaves at least three questions open:

- When the command is killed, is the output it already printed kept? Do child processes survive as orphans?
- When a script runs `claude -p`, how do you even notice that a command timed out?
- When should you use `BASH_DEFAULT_TIMEOUT_MS`, `BASH_MAX_TIMEOUT_MS`, a per-command `timeout`, or `run_in_background`?

Every error string and number below comes from 19 real `claude -p` sessions run on Claude Code 2.1.280 on 2026-09-26, copied verbatim.

## Analysis

I first read the relevant parts of the 2.1.280 binary (`claude.exe`, 217,254,576 bytes):

- Defaults: `var p=120000,d=600000`. `BASH_DEFAULT_TIMEOUT_MS` is only honored when `!isNaN(o)&&o>0`, otherwise it falls back to 120000. `BASH_MAX_TIMEOUT_MS` is `Math.max(max, default)`, so the ceiling is never lower than the default.
- Error template: `Command timed out after ${zt(this.#u)}`.
- A per-command timeout becomes `Math.min(requested||default, ceiling, …)`. When it gets clamped, only a `timeout_clamped` telemetry event is emitted — the model never sees it.
- **Auto-backgrounding on timeout**: when `!or&&Ee===void 0&&kYr(Be)` holds, a timed-out command is not killed but moved to the background. `kYr` takes the first word of the command and passes unless it is in `gYr=["sleep"]`. `or` is true when, among other things, `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` is set.

That last one was the surprise. **In 2.1.280 a Bash timeout takes one of two completely different paths:**

| Path | When | Tool result | `is_error` |
|---|---|---|---|
| Killed | First word is `sleep`, or `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1` | `Exit code 143\nCommand timed out after 2m 0s` | `true` |
| Backgrounded | Anything else, e.g. `echo …; sleep 300`, `(sleep 301; echo X) \| cat` | `Command did not complete within its 120s timeout and was moved to the background (ID: …)` | `false` |

So in 2.1.280 you only see "Command timed out after 2m 0s" in the first case. The experiments below use `sleep 300` to reproduce it reliably.

## Options

When two minutes is not enough there are three ways around it, each with a different cost:

| Option | Cost | Good for |
|---|---|---|
| `BASH_DEFAULT_TIMEOUT_MS` env var (plus `BASH_MAX_TIMEOUT_MS`) | Applies to every command, so a genuinely hung command also waits the full new limit; `0` or non-numeric values silently fall back to 120s; the ceiling defaults to 600000 ms, so going past 10 minutes needs MAX raised too | Sessions that are all long builds; headless CI-style jobs |
| Per-command `timeout` parameter (capped at `BASH_MAX_TIMEOUT_MS`, default 600000) | Relies on the model passing it; anything above the ceiling is **silently clamped** | A few known slow commands, e.g. one long test run |
| `run_in_background` | Returns immediately, but the completion notification carries no output, so you must read the file; in `-p` mode it is killed 5s after the final reply; in my run the model **fabricated a completion notification** | Interactive long tasks; under `-p` the model must poll in the foreground until done |

...

---

**[👉 Continue reading: Claude Code \"Command timed out after 2m 0s\": Two Timeout Paths, BASH_DEFAULT_TIMEOUT_MS and run_in_background Tested](https://tools.cooconsbit.com/en/articles/claude-code-command-timed-out-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
