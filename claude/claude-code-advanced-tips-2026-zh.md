---
title: "Claude Code 进阶实战：会话恢复、Checkpoint 回滚与 Headless CI 的 10 个官方技巧"
slug: claude-code-advanced-tips-2026-zh
category: claude
locale: zh
translationSlug: claude-code-advanced-tips-2026-en
tags: [Claude Code, 实用技巧, Headless, Checkpoint, 上下文管理, 成本控制]
summary: "从会话命名恢复、/rewind 时光机、上下文瘦身，到 headless 模式接入 CI、按路径生效的规则文件、自动记忆——10 个官方文档记载但多数人没用起来的 Claude Code 进阶技巧，每条都给出具体命令和适用场景。"
status: published
---

会用 Claude Code 和用好 Claude Code 之间，隔着一批"官方文档里写了、但多数人从没打开过的页面"。这篇不讲入门（新手请看站内的 Claude Code 快速上手指南），只挑 10 个能直接改变工作流的进阶技巧，全部来自官方文档，每条按"怎么操作 → 什么时候用"的结构展开。

## 目录

1. 给会话起名字，随时找回来
2. /rewind：代码和对话分开回滚的时光机
3. 上下文瘦身三件套：/clear、/compact、/context
4. 按路径生效的规则文件（Path-Scoped Rules）
5. 自动记忆：让 Claude 自己记笔记
6. Headless 模式：把 Claude Code 接进脚本和 CI
7. 结构化输出与成本核算
8. 探索 → 规划 → 实现 → 提交的四段式工作流
9. 富媒体上下文：截图、@文件、管道喂日志
10. Skills：把重复流程固化成一条命令

## 1. 给会话起名字，随时找回来

多任务并行时最大的痛苦是"昨天那个改到一半的会话去哪了"。解法是从一开始就命名：

```bash
claude -n payment-refactor     # 启动时命名
/rename payment-refactor       # 会话中途改名
claude --continue              # 恢复最近一个会话
claude --resume                # 打开会话选择器，按名字挑
```

配合 `/branch <name>` 还能从当前对话分叉出一个副本——主线继续改代码，分支去试另一个方案，两边共享此前的全部上下文。

**适用场景**：任何超过一天的任务、任何同时推进 2 个以上任务的人。

## 2. /rewind：代码和对话分开回滚的时光机

Claude Code 会在你每次发消息时自动打快照（checkpoint）。当它改崩了代码，你不需要手动 `git stash` 抢救：

- 按 `Esc` 两下或输入 `/rewind` 打开回滚菜单
- 选择恢复到任意一个历史时刻
- 关键是可以**只恢复代码**（对话记忆保留，Claude 还记得教训）或**只恢复对话**（代码保留，重新组织提问）

**适用场景**：让 Claude 尝试激进方案之前，你什么都不用做——快照是自动的，敢想敢试的底气就来自这里。注意 checkpoint 不能替代 git：它是会话内的撤销键，不是版本管理。

## 3. 上下文瘦身三件套：/clear、/compact、/context

长会话变笨、变贵的根源是上下文堆满了无关内容。三个命令对应三种情况：

| 命令 | 作用 | 什么时候用 |
|------|------|-----------|
| `/context` | 查看当前上下文占用明细 | 感觉响应变慢变贵时先诊断 |
| `/clear` | 清空上下文重新开始 | 切换到完全不相关的新任务 |
| `/compact 保留XX相关的决策` | 把历史压缩成摘要 | 任务没做完但上下文快满了 |

`/compact` 后面可以带指示语，告诉它压缩时重点保留什么——比如 `/compact 保留数据库 schema 的设计决策`，比无脑压缩效果好得多。

## 4. 按路径生效的规则文件（Path-Scoped Rules）

CLAUDE.md 大家都会写，但很多人把它写成了几百行的大杂烩——每个会话都全量加载，污染上下文。官方的解法是 `.claude/rules/` 目录 + frontmatter 声明生效路径：

```markdown
---
paths:
  - "src/api/**/*.ts"
---
# API 开发规范
- 所有端点必须做输入校验
- 错误响应使用统一格式 { code, msg, data }
```

只有当 Claude 实际触碰 `src/api/` 下的文件时，这份规则才会被加载。CLAUDE.md 留给全局共识（技术栈、构建命令），模块规范全部下沉到 rules。

**适用场景**：中大型代码库、前后端混合仓库、规范特别多的团队。

## 5. 自动记忆：让 Claude 自己记笔记

Claude Code 默认开启自动记忆：它会把跨会话有价值的发现（构建命令的坑、调试出的结论、你的代码风格偏好）写进项目记忆目录，下次会话自动带上。

