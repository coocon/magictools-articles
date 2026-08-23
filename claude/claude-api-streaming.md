# Streaming 与 Vision：实时响应与图像理解

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-api-streaming?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-api-streaming?utm_source=github&utm_medium=referral)**

## 流式响应 (Streaming)

默认情况下，API 会等到完整响应生成后才返回。Streaming 模式让你可以在生成过程中逐步接收内容，实现类似打字机的实时输出效果，大幅提升用户体验。

### Python 流式示例

```python
import anthropic

client = anthropic.Anthropic()

with client.messages.stream(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[{"role": "user", "content": "写一首关于编程的短诗"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

### TypeScript 流式示例

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const stream = client.messages.stream({
  model: "claude-sonnet-4-20250514",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Write a short poem about coding" }],
});

for await (const event of stream) {
  if (event.type === "content_block_delta" && event.delta.type === "text_delta") {
    process.stdout.write(event.delta.text);
  }
}
```

### 关键事件类型

- `message_start`：消息开始，包含模型信息
- `content_block_start`：内容块开始
- `content_block_delta`：增量文本内容（`text_delta`）
- `content_block_stop`：内容块结束
- `message_delta`：消息级别更新（包含 `stop_reason` 和 `usage`）
- `message_stop`：消息完成

...

---

**[👉 继续阅读全文：Streaming 与 Vision：实时响应与图像理解](https://tools.cooconsbit.com/zh/articles/claude-api-streaming?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
