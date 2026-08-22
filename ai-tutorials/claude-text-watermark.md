---
slug: "claude-text-watermark"
title: "Claude 文本水印：藏在随机数里的签名"
summary: "Anthropic 公开了 Claude 文本水印的完整技术细节：不加字、不加隐藏字符、不涨价，只是把选词时的随机数来源换成了「密钥 + 前文」。但它能证明的东西比大多数人以为的少得多——短文本查不出、代码几乎没有、帮你润色的那段基本查不到。"
category: ai-tutorials
tags: [Claude, Anthropic, AI水印, SynthID, 欧盟AI法案, AI检测, LLM]
coverImage: ""
status: published
locale: zh
source: authored
translationSlug: claude-text-watermark-en
---

# Claude 文本水印：藏在随机数里的签名

> "Future Claude models will generate text that contains a watermark."

---

2026 年 8 月 2 日起，欧盟要求向其市场提供服务的 AI 供应商标记 AI 生成内容。Anthropic 和另外约 190 家签署方在 2026 年 7 月签了 EU Code of Practice on Transparency of AI-Generated Content。所以这不是一家公司的产品决策——主流厂商都得上，只是各家的水印不一样。

Anthropic 这次把技术细节写得相当直白，包括那些对它自己不太有利的部分。以下 10 条，是我从原文里挑出来、认为值得单独拎出来讲的。

---

## 1. 水印没有往文本里加任何东西

> "When watermarking is used, choices are still made at random, but the _source_ of the randomness is different."

大部分人一听「水印」，脑子里是钞票上的暗纹、图片右下角的 logo、或者藏在 Unicode 里的零宽字符。**这三个都不是。**

原理是这样：LLM 一个词一个词地生成，每一步在候选词里挑。「今天天气又冷又…」，下一个词几乎不可能是「甜」，但「阴」和「灰」都行——读者根本不在乎选了哪个。这种「选哪个都一样」的岔路口，原本由一个随机数决定。

水印做的事，是换掉这个随机数的来源：

> "Instead of using an arbitrary random number generator to pick the next word, watermarking uses the key and a few words that come before to settle what word the model should pick."

密钥 + 前面几个词 → 决定这一步选什么。词还是那些词，句子还是那个句子，但整串选择连起来会形成一个持钥者能验、读者看不见的模式。

**我的看法：** 这个设计的聪明之处在于它是「减法」——不往数据里塞信息，只是把本来就存在的随机性换了个来源。所以它天然免疫「删掉隐藏字符」这类破解，因为压根没有隐藏字符可删。

---

## 2. 大富翁与圆周率：随机性可以被记账

原文里这个比喻，我认为是整篇最好的一段：

> "For all intents and purposes, the moves are still random: it makes no difference to the players—or to the outcome of the game—whether the randomness comes from pi or from dice rolls each time."

玩大富翁，每回合掷骰子决定走几步。现在换个玩法：不掷骰子，改用一本圆周率数字表，从随机某一位开始，往后依次取数当点数。

对局中的所有人来说，没有任何区别。游戏体验一样，胜负一样，随机性一样。但事后有人拿到完整的走位记录，又知道圆周率长什么样，他就能判断出：这局大概率是用圆周率跑的。

**我的看法：** 这个比喻的价值在于它把「随机」和「不可追溯」这两件事拆开了。我们习惯把二者划等号，但随机性完全可以是**可复现的随机**。密码学里这叫伪随机数生成器，只不过这次种子握在 Anthropic 手里，而不是 `time()`。

---

## 3. 「不影响质量」不是自说自话

> "In internal testing, we've seen no impact of watermarking on the content, level of creativity, or readability of Claude's text."

这句如果只有内测背书，我会打个问号。但原文接着给了 Google DeepMind 在 SynthID-Text 论文里做的事——**他们把带水印的模型直接上线，服务了一部分 Gemini 真实流量**，然后对比点赞和点踩：

> "They found no statistically significant differences from the unwatermarked model."

外加一组对照实验：

> "And in a controlled study, human raters comparing watermarked and unwatermarked answers side-by-side saw no difference in quality."

原文还澄清了一个容易误解的点：水印不会让模型永远偏爱「阴」或者「灰」。

> "Importantly, it isn't that the model will now always be biased toward overcast or grey."

也不会逼 Claude 去挑一个它本来根本不会用的生僻词。它只在「本来就无所谓」的候选里做选择。

