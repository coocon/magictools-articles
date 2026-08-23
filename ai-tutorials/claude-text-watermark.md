# Claude 文本水印：藏在随机数里的签名

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-text-watermark?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-text-watermark?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：Claude 文本水印：藏在随机数里的签名](https://tools.cooconsbit.com/zh/articles/claude-text-watermark?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
