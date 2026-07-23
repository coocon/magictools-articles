---
title: "Streaming 与 Vision：实时响应与图像理解"
slug: "claude-api-streaming"
category: "claude"
tags: [claude-api, streaming, vision, multimodal, advanced]
summary: "掌握 Claude API 的流式响应和视觉能力，实现打字机效果的实时输出，以及图片分析、文档理解等多模态应用。"
status: "published"
---

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

## 视觉能力 (Vision)

Claude 支持图像输入，可以分析图片内容、提取文字、理解图表等。图片通过 `image` 类型的内容块传入。

### 发送 Base64 图片

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
            {"type": "text", "text": "请描述这张图片的内容"}
        ]
    }]
)

print(message.content[0].text)
```

### 发送 URL 图片

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
            {"type": "text", "text": "分析这张图表的趋势"}
        ]
    }]
)
```

### 支持的图片格式

- JPEG、PNG、GIF、WebP
- 单张图片最大 20MB
- 单次请求最多 20 张图片
- 建议图片短边不超过 1568 像素以获得最佳性能

## 流式 + 视觉结合

你可以同时使用 Streaming 和 Vision，实现实时分析图片并逐步输出结果：

```python
with client.messages.stream(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {"type": "image", "source": {"type": "base64", "media_type": "image/png", "data": image_data}},
            {"type": "text", "text": "详细描述这张图片"}
        ]
    }]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

## 常见问题

### Streaming 会影响输出质量吗？

不会。Streaming 只改变内容的传输方式，不影响生成质量。最终拼接的完整文本与非流式模式完全一致。

### Vision 可以识别图片中的文字吗？

可以。Claude 具备优秀的 OCR 能力，可以识别截图、文档照片、手写文字中的内容。对于复杂表格和公式也有较好的支持。

### 多张图片如何传入？

在 `content` 数组中依次添加多个 `image` 类型的内容块即可。Claude 可以同时理解多张图片并进行对比分析。

### Streaming 模式下如何获取 Token 用量？

Token 用量信息在 `message_delta` 事件中返回，通常出现在流的末尾。Python SDK 的 `stream.get_final_message()` 方法可以直接获取完整的用量统计。
