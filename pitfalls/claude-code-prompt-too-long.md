# Claude Code 上下文太长怎么办：Prompt is too long 会自动压缩，中转站 / DeepSeek 的 maximum context length 却不会（实测）

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-prompt-too-long?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-prompt-too-long?utm_source=github&utm_medium=referral)**

> **先回答你搜的问题**
>
> - **看到 `Prompt is too long · …`**：Claude Code 认出了这是上下文超长。多轮对话里它会**自动压缩旧对话再重发**，通常你什么都不用做；如果提示说 *single exchange cannot be compacted*，说明是系统提示、工具定义、附件本身太大，要减少 MCP 工具或附件，或者 `/clear` 重开。
> - **看到 `API Error: 400 This model's maximum context length is …`**（用 DeepSeek 或中转站时最常见）：Claude Code **不认识这句话**，不会自动压缩，之后每一轮都会报同样的错。手动输入 `/compact`，实测能恢复。
> - **想一劳永逸**：告诉 Claude Code 模型的真实上限，让它在撞墙之前就压缩：
>   - 模型 ID 不以 `claude-` 开头（如 `glm-4.6`、`deepseek-…`）：`export CLAUDE_CODE_MAX_CONTEXT_TOKENS=131072`（换成你的模型的真实上限）
>   - 中转站给的模型 ID 以 `claude-` 开头：`export CLAUDE_CODE_AUTO_COMPACT_WINDOW=131072`（最小 100000）
> - **看到 `max_tokens` 相关的超限**：不用管，Claude Code 会自动把 `max_tokens` 调小后重发。

## 问题背景

对话越聊越长，或者一次塞进很多文件，迟早会碰到上下文上限。在 Claude Code 里，这件事有两种完全不同的表现：

```
Prompt is too long · the request is ~210000 tokens (limit 200000) but this conversation is only ~12807 tokens — …
```

```
API Error: 400 This model's maximum context length is 1048576 tokens. However, you requested 1053707 tokens (1045707 in the messages, 8000 in the completion).
```

第一种有些人从来没见过，因为 Claude Code 在后台自动压缩掉了；第二种用 DeepSeek 或中转站的人会反复看到，而且 `/clear` 之前每轮都在报。区别不在模型，而在**后端返回的报错文案**。

## 问题分析

官方 [错误文档](https://code.claude.com/docs/en/errors#prompt-is-too-long) 提到了一个细节：Bedrock 的超长报错是 `Input is too long for requested model.`，*Before v2.1.217, Claude Code didn't recognize the Bedrock wording, so auto-compact never triggered on it*。也就是说，自动压缩是**按文案识别**触发的。

在 2.1.285 的二进制里能找到识别函数，原样摘录：

```js
function BWn(e){let n=e.toLowerCase();return n.includes("prompt is too long")||n.includes("input is too long for requested model")}
function jWn(e){return e.toLowerCase().includes("context window")}          // 只用于 413
function M0r(e){return e.toLowerCase().includes("input length and `max_tokens` exceed context limit")}
function kA(e){ … return BWn(e.message)||bI(e.message,"prompt_too_long")}
```

所以 Claude Code 只认这几种说法：

| 后端文案（不区分大小写） | Claude Code 怎么处理 |
|---|---|
| 包含 `prompt is too long` | 当作上下文超长 → 自动压缩 |
| 包含 `input is too long for requested model`（Bedrock） | 同上 |
| HTTP 413 且包含 `context window` | 同上 |
| `input length and \`max_tokens\` exceed context limit: A + B > C` | 解析出数字，把 `max_tokens` 调小后重发 |
| **其他文案** | 普通的 400 错误，原样显示 |

而 DeepSeek 的超长报错是 `This model's maximum context length is 1048576 tokens. However, you requested … tokens (… in the messages, … in the completion)`（站内 [DeepSeek 1M 实测](/zh/hands-on/claude-code-deepseek-1m-context) 中从 DeepSeek 的 Anthropic 兼容端点拿到的原文），OpenAI 风格的中转站透传的也是 `maximum context length` 这一类说法，**都不在识别范围内**。

...

---

**[👉 继续阅读全文：Claude Code 上下文太长怎么办：Prompt is too long 会自动压缩，中转站 / DeepSeek 的 maximum context length 却不会（实测）](https://tools.cooconsbit.com/zh/articles/claude-code-prompt-too-long?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
