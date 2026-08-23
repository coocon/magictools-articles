# Claude Code Memory and Commands: CLAUDE.md, Slash Commands, and Workflow Shortcuts

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-memory-and-commands-guide-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-memory-and-commands-guide-en?utm_source=github&utm_medium=referral)**

Claude Code gets much more useful when you stop repeating the same instructions in every session. Anthropic's memory system and slash commands are the two features that make that possible. One preserves context across sessions. The other gives you fast controls for common actions while you are working.

The result is a more stable workflow: project rules stay in `CLAUDE.md`, personal preferences stay in your user memory, and session commands let you adjust Claude without rewriting the prompt from scratch.

## How Claude Code memory works

Anthropic documents memory as a hierarchy with several layers:

1. Enterprise policy memory for organization-wide instructions
2. Project memory in `./CLAUDE.md`
3. User memory in `~/.claude/CLAUDE.md`

That structure matters because it separates team-wide conventions from your own preferences. In practice, it lets Claude read the right instructions automatically when you open a project.

Claude Code also looks up memory files recursively from the current working directory, which is useful in larger repositories. You can see loaded memories with the `/memory` command.

...

---

**[👉 Continue reading: Claude Code Memory and Commands: CLAUDE.md, Slash Commands, and Workflow Shortcuts](https://tools.cooconsbit.com/en/articles/claude-code-memory-and-commands-guide-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
