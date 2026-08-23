# Claude Code Hooks 的 stdin 陷阱：python heredoc 会吃掉你的 hook JSON

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-hooks-stdin-trap?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-hooks-stdin-trap?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：Claude Code Hooks 的 stdin 陷阱：python heredoc 会吃掉你的 hook JSON](https://tools.cooconsbit.com/zh/articles/claude-code-hooks-stdin-trap?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
