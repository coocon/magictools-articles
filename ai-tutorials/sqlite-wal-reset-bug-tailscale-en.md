# The SQLite Bug That Hid for 16 Years: How Tailscale Found It

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/sqlite-wal-reset-bug-tailscale-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/sqlite-wal-reset-bug-tailscale-en?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 Continue reading: The SQLite Bug That Hid for 16 Years: How Tailscale Found It](https://tools.cooconsbit.com/en/articles/sqlite-wal-reset-bug-tailscale-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
