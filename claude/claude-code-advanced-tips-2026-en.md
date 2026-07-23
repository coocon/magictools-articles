---
title: "10 Advanced Claude Code Techniques: Session Recovery, Checkpoint Rewind, and Headless CI"
slug: claude-code-advanced-tips-2026-en
category: claude
locale: en
tags: [Claude Code, tips, headless, checkpoint, context management, cost control]
summary: "From named sessions and the /rewind time machine to context slimming, path-scoped rules, auto memory, and wiring Claude Code into CI with headless mode — 10 advanced techniques straight from the official docs that most users never open, each with the exact commands and when to use them."
status: published
---

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

## 3. The Context-Slimming Trio: /clear, /compact, /context

Long sessions get slower, dumber, and more expensive for one reason: the context fills with things that no longer matter. Three commands, three situations:

| Command | What it does | When |
|---------|--------------|------|
| `/context` | Show a breakdown of current context usage | Diagnose first, when things feel slow or costly |
| `/clear` | Wipe the context and start fresh | Switching to an unrelated task |
| `/compact keep the XX decisions` | Replace history with a summary | Task unfinished but context nearly full |

`/compact` accepts instructions about what to preserve — `/compact keep the database schema decisions` beats a blind compaction every time.

## 4. Path-Scoped Rules

Everyone writes a CLAUDE.md; too many people grow it into a 300-line dumping ground that gets fully loaded into every session, polluting the context. The official answer is the `.claude/rules/` directory with frontmatter declaring where each rule applies:

```markdown
---
paths:
  - "src/api/**/*.ts"
---
# API Development Standards
- Every endpoint must validate its inputs
- Error responses use the standard shape { code, msg, data }
```

The rule loads only when Claude actually touches files under `src/api/`. Keep CLAUDE.md for global facts (tech stack, build commands) and push module-specific conventions down into rules.

**When to use it**: medium-to-large codebases, mixed frontend/backend monorepos, teams with lots of conventions.

## 5. Auto Memory: Let Claude Take Its Own Notes

Claude Code ships with auto memory on by default: it records durable discoveries across sessions — build-command gotchas, debugging conclusions, your code-style preferences — into a project memory directory, and carries them into future sessions automatically.

```bash
/memory        # view the memory index and all memory files
```

Your job is only two things: periodically open `/memory` and delete stale entries (a wrong memory is worse than no memory), and when it records something it shouldn't, just say "forget that."

## 6. Headless Mode: Wire Claude Code into Scripts and CI

The `-p` flag turns Claude Code into a composable command-line tool, which is where it pulls away from being "a chat window":

```bash
# Simplest form: one question, one answer, no interactive UI
claude -p "summarize the module structure of this repo"

# In CI: pre-approve tools so nothing blocks on confirmation
claude -p "fix all lint errors" --allowedTools "Bash,Edit"

# Pipelines: log diagnosis as a shell filter
cat error.log | claude -p "find the root cause, ranked by likelihood"
```

**When to use it**: automated PR review, scheduled code audits, log-analysis pipelines, bulk refactoring scripts. The rule: grant minimal power with `--allowedTools` instead of opening everything up for convenience.

## 7. Structured Output and Cost Accounting

Pair headless mode with `--output-format json` and the output becomes machine-parseable — with the bill included:

```bash
claude -p "list every public API in this repo" --output-format json | jq '.total_cost_usd'
```

Every invocation tells you exactly what it cost. If you wire this into CI, ship that field to your metrics system — unmonitored LLM automation eventually produces a surprising invoice. For real-time processing of the output stream, use `--output-format stream-json`.

## 8. The Four-Phase Workflow: Explore → Plan → Implement → Commit

The rhythm the official best-practices guide keeps coming back to, and the one with the biggest payoff on non-trivial tasks:

1. **Explore** — read-only first: "Read src/auth and explain how session management currently works"
2. **Plan** — switch to plan mode (`Shift+Tab`), get a written implementation plan, review it yourself
3. **Implement** — execute the approved plan, and hand Claude a way to verify: "run the tests and fix failures until green"
4. **Commit** — only after you've confirmed the result, let it write the commit message and open the PR

The anti-pattern is a single "switch login to OAuth" and letting it run — with no exploration or plan, its definition of "done" won't match yours, and the rework rate is brutal. **Giving it a verification method** is the heart of step 3: with tests (or any reproducible check command), it can iterate to passing on its own.

## 9. Rich Context: Screenshots, @files, and Piped Logs

Four ways to feed the context precisely, all better than pasting walls of text:

```bash
# Paste screenshots directly (Cmd+V / Ctrl+V) — for UI bugs, one image beats a paragraph
# @-reference files instead of copying them
@package.json check the dependencies for versions with known vulnerabilities

# Hand it a URL; Claude fetches it
Set up a PostToolUse hook following https://code.claude.com/docs/en/hooks-guide

# Run shell commands inside the session; output lands in context automatically
! npm test
```

That last `!` prefix deserves its own mental bookmark: the command runs in your session and its output flows straight into the conversation — no more "run it, then paste the results" loop.

## 10. Skills: Turn Repeated Procedures into a Single Command

Any procedure you've explained to Claude three times should become a skill:

```markdown
# .claude/skills/fix-issue/SKILL.md
---
name: fix-issue
description: Fix a GitHub issue following the standard procedure
disable-model-invocation: true   # manual trigger only
---

1. Read the issue with gh issue view
2. Locate the relevant code and reproduce the problem first
3. Implement the fix and add a regression test
4. Run the full test suite; summarize the changes once green
```

From then on, `/fix-issue 1234` is the whole procedure. `disable-model-invocation: true` means only you can trigger it; drop that line and Claude will invoke it automatically when it judges the skill relevant. Commit your skills to the repo and every new teammate inherits the entire workflow on day one.

## Closing: A Priority Table

| If you fix just one thing | Do this |
|---------------------------|---------|
| Always losing yesterday's session | Technique 1: naming + resume |
| Afraid Claude will wreck the code | Technique 2: /rewind |
| Sessions get dumber as they get longer | Technique 3: /compact |
| CLAUDE.md is already 300 lines | Technique 4: path-scoped rules |
| Want automation | Techniques 6 + 7: headless + JSON output |
| Big tasks keep needing rework | Technique 8: the four-phase workflow |

## References

- Best practices: https://code.claude.com/docs/en/best-practices
- Session management: https://code.claude.com/docs/en/sessions
- Checkpointing: https://code.claude.com/docs/en/checkpointing
- Headless mode: https://code.claude.com/docs/en/headless
- Memory: https://code.claude.com/docs/en/memory
- Skills: https://code.claude.com/docs/en/skills
