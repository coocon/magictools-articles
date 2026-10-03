# Claude Code \"Invalid API key · Fix external API key\": Not logged in, Credit balance is too low, API Error 401/429/529 — Exact Messages and Retry Behavior, Tested

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-invalid-api-key-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-invalid-api-key-en?utm_source=github&utm_medium=referral)**

## Background

Point Claude Code at a compatible endpoint like DeepSeek, swap in a new key, or run out of credit, and `claude -p` will often print one of these:

```
Invalid API key · Fix external API key
Not logged in · Please run /login
Credit balance is too low
API Error: Request rejected (429) · …
API Error: 529 Overloaded. This is a server-side issue, usually temporary — …
```

The messages are readable enough. The hard questions are these:

- Did **Claude Code decide this locally**, or did **the backend return it**? Was a request even sent?
- Why does a wrong key make the command **hang for almost 3 minutes** before failing?
- In a script, is the error on stdout or stderr? What is the exit code? Which field in `--output-format json` should you check?
- If you run a third-party gateway, should you read `x-api-key` or `Authorization: Bearer`?

Every message and number below comes from runs on Claude Code 2.1.280 on this machine on 2026-09-27: 28 cases, 61 `claude -p` runs in total. **The backend is not the real API but a local stub**, so the status code can be anything I want, and the arrival time and headers of every request are recorded.

## Analysis

I first located the definitions of these messages in the 2.1.280 native binary (`claude.exe`, 217 MB):

```js
var _Ye="Not logged in \xB7 Please run /login",
    Gke="Invalid API key \xB7 Fix external API key",
    pat="Invalid auth token \xB7 Fix external auth token",
    mat="Invalid ANTHROPIC_CUSTOM_HEADERS \xB7 Fix the environment variable"
```

There is also a check: `function JM(e){return/^[a-zA-Z0-9-_]+$/.test(e)}`, which throws `Invalid API key format. API key must contain only alphanumeric characters, dashes, and underscores.` on failure.

From the static code alone it is tempting to conclude that "a key with special characters is rejected locally". **The test run disproved this**: `ANTHROPIC_API_KEY='bad key!@#'` was sent to the backend verbatim in `x-api-key`, the stub returned a normal response, and the run exited 0. Going back to the code, the only call site of `edo()` is in the OAuth API-key creation flow (`r.data?.raw_key` → `edo(s,n)`); a key from an environment variable never passes through it. So every conclusion here rests on the runs; static code is only used to explain what was observed.

Reading the code also turned up the logic that picks the message, later confirmed by the runs: whether a 401 shows `Invalid API key` depends on **whether the error message contains the string `x-api-key`**:

...

---

**[👉 Continue reading: Claude Code \"Invalid API key · Fix external API key\": Not logged in, Credit balance is too low, API Error 401/429/529 — Exact Messages and Retry Behavior, Tested](https://tools.cooconsbit.com/en/articles/claude-code-invalid-api-key-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
