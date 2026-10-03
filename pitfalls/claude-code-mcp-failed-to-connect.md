# Claude Code MCP server Failed to connect 怎么排查：CONNECTION_CLOSED、connection timed out after 30000ms、ENOENT 逐条实测

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-mcp-failed-to-connect?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-mcp-failed-to-connect?utm_source=github&utm_medium=referral)**

## 问题背景

给 Claude Code 挂上一个 MCP server 之后，`claude mcp list` 可能会显示这样一行：

```
✘ Failed to connect — CONNECTION_CLOSED: Connection closed
```

也可能是 `ENOENT`、`connection timed out after 30000ms`、`ECONNREFUSED`，或者 `claude -p` 在启动时直接报 `Error: Invalid MCP configuration:`。这些报错都在说同一件事：「连不上」。但「连不上」本身不是原因，没法照着修。

这篇把「连不上」拆回成一个个可判定的具体原因：每种原因各自产生什么报错原文，看到报错后怎么在一分钟内判断是哪一种、怎么修。所有报错原文都来自 2026-09-24 在本机的真实运行，逐字照抄。

## 问题分析

MCP server 从配置到可用要走四步，每一步失败的报错都不一样：

1. **读配置**：`--mcp-config` 文件能不能找到、JSON 是否合法、结构对不对
2. **起进程 / 建连接**：stdio server 要 spawn 一个命令（命令找不到就是 ENOENT）；http/sse server 要连 URL（端口上没服务就是 ECONNREFUSED）
3. **进程活下来**：进程起来后如果立刻退出，Claude Code 只能看到管道断了，也就是 `CONNECTION_CLOSED`
4. **MCP 握手**：进程活着，但必须在超时时间内回应 `initialize`，否则就是 `CONNECT_TIMEOUT`

难排查的是第 3 步和第 4 步：报错只说明失败发生在哪一步，不说明为什么失败。实验里最重要的发现也集中在这两步。

## 技术方案与选型

**被测 server 全部自己写**，放在实验目录的 `stubs/` 下，共三个：

- `good-server.mjs`：零依赖的最小 MCP stdio server，17 行，只实现 `initialize` / `tools/list` / `tools/call`，唯一的工具 `echo` 返回 `LAB-ECHO:<text>`
- `exit1-server.sh`：往 stderr 打一行 `lab-exit1: fatal: missing LAB_API_KEY, refusing to start` 后 `exit 1`
- `hang-server.mjs`：往 stderr 打一行后什么也不回，300 秒后自己退出

**排除项：为什么不装第三方 MCP server 来测。** 真实 server 失败时掺着各自的业务逻辑、网络和依赖版本，报错属于哪一层说不清楚；而且 `npx -y` 会联网下载，实验无法复现，也有供应链风险。stub 让每个 case 只改变一个变量，`LAB-ECHO:` 前缀还能证明工具结果确实来自这个 server，不是模型编的。

**排除项：为什么不在自己的真实配置上测。** 本机 `~/.claude.json` 里配了真实 server，一旦混进来就分不清是谁在报错，改坏了还会影响日常使用。所以每个 case 各用一个空的 `CLAUDE_CONFIG_DIR`（user 级 server 写在这个临时目录的 `.claude.json` 里）；`claude -p` 一律加 `--mcp-config <临时 json> --strict-mcp-config`，只有验证 strict 作用的那一次故意不加。

**两种观测方式**：

- `claude mcp list` 用来看健康检查原文，不花钱，可以随便跑。每次都加 `--debug-file` 保存完整日志，并记录退出码和墙钟
- `claude -p` 用来看模型侧的影响，总共只跑 8 次。每次都加 `< /dev/null`（不加会先白等 3 秒 stdin，延迟读数就废了），用 `--output-format stream-json --verbose` 同时拿到 init 里的 server 状态、工具列表，以及 result 里的 `duration_ms` / `num_turns` / `total_cost_usd`

环境：Claude Code 2.1.280（Mach-O 原生二进制）、macOS 26.3.1、Node v25.9.0；`-p` 实际使用的模型是 `claude-opus-5-5[1m]`，走第三方 `ANTHROPIC_BASE_URL`。

## 实测过程

### Case 1：命令不存在

stdio server 的 `command` 指向 `/nonexistent/mcp-server`。`claude mcp list`（退出码 0，墙钟 0.16s）：

```
Checking MCP server health…

ghost: /nonexistent/mcp-server  - ✘ Failed to connect — ENOENT: ENOENT: no such file or directory, posix_spawn '/nonexistent/mcp-server'
```

在 `claude -p` 里，这个 server 和其它坏 server 放在同一组跑（见 case 7）。stderr 是空的，init 里只显示 `'status': 'failed'`；debug 日志里是：

```
2026-09-24T02:06:21.642Z [DEBUG] MCP server "ghost": Connection failed after 3ms (ENOENT): ENOENT: no such file or directory, posix_spawn 'stdio'
```

...

---

**[👉 继续阅读全文：Claude Code MCP server Failed to connect 怎么排查：CONNECTION_CLOSED、connection timed out after 30000ms、ENOENT 逐条实测](https://tools.cooconsbit.com/zh/articles/claude-code-mcp-failed-to-connect?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
