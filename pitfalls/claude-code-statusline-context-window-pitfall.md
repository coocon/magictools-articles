---
title: "Fable 5 明明是 1M 上下文，状态栏却显示 200k：别急着换工具，先抓一份现场数据"
slug: claude-code-statusline-context-window-pitfall
category: pitfalls
locale: zh
tags: [Claude Code, statusline, 上下文窗口, Bug 复盘, LLM]
summary: "Claude Fable 5 官方确认 1M token 上下文窗口，但 Claude Code 底部状态栏的分母一直是 200k。第一反应是「换个更好的 statusline」——错了。本文复盘完整排障过程：用一行 tee 抓下 statusline 的 stdin 现场，实锤官方字段对新模型误报 200000，最后用一张模型表修正。附赠一个通用教训：换工具修不了数据源的错。"
status: published
source: authored
translationSlug: claude-code-statusline-context-window-pitfall-en
---

## 现象

Claude Code 里用 Claude Fable 5 干活，官方文档明确它是 **1M token 上下文窗口**（默认开启，不需要 beta header）。但底部自定义状态栏显示的是：

```
magictools | Fable 5 | Ctx 38% (76k/200k)
```

分母 200k。照这个算法，用到 16 万 token 就开始飘红报警——实际上离真正的窗口上限还差 84 万。

第一反应很自然：**是不是我这个 statusline 脚本太差了，换个社区流行的？**

这个念头是本文要复盘的第一个坑。

## 先想清楚：换 statusline 能解决吗

Claude Code 的 statusline 机制是：每次刷新时把一份 JSON 通过 stdin 喂给你配置的命令，里面带模型信息、工作目录、上下文用量等字段。所有 statusline——不管是自己写的 bash 脚本还是社区项目——拿到的都是**同一份 stdin**。

窗口大小只有两个来源：

1. 读 stdin 里的 `context_window.context_window_size` 字段
2. 按模型名查自己内置的「模型 → 窗口大小」表

如果问题出在字段本身报错了，换任何一个读这个字段的 statusline 都是换汤不换药；如果指望内置表，Fable 5 这种刚发布的模型大概率还没进第三方项目的表里。

**结论：先确认数据源，再谈换工具。** 否则换完发现还是 200k，白折腾一轮。

## 抓现场：一行 tee 拿到 stdin 实锤

statusline 脚本的 stdin 是瞬时的，平时看不到。排障最直接的办法是临时加一行，把每次的输入落盘：

```bash
input=$(cat)
printf "%s" "$input" > /tmp/statusline-debug.json   # 临时调试行，用完删
```

等状态栏刷新一次（随便让会话产生一条消息就行），文件里就是货真价实的现场：

```bash
jq '.model, .context_window' /tmp/statusline-debug.json
```

```json
{
  "id": "claude-fable-5",
  "display_name": "Fable 5"
}
{
  "total_input_tokens": 76334,
  "context_window_size": 200000,
  "used_percentage": 38
}
```

实锤了：**模型 id 明明是 `claude-fable-5`，Claude Code 报的 `context_window_size` 却是 200000。** 这不是脚本算错，是上游数据源就错了——Claude Code 不认识新模型的窗口大小时，会回落到 200k 默认值（对应已知 issue #76751，1M 会话误报 200k）。

到这一步，「换 statusline」的方案可以正式毙掉：谁来读这个字段都是 200000。

## 根因

三层叠加：

1. **官方字段误报**：Claude Code 对 1M 窗口模型（实测 `claude-fable-5`）的 `context_window_size` 报 200000
2. **脚本兜底写死**：脚本里字段缺失时 `ctx_size=200000`，进一步固化了这个值
3. **原有修正条件太苛刻**：脚本此前只在「已用量超过报告值」时才修正为 1M——意味着必须先用掉 20 万 token，分母才会变对。在那之前，百分比一直按 200k 算，红色告警全是虚惊

## 修复：一张模型表 + 保留兜底

既然上游字段靠不住，就在脚本里维护一张**已知 1M 窗口模型表**，按 `model.id` 强制修正：

```bash
model_id=$(printf '%s' "$input" | jq -r '.model.id // empty')

# 官方字段对 1M 窗口模型误报 200k（#76751，实测 claude-fable-5 报 200000）
# 已知 1M 窗口模型按表强制修正；[1m] 后缀显式声明 1M，优先于模型表
case "$model_id" in
  *"[1m]"*)
    ctx_size=1000000
    ;;
  *fable-5*|*mythos-5*|*opus-5*|*sonnet-5*|*opus-4-8*|*opus-4-7*|*opus-4-6*|*sonnet-4-6*)
    if [ -z "$ctx_size" ] || [ "$ctx_size" -le 200000 ] 2>/dev/null; then
      ctx_size=1000000
    fi
    ;;
esac
[ -z "$ctx_size" ] && ctx_size=200000
# 兜底：未知模型用量超过报告窗口时，也按 1M 修正
if [ -n "$ctx_used" ] && [ "$ctx_used" -gt "$ctx_size" ] 2>/dev/null; then
  ctx_size=1000000
fi
```

表里的型号（Fable/Mythos 5、Opus 5/4.8/4.7/4.6、Sonnet 5/4.6）都是官方文档确认 1M 窗口的。原来那条「用量超过报告值就修正」的逻辑保留，作为表外未知模型的兜底。

验证不用等真实会话——刚才抓的 debug JSON 就是现成测试用例：

```bash
bash ~/.claude/statusline-command.sh < /tmp/statusline-debug.json
# magictools | Fable 5 | Ctx 7% (79k/1M)
```

同一份输入，38% 变 7%，分母 200k 变 1M。改完记得删掉调试行、清掉 `/tmp` 的落盘文件。

## 一个容易忽略的尾巴：1M ≠ 可用 1M

Claude Code 会为 auto-compact 预留缓冲，1M 窗口的实际可用预算约 **830k** 左右。所以状态栏按 1M 算百分比时，心里要留一档：80% 飘红时就该收尾了，别等 100%。

## 带走的教训

1. **换工具之前，先确认坏的是不是数据源。** 所有下游消费同一份上游数据时，换下游是无效动作。这次如果直接换了社区 statusline，结果还是 200k，还多引入一个依赖。
2. **瞬时数据要落盘再排障。** stdin、管道、hook 输入这类「看不到的现场」，加一行 tee/重定向落盘，比盯着结果猜快得多。抓下来的现场还能直接当回归测试的输入。
3. **写死的兜底值要有失效预案。** `ctx_size=200000` 在写下的那天是对的，模型迭代后就成了暗雷。兜底值旁边最好留一张按标识符查的修正表，并给表外情况保留动态修正逻辑。
4. **新模型上线后，把「工具链对它的认知」也当成待验证项。** 模型能力升级了，编辑器、CLI、监控脚本里关于它的硬编码假设（窗口大小、价格、tokenizer）不会自动跟上。
