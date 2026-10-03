# Clash Verge 开了 TUN 虚拟网卡反而上不了网：Hysteria2 流量绕回 TUN 自己，Tailscale 又把 DNS 截走了

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/clash-verge-tun-not-working-tailscale-hysteria2?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/clash-verge-tun-not-working-tailscale-hysteria2?utm_source=github&utm_medium=referral)**

## 现象

环境：Mac mini，macOS 26.3.1；Clash Verge Rev 2.5.6，内核 mihomo v1.19.31，通过「服务模式」以 root 身份运行；订阅里有 7 个自建节点（5 个 Hysteria2、2 个 VLESS-REALITY），全局模式，当前选的是 `美国-DMIT-Hysteria2`。本机同时在跑 Tailscale。

- 只开「系统代理」时，浏览器、`curl -x http://127.0.0.1:7897` 全都正常。
- 在设置里打开「虚拟网卡模式（TUN）」后，开关能打开，也没有报错，但所有网站都打不开，**连百度都打不开**。

「连国内站都打不开」这一点很关键：如果只是 DNS 被污染，国内站应该还能用。现在国内外一起断，说明经过 TUN 的流量整体都没走通。

## 排查工具：直接连 mihomo 内核的 API

Clash Verge 的 GUI 只显示了开关状态，要排查得直接问内核。服务模式下，内核把控制接口开在一个 unix socket 上，当前用户可以直接访问（`secret` 在 Clash Verge 目录的 `config.yaml` 里）：

```bash
S=/var/run/clash-verge-service/users/501/verge-mihomo.sock
A="Authorization: Bearer set-your-secret"

curl -s --unix-socket $S -H "$A" http://x/version
# {"meta":true,"version":"v1.19.31"}

# 实时日志（debug 级别），另开一个终端挂着
curl -sN --unix-socket $S -H "$A" "http://x/logs?level=debug"

# 不用点 GUI，直接开关 TUN
curl -s --unix-socket $S -H "$A" -X PATCH http://x/configs -d '{"tun":{"enable":true}}'
```

后面所有实验都用这三条命令完成：开 TUN、看日志、用 `curl --noproxy '*'` 测连通性（绕过系统代理，确保流量真的走 TUN），测完再关。

## 第一步：TUN 本身起来了吗

```bash
$ ifconfig utun5 | grep inet
	inet 198.18.0.1 --> 198.18.0.1 netmask 0xfffffffc
$ netstat -rn -f inet | grep utun5 | head -3
1                  198.18.0.1         UGSc                utun5
2/7                198.18.0.1         UGSc                utun5
4/6                198.18.0.1         UGSc                utun5
```

网卡已经创建，`auto-route` 也用 `1/8`、`2/7`、`4/6`…… 这组拆分路由把默认路由盖住了。内核日志同样正常：

```
[TUN] default interface changed by monitor, => en1
[TUN] Tun adapter listening at: utun5([198.18.0.1/30],[]), mtu: 9000, auto route: true, auto redir: false, ip stack: gVisor
```

但连通性测试全挂：

```
https://www.google.com 000 8.002101s ip=31.13.92.37
https://www.baidu.com  000 8.005689s ip=103.235.47.188
```

所以 TUN 本身没问题，问题出在「进了 TUN 之后」。

## 第二步：日志里的可疑连接——代理在代理自己

接着看 TUN 打开后的连接日志：

```
[UDP] 198.18.0.1:55638 --> 179.253.x.x:443 using GLOBAL
[UDP] 198.18.0.1:56470 --> 163.7.x.x:8488 using GLOBAL
[UDP] 198.18.0.1:54918 --> 36.50.x.x:8488 using GLOBAL
[TCP] 198.18.0.1:57612 --> 103.235.47.188:443 using GLOBAL
```

前三条的目标地址**就是 Hysteria2 节点自己**（`:443` 是 DMIT 的 Hysteria2，`:8488` 是另外两台）。本机会往这些地址发 UDP 包的，只有 mihomo 自己。源地址是 `198.18.0.1`，说明这些包是**从 TUN 网卡进来的**。

...

---

**[👉 继续阅读全文：Clash Verge 开了 TUN 虚拟网卡反而上不了网：Hysteria2 流量绕回 TUN 自己，Tailscale 又把 DNS 截走了](https://tools.cooconsbit.com/zh/articles/clash-verge-tun-not-working-tailscale-hysteria2?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
