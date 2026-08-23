# Claude API Advanced: Batching, Caching, and Cost Optimization

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-api-advanced-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-api-advanced-en?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 Continue reading: Claude API Advanced: Batching, Caching, and Cost Optimization](https://tools.cooconsbit.com/en/articles/claude-api-advanced-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
