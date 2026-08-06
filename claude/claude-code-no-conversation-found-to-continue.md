---
title: "Claude Code 报 No conversation found to continue：你的 -p 会话被交互模式过滤掉了"
slug: claude-code-no-conversation-found-to-continue
category: claude
locale: zh
translationSlug: claude-code-no-conversation-found-to-continue-en
tags: [Claude Code, CLI, headless, 会话管理, 故障排查, claude-code-lab]
summary: "用 claude -p 跑完一条无头命令，接着敲 claude --continue 想进交互模式续聊，得到的却是 No conversation found to continue——但会话文件明明就在磁盘上。这是 v2.1.90 引入的回归：--resume 选择器'不展示 -p/SDK 会话'的过滤逻辑，误伤了本应无条件继续最近会话的 --continue。本文在 macOS + v2.1.220 上完整复现（GitHub issue 报告的是 Fedora，两平台坐实），给出两条实测可用的绕行：无头续聊 -p --continue 一直是通的；交互模式用 CLAUDE_CODE_ENTRYPOINT=sdk-cli 前缀可以绕过过滤、完整加载历史。"
status: published
lab:
  testedAt: "2026-08-06"
  ccVersion: "2.1.220"
  model: "claude-haiku-4-5（bug 与模型无关）"
  platform: macOS
  status: reproducible
docsUrl: https://docs.claude.com/en/docs/claude-code/cli-reference
---

## 官方文档怎么说

CLI 参考里 `--continue` 的语义非常简单：**继续当前目录下最近的一次会话**。没有任何附加条件——不区分这个会话是交互模式开的，还是 `claude -p "..."` 无头模式跑出来的。

这正是很多自动化工作流的基础假设：脚本里用 `-p` 批量干活，出问题了人再用 `--continue` 进交互模式接手排查。

## 实测发生了什么

macOS + Claude Code v2.1.220，一个干净的空目录，四步实验（终端为真实 TTY）：

```console
$ claude --max-turns 1 -p 'The answer is 42. Reply only with: OK'
OK

$ claude --continue
No conversation found to continue        # ← BUG：会话明明存在

$ claude -p --continue "What did I ask earlier?"
42                                        # ← 无头续聊完全正常

$ CLAUDE_CODE_ENTRYPOINT=sdk-cli claude --continue
# ← 交互界面正常打开，且完整加载了 -p 会话的历史
```

第 2 步和第 3 步的对比是关键：**同一份会话数据，无头模式续得上，交互模式说找不到**。会话文件好端端躺在 `~/.claude/projects/<项目路径>/` 下的 `.jsonl` 里——我们检查过，`-p` 创建的会话和交互会话存储在同一个位置、同一种格式。

## 坑在哪：一次过滤逻辑的误伤

这是 v2.1.90 引入的回归。那个版本的变更说明写着：*"Changed --resume picker to no longer show sessions created by `claude -p` or SDK invocations"*——`--resume` 的会话选择器不再展示 `-p`/SDK 创建的会话，这本身是合理的产品决策（选择器里全是脚本跑的一次性会话确实很吵）。

问题是这层过滤被**同时应用到了 `--continue`**。而 `--continue` 的语义是"继续最近的会话"，压根不该关心会话怎么创建的。于是：

- 目录里最近（或仅有）的会话是 `-p` 创建的 → 交互 `--continue` 把它过滤掉 → 报 `No conversation found to continue`
- 无头路径 `-p --continue` 在早期 issue（#43013）后被修过 → 一直正常
- 数据层无损：会话元数据里的 entrypoint 字段、jsonl 内容全部完好

GitHub 上的 issue（[#82536](https://github.com/anthropics/claude-code/issues/82536)，报告环境 Fedora + tmux）与我们在 macOS 上的复现相互印证——这是跨平台的逻辑层 bug，不是环境问题。从 v2.1.90 到 v2.1.220+ 持续存在。

## 两条实测可用的绕行

**1. 只需要续着问一句 → 用无头续聊（最简单）**

```bash
claude -p --continue "接着刚才的结果，帮我看下 X"
```

**2. 需要进交互模式接手 → 加环境变量前缀**

```bash
CLAUDE_CODE_ENTRYPOINT=sdk-cli claude --continue
```

这个变量让 CLI 以 SDK 入口的身份启动，绕过交互模式的会话过滤。实测交互界面正常打开、历史完整加载，后续操作与普通会话无异。发现此绕行的是 issue 作者，我们在 macOS 上验证有效。

另外注意一个排查陷阱：如果你通过管道或重定向运行 `claude --continue`（stdout 非 TTY），报错文案会变成 `No deferred tool marker found in the resumed session`——同一个 bug 的另一张面孔，别被它带偏去查什么 "tool marker"。

## 适用边界

- 影响 v2.1.90 起的所有版本（本文实测 v2.1.220，issue 报告同版本，截稿时官方未修复、无关联 PR）
- 只影响**交互模式**的 `--continue`/`--resume` 对 `-p`/SDK 会话的发现；无头 `-p --continue` 不受影响
- macOS 与 Fedora 双平台复现，与模型无关
- 按过滤机制推断（此条未单独实测）：目录里同时存在交互会话和更新的 `-p` 会话时，交互 `--continue` 会跳过 `-p` 会话接上较旧的交互会话——不报错，但接的不是真正的"最近一次"，比直接报错更隐蔽
