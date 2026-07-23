---
title: "Claude Agent SDK 入门：构建你的第一个智能代理"
slug: claude-agent-guide
category: claude
tags: [agent-sdk, tutorial, beginner, claude-agent]
summary: "从零开始学习 Claude Agent SDK，了解智能代理的核心概念，学会创建具备工具调用、多步推理能力的 AI Agent。"
status: published
---

## 什么是 AI Agent？

传统的聊天机器人只能进行单轮问答：你问一个问题，它返回一个回答。而 AI Agent（智能代理）则完全不同——它能够**自主规划、调用工具、多步推理**，像一个真正的助手一样完成复杂任务。

举个例子：你让一个普通聊天机器人"帮我查一下明天北京的天气并发邮件给同事"，它只能告诉你怎么做。但 AI Agent 可以自己调用天气 API 获取数据，然后调用邮件工具把结果发出去。

## 为什么选择 Claude Agent SDK？

Claude Agent SDK 是 Anthropic 官方提供的 Python 框架，专为构建智能代理而设计。它的核心优势包括：

- **开箱即用**：内置 Agent 循环、工具调用、多代理协作
- **安全可控**：支持 guardrails（安全护栏）和权限边界
- **灵活扩展**：轻松添加自定义工具，支持 Handoff（代理切换）
- **生产就绪**：提供监控、日志和错误处理机制

## 安装与环境准备

首先通过 pip 安装 SDK：

```python
pip install claude-agent-sdk
```

确保你已经设置了 Anthropic API 密钥：

```python
export ANTHROPIC_API_KEY="your-api-key-here"
```

## 核心概念

在开始编码之前，先了解三个关键概念：

### Agent（代理）

Agent 是核心执行单元，它拥有一个模型、一组工具和一段系统指令。Agent 会自主决定何时调用哪个工具来完成任务。

### Tool（工具）

工具是 Agent 可以调用的函数。你可以把任何 Python 函数变成工具，让 Agent 在需要时自动调用。

### Handoff（切换）

Handoff 允许一个 Agent 将任务交给另一个专门的 Agent 处理，实现多代理协作。

## 创建你的第一个 Agent

下面是一个最简单的 Agent 示例：

```python
from claude_agent_sdk import Agent, tool

# 定义一个工具
@tool
def get_weather(city: str) -> str:
    """获取指定城市的天气信息"""
    # 实际项目中这里会调用天气 API
    return f"{city}今天晴，气温 22°C"

# 创建 Agent
agent = Agent(
    name="天气助手",
    instructions="你是一个天气查询助手，帮助用户查询天气信息。",
    tools=[get_weather],
)

# 运行 Agent
result = agent.run("北京明天天气怎么样？")
print(result.output)
```

## 添加多个工具

Agent 的强大之处在于可以组合多个工具，自主决定调用顺序：

```python
from claude_agent_sdk import Agent, tool

@tool
def search_web(query: str) -> str:
    """搜索网页获取信息"""
    return f"搜索结果：关于 '{query}' 的最新信息..."

@tool
def send_email(to: str, subject: str, body: str) -> str:
    """发送邮件"""
    return f"邮件已发送给 {to}"

@tool
def summarize_text(text: str) -> str:
    """总结文本内容"""
    return f"摘要：{text[:100]}..."

agent = Agent(
    name="全能助手",
    instructions="你是一个多功能助手，可以搜索信息、发送邮件和总结内容。",
    tools=[search_web, send_email, summarize_text],
)

result = agent.run("搜索最新的 AI 新闻，总结后发邮件给 team@example.com")
print(result.output)
```

## Agent 执行流程

当你调用 `agent.run()` 时，SDK 内部执行以下循环：

1. 将用户消息发送给模型
2. 模型决定是否需要调用工具
3. 如果需要，执行工具并将结果返回给模型
4. 模型根据工具结果继续推理或返回最终答案
5. 重复步骤 2-4，直到任务完成

这个循环让 Agent 能够处理需要多步推理的复杂任务。

## 什么时候该用 Agent？

并非所有场景都需要 Agent。以下是简单的判断标准：

| 场景 | 推荐方案 |
|------|---------|
| 简单问答、文本生成 | 直接调用 API |
| 需要调用外部工具 | 使用 Agent |
| 多步骤、多工具协作 | 使用 Agent |
| 需要自主决策和规划 | 使用 Agent |

## 常见问题

### Claude Agent SDK 支持哪些编程语言？

目前 Claude Agent SDK 主要支持 Python。Anthropic 优先保证 Python SDK 的功能完整性和稳定性。其他语言的支持可能会在后续版本中添加。

### Agent 和普通 API 调用有什么区别？

普通 API 调用是单次请求-响应模式，而 Agent 会自动进行多轮交互：分析任务、选择工具、执行操作、验证结果。Agent 内部维护一个执行循环，能自主完成复杂的多步骤任务。

### Agent SDK 的使用成本如何？

Agent 的每次工具调用和推理步骤都会消耗 token。复杂任务可能需要多个步骤，因此成本会高于单次 API 调用。建议在开发阶段使用较小的模型进行测试，生产环境根据需求选择合适的模型。

### 如何调试 Agent 的执行过程？

Agent SDK 支持设置 verbose 模式，可以打印每一步的推理过程和工具调用详情。你也可以通过检查 `result.steps` 来查看完整的执行轨迹，了解 Agent 在每一步做了什么决策。
