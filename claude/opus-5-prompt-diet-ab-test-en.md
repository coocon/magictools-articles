# I Put My Opus 5 Config on the Recommended Diet, Then Ran 18 A/B Trials: 21% Cheaper, and I Can't Prove It's Better

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/opus-5-prompt-diet-ab-test-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/opus-5-prompt-diet-ab-test-en?utm_source=github&utm_medium=referral)**

There is a piece of advice going around: Claude Opus 5 already verifies and reasons on its own, so all those fallback rules sitting in your `CLAUDE.md` — "double-check before answering," "use a subagent to review the result" — are now actively hurting you. Delete them and you save tokens without losing quality. Anthropic's migration guidance says roughly this. Boris Cherny, who built Claude Code, puts it more bluntly: every six months, delete all your prompts and see what the model does without the shackles.

So I did it. My global `~/.claude/CLAUDE.md` had exactly the kind of relics the advice targets:

```markdown
### 2. Subagent Strategy
- Use subagents liberally to keep main context window clean
- For complex problems, throw more compute at it via subagents

### 4. Verification Before Done
- Never mark a task complete without proving it works
- Ask yourself: "Would a staff engineer approve this?"
```

I cut the verification section, inverted the subagent policy, and added a scope contract plus explicit length control. 113 lines became 87.

Which raised the actual question: **how would I know the result is better, rather than me having deleted something useful and then talking myself into it because the new file reads cleaner?**

So I built a harness. The short version: **the savings are real and remarkably consistent. "Better" I cannot demonstrate, and I found one counterexample.**

## The harness

The load-bearing piece is config isolation. Claude Code honors a `CLAUDE_CONFIG_DIR` environment variable, so the two `CLAUDE.md` versions each get their own directory, sharing an identical `settings.json` — same `opus` model, same thinking budget:

```bash
CLAUDE_CONFIG_DIR=./cfg-old claude -p "$PROMPT" \
  --output-format stream-json --verbose \
  --permission-mode acceptEdits \
  --allowedTools "Read,Edit,Write,Glob,Grep,Task,TodoWrite"
```

First, confirm the isolation actually bites. Ask each side to quote its own subagent rule back:

```
=== cfg-old ===
- Use subagents liberally to keep main context window clean
=== cfg-new ===
- Do it yourself by default. Delegate only when the work genuinely needs parallel isolation
```

Good. `--output-format stream-json` emits a structured event per tool call, which is where the tool counts and the `Task` (subagent) counts come from; the closing `result` event carries `usage.output_tokens`.

Three tasks, each aimed at one rule I had changed:

| Task | Targets | Content |
|---|---|---|
| A | Scope contract | Fix a real bug in a sandbox repo: the timeout `AbortController` never clears its timer. Deliberately surrounded by bait — `any` types, duplicated code, a retry loop with no backoff |
| B | Length control | Open-ended audit: "find anything under `src/` that could break the retry logic" |
| C | Delegation | Read-only exploration of a real repo (this site's source, 1000+ files): "find every code path that decides article hreflang / canonical" |

...

---

**[👉 Continue reading: I Put My Opus 5 Config on the Recommended Diet, Then Ran 18 A/B Trials: 21% Cheaper, and I Can't Prove It's Better](https://tools.cooconsbit.com/en/articles/opus-5-prompt-diet-ab-test-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
