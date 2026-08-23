# Claude API 高级用法：批处理、缓存与成本优化

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-api-advanced?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-api-advanced?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：Claude API 高级用法：批处理、缓存与成本优化](https://tools.cooconsbit.com/zh/articles/claude-api-advanced?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
