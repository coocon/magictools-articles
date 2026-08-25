# You Set ANTHROPIC_BASE_URL. Claude Code Ignored It.

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-proxy-baseurl-priority-pitfall-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-proxy-baseurl-priority-pitfall-en?utm_source=github&utm_medium=referral)**

## The Symptom

Point Claude Code at a third-party API gateway — the one-api / new-api family, anything that speaks the Anthropic wire format at your own URL. The setup looks textbook. Three lines in `~/.zshrc`:

```bash
export ANTHROPIC_BASE_URL="https://xxxx.example.com"
export ANTHROPIC_AUTH_TOKEN="sk-..."
export CLAUDE_CODE_USE_VERTEX=0
```

New shell, run `claude`, and the header says:

```
Fable 5 · Google Vertex AI
```

Vertex. The gateway's base URL might as well not exist.

Same day, a second problem. CloudCLI — the renamed claudecodeui, a web UI that drives the local `claude` binary remotely — runs under macOS launchd (the Mac's equivalent of systemd). Log in, and it keeps asking me to pick a provider, claiming it isn't authenticated. Meanwhile the CLI in my terminal is happily hitting the gateway. Same user. Same machine.

Two symptoms, two unrelated root causes, one thing in common: **you think the env var is set; the process disagrees.**

## Pitfall 1: settings.json `env` Outranks Your Shell Export

First, verify the shell itself. `echo $ANTHROPIC_BASE_URL` — correct. `echo $CLAUDE_CODE_USE_VERTEX` — `0`. Nothing wrong at that layer.

So go read Claude Code's own config. Sitting in the `env` block of `~/.claude/settings.json`, three lines of leftover test config:

```json
{
  "env": {
    "CLAUDE_CODE_USE_VERTEX": "1",
    "ANTHROPIC_VERTEX_PROJECT_ID": "hello",
    "CLOUD_ML_REGION": "global"
  }
}
```

`ANTHROPIC_VERTEX_PROJECT_ID` is literally `hello`. Obvious placeholder from a Vertex experiment months ago, never cleaned up.

That's the root cause: **the `env` field in `settings.json` is a forced injection, and it outranks anything you export in your shell.** The `CLAUDE_CODE_USE_VERTEX=0` in `.zshrc` gets overwritten with `"1"` at process start. Claude Code sees a complete Vertex config, takes the Vertex path, and never so much as looks at `ANTHROPIC_BASE_URL` — that variable isn't on the code path it chose.

...

---

**[👉 Continue reading: You Set ANTHROPIC_BASE_URL. Claude Code Ignored It.](https://tools.cooconsbit.com/en/articles/claude-code-proxy-baseurl-priority-pitfall-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
