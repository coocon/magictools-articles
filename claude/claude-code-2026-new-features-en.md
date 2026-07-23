---
title: "What's New in Claude Code in 2026: Agent Teams, Nested Subagents, and Background Sessions"
slug: claude-code-2026-new-features-en
category: claude
locale: en
tags: [Claude Code, Agent Teams, subagents, Fast Mode, new features 2026, AI coding]
summary: "The first half of 2026 pushed Claude Code well beyond the single-session workflow: multi-agent teams, subagents that recurse up to 5 levels deep, background sessions with Agent View, and an Opus-class Fast Mode. Based on the official docs and changelog, here is what each feature does, how to turn it on, and when it's actually worth using."
status: published
---

Claude Code has been shipping at a relentless pace through the first half of 2026. Many long-time users are still working in "one terminal, one session" mode, while the official capabilities have quietly evolved into something closer to "one developer directing a team of agents." This post filters the official docs and changelog down to the features that actually matter, with setup instructions and an honest take on when each one earns its cost.

## Table of Contents

1. Agent Teams: multi-agent collaboration
2. Nested subagents: recursive delegation up to 5 levels
3. Background sessions and Agent View
4. Fast Mode: an Opus-class model at higher speed
5. Permissions overhaul: Auto Mode and destructive-command guards
6. Worktree isolation: parallel edits without collisions
7. Small but useful updates
8. A practical decision guide

## 1. Agent Teams: Multi-Agent Collaboration (Experimental)

This is the most ambitious update of the year so far. Traditional subagents follow a one-way relationship: the main session delegates, the subagent reports back. Agent Teams turn that into genuine teamwork — **each teammate gets its own context window, teammates message each other directly, and they claim work from a shared task list** instead of routing everything through the main session.

It's experimental, so you enable it explicitly:

```json
// settings.json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

No new commands to learn — you describe the team you want in plain language:

```text
Spawn three teammates to look at this refactor from different angles:
one on UX, one on architecture, and one as devil's advocate hunting for risks.
```

Typical scenarios from the official docs:

| Scenario | How the team splits the work |
|----------|------------------------------|
| Parallel code review | One reviewer per dimension (correctness / performance / security) |
| Hard-to-reproduce bugs | Each teammate owns a competing hypothesis and tries to falsify the others |
| Cross-stack features | Frontend, backend, and tests each get an owner; interfaces are agreed via messages |
| Large migrations | Work split by module, with the shared task list preventing duplicate effort |

The key mental model: teammates are **peers, not subordinates**. Teams shine on problems that benefit from multiple perspectives. If your task is a pipeline (A, then B, then C), plain subagents are cheaper and simpler.

## 2. Nested Subagents: Recursive Delegation Up to 5 Levels

Subagents can now spawn subagents of their own, up to 5 levels deep. This fixes a long-standing pain: previously, if you sent a subagent to investigate a large module and it discovered three subsystems inside, it had to grind through all of them alone. Now it can delegate downward:

```text
Use a subagent to investigate the reconciliation logic in the payments module.
If the problem spans multiple subsystems, it may spawn its own subagents
to investigate them in parallel.
```

There's a cost guardrail built in: each session allows up to 200 subagent spawns by default, adjustable via `CLAUDE_CODE_MAX_SUBAGENTS_PER_SESSION`. Subagents run in the background by default and notify their parent when done — your main session never blocks.

## 3. Background Sessions and Agent View

To go with the multi-agent push, Claude Code shipped the observability to match:

- **`/fork <name>`** — clone the current session into an independent background session. A typical move: keep your main line focused on the feature, fork a copy to try a risky refactor, and let the two proceed without touching each other.
- **`claude agents`** — list every background session and teammate with its live status.
- **Agent View** — one line per agent with a color-coded status word (working / blocked / done) plus a one-sentence summary; struggling agents get flagged with a diagnosis.

The value here is turning delegated work from a black box into a dashboard. When a background task goes off the rails, you find out immediately — not thirty minutes later when you check on a pile of errors.

## 4. Fast Mode: An Opus-Class Model at Higher Speed

A common misconception is that "fast mode" secretly swaps in a smaller model. The official docs are explicit: **Fast Mode runs an Opus-class model with faster output — the intelligence tier does not change.**

```bash
/fast          # toggle fast mode within a session (CLI)
```

The decision rule is simple: for interactive development — where you're literally sitting there waiting for the answer — Fast Mode is a noticeable quality-of-life upgrade. For unattended batch work (headless runs, CI), latency doesn't matter, so standard mode is fine.

## 5. Permissions Overhaul: Auto Mode and Destructive-Command Guards

The permission system got a meaningful reorganization:

| Mode | Behavior | Best for |
|------|----------|----------|
| Manual (formerly "default") | Confirm each action | Production environments, unfamiliar codebases |
| Auto | A classifier auto-approves low-risk operations | Everyday development; kills confirmation fatigue |
| BypassPermissions | Skip all checks | Isolated environments only (containers / CI sandboxes) |

```bash
claude --permission-mode auto -p "fix all lint errors"
```

There's also a new safety net: even in auto mode, **destructive commands like `git reset --hard` or `terraform destroy` are intercepted and held for confirmation unless you explicitly asked for them**. That single change moves Auto Mode from "for the brave" to a sensible daily default.

## 6. Worktree Isolation: Parallel Edits Without Collisions

Multiple agents editing the same files will eventually collide. The official answer is git worktree isolation: a subagent launched with `isolation: worktree` works inside its own worktree copy and **cannot perform git operations against your main repository**. Worktrees that end up with no changes are cleaned up automatically.

The hardening that shipped alongside it: entering worktrees outside `.claude/worktrees/` requires confirmation; committed symlinks inside a repo are no longer followed (preventing out-of-tree file access); and "always allow" permission rules are shared across worktrees, so you don't re-approve everything in each copy.

## 7. Small but Useful Updates

- **`claude mcp login <server> --no-browser`** — authenticate MCP servers entirely from the CLI. If you run Claude Code on a remote box over SSH, you're no longer stuck at "can't open a browser."
- **Exact-match hook matchers** — patterns like `mcp__brave-search__.*` no longer substring-match, ending a whole class of accidental hook triggers.
- **Stacked skills** — invoke up to 5 skills in a single message (`/skill-a /skill-b do the thing`), chaining "fetch data + chart it + write the report" into one call.
- **`/doctor` (alias `/checkup`) overhaul** — a full installation health check: duplicate installs, PATH issues, settings.json syntax, broken hooks, unused skills/MCP servers, plugins slowing startup — with auto-fix for what it finds.

## 8. A Practical Decision Guide

A pragmatic escalation ladder:

1. **Small single-file change** — plain session. Don't turn anything on.
2. **Medium task needing research + edits** — a subagent (allow nesting if needed); keep the main context clean.
3. **Multi-perspective review / competing hypotheses** — Agent Teams, and let teammates challenge each other.
4. **Several independent changes in parallel** — background sessions + worktree isolation, merged one by one at the end.
5. **Everywhere** — make Auto Mode with destructive-command guards your default permission setup, and put `/doctor` on your monthly maintenance list.

Multi-agent work is not free — every teammate carries its own context window, and token usage scales with the headcount. The principle the official docs keep repeating is worth keeping on a sticky note: **if one session can solve it, don't deploy a team.**

## References

- Official changelog: https://code.claude.com/docs/en/changelog
- Agent Teams: https://code.claude.com/docs/en/agent-teams
- Fast Mode: https://code.claude.com/docs/en/fast-mode
- Permission modes: https://code.claude.com/docs/en/permission-modes
- Worktrees: https://code.claude.com/docs/en/worktrees
