# Claude Code 429 怎么办：Request rejected (429) 原文、会重试多久、retry-after 超过 60 秒直接放弃（实测）

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-429-rate-limit?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-429-rate-limit?utm_source=github&utm_medium=referral)**

> **先回答你搜的问题**
>
> - **`API Error: Request rejected (429)` 是什么**：后端（Anthropic 官方，或者你用的中转站 / 网关）说你请求太多了。`·` 后面那段是后端返回的原话，判断是谁限的流就看这一段。
> - **要不要手动重试**：一般不用。Claude Code 已经自己重试了最多 10 次、大约 3 分钟，你看到报错时说明这 3 分钟里一直在被限流。
> - **一瞬间就报错、根本没等**：后端给的 `retry-after` 超过了 60 秒，Claude Code 直接放弃了（实测阈值正好在 60 / 61 秒之间）。这种情况等一段时间再发就行，或者找中转站调大限额。
> - **交互模式下界面一直显示 `Retrying in Ns · attempt N/10`**：这是正常的退避等待，不是卡死，可以按 `Esc` 中断。
> - **月度消费上限触发的 429**：官方文档说这类 429 不带 `retry-after`，额度恢复前会一直失败。重试救不了，只能去控制台看额度。

## 问题背景

用 Claude Code 跑长任务，或者接的是国内中转站，时不时会看到这样一句：

```
API Error: Request rejected (429) · Number of request tokens has exceeded your per-minute rate limit
```

搜这句话的人通常想知道三件事：是谁在限流、Claude Code 自己会不会重试、要等多久。官方文档只说 SDK「会按指数退避重试，遇到 `retry-after` 头会遵守」，没说 Claude Code 具体重试几次、等多久，也没说 `retry-after` 很大时会怎样。

站内 [上一篇](/zh/articles/claude-code-invalid-api-key) 已经测过 429 的基本重试曲线，但留了几个空白：429 之后恢复成功的过程、交互模式下的显示、`retry-after` 取较大值时的行为、中转站的 429 长什么样。这篇专门把这些补齐。

## 问题分析

先确定 429 可能来自哪里，以及它们分别长什么样。下面都来自一手源：

| 来源 | 429 长什么样 | 出处 |
|---|---|---|
| Anthropic 官方 API | `{"type":"error","error":{"type":"rate_limit_error","message":"…"}}`，通常带 `retry-after` | [官方错误码文档](https://docs.claude.com/en/api/errors) |
| Anthropic 月度消费上限 | 同样是 `rate_limit_error`，但**不带 `retry-after`**，额度恢复前一直失败 | 同上，原文：*A tier spend-cap 429 has no `retry-after` header and keeps failing until access resumes* |
| new-api 按模型限流 | OpenAI 风格：`{"error":{"message":"您已达到请求数限制：N分钟内最多请求M次 (request id: …)","type":"new_api_error","code":""}}` | new-api 源码 `middleware/model-rate-limit.go`、`middleware/utils.go`（commit `1a4166d8e8`） |
| new-api 全局限流 | **空 body**，只有 `Retry-After` 头，值是整个限流窗口的秒数 | 同上 `middleware/rate-limit.go` 的 `writeRateLimited` |
| nginx `limit_req` | 一段 HTML 错误页 `<title>429 Too Many Requests</title>` | nginx 默认错误页 |
| DeepSeek | 文档只写了 `429 - Rate Limit Reached`，没给 body 格式 | [DeepSeek 错误码](https://api-docs.deepseek.com/quick_start/error_codes) |

另外，在 2.1.285 的二进制里能找到 `anthropic-ratelimit-unified-status`、`anthropic-ratelimit-unified-reset` 等一组响应头名，以及 `Usage limit reached` 等文案。这组头是订阅用户（Pro / Max）的用量限制，后面会验证：**用 API key 接入时它们不生效**。

## 技术方案与选型

| 方案 | 结论 | 理由 |
|---|---|---|
| **本地 stub 模拟后端** | ✅ 采用 | 状态码、body、`retry-after` 都能精确控制；每个请求带毫秒时间戳记录下来；不花钱，不影响真实账号 |
| 用真实账号撞限流 | ❌ 排除 | 429 很难稳定触发，`retry-after` 的值也控制不了；还会消耗真实额度，同一账号上的其他服务可能被连带限流 |
| 只读二进制里的字符串 | ❌ 不能单独用 | 文案被编译拆成了碎片，拼不出完整的显示模板；能说明「有这句话」，说明不了「什么时候显示」 |

需要说清楚的一点：**服务端返回什么是我们模拟的，Claude Code 客户端怎么反应（重试几次、等多久、显示什么）是真实的。** body 文案尽量照搬上表的一手源。

隔离做法和上一篇相同：

- 每次运行用 `env -i` 清空环境，配一个**全新的空 `CLAUDE_CONFIG_DIR`**，不碰本机的登录态和会话记录。
- `ANTHROPIC_BASE_URL` 指向 `127.0.0.1` 上的 stub，`ANTHROPIC_API_KEY` 用一把假 key。
- `HTTPS_PROXY` 也指向 stub：所有 `CONNECT` 外连都会被记录并拒绝。本轮记到的外连目标有 api.anthropic.com、github.com、raw.githubusercontent.com、downloads.claude.ai、registry.npmmirror.com，**全部被拦，没有一个请求到达真实服务**。
- `-p` 模式统一用 `claude -p 'reply with exactly OK' --model haiku < /dev/null`；交互模式在 tmux 里启动真实 TUI，每秒截一帧屏幕。

...

---

**[👉 继续阅读全文：Claude Code 429 怎么办：Request rejected (429) 原文、会重试多久、retry-after 超过 60 秒直接放弃（实测）](https://tools.cooconsbit.com/zh/articles/claude-code-429-rate-limit?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
