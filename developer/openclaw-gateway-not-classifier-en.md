---
title: "Half My openclaw Commands Ran, Half Didn't — It Looked Like a Permission Classifier, It Was launchd"
slug: openclaw-gateway-not-classifier-en
category: developer
locale: en
translationSlug: openclaw-gateway-not-classifier
tags: [openclaw, launchd, troubleshooting, CLI tools, permissions, developer workflow]
summary: "Same machine, same user, same global config. Two Claude Code windows running the same CLI — one worked, one didn't. The obvious suspect was the permission classifier, which really does block commands. But the cause sat a layer down: the daemon's plist was installed and never loaded, so every gateway-bound subcommand died while purely local ones printed fine. That half-working shape is what sells the permission theory. Full trace, including the hypothesis I got wrong by misreading my own logs."
status: published
---

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

```json
"allow": ["Bash", "Read", "Edit", ...]
```

A **bare `Bash`** — no parenthesized qualifier, so every Bash command is allowed. It applies identically to both windows, because global config doesn't discriminate by project in the first place.

Then window A's project-level `.claude/settings.local.json`:

```json
{
  "permissions": {
    "allow": ["Bash(cd:*)", "Bash(swift build:*)", "Bash(git status:*)", ...]
  }
}
```

An `allow` list and no `deny`. Project settings in Claude Code **stack onto** the global ones rather than replacing them, so a file containing only `allow` entries cannot revoke anything already permitted.

**Hypothesis 1 is out.** There is no permission-level difference between the two windows.

## Hypothesis 2: that config file is corrupt

Step back a level. What if `settings.local.json` fails to parse? A BOM, a stray invisible byte, a JSON syntax error — a parse failure could plausibly degrade the whole project to confirm-everything mode.

Easy to falsify, two commands:

```bash
$ xxd .claude/settings.local.json | head -1
00000000: 207b 2020 0a20 2020 2022 7065 726d 6973   {  .    "permis

$ python3 -c "import json; json.load(open('.claude/settings.local.json')); print('OK')"
OK
```

