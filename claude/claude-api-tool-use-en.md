# Claude Tool Use: Let AI Call External Tools and Functions

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-api-tool-use-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-api-tool-use-en?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 Continue reading: Claude Tool Use: Let AI Call External Tools and Functions](https://tools.cooconsbit.com/en/articles/claude-api-tool-use-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