**我的看法：** 线上 A/B 比任何 benchmark 都有说服力，因为它测的是真实用户的真实反应，而不是评测集。至于成本，原文写得很干脆：不产生额外 token，价格不变，速度影响可以忽略。这一条对开发者是实打实的好消息——很多人第一反应是「是不是要多收钱」。

---

## 4. 这是合规作业，不是产品创新

> "We're implementing watermarking to comply with the EU AI Act."

Anthropic 没有把这事包装成「我们主动为 AI 安全做贡献」，开门见山就是合规。签署方约 190 家，2026 年 7 月签的。

更值得注意的是这句：

> "We're applying watermarking globally at launch because we don't yet have a durable way to scope it by region."

**欧盟的法律，全球范围生效。** 理由是他们暂时没有可靠的办法按地区区分。原文说会继续评估其他方案。

**我的看法：** 这是布鲁塞尔效应的又一个标准样本——欧盟立一条法，全世界用户跟着改。技术上「按地区 scope」当然可以做（IP、账号归属地、API region），说「没有 durable way」，我理解成他们判断这种切分很容易被绕过，与其做个假的不如全量上。诚实，但也确实省事。

---

## 5. 技术不是新的，学术血统很清楚

> "Claude's text watermark is a version of the SynthID-Text approach published by Google DeepMind in a *Nature* paper in 2024."

不是自研，是 Google DeepMind 那套 SynthID-Text 的一个版本，2024 年发在 *Nature*。再往前追：

> "It belongs to a family of approaches that go back to a proposal by Scott Aaronson in 2022, all of which share the same design principle that we described above—the watermark only changes the source of the randomness used to pick among words."

Scott Aaronson 2022 年在 OpenAI 做客座研究时提的思路，四年后成了行业标准动作。

**我的看法：** 这一条我更愿意读成一个行业信号：**在合规这件事上，厂商之间没有必要各造各的轮子。** 大家用同一个技术家族、各持各的密钥，反而是最健康的格局——检测生态可以复用，而 A 家的密钥验不出 B 家的文本，隐私边界也清楚。

---

## 6. 水印只能回答一个问题

> "Using our key, one can only answer the question "What is the likelihood this was partly written by Claude?""

注意每个词：**likelihood**（概率，不是判定）、**partly**（部分参与）、**by Claude**（只认自家）。

它不能证明文本是人写的。也不能告诉你是不是别家 AI 写的——别家有别家的密钥，甚至可能用完全不同的水印方法。

还有一个硬限制：

> "Detecting a watermark also doesn't work well on small samples, where there are fewer word choices and thus less information to go on."

文本越长，可用的「岔路口」越多，置信度越高。反过来，一段话、一条推特，基本无解。

原文在「What does a watermark actually prove?」一节里说得更狠：

> "A watermark can only determine that Claude was likely involved with the content at some point. It cannot distinguish "Claude wrote this" from "Claude heavily edited this.""

**我的看法：** 这是全文最该被划重点的一段，因为它直接决定了水印**不能**用在哪。学校拿它判学生作弊？平台拿它自动封号？招聘方拿它筛简历？——一个只能输出「可能参与过」且区分不了「写」和「改」的信号，做不了任何单点裁决的依据。原文另有一句划清界限：水印不改变输出的归属权和法律责任。

---

## 7. 只有唯一正确答案的地方，没有水印

> "Watermarking is sparser on factual passages where there are fewer choices that can be made without decreasing the accuracy of the text."

原文举的例子非常好：「Isaac Newton 最著名的著作叫 *Principia*…」，下一个词是不是 *Mathematica* 关乎对错，**水印无处落脚**。

同理，「2 + 2 =」后面接什么，没有第二个同样好的选项（原文顺手玩了个梗：如果在聊乔治·奥威尔的《1984》，那没有比「5」更好的答案）。这种地方水印的「nudge」不会施加。

推到代码上，结论就很直接了：

> "For the same reason, code—which in very many cases has to be exact—has generally less watermarking than some other forms of text."

只有注释这类允许自由发挥的位置还能挂上水印：

> "Having said that, in areas where there _is_ an arbitrary choice between particular words or terms within the code, the watermark can be used, such as comments within code."

而且原文自己承认，这对实际产出的代码影响可以忽略不计。

