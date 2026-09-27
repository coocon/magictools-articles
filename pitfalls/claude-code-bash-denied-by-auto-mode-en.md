# Claude Code \"bash denied by auto mode\": Why It Blocks, \"could not evaluate\" and \"unavailable for this model\" Tested

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-bash-denied-by-auto-mode-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-bash-denied-by-auto-mode-en?utm_source=github&utm_medium=referral)**

## The problem

You run Claude Code in auto mode, and a Bash command gets blocked with a line like this:

```
bash denied by auto mode · [Code from External] · /permissions
```

If you drive it from a script with `claude -p`, you never see that status line. The model receives a longer tool result that starts with:

```
Permission for this action was denied by the Claude Code auto mode classifier. Reason: [Code from External].
```

The same permission mode has two lookalike messages that people search for interchangeably:

- `Auto mode could not evaluate this action and is blocking it for safety — run with --debug for details`
- `auto mode unavailable for this model`

This site already covers [temporarily unavailable, so auto mode cannot determine the safety of bash](/en/articles/claude-code-auto-mode-temporarily-unavailable-fix-en), which is what you get when the classifier can't be reached. The three messages here have different causes and different fixes. Every error string and number below comes from real sessions on Claude Code 2.1.280 on 2026-09-27, copied verbatim.

## Analysis

I started by reading the 2.1.280 binary (`claude.exe`, 217,254,576 bytes). Here is where each message comes from:

- **denied**: the UI line is built as `` `${tool name, lowercased} denied by auto mode` ``, followed by `· <reason>` (truncated past 80 characters) and `· /permissions`. The text sent to the model starts with the constant `"Permission for this action was denied by the Claude Code auto mode classifier. Reason: "`.
- **could not evaluate**: the constant is `"Auto mode could not evaluate this action and is blocking it for safety"`. When the underlying failure is a `refusal`, the builder also inserts "a safety check separate from auto mode blocked this request…".
- **unavailable for this model**: a `switch` maps four unavailability reasons to messages: `settings` (disabled in settings), `circuit-breaker` (`auto mode is unavailable for your plan`), `fast-mode` (`auto mode unavailable while fast mode is on · run /fast off`) and `model`. The model-support check includes a clause that says any model listed before `claude-opus-4-6` in the built-in list is unsupported (`function or(e,n){…return r!==-1&&r<Ig.indexOf(n)}`).

Reading the code also turned up something more important: **in 2.1.280 the classifier no longer runs locally by default**. The binary has a whole set of handlers for `server_no_result`, `server_unsupported` and `server_call_unavailable_*`, plus one message written specifically for proxy users:

...

---

**[👉 Continue reading: Claude Code \"bash denied by auto mode\": Why It Blocks, \"could not evaluate\" and \"unavailable for this model\" Tested](https://tools.cooconsbit.com/en/articles/claude-code-bash-denied-by-auto-mode-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
