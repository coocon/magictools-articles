# Claude Code Skills: Turn Repeated Workflows into a Slash Command with SKILL.md

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-skills-guide-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-skills-guide-en?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 Continue reading: Claude Code Skills: Turn Repeated Workflows into a Slash Command with SKILL.md](https://tools.cooconsbit.com/en/articles/claude-code-skills-guide-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
