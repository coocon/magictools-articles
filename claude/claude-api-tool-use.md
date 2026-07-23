---
title: "Claude Tool Use：让 AI 调用外部工具和函数"
slug: "claude-api-tool-use"
category: "claude"
tags: [claude-api, tool-use, function-calling, advanced]
summary: "学习 Claude 的 Tool Use 功能，让 AI 自主调用你定义的函数和 API，实现天气查询、数据库操作、网页搜索等实际场景。"
status: "published"
---

## 什么是 Tool Use

Tool Use（也称 Function Calling）让 Claude 能够调用你预定义的外部函数。Claude 会根据对话内容判断何时需要使用工具，生成符合你定义 Schema 的参数，你在本地执行函数后将结果返回给 Claude，它再基于结果生成最终回复。

## 工作流程

1. 你在请求中定义可用工具（名称、描述、参数 JSON Schema）
2. Claude 判断需要使用工具时，返回 `tool_use` 类型的内容块
3. 你在本地执行对应函数，获取结果
4. 将结果作为 `tool_result` 发回 Claude
5. Claude 基于工具结果生成最终回复

## Python 示例

```python
import anthropic
import json

client = anthropic.Anthropic()

# 定义工具
tools = [
    {
        "name": "get_weather",
        "description": "获取指定城市的当前天气信息",
        "input_schema": {
            "type": "object",
            "properties": {
                "city": {
                    "type": "string",
                    "description": "城市名称，如 北京、上海"
                }
            },
            "required": ["city"]
        }
    }
]

# 模拟天气函数
def get_weather(city: str) -> str:
    return json.dumps({"city": city, "temp": "22°C", "condition": "晴"})

# 第一次请求
response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    tools=tools,
    messages=[{"role": "user", "content": "北京今天天气怎么样？"}]
)

# 处理 tool_use
if response.stop_reason == "tool_use":
    tool_block = next(b for b in response.content if b.type == "tool_use")
    result = get_weather(**tool_block.input)

    # 返回工具结果
    final = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=1024,
        tools=tools,
        messages=[
            {"role": "user", "content": "北京今天天气怎么样？"},
            {"role": "assistant", "content": response.content},
            {"role": "user", "content": [
                {"type": "tool_result", "tool_use_id": tool_block.id, "content": result}
            ]}
        ]
    )
    print(final.content[0].text)
```

## TypeScript 示例

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const tools: Anthropic.Tool[] = [
  {
    name: "get_weather",
    description: "Get current weather for a city",
    input_schema: {
      type: "object" as const,
      properties: {
        city: { type: "string", description: "City name" }
      },
      required: ["city"]
    }
  }
];

const response = await client.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 1024,
  tools,
  messages: [{ role: "user", content: "What's the weather in Tokyo?" }],
});

// Handle tool_use blocks in response.content
```

## tool_choice 参数

通过 `tool_choice` 控制工具使用行为：

- `{"type": "auto"}`：默认值，Claude 自行决定是否使用工具
- `{"type": "any"}`：强制使用某个工具
- `{"type": "tool", "name": "get_weather"}`：强制使用指定工具

## 最佳实践

- **描述要清晰**：工具描述越详细，Claude 越能准确判断何时使用
- **处理错误**：工具执行失败时，在 `tool_result` 中返回 `is_error: true` 和错误信息
- **限制工具数量**：工具过多会影响准确率，建议控制在 10 个以内
- **参数校验**：始终验证 Claude 生成的参数，再执行实际函数

## 常见问题

### Tool Use 和直接在 Prompt 中要求输出 JSON 有什么区别？

Tool Use 提供结构化的交互协议，Claude 会严格按照你定义的 JSON Schema 生成参数，可靠性远高于让模型自行输出 JSON。此外 Tool Use 支持多轮工具调用，适合复杂的多步骤任务。

### 一次请求可以调用多个工具吗？

可以。Claude 可能在一次响应中返回多个 `tool_use` 块，你需要依次执行每个工具并将所有结果一起返回。

### Tool Use 会增加费用吗？

工具定义会计入输入 Token，因此工具越多、描述越长，费用越高。建议精简工具描述，只在需要时传入相关工具。
