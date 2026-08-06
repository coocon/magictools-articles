---
title: "I Put My Opus 5 Config on the Recommended Diet, Then Ran 18 A/B Trials: 21% Cheaper, and I Can't Prove It's Better"
slug: opus-5-prompt-diet-ab-test-en
category: claude
locale: en
translationSlug: opus-5-prompt-diet-ab-test
tags: [Claude Code, Claude Opus 5, prompt engineering, CLAUDE.md, A/B testing, developer workflow]
summary: "\"Opus 5 verifies itself, so delete your fallback prompts\" is advice you hear everywhere. I took it, then isolated both configs with CLAUDE_CONFIG_DIR and ran 18 headless trials on identical tasks. Output tokens dropped 21% and wall clock 20-29%, with zero counterexamples. But two of the rewritten rules never fired at all, and one result points the other way: the old config's most thorough run covered a strict superset of what the new one found. Why cheaper and better are separate questions."
status: published
---

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

A and B run in a sandbox copied fresh from a git baseline each time. C runs against the real repo with only `Read/Glob/Grep/Task` allowed and no `Edit/Write`, so there is nothing to mutate. The sandbox deliberately lives outside the project directory — otherwise it inherits the project-level `CLAUDE.md`, which is one more variable I do not control.

Three trials per cell: 2 configs × 3 tasks × 3 trials = 18 runs.

## The numbers

```
Task  Metric          old mean  new mean    delta   old x3 / new x3
A     out_tokens           945       656      -31%   [826, 1037, 973] / [669, 509, 789]
A     reply_chars          430       166      -61%   [399, 437, 454] / [221, 74, 203]
A     secs                  18        14      -25%   [15, 21, 19] / [12, 11, 18]
A     subagents              0         0       n/a   [0, 0, 0] / [0, 0, 0]

B     out_tokens          4314      3026      -30%   [4681, 4456, 3805] / [3684, 2570, 2824]
B     tool_calls             7         5      -25%   [9, 6, 5] / [6, 5, 4]
B     reply_chars         2372      1818      -23%   [2200, 2579, 2338] / [2141, 1628, 1686]
B     secs                  77        54      -29%   [92, 73, 66] / [64, 46, 53]

C     out_tokens          9410      7903      -16%   [8051, 10600, 9580] / [8060, 7925, 7725]
C     tool_calls            22        19      -15%   [16, 25, 25] / [21, 18, 17]
C     secs                 149       120      -20%   [123, 183, 142] / [131, 120, 109]

Output tokens across 18 runs: old=44009  new=34755  (-21.0%)
```

On cost there is **not a single counterexample**. Three tasks, nine paired runs, output tokens and wall clock down in every direction. Task A's reply length dropped 61% — the one new rule telling it to stay concise does visible work.

Now the part that reads less well.

## Finding one: two of the rules never fired

Look at the `subagents` column. Zero across all 18 runs.

The old config says, in as many words, to use subagents liberally and to throw more compute at hard problems through them. The `Task` tool was in the `--allowedTools` allowlist. Task C is cross-directory exploration in a 1000-file repository, which is the single most delegation-friendly scenario in the set. It delegated exactly zero times.

So the subagent rule I spent effort rewriting **was never exercised by this experiment**. It might help. It might not. I do not know.

Scope creep is the same story. On task A both configs touched only `http.ts`. Neither fixed the `any` types, added backoff, or wrote tests along the way. The new config's diff was actually *larger* (+10/-6 versus +4/-2), but reading it shows that is a structural choice, not creep:

```diff
# old: hoist controller and timer out of the try, add finally to the existing try/catch
+    const ctrl = new AbortController();
+    const timer = setTimeout(() => ctrl.abort(), TIMEOUT);
     try {
...
+    } finally {
+      clearTimeout(timer);
     }

# new: nest a second try/finally in place
+      const timer = setTimeout(() => ctrl.abort(), TIMEOUT);
+      try {
+        ...
+      } finally {
+        clearTimeout(timer);
+      }
```

