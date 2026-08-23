# Streaming and Vision: Real-time Responses and Image Understanding

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-api-streaming-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-api-streaming-en?utm_source=github&utm_medium=referral)**

## Streaming Responses

By default, the API waits until the full response is generated before returning. Streaming mode lets you receive content incrementally as it is generated, creating a typewriter-like real-time output effect that significantly improves user experience.

### Python Streaming Example

```python
import anthropic

client = anthropic.Anthropic()

with client.messages.stream(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Write a short poem about coding"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

### TypeScript Streaming Example

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

### Key Event Types

- `message_start`: Message begins, includes model info
- `content_block_start`: Content block begins
- `content_block_delta`: Incremental text content (`text_delta`)
- `content_block_stop`: Content block ends
- `message_delta`: Message-level update (includes `stop_reason` and `usage`)
- `message_stop`: Message complete

...

---

**[👉 Continue reading: Streaming and Vision: Real-time Responses and Image Understanding](https://tools.cooconsbit.com/en/articles/claude-api-streaming-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
