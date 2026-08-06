---
title: "Which Model Is Claude Code Actually Using? settings.json vs ANTHROPIC_MODEL vs --model, Tested"
slug: claude-code-model-priority-tested-en
category: claude
locale: en
translationSlug: claude-code-model-priority-tested
tags: [Claude Code, model configuration, settings.json, ANTHROPIC_MODEL, CLI, claude-code-lab]
summary: "Multi-agent setups, CI, and batch scripts all rest on one assumption: the session actually runs on the model you configured. But the model can be set in four places — settings.json, the ANTHROPIC_MODEL environment variable, the --model flag, and /model in-session — and the docs never give you one table saying which wins. Meanwhile a GitHub issue reports settings.json silently ignored on Windows, with a full day of work run on the wrong model and discarded. Using modelUsage as hard evidence, we tested the chain layer by layer: --model > ANTHROPIC_MODEL > project settings.json > built-in default, with the [1m] suffix form working too — plus a verification method more trustworthy than any UI hint."
status: published
lab:
  testedAt: "2026-08-06"
  ccVersion: "2.1.220"
  model: "haiku-4-5 / sonnet-5 / fable-5[1m] (matrix-tested)"
  platform: macOS
  status: reproducible
docsUrl: https://docs.claude.com/en/docs/claude-code/settings
---

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

The resulting priority chain (high → low):

```
--model flag  >  ANTHROPIC_MODEL env  >  project settings.json  >  built-in default
```

It matches the Unix intuition that configuration closer to the invocation wins — but until now that was intuition. Now it is evidence.

## About that Windows issue

Issue #82466 reports a different layer: the **user-level** `~/.claude/settings.json` (note: not project-level) being ignored on Windows 11 + PowerShell, and `/model <exact-id>` in interactive sessions replying "Kept model as X" and refusing to switch. The reporter ruled out env vars, project overrides, and every known on-disk location, and suspects some client-side UI state that never hits disk.

We **could not reproduce it** on macOS (every layer worked, as the table shows), and the issue has no official response at the time of writing. If your model looks wrong on Windows: do not assume you misconfigured it — collect evidence with the `modelUsage` method above, then add your environment details to that issue. Every cross-environment report moves the fix closer.

## Practical recommendations

- **CI / scripts**: pass `--model` explicitly — highest priority, smallest scope, immune to any persisted state
- **Project-wide default**: use the project's `.claude/settings.json` (checked into the repo, shared with the team) — verified reliable
- **Verification**: whenever a model "feels wrong", run one `--output-format json` call and read `modelUsage`; in interactive sessions, run `/model` with no argument and **trust the highlighted item in the picker** — the issue reporter found the "Kept model as X" text misleading, while the picker highlight reflects the real current state
- **Cost-sensitive setups**: `modelUsage` also carries token counts and cost, handy for auditing bills

## Scope and boundaries

- Test environment is in the scope card above; the matrix covers headless (`-p`) with project-level settings / env / flag. The **user-level** `~/.claude/settings.json` and interactive `/model` switching are outside this article's tested scope (the former is unsafe to mutate on a production machine, the latter is discussed in the issue)
- Bedrock / Vertex channels have their own model ID schemes and env vars; conclusions here apply to direct Anthropic API/subscription only
- Priority ordering is stable design semantics and should hold across versions; we will update the status here once the Windows issue is fixed