The file is genuinely a little grubby — the first byte is `0x20`, meaning there's a space before the `{`, plus two trailing ones. But there's no BOM (it doesn't start with `EF BB BF`), and the parser accepts it without complaint.

**Hypothesis 2 is out.** Leading whitespace is legal JSON. Ugly, harmless.

## Hypothesis 3: the orphaned log entries mean commands were denied

This machine has a hook that records every tool call into `~/tools/logs/claude-hooks-*.jsonl`. Grouping by session showed both windows cleanly:

```
('159f3a23', '/Users/duoduo/4khz/tingease')            37 entries   ← window A
('cb28e90b', '/Users/duoduo/4khz/tingease/AudioCore')  41 entries   ← window B
```

Reading window A's entries, something jumped out: several `PreToolUse` records had **no matching `PostToolUse`**.

```
[00:50:09] PreToolUse       ← unmatched
[00:50:33] PreToolUse       ← unmatched
[00:50:53] PreToolUse
[00:50:53] PostToolUse      ← properly paired
[00:51:03] PreToolUse       ← unmatched
```

The reasoning felt airtight at the time: `PostToolUse` only fires after a tool actually executes, so an orphaned `PreToolUse` is a call the permission layer killed. Not only did this confirm the permission theory, it came with second-level timestamps.

**The reasoning was wrong**, and wrong in a very ordinary way. A missing `PostToolUse` only tells you *this call didn't complete normally*. It does not distinguish a permission denial from a command that errored, timed out, or got interrupted. I treated a multi-valued signal as single-valued evidence — and the value I assigned happened to be the one I already believed.

The experiment that actually discriminates is to go to that window's directory and run the thing:

```bash
$ cd /Users/duoduo/4khz/tingease && openclaw gateway status
CLI version: 2026.7.1-2 (/opt/homebrew/bin/openclaw)
Gateway version: 2026.7.1-2
Runtime: running (pid 80719, state active)
Connectivity probe: ok
Capability: admin-capable
```

**Hypothesis 3 is out.** Same directory, same command, perfectly healthy. Not cwd. Not permissions.

## The decisive evidence was a timeline, not a status check

That last command exposed the sharper problem: **it's healthy now.** Window A was broken *then*. A status snapshot cannot answer "what was happening at the time." Only a timeline can.

Put the hook log's timestamps next to what I measured when I picked the problem up:

```
00:49:58  [window A]  "go look at what's wrong with openclaw and fix it"
00:52:36  [window A]  "startup failed, please fix openclaw"
01:12:10  [window A]  "openclaw gateway status"
01:1x     [window B]  I take over; first diagnostic below
```

My first command was `openclaw daemon status`:

```
Service: LaunchAgent (not loaded)
Runtime: unknown (Bad request.
  Could not find service "ai.openclaw.gateway" in domain for user gui: 501)
Connectivity probe: failed
  connect ECONNREFUSED 127.0.0.1:18789
```

`ECONNREFUSED` means nothing was listening on 18789 at all. Across the entire stretch window A spent struggling — 00:49 to 01:12 — the gateway had never been up.

And the only reason window B "could run openclaw" is that my very first action was:

```bash
$ openclaw daemon start
Gateway LaunchAgent was installed but not loaded; re-bootstrapped launchd service.
```

I didn't invoke it more cleverly. I repaired the service. **The two windows were never treated differently — they reached the same shared service at different times, and its state changed in between.**

## Root cause: installed, never loaded

That message says all of it: `installed but not loaded`.

launchd takes services in two steps. A plist sitting in `~/Library/LaunchAgents/` completes step one; it still has to be bootstrapped into the user's launchd domain to actually be managed. Here step one succeeded and step two never did, producing an awkward in-between state:

- `openclaw daemon status` reads the plist and reports the service as installed
- launchctl has no such label, hence `Could not find service ... in domain for user gui: 501`
- nothing listens on the port, so every gateway-bound subcommand gets `ECONNREFUSED`

`openclaw daemon start` supplies the missing second step.

With that fixed, I checked the persistence settings so it wouldn't recur on the next reboot:

```bash
$ plutil -p ~/Library/LaunchAgents/ai.openclaw.gateway.plist | grep -E "RunAtLoad|KeepAlive"
  "KeepAlive" => true
  "RunAtLoad" => true

$ launchctl print-disabled gui/501 | grep openclaw
  "ai.openclaw.gateway" => enabled
```

`RunAtLoad` handles start-on-login, `KeepAlive` handles restart-on-crash, both true, and it had never been `launchctl disable`d. The configuration was sound — the unloaded state was a one-off, not a design flaw.

Because the service is machine-wide, fixing it restored both windows at once. Window A had to change nothing.

## The general lesson

**When identical environments give different results, check the shared stateful dependency before you check their individual configs.**

Config belongs to each window. The service is shared by the whole machine. When the difference between two environments is *time* rather than *settings*, only a shared stateful component can explain it. I had that backwards — I spent my first minutes diffing two settings files and validating JSON, all of which is stateless, and **stateless things don't break only during a particular window of time.** One question would have redirected me in three minutes: what do these two windows share that can change on its own?

Three more:

- **"Some features work" is routing information, not filtering information.** When class A works and class B doesn't, the first instinct shouldn't be "something is picking and choosing" but "these two classes take different paths — where do they diverge?" Find the dividing line — here, whether a WebSocket gets opened — and the root cause is usually already visible.
- **Absence in a log is a multi-valued signal.** That's precisely where hypothesis 3 fell over. A missing `PostToolUse` could be denial, error, timeout, or interruption. Using a multi-valued signal to confirm a hypothesis you already lean toward is one of the easiest traps in debugging — it hands you confidence, not information.
- **A snapshot can't testify about history.** "It works when I run it now" does not refute "it was broken then." Reconstructing the past takes timestamped records, cross-referenced against a second independent timeline — here, the hook log's prompt timestamps lined up against the `ECONNREFUSED` I hit when I took over.

The one-line version: when a CLI runs half its commands and refuses the other half, don't start by assuming something is screening them — go find the dependency that splits them in two. Most of the time the rejected half simply shares a service that has quietly fallen over.

Related reading: our piece on [Claude Code's auto mode judging a model with a model](/en/articles/claude-code-auto-mode-permission-trap-en) describes what it looks like when the permission classifier *genuinely* fails — a useful contrast. There, "reads work, commands don't" was caused by the classifier. Here, an almost identical half-paralysis had nothing to do with permissions at all. Read together they train the instinct that separates them: **are the failing commands grouped by how dangerous they are, or by what they depend on?**
