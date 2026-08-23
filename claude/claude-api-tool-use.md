# Claude Tool Use：让 AI 调用外部工具和函数

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-api-tool-use?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-api-tool-use?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：Claude Tool Use：让 AI 调用外部工具和函数](https://tools.cooconsbit.com/zh/articles/claude-api-tool-use?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
