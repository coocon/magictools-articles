---
title: "The SQLite Bug That Hid for 16 Years: How Tailscale Found It"
slug: sqlite-wal-reset-bug-tailscale-en
summary: "Nineteen database corruptions in six months, all traced to a data race that had been sitting in SQLite for roughly 16 years. Here are 10 lessons from Tailscale's investigation into the WAL-Reset bug."
category: ai-tutorials
tags: [SQLite, Tailscale, database, WAL, debugging, data race, postmortem]
coverImage: ""
status: published
locale: en
source: authored
translationSlug: sqlite-wal-reset-bug-tailscale
---

# The SQLite Bug That Hid for 16 Years: How Tailscale Found It

> Based on Tailscale's engineering blog post "How we tracked down a 16-year-old SQLite bug" (2026-08-12). All quotes are verbatim from the original. Link at the end.

---

Some context first, or the clues won't make sense.

Tailscale's control plane looks like a single endpoint from the outside (controlplane.tailscale.com). Internally it's split into coordination servers they call shards. Each tailnet lives on exactly one shard at a time and can migrate seamlessly. Each shard has its own SQLite database, accessed exclusively by a single Go process — the textbook way to run SQLite. They've been on this setup since 2022, incident-free since early 2023.

Then last August, a data pipeline reading their S3 backups threw an error. `PRAGMA integrity_check` confirmed it: the database was corrupt. They repaired it, investigated, found nothing.

Over the next six months, that happened eighteen more times.

Here are the 10 things worth taking away.

---

## 1. Scale Turns "Vanishingly Rare" Into "Again This Month"

> "When operating at scale, even rare events can occur with some frequency. In total, we faced 19 separate instances of database corruption over six months before we finally resolved the underlying bug."

SQLite corruption is possible but highly unusual. Any single occurrence gets filed under cosmic rays and forgotten. Nineteen occurrences is no longer an accident — it's a defect with a return address.

Worth noting how precisely they scope the blast radius: these databases hold metadata about tailnets and devices, never private encryption keys or network traffic. In the earliest incidents, recovery meant a handful of newly added devices or configuration changes didn't persist. And every single corruption required stopping the control plane process on that shard to repair or restore — over an hour of downtime in the early days.

They also make a point most engineering teams skip: they post a global incident on their status page even when only a few tailnets are affected, because repeated downtime erodes trust whether or not you were personally hit.

**My take:** "Statistically impossible" is not a conclusion. It's a denominator you haven't calculated yet. Every order of magnitude you add — instances, requests, uptime — drags the tail closer to the center. Scale is a machine for converting theoretical failures into weekly ones.

---

## 2. A Textbook-Clean Architecture Still Got Hit

> "A single Go process exclusively accesses that database, and serves the control plane for those tailnets. This single-writer design is exactly how SQLite is meant to be used."

Early in the story this reads as an alibi: no competing writers, no SQLite over NFS, no pretending it's a distributed database. Because the architecture was clean, every "you're holding it wrong" explanation was eliminated up front.

But the real payoff arrives in point 5. Precisely because it's single-writer with serialisable transactions, the transaction history is completely linear and deterministic — something that simply isn't true in a multi-writer database like Postgres or MySQL.

**My take:** A clean architecture doesn't prevent incidents. It determines whether you can investigate them. Reproducibility, replayability, attributability — those are decided at design time, not at incident time. You can't bolt them on at 3 a.m.

---

## 3. When Every Correlation Comes Back Empty, Go Forensic

> "It wasn't tied to a single shard, or customer, or tailnet feature, or time of day, or load level."

The standard playbook is correlation hunting: which host, which customer, which new feature, which hour, how much load. Tailscale checked every axis. All empty. They also reviewed recent changes and went through all their low-level SQLite code with a fine-toothed comb. Also nothing.

No reliable trigger conditions means no synthetic reproduction. So the strategy had to change:

> "Instead, we had to rely on deploying passive, forensic telemetry in our live environment to catch the corruption red-handed."

**My take:** There are two ways to chase a bug you can't reproduce: build an ever-more-elaborate repro rig, or turn production into an evidence chain. When the trigger conditions are unknown, the first is a money pit — you're guessing at the very thing you're trying to learn. The second requires shipping diagnostic code to live systems, which is less a technical problem than an organizational permission problem.

---

## 4. The Six Quiet Weeks Were the Most Dangerous Part

> "We had a six-week period between October and December when there were no corruption incidents, before they returned as an unwelcome Christmas present."

Incidents came hours apart, then weeks apart. If any plausible fix had shipped near the start of that six-week gap, it would almost certainly have been recorded as the fix.

