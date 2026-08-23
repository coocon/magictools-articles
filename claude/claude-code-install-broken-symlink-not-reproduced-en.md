# claude install Creates a Broken %h Symlink? Clean Debian Test Comes Out Fine (Not Reproduced)

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-install-broken-symlink-not-reproduced-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-install-broken-symlink-not-reproduced-en?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 Continue reading: claude install Creates a Broken %h Symlink? Clean Debian Test Comes Out Fine (Not Reproduced)](https://tools.cooconsbit.com/en/articles/claude-code-install-broken-symlink-not-reproduced-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
