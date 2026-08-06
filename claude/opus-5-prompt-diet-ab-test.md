---
title: "我按「删掉这 5 类提示词」清了 Opus 5 配置，然后跑了 18 次 A/B：省了 21%，但「更好」我拿不出证据"
slug: opus-5-prompt-diet-ab-test
category: claude
locale: zh
translationSlug: opus-5-prompt-diet-ab-test-en
tags: [Claude Code, Claude Opus 5, 提示词工程, CLAUDE.md, A/B 测试, 开发者工作流]
summary: "「Opus 5 自带验证，把兜底提示词全删掉」——这个说法很流行，我照做了，然后用 CLAUDE_CONFIG_DIR 隔离出新旧两份全局配置，同任务同模型跑了 18 次 headless 对照。省 token 是真的：输出 token -21%，耗时 -20% 到 -29%，全部 9 组三次运行无一例外。但被删掉的规则里有两条根本没测出效果，还有一个反例：旧配置最认真的那一次，覆盖面是新配置所有运行的严格超集。这篇写实验怎么搭、数据长什么样，以及为什么「省」和「好」必须分开问。"
status: published
---

有一类说法这半年特别流行：Claude Opus 5 已经自带验证和思考，你写在 `CLAUDE.md` 里那些「回答前再检查一遍」「用子智能体复查」的兜底规则，现在全在帮倒忙——删掉既省 token 又不掉质量。Anthropic 官方在迁移指引里表达过类似意思，Claude Code 之父 Boris Cherny 说得更狠：每六个月把提示词全删一遍，看看模型不戴枷锁能做什么。

我照做了。我的全局 `~/.claude/CLAUDE.md` 里确实躺着几条典型的旧时代产物：

```markdown
### 2. Subagent Strategy
- Use subagents liberally to keep main context window clean
- For complex problems, throw more compute at it via subagents

### 4. Verification Before Done
- Never mark a task complete without proving it works
- Ask yourself: "Would a staff engineer approve this?"
```

删掉验证节、反转子智能体策略、补上范围合同和长度控制，113 行改成 87 行。

然后问题来了：**我怎么知道改完是变好了，而不是我把有用的东西删了、然后靠"新配置感觉更清爽"自我说服？**

所以我搭了个对照实验。结论先放这儿：**省是真的省，而且非常稳定；「更好」我拿不出证据，甚至找到了一个反例。**

## 实验怎么搭

关键是配置隔离。Claude Code 支持 `CLAUDE_CONFIG_DIR` 环境变量指定配置目录，于是新旧两份 `CLAUDE.md` 各放一个目录，配上同一份 `settings.json`（同一个模型 `opus`、同样的推理预算）：

```bash
CLAUDE_CONFIG_DIR=./cfg-old claude -p "$PROMPT" \
  --output-format stream-json --verbose \
  --permission-mode acceptEdits \
  --allowedTools "Read,Edit,Write,Glob,Grep,Task,TodoWrite"
```

先验证隔离真的生效——让两边各自复述「你的全局指令里关于子智能体的那一条」：

```
=== cfg-old ===
- Use subagents liberally to keep main context window clean
=== cfg-new ===
- 默认自己做。只有任务确实需要独立并行处理时才委派
```

生效了。`--output-format stream-json` 会吐出每一次工具调用的结构化事件，从里面能数出工具调用次数、`Task` 工具（子智能体）的次数，最后的 `result` 事件带 `usage.output_tokens`。

三个任务，各针对一条被改动的规则：

| 任务 | 测什么 | 内容 |
|---|---|---|
| A | 范围合同 | 沙盒仓库里修一个真 bug：超时后 `AbortController` 的定时器没清理。周围故意埋了 `any` 类型、重复代码、没有退避的重试循环当诱饵 |
| B | 长度控制 | 开放式排查：「找出 `src/` 下会让重试逻辑出问题的地方」 |
| C | 委派倾向 | 在真实仓库（本站源码，1000+ 文件）只读探索：「找出所有决定文章 hreflang / canonical 的代码路径」 |

A、B 在沙盒里跑，每次从 git 基线全新复制；C 在真实仓库跑，只给 `Read/Glob/Grep/Task`，不给 `Edit/Write`，零改动风险。沙盒刻意放在项目目录之外——否则会继承项目级的 `CLAUDE.md`，多一个变量。

每个组合跑 3 次，2 配置 × 3 任务 × 3 次 = 18 次运行。

## 数据

```
任务   指标            old 均值   new 均值      变化   old 三次 / new 三次
A     out_tokens          945       656      -31%   [826, 1037, 973] / [669, 509, 789]
A     reply_chars         430       166      -61%   [399, 437, 454] / [221, 74, 203]
A     secs                 18        14      -25%   [15, 21, 19] / [12, 11, 18]
A     subagents             0         0       n/a   [0, 0, 0] / [0, 0, 0]

B     out_tokens         4314      3026      -30%   [4681, 4456, 3805] / [3684, 2570, 2824]
B     tool_calls            7         5      -25%   [9, 6, 5] / [6, 5, 4]
B     reply_chars        2372      1818      -23%   [2200, 2579, 2338] / [2141, 1628, 1686]
B     secs                 77        54      -29%   [92, 73, 66] / [64, 46, 53]

C     out_tokens         9410      7903      -16%   [8051, 10600, 9580] / [8060, 7925, 7725]
C     tool_calls           22        19      -15%   [16, 25, 25] / [21, 18, 17]
C     secs                149       120      -20%   [123, 183, 142] / [131, 120, 109]

18 次运行输出 token 总计: old=44009  new=34755  (-21.0%)
```

