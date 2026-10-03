# Clash Verge TUN Mode Breaks All Internet Access: Hysteria2 Traffic Loops Back Into the TUN, and Tailscale Hijacks DNS

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/clash-verge-tun-not-working-tailscale-hysteria2-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/clash-verge-tun-not-working-tailscale-hysteria2-en?utm_source=github&utm_medium=referral)**

## Symptoms

Setup: Mac mini, macOS 26.3.1; Clash Verge Rev 2.5.6 with the mihomo v1.19.31 core, running as root through "service mode". The subscription has 7 self-hosted nodes (5 Hysteria2, 2 VLESS-REALITY), global mode, with `美国-DMIT-Hysteria2` (a US Hysteria2 node on DMIT) selected. Tailscale is also running on this machine.

- With only "system proxy" on, the browser and `curl -x http://127.0.0.1:7897` work fine.
- After turning on "virtual network adapter mode (TUN)" in settings, the switch turns on without any error, but no website loads, **not even Baidu**.

"Even Chinese sites fail" is the key detail. If DNS poisoning were the only problem, domestic sites would still work. With both domestic and foreign sites down, nothing going through the TUN is getting out.

## Debugging tool: talk to the mihomo core's API directly

The Clash Verge GUI only shows switch states, so debugging means asking the core directly. In service mode, the core exposes its control interface on a unix socket that the current user can access (the `secret` is in `config.yaml` in the Clash Verge directory):

```bash
S=/var/run/clash-verge-service/users/501/verge-mihomo.sock
A="Authorization: Bearer set-your-secret"

curl -s --unix-socket $S -H "$A" http://x/version
# {"meta":true,"version":"v1.19.31"}

# Live logs at debug level; keep this running in another terminal
curl -sN --unix-socket $S -H "$A" "http://x/logs?level=debug"

# Toggle TUN without touching the GUI
curl -s --unix-socket $S -H "$A" -X PATCH http://x/configs -d '{"tun":{"enable":true}}'
```

Every experiment below uses these three commands: turn TUN on, watch the log, test connectivity with `curl --noproxy '*'` (bypassing the system proxy so the traffic really goes through the TUN), then turn it off again.

## Step 1: is the TUN itself up?

```bash
$ ifconfig utun5 | grep inet
	inet 198.18.0.1 --> 198.18.0.1 netmask 0xfffffffc
$ netstat -rn -f inet | grep utun5 | head -3
1                  198.18.0.1         UGSc                utun5
2/7                198.18.0.1         UGSc                utun5
4/6                198.18.0.1         UGSc                utun5
```

The interface exists, and `auto-route` has covered the default route with split routes (`1/8`, `2/7`, `4/6`…). The core log looks normal too:

```
[TUN] default interface changed by monitor, => en1
[TUN] Tun adapter listening at: utun5([198.18.0.1/30],[]), mtu: 9000, auto route: true, auto redir: false, ip stack: gVisor
```

But every connectivity test fails:

...

---

**[👉 Continue reading: Clash Verge TUN Mode Breaks All Internet Access: Hysteria2 Traffic Loops Back Into the TUN, and Tailscale Hijacks DNS](https://tools.cooconsbit.com/en/articles/clash-verge-tun-not-working-tailscale-hysteria2-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
