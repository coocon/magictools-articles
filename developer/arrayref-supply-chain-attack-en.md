---
title: "86 Minutes: A Rust Supply Chain Attack Weaponized the Yank Mechanism"
slug: arrayref-supply-chain-attack-en
summary: "On August 20, 2026, arrayref was poisoned. The interesting part isn't the malware — it's that the attacker turned cargo's own yank warning into the delivery vector. Yank every good version, and the toolchain itself tells users to upgrade into the backdoor. The whole thing was live for 86 minutes, and it landed squarely on the hardest problem in Rust's dependency model: build.rs is arbitrary code execution at compile time."
category: developer
tags: [Rust, cargo, supply-chain-security, dependency-management, build.rs, software-supply-chain]
coverImage: ""
status: published
locale: en
source: authored
translationSlug: arrayref-supply-chain-attack
---

# 86 Minutes: A Rust Supply Chain Attack Weaponized the Yank Mechanism

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

```
warning: consider updating to a version that is not yanked
```

So what did the attacker do after taking over the account? **Yanked 0.3.5 through 0.3.9 — every good version.**

Which produced this situation across the ecosystem: your project pins `arrayref` 0.3.9, you run cargo, and the toolchain personally informs you that your version has been deprecated by its author and you should move to one that isn't yanked. The only version that wasn't yanked at that moment was 0.3.10, published at 07:15.

That was the entire social engineering budget. No phishing email, no forged documentation. The attacker simply **took over the package manager's safety warning and had it speak on their behalf**. A mechanism that nags you daily to stay current became "please upgrade into my backdoor."

Worth noting: the typosquatted dependency `proc-macro1` was impersonating dtolnay's `proc-macro2`, published from an account named `dtolney` — one letter off from the real maintainer, `dtolnay`. Scanning a dependency tree, you would not catch that.

One step in the Rust team's response was quietly smart: they didn't just delete the malicious versions, they **un-yanked the maliciously yanked ones**. Otherwise everyone would have finished cleanup still pinned to a version marked deprecated, with the lure still sitting there.

## 3. Why routine dependency auditing can't see it

The library code in all three poisoned crates was clean. Read `src/` in `arrayref` 0.3.10 and you'll find nothing.

The malicious logic lived entirely in `proc-macro1`'s `build.rs`.

This is the hardest structural problem in Rust's dependency model: **`build.rs` is arbitrary code execution at compile time**. Cargo compiles and runs it automatically — no confirmation, no capability declaration.

Which means:

**You don't need to run your program. `cargo build` is enough.**

You don't even need `cargo build`. `cargo check`, rust-analyzer indexing in the background, a dependency warm-up step in CI — anything that evaluates the build graph can reach it.

This particular build script rebuilt the attacker's server addresses from base64 (so static scanning finds no plaintext domain), pulled a second-stage binary over TLS **with certificate validation disabled**, and executed it during compilation.

Compare that to your normal defenses and notice how few of them sit on this path:

- **Code review**: the library code is clean. It passes.
- **`cargo audit` / RustSec**: driven by a known-vulnerability database. The window was 86 minutes; by advisory time you were either already hit or already fine.
- **Lockfiles**: `Cargo.lock` does pin versions — but the entire point of that yank warning is to talk you into changing the lockfile.
- **CI sandboxing**: if your runner holds cargo/npm credentials, cloud credentials, or SSH keys, compile-time RCE is credential compromise.

Exactly one thing would reliably have caught this: **someone or something looking at what dependencies a new version added, at the moment it was pulled.** `arrayref` had zero dependencies for ten years and suddenly had one. That signal is loud enough — if anyone is watching for it.

## 4. What to do right now

Get the version relationship straight first, because plenty of secondhand coverage has it backwards:

**0.3.10 is the malicious version. The last safe version is 0.3.9. You pin backward, not upgrade forward.**

Safe versions for all three:

- `arrayref` → **0.3.9**
- `internment` → **0.8.6**
- `append-only-vec` → **0.1.8**

If you see advice telling you to "upgrade to 0.3.8 or later," that sentence would have walked people straight into the malicious version on August 20. The malicious releases are deleted from crates.io now and can no longer be installed, but make sure the guidance you're working from is correct.

### Check whether your machine actually pulled it

The advisory ships a command that inspects the local cache directly. This is more reliable than reading `Cargo.lock` — it reflects what this machine actually downloaded:

