---
title: "Claude Code Quickstart Guide: A Practical First Session"
slug: "claude-code-quickstart-guide-en"
category: ai-tutorials
tags:
  - claude code
  - anthropic
  - terminal
  - coding
summary: "A practical first-session guide to Claude Code: install it, log in, ask useful questions, and complete a first code change with a clean workflow."
coverImage: ""
status: published
scheduledAt: ""
---

Claude Code is Anthropic's terminal-based coding assistant. The fastest way to get value from it is not to memorize every command. It is to set up a clean first session, ask for a concrete task, and let Claude inspect the codebase before you tell it what to change.

Anthropic's quickstart and overview docs make the onboarding path very simple: install the CLI, log in with a Claude.ai or Anthropic Console account, enter a project directory, and start a session with `claude`. From there, Claude can help you understand the codebase, propose changes, run tests, and keep working in the same conversation.

## What to prepare before you start

Before opening Claude Code, make sure you have:

1. A terminal open
2. A real code project to work on
3. A Claude.ai or Anthropic Console account
4. Node.js 18 or newer, if you plan to install the npm package

That is enough to start. You do not need a long setup phase or a special config file to try the tool for the first time.

## Install and launch Claude Code

Anthropic documents two installation paths:

```bash
npm install -g @anthropic-ai/claude-code
```

Or, for the newer native installer:

```bash
curl -fsSL claude.ai/install.sh | bash
```

After installation, change into a project folder and launch Claude Code:

```bash
cd /path/to/your/project
claude
```

Claude will prompt you to log in if needed. Once authenticated, your session is ready to use.

## Start with questions about the codebase

Your first prompt should usually be descriptive, not prescriptive. Start by asking Claude to explain what it sees:

- What does this project do?
- Where is the main entry point?
- What technologies does this repository use?
- Explain the folder structure

This is the safest first step because it lets Claude inspect the project before you ask for a change. You get a better answer, and Claude gets the context it needs to work like a teammate instead of a guessing machine.

## Make your first change

Once Claude understands the codebase, give it a concrete fix:

```text
There's a bug where users can submit an empty form. Find the cause and fix it.
```

Anthropic describes the typical workflow clearly: Claude locates the relevant code, understands the surrounding context, implements a solution, and runs tests if they are available. That is the right mental model for first-time use.

## Use follow-up prompts to steer the work

Claude Code works best when you keep the conversation focused:

- Ask for a plan before a large edit
- Ask it to explain a risky change
- Ask it to narrow the scope if the result is too broad
- Ask it to verify the fix with tests or a smaller reproduction

This is the main habit to build in the first week. A good first session is not about getting the perfect answer immediately. It is about shaping the work in small, checkable steps.

## Helpful daily commands

Anthropic's quickstart highlights a few commands that matter early:

```bash
claude
claude "fix the build error"
claude -p "explain this function"
claude -c
claude -r
```

Inside a session, `/help` shows the available commands, and `/clear` resets conversation history when you want to start fresh.

## A practical first-session checklist

Use this sequence the first time you try Claude Code:

1. Install the CLI
2. Log in
3. Open a real project
4. Ask Claude to explain the repository
5. Give one narrow task
6. Review the output before expanding scope

That workflow is simple on purpose. It helps you learn how Claude behaves in your environment before you rely on it for larger edits.

## Official References

- [Quickstart](https://docs.anthropic.com/en/docs/claude-code/quickstart)
- [Claude Code overview](https://docs.anthropic.com/en/docs/claude-code/overview)
- [Slash commands](https://docs.anthropic.com/en/docs/claude-code/slash-commands)

Sources reviewed on March 29, 2026. Feature availability, plan limits, and interface details can change, so confirm current behavior in the linked official Anthropic resources.