This detail gets detonated in point 10. Having been burned once by a deceptive calm is exactly why they later refused to close the case on silence alone.

**My take:** Intermittent bugs aren't dangerous because they break things. They're dangerous because they manufacture false evidence. A quiet stretch plus a coincidental deploy produces a causal story out of thin air, and the team then builds on that story for months. **Absence of reproduction is not repair.** It's just a miss.

---

## 5. The Turning Point: A Committed Write That Wasn't There

> "In two incidents, our transaction logs failed to replay cleanly. Upon closer inspection, we discovered that data written and committed by one transaction was inexplicably invisible to later transactions. A write had vanished into thin air without raising an error. That should be impossible!"

While keeping the platform running, they built a transaction logging pipeline: every SQL statement that modified the database got streamed to a separate log file. Single-writer plus serialisable transactions means that log replays linearly by construction — so if the replay doesn't come out clean, reality and the log have diverged.

Committed. No error. Invisible to everything after it. That combination kills a whole category of theories at once: not an application logic bug, not a write conflict, but **the storage layer losing data it had already promised to keep**.

**My take:** This is the most elegant move in the whole investigation. It converts "how did the database get corrupted" into "where did one committed write go" — a dramatically smaller and more concrete question. Good debugging isn't about gathering more information. Every step should shrink the search space.

---

## 6. Ten Pages in the WAL, Twenty Pages Copied Out

> "If there are 10 pages in the WAL file and 20 pages get copied to the database, something is clearly wrong."

Quick refresher on the mechanism. A SQLite database is a series of pages; updating data means replacing some of them. With Write-Ahead Logging enabled, new pages don't go straight into the database file — they go into the WAL file first. The WAL can't grow forever, so at some point those pages get copied back into the main database file. That process is **checkpointing**.

Their metrics showed SQLite reporting more pages copied out of the WAL than the WAL actually contained. The arithmetic didn't work, and checkpointing became the prime suspect.

To dig deeper, the SQLite developers built a wrapper around the virtual filesystem — the **tmstmpvfs shim** — that emits extra tracing information and logs. Tailscale deployed it into production and waited for the next corruption.

**My take:** The lesson here isn't the tool, it's the ordering: hypothesis first, instrument second. They had already narrowed suspicion to checkpointing, then built a probe that watches only checkpointing. Doing it the other way around — install broad observability and hope the truth surfaces from the dashboards — usually just buys you more expensive noise.

---

## 7. Sixteen Years Unnoticed, and Rare Enough to Need Deliberate Triggering

> "It could exist that long because it was rare — so rare, the SQLite developers had to add code to deliberately trigger it in their testing environments."

The SQLite developers named it the **WAL-Reset bug** and estimate it was present in SQLite for at least 16 years. The bug itself is a rare data race between a checkpoint and a write transaction:

> "If a write occurs at a specific time during a checkpoint, the checkpointing process gets confused — it thinks some of the pages have been copied from the WAL into the main database file, but they haven't. Those pages never get written to the database file, and that data is permanently lost."

Why does the database become *corrupt* rather than merely lossy? Because other pages that reference those pages — an index, for example — do get written to the database file. The pointers survive; their targets don't. The fix adds a check to the checkpointing function that detects when the WAL has been reset by another thread.

**My take:** "This code has run for 16 years without incident" is the most seductive false comfort in software. All it proves is that nobody assembled the trigger conditions in those 16 years, not that the conditions don't exist. SQLite is among the most aggressively tested open-source projects on earth and it still hid this for 16 years. What exactly makes your legacy module cleaner?

---

## 8. Why Tailscale, and Not Everyone Else

> "They also explained why we were more likely to hit the bug than other SQLite users: we take manual control of the checkpointing process, and we checkpoint very aggressively."

Why take manual control at all? Backups. Their pipeline takes a complete snapshot of the database every few minutes and uploads the whole SQLite file to S3. Getting a consistent snapshot means owning the timing of checkpoints yourself.

It's a familiar trade: they bought determinism in backups by giving up determinism in the default code path. The blog's own assessment is admirably restrained — "This non-standard approach seemed suspicious."

**My take:** A data race is a probability, and call frequency is your sample count. If other users checkpoint a few times a day and you checkpoint a few times a minute, your exposure to the same latent bug can be thousands of times higher. **Cranking any operation up an order of magnitude effectively volunteers you as the world's stress test** — you find the bug first, in production, on your own uptime.

---

## 9. Fix One Bug, Trip Another: 3.52.0 Was Withdrawn

> "Because this change caused false corruption warnings, the SQLite developers withdrew the 3.52.0 release and instead published 3.51.3, which only contained a fix for the WAL-Reset bug."

