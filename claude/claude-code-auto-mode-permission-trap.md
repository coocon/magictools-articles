# Claude Code 的 auto 模式是用模型审模型的——模型一挂，连 cat 都跑不了

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-auto-mode-permission-trap?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-auto-mode-permission-trap?utm_source=github&utm_medium=referral)**

会话跑到一半，一条再普通不过的命令被拦了下来：

```
claude-opus-5[1m] is temporarily unavailable, so auto mode cannot
determine the safety of Bash right now.
Note: reading files, searching code, and other read-only operations
do not require the classifier and can still be used.
```

反直觉的地方在最后那句：读文件、搜代码一切正常，唯独跑不了命令。如果只是"模型挂了"，不该是这个半瘫的样子——读文件同样要模型来发起。这个不对称本身就是最有价值的诊断线索。

接下来的排查里，我提出了三个听起来都很合理的假设，又把它们逐个推翻。写下来的价值不在最终那条修复命令，而在推翻的过程。

## 报错读起来像模型挂了，其实是判定链断了

拆开看，这里有两条独立的模型调用链：

1. **主循环**：模型读你的话、决定调哪个工具——这条一直是通的，所以 Read、Grep 照常工作
2. **安全判定**：auto 模式在真正执行 Bash 之前，**额外发一次请求给模型**，问"这条命令安全吗"——断的是这条

判定链一断，权限系统就拿不到"安全/不安全"的结论。它不能默认放行，于是全拦。而只读工具压根不进这个环节，所以毫发无伤。

**判定器用的就是你正在用的那个模型。** 这一点有直接证据：报错里点名的模型 ID，会随你切换会话模型而同步变化。我把配置从 `opus[1m]` 切到 `opus` 之后，报错文本也从 `claude-opus-5[1m]` 变成了 `claude-opus-5`。判定器不是某个独立的小模型，就是你会话里这一个。

## auto 模式到底在信任什么

把 Claude Code 的几种权限模式按"信任来源"排一下，这次故障的位置就清楚了：

| 模式 | 谁来决定放不放行 | 模型不可用时 |
|------|------------------|--------------|
| 默认（default） | 你，逐条确认 | 不受影响 |
| 接受编辑（acceptEdits） | 你事先的一次性授权，只覆盖文件编辑 | 不受影响 |
| 计划模式（plan） | 规则：一律只读 | 不受影响 |
| **自动（auto）** | **模型，逐条判定** | **判定链断裂，全面拦截** |
| 绕过（bypassPermissions） | 没人 | 不受影响（也没有保护） |

auto 模式的交易很划算：用一次模型调用，换掉几十次"要执行这条命令吗？"的打断。代价直到出事那天才显形——**你把权限系统的可用性，绑死在了模型的可用性上**。其他几种模式的判定依据要么是你本人，要么是静态规则，都不需要网络往返；只有 auto 需要。

这不是设计缺陷，是一笔明码标价的取舍。问题在于这笔账在顺利的时候完全不可见，所以撞上的人第一反应总是"服务挂了，等等吧"，而不是"我该换个模式"。

## 假设一：是那个 1M 变体的锅

我的 `~/.claude/settings.json` 里写着 `"model": "opus[1m]"`，同时 `ANTHROPIC_BASE_URL` 指向一个第三方 API 中转。这两条凑在一起，指向一个很顺的解释：第三方中转对**非常规模型 ID** 的支持普遍最薄弱——1M 上下文变体、预览版、带后缀的特化型号，往往在官方端点可用、在中转上缺失。判定请求打到一个中转不认识的模型 ID，自然拿不到结果。

...

---

**[👉 继续阅读全文：Claude Code 的 auto 模式是用模型审模型的——模型一挂，连 cat 都跑不了](https://tools.cooconsbit.com/zh/articles/claude-code-auto-mode-permission-trap?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
