# Claude Code MCP server Failed to connect: CONNECTION_CLOSED, connection timed out after 30000ms and ENOENT, reproduced one by one

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-mcp-failed-to-connect-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-mcp-failed-to-connect-en?utm_source=github&utm_medium=referral)**

## Background

You add an MCP server to Claude Code, run `claude mcp list`, and get:

```
✘ Failed to connect — CONNECTION_CLOSED: Connection closed
```

Or `ENOENT`, or `connection timed out after 30000ms`, or `ECONNREFUSED`, or `claude -p` refusing to start with `Error: Invalid MCP configuration:`. All of them mean the same thing — "can't connect" — and none of them tells you what to fix.

This post turns "can't connect" back into specific, checkable causes: which exact error text each cause produces, how to tell them apart in about a minute, and what fixes each one. Every error string below was captured verbatim from real runs on my machine on 2026-09-24.

## What can actually go wrong

Getting from a config entry to a usable MCP server takes four steps, and each fails with a different error:

1. **Read the config** — does the `--mcp-config` file exist, is it valid JSON, is the shape right
2. **Spawn / connect** — a stdio server spawns a command (missing command → ENOENT); an http/sse server connects to a URL (nothing listening → ECONNREFUSED)
3. **Stay alive** — if the process exits right away, all Claude Code sees is a closed pipe: `CONNECTION_CLOSED`
4. **MCP handshake** — the process is alive but must answer `initialize` before the timeout, or you get `CONNECT_TIMEOUT`

Steps 3 and 4 are the hard ones: the error tells you *where* it failed, never *why*. That's also where the most useful findings of this experiment are.

## Approach and what I ruled out

**Every server under test is a hand-written stub**, three in total:

- `good-server.mjs` — a 17-line, zero-dependency MCP stdio server implementing only `initialize` / `tools/list` / `tools/call`, with a single `echo` tool that returns `LAB-ECHO:<text>`
- `exit1-server.sh` — prints `lab-exit1: fatal: missing LAB_API_KEY, refusing to start` to stderr, then `exit 1`
- `hang-server.mjs` — prints one line to stderr and then never replies; exits by itself after 300 seconds

**Ruled out: testing with real third-party MCP servers.** When a real server fails, the failure is tangled up with its own logic, network and dependency versions, so you can't tell which layer broke. `npx -y` also downloads code at run time, which makes runs unreproducible and adds supply-chain risk. Stubs change exactly one variable per case, and the `LAB-ECHO:` prefix proves a tool result really came from the server rather than being made up by the model.

**Ruled out: testing against my real config.** My `~/.claude.json` has real servers configured. Mixing them in would make it impossible to tell who is failing, and breaking them would break my daily setup. So each case gets its own empty `CLAUDE_CONFIG_DIR` (user-scoped servers go into a `.claude.json` inside that temp directory), and every `claude -p` run uses `--mcp-config <temp json> --strict-mcp-config`, except the one run that deliberately drops strict to test what it does.

...

---

**[👉 Continue reading: Claude Code MCP server Failed to connect: CONNECTION_CLOSED, connection timed out after 30000ms and ENOENT, reproduced one by one](https://tools.cooconsbit.com/en/articles/claude-code-mcp-failed-to-connect-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
