---
title: "MCP Server 把 CPU 吃满 100%：一次 Cloudflare MCP 失控进程的排查与止血"
slug: mcp-stdio-process-cpu-100-pitfall
category: pitfalls
locale: zh
tags: [MCP, Claude Code, 性能排查, 踩坑实录]
summary: "Mac 风扇狂转、CPU 100%，定位到元凶是 Cloudflare 的 stdio MCP server：两个进程各吃满一个核，背后还堆着一排历史会话残留的僵尸实例。本文复盘现象、定位过程、stdio MCP 的进程模型缺陷，以及为什么低频运维操作用 REST API 比挂一个常驻 MCP 更合理。"
status: published
source: authored
---

日常写代码时突然发现机器风扇狂转，`top` 一看 CPU 被吃满。顺藤摸瓜定位到的元凶有点意外：不是编译任务，不是浏览器，而是挂在 Claude Code 上的 **Cloudflare MCP server**。

这篇文章复盘完整的排查过程、stdio 型 MCP 的进程模型问题，以及最后的处置决策：**移除 MCP，改用 Cloudflare REST API**。

## 现象：两个进程各吃满一个核，后面还排着一队僵尸

第一步先看进程。按 CPU 排序：

```bash
ps aux | grep -i mcp | grep -v grep
```

输出（已脱敏）：

```
bjhl  56484  99.8  node .../node_modules/.bin/mcp-server-cloudflare run <account-id>
bjhl  56483  99.1  node .../node_modules/.bin/mcp-server-cloudflare run <account-id>
bjhl  77253   0.0  node .../node_modules/.bin/mcp-server-cloudflare run <account-id>
bjhl  77075   0.0  npm exec @cloudflare/mcp-server-cloudflare run <account-id>
bjhl  56199   0.0  npm exec @cloudflare/mcp-server-cloudflare run <account-id>
bjhl  56197   0.0  npm exec @cloudflare/mcp-server-cloudflare run <account-id>
bjhl  39408   0.0  node .../node_modules/.bin/mcp-server-cloudflare run <account-id>
bjhl  39332   0.0  npm exec @cloudflare/mcp-server-cloudflare run <account-id>
```

三个信息量很大的细节：

1. **两个实例各吃满约 100% CPU**（56483 / 56484）——不是内存泄漏，是忙循环（busy loop），单核跑满。
2. **同一个 MCP server 同时存在 7~8 个进程**。PID 跨度很大（39xxx / 56xxx / 77xxx），说明它们来自不同时间启动的多个会话，旧的没退干净，新的又起来了。
3. 每个实例实际是**一对进程**：`npm exec` 父进程 + `node` 子进程——因为配置里用的是 `npx -y @cloudflare/mcp-server-cloudflare` 这种启动方式，每次拉起都要走一遍 npm 的解析链。

## 为什么会这样：stdio MCP 的进程模型

用 `claude mcp get cloudflare` 确认配置：

```
cloudflare:
  Scope: User config (available in all your projects)
  Status: ✔ Connected
```

这是一个 **user 级作用域的 stdio 型 MCP**——意味着**每一个** Claude Code 会话启动时都会 spawn 一个属于自己的 server 进程。开三个终端窗口就是三套进程，这就解释了为什么会堆出一排实例。

stdio MCP 的生命周期约定是：客户端退出 → 子进程的 stdin 收到 EOF → server 自行退出。但这个约定依赖 server 正确处理 EOF。从现场看，两个吃满 CPU 的实例最可能的情况是（这是基于现场的推断，我没有进一步 profile 它的内部代码）：**父会话已经不在了，server 卡在一个不处理流关闭的读循环里**——read 立即返回、循环立即重试，于是单核 100%，还永远不退出。

而另外几个 0% CPU 的残留实例，虽然安静，但也是同一个回收缺陷的产物——只是它们碰巧闲着，没进入忙循环。

归纳一下，这次踩的坑其实是三层问题叠加：

| 层 | 问题 |
|----|------|
| 使用方式 | 低频使用的运维类工具，被配置成 user 级常驻——每个会话都付出一套进程的成本 |
| 启动链路 | `npx` 启动使进程翻倍（npm exec + node），且首次启动要走网络解析 |
| server 实现 | 会话结束后进程不退出，甚至进入忙循环 |

