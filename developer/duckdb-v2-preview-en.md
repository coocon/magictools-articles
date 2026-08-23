# DuckDB v2.0 Preview: The Embedded Database Grows a Server Mode

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/duckdb-v2-preview-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/duckdb-v2-preview-en?utm_source=github&utm_medium=referral)**

On August 17, 2026, DuckDB co-creators Mark Raasveldt and Hannes Mühleisen published "A Preview of DuckDB v2.0". The stable release lands this fall, but preview builds already run most of what they announced.

Since v1.5 shipped in March, the project has accumulated more than 10,000 commits. The v2.0 codename is "Cyanoptera" (the cinnamon teal), and what it does is more radical than a version bump: **a database that built its identity on being in-process and dependency-free now officially has a server mode.**

One thing to clear up first. As this news spread, two numbers traveled with it: "aggregate queries 40x faster" and "compression improved by 20%." We checked the official post line by line — the 40x is real but the workload isn't aggregation, and the 20% figure **never appears in the post at all**. Here's the fact-checked version.

---

## The 40x Is Real — But the Star Is Recursive CTEs

The official post contains exactly three performance multipliers, and the big one belongs to **recursive CTEs**: a single-source reachability query over a graph with one million edges takes 4.90 seconds on v1.5.4 and 0.12 seconds on the v2.0 preview. The post itself adds the exclamation mark: "about 40× faster (!)".

That comes from a rewrite of the recursive query executor — a genuine win if your workload involves graph traversal, hierarchy expansion, or BOM explosion. If your pipeline is ordinary GROUP BYs and window functions, do not budget for 40x.

Aggregation did improve, but the post gives no multiplier for it: partial aggregates can now be pushed below joins, redundant aggregates get reused, and aggregations spill to disk when they exceed memory. The other two published numbers: Windows CLI result materialization got ~2.2x faster with multithreading, and after replacing ICU with an in-house implementation, timezone conversion over 25 million rows went from 0.24s to 0.11s (2.2x).

**This is a good refresher on how to read vendor benchmarks: find the exact scenario behind the number before deciding whether it applies to you.** Running your three slowest production queries against the preview beats retweeting any headline.

## The Real Headline: CONNECT and the Quack Protocol

Products in the SQLite lineage have upheld a decades-old doctrine: embedded databases don't do networking. v2.0 tears that up.

...

---

**[👉 Continue reading: DuckDB v2.0 Preview: The Embedded Database Grows a Server Mode](https://tools.cooconsbit.com/en/articles/duckdb-v2-preview-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
