---
title: "Claude Tool Use: Let AI Call External Tools and Functions"
slug: "claude-api-tool-use-en"
category: "claude"
tags: [claude-api, tool-use, function-calling, advanced]
summary: "Learn Claude's Tool Use feature to let AI autonomously call your defined functions and APIs for weather queries, database operations, web searches, and more."
status: "published"
---

## What is Tool Use

Tool Use (also known as Function Calling) enables Claude to call external functions you define. Claude determines when a tool is needed based on the conversation, generates parameters matching your JSON Schema, you execute the function locally and return the result, and Claude generates a final response based on that result.

## How It Works

1. Define available tools in your request (name, description, parameter JSON Schema)
2. When Claude decides a tool is needed, it returns a `tool_use` content block
3. Execute the corresponding function locally and get the result
4. Send the result back as a `tool_result` message
5. Claude generates the final response based on the tool result

## Python Example

```python
import anthropic
import json

client = anthropic.Anthropic()

# Define tools
tools = [
    {
        "name": "get_weather",
        "description": "Get current weather information for a specified city",
        "input_schema": {
            "type": "object",
            "properties": {
                "city": {
                    "type": "string",
                    "description": "City name, e.g. Tokyo, New York"
                }
            },
            "required": ["city"]
        }
    }
]

# Mock weather function
def get_weather(city: str) -> str:
    return json.dumps({"city": city, "temp": "22°C", "condition": "sunny"})

# First request
response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    tools=tools,
    messages=[{"role": "user", "content": "What's the weather in Tokyo?"}]
)

# Handle tool_use
if response.stop_reason == "tool_use":
    tool_block = next(b for b in response.content if b.type == "tool_use")
    result = get_weather(**tool_block.input)

    # Return tool result
    final = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=1024,
        tools=tools,
        messages=[
            {"role": "user", "content": "What's the weather in Tokyo?"},
            {"role": "assistant", "content": response.content},
            {"role": "user", "content": [
                {"type": "tool_result", "tool_use_id": tool_block.id, "content": result}
            ]}
        ]
    )
    print(final.content[0].text)
```

## TypeScript Example

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

## The tool_choice Parameter

Control tool usage behavior with `tool_choice`:

- `{"type": "auto"}`: Default — Claude decides whether to use tools
- `{"type": "any"}`: Force Claude to use some tool
- `{"type": "tool", "name": "get_weather"}`: Force a specific tool

## Best Practices

- **Write clear descriptions**: The more detailed your tool descriptions, the better Claude can determine when to use them
- **Handle errors**: When tool execution fails, return `is_error: true` with an error message in the `tool_result`
- **Limit tool count**: Too many tools reduce accuracy — aim for 10 or fewer
- **Validate parameters**: Always validate Claude's generated parameters before executing actual functions

## FAQ

### How is Tool Use different from asking Claude to output JSON in the prompt?

Tool Use provides a structured interaction protocol where Claude strictly generates parameters according to your JSON Schema definition, making it far more reliable than freeform JSON output. Additionally, Tool Use supports multi-turn tool calls, making it suitable for complex multi-step tasks.

### Can Claude call multiple tools in a single request?

Yes. Claude may return multiple `tool_use` blocks in a single response. You need to execute each tool and return all results together.

### Does Tool Use cost extra?

Tool definitions count toward input tokens, so more tools and longer descriptions mean higher costs. Keep descriptions concise and only include relevant tools in each request.