**我的看法：** 这条对开发者是双向的好消息和坏消息。好消息：水印不会碰你的代码逻辑，一个变量名都不会为了埋信息而改。坏消息：如果你指望靠水印识别「这段代码是不是 AI 写的」，基本别想了——删掉注释就更没了。**信息密度越高的文本，水印越稀薄**，这是这个方案的结构性代价，不是工程 bug。

---

## 8. 让 Claude 帮你润色的那段，多半查不出来

> "The watermark only applies to words Claude chooses."

这句是整套机制的边界条件。你写完一篇稿，丢给 Claude 改语法和标点，回来的东西绝大部分词还是你的，水印能附着的位置只有那几处修改——很可能少到根本测不出来。

> "The more Claude writes, the more decisions it has to make, and the more space there is for a watermark."

至于主动抹除：

> "Light editing probably won't remove the watermark completely; a complete rewrite where every word is replaced will."

Anthropic 补了一句我很喜欢的话：

> "In the latter case, of course, it's arguable whether the text can any longer be described as AI-generated."

**我的看法：** 承认「能被绕过」是这篇文档最加分的地方。而且那句反问站得住脚——如果你逐词重写了整篇，那这文本还算不算 AI 生成的，本身就值得吵一架。但也别读成「水印形同虚设」：它防的从来不是刻意对抗者，而是给大规模、无标注的 AI 内容流通提供一个可查的默认标记。对抗性场景下它会输，这一点原文没藏。

---

## 9. 和 Pangram 那类 AI 检测器不是一回事

> "AI detection software uses a different method, because the companies that provide it don't have our key."

这是本质区别，值得所有用过「AI 率检测」的人看一眼。第三方检测器没有密钥，只能从文风上猜——原文点名了两个特征：

> "For example, AI models appear to be fond of the construction "this isn't [X], it's [Y]", and use the word "quietly" a lot more than you might expect."

「这不是 X，而是 Y」的句式，和高频出现的 "quietly"。

**我的看法：** 一个查密码学签名，一个查写作习惯，可靠性差着数量级。文风检测的假阳性代价已经有人付过了——写作风格偏正式的非母语者被误判成 AI，是这两年反复出现的事故。水印至少在**原理上**不会因为「你写得像 AI」就判你有罪，它只认自家模型留下的密钥痕迹。当然代价是覆盖面窄得多：只认 Claude，只认足够长的文本。

顺带一提，被官方点名的这两个「AI 味」特征，值得每个用 AI 写东西的人拿去自查一遍。

---

## 10. 图片不用水印，用 C2PA；以及落地节奏

文件走的是完全不同的一条路。Claude 产出 .png / .jpg / .svg 这类文件时，会在元数据里附一个加密签名的说明，走 C2PA 这个开放行业标准——相机厂商和修图软件也用它记录图片来源。

> "This metadata label is very different from a watermark. Nothing in the file changes—it is not embedded or hidden."

**元数据不是水印。** 它写在文件属性里，不改动文件内容，也因此——这是我的推论——重新导出一次基本就没了。原文说任何 C2PA-aware 的工具都能读，Anthropic 也会提供自己的查询工具。

剩下三件事的时间表：

- 检测 API：`"We will soon be offering a watermark detection API."` 细节还在定。
- 旧模型：2026 年 8 月 2 日之前发布的 Anthropic 模型有过渡期，未来几个月陆续补上。
- 翻译：`"Yes. A translation produced by Claude carries a watermark, because in this case every word is chosen by Claude."`

**我的看法：** 翻译这条最容易被忽略，但影响面可能最大——很多人不觉得「让 AI 翻一下」算 AI 生成内容，可从水印的定义看，每个词都是 Claude 选的，水印密度反而是最高的那一档。你把自己写的中文丢给 Claude 翻成英文，输出的英文版是**满水印**的。这个直觉落差，我猜会是第一批真实纠纷的来源。

---

## 写在最后

这篇官方文档最值得称道的地方，不是它解释了水印怎么工作，而是它花了差不多一半篇幅解释**水印什么时候不工作**：短文本、事实性内容、代码、润色、重写。

一个能被绕过、只能给概率、区分不了「写」和「改」的信号，仍然是有价值的——前提是所有人都清楚它的边界。**麻烦从来不来自技术本身，而来自把「likely involved」当成「铁证」的那些人。**

原文：[How Claude's text watermarking works](https://www.anthropic.com/news/claude-text-watermark)
