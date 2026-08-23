# Which Model Is Claude Code Actually Using? settings.json vs ANTHROPIC_MODEL vs --model, Tested

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-model-priority-tested-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-model-priority-tested-en?utm_source=github&utm_medium=referral)**

## Why this deserves scrutiny

A wrong model configuration does not error — the session runs fine, just on a different model. In GitHub issue [#82466](https://github.com/anthropics/claude-code/issues/82466), a user configured Fable in `~/.claude/settings.json`, ran multi-agent workloads for a full day silently on Sonnet, and had to discard everything.

The official docs describe the `model` field in `settings.json`, the `ANTHROPIC_MODEL` environment variable, the `--model` flag, and the `/model` command separately — but **never one table saying who wins when they coexist**. Here are the measured results.

## Method: hard evidence from modelUsage

In headless mode with `--output-format json`, the `modelUsage` key in the result is the **model ID you were actually billed for** — more reliable than any UI display or the model's self-description (what a model says about itself can be shaped by system prompts and does not count as evidence):

```bash
claude -p "reply ok" --output-format json | python3 -c \
  "import json,sys; print(list(json.load(sys.stdin)['modelUsage'].keys()))"
```

## The tested priority matrix

Environment: macOS + Claude Code v2.1.220. Each row is an independent experiment in a clean directory, evidence taken from `modelUsage`:

| Configuration | Actually used | Conclusion |
|---|---|---|
| Project `.claude/settings.json` set to haiku, nothing else | haiku | ✅ project-level settings honored |
| settings set to `claude-fable-5[1m]` (1M-context suffix) | `claude-fable-5[1m]` | ✅ the `[1m]` suffix form works too |
| settings haiku + env `ANTHROPIC_MODEL=sonnet` | sonnet | env var **overrides** settings |
| settings haiku + flag `--model sonnet` | sonnet | flag **overrides** settings |
| env haiku + flag `--model sonnet` | sonnet | flag **overrides** env var |

...

---

**[👉 Continue reading: Which Model Is Claude Code Actually Using? settings.json vs ANTHROPIC_MODEL vs --model, Tested](https://tools.cooconsbit.com/en/articles/claude-code-model-priority-tested-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
