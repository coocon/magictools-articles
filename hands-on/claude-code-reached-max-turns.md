# Claude Code 报 Error: Reached max turns (1)：无头护栏掐断时，文件可能已经写了

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-reached-max-turns?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-reached-max-turns?utm_source=github&utm_medium=referral)**

这是 Claude Code 报错词族的第 3 篇。前两篇分别是 auto mode 不可用和 [MCP 连不上（Failed to connect）](/zh/articles/claude-code-mcp-failed-to-connect)，这次换一条新报错：无头模式 `claude -p` 的两道护栏被触发。

```text
Error: Reached max turns (1)
Error: Exceeded USD budget (0.01)
```

如果你用的是 `--output-format json`，看到的就不是这两行字，而是 `"subtype": "error_max_turns"`、`"terminal_reason": "max_turns"`，外加一个**不存在**的 `result` 字段。

![claude -p 的两个护栏报错，xxd 证明报错在 stdout、不带换行](https://cdn.tools.cooconsbit.com/uploads/articles/2026-09-25-max-turns/01-two-guardrail-errors.png)

## 问题背景

把 Claude Code 放进 CI、cron 或批处理脚本时，给它套上限是常规操作：

- `--max-turns N`：限制轮数，防止模型在工具调用里兜圈子
- `--max-budget-usd X`：限制花费，防止一次任务烧穿预算

[官方 CLI 参考](https://code.claude.com/docs/en/cli-reference)（2026-09-25 抓取）对这两个参数的描述都只有一句话：

- `--max-turns`：「Limit the number of agentic turns (print mode only). Exits with an error when the limit is reached. No limit by default.」
- `--max-budget-usd`：「Maximum dollar amount to spend on API calls before stopping (print mode only).」

文档没回答的恰恰是写脚本最关心的几件事：

- 报错打到哪条流？
- 退出码是多少？
- json 里长什么样？
- 被掐断时，模型已经做了的事算不算数？
- 「限 1 轮」到底是限什么？

这篇就把这些实测出来。

## 问题分析

所有实验都用同一个三步任务：读 `data.txt` 第 3 行的数字（42），乘以 2 写进 `out.txt`，再回复结果。正确完成时 `out.txt` 为 `84`，至少需要 3 次模型调用：Read → Write → 回复。这样一来，在不同位置掐断的效果看得很清楚。

先说结论。实测下来，这两道护栏有 4 个反直觉的地方：

1. **报错打在 stdout，不在 stderr。** `2>/dev/null` 过滤不掉它。它还不带换行，拼进日志会和下一行粘在一起。
2. **json 模式没有 `result` 键。** `jq -r .result` 打出的是四个字母 `null`，jq 自己的退出码是 0，下游脚本会把字符串 `"null"` 当成模型的回答继续往下传。这是这组报错最隐蔽的失败面。
3. **报失败 ≠ 没产生副作用。** 第 N 次调用发出的工具调用照常执行。`--max-turns 2` 报错时，文件已经写好了。
4. **预算是事后检查。** 要等一次 API 调用返回才比较累计花费，所以最多会超出一次调用的钱。超线的那一次调用可能恰好已经把任务做完了，可你拿到的仍是 exit 1。

另外，`--max-turns` 在 2.1.280 的 `claude --help` 里**查不到**，文档里却有。`--max-budget-usd` 在 help 里有，并注明「only works with --print」。

## 技术方案与选型

下游脚本要判断一次 `claude -p` 有没有成功，有这么几种做法：

| 方案 | 结论 | 理由 |
|---|---|---|
| 只看退出码 | **用**（判成败够了） | 两天 22 个 json run 里，claude exit 1 与判定 FAIL 一一对应 |
| `--output-format json` + 判 `is_error` 与 `subtype` | **用**（要知道原因时） | `terminal_reason` 能区分 max_turns / budget_exhausted / api_error |
| text 模式匹配 stdout 里的 `Error:` 字符串 | 排除 | 报错与正常回答走同一条流，不带换行，模型的正常回答里也可能出现 `Error:` |
| `jq -r .result` 取值后判空 | 排除 | 失败时打印字面量 `null`，jq exit 0，`[ -n "$r" ]` 照样为真 |
| 只看 `subtype` | 排除 | 网关 503 时 `subtype` 是 `success`，但 `is_error` 是 true（见下文） |
| 靠 `--max-turns` 约束交互模式 | 排除 | 交互模式不执行该参数（E6 实测，1 次） |
| 被掐断后 `--resume` 续跑来省钱 | 不作为省钱手段 | 09-25 续跑比重跑贵约 32%，09-23 两者持平 |

最终用的判定脚本如下。它跑遍了两天全部 22 个 json run，结论与 claude 的退出码完全一致：

```bash
#!/bin/bash
f="$1"
if ! jq -e . "$f" >/dev/null 2>&1; then echo "FAIL no-json"; exit 1; fi
read -r is_error subtype reason < <(jq -r '[(.is_error|tostring), (.subtype // "none"), (.terminal_reason // "none")] | @tsv' "$f")
if [ "$is_error" = "false" ] && [ "$subtype" = "success" ]; then
  echo "OK   $(jq -r '.result' "$f")"; exit 0
fi
echo "FAIL subtype=$subtype terminal_reason=$reason errors=$(jq -c '.errors // .result' "$f")"; exit 1
```

...

---

**[👉 继续阅读全文：Claude Code 报 Error: Reached max turns (1)：无头护栏掐断时，文件可能已经写了](https://tools.cooconsbit.com/zh/articles/claude-code-reached-max-turns?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
