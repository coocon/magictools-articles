# Agent Tool Orchestration: Multi-step Reasoning and Task Decomposition

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-agent-tools-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-agent-tools-en?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 Continue reading: Agent Tool Orchestration: Multi-step Reasoning and Task Decomposition](https://tools.cooconsbit.com/en/articles/claude-agent-tools-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
