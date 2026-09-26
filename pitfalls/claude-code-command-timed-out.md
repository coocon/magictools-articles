# Claude Code 报错 Command timed out after 2m 0s 怎么解决：两条超时路径、BASH_DEFAULT_TIMEOUT_MS 与 run_in_background 实测

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-command-timed-out?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-command-timed-out?utm_source=github&utm_medium=referral)**

## 问题背景

让 Claude Code 跑一个慢命令，比如冷启动的 `npm install`、整套测试、`docker build`，或者一段 `sleep` 轮询，过了两分钟，工具结果变成这样：

```
Exit code 143
Command timed out after 2m 0s
```

这是 Claude Code 的 Bash 工具默认超时：120000 ms。常见的处理办法是「把超时调大」，但实际碰到后，至少还有三个问题没回答：

- 命令被杀后，已经输出的内容还在不在？子进程会不会变成孤儿进程？
- 用脚本跑 `claude -p` 时，怎么发现命令其实超时了？
- `BASH_DEFAULT_TIMEOUT_MS`、`BASH_MAX_TIMEOUT_MS`、单条命令的 `timeout` 参数、`run_in_background`，分别该在什么场景用？

这篇的所有报错原文和数字，都来自 2026-09-26 在本机 Claude Code 2.1.280 上跑的 19 次真实 `claude -p` 会话，逐字照抄。

## 问题分析

先读了一遍 2.1.280 的二进制（`claude.exe`，217,254,576 字节），相关代码有这几段：

- 默认值和上限：`var p=120000,d=600000`。`BASH_DEFAULT_TIMEOUT_MS` 只有在 `!isNaN(o)&&o>0` 时才生效，否则回落 120000。`BASH_MAX_TIMEOUT_MS` 取 `Math.max(max, default)`，所以上限永远不会小于默认值。
- 报错模板：`Command timed out after ${zt(this.#u)}`。
- 单条命令的 timeout 取 `Math.min(请求值||默认值, 上限, …)`。被夹小时只发一个 `timeout_clamped` 遥测事件，模型看不到。
- **超时自动转后台**：满足 `!or&&Ee===void 0&&kYr(Be)` 时，超时的命令不杀，改为转到后台。`kYr` 会取命令的首词，只要不在 `gYr=["sleep"]` 里就放行。`or` 为真的情况包括设置了 `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS`。

最后一条是这次最意外的发现。**2.1.280 里 Bash 超时有两条完全不同的路径**：

| 路径 | 触发条件 | 工具结果 | `is_error` |
|---|---|---|---|
| 被杀 | 首词是 `sleep`，或设了 `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1` | `Exit code 143\nCommand timed out after 2m 0s` | `true` |
| 转后台 | 其它命令，例如 `echo …; sleep 300`、`(sleep 301; echo X) \| cat` | `Command did not complete within its 120s timeout and was moved to the background (ID: …)` | `false` |

也就是说，在 2.1.280 里真正会看到「Command timed out after 2m 0s」的，只有第一种情况。标题里这条报错，下面的实验主要用 `sleep 300` 来稳定复现。

## 技术方案与选型

默认两分钟不够用时，有三种绕法，它们的代价不一样：

| 绕法 | 代价 | 适用场景 |
|---|---|---|
| 环境变量 `BASH_DEFAULT_TIMEOUT_MS`（配合 `BASH_MAX_TIMEOUT_MS`） | 对所有命令生效，真卡死的命令也要等满新阈值；`0` 或非数字会静默回落 120 秒；上限默认 600000 ms，想超过 10 分钟必须同时抬高 MAX | 整个会话都在跑长构建，或 CI 类无头任务 |
| 单条命令传 `timeout` 参数（上限是 `BASH_MAX_TIMEOUT_MS`，默认 600000） | 要靠模型主动传参；超过上限会被**静默夹取** | 已知个别命令慢，例如一次长测试 |
| `run_in_background` | 立即返回，但完成通知不带输出，要再读文件；`-p` 模式下最终回复 5 秒后就会被 kill；实测模型还可能**编造完成通知** | 交互式长任务；在 `-p` 下必须让模型前台轮询到完成 |

排除项：

- **不推荐在命令外面再套一层 `timeout 600 …`**。外层的 GNU `timeout` 只能让命令提前结束，没法延长 Claude Code 自己的 120 秒计时器（这是按代码逻辑推断的，本轮未实测）。
- **不推荐靠 `nohup … &` 把进程甩出去**。本轮 `sleep 305 & wait` 的实验里，`&` 起的子进程和外层 shell 同属一个进程组，超时时被一起杀掉（见下文 B2c）。就算甩出去了，模型也拿不到结果。
- **本轮不讨论 `CLAUDE_CODE_AUTO_BACKGROUND_TIMEOUT_MS`**。二进制里有这个变量（会把可转后台命令的超时缩短），但本轮没实测，不给建议。

## 实测过程

所有会话都用同一种形式调用，每个 run 用独立的 `work/` 目录和隔离的 config 目录：

```bash
cd <run>/work
CLAUDE_CONFIG_DIR=<lab>/claude-config claude -p '<prompt>' \
  --allowedTools Bash --model haiku --setting-sources user \
  --debug-file <run>/debug.log --output-format text < /dev/null
```

...

---

**[👉 继续阅读全文：Claude Code 报错 Command timed out after 2m 0s 怎么解决：两条超时路径、BASH_DEFAULT_TIMEOUT_MS 与 run_in_background 实测](https://tools.cooconsbit.com/zh/articles/claude-code-command-timed-out?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
