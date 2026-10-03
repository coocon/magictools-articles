# Claude Code MCP 显示 Connected 却 0 个工具：Invalid result for tools/list（ttlMs / cacheScope）实测与修法

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-mcp-tools-list-ttlms-cachescope?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-mcp-tools-list-ttlms-cachescope?utm_source=github&utm_medium=referral)**

## 问题背景

MCP server 配好之后，`claude mcp list` 不是 `✘ Failed to connect`，而是一个黄色感叹号：

```
roblox-like: /tmp/mcp-ttl/roblox-like-server  - ! Connected · tools fetch failed — Invalid result for tools/list: [ { "expected": "number", "code": "invalid_type", "path": [ "ttlMs" ], "message": "Invalid input: expected number, received undefined" }, { "code": "invalid_value", "values": [ "public", "private" ], "path": [ "cacheScope" ], "message": "Invalid option: expected one of \"public\"|\"private\"" } ]
```

会话里的表现是：server 状态显示 `connected`，但模型一个该 server 的工具都看不到。重启、新开会话、重启电脑都没用。

GitHub 上对应的是 [anthropics/claude-code#97319](https://github.com/anthropics/claude-code/issues/97319)（2026-09-26 提交，到 10-03 仍是 OPEN），触发者是 Roblox Studio 官方的 MCP bridge。报告人的判断是「tools/list 响应里带了新版协议的额外字段 `ttlMs` / `cacheScope`，Claude Code 的校验太严，把整个响应拒掉了」，评论区给出的两个绕过办法是「降级到 2.1.280」和「设 `MCP_PROTOCOL_NEGOTIATION=legacy`」。

这篇用自写的 stub server 把这条报错拆开：到底是字段多了还是少了，为什么同一个版本有人中招有人没事，降级为什么能好，以及用户侧和 server 侧各该怎么修。所有报错原文都来自 2026-10-03 在本机的真实运行。

## 问题分析

先看规范。MCP 的 [2026-07-28 版 schema](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/schema/2026-07-28/schema.ts) 里，`ListToolsResult` 同时继承了 `PaginatedResult` 和 `CacheableResult`：

```ts
export interface CacheableResult extends Result {
  ttlMs: number;                     // 客户端可缓存多少毫秒，0 表示立即过期
  cacheScope: "public" | "private";  // 类似 HTTP Cache-Control 的 public / private
}

export interface Result {
  _meta?: ResultMetaObject;
  resultType: ResultType;            // "complete" | "input_required" | string，本版起必填
  [key: string]: unknown;            // 允许任意额外字段
}
```

这里有两个关键点：

1. `ttlMs`、`cacheScope`、`resultType` 在这一版里都是**必填**。
2. `Result` 带 `[key: string]: unknown`，规范明确**允许额外字段**。

再看报错本身：`"expected": "number"` + `invalid_type` 指向 `ttlMs`，`invalid_value` + `["public","private"]` 指向 `cacheScope`。字段如果是「多出来的未知字段」，校验器不会知道它该是 number、该在 public/private 里选。只有在**按 2026-07-28 的 schema 校验、而 server 没给这两个字段**时，才会是这个形状。2.1.288 的措辞更直白：`expected number, received undefined`。

所以要验证的假设是：**server 和 Claude Code 协商到了 2026-07-28，但 server 的 tools/list 仍按旧版协议回包，漏了新版的必填字段。** 还要回答两个问题：stdio server 什么时候会协商到 2026-07-28？降级到 2.1.280 为什么能好？

## 技术方案与选型

- **被测对象**：Claude Code 2.1.280（issue 评论里「最后一个能用的版本」）、2.1.285（本机日常版本）、2.1.288（10-02 发布的最新版）。2.1.280 和 2.1.288 用 `npm install` 装在隔离目录，不碰全局安装。
- **MCP server**：自写 Node stdio stub，约 70 行，逐行记录收到的 JSON-RPC。用环境变量切换各种回包：`initialize` 回什么协议版本、是否实现 `server/discover`、tools/list 里给不给 `resultType` / `ttlMs` / `cacheScope`、给错类型、加未知字段。
- **模型侧**：本地假的 Messages API（固定回一句 `STUB_OK`），同时充当 `HTTPS_PROXY`，记录所有 CONNECT 并回 403。整个实验零真实 API 调用：stub 共记录到 97 次 `CONNECT api.anthropic.com:443`，全部被拦。
- **读数**：`--output-format stream-json --verbose` 第一行 `init` 里的 `mcp_servers[].status` 和 `tools` 里有没有 `mcp__ttlstub__ping`，加上 `--debug-file` 里该 server 的日志。每个 case 用独立的 `HOME`，cwd 放在仓库外。

...

---

**[👉 继续阅读全文：Claude Code MCP 显示 Connected 却 0 个工具：Invalid result for tools/list（ttlMs / cacheScope）实测与修法](https://tools.cooconsbit.com/zh/articles/claude-code-mcp-tools-list-ttlms-cachescope?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