Both fixes are correct, and both cover all three exit paths (normal return, the 5xx `continue`, the exception). "Diff size," the proxy metric I was rather pleased with, turns out not to point at scope creep at all.

The honest summary: **across 18 runs I measured a difference on exactly one axis, cost.**

## Finding two: a counterexample

For task C I meant to compare coverage — regex out the source paths each answer cites and see who finds more. The first two trials looked great: the new config's union was 11 files, a strict superset of the old config's 4. Cheaper *and* more thorough, exactly the conclusion I wanted.

Then I chased an outlier. Run `old-C-3` returned a 214-character reply, far below everything else. The reason is that it did not answer in the terminal at all — it wrote the inventory into a plan file:

> Inventory complete, written to `plans/hreflang-canonical-floofy-sparkle.md` (file path plus line numbers for each). No code was modified.

Count that file, and the conclusion inverts:

```
old union: 20 files | new union: 11 files
only old found: scripts/backfill-article-locale.ts, scripts/sync-articles.ts,
                src/app/[locale]/layout.tsx, src/app/robots.ts,
                src/app/api/automation/articles/route.ts,
                src/components/dashboard/article-editor.tsx ... (9 total)
only new found: (none)
```

The old config's most thorough single run covers a **strict superset** of everything all three new-config runs found.

Something similar showed up once on task B. The old config's first trial spent 9 tool calls to the new config's 6, because it actually started Node and measured:

> Measured (Node 25.9): `request returned after 520 ms` / `process actually exited after 10004 ms`

That is precisely the "verify it again" behavior I deleted. It was slower and more expensive — and it produced a measured number instead of a conclusion inferred from reading code.

## So "better" needs splitting

The one claim my data supports: **on identical tasks with an identical model, the trimmed config spends 21% fewer output tokens and 20-29% less wall clock, very consistently.**

It does **not** support any of these:

- The new config produces higher quality. The coverage comparison collapsed, and the one counterexample points the other way.
- The subagent rewrite was correct. Zero delegations in 18 runs; the rule never fired.
- The scope contract did anything. Neither side crept, so there is nothing to tell apart.

n=3, one model, three tasks I designed myself. That sample is enough to see a stable 21% gap. It is nowhere near enough to judge quality, where the variance is far larger and I have no neutral grader.

Delete or keep, then? My reading: those fallback prompts were never making the model smarter. They were **forcing an extra step in the cases where it would otherwise coast**. Opus 5 does not need that shove most of the time, which is why deleting it pays reliably, every single run. But occasionally the shove really does kick something loose — one measured benchmark, one 20-file inventory. What you delete is a ceiling that is rare but real.

The trade is personal. For day-to-day code changes I will take a reliable 21% over a rare ceiling. For a security audit or a production incident, I am pasting that verification section back in temporarily — and now I understand what the original advice meant by "add it to the one step that needs it, not as a global rule."

## If you want to run this yourself

Three things matter; the rest is legwork.

1. **Isolate with `CLAUDE_CONFIG_DIR`**, and copy `settings.json` in alongside the `CLAUDE.md` or you get no auth. Strip `hooks` and `statusLine` out of the copy — the paths do not resolve in a sandbox and they will pollute your real hook log.
2. **Put the sandbox outside your project directory**, or it inherits the project-level `CLAUDE.md`, which is a variable you are not controlling.
3. **Pull metrics out of `--output-format stream-json`**: `select(.type=="assistant")|.message.content[]?|select(.type=="tool_use")|.name` for tool calls, `select(.name=="Task")` for subagents, and the trailing `result` event for `usage.output_tokens`.

And one that is not technical: **check that your proxy metric measures the thing you think it measures.** I used diff size to stand in for scope creep and got a read on code-structure preference instead. I used reply length to stand in for coverage and missed the run that wrote its answer to a file. Both times I only caught it by reading raw output. When the run finishes, do not go straight to the summary table — read a few transcripts first.
