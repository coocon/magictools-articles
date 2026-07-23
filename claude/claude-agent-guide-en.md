---
title: "Getting Started with Claude Agent SDK: Build Your First Intelligent Agent"
slug: claude-agent-guide-en
category: claude
tags: [agent-sdk, tutorial, beginner, claude-agent]
summary: "Learn Claude Agent SDK from scratch. Understand core agent concepts and build AI agents capable of tool use and multi-step reasoning."
status: published
---

## What Are AI Agents?

Traditional chatbots handle simple request-response interactions: you ask a question, they give an answer. AI Agents are fundamentally different -- they can **plan autonomously, invoke tools, and reason across multiple steps**, acting like a real assistant to accomplish complex tasks.

For example, if you ask a regular chatbot to "check tomorrow's weather in New York and email the report to my colleague," it can only tell you how to do it. An AI Agent, however, can call a weather API to fetch the data, then use an email tool to send the results automatically.

## Why Choose Claude Agent SDK?

Claude Agent SDK is Anthropic's official Python framework designed specifically for building intelligent agents. Key advantages include:

- **Ready out of the box**: Built-in agent loop, tool calling, and multi-agent collaboration
- **Safe and controllable**: Supports guardrails and permission boundaries
- **Flexible and extensible**: Easily add custom tools and support Handoffs between agents
- **Production-ready**: Provides monitoring, logging, and error handling mechanisms

## Installation and Setup

Install the SDK via pip:

```python
pip install claude-agent-sdk
```

Make sure you have your Anthropic API key configured:

```python
export ANTHROPIC_API_KEY="your-api-key-here"
```

## Core Concepts

Before diving into code, understand these three key concepts:

### Agent

An Agent is the core execution unit. It has a model, a set of tools, and system instructions. The Agent autonomously decides when and which tools to call to complete a task.

### Tool

A Tool is a function that an Agent can invoke. You can turn any Python function into a tool that the Agent will automatically call when needed.

### Handoff

A Handoff allows one Agent to delegate a task to another specialized Agent, enabling multi-agent collaboration.

## Creating Your First Agent

Here is the simplest possible Agent example:

```python
from claude_agent_sdk import Agent, tool

# Define a tool
@tool
def get_weather(city: str) -> str:
    """Get weather information for a specified city"""
    # In a real project, this would call a weather API
    return f"Today in {city}: Sunny, 72°F"

# Create an Agent
agent = Agent(
    name="Weather Assistant",
    instructions="You are a weather assistant that helps users check weather information.",
    tools=[get_weather],
)

# Run the Agent
result = agent.run("What's the weather like in New York tomorrow?")
print(result.output)
```

## Adding Multiple Tools

The real power of Agents comes from combining multiple tools -- the Agent autonomously decides the calling order:

```python
from claude_agent_sdk import Agent, tool

@tool
def search_web(query: str) -> str:
    """Search the web for information"""
    return f"Search results: Latest information about '{query}'..."

@tool
def send_email(to: str, subject: str, body: str) -> str:
    """Send an email"""
    return f"Email sent to {to}"

@tool
def summarize_text(text: str) -> str:
    """Summarize text content"""
    return f"Summary: {text[:100]}..."

agent = Agent(
    name="Multi-purpose Assistant",
    instructions="You are a versatile assistant that can search information, send emails, and summarize content.",
    tools=[search_web, send_email, summarize_text],
)

result = agent.run("Search for the latest AI news, summarize it, and email it to team@example.com")
print(result.output)
```

## Agent Execution Flow

When you call `agent.run()`, the SDK internally executes the following loop:

1. Send the user message to the model
2. The model decides whether it needs to call a tool
3. If needed, execute the tool and return the result to the model
4. The model continues reasoning based on tool results or returns the final answer
5. Repeat steps 2-4 until the task is complete

This loop enables Agents to handle complex tasks requiring multi-step reasoning.

## When Should You Use Agents?

Not every scenario requires an Agent. Here is a simple decision guide:

| Scenario | Recommended Approach |
|----------|---------------------|
| Simple Q&A, text generation | Direct API call |
| Requires external tool calls | Use an Agent |
| Multi-step, multi-tool coordination | Use an Agent |
| Requires autonomous planning | Use an Agent |

## FAQ

### What programming languages does Claude Agent SDK support?

Currently, Claude Agent SDK primarily supports Python. Anthropic prioritizes feature completeness and stability for the Python SDK. Support for additional languages may be added in future releases.

### What is the difference between an Agent and a regular API call?

A regular API call follows a single request-response pattern, while an Agent automatically conducts multi-turn interactions: analyzing tasks, selecting tools, executing operations, and verifying results. The Agent maintains an internal execution loop that autonomously completes complex multi-step tasks.

### How much does it cost to use Agent SDK?

Each tool invocation and reasoning step in an Agent consumes tokens. Complex tasks may require multiple steps, so costs will be higher than a single API call. It is recommended to use smaller models for testing during development and choose the appropriate model for production based on your requirements.

### How can I debug the Agent's execution process?

The Agent SDK supports a verbose mode that prints each reasoning step and tool call details. You can also inspect `result.steps` to view the complete execution trace and understand what decisions the Agent made at each step.
