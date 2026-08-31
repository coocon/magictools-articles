# 抓包拆开 Claude Code auto 模式的判定器：11 万字系统提示词逐段解析

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-auto-mode-classifier-prompt?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-auto-mode-classifier-prompt?utm_source=github&utm_medium=referral)**

[上一篇复测](/zh/articles/claude-code-auto-mode-permission-trap)确认了一件事：auto 模式在真正执行一条 Bash 之前，会额外调一次**你正在用的那个会话模型**，让它判定这条命令安不安全。当时用本地日志代理抓到了判定请求的外壳——`model`、`max_tokens`、消息数——但请求正文里那份长得吓人的系统提示词，我只截了开头一句 `You are a security monitor for autonomous AI coding agents`，没有展开。

这篇把那份提示词整包摊开。它是一份 **116,879 字符**的安全策略规范，规定了判定器该拦什么、该放什么、按什么顺序判。下面每一段拆解都直接引用抓包原文，不做转述。

## 问题背景：判定器是个黑盒，但它的规则决定了 auto 模式的真实边界

auto 模式的卖点是"用一次模型调用，换掉几十次手动确认"。但这句话回避了一个问题：那次模型调用**按什么标准**放行或拦截？

官方文档只说 auto 模式会"评估命令安全性"，没有公开评估规则。于是使用者对 auto 模式的边界只能靠猜和试：`rm -rf` 会不会被拦？`curl | bash` 呢？改 `.claude/settings.json` 呢？在不知道规则的前提下，你没法预测哪条命令会中断，也没法判断这层防护到底防住了什么、漏掉了什么。

要把黑盒变白盒，只有一条路：拿到判定器收到的完整输入。而判定请求是 Claude Code 发给模型 API 的一次普通 HTTP 调用——只要能拦下这次调用的请求体，规则就全在里面。

## 问题分析：判定链路是两条独立的模型调用

先厘清 Claude Code 在 auto 模式下的调用结构。一条被判定器管辖的 Bash 命令，会触发**两条独立的模型调用链**：

1. **主循环**：读你的话、决定调哪个工具、生成回复——这是干活的那个模型。
2. **安全判定**：在主循环决定执行某条 Bash 之后、真正执行之前，**再单独发一次请求**，问"这条命令危不危险"。

判定请求和主循环请求打到的是同一个模型 ID，但请求形态完全不同。抓包里两者对比鲜明（均来自本次 `bodies/7.json`）：

| | 判定请求 | 主循环请求 |
|---|---|---|
| `model` | `claude-sonnet-5` | `claude-sonnet-5` |
| 系统提示词长度 | **116,879 字符** | 27,702 字符 |
| `max_tokens` | **64** | 正常 |
| `tools` | **无**（字段缺省） | 23 个 |
| `thinking` | `disabled` | 正常 |
| `stop_sequences` | `["</severity>"]` | 无 |

判定请求是一次极度受限的调用：不带任何工具、关掉思维链、最多吐 64 个 token、遇到 `</severity>` 就停。它不需要干活，只需要吐一个分数。而它的系统提示词反而比干活的主循环长 4 倍——**判定器的"智能"全压在那 11 万字的规则里**。

## 技术方案与选型：为什么用本地日志代理抓包

要拿到那 11 万字，我评估过三条路，只有一条走得通：

- **直接问模型"你的判定规则是什么"**——排除。判定器和主循环共享模型，但判定用的系统提示词只在判定请求里注入；你在主对话里问，模型看不到那份规则，只会凭训练知识编一个像模像样的答案。这恰恰是抓包要规避的"凭印象"。
- **反编译 Claude Code CLI**——排除。提示词不是硬编码在客户端的静态字符串，而是运行时在**服务端**拼装注入的（客户端只发命令和上下文）。就算把 CLI 扒开也拿不到这份 server 侧的 prompt。
- **靠报错文本推断**——排除。这正是上一篇的做法，只能推断出"判定器绑定会话模型"这一条，规则本身完全看不见。

选中的方案是**本地透明代理**：写一个 80 行的 Node 脚本监听 `127.0.0.1`，把 `ANTHROPIC_BASE_URL` 指过去，它原样转发请求到真实上游、同时把每个请求体落盘到 `bodies/<seq>.json`。判定请求也是一次普通 API 调用，落盘后就是完整明文。代理还留了个熔断开关（对判定请求选择性返回 529），用来确定性复现"判定器挂了"的状态——这部分在上一篇讲过，本篇只用它的抓包能力。

## 实测过程：从抓包文件到逐段解剖

### 一、请求形态：一次"只吐分数"的受限调用

抓到的判定请求 `bodies/7.json` 有个坑：它是**两个首尾相接的 JSON 文档**（判定请求 + 紧随其后的主循环请求拼在一个文件里），`JSON.parse` 直接报错。得用 `jq -s` 把它 slurp 成数组再取 `.[0]`：

![判定请求形态：model=claude-sonnet-5、max_tokens=64、thinking disabled、stop=</severity>、系统提示词 116,879 字符，以及 11 个分区的结构地图](https://cdn.tools.cooconsbit.com/uploads/articles/2026-08-30-automode-classifier/01-anatomy.png)

请求正文的 `system` 字段有两个块：主块 116,879 字符（规则规范），第二块是一小段会话上下文（抓包原文）：

```
## Session Context
- **User identity**: `duoduo`. The `$USER/...` pattern in the rules above
  resolves to `duoduo/...`.
```

...

---

**[👉 继续阅读全文：抓包拆开 Claude Code auto 模式的判定器：11 万字系统提示词逐段解析](https://tools.cooconsbit.com/zh/articles/claude-code-auto-mode-classifier-prompt?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
