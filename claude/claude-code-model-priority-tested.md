---
title: "Claude Code 到底在用哪个模型？settings.json、环境变量、--model 的优先级实测"
slug: claude-code-model-priority-tested
category: claude
locale: zh
translationSlug: claude-code-model-priority-tested-en
tags: [Claude Code, 模型配置, settings.json, ANTHROPIC_MODEL, CLI, claude-code-lab]
summary: "多智能体、CI、批量脚本都依赖一个前提：会话真的跑在你配置的模型上。但模型可以在四个地方指定——settings.json、ANTHROPIC_MODEL 环境变量、--model 参数、会话内 /model——文档没有一张表告诉你谁覆盖谁，GitHub 上还有 Windows 用户报告 settings.json 配置被静默忽略、一整天的任务跑错模型全部作废。本文用 modelUsage 铁证逐层实测出优先级链：--model > ANTHROPIC_MODEL > 项目 settings.json > 内置默认，[1m] 后缀写法同样生效；并给出一个比任何 UI 提示都可靠的验证手段。"
status: published
lab:
  testedAt: "2026-08-06"
  ccVersion: "2.1.220"
  model: "haiku-4-5 / sonnet-5 / fable-5[1m]（矩阵实测）"
  platform: macOS
  status: reproducible
docsUrl: https://docs.claude.com/en/docs/claude-code/settings
---

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

得出的优先级链（高 → 低）：

```
--model 参数  >  ANTHROPIC_MODEL 环境变量  >  项目 settings.json  >  内置默认
```

这符合"越接近本次调用的配置越优先"的 Unix 直觉，但在此之前它只是直觉——现在是证据。

## 关于那个 Windows issue

issue #82466 报告的是另一层现象：**用户级** `~/.claude/settings.json`（注意不是项目级）在 Windows 11 + PowerShell 下被忽略，且交互会话里 `/model <精确ID>` 返回 "Kept model as X" 拒绝切换。报告者排查了 env、项目覆盖、`lastModel` 等所有已知落盘位置均无异常，怀疑存在未落盘的客户端 UI 状态覆盖。

我们在 macOS 上**无法复现**（如上表，配置层层生效），截稿时该 issue 无官方回应。如果你在 Windows 上遇到模型不对：先别怀疑自己的配置写错了，用上面的 `modelUsage` 方法取证，然后到该 issue 下补充你的环境信息——多一份跨环境报告，修复就近一步。

## 实用建议

- **CI / 脚本**：显式传 `--model`，它优先级最高、作用域最小，不受任何持久化状态干扰
- **项目统一默认**：写项目 `.claude/settings.json`（可入库共享），实测可靠
- **验证手段**：任何"感觉模型不对"的时刻，跑一次 `--output-format json` 看 `modelUsage`；交互会话里则运行不带参数的 `/model` 打开选择器**看高亮项**——issue 报告者的经验是文字提示 "Kept model as X" 可能误导，选择器高亮才是当前真实状态
- **计费敏感场景**：`modelUsage` 同时带 token 数与成本，顺手可核对账单

## 适用边界

- 实测环境见文首适用范围卡；矩阵覆盖无头模式（`-p`）下的项目级 settings / env / flag 三层，**用户级** `~/.claude/settings.json` 与交互模式 `/model` 切换未在本文实测范围内（前者不便在生产机上安全变更，后者见 issue 讨论）
- Bedrock / Vertex 通道有各自的模型 ID 体系与 env 变量，本文结论仅针对直连 Anthropic API/订阅
- 优先级属长期稳定的设计语义，预期跨版本有效；但 Windows issue 修复后本文会更新状态
