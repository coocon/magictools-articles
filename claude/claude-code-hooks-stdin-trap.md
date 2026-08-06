---
title: "Claude Code Hooks 的 stdin 陷阱：python heredoc 会吃掉你的 hook JSON"
slug: claude-code-hooks-stdin-trap
category: claude
locale: zh
translationSlug: claude-code-hooks-stdin-trap-en
tags: [Claude Code, hooks, PostToolUse, shell, python, 故障排查, claude-code-lab]
summary: "官方文档说 hook 通过 stdin 接收 JSON 输入，这没错。但如果你在 hook 脚本里用 python heredoc（python3 - <<'EOF'）来解析这份 JSON，会掉进一个静默失败的坑：heredoc 把 python 的 stdin 重定向成了脚本本身，json.load(sys.stdin) 读到的永远是空。更糟的是，按最佳实践写的 hook 会吞掉一切异常静默退出——你不会看到任何报错，只会发现 hook '好像没生效'。这篇记录真实踩坑过程、两行代码的修复方案，和一个通用教训：静默容错的代码，调试时是你自己的敌人。"
status: published
lab:
  testedAt: "2026-08-05"
  ccVersion: "2.1.220"
  model: "claude-fable-5"
  platform: macOS
  status: reproducible
docsUrl: https://docs.claude.com/en/docs/claude-code/hooks
---

## 官方文档怎么说

Claude Code 的 hooks 机制很直接：在 `settings.json` 里注册一条命令，事件（比如 `PostToolUse`）触发时，Claude Code 会启动这个命令，并**把事件的 JSON 数据通过 stdin 传给它**。文档原话很简洁——hook 从 stdin 读 JSON，处理，按需返回退出码。

看起来是 Unix 管道的标准玩法，实现一个"自动记录失败命令"的 hook 应该十分钟搞定。

## 实测发生了什么

我要做的事：每当 Bash 命令失败，把它记到 `tasks/lessons-inbox.md` 收件箱里（去重、过滤良性失败）。逻辑不复杂但 shell 写起来啰嗦，自然的选择是让 python 干活，于是第一版 hook 长这样：

```sh
#!/bin/sh
python3 - <<'PYEOF' 2>/dev/null
import json, sys

data = json.load(sys.stdin)   # 读 hook 传来的 JSON —— 你以为
cmd = data.get("tool_input", {}).get("command", "")
# ... 判定失败、写收件箱 ...
PYEOF
exit 0
```

跑起来毫无报错。然后连续触发了好几条注定失败的命令——收件箱文件始终是空的。没有报错、没有日志、`settings.json` 配置检查了三遍没问题，hook 就是"没生效"。

## 坑在哪：heredoc 重定向了 stdin

拆开这条命令的数据流就清楚了：

1. Claude Code 启动 hook 进程，把事件 JSON 接到进程的 **stdin** 上
2. `python3 -` 的意思是"从 stdin 读**程序本身**"
3. `<<'PYEOF'` heredoc 把 python 的 stdin **重定向**成了 heredoc 的内容——也就是那段 python 脚本

于是 python 的 stdin 被脚本文本占满，读完程序后已是 EOF。脚本里的 `json.load(sys.stdin)` 读到空输入，抛 `JSONDecodeError`——而这个异常被 `2>/dev/null` 和"hook 永不阻塞"的容错设计**完整吞掉**了。Claude Code 侧看到 hook 正常退出（exit 0），一切"成功"。

真正的 hook JSON 呢？它还挂在外层 sh 进程的 stdin 上，但 heredoc 生效后 python 已经不可能读到它了。

这个坑的恶心之处是**三层静默的叠加**：heredoc 语义上完全合法（shell 不报错）、python 异常被重定向吞掉（运行时不报错）、hook 按最佳实践 exit 0（Claude Code 不报错）。每一层单独看都是正确设计，叠在一起就是一个无声的黑洞。

## 修复：先落盘，再传路径

修复只要两行——在 python 启动**之前**，先用 `cat` 把外层进程的 stdin 消费掉、落到临时文件，再把文件路径通过环境变量传进去：

```sh
#!/bin/sh
TMP_IN="$(mktemp)" || exit 0
cat > "$TMP_IN" 2>/dev/null || true          # 先把 hook JSON 从 stdin 接下来
CL_HOOK_INPUT="$TMP_IN" python3 - <<'PYEOF' 2>/dev/null
import json, os, sys

try:
    data = json.load(open(os.environ["CL_HOOK_INPUT"], encoding="utf-8"))
except Exception:
    sys.exit(0)

if data.get("tool_name") != "Bash":
    sys.exit(0)
# ... 失败判定、良性过滤、按命令哈希去重、追加收件箱 ...
PYEOF
rm -f "$TMP_IN"
exit 0
```

改完立刻生效：失败命令一条条落进收件箱，去重和过滤都正常。

顺带附上这个 hook 完整版里几个值得抄的设计（都是实际跑了一段时间验证过的）：

- **永不阻塞**：任何异常路径都 `exit 0`，包括 `mktemp` 失败——hook 挂了不能连累主流程
- **良性失败过滤**：`grep` / `rg` / `diff` / `test` 非零退出是正常语义，不记
- **按命令哈希去重**：同一条命令反复失败只记一次，收件箱不膨胀

## 适用边界

- 只影响"hook 脚本内用 heredoc 给解释器喂脚本"的写法——`python3 - <<EOF`、`node - <<EOF` 同理都会中招；如果你的 python 脚本是独立文件（`python3 hook.py`），stdin 原样传递，没有这个问题
- `python3 -c '一行代码'` 不占 stdin，短逻辑可以用它绕开；但超过几行就不可维护了，先落盘方案更稳
- 本文实测于文首标注的环境；hooks 通过 stdin 传 JSON 是文档明确的稳定契约，此坑源于 shell 语义而非 Claude Code 版本行为，预期长期有效

## 通用教训

hook 这类"配角代码"通常被要求静默容错——这没错，但**开发调试期请先把 `2>/dev/null` 摘掉**。这次如果早点看到 `JSONDecodeError: Expecting value`，十分钟就能定位；带着全套静默设计去调试，多花了一个数量级的时间。容错是给生产的，不是给排障的。
