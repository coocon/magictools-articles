---
title: "OpenAI Content Provenance API 解读：一个接口同查 C2PA 与 SynthID"
slug: "openai-content-provenance-api-guide"
category: ai-tutorials
locale: zh
source: authored
tags:
  - OpenAI
  - Content Provenance
  - C2PA
  - SynthID
  - AI 检测
  - API
summary: "OpenAI 上线了 Content Provenance API：向 /v1/content_provenance_checks 上传图片或音频，同步返回 C2PA 内容凭证与 SynthID 水印两路检测结果。本文拆解请求方式、返回字段的精确语义（detected 的判定条件比想象中严格）、支持格式与限制、以及官方自己写明的使用边界——它不是通用 AI 检测器，只认 OpenAI 自家信号。"
coverImage: ""
status: published
scheduledAt: ""
---

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

几个设计细节值得注意：

- **没有顶层 outcome**。图片返回 C2PA + SynthID 两条结果，音频只返回 SynthID 一条，每条要独立解读，API 不替你汇总"是/不是 AI 生成"
- **不适用的检查直接省略**，而不是返回 `not_detected`——解析时按 `type` 找，别按下标取

### C2PA 结果：detected 的判定比想象中严格

`outcome` 为 `detected` 需要**同时满足三个条件**：manifest 的 `validation_state` 是 `trusted` 或 `valid`、签发者是 OpenAI、且清单里含 AI 生成动作。任何一条不满足——第三方签发的清单、没有 AI 生成动作的清单、`invalid`（签名校验失败）或 `not_present`（没有清单）——都是 `not_detected`。

这意味着一个重要的解读技巧：**`outcome` 是 `not_detected` 时，`issuer` 和 `validation_state` 字段仍然可能有内容**。比如一张 Adobe Firefly 签名的图，outcome 是 `not_detected`（不是 OpenAI 签的），但 `issuer` 会告诉你它是谁签的。只看 outcome 会丢掉这部分信息。

`validation_state` 的四档语义与 c2pa-rs 一致：`trusted`（签名有效且证书受信）、`valid`（签名有效但证书不在信任列表）、`invalid`（校验失败，不可作为出处证据）、`not_present`（无清单）。`valid` 和 `invalid` 的区别我们在 C2PA 指南里用一个真实踩坑案例展开过——签名有效性和签发者受信是两个独立维度，别混。

### SynthID 结果

语义简单得多：`detected` 表示识别出了受支持的水印；`not_detected` **只表示没检出**，不排除内容是 AI 生成或 AI 修改的。`model` 和 `generated_at` 有则给、无则 null。

## 支持格式与限制

- **图片**：PNG、JPEG、WebP
- **音频**：MP3、Opus、AAC、FLAC、WAV、PCM；解码后时长 ≤ 60 秒。Opus 要把 media type 设成 `audio/ogg`
- **单文件 ≤ 50 MiB**，一次一个文件
- **错误码**：文件格式不支持/损坏返回 400；组织没有访问权限返回 404；超速率限制返回 429（带 `Retry-After`）
- **速率限制很严**，官方明说是防滥用；需要更高配额要单独申请，逐案审批
- **不适用 Zero Data Retention**——对数据驻留有硬要求的场景要注意这条

## 官方划出的边界（比接口本身更重要）

文档里有一节使用指引，态度相当克制，值得原样转述：

1. **它不是通用 AI 检测器**。只认 OpenAI 自家信号，检测不了其他公司模型生成的内容——`not_detected` 既不能证明"内容是人写的"，也不能证明"不是 AI 生成的"
2. **`not_detected` 的原因清单很长**：元数据被剥离、水印随压缩/裁剪/截图/转格式退化、内容出自加信号之前的旧模型、或者格式本身不支持。拿到原始文件再验，结论才有分量
3. **归因前先看 issuer**。检出 C2PA 清单不等于 OpenAI 生成，第三方签发的清单要按签发者归因
4. **高风险场景要配人工复核**，别把自动结果当终审
5. 明令禁止用重复查询来逆向、去除或规避水印

这个坦诚劲儿与 Anthropic 在 Claude 水印文档里的口径一致（见《[Claude 隐形水印是什么](https://tools.cooconsbit.com/articles/claude-text-watermark-guide)》）：溯源信号是**单向弱证据**——检出说明一件确定的事，检不出什么都说明不了。整个行业在这一点上罕见地没有夸大宣传。

## 什么场景用它，什么场景不用

**适合**：内容审核管线里给"疑似 AI 图/音频"加一道确定性证据、新闻编辑室核图、平台给 AI 生成内容打标、信任与安全工作流。

**不适合/不需要**：

- **判断任意内容是否 AI 生成**——它只覆盖 OpenAI 信号，这个问题目前没有可靠的通用答案
- **只想看一张图的 C2PA 凭证**——不需要 API key 和速率配额，用浏览器本地验证工具十秒搞定：[MagicTools C2PA 验证器](https://tools.cooconsbit.com/tools/c2pa-verifier) 在本地跑 WASM 验签、文件不上传，且**不限签发者**，OpenAI、Claude、Adobe、相机签的都能看
- **检查文本水印**——本 API 只收图片和音频。文本那条线的现状（字符层隐写可查、统计水印待官方检测器）见《[AI 文本隐形水印检测指南](https://tools.cooconsbit.com/articles/ai-text-watermark-detection-guide)》

## 常见问题 FAQ

### Content Provenance API 能检测 Midjourney、Claude 或 Stable Diffusion 生成的图吗？

不能给出 `detected`。API 只认 OpenAI 自家的 C2PA 签名和水印信号。不过对带第三方 C2PA 清单的图片，返回里的 `issuer` 和 `validation_state` 仍会描述该清单——你能知道"这是 Adobe 签的"，只是 outcome 不会是 detected。

### 这个 API 收费吗？怎么开通？

接口对有权限的组织开放，速率限制严格，更高配额需通过官方表单申请、逐案审批。具体计费以 OpenAI 官方定价页为准。注意它不适用 Zero Data Retention 政策。

### `not_detected` 能证明图片不是 AI 生成的吗？

不能。元数据被剥离、水印退化、旧模型产物、其他厂商模型生成，都会得到 `not_detected`。官方明确说它是"未检出证据"，不是"人类创作证明"。

### 图片和音频的返回有什么区别？

图片返回 C2PA 和 SynthID 两条结果；音频只返回 SynthID 一条。API 会省略不适用的检查，解析时按 `type` 字段匹配，不要假设数组长度。

### 有没有不用写代码的验证方式？

有两个：OpenAI 官方网页版 [openai.com/verify](https://openai.com/verify/)（查 OpenAI 双信号），以及 [MagicTools C2PA 验证器](https://tools.cooconsbit.com/tools/c2pa-verifier)（浏览器本地验签、不限签发者、文件不上传）。日常核图用后者更快，需要 SynthID 判定时用前者。

## 参考链接

- [Content provenance — OpenAI API 官方文档](https://developers.openai.com/api/docs/guides/content-provenance)
- [Content provenance API reference](https://developers.openai.com/api/reference/resources/content_provenance_checks/methods/create)
- [openai.com/verify 网页版验证工具](https://openai.com/verify/)
- [C2PA 技术规范](https://c2pa.org/specifications/specifications/2.1/index.html)
- [SynthID — Google DeepMind](https://deepmind.google/technologies/synthid/)