## 止血：移除配置 + 清理进程

处置很直接。先从配置里摘掉：

```bash
claude mcp remove cloudflare -s user
# Removed MCP server cloudflare from user config
```

再清掉所有存量进程（包括安静的僵尸）：

```bash
pkill -f "mcp-server-cloudflare"
```

复查确认：

```bash
ps aux | grep -i cloudflare | grep -v grep
# （无输出）
claude mcp list | grep -i cloudflare
# （无输出）
```

CPU 应声回落，风扇安静。

## 为什么不装回去：低频操作，API 比 MCP 更合理

移除之后要回答一个问题：以后还要操作 Cloudflare 怎么办？

答案是直接用 **Cloudflare REST API**。它的 API 设计成熟、文档完善，一条 curl 就能完成 MCP tool 背后同样的调用：

```bash
# 列出 zone
curl -s "https://api.cloudflare.com/client/v4/zones" \
  -H "Authorization: Bearer $CF_API_TOKEN" | jq '.result[].name'

# 清除某个 zone 的缓存
curl -s -X POST "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/purge_cache" \
  -H "Authorization: Bearer $CF_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{"purge_everything":true}'

# 查 DNS 记录
curl -s "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records" \
  -H "Authorization: Bearer $CF_API_TOKEN" | jq '.result[] | {name, type, content}'
```

AI 助手本来就会写 curl——让它直接调 API，和让它通过 MCP tool 调,能力上没有区别。区别在成本模型：

- **MCP（常驻）**：不管用不用，每个会话都要养一个进程；stdio 实现质量参差，回收失败就变成本文这种事故。
- **API（按需）**：不用的时候成本为零；出问题时排查路径就是一条 HTTP 请求，透明得多。

我的取舍标准沉淀成一句话：**高频、强交互、需要状态的工具才值得挂 MCP；低频运维操作，直接教 AI 调 API。** Cloudflare 的场景对我来说是后者——一个月改不了几次 DNS，为它常驻一组进程不划算。

## 可以带走的检查清单

1. **定期体检**：`claude mcp list` 看看挂了多少 server，问自己每一个的使用频率是否配得上常驻成本。
2. **风扇一响先查 MCP**：`ps aux | grep mcp`，重点看是否有多个同名实例堆积——那是会话回收失败的信号。
3. **user 级作用域慎用**：user scope 的 stdio MCP 是「每个会话 × 每个项目」都起进程；只在单个项目用的工具，配到 project scope。
4. **npx 启动的 MCP 成本更高**：进程翻倍 + 首启走网络。真的高频使用，考虑全局安装后用绝对路径启动。
5. **移除 ≠ 失去能力**：绝大多数 SaaS 的 MCP server 只是 REST API 的薄封装，curl + API token 永远是兜底路径。

## 常见问题 FAQ

### MCP server 进程占用 CPU 100% 怎么排查？

先 `ps aux | grep mcp` 按 CPU 排序找到具体进程，看命令行确认是哪个 MCP server；再数同名实例数量——多个实例堆积说明历史会话的进程没有被回收。止血手段是 `pkill -f "<server 名>"`，根治手段是评估这个 MCP 是否值得常驻，低频工具直接移除改用 API。

### 如何彻底移除 Claude Code 中的某个 MCP server？

先用 `claude mcp get <name>` 确认作用域（user / project / local），再用对应的 `claude mcp remove <name> -s <scope>` 移除配置，最后 `pkill -f` 清理仍在运行的存量进程。只删配置不清进程，失控实例会继续吃 CPU。

### 什么场景该用 MCP，什么场景该直接调 API？

高频、强交互、需要维持状态（如浏览器会话、数据库连接）的工具适合 MCP；低频的运维类操作（改 DNS、清缓存、查配置）直接让 AI 写 curl 调 REST API 更划算——零常驻成本，出问题时排查路径也更透明。

## 参考链接

- [Model Context Protocol 规范：stdio transport 生命周期](https://modelcontextprotocol.io/specification/2025-06-18/basic/transports)
- [Cloudflare API 文档](https://developers.cloudflare.com/api/)
- [Claude Code MCP 配置文档](https://docs.anthropic.com/en/docs/claude-code/mcp)
