---
title: "Claude API 高级用法：批处理、缓存与成本优化"
slug: "claude-api-advanced"
category: "claude"
tags: [claude-api, optimization, caching, batching, advanced]
summary: "探索 Claude API 的高级功能：Message Batches API 批量处理降低成本 50%，Prompt Caching 缓存长上下文节省 90% 输入费用，以及其他成本优化策略。"
status: "published"
---

## Message Batches API

当你有大量不需要即时响应的请求时，Batches API 可以帮你节省 50% 的费用。批量请求会在 24 小时内异步处理完成。

### 创建批处理任务

```python
import anthropic

client = anthropic.Anthropic()

batch = client.messages.batches.create(
    requests=[
        {
            "custom_id": "request-1",
            "params": {
                "model": "claude-sonnet-4-20250514",
                "max_tokens": 1024,
                "messages": [{"role": "user", "content": "总结量子计算的核心概念"}]
            }
        },
        {
            "custom_id": "request-2",
            "params": {
                "model": "claude-sonnet-4-20250514",
                "max_tokens": 1024,
                "messages": [{"role": "user", "content": "解释机器学习与深度学习的区别"}]
            }
        }
    ]
)

print(f"Batch ID: {batch.id}, Status: {batch.processing_status}")
```

### 查询和获取结果

```python
# 查询状态
batch = client.messages.batches.retrieve(batch.id)
print(f"Status: {batch.processing_status}")

# 获取结果（处理完成后）
for result in client.messages.batches.results(batch.id):
    print(f"{result.custom_id}: {result.result.message.content[0].text[:100]}")
```

## Prompt Caching

对于包含大量固定上下文的场景（如长文档分析、系统提示），Prompt Caching 可以缓存输入内容，后续请求复用缓存，节省高达 90% 的输入 Token 费用。

### 使用缓存

```python
import anthropic

client = anthropic.Anthropic()

long_document = "这是一篇非常长的文档..." * 500  # 长上下文

# 在需要缓存的内容块上添加 cache_control
response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    system=[
        {
            "type": "text",
            "text": long_document,
            "cache_control": {"type": "ephemeral"}
        }
    ],
    messages=[{"role": "user", "content": "这篇文档的核心观点是什么？"}]
)

# 后续相同上下文的请求将命中缓存
# usage.cache_creation_input_tokens: 首次缓存写入
# usage.cache_read_input_tokens: 后续缓存命中
print(f"Cache read: {response.usage.cache_read_input_tokens} tokens")
```

### 缓存定价

- 缓存写入：比正常输入价格高 25%
- 缓存读取：比正常输入价格低 90%
- 缓存有效期：5 分钟（每次命中自动续期）

## Extended Thinking

Extended Thinking 让 Claude 在回答前进行深度思考，适合复杂的推理、数学和编程问题。

```python
response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=8000,
    thinking={
        "type": "enabled",
        "budget_tokens": 5000  # 思考过程的 Token 预算
    },
    messages=[{"role": "user", "content": "证明根号 2 是无理数"}]
)

for block in response.content:
    if block.type == "thinking":
        print(f"思考过程: {block.thinking[:200]}...")
    elif block.type == "text":
        print(f"最终回答: {block.text}")
```

## Token 计数 API

在发送请求前预估 Token 用量，帮助你做好成本控制：

```python
count = client.messages.count_tokens(
    model="claude-sonnet-4-20250514",
    messages=[{"role": "user", "content": "Hello, world!"}]
)
print(f"Input tokens: {count.input_tokens}")
```

## 成本优化策略

| 策略 | 节省幅度 | 适用场景 |
|------|---------|---------|
| 使用 Haiku 模型 | 约 90% | 简单分类、摘要任务 |
| Message Batches | 50% | 离线批量处理 |
| Prompt Caching | 最高 90% | 重复长上下文 |
| 精简 Prompt | 可观 | 所有场景 |
| 合理设置 max_tokens | 可观 | 避免浪费 |

## 速率限制与重试

```python
import anthropic

client = anthropic.Anthropic()

# SDK 内置自动重试（默认 2 次）
client = anthropic.Anthropic(max_retries=3)

# 也可手动处理
try:
    response = client.messages.create(...)
except anthropic.RateLimitError:
    # 等待后重试，或切换到更低的请求频率
    pass
```

## 常见问题

### Batches API 的结果可以保存多久？

批处理结果在完成后保留 29 天。请在此期间下载并保存你需要的结果。

### Prompt Caching 对所有模型都有效吗？

Prompt Caching 支持 Claude Sonnet、Opus 和 Haiku 等主要模型。缓存内容至少需要 1024 个 Token 才会被缓存，过短的内容不会触发缓存机制。

### Extended Thinking 的 budget_tokens 设多少合适？

根据问题复杂度调整。简单推理任务设置 2000-5000 即可，复杂数学证明或多步骤推理可以设置 10000 以上。budget_tokens 不会额外收费，只有实际使用的 Token 计费。

### 如何监控 API 使用成本？

在 console.anthropic.com 的 Usage 页面可以查看实时用量和费用。建议设置 Spending Limit 防止意外超支，同时通过 API 响应中的 `usage` 字段跟踪每次请求的 Token 消耗。
