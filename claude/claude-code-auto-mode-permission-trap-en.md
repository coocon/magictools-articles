---
title: "Claude Code's Auto Mode Judges a Model With a Model — When the Model Is Down, You Can't Even Run cat"
slug: claude-code-auto-mode-permission-trap-en
category: claude
locale: en
translationSlug: claude-code-auto-mode-permission-trap
tags: [Claude Code, permissions, auto mode, API relay, troubleshooting, developer workflow, claude-code-lab]
lab:
  testedAt: "2026-07-28"
  ccVersion: "2.1.220"
  model: "auto mode (classifier: claude-opus-5[1m])"
  platform: macOS
  status: reproducible
docsUrl: https://docs.claude.com/en/docs/claude-code/settings
summary: "Auto permission mode calls your session model to judge whether each Bash command is safe — the judge and the worker are the same model. When it goes unavailable you land in a counterintuitive half-paralysis: reading files works, but you can't run a single cat. This is a log of a real debugging session in which I proposed three entirely reasonable hypotheses and knocked all three down with controlled experiments, leaving exactly one dependable way out."
status: published
---

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

**The classifier runs on the exact model you're working with.** There's direct evidence for this: the model ID named in the error changes when you switch session models. After I moved my config from `opus[1m]` to `opus`, the error text shifted from `claude-opus-5[1m]` to `claude-opus-5` in lockstep. The classifier isn't some separate small model — it's the one in your session.

## What each permission mode actually trusts

Line up Claude Code's permission modes by where their trust comes from, and this outage locates itself immediately:

| Mode | Who decides | When the model is unavailable |
|------|-------------|-------------------------------|
| default | You, prompt by prompt | Unaffected |
| acceptEdits | Your one-time up-front grant, scoped to file edits | Unaffected |
| plan | A rule: read-only, always | Unaffected |
| **auto** | **A model, command by command** | **Chain breaks, everything blocked** |
| bypassPermissions | Nobody | Unaffected (and unprotected) |

Auto mode offers a genuinely good trade: one model call buys you out of dozens of "run this command?" interruptions. The cost stays invisible until the day it doesn't — **you have coupled the availability of your permission system to the availability of a model**. Every other mode resolves its verdict from you or from a static rule, with no network round trip. Only auto needs one.

This is not a design flaw. It's a priced trade-off. The problem is that the price is completely invisible while things work, so the first reaction on hitting it is always "the service is down, I'll wait" rather than "I should switch modes."

## Hypothesis one: blame the 1M variant

My `~/.claude/settings.json` had `"model": "opus[1m]"`, and `ANTHROPIC_BASE_URL` pointed at a third-party API relay. Put those two together and a very tidy explanation presents itself: relays are consistently weakest on **non-standard model IDs** — 1M-context variants, previews, suffixed specializations frequently exist on the official endpoint and are missing from a relay. Send a judgment request for a model ID the relay doesn't recognize, and of course nothing comes back.

So I changed the config, then ran `/model opus` to make it take effect immediately (**the `model` field in `settings.json` is only read on new sessions** — edit the file without switching and the identical error reproduces verbatim, as I found out by wasting one round on exactly that).

Then I reran the command:

```
claude-opus-5 is temporarily unavailable, so auto mode cannot
determine the safety of Bash right now.
```

Different model ID. Same dead classifier. **Hypothesis one is out.**

The reality is far more boring: that model was unavailable on my access path as a whole — an upstream relay failure, a quota, something in that family — and had nothing to do with the ID being "non-standard." I had promoted a plausible suspicion straight to a conclusion, skipping the verification step in between.

The failed fix did hand me an unplanned prize, though. It was precisely *because* the ID in the error tracked my `/model` switch that I got hard evidence the classifier is bound to the session model. **In debugging, a hypothesis you knock down is often worth more than one you confirm.**

## Hypothesis two: the allowlist will save me

The second idea was more seductive. I had written my frequent commands into `permissions.allow` as exact-prefix rules:

```json
"allow": ["Bash(cat *)", "Bash(ls *)", "Bash(jq *)", "Bash(rg *)"]
```

The reasoning: a command matching the allowlist should be released directly, never bothering the classifier — which makes the allowlist a degraded-mode channel for exactly this kind of outage. Airtight.

And I even "had evidence": during the outage, a command matching `Bash(jq *)` had gone through.

That evidence was fake. Replaying the whole session shows why: **several compound commands matching no allowlist rule at all had also gone through in the same window.** The outage was intermittent — the classifier was flickering. That `jq` succeeded because it happened to land in a few seconds when the classifier was alive. I had mistaken a coincidence for a mechanism.

A convincing experiment has to be able to produce a negative. So while the outage was still active, I ran:

```bash
ls /path/to/project/articles/claude/
```