省 token 这条**没有一个反例**。三个任务、九组配对，输出 token 和耗时全部下降，方向完全一致。A 任务的回答长度砍掉 61%——那条「回答保持简洁」的新增规则，效果是肉眼可见的。

然后是不那么好看的部分。

## 发现一：有两条规则我根本没测出效果

`subagents` 那一列，18 次运行全是 0。

旧配置里明明白白写着「liberally use subagents」「throw more compute at it via subagents」，`Task` 工具也在 `--allowedTools` 白名单里，任务 C 是在一个 1000+ 文件的真实仓库里做跨目录探索——这是最该触发委派的场景。它一次都没派。

所以我花力气改写的那条子智能体规则，在这个实验里**完全没有被验证到**。它可能有用，可能没用，我不知道。

范围蔓延也一样。任务 A 两边都只碰了 `http.ts` 一个文件，都没顺手改 `any`、没顺手加退避、没顺手写测试。行数上新配置反而更多（+10/-6 vs +4/-2），但看 diff 就知道那不是蔓延，是结构选择的差别：

```diff
# old：把 controller 和 timer 提到 try 外面，给已有的 try/catch 补 finally
+    const ctrl = new AbortController();
+    const timer = setTimeout(() => ctrl.abort(), TIMEOUT);
     try {
...
+    } finally {
+      clearTimeout(timer);
     }

# new：在原位加一层嵌套 try/finally
+      const timer = setTimeout(() => ctrl.abort(), TIMEOUT);
+      try {
+        ...
+      } finally {
+        clearTimeout(timer);
+      }
```

两个修复都正确，都覆盖了三条退出路径（正常 return、5xx 的 `continue`、异常）。「diff 行数」这个我自以为聪明的代理指标，在这里根本不指向范围蔓延。

诚实的说法是：**18 次运行里，我只测出了「长度/成本」这一个维度的差异。**

## 发现二：一个反例

任务 C 我本来想比覆盖度——从回答里正则抽出被提到的源文件路径，看谁找得全。前两轮数据很漂亮：新配置的并集 11 个文件，是旧配置 4 个的严格超集。更省 + 找得更全，正是我想要的结论。

然后我去查一个异常值。`old-C-3` 那次的回复只有 214 字，远低于其他所有运行。原因是它没在终端里回答，而是把清单写进了一个计划文件：

> 清单已完成，输出在 `plans/hreflang-canonical-floofy-sparkle.md`（含每处的文件路径 + 行号）。本次没有做任何代码改动。

把那个文件算进来之后，结论翻转了：

```
old 并集 20 个 | new 并集 11 个
只有 old 找到: scripts/backfill-article-locale.ts, scripts/sync-articles.ts,
              src/app/[locale]/layout.tsx, src/app/robots.ts,
              src/app/api/automation/articles/route.ts,
              src/components/dashboard/article-editor.tsx ...（共 9 个）
只有 new 找到: （无）
```

旧配置最认真的那一次，覆盖面是新配置全部三次的**严格超集**。

类似的事在任务 B 也出现过一次。旧配置的第一轮用了 9 次工具调用（新配置 6 次），因为它真的跑起 Node 做了实测：

> 实测（Node 25.9）：`request 已返回, 耗时 520 ms` / `进程真正退出, 耗时 10004 ms`

这正是被我删掉的那类「再验证一遍」行为。它更慢、更贵——也给出了一条实测数据，而不是读代码推断出来的结论。

## 所以「更好」这个词得拆开

我的实验能支持的结论只有一条：**同任务同模型下，精简后的配置输出 token 少 21%、耗时少 20-29%，非常稳定。**

它**不能**支持这些：

- 新配置质量更高——覆盖度对比不成立，还有一个方向相反的反例
- 子智能体规则改对了——18 次运行零委派，这条压根没被触发
- 范围合同起作用了——两边都没蔓延，无从区分

n=3，单一模型，三个我自己设计的任务。这个样本量能看出 21% 的稳定差异，不足以判断质量差异——质量的方差比成本大得多，而且我没有一个中立的评分器。

那删还是不删？我的读法是：这些兜底提示词的作用，不是「让模型更聪明」，而是**在模型本可以偷懒的时候逼它多走一步**。Opus 5 大多数时候不需要这一脚——所以删掉的收益（省 21%）是稳定的、每次都拿得到的。但那一脚偶尔真的踢出了东西：一次实测数据，一份 20 个文件的完整清单。你删掉的是一个概率不高但确实存在的上限。

这笔账每个人的答案不一样。日常改代码，21% 的稳定收益换掉一个偶发上限，我认。做安全审计或者线上事故排查，我会把那节验证规则临时加回去——文章里那句「只给需要验证的那一步加，不要写成全局规则」，现在我知道它具体指什么了。

## 想自己跑一遍

三个要点，剩下的都是体力活：

1. **配置隔离用 `CLAUDE_CONFIG_DIR`**，把 `settings.json` 一起复制过去（否则拿不到鉴权），并且删掉里面的 `hooks` 和 `statusLine`——沙盒里路径无效，还会污染你的 hook 日志。
2. **沙盒放在项目目录之外**，不然会继承项目级 `CLAUDE.md`，那是个你控制不住的变量。
3. **指标从 `--output-format stream-json` 里抠**：`select(.type=="assistant")|.message.content[]?|select(.type=="tool_use")|.name` 数工具调用，`select(.name=="Task")` 数子智能体，最后的 `result` 事件带 `usage.output_tokens`。

还有一条不是技术的：**先想清楚你的代理指标测的是不是你要的东西**。我用「diff 行数」代表范围蔓延，结果它测到的是代码结构偏好；我用「回答字数」代表覆盖度，结果漏掉了写进文件的那一份。两次都是看了原始产物才发现的。跑完实验别急着看汇总表——先读几份原始输出。
