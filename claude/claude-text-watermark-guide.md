# Claude 隐形水印是什么？文本水印的原理、检测与局限一文说清

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-text-watermark-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-text-watermark-guide?utm_source=github&utm_medium=referral)**

2026 年 8 月，Anthropic 在官方帮助中心发布了《How Claude marks AI-generated content》，确认了一件此前只在传闻中的事：**自 2026 年 8 月 2 日起发布的 Claude 模型，生成的文本会在模型层面嵌入不可见水印（invisible watermark），生成的图片等文件会附加数字签名的溯源元数据**。政策全球生效，覆盖所有 Claude 产品线，用户没有关闭开关。

消息公布后争议很大，但网上大量讨论建立在一个错误假设上——以为水印是往文本里塞隐藏字符。这篇文章基于官方文档和公开技术分析，把机制、边界和常见误解逐条说清。

## 官方确认了什么

Anthropic 签署了欧盟《人工智能法案》第 50(2) 条《AI 生成内容透明度行为准则》，这是水印政策的直接法律背景。官方文档给出的承诺范围：

- **模型**：2026 年 8 月 2 日及之后发布的 Claude 模型，上线即支持标记；更早的存量模型会在后续逐步补上支持
- **产品**：覆盖 Claude Platform（API）、Claude 应用、Claude Code、Claude Cowork、Claude Tag——**所有生成文本都会带嵌入式水印**
- **云平台**：通过 AWS、Google Cloud、Microsoft Foundry 调用受支持的 Claude 模型，文本水印同样生效
- **地区**：虽然动因是欧盟法规，但标记**在全球范围内适用**，不区分地区

两套机制互补：

1. **文本嵌入水印**：生成文本时直接"织入"文字本身，肉眼不可见，官方称不改变语义、质量和可读性。因为水印是文本的一部分，**复制粘贴会跟着走**，部分编辑后仍可能残留
2. **文件签名元数据**：生成 .svg、.png、.jpg 等受支持的文件类型时，附加遵循 C2PA（内容来源与真实性联盟）开放标准的签名元数据，与 Google、Adobe 使用的是同一套行业标准

## 原理：不是零宽字符，是概率分布里的指纹

很多人听到"隐形文本水印"，第一反应是零宽空格（U+200B）、变体选择符这类隐藏 Unicode 字符。**这不是 Claude 用的方案**——隐藏字符在十六进制编辑器里一览无余，一次查找替换就能清空，作为溯源机制毫无价值。

最初的帮助中心文档没有披露具体算法，但多家媒体和技术分析指向同一类技术：**采样水印（token sampling watermark）**，与 Google DeepMind 2024 年发表在 Nature 上的 SynthID-Text 同源，思路可追溯到 Scott Aaronson 2022 年的提案。**2026 年 8 月 16 日，Anthropic 在官网发布《How Claude's Text Watermark Works》，正式确认了这一机制**（见下方更新章节）。工作方式大致是：

- 模型生成每个词时，本来就在一堆候选词里按概率采样
- 水印算法按某种隐藏规则，对部分候选词施加**轻微的概率偏置**
- 单看一句话与正常输出没有区别；但文本足够长时，这种统计偏好会形成可被专门检测器识别的模式

可以理解为**藏在文本概率分布里的指纹**，而不是藏在字符里的暗号。这解释了两件事：为什么复制粘贴带不掉（指纹就是文字选择本身），以及为什么网上流传的"AI 水印清除工具"（删零宽字符、替换弯引号那套）对它完全无效——它们针对的是另一种根本不存在于 Claude 输出里的机制。

据 Forbes 报道，有 Anthropic 工程师在社交媒体上补充了三个细节：检测 API 后续会开放给用户自行调用；模型本身并不知道自己被加了水印；以及一句坦率的评价——"它不完美，你可以通过编辑去掉它，但这是第一步"。

## 2026-08-16 更新：官方公开技术细节，争议随之升级

本文首发时上面这段还是"合理推测"，现在有了官方答案。8 月 16 日 Anthropic 发布技术说明[《How Claude's Text Watermark Works》](https://www.anthropic.com/news/claude-text-watermark)，要点：

- **确认是统计水印**：生成每个 token 时，按密钥把候选词动态分成"绿/红"两组，对绿组施加轻微概率偏置。文本里**没有添加任何字符，也没有隐藏字符**——同一个词这次在绿组、下次可能在红组，因此不存在一份"Claude 偏爱词表"
- **只有持有密钥的 Anthropic 能检测**，且各家水印互不相通：Claude 检测不了 Gemini 的 SynthID 标记，反之亦然
- **生效阈值约 200 token（约 150 词）以上**——这也是欧盟《AI 生成内容透明度行为准则》的适用下限；准则同时要求提供商在服务条款里**禁止用户移除水印**，并要求水印对复制粘贴、截图、OCR、翻译等"常规处理"保持稳健

...

---

**[👉 继续阅读全文：Claude 隐形水印是什么？文本水印的原理、检测与局限一文说清](https://tools.cooconsbit.com/zh/articles/claude-text-watermark-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
