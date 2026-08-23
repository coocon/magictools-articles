# MCP Server 把 CPU 吃满 100%：一次 Cloudflare MCP 失控进程的排查与止血

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/mcp-stdio-process-cpu-100-pitfall?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/mcp-stdio-process-cpu-100-pitfall?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：MCP Server 把 CPU 吃满 100%：一次 Cloudflare MCP 失控进程的排查与止血](https://tools.cooconsbit.com/zh/articles/mcp-stdio-process-cpu-100-pitfall?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
