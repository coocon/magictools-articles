# Claude Code 接 DeepSeek / GLM / Kimi 开 auto 模式：temporarily unavailable 括号里的原因怎么看，哪些重试没用（故障注入实测）

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-auto-mode-third-party-models?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-auto-mode-third-party-models?utm_source=github&utm_medium=referral)**

> **先回答你搜的问题**
>
> - **报错里的模型名为什么是 `glm-5.3` / `deepseek-v4-pro` / `k3`**：auto 模式每执行一个有副作用的动作（Bash、写文件等），都会用**你的会话模型**额外发一次「安全判定」请求，发到你配置的**同一个后端**。报错里的名字就是这个判定器，也就是你自己的会话模型。
> - **先看括号**：`(server error)`、`(rate-limited)`、`(overloaded)`、`(connection failed)` 是后端临时出问题，Claude Code 已经自动试了 5 次；`(timed out)` 是判定请求 60 秒内没返回。
> - **没有括号最要紧**：说明后端直接拒收了判定请求（4xx），Claude Code 只试 1 次，**重试多少遍结果都一样**，要去查后端配置。
> - **报 `Auto mode could not evaluate this action`**：后端返回了，但模型没按格式给出判定。连续 3 次后，auto 模式会退回人工确认。
> - **想单独给判定器换个模型**：实测做不到。`CLAUDE_CODE_AUTO_MODE_MODEL` 在 2.1.285 不生效，要换只能换会话模型。

## 问题背景

[《temporarily unavailable, so auto mode cannot determine the safety of bash 怎么解决》](/zh/articles/claude-code-auto-mode-temporarily-unavailable-fix)是本站最近 28 天搜索流量最高的一篇。看 Bing 带来的搜索词会发现，相当一部分报错里的模型名不是 Claude，而是 `glm-5.3`、`glm5.2`、`glm-5.3-flash`、`claude-glm-5.3`，还有带 `cc switch` 前缀的。也就是说，很多人是通过 cc-switch、claude-code-router 之类的工具，把 Claude Code 接到智谱、Kimi、DeepSeek 等第三方后端后撞上这个报错的。

那篇文章对这类场景只写了一句「换健康的后端」。这类用户真正想知道的是：

1. 第三方模型到底能不能跑 auto 模式？判定请求长什么样，要花多少钱？
2. 括号里那几种原因各对应什么故障？哪些等一等就好，哪些怎么重试都没用？
3. 能不能只给判定器换一个更快、更便宜的模型？

## 问题分析

先把机制讲清楚。auto 模式的思路是用一次模型调用代替人工点确认：执行有副作用的动作之前，先让模型判定这个动作安不安全（详见[《抓包拆开 Claude Code auto 模式的判定器》](/zh/hands-on/claude-code-auto-mode-classifier-prompt)）。

这次实测发现，2.1.285 的判定其实分两条路。调试日志里写得很明白，接第三方后端时，第一次判定就会出现这三行：

```
[server-classifier] the platform gave no classification for this request (server_no_result); treating that as this deployment's answer: auto mode classifies locally for the rest of this session and stops sending the classifier context
[server-classifier] Bash: no server verdict for this call (server_no_result); the local classifier decides it
[server-classifier] auto mode fell back to billed classifier requests; no dialog surface to warn on, continuing in auto mode
```

意思是：Claude Code 先看平台有没有在服务端给出判定结果，第三方后端当然不会给（`server_no_result`），于是本会话剩下的时间都改用**本地判定**，也就是日志里说的「billed classifier requests」：每个需要判定的动作，单独向你的后端发一次计费请求。第三方用户看到的这条报错，全部来自这条本地判定链路。

用 2.1.285 的源码对照，括号里的原因由一个函数根据 HTTP 状态码和错误类型拼出来：

```js
if(e===429)return" (rate-limited)";
if(e===529)return" (overloaded)";
if(e!==void 0&&e>=500&&e<600)return" (server error)";
if(n==="wall_clock_timeout"||n==="connection_timeout"||...)return" (timed out)";
if(n==="connection_error")return" (connection failed)";
```

源码只能说明「理论上会怎样」。重试几次、等多久、模型收到报错后会怎么做，必须实际跑出来才知道。

## 技术方案与选型

**做法：在 Claude Code 和真实后端之间放一个故障注入代理。** Claude Code 的 `ANTHROPIC_BASE_URL` 指向本地代理；主循环请求原样转发给真实后端，判定请求（系统提示词以 `You are a security monitor for autonomous AI coding agents` 开头）按用例改成返回 500、429、挂起、断连等。这样主循环是真模型在跑，故障只落在判定器上。

...

---

**[👉 继续阅读全文：Claude Code 接 DeepSeek / GLM / Kimi 开 auto 模式：temporarily unavailable 括号里的原因怎么看，哪些重试没用（故障注入实测）](https://tools.cooconsbit.com/zh/articles/claude-code-auto-mode-third-party-models?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
