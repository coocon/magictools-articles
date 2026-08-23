# Claude API 快速入门：从零开始构建 AI 应用

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-api-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-api-guide?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：Claude API 快速入门：从零开始构建 AI 应用](https://tools.cooconsbit.com/zh/articles/claude-api-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
