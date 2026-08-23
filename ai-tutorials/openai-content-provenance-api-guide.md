# OpenAI Content Provenance API 解读：一个接口同查 C2PA 与 SynthID

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/openai-content-provenance-api-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/openai-content-provenance-api-guide?utm_source=github&utm_medium=referral)**

内容溯源这件事，过去一年从标准文档走进了各家产品：Anthropic 给 Claude 的输出加了水印和 C2PA 签名，Google 推 SynthID，行业验证工具逐渐齐活。OpenAI 最近补上了自己的一块拼图——**Content Provenance API**：一个同步接口，上传文件即返回"这份内容带不带 OpenAI 的溯源信号"。

这篇文章把接口细节和字段语义拆开讲清，重点放在官方文档里几处容易误读的地方。

## 接口做什么：双信号并查

向 `POST /v1/content_provenance_checks` 上传一个图片或音频文件，API 同时检查两类信号：

| 信号 | 适用媒体 | 检查什么 | 特性 |
|------|---------|---------|------|
| **C2PA Content Credentials** | 图片 | 文件内嵌的签名元数据（签发者、AI 生成标记） | 信息丰富，但编辑/转发/转存会剥离 |
| **SynthID** | 图片 + 音频 | 直接嵌在像素/波形里的水印 | 信息少，但能扛住部分变换 |

两路信号一脆一韧，互为补充——这正是我们在《[C2PA 内容凭证验证指南](https://tools.cooconsbit.com/articles/c2pa-content-credentials-guide)》里说的"元数据 + 像素水印双轨"思路的产品化。

调用是同步的：响应即结果，不需要建后台任务、轮询或先传 Files API。不想写代码的话，官方也提供了网页版 [openai.com/verify](https://openai.com/verify/)。

```bash
curl https://api.openai.com/v1/content_provenance_checks \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -F "file=@./example.png;type=image/png"
```

Python SDK（2.52.0+）：

```python
from openai import OpenAI

client = OpenAI()

with open("./example.png", "rb") as image:
    result = client.content_provenance_checks.create(
        file=("example.png", image, "image/png"),
    )
```

## 返回结构与字段语义

一张图片的典型返回：

```json
{
  "object": "content_provenance_check",
  "created_at": 1778000000,
  "results": [
    {
      "type": "c2pa",
      "outcome": "detected",
      "validation_state": "trusted",
      "issuer": "OpenAI OpCo, LLC",
      "model": "gpt-image",
      "generated_at": "2026-07-27T18:34:12Z"
    },
    {
      "type": "synthid",
      "outcome": "not_detected",
      "model": null,
      "generated_at": null
    }
  ]
}
```

...

---

**[👉 继续阅读全文：OpenAI Content Provenance API 解读：一个接口同查 C2PA 与 SynthID](https://tools.cooconsbit.com/zh/articles/openai-content-provenance-api-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
