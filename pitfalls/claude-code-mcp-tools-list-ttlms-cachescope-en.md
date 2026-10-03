# Claude Code MCP Shows Connected but 0 Tools: \"Invalid result for tools/list\" (ttlMs / cacheScope), Reproduced and Fixed

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-mcp-tools-list-ttlms-cachescope-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-mcp-tools-list-ttlms-cachescope-en?utm_source=github&utm_medium=referral)**

## The problem

You configure an MCP server, and `claude mcp list` does not say `✘ Failed to connect`. It shows a yellow exclamation mark instead:

```
roblox-like: /tmp/mcp-ttl/roblox-like-server  - ! Connected · tools fetch failed — Invalid result for tools/list: [ { "expected": "number", "code": "invalid_type", "path": [ "ttlMs" ], "message": "Invalid input: expected number, received undefined" }, { "code": "invalid_value", "values": [ "public", "private" ], "path": [ "cacheScope" ], "message": "Invalid option: expected one of \"public\"|\"private\"" } ]
```

Inside a session, the server status is `connected`, yet the model cannot see a single tool from it. Restarting, opening a new session or rebooting does not help.

The matching GitHub issue is [anthropics/claude-code#97319](https://github.com/anthropics/claude-code/issues/97319) (opened 2026-09-26, still OPEN on 10-03), triggered by Roblox Studio's official MCP bridge. The reporter's diagnosis: the tools/list response carries extra fields from a newer protocol revision (`ttlMs`, `cacheScope`), and Claude Code's validation is too strict, so it throws away the whole response. Two workarounds came up in the comments: downgrade to 2.1.280, or set `MCP_PROTOCOL_NEGOTIATION=legacy`.

This article takes the error apart with a hand-written stub server. Are the fields extra or missing? Why does the same version break for some people and not others? Why does downgrading help? And what should users and server authors each do? Every error message below comes from real runs on my machine on 2026-10-03.

## Analysis

Start with the spec. In the [MCP 2026-07-28 schema](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/schema/2026-07-28/schema.ts), `ListToolsResult` extends both `PaginatedResult` and `CacheableResult`:

```ts
export interface CacheableResult extends Result {
  ttlMs: number;                     // how many ms the client may cache this; 0 = immediately stale
  cacheScope: "public" | "private";  // like HTTP Cache-Control public / private
}

export interface Result {
  _meta?: ResultMetaObject;
  resultType: ResultType;            // "complete" | "input_required" | string; required from this revision
  [key: string]: unknown;            // arbitrary extra fields are allowed
}
```

Two things matter here:

1. `ttlMs`, `cacheScope` and `resultType` are all **required** in this revision.
2. `Result` has `[key: string]: unknown`, so the spec explicitly **allows extra fields**.

Now look at the error. `"expected": "number"` with `invalid_type` points at `ttlMs`; `invalid_value` with `["public","private"]` points at `cacheScope`. If these were unknown extra fields, the validator would have no idea one should be a number and the other one of public/private. The error has this shape only when the client **validates against the 2026-07-28 schema and the server did not send those fields**. 2.1.288 says it outright: `expected number, received undefined`.

...

---

**[👉 Continue reading: Claude Code MCP Shows Connected but 0 Tools: \"Invalid result for tools/list\" (ttlMs / cacheScope), Reproduced and Fixed](https://tools.cooconsbit.com/en/articles/claude-code-mcp-tools-list-ttlms-cachescope-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
