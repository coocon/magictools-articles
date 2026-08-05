---
title: "Node.js 内置 fetch 不走 HTTPS_PROXY：Google API 连不上的第二个坑"
slug: "nodejs-fetch-ignores-https-proxy"
category: developer
locale: zh
source: authored
tags:
  - nodejs
  - fetch
  - proxy
  - undici
  - google-api
  - troubleshooting
summary: "curl 带代理能通，Node 脚本里的 fetch 却一直超时？Node.js 内置 fetch（undici）设计上就不读 HTTPS_PROXY 等环境变量。本文用 Google Search Console API 的实际排查过程讲清这个行为，并给出三种修复路径：google-auth-library 的 client.request()、undici ProxyAgent、以及新版 Node 的实验性开关。"
coverImage: ""
status: published
scheduledAt: ""
---

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

被坑的路径通常是：先用 curl 验证"代理没问题"，再写 fetch 版本，然后对着超时怀疑人生——**curl 的成功恰恰是误导项**，它和 fetch 根本不在一套代理发现机制里。

## 四、三种修复路径

### 路径一（推荐）：调 Google API 就别裸写 fetch，用 google-auth-library 的 client.request()

如果你的场景和我一样是调 Google 系 API，最顺的解法是让认证库顺便把 HTTP 层也包了：

```js
import { GoogleAuth } from "google-auth-library";

const auth = new GoogleAuth({
  scopes: ["https://www.googleapis.com/auth/webmasters.readonly"],
});
const client = await auth.getClient();

// client.request uses gaxios under the hood, which honors HTTPS_PROXY
const res = await client.request({
  url: "https://searchconsole.googleapis.com/webmasters/v3/sites",
});
```

这个方案有两个额外好处：

- **token 刷新自动处理**，不用自己拼 Authorization 头
- 如果你已经在用 `@google-analytics/data` 之类的官方 SDK，`google-auth-library` 是它们的传递依赖，**不需要新增任何安装**，也省掉了 `googleapis` 这个大包

gaxios 对代理地址的要求和 grpc-js 一样严格——值必须带 `http://` 前缀，裸写 `127.0.0.1:7897` 会抛 `Invalid URL`。

### 路径二（通用）：undici ProxyAgent

不是 Google API、就想用 fetch 的场景，显式给 undici 配 dispatcher：

```js
import { ProxyAgent, setGlobalDispatcher } from "undici";

setGlobalDispatcher(new ProxyAgent("http://127.0.0.1:7897"));

// every fetch in this process now goes through the proxy
const res = await fetch("https://example.com");
```

不想影响全局就按请求传：

```js
const res = await fetch(url, {
  dispatcher: new ProxyAgent("http://127.0.0.1:7897"),
});
```

（`dispatcher` 是 undici 扩展选项，TypeScript 下可能需要类型断言。）

### 路径三（新版 Node）：实验性环境变量开关

较新版本的 Node.js 提供了实验性的 `NODE_USE_ENV_PROXY=1`，让内置 fetch 尊重环境代理变量。如果你的 Node 版本支持且能接受实验性特性，这是改动最小的方案——但在写死 Node 22 的项目里（比如我这个），前两条路径更稳。用之前先在目标版本上实测，别把实验性开关写进生产脚本。

## 五、和上一篇合起来看：Google API 代理配置速查

同一套 Google 凭证，不同 API 的传输层完全不同，代理配置也就完全不同：

| API | 传输层 | 代理配置 |
|-----|--------|---------|
| GA4 Data API（@google-analytics/data） | gRPC | 小写 `grpc_proxy=http://...`（大写 HTTPS_PROXY 无效） |
| Search Console / 大多数 googleapis | REST（gaxios） | `HTTPS_PROXY=http://...`（大小写都认） |
| 自己裸写 fetch | undici | 环境变量全部无效，必须代码里配 ProxyAgent |

我的实际做法是脚本前缀统一带上两个变量，覆盖前两种情况：

```bash
HTTPS_PROXY=http://127.0.0.1:7897 grpc_proxy=http://127.0.0.1:7897 node script.js
```

## 六、排查口诀

- **curl 通 ≠ 你的代码会通**：curl 有自己的代理发现机制，验证不了你所用 HTTP 库的行为
- 换库先问一句：**这个库读环境代理变量吗？读哪个？什么大小写？**——直接翻 node_modules 里的源码，比查文档快且不会过时
- Node 内置 fetch 超时且环境有代理时，第一反应就该是 undici 不读环境变量，别浪费时间查 DNS 和防火墙
