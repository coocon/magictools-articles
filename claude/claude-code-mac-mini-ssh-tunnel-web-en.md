# Turn a Home Mac mini Into an Always-On Claude Code Workstation: claudecodeui + SSH Reverse Tunnel, Take Over Sessions From Any Browser

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-mac-mini-ssh-tunnel-web-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-mac-mini-ssh-tunnel-web-en?utm_source=github&utm_medium=referral)**

The Mac mini at home stays on around the clock: Claude Code manages the workspaces of five or six projects on it, and long tasks routinely run for tens of minutes. The problem is that I'm not always home — when I'm out, how do I check where a task is, approve a permission request, or hand it a new job?

The goal is specific: **from any device with a browser**, open a URL, log in, and see and operate the Claude Code sessions on the Mac mini. No client install, no VPN — it has to work on an office computer and on a phone.

This setup has been live for a week and is in daily use. The article follows the real decision order: two selection rounds first (which web UI, which transport), then the full working configuration, then actual screenshots and operating numbers, and finally the pitfalls already hit.

## Selection Round One: Which Web UI

There are four ways to "operate Claude Code in a browser." I ruled them out one by one until the last:

**The official web version (claude.ai/code)**: eliminated first. It runs in Anthropic's cloud sandbox and operates on a copy of the code inside that sandbox — whereas what I need is to operate the **local workspaces on the Mac mini**: local repos, local `.env` files, local CLIs that are already logged in. They are simply not the same thing.

**ttyd + tmux (terminal-in-a-browser)**: architecturally entirely feasible — hang a tmux session on a web page via ttyd and you have a remote terminal. But five minutes of using it on a phone tells you where the problem is: typing into a terminal on a touchscreen is torture — arrow keys, Ctrl combos, scrolling back through output, every one of them is awkward. What it gives you is "a remote screen," not "an interface designed for mobile."

**code-server (VS Code in the browser)**: works, but roundabout. It is fundamentally an editor, and Claude Code still has to run inside its integrated terminal — so you're wrapping a heavy shell (memory measured in GB) around a worse terminal experience, and the mobile layout is basically unusable.

**claudecodeui (open source, 13.5k stars)**: the final choice. The decisive point is its data model —

> claudecodeui **does not maintain its own session state** — it reads the session JSONL files under `~/.claude/projects/` directly. That means a Claude Code session you start in the terminal is **the same one** you see on the web; conversely, a session started from the web is right there when you get back to the computer and run `claude --resume`. The web UI is just another view of the same data — there is no separate ledger of "web sessions" versus "terminal sessions."

...

---

**[👉 Continue reading: Turn a Home Mac mini Into an Always-On Claude Code Workstation: claudecodeui + SSH Reverse Tunnel, Take Over Sessions From Any Browser](https://tools.cooconsbit.com/en/articles/claude-code-mac-mini-ssh-tunnel-web-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
