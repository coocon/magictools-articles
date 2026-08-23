# Claude Agent SDK 入门：构建你的第一个智能代理

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-agent-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-agent-guide?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：Claude Agent SDK 入门：构建你的第一个智能代理](https://tools.cooconsbit.com/zh/articles/claude-agent-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