```bash
find ~/.cargo/registry/cache -type f \( \
  -name 'append-only-vec-0.1.9.crate' -o \
  -name 'arrayref-0.3.10.crate' -o \
  -name 'internment-0.8.7.crate' -o \
  -name 'proc-macro1-*.crate' -o \
  -name 'proc-macro-en-*.crate' -o \
  -name 'aovine-*.crate' -o \
  -name 'arone-*.crate' -o \
  -name 'aronenao-*.crate' -o \
  -name 'tinymember-*.crate' \
\) -print
```

No output means this machine never downloaded any of them. You're clear.

### Check the lockfile too

```bash
grep -A2 'name = "arrayref"' Cargo.lock
grep -n 'proc-macro1' Cargo.lock
```

A `0.3.10` or any `proc-macro1` entry means the payload executed on that machine.

### If you were hit

This isn't a "just bump the version" situation. Compile-time execution means the attacker got user-level access on a build machine:

1. Look for dropped binaries.
2. Check egress logs — public analyses name `23.254.165.112`.
3. **Rotate every credential on that machine**: crates.io tokens, npm tokens, SSH keys, cloud credentials, CI secrets. Whatever the build host could reach.

The RustSec advisory is RUSTSEC-2026-0260. As of publication no CVE was assigned and there's no evidence of widespread exploitation. But "no evidence" and "didn't happen" aren't the same claim, particularly when the window covered European and Asian working hours.

## 5. Attribution: not random opportunism

Wiz's analysis found significant infrastructure overlap between this campaign and recent DPRK-linked supply chain attacks, including the operations against `Mastra` and `axios`.

That changes what the event means. A random script kid is bad luck. An organized team with sustained investment, that stages infrastructure two days ahead and studies cargo's yank semantics closely enough to weaponize them, is something else — because whether *this* attempt succeeded matters less than the fact that **the technique will be reused**.

Yank-as-lure transfers cleanly. npm has `deprecate`. PyPI has yank. Any ecosystem where a maintainer can flag old versions as not-recommended has an isomorphic attack surface. The only variable is whether that ecosystem's build process has a `build.rs`-shaped hole that executes code by default — npm's `postinstall` does, Python's `setup.py` does.

## 6. The structural problem underneath

Short-term remediation has a ceiling. The real issue is that several defaults in cargo's trust model are on the attacker's side:

**Default one: build.rs executes silently.** No permission model, no network access declaration, no "this crate wants to make network calls at compile time — allow?" prompt. The Cargo team has discussed sandboxing build scripts for years; nothing has shipped.

**Default two: adding a dependency creates zero friction.** A crate with a ten-year zero-dependency history adds one, and `cargo update` doesn't say a word. The only signal you get is the lockfile diff — which most teams collapse in code review.

**Default three: the yank warning trusts the maintainer account unconditionally.** That's precisely what got used here. The toolchain assumes the account speaks for the maintainer, and accounts get stolen.

Until those defaults change, the effective moves are limited but real:

- **Ban automatic `cargo update` in CI.** Dependency bumps should be a PR a human reviews.
- **Separate build environments from credentials.** If compilation doesn't need your cloud keys, don't let it reach them.
- **Weight lockfile diffs in review** — specifically watch for *added dependencies*, not just version bumps.
- **`cargo vendor` or a dependency mirror** is worth the cost on high-value projects. It converts "upstream can change at any time" into "upstream changed and I have to actively agree."

A closing note on the response: 86 minutes from report to deletion is genuinely good, and remembering to un-yank was the right instinct. But fast response was partly luck — Nextron Systems' research team happened to find and report it. Absent that report, how long would `arrayref` 0.3.10 have stayed up?

That's the part that should keep you up at night.

---

**Sources**

- [Supply chain attack on arrayref — Rust Blog (official advisory)](https://blog.rust-lang.org/2026/08/20/supply-chain-attack-on-arrayref/)
- [Malware: arrayref 0.3.10 executes a remote payload at build time via typosquatted proc-macro1 — rustsec/advisory-db #3161](https://github.com/rustsec/advisory-db/issues/3161)
- [Malicious Rust Crate arrayref Runs a Build-Time Payload — SafeDep](https://safedep.io/arrayref-proc-macro1-rust-build-time-malware/)
- [Rust Supply Chain Attack on arrayref: Significant Overlap with DPRK Campaigns — Wiz](https://www.wiz.io/blog/rust-supply-chain-attack-on-arrayref-significant-overlap-with-dprk-campaigns)
- [Rust Crates arrayref & append-only-vec Compromised — Semgrep](https://semgrep.dev/blog/2026/rust-crates-arrayref-append-only-vec-compromised-proc-macro1/)
- [arrayref, internment, and append-only-vec Poisoned by the proc-macro1 Build-Time Dropper — StepSecurity](https://www.stepsecurity.io/blog/arrayref-rust-crate-supply-chain-attack)
