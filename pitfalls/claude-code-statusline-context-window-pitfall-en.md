# Fable 5 Has a 1M Context Window, So Why Does the Status Line Say 200k? Capture the Data Before You Swap the Tool

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-statusline-context-window-pitfall-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-statusline-context-window-pitfall-en?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 Continue reading: Fable 5 Has a 1M Context Window, So Why Does the Status Line Say 200k? Capture the Data Before You Swap the Tool](https://tools.cooconsbit.com/en/articles/claude-code-statusline-context-window-pitfall-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
