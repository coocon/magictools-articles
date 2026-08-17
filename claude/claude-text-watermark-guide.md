---
title: "Claude 隐形水印是什么？文本水印的原理、检测与局限一文说清"
slug: claude-text-watermark-guide
category: claude
locale: zh
source: authored
tags: [Claude, AI 水印, SynthID, C2PA, EU AI Act, AI 检测, Anthropic]
summary: "2026 年 8 月起，新发布的 Claude 模型会在生成的每段文本里嵌入不可见水印，图片等文件附加 C2PA 签名元数据，全球生效且无法关闭。本文基于 Anthropic 官方文档拆解水印的真实机制（不是零宽字符）、复制粘贴为什么带不掉、目前能否检测和去除，以及官方自己承认的检测边界。"
status: published
---

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

官方说明发布次日，知名博主 John Gruber 在 Daring Fireball 发文[《Anthropic's 'Watermark' Text Adulteration in Claude Is a Perversion of Writing》](https://daringfireball.net/2026/08/anthropics_watermark_text_adulteration_in_claude_is_a_perversion_of_writing)，把争议推到台前。他的核心论点不是隐私，而是**写作质量**：文本水印和图片元数据不同，它改动的就是"选了哪个词"本身——同义词之间没有完全等价的，"leaped at the chance" 和 "jumped at the opportunity" 不是同一句话。一旦知道有绿/红词表在起作用，**每一个用词都变得可疑**：模型选这个词，是因为它最贴切，还是因为它在绿组里？Gruber 还指出官方"imperceptible、不影响质量"的措辞与 SynthID 的 Nature 论文自证存在落差——论文只证明了人类评价差异"统计不显著"，这不等于"不可感知"。

技术社区的另一层批评来自 James Padolsey（他的[交互式图解](https://declaude.org/watermarking/)是理解这类水印最好的入门读物）：这种水印**对老实人最严、对造假者最弱**——普通用户的润色、辅助写作会被打上标记，而有意冒充人类写作的人，用一次不合规模型的改写就能把统计信号洗掉。他自己的 Declaude 就是现成例子：粘贴进去、输出改写后的"素文本"，水印随之消失。这与本文原版的判断一致：它是弱信号，不是判决书。

对读者的实际结论没有变化，反而更清晰了：

1. **别再用"查隐藏字符"的思路防 Claude 水印**——官方已明确没有隐藏字符。想验证一段文本里有没有零宽字符、变体选择符这类**另一种**隐写（其他工具或恶意注入仍在用），可以用我们的 [AI 水印检查器](https://tools.cooconsbit.com/tools/ai-watermark-checker) 扫一遍，还你一个明确结论
2. **文件溯源看 C2PA**：Claude 生成的图片等文件走的是 C2PA 签名元数据，这部分是可以自助验证的——把文件拖进 [C2PA 内容凭证验证器](https://tools.cooconsbit.com/tools/c2pa-verifier)，浏览器内本地验签，不上传文件
3. **文本水印本身，第三方目前无法检测也无法可靠去除**，这一点官方技术说明反而坐实了：没有密钥就没有检测

## 现在能检测吗

截至本文发布，**还没有公开可用的官方检测器**。官方文档的说法是"正在支持用户和第三方检测 Claude 的标记"，检测机制细节将在后续技术文档中公布。

这意味着当前市面上任何声称"能检测 Claude 水印"的工具都值得怀疑：通用的"AI 内容检测器"做的是写作风格分类，与水印检测是两回事——前者对人类作者有众所周知的误伤率，后者需要 Anthropic 尚未公布的密钥或算法细节。

## 官方自己列出的边界

值得肯定的是，Anthropic 在文档里主动写清了检测结论的局限性，这两条对所有想拿"水印检测"当裁决工具的人都很重要：

**检出水印 ≠ Claude 是作者。** 大量用户拿 Claude 校对、翻译、总结、转换文件格式——输出会带上水印，但底层的观点、文字、数据完全来自人类。你用 Claude 润色自己写的段落，产出的就是带标记的"你自己的想法"。

**未检出水印 ≠ 不是 AI 写的。** 官方列举的失效场景包括：

- 由 2026 年 8 月 2 日之前发布的模型生成
- 文本被大量编辑、改写、翻译或混入其他文字
- 段落太短，承载不了可靠的统计信号
- 文件元数据在格式转换、重新保存、截图中被剥离
- 通过暂不支持标记的平台或文件类型产出

换句话说，水印是一个**单向的弱信号**：检出说明"可能经过 Claude 处理"，检不出什么也说明不了。任何把它当成"AI 判定终审"的用法——比如学校或雇主拿检测结果直接下结论——都超出了机制本身的设计能力，这也是这次争议最集中的点。

## 对实际使用的影响

对普通用户和开发者，几件事值得知道：

- **无法关闭**。水印在模型层面注入，不区分产品入口，API 调用同样带标
- **Claude Code 写的代码注释、文档、commit message 同在覆盖范围内**——官方口径是"所有生成文本"。至于代码本身能承载多少统计信号（标识符和语法高度受约束），官方未单独说明
- **质量权衡是真实存在的**。官方称水印不影响输出质量；学术研究则记录过水印强度与文本质量之间的普遍权衡——越抗编辑的水印越容易扰动输出。具体到 Claude 的实现调到哪个点位，外部暂无法验证
- **如果你的产品建立在 Claude API 上**，欧盟《AI 法案》第 50 条对你的产品可能有独立的透明度义务，官方建议自行评估，后续会出配套技术指引

## 常见问题 FAQ

### Claude 的文本水印可以关闭吗？

不能。水印在模型层面全量注入，覆盖所有产品线和云平台入口，官方未提供任何 opt-out 选项。这是 Anthropic 履行欧盟 AI 法案第 50(2) 条透明度承诺的一部分，且全球生效。

### 复制粘贴或者转成其他格式能去掉文本水印吗？

不能。水印藏在词汇选择的统计分布里，不是隐藏字符，复制粘贴、改字体、转格式都带不掉。官方承认重度编辑、改写、翻译**可能**破坏信号，但没有可靠的"去除"手段——而市面上删零宽字符的"去水印工具"针对的是另一种机制，对 Claude 水印无效。

### 现在有工具能检测一段文字是不是 Claude 写的吗？

暂时没有。官方 8 月 16 日的技术说明确认：水印基于密钥分配的绿/红词表概率偏置，**只有 Anthropic 自己能检测**，检测 API 尚未开放。现有的第三方"AI 检测器"做的是写作风格判断，不是水印验证，误判率高，结果不能作为依据。如果你想排查的是文本里有没有零宽字符等隐藏字符（另一类隐写，与 Claude 水印无关），可以用 [AI 水印检查器](https://tools.cooconsbit.com/tools/ai-watermark-checker) 本地扫描。

### 旧版 Claude 模型生成的内容带水印吗？

2026 年 8 月 2 日之前发布的模型暂不带标记。Anthropic 表示正在为存量模型补充标记支持，会更新官方文档。

### 用 Claude 修改我自己写的文章，成品会被标记为 AI 生成吗？

会带上水印——这正是官方明确提示的边界：水印只说明内容"经过 Claude 处理"，不代表 Claude 是作者。如果有机构拿水印检测结果直接判定"这是 AI 写的"，属于对机制的误用。

## 参考链接

- [How Claude marks AI-generated content — Anthropic 官方帮助中心](https://support.claude.com/en/articles/16266773-how-claude-marks-ai-generated-content)
- [How Claude's Text Watermark Works — Anthropic 官方技术说明（2026-08-16）](https://www.anthropic.com/news/claude-text-watermark)
- [Anthropic's 'Watermark' Text Adulteration in Claude Is a Perversion of Writing — Daring Fireball](https://daringfireball.net/2026/08/anthropics_watermark_text_adulteration_in_claude_is_a_perversion_of_writing)
- [How AI Text Watermarking Works — James Padolsey 交互式图解](https://declaude.org/watermarking/)
- [Anthropic's Weak Watermarks Appease a Weak Law — James Padolsey](https://blog.j11y.io/2026-08-12_Anthropics-weak-watermarks-appease-a-weak-law/)
- [Claude Will Put Invisible Watermarks On AI Text And Images — Forbes](https://www.forbes.com/sites/maryroeloffs/2026/08/11/claude-will-put-invisible-watermarks-on-ai-text-and-images-and-the-internet-isnt-happy/)
- [Claude AI Watermark: How Anthropic Marks AI-Generated Text — Business Standard](https://www.business-standard.com/technology/tech-news/claude-invisible-watermark-ai-generated-text-how-it-works-126081100381_1.html)
- [Claude 首推隐形水印 生成文字复制仍留痕 — unwire.hk](https://unwire.hk/2026/08/11/claude-invisible-watermark-ai-generated-text-c2pa/software/)
- [Scalable watermarking for identifying large language model outputs（SynthID-Text）— Nature, 2024](https://www.nature.com/articles/s41586-024-08025-4)
- [C2PA 内容来源与真实性联盟规范](https://c2pa.org/)
