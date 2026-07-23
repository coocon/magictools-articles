---
title: "Streaming and Vision: Real-time Responses and Image Understanding"
slug: "claude-api-streaming-en"
category: "claude"
tags: [claude-api, streaming, vision, multimodal, advanced]
summary: "Master Claude API's streaming responses and vision capabilities. Build typewriter-effect real-time output and multimodal apps for image analysis and document understanding."
status: "published"
---

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

## Vision Capabilities

Claude supports image inputs and can analyze image content, extract text, understand charts, and more. Images are passed via `image` type content blocks.

### Sending Base64 Images

```python
import anthropic
import base64

client = anthropic.Anthropic()

with open("screenshot.png", "rb") as f:
    image_data = base64.standard_b64encode(f.read()).decode("utf-8")

message = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {
                "type": "image",
                "source": {
                    "type": "base64",
                    "media_type": "image/png",
                    "data": image_data
                }
            },
            {"type": "text", "text": "Describe what you see in this image"}
        ]
    }]
)

print(message.content[0].text)
```

### Sending URL Images

```python
message = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {
                "type": "image",
                "source": {
                    "type": "url",
                    "url": "https://example.com/chart.png"
                }
            },
            {"type": "text", "text": "Analyze the trends in this chart"}
        ]
    }]
)
```

### Supported Image Formats

- JPEG, PNG, GIF, WebP
- Maximum 20MB per image
- Up to 20 images per request
- Recommended to keep the shorter side under 1568 pixels for optimal performance

## Combining Streaming and Vision

You can use Streaming and Vision together to analyze images in real-time with incremental output:

```python
with client.messages.stream(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {"type": "image", "source": {"type": "base64", "media_type": "image/png", "data": image_data}},
            {"type": "text", "text": "Describe this image in detail"}
        ]
    }]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

## FAQ

### Does streaming affect output quality?

No. Streaming only changes how content is delivered, not how it is generated. The final concatenated text is identical to non-streaming mode.

### Can Vision recognize text in images?

Yes. Claude has excellent OCR capabilities and can extract text from screenshots, document photos, and handwritten notes. It also handles complex tables and formulas reasonably well.

### How do I send multiple images?

Add multiple `image` type content blocks sequentially in the `content` array. Claude can understand multiple images simultaneously and perform comparative analysis.

### How do I get token usage in streaming mode?

Token usage information is returned in the `message_delta` event, typically at the end of the stream. The Python SDK's `stream.get_final_message()` method provides the complete usage statistics directly.
