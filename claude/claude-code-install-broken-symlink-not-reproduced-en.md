---
title: "claude install Creates a Broken %h Symlink? Clean Debian Test Comes Out Fine (Not Reproduced)"
slug: claude-code-install-broken-symlink-not-reproduced-en
category: claude
locale: en
translationSlug: claude-code-install-broken-symlink-not-reproduced
tags: [Claude Code, installation, install.sh, symlink, Linux, claude-code-lab]
summary: "A Fedora user reports on GitHub that the official one-line installer leaves ~/.local/bin/claude as a broken symlink pointing at the literal %h/.local/share/claude/versions/2.1.220 — the %h placeholder never expanded to the home directory. We ran the exact same install command in a clean Debian 12 container: the resulting symlink is correct (%h properly expanded), and install.sh now ships 2.1.223 (the report was against 2.1.220). This is a not-reproduced field report: full environment, commands, and raw output are included, along with three candidate explanations and a one-minute manual fix if you are currently stuck on the broken link."
status: published
lab:
  testedAt: "2026-08-06"
  ccVersion: "2.1.223"
  model: "n/a (install flow, model-independent)"
  platform: "Linux (Docker debian:bookworm-slim)"
  status: not-reproduced
docsUrl: https://docs.claude.com/en/docs/claude-code/setup
---

> **What this article is**: a not-reproduced field report. We could not reproduce the reported problem — which does not mean the report is wrong, only that it does not show up in the environment and with the method below. Every command and its raw output is included; cross-check us.

## What the original report says

GitHub issue [#83484](https://github.com/anthropics/claude-code/issues/83484) (Fedora 44 + bash, Claude Code 2.1.220): after running the official one-line installer, `~/.local/bin/claude` is a broken symlink —

```console
$ curl -fsSL https://claude.ai/install.sh | bash
$ file ~/.local/bin/claude
/home/user/.local/bin/claude: broken symbolic link to %h/.local/share/claude/versions/2.1.220
```

The `%h` in the target is an unexpanded placeholder (the systemd-style home-directory specifier) that should have become `/home/user`. The reporter traced the problem not to the shell installer but to the binary's internal `install` subcommand, and confirmed it is a regression ("this worked in a previous version"). The issue links a fix PR (#83738, unmerged at reporting time).

## Our test: it comes out fine

Environment: a clean `debian:bookworm-slim` Docker container, root user, 2026-08-06, running the exact command from the report:

```console
$ curl -fsSL https://claude.ai/install.sh | bash
$ ls -la /root/.local/bin/claude
lrwxrwxrwx 1 root root 42 Aug  6 10:23 /root/.local/bin/claude -> /root/.local/share/claude/versions/2.1.223
$ file /root/.local/bin/claude
/root/.local/bin/claude: symbolic link to /root/.local/share/claude/versions/2.1.223
```

Two key observations:

1. **The symlink is valid** — the home directory in the target path expanded correctly to `/root`, with no literal `%h` anywhere
2. **install.sh currently ships 2.1.223**, three releases past the reported 2.1.220 (2.1.221/222/223 released August 3/4/5)

## Why it did not reproduce: three candidate explanations

Ordered by strength of evidence:

**Explanation 1: fixed somewhere in 2.1.221–2.1.223 (most likely).** The report was against 2.1.220, a fix PR exists, and install.sh now ships 2.1.223 — if the fix landed in any of those releases, fresh installs simply no longer hit it. We could not find release notes explicitly saying "fixed %h expansion", so this remains unconfirmed.

**Explanation 2: distro-specific.** The report is from Fedora 44; we tested Debian 12. `%h` is the home-directory specifier used in systemd unit files — if the binary's install subcommand reads a path template from systemd-related configuration on some systems, behavior could vary by distro. This is a mechanism guess; we have no Fedora environment to verify it.

**Explanation 3: leftover state in the reporter's environment.** The reporter said a previous version worked, so remnants of the older install may have participated in path generation. A clean container has no remnants — which might be exactly why we cannot trigger it. Equally unconfirmed.

## Stuck on the broken link right now? The one-minute fix

Whatever the root cause, the repair is deterministic (the workaround the reporter verified in the issue):

```bash
# Point the broken link at the version directory that actually exists
ls ~/.local/share/claude/versions/        # see what versions you have
ln -sf ~/.local/share/claude/versions/<version> ~/.local/bin/claude
claude --version                          # verify
```

Or simpler: re-run the official install command — the 2.1.223 it currently ships came out clean in our test.

## Scope and boundaries

- This article proves only: **a clean Debian 12 environment + the 2026-08-06 install.sh (shipping 2.1.223) does not exhibit the problem**
- It does not prove: that Fedora is fixed, that the 2.1.220 problem never existed, or that every environment is fine
- If you can still reproduce it on another distro (especially Fedora with 2.1.221+), please add your environment details to issue #83484 — that would directly falsify explanation 1 and substantiate explanation 2
