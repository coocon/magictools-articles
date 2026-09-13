# Claude Code 报错 temporarily unavailable, so auto mode cannot determine the safety of bash 怎么解决

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-auto-mode-temporarily-unavailable-fix?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-auto-mode-temporarily-unavailable-fix?utm_source=github&utm_medium=referral)**

## 现象

在 Claude Code 的 auto 模式（自动批准权限模式）下干活，某次工具调用突然被拦，终端里出现类似这样的提示：

> error: claude-opus-5[1m] is temporarily unavailable, so auto mode cannot determine the safety of bash right now. wait a moment and then try this action again. if it keeps failing, continue with other tasks that don't require this action and come back to it later. note: reading files, searching code, and other read-only operations do not require the classifier and can still be used.

模型名可能是 `claude-opus-4-8`、`claude-sonnet-4-6`、`claude-sonnet-5`、`claude-fable-5`，也可能是 `glm-5.3`、`k3` 这类第三方模型；`bash` 的位置可能是 `write`、`edit`、`webfetch`、`websearch`、`skill`、`croncreate` 甚至某个 MCP 工具名；括号里的原因可能是 `timed out`、`server error`、`rate-limited`、`connection failed`，或者干脆没有括号。**这些全是同一个问题**，处理方式一样，下文有完整变体速查。

## 根因：挂掉的是"判定器"，不是你的命令被判危险

auto 模式的本质是把"人工点确认"换成"模型判定器"：每次要执行 Bash、写文件这类**有副作用**的动作前，Claude Code 会额外发起一次受限的模型调用，让会话模型扮演安全监视器（system prompt 开头就是 "You are a security monitor for autonomous AI coding agents"），只吐一个危险度判分。我们此前[用本地日志代理把这次判定请求整包抓下来逐段解析过](/zh/hands-on/claude-code-auto-mode-classifier-prompt)，机制细节可以看那篇。

看懂机制，这条报错就好理解了：

- **报错说的是判定器这次调用失败了**——模型服务暂时不可用/超时/限流，判分拿不到。
- **拿不到判分时，Claude Code 的选择是"宁可不做"**：既不放行也不判罪，让你稍后重试。所以这不是对你命令的安全判定（新版报错文案甚至直接写了 "This is not a judgement about the action"）。
- **只读操作（读文件、搜代码）不需要判分**，所以照常能用——这也解释了为什么卡住的总是 bash/write/edit 这类动作。

## 解决：按顺序试这四步

1. **等几秒，原样重试**。绝大多数情况是模型服务的瞬时抖动（高峰期尤其常见），重试一两次就过了。
2. **先干别的**。让 Claude 继续做只读类工作（读代码、分析、规划），过几分钟再回头执行被卡的动作——报错文案自己推荐的就是这个策略。
3. **持续失败：换个会话模型**。判定器用的就是你的会话模型——`/model` 切到另一个可用模型，判定器会跟着换。如果 `claude-opus-5` 在过载，切 `claude-sonnet-5` 往往立刻恢复。用第三方路由（报错里出现 `glm-5.3`、`k3` 等名字的场景）同理：切回官方模型或换一个健康的后端。
4. **还不行：退出 auto 模式**。切回默认权限模式（每次动作人工确认），判定器就不在链路里了，动作照常执行——代价是你要自己点确认。确认完这波工作，再切回 auto。

...

---

**[👉 继续阅读全文：Claude Code 报错 temporarily unavailable, so auto mode cannot determine the safety of bash 怎么解决](https://tools.cooconsbit.com/zh/articles/claude-code-auto-mode-temporarily-unavailable-fix?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
