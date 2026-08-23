# Claude API Quick Start: Build AI Applications from Scratch

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-api-guide-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-api-guide-en?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 Continue reading: Claude API Quick Start: Build AI Applications from Scratch](https://tools.cooconsbit.com/en/articles/claude-api-guide-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
