# Claude Code's Auto Mode Judges a Model With a Model — When the Model Is Down, You Can't Even Run cat

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-auto-mode-permission-trap-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-auto-mode-permission-trap-en?utm_source=github&utm_medium=referral)**

Mid-session, an entirely unremarkable command got blocked:

```
claude-opus-5[1m] is temporarily unavailable, so auto mode cannot
determine the safety of Bash right now.
Note: reading files, searching code, and other read-only operations
do not require the classifier and can still be used.
```

The counterintuitive part is that last sentence. Reading files and searching code still work; only shell commands are dead. If the model were simply down, you would not get this selective paralysis — reading a file also requires the model to initiate the call. That asymmetry is the single most useful diagnostic clue in the message.

What follows is the debugging session that came next. I formed three entirely reasonable hypotheses and knocked all three down. The value here isn't the final fix command — it's the knocking down.

## The error reads like a model outage; it's a broken judgment chain

There are two independent model call chains here:

1. **The main loop** — the model reads your request and decides which tool to invoke. This one stayed healthy, which is why Read and Grep kept working.
2. **The safety judgment** — before auto mode actually executes a Bash command, it makes an **extra request to the model** asking "is this command safe?" This is the one that broke.

With that chain down, the permission system has no verdict to act on. It cannot default to "allow," so it blocks everything. Read-only tools never enter that step at all, so they come through untouched.

![Two model call chains side by side: a Read/Grep request goes request → main loop → tool execution and succeeds, while a Bash request goes request → main loop → safety classifier, where the second model call is unavailable, leaving no verdict and blocking the command](https://cdn.tools.cooconsbit.com/uploads/hermes/2026-08-09/1786292779000-b21019d6.png)

*A diagram, not a screenshot — this failure only appears while the classifier is genuinely unreachable, so it can't be staged on demand. The error text quoted at the bottom is verbatim from the session on 2026-07-28 (v2.1.220).*

**The classifier runs on the exact model you're working with.** There's direct evidence for this: the model ID named in the error changes when you switch session models. After I moved my config from `opus[1m]` to `opus`, the error text shifted from `claude-opus-5[1m]` to `claude-opus-5` in lockstep. The classifier isn't some separate small model — it's the one in your session.

## What each permission mode actually trusts

...

---

**[👉 Continue reading: Claude Code's Auto Mode Judges a Model With a Model — When the Model Is Down, You Can't Even Run cat](https://tools.cooconsbit.com/en/articles/claude-code-auto-mode-permission-trap-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
