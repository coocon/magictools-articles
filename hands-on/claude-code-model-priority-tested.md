# Claude Code 到底在用哪个模型？settings.json、环境变量、--model 的优先级实测

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-model-priority-tested?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-model-priority-tested?utm_source=github&utm_medium=referral)**

## 为什么要较真这件事

模型配置错了不报错——会话照常跑，只是跑在另一个模型上。GitHub issue [#82466](https://github.com/anthropics/claude-code/issues/82466) 里的用户在 `~/.claude/settings.json` 配了 Fable，多智能体任务静默跑了一整天 Sonnet，发现时全部作废重做。

官方文档分别介绍了 `settings.json` 的 `model` 字段、`ANTHROPIC_MODEL` 环境变量、`--model` 参数和 `/model` 命令，但**没有一张表写清同时存在时谁赢**。下面是实测结果。

## 实测方法：用 modelUsage 拿铁证

无头模式加 `--output-format json`，返回结果里的 `modelUsage` 键就是**实际计费的模型 ID**——比任何界面显示、模型自述都可靠（模型自述"我是谁"可能受系统提示影响，不能作数）：

```bash
claude -p "reply ok" --output-format json | python3 -c \
  "import json,sys; print(list(json.load(sys.stdin)['modelUsage'].keys()))"
```

## 优先级实测矩阵

环境：macOS + Claude Code v2.1.220，每一行都是干净目录里的独立实验，证据取 `modelUsage`：

| 实验配置 | 实际使用 | 结论 |
|---|---|---|
| 项目 `.claude/settings.json` 配 haiku，无其它配置 | haiku | ✅ 项目级 settings 生效 |
| settings 配 `claude-fable-5[1m]`（1M 上下文后缀） | `claude-fable-5[1m]` | ✅ 带 `[1m]` 后缀的写法同样生效 |
| settings 配 haiku + 环境变量 `ANTHROPIC_MODEL=sonnet` | sonnet | 环境变量 **覆盖** settings |
| settings 配 haiku + 参数 `--model sonnet` | sonnet | 参数 **覆盖** settings |
| 环境变量 haiku + 参数 `--model sonnet` | sonnet | 参数 **覆盖** 环境变量 |

...

---

**[👉 继续阅读全文：Claude Code 到底在用哪个模型？settings.json、环境变量、--model 的优先级实测](https://tools.cooconsbit.com/zh/articles/claude-code-model-priority-tested?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
