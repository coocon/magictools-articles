# 86 Minutes: A Rust Supply Chain Attack Weaponized the Yank Mechanism

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/arrayref-supply-chain-attack-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/arrayref-supply-chain-attack-en?utm_source=github&utm_medium=referral)**

> "We do not believe the author of arrayref to be acting maliciously, but their computer or credentials are likely compromised."
> — Rust Security Response Team, 2026-08-20

---

At 07:15 UTC on August 20, 2026, the Rust Security Response Team received a report that the `proc-macro1` crate was malicious. They confirmed it: the crate shipped a build script that downloaded and executed a remote payload at compile time.

Pulling that thread made the story much bigger. For the first time in ten years, `arrayref` had added a dependency — and it was `proc-macro1`.

`arrayref` is the kind of crate you've probably never written into a `Cargo.toml` yourself, but almost certainly have in your dependency tree. 245 million all-time downloads on crates.io, 53.7 million in the last 90 days, 403 crates depending on it directly. It sits underneath `blake3`, `winit`, and `tiny-skia`, and underneath large parts of the Solana and Ethereum tooling stacks.

The malicious version was live for 86 minutes. But what happened in those 86 minutes is worth a careful read by anyone who runs cargo — because the attacker took a mechanism that exists **to protect you** and turned it into the lure.

## 1. The precise timeline

The official advisory gives clean numbers. Here they are verbatim:

| Crate | Malicious version | Published (UTC) | Deleted (UTC) | Time live |
|---|---|---|---|---|
| `arrayref` | 0.3.10 | 2026-08-20 07:15:00 | 08:41:40 | 86 min |
| `internment` | 0.8.7 | 2026-08-20 07:34:07 | 09:04:11 | 90 min |
| `append-only-vec` | 0.1.9 | 2026-08-20 07:37:49 | 09:25:24 | 107 min |

Plus a set of attacker-owned crates deleted wholesale (all versions): `proc-macro1`, `proc-macro-en`, `aovine`, `arone`, `aronenao`, `tinymember`.

All three poisoned crates belong to the same author. The Rust team's assessment is that the author was not acting maliciously — their machine or credentials were compromised. The account was locked as a precaution.

One detail that's easy to skim past: **the attacker's infrastructure was already in place**. The crates `arone` and `aronenao`, both carrying malicious build scripts, had their final publishes on 2026-08-18 — two days before the `arrayref` push. This wasn't opportunistic. It was staged and waiting for a credential to land.

## 2. The genuinely novel part: yank as the lure

"Sneak a malicious dependency into a popular crate" is an old play. What deserves its own section here is the delivery.

First, what `yank` means. On crates.io, yanking a version says: this version still downloads (existing lockfiles won't break), but stop using it. It's a normal maintainer tool for flagging buggy or insecure releases. When your dependency tree contains a yanked version, cargo tells you:

...

---

**[👉 Continue reading: 86 Minutes: A Rust Supply Chain Attack Weaponized the Yank Mechanism](https://tools.cooconsbit.com/en/articles/arrayref-supply-chain-attack-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
