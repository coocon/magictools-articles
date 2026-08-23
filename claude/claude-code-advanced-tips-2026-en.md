# 10 Advanced Claude Code Techniques: Session Recovery, Checkpoint Rewind, and Headless CI

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-advanced-tips-2026-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-advanced-tips-2026-en?utm_source=github&utm_medium=referral)**

There's a gap between using Claude Code and using it well — and that gap is mostly a set of official doc pages nobody opens. This post skips the basics (see our Claude Code quickstart guide for that) and covers 10 techniques that genuinely change your workflow. Everything comes from the official documentation, and every item follows the same structure: how to do it, then when to use it.

## Table of Contents

1. Name your sessions so you can always find them
2. /rewind: a time machine that restores code and conversation separately
3. The context-slimming trio: /clear, /compact, /context
4. Path-scoped rules
5. Auto memory: let Claude take its own notes
6. Headless mode: wire Claude Code into scripts and CI
7. Structured output and cost accounting
8. The four-phase workflow: explore → plan → implement → commit
9. Rich context: screenshots, @files, and piped logs
10. Skills: turn repeated procedures into a single command

## 1. Name Your Sessions So You Can Always Find Them

The biggest annoyance of juggling tasks is "where did yesterday's half-finished session go?" The fix is naming things from the start:

```bash
claude -n payment-refactor     # name it at launch
/rename payment-refactor       # rename mid-session
claude --continue              # resume the most recent session
claude --resume                # open the session picker and choose by name
```

Add `/branch <name>` and you can fork a copy of the current conversation — the main line keeps shipping while the branch explores an alternative approach, both sharing everything discussed so far.

**When to use it**: any task that spans more than a day, and anyone running two or more tasks in parallel.

## 2. /rewind: A Time Machine That Restores Code and Conversation Separately

Claude Code automatically snapshots your workspace every time you send a message (checkpointing). When an edit goes sideways, you don't need a frantic `git stash` rescue:

- Press `Esc` twice, or type `/rewind`, to open the rewind menu
- Pick any point in history to restore
- Crucially, you can restore **code only** (the conversation survives, so Claude remembers what went wrong) or **conversation only** (the code survives, and you re-ask with better framing)

**When to use it**: before letting Claude attempt something aggressive — which requires doing nothing, because snapshots are automatic. That's exactly where the confidence to experiment comes from. One caveat: checkpoints are a session-level undo button, not version control. They don't replace git.

...

---

**[👉 Continue reading: 10 Advanced Claude Code Techniques: Session Recovery, Checkpoint Rewind, and Headless CI](https://tools.cooconsbit.com/en/articles/claude-code-advanced-tips-2026-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
