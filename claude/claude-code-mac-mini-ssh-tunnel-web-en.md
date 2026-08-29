# Turn a Home Mac mini Into an Always-On Claude Code Workstation: SSH Reverse Tunnel to a VPS, Manageable From Any Browser — and Why I Don't Use Tailscale

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-mac-mini-ssh-tunnel-web-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-mac-mini-ssh-tunnel-web-en?utm_source=github&utm_medium=referral)**

The Mac mini at home stays on around the clock: Claude Code runs long tasks on it and manages the workspaces of several projects. The problem is that I'm not always home — and when I'm out, I still want to check how a task is going or hand it a new one.

The goal is specific: **from any device with a browser** — phone, iPad, an office computer — open a URL, enter a password, and see and operate Claude Code on the Mac mini. No client install, no VPN.

This setup has been running stably for months, and it comes down to three parts: a local web management UI, one SSH reverse tunnel, and nginx on a VPS. This article covers the architecture and the "why not Tailscale" question first, then the full step-by-step.

## The architecture: a three-leg relay

```
[Mac mini, home]                     [VPS, public]                 [any browser]
Claude Code Web UI          SSH reverse tunnel   nginx (TLS+auth)
127.0.0.1:3008    ────────►  127.0.0.1:18080  ────────►  https://code.example.com
(loopback only)     ssh -R      (loopback only)    reverse proxy
```

Three deliberate design choices:

1. **The web UI on the Mac mini listens on `127.0.0.1` only.** Other devices on the home LAN can't reach it, let alone the internet.
2. **The SSH reverse tunnel is dialed out by the Mac mini** to the VPS, where it opens a port (18080) on the VPS's loopback address. No public IP at home, carrier-grade NAT, a locked-down ISP router — none of it matters, because the connection goes from the inside out.
3. **That tunnel port on the VPS also binds loopback only.** The single public entrance is nginx: TLS, a layer of Basic Auth, and the app's own login. Three doors.

The web UI itself is interchangeable. I use the open-source claudecodeui (a web management interface for Claude Code — sessions, new tasks, a terminal), but ttyd + tmux works with the identical architecture — the tunnel just forwards one local port.

## Why an SSH reverse tunnel instead of Tailscale?

This is the question I get most. Tailscale (and self-hosted Headscale) is the textbook answer for "reach a machine at home" — but for **this specific requirement**, a browser-reachable web entrance, the SSH tunnel wins on four dimensions.

### 1. Zero install on the accessing side — this one is decisive

Tailscale's model is "devices join a network": every device that wants access must install the client and log in to the same account. Your phone can. Your iPad can. **The office computer often can't**, and a borrowed device certainly won't.

...

---

**[👉 Continue reading: Turn a Home Mac mini Into an Always-On Claude Code Workstation: SSH Reverse Tunnel to a VPS, Manageable From Any Browser — and Why I Don't Use Tailscale](https://tools.cooconsbit.com/en/articles/claude-code-mac-mini-ssh-tunnel-web-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