```bash
/memory        # 查看记忆索引和所有记忆文件
```

你要做的只有两件事：定期打开 `/memory` 删掉过时条目（错误的记忆比没有记忆更糟）；发现它记了不该记的内容时直接说"忘掉这条"。

## 6. Headless 模式：把 Claude Code 接进脚本和 CI

`-p` 参数让 Claude Code 变成一个可编排的命令行工具，这是它和"聊天窗口"拉开差距的地方：

```bash
# 最简用法：一问一答，不进交互界面
claude -p "总结这个仓库的模块结构"

# CI 里跑：预授权工具，防止卡在确认上
claude -p "修复所有 lint 错误" --allowedTools "Bash,Edit"

# 管道组合：日志诊断流水线
cat error.log | claude -p "找出根因，按可能性排序"
```

**适用场景**：PR 自动审查、定时代码巡检、日志分析管道、批量重构脚本。原则是用 `--allowedTools` 给最小权限，而不是图省事全放开。

## 7. 结构化输出与成本核算

headless 模式配合 `--output-format json`，输出变成机器可解析的结构，还自带账单：

```bash
claude -p "列出本仓库所有对外 API" --output-format json | jq '.total_cost_usd'
```

每次调用花了多少钱一目了然。接进 CI 后建议把这个字段落进指标系统——LLM 自动化的成本不监控，迟早收到意外账单。需要实时处理输出流的场景用 `--output-format stream-json`。

## 8. 探索 → 规划 → 实现 → 提交的四段式工作流

官方最佳实践文档反复强调的节奏，对中大型任务收益巨大：

1. **探索**：先让它只读不写——"读一遍 src/auth，讲清楚现在的会话管理流程"
2. **规划**：进入计划模式（`Shift+Tab` 切换），产出实施方案，人工把关
3. **实现**：按批准的计划动手，同时给它验证手段——"跑测试，失败就修到全绿"
4. **提交**：确认无误后再让它写提交信息、开 PR

反模式是一句"把登录改成 OAuth"直接开干——没有探索和规划，它对"改到什么程度算完"的理解和你不一致，返工率极高。**给验证手段**是第 3 步的关键：有测试跑测试，没测试给它一个可复现的验证命令，它就能自己迭代到通过。

## 9. 富媒体上下文：截图、@文件、管道喂日志

四种往上下文里精准喂料的方式，比大段粘贴高效得多：

```bash
# 截图直接粘贴（Cmd+V / Ctrl+V）——UI bug 一图胜千言
# @ 引用文件，避免手动复制
@package.json 检查依赖里有没有已知漏洞的版本

# URL 直接给，Claude 会自动抓取
按照 https://code.claude.com/docs/en/hooks-guide 的写法帮我配一个 PostToolUse hook

# 会话内直接跑 shell 命令，输出自动进上下文
! npm test
```

最后那个 `!` 前缀值得单独记住：命令在你的会话里执行、输出直接落进对话，省去了"跑完再粘贴结果"的循环。

## 10. Skills：把重复流程固化成一条命令

任何你对 Claude 说过三遍以上的流程，都应该变成 skill：

```markdown
# .claude/skills/fix-issue/SKILL.md
---
name: fix-issue
description: 按标准流程修复 GitHub issue
disable-model-invocation: true   # 只允许手动触发
---

1. 用 gh issue view 读取 issue 详情
2. 定位相关代码，先复现问题
3. 实现修复，补回归测试
4. 跑全量测试，通过后总结改动
```

之后 `/fix-issue 1234` 一条命令就是完整流程。`disable-model-invocation: true` 表示只有你能触发；去掉这行则 Claude 在判断相关时会自动调用。团队把 skills 提交进仓库，新人第一天就继承全部工作流。

## 收尾：一张优先级表

| 如果你只想先改一件事 | 做这个 |
|---------------------|--------|
| 经常"找不到昨天的会话" | 技巧 1：命名 + resume |
| 怕 Claude 改崩代码 | 技巧 2：/rewind |
| 会话越聊越笨 | 技巧 3：/compact |
| CLAUDE.md 已经几百行 | 技巧 4：path-scoped rules |
| 想做自动化 | 技巧 6 + 7：headless + JSON 输出 |
| 大任务返工率高 | 技巧 8：四段式工作流 |

## 参考资料

- 最佳实践：https://code.claude.com/docs/en/best-practices
- 会话管理：https://code.claude.com/docs/en/sessions
- Checkpointing：https://code.claude.com/docs/en/checkpointing
- Headless 模式：https://code.claude.com/docs/en/headless
- 记忆系统：https://code.claude.com/docs/en/memory
- Skills：https://code.claude.com/docs/en/skills
