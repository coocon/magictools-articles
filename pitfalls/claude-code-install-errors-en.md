# Claude Code install errors, reproduced: EACCES, a 600s mirror stall, Node 20 silently getting an old version, a region-block install.sh, and the native installer removing your npm copy

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-install-errors-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-install-errors-en?utm_source=github&utm_medium=referral)**

People searching "claude code install" rarely lack a tutorial. What they lack is **an explanation of the error they just hit**. The install steps are already covered in the [Claude Code quickstart](/en/articles/claude-code-quickstart-guide-en), so this article skips them. Instead I reproduced every install failure I could on a real machine and recorded the verbatim error, trigger, wall time and exit code for each, followed by the fix.

The three that surprised me most:

1. **Node 20 does not fail**. It **silently installs 2.1.197**. Starting with 2.1.198 the package requires `node >=22`, so npm quietly picks the newest version whose engines still match, with no warning at all.
2. **`curl -fsSL https://claude.ai/install.sh` from a blocked region exits 0** and saves a 447,830-byte "App unavailable in region" HTML page. Pipe it into `bash` and all you see is `syntax error near unexpected token '<'`.
3. **The native installer runs `npm uninstall -g @anthropic-ai/claude-code`** and prints nothing about it. During this test it really did remove the global install on my machine; the full story is in "Gotchas" below.

![A fake npm logged the native installer's call: npm uninstall -g @anthropic-ai/claude-code](https://cdn.tools.cooconsbit.com/uploads/articles/2026-09-29-cc-install/05-native-installer-npm-uninstall-en.png)

## Background

Claude Code currently has two official install paths:

- **npm**: `npm install -g @anthropic-ai/claude-code`. The package is now just a shell. The real program comes from a per-platform optionalDependency (for example `@anthropic-ai/claude-code-darwin-arm64`, a 98,952,459-byte tarball), and the postinstall script `install.cjs` hard-links that native binary to `bin/claude.exe`. **The `claude` command is no longer JavaScript.** It is a Mach-O executable (226,563,088 bytes for 2.1.284).
- **Native installer**: `curl -fsSL https://claude.ai/install.sh | bash`. The script downloads a binary from `downloads.claude.ai`, verifies its sha256, runs `claude install`, and ends up at `~/.local/bin/claude`.

Each path fails in its own ways. On a network in mainland China, several of those errors **look entirely unrelated to their real cause**.

## Analysis

Here is where a Claude Code install can break:

| Stage | What goes wrong | Rows below |
|---|---|---|
| Writing the global prefix | No write permission | #1 |
| npm cache | Cache not writable | #2 |
| Registry | Unreachable, DNS fails, slow mirror | #3 #4 #6 |
| Proxy | npmrc and env vars disagree | #5 |
| Node version | engines gate | #7 #8 |
| Native installer | Region block, download host unreachable, replaces the npm copy | #9 #10 #12 |
| After install | PATH, postinstall skipped, wrong way to launch | #11 #13 #14 #15 |

...

---

**[👉 Continue reading: Claude Code install errors, reproduced: EACCES, a 600s mirror stall, Node 20 silently getting an old version, a region-block install.sh, and the native installer removing your npm copy](https://tools.cooconsbit.com/en/articles/claude-code-install-errors-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
