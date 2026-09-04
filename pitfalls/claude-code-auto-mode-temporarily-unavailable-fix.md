# Claude Code 报错 temporarily unavailable, so auto mode cannot determine the safety of bash 怎么解决

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-auto-mode-temporarily-unavailable-fix?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-auto-mode-temporarily-unavailable-fix?utm_source=github&utm_medium=referral)**

## 现象

在 Claude Code 的 auto 模式（自动批准权限模式）下干活，某次工具调用突然被拦，终端里出现类似这样的提示：

> error: claude-opus-5[1m] is temporarily unavailable, so auto mode cannot determine the safety of bash right now. wait a moment and then try this action again. if it keeps failing, continue with other tasks that don't require this action and come back to it later. note: reading files, searching code, and other read-only operations do not require the classifier and can still be used.

模型名可能是 `claude-opus-4-8`、`claude-sonnet-4-6`、`claude-sonnet-5`、`claude-fable-5`，也可能是 `glm-5.3`、`k3` 这类第三方模型；`bash` 的位置可能是 `write`、`edit`、`webfetch`、`websearch`、`skill`、`croncreate` 甚至某个 MCP 工具名；括号里的原因可能是 `timed out`、`server error`、`rate-limited`、`connection failed`，或者干脆没有括号。**这些全是同一个问题**，处理方式一样，下文有完整变体速查。

## 根因：挂掉的是"判定器"，不是你的命令被判危险

auto 模式的本质是把"人工点确认"换成"模型判定器"：每次要执行 Bash、写文件这类**有副作用**的动作前，Claude Code 会额外发起一次受限的模型调用，让会话模型扮演安全监视器（system prompt 开头就是 "You are a security monitor for autonomous AI coding agents"），只吐一个危险度判分。我们此前[用本地日志代理把这次判定请求整包抓下来逐段解析过](/zh/hands-on/claude-code-auto-mode-classifier-prompt)，机制细节可以看那篇。

...

---

**[👉 继续阅读全文：Claude Code 报错 temporarily unavailable, so auto mode cannot determine the safety of bash 怎么解决](https://tools.cooconsbit.com/zh/articles/claude-code-auto-mode-temporarily-unavailable-fix?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
