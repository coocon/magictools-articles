---
title: "Fable 5 Has a 1M Context Window, So Why Does the Status Line Say 200k? Capture the Data Before You Swap the Tool"
slug: claude-code-statusline-context-window-pitfall-en
category: pitfalls
locale: en
tags: [Claude Code, statusline, context window, bug postmortem, LLM]
summary: "Claude Fable 5 officially ships with a 1M-token context window, yet the Claude Code status line kept showing 200k as the denominator. The first instinct — 'let's switch to a better statusline' — was wrong. This postmortem walks through the full debugging process: one line of tee to capture the statusline's stdin, hard evidence that the official field misreports 200000 for new models, and a model-table fix. Plus a general lesson: swapping tools never fixes a broken data source."
status: published
source: authored
translationSlug: claude-code-statusline-context-window-pitfall
---

## Symptom

Working in Claude Code with Claude Fable 5, whose docs clearly state a **1M-token context window** (on by default, no beta header needed). But the custom status line at the bottom showed:

```
magictools | Fable 5 | Ctx 38% (76k/200k)
```

A 200k denominator. By that math, the bar would turn red at 160k tokens — while the real window still had 840k of headroom.

The first instinct was natural: **maybe my statusline script is just bad, should I switch to a popular community one?**

That instinct is the first pitfall this postmortem covers.

## Think First: Would Switching Statuslines Even Help?

Claude Code's statusline mechanism works like this: on every refresh, it pipes a JSON payload via stdin to whatever command you configured, carrying model info, working directory, context usage, and so on. Every statusline — your own bash script or a community project — receives **the same stdin**.

There are only two places a window size can come from:

1. The `context_window.context_window_size` field in stdin
2. A built-in "model → window size" lookup table keyed by model name

If the field itself is wrong, any statusline that reads it is the same soup in a different bowl. And if you are counting on a built-in table, a freshly released model like Fable 5 almost certainly is not in third-party tables yet.

**Conclusion: verify the data source before talking about switching tools.** Otherwise you swap everything out and still see 200k.

## Capture the Scene: One Line of tee for Hard Evidence

The statusline's stdin is ephemeral — you never see it normally. The most direct debugging move is a temporary line that dumps every input to disk:

```bash
input=$(cat)
printf "%s" "$input" > /tmp/statusline-debug.json   # temporary debug line, delete after use
```

Wait for one status line refresh (any new message in the session triggers it), and the file contains the genuine article:

```bash
jq '.model, .context_window' /tmp/statusline-debug.json
```

```json
{
  "id": "claude-fable-5",
  "display_name": "Fable 5"
}
{
  "total_input_tokens": 76334,
  "context_window_size": 200000,
  "used_percentage": 38
}
```

Hard evidence: **the model id is clearly `claude-fable-5`, yet Claude Code reports `context_window_size` as 200000.** The script was not miscalculating — the upstream data source was wrong. When Claude Code does not know a new model's window size, it falls back to the 200k default (the known issue #76751, where 1M sessions get misreported as 200k).

At this point, the "switch statuslines" option is officially dead: whoever reads that field gets 200000.

## Root Cause

Three layers stacked up:

1. **The official field misreports**: Claude Code reports `context_window_size` as 200000 for 1M-window models (verified with `claude-fable-5`)
2. **The script's hardcoded fallback**: when the field is missing, the script set `ctx_size=200000`, further cementing the value
3. **The existing correction was too strict**: the script only corrected to 1M once "usage exceeded the reported size" — meaning you had to burn through 200k tokens before the denominator turned right. Until then, every percentage was computed against 200k and every red alert was a false alarm

## The Fix: A Model Table Plus the Old Fallback

Since the upstream field cannot be trusted, maintain a **table of known 1M-window models** in the script and force-correct by `model.id`:

```bash
model_id=$(printf '%s' "$input" | jq -r '.model.id // empty')

# The official field misreports 200k for 1M-window models (#76751,
# verified: claude-fable-5 reports 200000).
# Force-correct known 1M-window models via this table; a [1m] suffix
# explicitly declares 1M and takes priority over the table.
case "$model_id" in
  *"[1m]"*)
    ctx_size=1000000
    ;;
  *fable-5*|*mythos-5*|*opus-5*|*sonnet-5*|*opus-4-8*|*opus-4-7*|*opus-4-6*|*sonnet-4-6*)
    if [ -z "$ctx_size" ] || [ "$ctx_size" -le 200000 ] 2>/dev/null; then
      ctx_size=1000000
    fi
    ;;
esac
[ -z "$ctx_size" ] && ctx_size=200000
# Fallback: for unknown models, correct to 1M when usage exceeds the reported window
if [ -n "$ctx_used" ] && [ "$ctx_used" -gt "$ctx_size" ] 2>/dev/null; then
  ctx_size=1000000
fi
```

Every model in the table (Fable/Mythos 5, Opus 5/4.8/4.7/4.6, Sonnet 5/4.6) has a documentation-confirmed 1M window. The old "correct when usage exceeds the reported value" logic stays as the fallback for models outside the table.

Verification needs no live session — the debug JSON captured earlier is a ready-made test case:

```bash
bash ~/.claude/statusline-command.sh < /tmp/statusline-debug.json
# magictools | Fable 5 | Ctx 7% (79k/1M)
```

Same input: 38% becomes 7%, and the denominator goes from 200k to 1M. Remember to delete the debug line and the dump file in `/tmp` afterwards.

## An Easily Missed Tail: 1M ≠ 1M Usable

Claude Code reserves a buffer for auto-compact, so the practical budget of a 1M window is roughly **830k**. With the status line computing percentages against 1M, keep one notch of margin in your head: when it turns red at 80%, start wrapping up — do not wait for 100%.

## Lessons to Take Away

1. **Before swapping tools, confirm whether the data source is what is broken.** When every downstream consumes the same upstream data, replacing the downstream is a no-op. Had I switched to a community statusline right away, it would still show 200k — plus one extra dependency.
2. **Persist ephemeral data before debugging it.** For invisible scenes like stdin, pipes, and hook inputs, one line of tee/redirect to disk beats staring at outputs and guessing. The captured scene doubles as a regression-test input.
3. **Every hardcoded fallback needs an expiry plan.** `ctx_size=200000` was correct the day it was written and became a landmine after models iterated. Next to a fallback value, keep an identifier-keyed correction table, plus dynamic correction logic for anything outside the table.
4. **When a new model ships, treat your toolchain's knowledge of it as unverified.** Model capabilities upgrade; the hardcoded assumptions about them in editors, CLIs, and monitoring scripts (window size, pricing, tokenizer) do not update themselves.
