---
title: "Claude Code Plan Mode: Research First, Code Later, Rework Less"
slug: claude-code-plan-mode-guide-en
category: claude
locale: en
translationSlug: claude-code-plan-mode-guide
tags: [Claude Code, plan mode, workflow, refactoring, AI coding, code review]
summary: "Plan Mode is Claude Code's read-only planning mode: Claude researches your codebase and presents an implementation plan, and only touches code after you approve. This guide covers how to enter it (Shift+Tab cycling), what it actually blocks, which tasks deserve it, and four techniques that make planning genuinely improve output quality."
status: published
---

The most expensive thing about Claude Code is not tokens — it is rework. Claude confidently edits eight files, you read the diff, and the whole approach was wrong. Roll back, start over. Plan Mode is the switch built for exactly this: Claude researches read-only and proposes a plan, and only after you approve does it touch code.

## What Plan Mode is: read-only research plus an approval gate

In Plan Mode, Claude's behavior splits into two phases:

1. **Research (read-only)**: it can read files, search code, and analyze structure — but it cannot edit files or run commands that change system state
2. **Approval**: it distills the research into an implementation plan and presents it; approve and it exits Plan Mode to execute, or push back and it revises the plan

The key value is separating "think it through" from "do the work" at the mechanism level. Without Plan Mode you can always write "don't change code yet" in your prompt, but that is a verbal agreement. Plan Mode is a hard constraint: during research, file edits are blocked even if attempted.

## How to enter: cycle with Shift+Tab

Press **Shift+Tab** during a session to cycle through three permission modes:

- **Normal**: every sensitive action asks for confirmation
- **Auto-accept**: file edits go through automatically — good for mechanical tasks you trust
- **Plan Mode**: read-only planning, all modifications blocked

You can also start a session directly in it: `claude --permission-mode plan`. If your team has a "big changes need a plan first" convention, write it into the project's `CLAUDE.md` — Claude will then proactively start non-trivial tasks in a planning flow.

## Which tasks deserve Plan Mode

A practical threshold: **if the change takes more than 3 steps, or more than one reasonable approach exists, plan first.** Concretely:

**Turn it on for:**

- Multi-file refactors ("extract the auth logic into middleware")
- First changes in an unfamiliar codebase — the research phase doubles as your codebase tour
- Features with architectural forks ("add real-time notifications" — WebSocket or polling?)
- Bugs with an unknown root cause: let it investigate read-only and present a diagnosis, instead of guess-and-edit contaminating the scene

**Skip it for:**

- Typo fixes and one-line style tweaks
- Tasks where you have already specified the exact change and only execution remains
- Pure lookup questions ("where is this function called?") — just ask; planning adds a detour

The cost of using Plan Mode on a small task is one extra round trip. The cost of skipping it on a large task is wholesale rework. When unsure, err toward planning.

## Four techniques that make planning actually improve quality

**1. Make it ask questions before proposing.** Add "if the requirements are ambiguous, ask me first" to your planning prompt. Every question surfaced during planning is one rework avoided during execution.

**2. Require verification steps in the plan.** A good plan lists not just "which files change" but "how we prove it worked" — which test to run, which log to check. Acceptance criteria written into the plan give you something to audit against after execution.

**3. Persist the plan to a file.** Have Claude write the approved plan into something like `tasks/todo.md` and check items off as it executes. If the session is interrupted or the context gets compacted, that file is the anchor for resuming exactly where things stood.

**4. When execution drifts, go back to planning.** If a premise turns out wrong mid-execution, do not let Claude "adjust as it goes" — switch back to Plan Mode and re-evaluate. Pushing through after a wrong turn almost always costs more than stopping to re-plan.

## How it composes with the rest of Claude Code

Plan Mode is not an island. During research it pairs naturally with **subagents** — send them to investigate different subsystems in parallel while the main context receives only conclusions. During execution, the approved plan pairs with **auto-accept** so mechanical edits stop asking for confirmation one by one. And the team rule about *when* planning is mandatory belongs in **CLAUDE.md**, where every future session inherits it automatically.

In one sentence: Plan Mode turns AI coding from "give an order, gamble on the result" into "review the approach, then release the brakes." The bigger the task, the more rework that one approval round saves.

Related reading on this site: the "Claude Code Quickstart Guide" if you are still building your basic workflow, and "10 Advanced Claude Code Techniques" for session recovery, checkpoint rewind, and other capabilities that complement Plan Mode.
