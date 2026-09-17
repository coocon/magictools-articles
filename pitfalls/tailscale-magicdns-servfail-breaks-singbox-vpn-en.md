# Service Up, Ports Open, Certs Valid, VPN Dead for 4 Hours: Tailscale Took Over DNS and Left the Proxy Box With No Upstream

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/tailscale-magicdns-servfail-breaks-singbox-vpn-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/tailscale-magicdns-servfail-breaks-singbox-vpn-en?utm_source=github&utm_medium=referral)**

## Symptoms

Two DMIT VPSes in the same Los Angeles datacenter, same setup, both running sing-box as proxy nodes (VLESS-REALITY on tcp/443, Hysteria2 on udp/443). On the evening of September 16 both nodes on the primary box (call it dmit-usa) started timing out in every client. The second box (dmit-usa-eb) kept working.

The first round of checks came back all green:

```bash
$ ssh dmit-usa 'systemctl is-active sing-box nginx; ss -lntup | grep -E ":443 |:8444|:8445"'
active
active
udp   UNCONN 0 0   *:443            *:*   users:(("sing-box",...))
tcp   LISTEN 0 4096 127.0.0.1:8444  ...   users:(("sing-box",...))
tcp   LISTEN 0 511  127.0.0.1:8445  ...   users:(("nginx",...))
tcp   LISTEN 0 511  0.0.0.0:443     ...   users:(("nginx",...))
```

`nginx -t` passed. The Let's Encrypt certificate used by Hysteria2 had 81 days left. The last sing-box restart in systemd was two days earlier, part of a certificate cleanup, with no abnormal exits since.

The only thing out of place was the log directory:

```
-rw-r--r-- 1 sing-box sing-box 13710190 Sep 16 23:15 sing-box.log      # today, 13 MB
-rw-r--r-- 1 sing-box sing-box  7329135 Sep 16 00:30 sing-box.log.1    # yesterday, 7 MB
-rw-r--r-- 1 sing-box sing-box    50237 Sep  7 00:15 sing-box.log.10.gz # a normal day, tens of KB compressed
```

Tens of kilobytes a day normally, more than ten megabytes a day for the last two. **The service was not dead. It was spamming something.**

## Diagnosis: three commands from "the log got big" to the root cause

### Step 1: what is it spamming?

Normalize and count the error lines in the last 2 MB of the log:

```bash
tail -c 2000000 /worker/logs/sing-box/sing-box.log \
  | grep -oE "(ERROR|failed|handshake)[^\"]{0,80}" \
  | sed -E 's/[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+(:[0-9]+)?//g' \
  | sort | uniq -c | sort -rn | head
```

```
5307 ERROR  inbound/vless[vless-in]: process connection from
2826 handshake: REALITY: failed to dial dest: lookup www.apple.com: (exchange6: SERVFAIL
2499 handshake: REALITY: failed to dial dest: lookup www.apple.com: (exchange4: SERVFAIL
  40 ERROR  connection: open connection to oauthaccountmanager.googleapis.com:443 ...
```

Two things here:

- The REALITY handshake fails because `lookup www.apple.com` returns **SERVFAIL**. REALITY works by having the server actually dial the borrowed target site (www.apple.com here) during the handshake and use its TLS handshake as camouflage. If the target cannot be resolved, the handshake cannot proceed, and every VLESS connection is rejected.
- The `open connection to xxx:443` failures further down are Hysteria2. The tunnel is already up, but outbound connections to the destination hostnames fail to resolve in exactly the same way. That is why both protocols broke at once and in the same way: clients could reach the port, but not a single page would load.

...

---

**[👉 Continue reading: Service Up, Ports Open, Certs Valid, VPN Dead for 4 Hours: Tailscale Took Over DNS and Left the Proxy Box With No Upstream](https://tools.cooconsbit.com/en/articles/tailscale-magicdns-servfail-breaks-singbox-vpn-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
