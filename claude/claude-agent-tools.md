---
title: "Agent 工具编排：多步推理与任务分解"
slug: claude-agent-tools
category: claude
tags: [agent-sdk, tool-orchestration, multi-agent, advanced]
summary: "深入学习 Agent SDK 的工具编排能力，掌握多步推理、任务分解和多 Agent 协作模式，构建能处理复杂任务的智能系统。"
status: published
---

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

## 任务分解策略

处理复杂任务的关键是合理分解。常见模式包括：

- **编排者模式**：一个主 Agent 拆分任务，分配给专业子 Agent
- **流水线模式**：Agent 按顺序接力处理，每个 Agent 负责一个阶段
- **并行模式**：多个 Agent 同时处理独立子任务，最后汇总结果

## 步骤间的校验与容错

在多步流程中，每一步之间添加校验逻辑至关重要：

```python
@tool
def validate_output(data: str, schema: str) -> str:
    """校验上一步输出是否符合预期格式"""
    if not data or len(data) < 10:
        return "校验失败：数据为空或过短，请重新执行上一步"
    return "校验通过"
```

当某一步失败时，Agent 可以自动重试或切换备选方案，确保整体流程的健壮性。

## 常见问题

### Handoff 和直接调用多个工具有什么区别？

Handoff 是将控制权完全交给另一个 Agent，该 Agent 有自己独立的指令和工具集。而多工具调用是同一个 Agent 在自己的上下文中使用不同工具。Handoff 适合专业分工明确的场景。

### 多 Agent 系统如何共享上下文？

Agent 之间通过 Handoff 传递消息历史来共享上下文。协调者 Agent 会将当前对话状态传递给下一个 Agent，确保信息不丢失。

### 工具链过长时如何控制成本？

可以为 Agent 设置 `max_turns` 参数限制最大执行步数，同时在关键节点添加校验工具，避免无效的重复调用。选择合适的模型也能有效控制成本。
