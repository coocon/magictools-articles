# Node.js 内置 fetch 不走 HTTPS_PROXY：Google API 连不上的第二个坑

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/nodejs-fetch-ignores-https-proxy?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/nodejs-fetch-ignores-https-proxy?utm_source=github&utm_medium=referral)**

## 一、问题描述

前一篇《GA4 Data API 代理环境超时 DEADLINE_EXCEEDED》修好了 gRPC 的代理，我照着同样的环境变量思路给 Google Search Console 写数据拉取脚本——GSC 是普通 REST API，用内置 fetch 就够了：

```js
const res = await fetch(
  "https://searchconsole.googleapis.com/webmasters/v3/sites",
  { headers: { Authorization: `Bearer ${token}` } }
);
```

跑起来的表现：

```
TypeError: fetch failed
  cause: ConnectTimeoutError: Connect Timeout Error
```

而同一个终端里：

- `curl -x http://127.0.0.1:7897 https://searchconsole.googleapis.com` 正常
- `HTTPS_PROXY` 大写小写都 export 了
- 上一篇修 GA4 的 `grpc_proxy` 也还在

环境变量摆满一桌子，fetch 一个都不看。

## 二、环境

| 项目 | 详情 |
|------|------|
| OS | macOS（Darwin 23.6） |
| Node.js | 22.18.0（内置 fetch 由 undici 提供） |
| 目标 API | Google Search Console（REST） |
| 代理 | 本机 Clash 混合端口 127.0.0.1:7897 |

## 三、根因：这不是 bug，是设计

Node.js 从 v18 开始内置的 `fetch` 底层是 **undici**。undici 的默认行为是：**完全忽略 `HTTP_PROXY` / `HTTPS_PROXY` / `NO_PROXY` 等环境变量**，需要代理必须在代码里显式配置 dispatcher。

这和大多数人的直觉相反，因为几乎所有常用工具都读这些变量：

| 工具 / 库 | 读环境代理变量吗 |
|-----------|-----------------|
| curl / wget | ✅ 读 |
| axios | ✅ 读 |
| gaxios（googleapis 全家桶的 HTTP 层） | ✅ 读大写 `HTTPS_PROXY` 和小写 `https_proxy` |
| request / node-fetch（配 agent 库） | 生态方案普遍支持 |
| **Node 内置 fetch（undici）** | ❌ 默认不读 |

...

---

**[👉 继续阅读全文：Node.js 内置 fetch 不走 HTTPS_PROXY：Google API 连不上的第二个坑](https://tools.cooconsbit.com/zh/articles/nodejs-fetch-ignores-https-proxy?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
