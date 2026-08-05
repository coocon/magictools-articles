---
title: "GA4 Data API 代理环境超时 DEADLINE_EXCEEDED：gRPC 不认大写 HTTPS_PROXY"
slug: "ga4-data-api-grpc-proxy-deadline-exceeded"
category: developer
locale: zh
source: authored
tags:
  - ga4
  - grpc
  - proxy
  - nodejs
  - troubleshooting
summary: "GA4 Data API 在代理环境下 60 秒超时报 DEADLINE_EXCEEDED，而 curl 走同一个代理完全正常。根因是 SDK 底层走 gRPC，而 grpc-js 只读小写的 grpc_proxy / https_proxy，大写 HTTPS_PROXY 对它不可见。本文从报错现象到源码逐层排查，给出一次配对的修复方案。"
coverImage: ""
status: published
scheduledAt: ""
---

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

**第四步：翻 grpc-js 源码验证。** 打开 `node_modules/@grpc/grpc-js/build/src/http_proxy.js`，`getProxyInfo()` 的逻辑一目了然：

```js
if (process.env.grpc_proxy) {
    envVar = 'grpc_proxy';
    proxyEnv = process.env.grpc_proxy;
}
else if (process.env.https_proxy) { ... }
else if (process.env.http_proxy) { ... }
else {
    return {};   // no proxy found -> direct connection -> blocked -> 60s timeout
}
```

三个变量**全部是小写**：`grpc_proxy` → `https_proxy` → `http_proxy`，按此优先级取第一个。而类 Unix 系统的环境变量是大小写敏感的——我 export 的大写 `HTTPS_PROXY`，对 grpc-js 来说等于不存在。

## 四、根因

三层叠在一起：

1. **`@google-analytics/data` 走 gRPC 而不是 REST**。它和 `googleapis` 家族（GSC、Drive 等走 REST/gaxios）行为完全不同，不能照搬代理经验。
2. **grpc-js 只读小写的 `grpc_proxy` / `https_proxy` / `http_proxy`**，大写 `HTTPS_PROXY` 不在它的查找列表里。而 curl、gaxios 等大小写都认，这就是"curl 通但 SDK 不通"的直接原因。
3. 读不到代理配置时 grpc-js **静默直连**，不抛任何"代理未生效"的提示，直连被墙后表现为 60 秒 `DEADLINE_EXCEEDED`——报错离根因隔了十万八千里。

## 五、还有一个连环坑：代理地址必须带 http:// 前缀

修复时如果图省事写成裸地址：

```bash
grpc_proxy=127.0.0.1:7897 node report.js
```

会得到另一个报错，然后**照样 60 秒超时**：

```
cannot parse value of "grpc_proxy" env var
```

源码里 `getProxyInfo()` 用 `new URL(proxyEnv)` 解析变量值，解析失败（或 scheme 不是 `http:`）就打一行日志然后 `return {}`——效果和没设一样。所以值必须是完整 URL：

```bash
grpc_proxy=http://127.0.0.1:7897
```

## 六、修复方案

一次配齐两个变量（大写给 REST 系工具留着，小写专供 gRPC）：

```bash
HTTPS_PROXY=http://127.0.0.1:7897 \
grpc_proxy=http://127.0.0.1:7897 \
node scripts/ga-report.js
```

验证：原来卡 60 秒的请求 2 秒内返回数据。

几点取舍说明：

- 理论上只 export 小写 `https_proxy` 一个也能同时满足 grpc-js 和多数 HTTP 客户端，但**显式设 `grpc_proxy` 意图更清晰**（它在 grpc-js 里优先级最高），也不会影响其他不该走代理的流量。
- 不建议直接写进 shell 配置文件全局生效——国内 API（如果你的脚本还调别的服务）被误导流到境外代理反而会挂。按脚本粒度前缀注入最稳。
- 如果你在 `package.json` 的 npm script 里跑，记得代理变量要写在 `npm run` 之前，而不是 script 内容里。

## 七、延伸：同一套凭证，GSC 脚本又是另一个坑

修好 GA4 后，我照着同样的思路给 Google Search Console 写数据脚本，结果又连不上——因为 GSC API 走的是 REST，而 Node.js 内置的 fetch **不读任何代理环境变量**，和这次的坑方向刚好相反。那是另一篇文章的内容：《Node.js 内置 fetch 不走 HTTPS_PROXY：Google API 连不上的第二个坑》。

## 八、排查口诀

- 报错里出现 **LB pick / channel** 这类词 → 对方是 gRPC，别按 HTTP 思路查
- **curl 通但 SDK 不通** → 十有八九是环境变量没被 SDK 读到，翻源码确认它读哪个变量、什么大小写
- 代理类环境变量的值**永远写完整 URL**（带 `http://`），裸 host:port 在不同库里的解析行为完全不可预测
