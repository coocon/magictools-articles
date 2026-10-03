# Claude Code 报错 Invalid API key · Fix external API key 怎么排查：Not logged in、Credit balance is too low、API Error 401/429/529 原文与重试实测

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-invalid-api-key?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-invalid-api-key?utm_source=github&utm_medium=referral)**

## 问题背景

把 Claude Code 接到 DeepSeek 这类兼容端点、换了一把 key、或者额度用完之后，跑 `claude -p` 常会看到下面几句中的一句：

```
Invalid API key · Fix external API key
Not logged in · Please run /login
Credit balance is too low
API Error: Request rejected (429) · …
API Error: 529 Overloaded. This is a server-side issue, usually temporary — …
```

报错本身不难看懂，难的是这几个问题：

- 这句话是 **Claude Code 本地判断出来的**，还是**后端返回的**？请求到底发出去了没有？
- 为什么 key 错了，命令要**卡将近 3 分钟**才报错？
- 写脚本时，报错在 stdout 还是 stderr？退出码是多少？`--output-format json` 里该看哪个字段？
- 如果你是第三方网关，应该读 `x-api-key` 还是 `Authorization: Bearer`？

本文的报错原文和数字，全部来自 2026-09-27 在本机 Claude Code 2.1.280 上的实测：28 组 case，共 61 次 `claude -p`。**后端不是真实 API，而是一个本地 stub**，这样可以任意指定返回码，并记下每个请求到达的时间和携带的 header。

## 问题分析

先在 2.1.280 的原生二进制（`claude.exe`，217 MB）里找到了这几句文案的定义：

```js
var _Ye="Not logged in \xB7 Please run /login",
    Gke="Invalid API key \xB7 Fix external API key",
    pat="Invalid auth token \xB7 Fix external auth token",
    mat="Invalid ANTHROPIC_CUSTOM_HEADERS \xB7 Fix the environment variable"
```

还有一条校验：`function JM(e){return/^[a-zA-Z0-9-_]+$/.test(e)}`，不通过就抛 `Invalid API key format. API key must contain only alphanumeric characters, dashes, and underscores.`。

光看静态代码，很容易得出「key 带特殊字符会在本地被拦」的结论。**实测推翻了这一点**：`ANTHROPIC_API_KEY='bad key!@#'` 被原样放进 `x-api-key` 发给了后端，stub 返回正常响应后 exit 0。回头再查代码，`edo()` 的唯一调用点在 OAuth 创建 API key 的流程里（`r.data?.raw_key` → `edo(s,n)`），环境变量里的 key 根本不经过它。所以本文所有结论都以实测为准，静态代码只用来解释现象。

读代码时还看到一处决定文案的逻辑，后面实测也印证了：401 会不会显示 `Invalid API key`，取决于错误 message 里**是否包含 `x-api-key` 这个字符串**：

```js
function uat(e){return e instanceof Error&&e.message.toLowerCase().includes("x-api-key")}
// ……
if(uat(e)){ …; return Ro({error:"authentication_failed",
  content: M==="ANTHROPIC_API_KEY"||M==="apiKeyHelper" ? Gke : _Ye}) }
```

## 技术方案与选型

目标是断言「请求发没发、发了几次、隔多久、带了哪个 header」，所以需要能控制后端返回什么，并记录每一个到达的请求。

| 方案 | 结论 | 理由 |
|---|---|---|
| **本地 stub（零依赖 Node）** | ✅ 采用 | 返回码和 body 可以任意指定；每个请求按毫秒时间戳和全量 header 写进 jsonl；不花钱，也不打扰真实服务 |
| 用真实 key 打 api.anthropic.com | ❌ 排除 | 429/529/5xx 造不出来；错 key 反复重试，等于对官方接口做无意义的压测 |
| 接真实第三方端点（DeepSeek 等） | ❌ 排除 | 错误体的格式由对方决定，无法控制变量；同一个错误在不同时间的返回可能不一样 |
| mitmproxy 抓包 | ❌ 排除 | 需要改证书信任链，还会引入一个额外变量；而 `ANTHROPIC_BASE_URL` 可以直接指向 http 的 stub，用不上 |
| 只读二进制里的字符串 | ❌ 不能单独用 | 上面的格式校验就是反例：代码里有，但这条路径上不生效 |

具体做法：

- `lab/stub.mjs` 监听 `127.0.0.1`，按 case 配置对 `POST /v1/messages` 返回合法 SSE、指定的状态码和 body，或者直接断开连接。
- 每次运行用 `env -i` 清空环境，配一个**全新的空 `CLAUDE_CONFIG_DIR`**，避免复用本机的 OAuth 登录态，工作目录也单独建。
- `HTTPS_PROXY` 同样指向 stub：遇到 `CONNECT` 就记录目标主机并返回 403。这样**任何外连都会被记录并拦截**，保证整个实验没有请求打到真实的 api.anthropic.com。
- 统一的调用方式：`claude -p 'reply with exactly OK' --model haiku --output-format text|json < /dev/null`。

...

---

**[👉 继续阅读全文：Claude Code 报错 Invalid API key · Fix external API key 怎么排查：Not logged in、Credit balance is too low、API Error 401/429/529 原文与重试实测](https://tools.cooconsbit.com/zh/articles/claude-code-invalid-api-key?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
