# GA4 Data API 代理环境超时 DEADLINE_EXCEEDED：gRPC 不认大写 HTTPS_PROXY

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/ga4-data-api-grpc-proxy-deadline-exceeded?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/ga4-data-api-grpc-proxy-deadline-exceeded?utm_source=github&utm_medium=referral)**

## 一、问题描述

在本地跑一个用 `@google-analytics/data` SDK 拉 GA4 报表的 Node.js 脚本，等了整整一分钟后抛出：

```
❌ 4 DEADLINE_EXCEEDED: Deadline exceeded after 59.999s,name resolution: 0.016s,Waiting for LB pick
```

诡异的地方在于：

- 终端里明明已经 `export HTTPS_PROXY=http://127.0.0.1:7897`
- 用 curl 走同一个代理访问 Google 完全正常：`curl -x http://127.0.0.1:7897 https://analyticsdata.googleapis.com` 秒回
- Service Account 的 JSON key 没动过，之前在别的脚本里验证过有效

网络通、凭证对、代理在跑，但 SDK 就是连不上。

## 二、环境

| 项目 | 详情 |
|------|------|
| OS | macOS（Darwin 23.6） |
| Node.js | 22.18.0 |
| SDK | @google-analytics/data v4 |
| 代理 | 本机 Clash 混合端口 127.0.0.1:7897 |
| 网络环境 | 国内，访问 Google API 必须走代理 |

## 三、排查过程

**第一步：排除凭证问题。** 如果是 Service Account 或权限问题，Google 会返回 401 / 403，而不是干等 60 秒超时。`DEADLINE_EXCEEDED` 是典型的"请求根本没到对端"——凭证嫌疑排除。

**第二步：排除代理本身。** `curl -x http://127.0.0.1:7897` 直连 `analyticsdata.googleapis.com` 正常返回，代理链路没问题。

**第三步：盯着报错措辞看。** 真正的线索藏在报错的后半句：

```
Waiting for LB pick
```

"LB pick"（负载均衡器选路）不是 HTTP 世界的词，这是 **gRPC channel 的内部状态**。也就是说，`@google-analytics/data` 底层压根不走 HTTP/REST，而是走 gRPC——我设置的 `HTTPS_PROXY` 是给 HTTP 客户端看的，gRPC 栈可能根本没读它。

...

---

**[👉 继续阅读全文：GA4 Data API 代理环境超时 DEADLINE_EXCEEDED：gRPC 不认大写 HTTPS_PROXY](https://tools.cooconsbit.com/zh/articles/ga4-data-api-grpc-proxy-deadline-exceeded?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