The rollout deserves its own note. They shipped 3.52.0 to a few canary shards first, then to the rest — and the backup monitor promptly turned red, reporting corruption in 13 different databases.

False alarm. Those databases weren't actually corrupt; they'd hit a separate problem involving stale expression indexes. Tailscale was storing high-precision timestamps as text and converting them to floating-point in a VIRTUAL generated column. The same 3.52.0 release that fixed the data race also included an optimisation that subtly changed rounding behaviour for text-to-float conversions. The index held the old rounding; recomputation produced the new one; the check called it corruption.

So the SQLite team pulled 3.52.0 and shipped 3.51.3 with only the WAL-Reset fix. Tailscale reduced their timestamp precision to integer seconds, and SQLite added automated self-healing indexes in 3.53.0.

**My take:** Two signals here. First, **canaries are not ceremony** — a full-fleet rollout would have produced 13 simultaneous corruption alerts immediately after shipping a corruption fix, and the natural first conclusion would have been that the fix caused it. Second, version numbers don't encode safety. 3.52.0 is newer than 3.51.3, and 3.51.3 is the correct choice. Read release notes and withdrawal notices, not integers.

---

## 10. Silence Isn't Proof. Get a Positive One.

> "An absence of corruption incidents doesn't mean things are fixed — we'd already had one six-week period of deceptive calm. We wanted positive proof that this data race was actively occurring."

So they did something counterintuitive: they patched their already-fixed SQLite driver to **log a warning whenever the two operations overlap** — the exact condition the bug required. Then they deployed it and waited.

Two months later the alert fired: "SQLite attempted corruption... in party mode, but the system prevented it."

That alert is the whole point. It proves the precise conditions for the WAL-Reset bug do occur in their production environment, which means the patch is doing real work rather than sitting next to a coincidence. Since it fired, they've run another four months with no database incidents.

**My take:** "No evidence the problem is still happening" and "evidence the problem is being caught" are entirely different claims. The first reassures a team; only the second closes a case. Most teams delete the instrumentation the moment a fix ships, throwing away the one signal that would have been worth keeping. **Leave behind a log line whose only job is to prove the fix is earning its keep.** It costs almost nothing.

---

## Closing: Where "Boring Technology" Stops Protecting You

The heaviest line in the post is the last lesson:

> "This investigation is a useful reminder: running boring technology in a non-standard way is a risk. The common paths and standard configurations are incredibly well-tested and reliable. Everything we were doing was a public, documented, supported configuration — but by taking manual control of the checkpointing process and running at our own aggressive pace, we stepped off the well-trodden operational path."

Note what they don't say. They don't say don't use SQLite, and they don't retract the boring-technology principle. What they say is sharper: **choosing a battle-tested technology is not the same as choosing a battle-tested path through it.** Reliability accumulates on the main road, worn smooth by everyone who walked it before you. The moment you take an exit ramp for a perfectly good reason — consistent backups, in this case — that accumulated reliability stops transferring automatically.

"Public, documented, supported" deserves a second look too. Those three words are the highest grade most architecture reviews can award, and they still don't mean "validated by a large population of real production systems." Documented as legal and exercised a hundred thousand times a day are different properties.

One more thing worth flagging: Tailscale paid for a professional support contract with the SQLite developers, and funded the open-source VFS shim that isolated the race condition. The bug got fixed, and every SQLite user on earth benefits — including everyone who paid nothing. That's a rare case of open-source economics closing the loop cleanly, and it's worth naming when it happens.

**Three practical takeaways for the rest of us:**

1. **Check your SQLite version.** If you use WAL mode and drive checkpoints manually, move to 3.51.3 or later — note that's *not* 3.52.0, which was withdrawn. Most users on defaults are at very low risk, but confirming costs nothing.
2. **Rehearse restores; don't just have backups.** Tailscale live-tested their recovery process over a dozen times during this period and drove downtime from over an hour to well under it. A restore path nobody has actually run is not a restore path.
3. **Write down every place you leave the default path.** One line per deviation: what we do differently, why, and what we gave up. On the day something breaks, that list is the first map anyone reaches for.

Their own closing is the right one to borrow. The period was frustrating, but they came out stronger than they went in: the long-standing SQLite bug is patched, dozens of incidental issues they spotted along the way got fixed, and the backup and recovery process has been proven under fire.

**Chasing one hard enough bug tends to leave you with more than the fix.**

---

*Source: Tailscale engineering blog, "How we tracked down a 16-year-old SQLite bug"*
*Original: https://tailscale.com/blog/sqlite-wal-reset-bug*
