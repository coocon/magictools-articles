---
title: "Claude API Advanced: Batching, Caching, and Cost Optimization"
slug: "claude-api-advanced-en"
category: "claude"
tags: [claude-api, optimization, caching, batching, advanced]
summary: "Explore advanced Claude API features: Message Batches API for 50% cost reduction, Prompt Caching to save 90% on input costs for long contexts, and other optimization strategies."
status: "published"
---

## Message Batches API

When you have large volumes of requests that don't need immediate responses, the Batches API saves you 50% on costs. Batch requests are processed asynchronously within 24 hours.

### Creating a Batch

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
                "messages": [{"role": "user", "content": "Summarize the core concepts of quantum computing"}]
            }
        },
        {
            "custom_id": "request-2",
            "params": {
                "model": "claude-sonnet-4-20250514",
                "max_tokens": 1024,
                "messages": [{"role": "user", "content": "Explain the difference between ML and deep learning"}]
            }
        }
    ]
)

print(f"Batch ID: {batch.id}, Status: {batch.processing_status}")
```

### Checking Status and Retrieving Results

```python
# Check status
batch = client.messages.batches.retrieve(batch.id)
print(f"Status: {batch.processing_status}")

# Get results (after processing completes)
for result in client.messages.batches.results(batch.id):
    print(f"{result.custom_id}: {result.result.message.content[0].text[:100]}")
```

## Prompt Caching

For scenarios with large fixed contexts (long document analysis, system prompts), Prompt Caching stores input content so subsequent requests reuse the cache, saving up to 90% on input token costs.

### Using Cache

```python
import anthropic

client = anthropic.Anthropic()

long_document = "This is a very long document..." * 500  # Long context

# Add cache_control to content blocks you want cached
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
    messages=[{"role": "user", "content": "What are the key points of this document?"}]
)

# Subsequent requests with the same context will hit cache
# usage.cache_creation_input_tokens: first-time cache write
# usage.cache_read_input_tokens: subsequent cache hits
print(f"Cache read: {response.usage.cache_read_input_tokens} tokens")
```

### Cache Pricing

- Cache write: 25% more than standard input pricing
- Cache read: 90% less than standard input pricing
- Cache TTL: 5 minutes (auto-renewed on each hit)

## Extended Thinking

Extended Thinking lets Claude reason deeply before answering, ideal for complex reasoning, math, and coding problems.

```python
response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=8000,
    thinking={
        "type": "enabled",
        "budget_tokens": 5000  # Token budget for thinking
    },
    messages=[{"role": "user", "content": "Prove that the square root of 2 is irrational"}]
)

for block in response.content:
    if block.type == "thinking":
        print(f"Thinking: {block.thinking[:200]}...")
    elif block.type == "text":
        print(f"Answer: {block.text}")
```

## Token Counting API

Estimate token usage before sending requests to manage costs:

```python
count = client.messages.count_tokens(
    model="claude-sonnet-4-20250514",
    messages=[{"role": "user", "content": "Hello, world!"}]
)
print(f"Input tokens: {count.input_tokens}")
```

## Cost Optimization Strategies

| Strategy | Savings | Use Case |
|----------|---------|----------|
| Use Haiku model | ~90% | Simple classification, summaries |
| Message Batches | 50% | Offline bulk processing |
| Prompt Caching | Up to 90% | Repeated long contexts |
| Optimize prompts | Significant | All scenarios |
| Set proper max_tokens | Significant | Avoid waste |

## Rate Limits and Retries

```python
import anthropic

# SDK has built-in auto-retry (default 2 retries)
client = anthropic.Anthropic(max_retries=3)

# Manual handling
try:
    response = client.messages.create(...)
except anthropic.RateLimitError:
    # Wait and retry, or reduce request frequency
    pass
```

## FAQ

### How long are Batches API results retained?

Batch results are retained for 29 days after completion. Download and save any results you need within this period.

### Does Prompt Caching work with all models?

Prompt Caching is supported on major models including Claude Sonnet, Opus, and Haiku. Cached content must be at least 1024 tokens — shorter content will not trigger the caching mechanism.

### What should I set budget_tokens to for Extended Thinking?

Adjust based on problem complexity. For simple reasoning, 2000-5000 is sufficient. For complex mathematical proofs or multi-step reasoning, set 10000 or higher. You are only charged for tokens actually used, not the full budget.

### How can I monitor API usage costs?

Check the Usage page at console.anthropic.com for real-time usage and costs. Set a Spending Limit to prevent unexpected overcharges, and track per-request token consumption via the `usage` field in API responses.