`Bash(ls *)` **was an existing rule, written well before that day's edits**, and the command has no compound structure — the cleanest possible match. It was blocked anyway, with the same "cannot determine the safety" message. **Hypothesis two is out.**

The corrected conclusion: **under auto mode, an allowlist is not a moat. Every Bash command goes through the classifier.**

The methodological lesson is worth more than the conclusion: **while debugging an intermittent failure, "this command succeeded" proves almost nothing**, because you can't distinguish "it took an exempt path" from "the classifier happened to be alive that second." Only failures carry information — a command that *should* have been exempt and wasn't tells you the exempt path doesn't exist. To test a mechanism, go build the experiment that can fail.

## Hypothesis three: surely a different model family would work

With hypothesis two down, one idea remained: since it wasn't about the ID suffix, maybe it was opus specifically failing on this access path. Switch to a genuinely different family — `/model sonnet` — and see if the whole thing clears.

This was the most cautious hypothesis of the three, and the most likely to be right. Opus and sonnet are separate models; if only one of them were misbehaving, switching families should route around it.

Same command, rerun:

```
claude-sonnet-5 is temporarily unavailable, so auto mode cannot
determine the safety of Bash right now.
```

Different model ID again. Same dead classifier. **Hypothesis three is out.**

This time the conclusion is the cleanest of the three: **the problem isn't any specific model — the entire judgment path is down**, most likely a failure somewhere between my relay and the classification endpoint, unrelated to whether the front-end model is opus or sonnet. Three model IDs, three identical errors, and the only thing that ever changed was the string in the error text tracking `/model` — which, one last time, confirms that the classifier is bound to whatever session model you're running.

## So what actually works

With all three hypotheses down, what's left is the cleanest picture of the three: **no configuration-level fix can route around a judgment path that is down as a whole.** You can't know when it recovers, and model selection was never the lever, because the fault was never at the model-selection layer.

**The one dependable move: switch permission modes.** `Shift+Tab` back to default. Its verdict comes from you, it issues zero judgment requests, and the classifier's health becomes irrelevant. Confirming each action is tedious, but it's the only route standing after three controlled experiments ruled out everything else.

**Allowlists are still worth configuring — just don't file them under disaster recovery.** The confirmation dialogs they eliminate are a real day-to-day gain; they simply won't help when the classifier is down. Worth knowing: `deny` takes precedence over `allow`, so you can be generous with read-only commands and whatever you genuinely want blocked won't be overridden.

```json
{
  "permissions": {
    "allow": [
      "Bash(cat *)", "Bash(head *)", "Bash(tail *)", "Bash(find *)",
      "Bash(rg *)", "Bash(ls *)", "Bash(jq *)", "Bash(wc *)",
      "Bash(git status*)", "Bash(git diff*)", "Bash(git log*)",
      "Bash(npm run build*)", "Bash(npm run typecheck*)", "Bash(npm test*)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Read(./secrets/**)"
    ]
  }
}
```

## The general lesson

Set Claude Code aside and this outage is about something broader: **the moment you implement an infrastructure function with a model, that model's availability becomes the floor on that function's availability.**

Safety classification, request routing, intent detection, content moderation — all of these "model as middleware" designs buy flexibility that static rules can't reach, and pay for it with one more flaky external dependency. Whether the trade is worth it comes down to a single question: does the function have a **degradation path that doesn't involve the model**? Auto mode's degradation path isn't a config setting — it's falling back to a different mode, which means you have to know it exists before you need it.

Three takeaways at the debugging level:

- **The shape of the paralysis carries more information than the error text.** Feature A works, feature B doesn't: they don't share a call path. Following that asymmetry beats retrying and refreshing the status page.
- **In an intermittent failure, success is not evidence.** That's exactly how hypothesis two went wrong. To test a mechanism, go find the experiment that can fail.
- **It takes three hypotheses landing on the same error before you get to call it a conclusion.** Hypothesis one failing alone reads like a botched fix. Hypothesis three failing alone reads like bad luck. Only after three independent probes — dropping the suffix, testing allowlist exemption, switching model families entirely — all hit the identical error does "I did something wrong" stop being a plausible explanation, leaving "this is systemic" as the only one left standing.

In one line: auto mode has a model vet a model on your behalf. What you save is dozens of confirmations; what you stake is the availability of your permission system. There's no insurance policy on that trade — only an exit, and the exit is `Shift+Tab`.

Related reading: *Claude Code Plan Mode: Research First, Code Later, Rework Less* covers how a different permission mode cuts rework, and *Claude Code Hooks: Custom Automation Workflows* describes intercepting and rewriting tool calls outside the permission system. Both land where this piece does: the protection that doesn't need a model round trip is the one you can actually hold onto when things break.
