---
title: "Claude Code Memory and Commands: CLAUDE.md, Slash Commands, and Workflow Shortcuts"
slug: "claude-code-memory-and-commands-guide-en"
category: ai-tutorials
tags:
  - claude code
  - anthropic
  - memory
  - slash commands
summary: "How to use Claude Code's memory system and slash commands to keep project instructions, personal preferences, and reusable workflows organized."
coverImage: ""
status: published
scheduledAt: ""
---

Claude Code gets much more useful when you stop repeating the same instructions in every session. Anthropic's memory system and slash commands are the two features that make that possible. One preserves context across sessions. The other gives you fast controls for common actions while you are working.

The result is a more stable workflow: project rules stay in `CLAUDE.md`, personal preferences stay in your user memory, and session commands let you adjust Claude without rewriting the prompt from scratch.

## How Claude Code memory works

Anthropic documents memory as a hierarchy with several layers:

1. Enterprise policy memory for organization-wide instructions
2. Project memory in `./CLAUDE.md`
3. User memory in `~/.claude/CLAUDE.md`

That structure matters because it separates team-wide conventions from your own preferences. In practice, it lets Claude read the right instructions automatically when you open a project.

Claude Code also looks up memory files recursively from the current working directory, which is useful in larger repositories. You can see loaded memories with the `/memory` command.

## What belongs in project memory

Use `CLAUDE.md` for instructions that should travel with the codebase:

- Build and test commands
- Naming conventions
- Architecture notes
- Team-specific coding rules
- Common workflow reminders

The goal is to store the instructions you would otherwise keep repeating at the start of every session. That saves time and reduces inconsistency.

## How to add or edit memory quickly

Anthropic provides two easy paths:

- Start a message with `#` to add a memory quickly
- Use `/memory` to open the relevant memory file in your editor

Example:

```text
# Always prefer small, reviewable changes
```

That is enough to capture a simple preference without interrupting your flow.

## Slash commands that matter most

The built-in slash commands Anthropic highlights are especially practical for day-to-day work:

- `/init` to initialize a project with a `CLAUDE.md` guide
- `/clear` to clear conversation history
- `/compact` to compress the conversation when it gets too long
- `/config` to view or modify configuration
- `/mcp` to manage MCP connections
- `/model` to switch models
- `/permissions` to review or update permissions
- `/help` to see what is available

These commands let you adjust the session without breaking your flow or retyping large instructions.

## Custom commands for repetitive work

Anthropic also supports custom slash commands stored as Markdown files. That is useful when your team repeats the same tasks often.

Project commands live in `.claude/commands/`, and personal commands live in `~/.claude/commands/`. You can use them for things like:

- Code review checklists
- Security review prompts
- Refactor instructions
- Documentation update workflows

If you routinely ask Claude to do the same kind of task, a command is often better than a long prompt because it is easier to reuse and easier to keep consistent.

## A simple memory strategy

Use a three-layer rule:

1. Put team rules in project memory
2. Put personal defaults in user memory
3. Put one-off instructions in the current session

That keeps Claude Code predictable. It also reduces the temptation to stuff everything into a single prompt.

## Official References

- [Manage Claude's memory](https://docs.anthropic.com/en/docs/claude-code/memory)
- [Slash commands](https://docs.anthropic.com/en/docs/claude-code/slash-commands)
- [Quickstart](https://docs.anthropic.com/en/docs/claude-code/quickstart)

Sources reviewed on March 29, 2026. Feature availability, plan limits, and interface details can change, so confirm current behavior in the linked official Anthropic resources.
