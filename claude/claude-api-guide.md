---
title: "Claude API 快速入门：从零开始构建 AI 应用"
slug: "claude-api-guide"
category: "claude"
tags: [claude-api, tutorial, beginner, sdk]
summary: "从零开始学习 Claude API，涵盖 API Key 申请、SDK 安装、第一次调用、对话管理和常见用法，快速构建你的第一个 AI 应用。"
status: "published"
---

## 什么是 Claude API

Claude API 是 Anthropic 提供的大语言模型接口服务，让开发者可以在自己的应用中集成 Claude 的对话、写作、分析和编程能力。通过简单的 HTTP 请求或 SDK 调用，你就可以构建聊天机器人、内容生成工具、代码助手等各类 AI 应用。

与直接使用 claude.ai 网页不同，API 让你拥有完全的控制权——自定义系统提示、管理对话上下文、调整模型参数，并将 AI 能力无缝嵌入到产品中。

## 获取 API Key

1. 访问 [console.anthropic.com](https://console.anthropic.com) 并注册账号
2. 进入 **API Keys** 页面，点击 **Create Key**
3. 复制生成的密钥，妥善保存（密钥只显示一次）
4. 设置环境变量：

```bash
export ANTHROPIC_API_KEY="sk-ant-api03-xxxxxxxxxxxx"
```

## 安装 SDK

Claude 提供 Python 和 TypeScript 两个官方 SDK。

**Python：**

```bash
pip install anthropic
```

**TypeScript / Node.js：**

```bash
npm install @anthropic-ai/sdk
```

## 第一次 API 调用

### Python 示例

```python
import anthropic

client = anthropic.Anthropic()

message = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "你好，请介绍一下你自己。"}
    ]
)

print(message.content[0].text)
```

### TypeScript 示例

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const message = await client.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 1024,
  messages: [
    { role: "user", content: "你好，请介绍一下你自己。" }
  ],
});

console.log(message.content[0].text);
```

## 理解 API 响应

API 返回的消息对象包含以下关键字段：

- `id`：消息唯一标识
- `content`：响应内容数组，每个元素有 `type` 和 `text`
- `model`：实际使用的模型
- `stop_reason`：停止原因（`end_turn` 表示正常结束）
- `usage`：Token 用量（`input_tokens` 和 `output_tokens`）

## 多轮对话

Claude API 是无状态的，你需要在每次请求中传入完整的对话历史：

```python
import anthropic

client = anthropic.Anthropic()

conversation = [
    {"role": "user", "content": "我叫小明，我是一名前端开发者。"},
]

response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=conversation
)

# 将 AI 回复加入对话历史
conversation.append({"role": "assistant", "content": response.content[0].text})
conversation.append({"role": "user", "content": "你还记得我的名字和职业吗？"})

response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=conversation
)

print(response.content[0].text)
```

## 模型选择

Anthropic 提供多个 Claude 模型，适用于不同场景：

| 模型 | 特点 | 适用场景 |
|------|------|---------|
| `claude-opus-4-20250514` | 最强能力，复杂推理 | 高难度分析、学术研究 |
| `claude-sonnet-4-20250514` | 性能与速度平衡 | 日常开发、内容生成 |
| `claude-haiku-4-5-20251001` | 极速响应，低成本 | 分类、摘要、简单问答 |

## 关键参数

```python
message = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=2048,        # 最大输出 Token 数
    temperature=0.7,        # 随机性（0 = 确定性，1 = 更有创意）
    system="你是一个专业的技术文档写手。",  # 系统提示
    messages=[
        {"role": "user", "content": "写一段 API 说明"}
    ]
)
```

- **max_tokens**：控制响应长度上限，必填参数
- **temperature**：调节输出的随机性，默认 1.0
- **system**：设定 AI 的角色和行为规则

## 错误处理

```python
import anthropic

client = anthropic.Anthropic()

try:
    message = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=1024,
        messages=[{"role": "user", "content": "Hello"}]
    )
except anthropic.RateLimitError:
    print("请求过于频繁，请稍后重试")
except anthropic.AuthenticationError:
    print("API Key 无效，请检查配置")
except anthropic.APIError as e:
    print(f"API 错误: {e.message}")
```

## 定价概览

Claude API 按 Token 计费，不同模型价格不同。以 Claude Sonnet 为例：

- 输入：约 $3 / 百万 Token
- 输出：约 $15 / 百万 Token

建议在开发阶段使用 Haiku 模型降低成本，生产环境根据需求选择合适的模型。

## 常见问题

### Claude API 和 ChatGPT API 有什么区别？

Claude API 由 Anthropic 开发，使用 Messages API 格式，支持更长的上下文窗口（200K tokens）。两者的调用方式和参数格式不同，但核心概念类似。Claude 在长文档处理、代码生成和遵循指令方面有独特优势。

### API Key 泄露了怎么办？

立即登录 console.anthropic.com 撤销该密钥并创建新密钥。API Key 应通过环境变量管理，切勿硬编码在代码中或提交到 Git 仓库。

### 免费额度有多少？

Anthropic 通常为新账户提供一定金额的免费额度（具体金额可能变化），用完后需要绑定支付方式。建议在 Console 的 Billing 页面查看实时额度信息。

### 如何控制 API 调用成本？

可以通过以下方式控制成本：使用 Haiku 模型处理简单任务；合理设置 max_tokens 避免过长输出；使用 Prompt Caching 缓存重复的长上下文；利用 Message Batches API 批量处理获得 50% 折扣。
