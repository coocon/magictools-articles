# Fable 5 明明是 1M 上下文，状态栏却显示 200k：别急着换工具，先抓一份现场数据

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-statusline-context-window-pitfall?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-statusline-context-window-pitfall?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：Fable 5 明明是 1M 上下文，状态栏却显示 200k：别急着换工具，先抓一份现场数据](https://tools.cooconsbit.com/zh/articles/claude-code-statusline-context-window-pitfall?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
