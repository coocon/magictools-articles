---
title: "Claude API Quick Start: Build AI Applications from Scratch"
slug: "claude-api-guide-en"
category: "claude"
tags: [claude-api, tutorial, beginner, sdk]
summary: "Learn Claude API from scratch. Covers API key setup, SDK installation, first API call, conversation management, and common patterns to build your first AI app."
status: "published"
---

## What is Claude API

Claude API is a large language model interface service provided by Anthropic, enabling developers to integrate Claude's conversation, writing, analysis, and coding capabilities into their own applications. With simple HTTP requests or SDK calls, you can build chatbots, content generation tools, code assistants, and various other AI applications.

Unlike using claude.ai directly, the API gives you full control — custom system prompts, conversation context management, model parameter tuning, and seamless AI integration into your products.

## Getting Your API Key

1. Visit [console.anthropic.com](https://console.anthropic.com) and create an account
2. Go to the **API Keys** page and click **Create Key**
3. Copy the generated key and store it securely (the key is shown only once)
4. Set it as an environment variable:

```bash
export ANTHROPIC_API_KEY="sk-ant-api03-xxxxxxxxxxxx"
```

## Installing the SDK

Claude offers official SDKs for Python and TypeScript.

**Python:**

```bash
pip install anthropic
```

**TypeScript / Node.js:**

```bash
npm install @anthropic-ai/sdk
```

## Your First API Call

### Python Example

```python
import anthropic

client = anthropic.Anthropic()

message = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "Hello! Please introduce yourself."}
    ]
)

print(message.content[0].text)
```

### TypeScript Example

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const message = await client.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 1024,
  messages: [
    { role: "user", content: "Hello! Please introduce yourself." }
  ],
});

console.log(message.content[0].text);
```

## Understanding the Response

The returned message object contains these key fields:

- `id`: Unique message identifier
- `content`: Array of content blocks, each with `type` and `text`
- `model`: The model actually used
- `stop_reason`: Why generation stopped (`end_turn` means normal completion)
- `usage`: Token counts (`input_tokens` and `output_tokens`)

## Multi-turn Conversations

Claude API is stateless — you must include the full conversation history with each request:

```python
import anthropic

client = anthropic.Anthropic()

conversation = [
    {"role": "user", "content": "My name is Alex and I'm a frontend developer."},
]

response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=conversation
)

# Add AI response to conversation history
conversation.append({"role": "assistant", "content": response.content[0].text})
conversation.append({"role": "user", "content": "Do you remember my name and profession?"})

response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=conversation
)

print(response.content[0].text)
```

## Model Selection

Anthropic offers multiple Claude models for different use cases:

| Model | Strengths | Best For |
|-------|-----------|----------|
| `claude-opus-4-20250514` | Most capable, complex reasoning | Advanced analysis, research |
| `claude-sonnet-4-20250514` | Balanced performance and speed | General development, content |
| `claude-haiku-4-5-20251001` | Ultra-fast, low cost | Classification, summaries, Q&A |

## Key Parameters

```python
message = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=2048,        # Maximum output tokens
    temperature=0.7,        # Randomness (0 = deterministic, 1 = creative)
    system="You are a professional technical writer.",  # System prompt
    messages=[
        {"role": "user", "content": "Write an API description"}
    ]
)
```

- **max_tokens**: Controls maximum response length, required parameter
- **temperature**: Adjusts output randomness, defaults to 1.0
- **system**: Sets the AI's role and behavioral rules

## Error Handling

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
    print("Too many requests, please retry later")
except anthropic.AuthenticationError:
    print("Invalid API key, please check your configuration")
except anthropic.APIError as e:
    print(f"API error: {e.message}")
```

## Pricing Overview

Claude API charges per token, with prices varying by model. For Claude Sonnet:

- Input: ~$3 per million tokens
- Output: ~$15 per million tokens

Use Haiku during development to minimize costs, and choose the appropriate model for production based on your requirements.

## FAQ

### How is Claude API different from ChatGPT API?

Claude API is developed by Anthropic and uses the Messages API format, supporting a longer context window (200K tokens). While the calling conventions and parameter formats differ, the core concepts are similar. Claude has unique strengths in long document processing, code generation, and instruction following.

### What should I do if my API key is leaked?

Immediately log into console.anthropic.com to revoke the key and create a new one. API keys should be managed through environment variables — never hardcode them in source code or commit them to Git repositories.

### How much free credit do I get?

Anthropic typically provides new accounts with a certain amount of free credit (the exact amount may vary). Once exhausted, you need to add a payment method. Check the Billing page in the Console for real-time credit information.

### How can I control API costs?

You can control costs by: using the Haiku model for simple tasks; setting reasonable max_tokens to avoid overly long outputs; using Prompt Caching for repeated long contexts; and leveraging the Message Batches API for 50% batch processing discounts.
