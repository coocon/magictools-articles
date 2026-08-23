# Half My openclaw Commands Ran, Half Didn't — It Looked Like a Permission Classifier, It Was launchd

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/openclaw-gateway-not-classifier-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/openclaw-gateway-not-classifier-en?utm_source=github&utm_medium=referral)**

Two Claude Code windows, one Mac, one user, one global `settings.json`. Window A sat in `~/4khz/tingease`, window B in `~/4khz/tingease/AudioCore`. Same `openclaw` command. B ran it fine. A kept failing.

The user's read was the natural one: "Your classifier works — the other window can't execute."

That suspicion is well earned. Claude Code's auto permission mode really does call a model to judge each Bash command before letting it through, and when that judgment chain breaks, everything gets blocked. We've written up that exact trap before (linked at the end). But this wasn't it. The cause sat in a completely different layer — and what sold the permission theory was the *shape* of the failure.

## The counterintuitive part: not dead, half dead

Both of these ran in the same window seconds apart. The first one worked:

```
$ openclaw --help
OpenClaw 2026.7.1-2 (0790d9f) — All your chats, one OpenClaw.
Usage: openclaw [options] [command]
...
```

The second didn't:

```
$ openclaw health
[openclaw] Could not start the CLI.
[openclaw] Reason: gateway closed (1006 abnormal closure): no close reason
Gateway target: ws://127.0.0.1:18789
```

**Same binary, some subcommands fine, others dead.** That shape misleads badly. A network outage or a crashed binary takes everything down; the fact that anything printed at all proves the binary, the PATH, and the exec bit are all healthy. Of the remaining explanations, "something is filtering commands one by one" sounds the most sensible.

Slice it differently and it resolves. openclaw's subcommands fall into two natural groups:

| Type | Examples | Touches the gateway? |
|------|----------|---------------------|
| Purely local | `--help`, `daemon status`, `plugins list` | No — reads files and config directly |
| Gateway-bound | `health`, `gateway status`, `channels`, `agent` | Yes — dials `ws://127.0.0.1:18789` |

**Everything that worked was in group one. Everything that failed was in group two.** That's not filtering, that's one shared dependency going down and taking its dependents with it. The dividing line has nothing to do with how dangerous a command is — it's whether the command opens that WebSocket.

That clue should have ended it immediately. It didn't, because I didn't look there first. What follows is the detour I actually took.

## Hypothesis 1: the two windows have different permissions

The most direct explanation: window A's project directory carries extra restrictions. Two places to check.

The global `~/.claude/settings.json` had this in `permissions.allow`:

...

---

**[👉 Continue reading: Half My openclaw Commands Ran, Half Didn't — It Looked Like a Permission Classifier, It Was launchd](https://tools.cooconsbit.com/en/articles/openclaw-gateway-not-classifier-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
