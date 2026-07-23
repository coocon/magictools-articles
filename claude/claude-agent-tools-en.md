---
title: "Agent Tool Orchestration: Multi-step Reasoning and Task Decomposition"
slug: claude-agent-tools-en
category: claude
tags: [agent-sdk, tool-orchestration, multi-agent, advanced]
summary: "Deep dive into Agent SDK's tool orchestration. Master multi-step reasoning, task decomposition, and multi-agent collaboration patterns for complex task handling."
status: published
---

## Tool Chains: Sequencing Multi-step Operations

In real-world scenarios, Agents often need to chain multiple tools together -- using Tool A's output as Tool B's input. The Agent SDK natively supports this pattern, with the model automatically planning the invocation order.

```python
from claude_agent_sdk import Agent, tool

@tool
def fetch_data(url: str) -> str:
    """Fetch data from a specified URL"""
    return '{"revenue": 5000000, "growth": "15%"}'

@tool
def analyze_data(data: str) -> str:
    """Analyze data and generate a report"""
    return f"Analysis: Data shows steady revenue growth of 15% year-over-year."

@tool
def format_report(analysis: str, format: str) -> str:
    """Format the analysis into a specified output format"""
    return f"# Quarterly Report\n\n{analysis}\n\nFormat: {format}"

agent = Agent(
    name="Data Analyst",
    instructions="You are a data analysis expert. Follow this sequence: fetch data, analyze it, generate a formatted report.",
    tools=[fetch_data, analyze_data, format_report],
)

result = agent.run("Analyze data from https://api.example.com/q4-data and generate a Markdown report")
```

## Multi-Agent Collaboration: The Handoff Pattern

When tasks span multiple domains, use Handoffs to let different Agents handle their specialties:

```python
from claude_agent_sdk import Agent, tool

researcher = Agent(
    name="Researcher",
    instructions="You are responsible for searching and gathering information, then hand off to the editor.",
    tools=[search_web],
)

editor = Agent(
    name="Editor",
    instructions="You organize and polish content into a final article.",
    tools=[format_report],
)

orchestrator = Agent(
    name="Orchestrator",
    instructions="Coordinate the researcher and editor to produce content.",
    handoffs=[researcher, editor],
)

result = orchestrator.run("Write an article about the latest advances in quantum computing")
```

## Task Decomposition Strategies

The key to handling complex tasks is proper decomposition. Common patterns include:

- **Orchestrator pattern**: A primary Agent breaks down the task and delegates to specialized sub-agents
- **Pipeline pattern**: Agents process sequentially in relay, each handling one stage
- **Parallel pattern**: Multiple Agents process independent subtasks simultaneously, then merge results

## Validation and Error Recovery Between Steps

Adding validation logic between steps in a multi-step flow is critical:

```python
@tool
def validate_output(data: str, schema: str) -> str:
    """Validate that the previous step's output matches the expected format"""
    if not data or len(data) < 10:
        return "Validation failed: data is empty or too short. Please retry the previous step."
    return "Validation passed"
```

When a step fails, the Agent can automatically retry or switch to an alternative approach, ensuring overall flow robustness.

## FAQ

### What is the difference between Handoff and calling multiple tools?

A Handoff transfers full control to another Agent that has its own independent instructions and toolset. Multi-tool calling means the same Agent uses different tools within its own context. Handoffs are ideal for scenarios with clear domain specialization.

### How do multiple Agents share context?

Agents share context through message history passed via Handoffs. The orchestrator Agent forwards the current conversation state to the next Agent, ensuring no information is lost.

### How do I control costs when tool chains become long?

You can set the `max_turns` parameter on an Agent to limit the maximum number of execution steps. Adding validation tools at key checkpoints also prevents wasteful repeated calls. Choosing an appropriate model for each step further helps control costs.
