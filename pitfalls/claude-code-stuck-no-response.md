# Claude Code 一直卡住、转圈没反应怎么办：后端不回时它要等 6 分钟，重试满 10 次可能卡一个多小时（实测）

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-stuck-no-response?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-stuck-no-response?utm_source=github&utm_medium=referral)**

> **先回答你搜的问题**
>
> - **转圈不动、计时一直在涨（`Clauding… (4m 34s)`）**：Claude Code 在等后端回数据，后端一个字节都没回。用 API key 加 `ANTHROPIC_BASE_URL`（中转站、网关）时，**它每次要等满 6 分钟才判定超时**。
> - **停在 `Retrying in 0s · attempt 1/10` 不动**：上一次已经超时，这是重发的请求又在等 6 分钟，不是死机。
> - **会卡多久**：默认重试 10 次，后端一直不回的话，实测要约 69 分钟（4147.78 秒）才最终报 `Request timed out`。
> - **马上能做的**：按 `Esc` 中断，再发一次。反复出现就是后端或网络的问题，先单独用 curl 测一下你的 base URL 通不通（命令见下文「实践效果」第 1 条）。
> - **想让它早点放弃**：设 `API_TIMEOUT_MS=60000`（只能调小，调大没用）和 `CLAUDE_CODE_MAX_RETRIES=2`。
> - **只在长回复时卡、短问题正常**：很可能是中转站把回复缓冲到生成完才返回。生成超过 6 分钟的回复会被 Claude Code 掐断重发，**永远拿不到结果**。换支持流式的中转，或者让任务拆小。
> - **卡在执行某条命令上**（界面显示的是 Bash 而不是转圈）：那是另一回事，见 [Command timed out after 2m 0s](/zh/articles/claude-code-command-timed-out)。

## 问题背景

「Claude Code 一直卡住」是个很笼统的说法，界面上通常就是一个转圈的动词加计时，没有任何报错：

```
❯ 帮我把这个函数重构一下
✢ Clauding… (4m 34s)
```

等久了，有时会变成这样，然后又不动了：

```
✻ API error · Retrying in 0s · attempt 1/10
```

这时你无法判断它是在干活、在等网络，还是真的死了。官方文档在 [Streaming idle watchdogs](https://code.claude.com/docs/en/network-config#streaming-idle-watchdogs) 和 [No response from API](https://code.claude.com/docs/en/errors#no-response-from-api) 里列了一堆计时器，但每个计时器的生效条件都不一样，读者很难对号入座。所以这次直接把几种「卡住」做出来，量它到底等多久、界面显示什么。

## 问题分析

从网络上看，「卡住」只有四种可能，分别对应文档里不同的计时器：

| 卡在哪 | 后端在干什么 | 文档里管它的计时器 |
|---|---|---|
| ① 没回响应头 | 连接收了，但一直不回 HTTP 头（比如上游排队、缓冲型中转在等生成完） | 首字节超时（文档说：**设了 `ANTHROPIC_BASE_URL` 时不生效**），否则就是 `API_TIMEOUT_MS`（文档写默认 10 分钟） |
| ② 回了头，之后静默 | 回了 `200` 和 SSE 头，然后一个字节都不发 | 字节级看门狗（网关默认 300 秒）、事件级看门狗（300 秒） |
| ③ 只发 keep-alive ping | 一直发 `event: ping`，但没有正文 | ping 能重置字节级看门狗，事件级看门狗不认 |
| ④ 断在半路 | 正文发了一部分，然后不动了 | 字节级看门狗 |

几个和本文结论直接相关的官方原文：

- `API_TIMEOUT_MS`：*Timeout for API requests in milliseconds (default: 600000, or 10 minutes)*（[环境变量文档](https://code.claude.com/docs/en/env-vars)）
- 首字节超时运行在直连 Anthropic API 上，*but not when `ANTHROPIC_BASE_URL` … routes them through a gateway*（[network-config](https://code.claude.com/docs/en/network-config#streaming-idle-watchdogs)）
- 自动重试：*Claude Code retries transient failures up to 10 times with exponential backoff*（[errors](https://code.claude.com/docs/en/errors#automatic-retries)）

## 技术方案与选型

| 方案 | 结论 | 理由 |
|---|---|---|
| **本地 stub 模拟四种卡法** | ✅ 采用 | 能精确制造「不回头 / 静默 / 只发 ping / 半路断」，每个请求带毫秒时间戳记录，配合 Claude Code 的 `--debug-file` 能直接看到是哪个计时器触发的 |
| 拔网线、真实中转站 | ❌ 排除 | 卡法不可控，也区分不出是哪一种；会消耗真实额度 |
| 只读文档推算 | ❌ 不能单独用 | 实测有一处和文档不一致（见下文的 6 分钟） |

隔离做法和 [429 那篇](/zh/articles/claude-code-429-rate-limit) 相同：`env -i` 清空环境、每次全新的空 `CLAUDE_CONFIG_DIR`、假 key、`ANTHROPIC_BASE_URL` 指向 `127.0.0.1` 上的 stub，`HTTPS_PROXY` 也指向 stub 以拦截一切外连。debug 日志里能看到遥测上报（`1P event logging`）、bootstrap、MCP registry 的外连全部被拒，**没有请求到达真实服务**。

...

---

**[👉 继续阅读全文：Claude Code 一直卡住、转圈没反应怎么办：后端不回时它要等 6 分钟，重试满 10 次可能卡一个多小时（实测）](https://tools.cooconsbit.com/zh/articles/claude-code-stuck-no-response?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
