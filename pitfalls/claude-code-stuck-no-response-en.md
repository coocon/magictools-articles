# Claude Code Stuck on a Spinner With No Response: It Waits 6 Minutes per Attempt, and 10 Retries Can Hang It for Over an Hour (Tested)

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-stuck-no-response-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-stuck-no-response-en?utm_source=github&utm_medium=referral)**

> **Short answers first**
>
> - **Spinner with a growing timer (`Clauding… (4m 34s)`)**: Claude Code is waiting for the backend, which hasn't sent a byte. With an API key and `ANTHROPIC_BASE_URL` (gateway / relay), **it waits a full 6 minutes per attempt before timing out**.
> - **Stuck at `Retrying in 0s · attempt 1/10`**: the previous attempt timed out; the retry is now waiting another 6 minutes. It is not frozen.
> - **How long can it hang?** With the default 10 retries and a backend that never answers, it took about 69 minutes (4147.78 s) to finally show `Request timed out`.
> - **What to do now**: press `Esc` and send again. If it keeps happening, the problem is the backend or network — test your base URL directly with curl (see "What to do", item 1).
> - **Fail sooner**: set `API_TIMEOUT_MS=60000` (only lowering works; raising doesn't) and `CLAUDE_CODE_MAX_RETRIES=2`.
> - **Only long tasks hang, short questions are fine**: your gateway probably buffers the reply until generation finishes. A reply that takes over 6 minutes is cut off and retried every time — **it never arrives**. Use a gateway that streams, or split the task.
> - **Stuck on a command** (the UI shows a Bash call, not a spinner): different problem — see [Command timed out after 2m 0s](/en/articles/claude-code-command-timed-out-en).

## Background

"Claude Code is stuck" usually means a spinner verb and a timer, with no error:

```
❯ refactor this function
✢ Clauding… (4m 34s)
```

After a while it may change to this and then stop moving again:

```
✻ API error · Retrying in 0s · attempt 1/10
```

You can't tell whether it is working, waiting on the network, or dead. The docs list several timers under [Streaming idle watchdogs](https://code.claude.com/docs/en/network-config#streaming-idle-watchdogs) and [No response from API](https://code.claude.com/docs/en/errors#no-response-from-api), each with different conditions. So we reproduced each kind of stall and measured how long Claude Code waits and what it shows.

## What "stuck" can mean on the wire

| Where it stalls | What the backend does | Timer in the docs |
|---|---|---|
| ① No response headers | Accepts the connection but never sends HTTP headers (upstream queueing, a buffering gateway waiting for generation to finish) | The first-byte deadline (docs: **does not run when `ANTHROPIC_BASE_URL` is set**), otherwise `API_TIMEOUT_MS` (docs: default 10 minutes) |
| ② Headers, then silence | Sends `200` and SSE headers, then nothing | Byte-level watchdog (300 s default on gateways), event-level watchdog (300 s) |
| ③ Keep-alive pings only | Keeps sending `event: ping`, no content | Pings reset the byte-level watchdog; the event-level watchdog ignores them |
| ④ Stalls mid-response | Sends part of the answer, then stops | Byte-level watchdog |

...

---

**[👉 Continue reading: Claude Code Stuck on a Spinner With No Response: It Waits 6 Minutes per Attempt, and 10 Retries Can Hang It for Over an Hour (Tested)](https://tools.cooconsbit.com/en/articles/claude-code-stuck-no-response-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
