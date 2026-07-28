---
title: "Claude Code Skills: Turn Repeated Workflows into a Slash Command with SKILL.md"
slug: claude-code-skills-guide-en
category: claude
locale: en
translationSlug: claude-code-skills-guide
tags: [Claude Code, skills, SKILL.md, slash commands, workflow automation, context management]
summary: "Skills are the most underrated feature in Claude Code: write a workflow into a SKILL.md file and invoke it with a slash command, or let Claude load it automatically when the task matches. This guide covers the file structure, frontmatter, auto-trigger mechanics, how skills differ from CLAUDE.md, hooks, and subagents, plus three ready-to-copy examples."
status: published
---

If you find yourself giving Claude Code the same long instruction every week — "pull the data first, then format the report like this, and watch out for these three gotchas" — what you need is not a better prompt. It's a Skill.

A Skill is a reusable instruction package for Claude Code: write a workflow into a `SKILL.md` file in a conventional directory, and you can invoke it manually with `/skill-name` or let Claude load it automatically when it recognizes a matching task. Skills solve the "expert knowledge never gets captured" problem: the workflow you spent hours refining now travels with the project, available to every teammate and every fresh session.

## The file structure: one folder, one SKILL.md

A minimal skill looks like this:

```
.claude/skills/
└── deploy-check/
    └── SKILL.md
```

`SKILL.md` has two parts — frontmatter and body:

```markdown
---
name: deploy-check
description: Pre-deploy checklist. Use when the user says "ready to deploy",
  "pre-release check", or asks to ship.
---

Run these checks in order. Stop and report if any step fails:

1. Run `npm run typecheck` and confirm there are no type errors
2. Run `npm run build` and confirm the build passes
3. Check git status and list uncommitted files
4. Confirm the version number in CHANGELOG has been bumped
```

There are two locations with different scopes:

- **Project level**: `.claude/skills/<name>/SKILL.md` — lives in the repo, shared with the team
- **Personal level**: `~/.claude/skills/<name>/SKILL.md` — works across all your projects, visible only to you

## The description field is everything: it controls auto-loading

Skills trigger in two ways. Manual is obvious: type `/deploy-check` in a session. The clever part is automatic triggering — on every turn, Claude compares the task at hand against every skill's `description` and loads the matching SKILL.md on its own.

So do not write the description as a feature summary. Write it as a **list of trigger conditions**:

```yaml
# ❌ Weak description: Claude has no idea when to use it
description: GA4 analytics skill

# ✅ Strong description: spell out what the user might actually say
description: Use when analyzing the site's GA4 traffic. Triggers include
  "check pageviews", "which page is hottest", "traffic sources",
  "did the redesign work", pageviews, traffic.
```

This is the opposite of writing docs for humans. Human docs explain *what it is*; a description written for Claude must enumerate *when to use it*.

## Progressive loading: skills are a context-management tool

The key difference between Skills and `CLAUDE.md` is load timing. CLAUDE.md goes into context in full at the start of every session, so a long one burns tokens continuously. A skill keeps only its name and description resident — the body loads only when triggered.

That suggests a practical architecture rule:

- **CLAUDE.md holds rules that always apply**: code style, build commands, hard constraints
- **Skills hold scenario-specific procedures**: release checks, analytics runs, report generation, the operating manual for one subsystem

A skill folder can also carry companion files (reference docs, scripts) that SKILL.md points to. Claude reads them only when needed, so even documentation-heavy workflows will not blow up your context window.

## Skills vs hooks vs subagents: who does what

These three mechanisms get mixed up constantly. Sorting them by determinism makes it clear:

| Mechanism | What it is | When to use |
|-----------|-----------|-------------|
| Hooks | Deterministic shell commands that always run at an event | Lint before commit, auto-format after file writes |
| Skills | On-demand instructions that Claude follows | Multi-step procedures, domain playbooks |
| Subagents | Separate-context workers that report back | Broad searches, parallel exploration, heavy isolated work |

The rule of thumb: **if it must happen every single time, use a hook; if it needs Claude's judgment and adaptation, use a skill; if it would pollute the main context, hand it to a subagent**. They compose, too — a skill's body can legitimately say "delegate this step to a subagent."

## Three skills you can copy today

**1. Deploy check (deploy-check)**: the example above. Its value is moving "what to check before shipping" out of one person's head and into the repo.

**2. Weekly report (weekly-report)**:

```markdown
---
name: weekly-report
description: Use when generating the weekly report. Triggers include
  "weekly report", "this week's numbers".
---

1. Run `npm run ga:report -- top-pages --days 7`
2. Compare against last week and flag pages that moved more than 20%
3. Lead with the conclusion: one-sentence summary first, data table after
4. Analytics lag 24-48 hours — always note the data cutoff date at the end
```

**3. Debug playbook (debug-build)**: capture your project's specific traps — "if the build fails, check the Node version is 20+ first", "on Prisma errors, run db:generate before anything else." New teammates (and new sessions) stop re-learning the same lessons.

## How to start

Begin with whatever instruction you have repeated more than twice in the past week: paste it verbatim into a SKILL.md, write the trigger conditions, and invoke it with `/skill-name` next time. Polish the steps after the first successful run. Skills compound — every one you capture pays out in every future session.

For more advanced Claude Code capabilities, continue with our "10 Advanced Claude Code Techniques" and "Claude Code Memory and Commands" guides on this site.
