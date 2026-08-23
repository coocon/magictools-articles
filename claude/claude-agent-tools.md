# Agent 工具编排：多步推理与任务分解

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-agent-tools?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-agent-tools?utm_source=github&utm_medium=referral)**

## 工具链：串联多步操作

在实际场景中，Agent 往往需要将多个工具串联起来——用工具 A 的输出作为工具 B 的输入。Agent SDK 天然支持这种模式，模型会自动规划调用顺序。

```python
from claude_agent_sdk import Agent, tool

@tool
def fetch_data(url: str) -> str:
    """从指定 URL 抓取数据"""
    return '{"revenue": 5000000, "growth": "15%"}'

@tool
def analyze_data(data: str) -> str:
    """分析数据并生成报告"""
    return f"分析结果：数据显示收入增长稳健，同比增长 15%。"

@tool
def format_report(analysis: str, format: str) -> str:
    """将分析结果格式化为指定格式"""
    return f"# 季度报告\n\n{analysis}\n\n生成格式：{format}"

agent = Agent(
    name="数据分析师",
    instructions="你是数据分析专家。按顺序：抓取数据、分析数据、生成格式化报告。",
    tools=[fetch_data, analyze_data, format_report],
)

result = agent.run("分析 https://api.example.com/q4-data 的数据，生成 Markdown 报告")
```

## 多 Agent 协作：Handoff 模式

当任务涉及多个专业领域时，可以通过 Handoff 让不同 Agent 各司其职：

```python
from claude_agent_sdk import Agent, tool

researcher = Agent(
    name="研究员",
    instructions="你负责搜索和收集信息，完成后交给编辑。",
    tools=[search_web],
)

editor = Agent(
    name="编辑",
    instructions="你负责整理和润色内容，输出最终文章。",
    tools=[format_report],
)

orchestrator = Agent(
    name="协调者",
    instructions="协调研究员和编辑完成内容生产任务。",
    handoffs=[researcher, editor],
)

result = orchestrator.run("写一篇关于量子计算最新进展的文章")
```

...

---

**[👉 继续阅读全文：Agent 工具编排：多步推理与任务分解](https://tools.cooconsbit.com/zh/articles/claude-agent-tools?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
