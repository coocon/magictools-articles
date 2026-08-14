---
title: "同一个提示词，11 个模型差出 200 倍"
slug: one-prompt-11-models
summary: "Netlify 用开源评测框架 AXIS，把同一个建站提示词喂给 11 个主流模型，credit 消耗从 2.4 到 519。这是一份关于模型选型的一手实证，也是一份关于「贵到底值不值」的账单。"
category: ai-tutorials
tags: [模型评测, AI编程, Netlify, OpenRouter, Claude, GPT, Gemini, DeepSeek, Kimi, GLM]
coverImage: ""
status: published
locale: zh
source: authored
translationSlug: one-prompt-11-models-en
---

# 同一个提示词，11 个模型差出 200 倍

> 素材来自 Netlify 官方博客《More models, more choice: Comparing 11 different AI models》（2026-08-13）。所有引语逐字取自原文。

---

模型选型这件事，过去两年基本靠三样东西支撑：榜单、发布会 PPT、和推特上互相转发的「我试了一下，绝了」。

Netlify 刚做了一件更有用的事。他们跟 OpenRouter 合作，把 Kimi K3、GLM 5.2、DeepSeek V4 这批开源模型接进了 Agent Runners，然后用自家已经开源的评测框架 AXIS，拿**同一个建站提示词**，跑了 11 个模型、每个跑 3 次，把每一次的 credit 消耗和生成结果全部摊开给你看。

结果：最贵的平均 519 credits，最便宜的 2.4。**差 200 倍。**

更有意思的不是这个数字，是数字背后那些说不清道不明的东西——为什么便宜的那个反而看着像回事，为什么贵的那个会突然失控烧掉 1055 credits，为什么同代不同代的模型差得像换了行业。

以下是 10 个值得拿走的判断。

---

## 1. 这不是榜单，是一张账单

> These checks are very much focused on _correct functionality of the generated site rather than its design_, e.g.: does it use a database when a user's needs call for it? Does it properly use Netlify Database in that case?

Netlify 内部本来就在用 AXIS 做自动评测，标准很实在：该用数据库的时候用没用、用对没有、不该用数据库的时候有没有过度设计。分数不达标的模型，直接不上 Agent Runners。

但这次他们做的事不一样。作者自己说得很清楚：

> Of course, this is going to be a much more subjective test than our internal test suites, but it's also going to be a very fun one.

也就是说，这篇报告本身是**主观的**——它不告诉你谁分高，它告诉你「花这些钱，你会拿到什么东西」。

**My take:** 榜单回答「谁更强」，账单回答「我该买哪个」。后者才是我每天真正要做的决定。

---

## 2. 提示词的最后一句才是真正的考题

> Build a one-page site for a neighbourhood coffee shop: opening hours, the address, a short menu and a photo. Nothing on it changes unless I edit it myself.

一句话建一个咖啡店单页。前半句是需求，**后半句才是陷阱**：

> The last sentence was added as a hint to the model that no fancy Content Management System is needed.

「除非我自己改，否则页面内容不会变」——这是在暗示模型：不需要 CMS，不需要数据库，一个静态页就够了。听不懂这句暗示的模型，会给你上一整套后台管理。

**My take:** 真正区分模型的从来不是「能不能做」，是「知不知道什么不用做」。过度设计和做不出来一样，都是没听懂人话。

---

## 3. 200 倍价差，这张表本身就是结论

| 模型 | 平均 credits | 三次实跑 |
|---|---|---|
| Claude Opus 5 | 519 | 253 · 249 · 1,055 |
| Claude Sonnet 5 | 143 | 81 · 245 · 103 |
| GPT 5.6 Sol（默认 low effort） | 141 | 173 · 158 · 92 |
| Gemini 3.6 Flash | 103 | 109 · 91 · 111 |
| Kimi K3 | 102 | 125 · 95 · 86 |
| Gemini 3.1 Pro | 53 | 57 · 52 · 49 |
| GPT 5.6 Terra | 39 | 43 · 23 · 49 |
| DeepSeek V4 Pro | 37 | 47 · 30 · 33 |
| GLM 5.2 | 27 | 15 · 42 · 24 |
| Kimi K2.7 Code | 19 | 21 · 18 · 17 |
| DeepSeek V4 Flash (0731) | 2.4 | 3.4 · 1.3 · 2.5 |

> That's a pretty wide distribution, eh? Not only that: the Claude Opus average is heavily slanted upwards because one of its three runs spent a whopping 1,055 credits!

配上 Netlify 的额度：免费版 300 credits，Personal 1,000，Pro 3,000，Pro 加购 $10 换 1,500。

翻译一下：**Opus 5 的那次 1055，一次就烧掉免费版三倍多的额度，Pro 版三分之一。**同样的额度换成 DeepSeek V4 Flash，够你跑 400 多次。

**My take:** 别再问「哪个模型最强」了。先问自己一个月有多少 credits，再问这个任务值不值得用掉其中的 1/3。

---

## 4. Opus 的 1055 credits 不是意外，是脾气

> But in all the tests I've done, Opus does have a tendency to run off with excessive credit usage (compared to its "typical" baseline) more than other models. It does not guarantee a worse or better outcome, though. It's something that just _happens_ pretty frequently.

这段是全文最值钱的一句话。作者的意思不是「Opus 偶尔翻车」，是「Opus **经常**这样，而且烧掉 4 倍的钱不保证给你 4 倍的东西」。

那次 1055 的产出确实好看——带咖啡豆的邮戳元素是纯文本做的、可以做动画，底部还有手写的地图，暗色模式开箱即用。但另外两次分别只花了 253 和 249，结果也不差。

> As to whether the first result is truly "4x better" or not, opinions might vary.

**My take:** 这是成本预测性的问题，不是质量问题。一个方差这么大的模型，你没法给它排预算。真要用 Opus，要么盯着它跑，要么接受抽奖。

---

## 5. 中端塌陷：Sonnet 5 输给了「省着想」的 GPT

同价位（143 vs 141）的对比，作者给了一个不太客气的判断：

> I think OpenAI's top-tier model in low effort mode wins over Anthropic's mid-tier model when it comes to basic design intuition, at least in this scenario.

Sonnet 5 的问题不是难看，是**变薄**：

> There's still some delightful detail in each of these, just _less so_ (and less content in general). The vector graphics is noticeably simpler and not really something you'd consider for a live site.

而 GPT 5.6 Sol 哪怕被强制降到 low effort，内容仍然更丰富，也没有那些奇形怪状的矢量图形（代价是配图偏通用）。

**My take:** 顶级模型「少想一点」，可能比中端模型「使劲想」更值。降 effort 是比降型号更划算的省钱方式——至少在这个场景里是。

---

## 6. Terra 不是低配版 Sol，是另一种审美

从 Sol 降到 Terra（141 → 39 credits，OpenAI 的序列是 Sol→Terra→Luna），并没有出现 Opus→Sonnet 那种断崖：

> here it seems like Terra has a _different visual language_, and not a necessarily worse one.

Terra 的内容确实更简单，也有小毛病——左边那次图片丢了，中间那次图上文字对比度不够。但整体是「另一种风格」，不是「劣化版」。

于是作者给出了他自己的实际用法：

> Up to this point, if I had a very vague idea of what design & language I'd like for a project, my personal inclination would be to run the same prompt with Opus 5 and GPT 5.6 Terra, and get two very different but worthwhile takes.

Opus 5 加 Terra，一次 519 一次 39，加起来不到 560——比 Opus 单独跑两遍还便宜，但你拿到的是**两个视角**而不是两个近似解。

**My take:** 这是全文最可操作的一条。想法模糊的时候，别指望一个模型给你最优解，让两个审美不同的模型各说各话，你从中间挑。

---

## 7. Gemini 的代际差，大到不像同一个厂

3.1 Pro 花了 53 credits，作者连链接都懒得放：

> Here is what Gemini 3.1 Pro generated for 53 credits on average. I'm not even putting the links to the live site here, because there's really nothing to see.

三次跑出来的东西高度雷同，而且——

> Yes, these are wholly separate runs. It did what we asked in the prompt, and really nothing more.

「你要什么它给什么，多一点都没有。」

3.6 Flash 完全是另一回事：花了 103 credits，作者的评价是 `seems like a whole new generation`，在内容上下的功夫多得多。唯一的槽点是重复：

> All models repeat themselves, but it seems like Gemini might repeat itself even more.

**My take:** 「Pro 比 Flash 强」这个直觉已经过期了。代际 > 型号档位。用一年前的 Pro 不如用现在的 Flash，而且后者还更贵——这本身就说明它干了更多活。

---

## 8. 开源模型：位置真实，但别用错题去考它

Kimi K3 花了 102 credits，表现不算突出。作者主动替它说了句公道话：

> To be clear, Kimi K3 is marketed mostly as a frontier model for long-horizon agentic tasks, and various benchmarks and reviews confirm its prowess in that field. It was built to take on Fable 5 more than Opus 5. But in this narrow design-led task, it does not particularly shine among others. **To really do this model justice, we'd need a wholly different set of prompts engineered for a complex web app,**

Kimi K2.7 Code 只要 19 credits，但：

> Despite some hype about Kimi's visual capabilities from around the K2.6 model launch, in terms of design or content there's really not much to see here.

GLM 5.2 是最有趣的一个。27 credits 均价，三次分别 15、42、24，但结果差异大到离谱：

> Surprisingly, these runs are - maple aside - very different, as if coming from a few different models. For the relatively low credit cost of GLM, it's probably worthwhile to run it a few times before settling on what this model can do for you.

（另：GLM 有个偏执的爱好是往页面里塞枫叶。）

还有一条硬性能力边界：

> Note that being a text-only model that does not receive image inputs, GLM in its current 5.2 iteration cannot do something that Kimi models can: get screenshots from the user for inspiration, as in "this is the kind of design I'm looking for".

**My take:** 便宜模型的正确打开方式是**多跑几次挑一个**，反正一次才二十几 credits。但纯文本模型是硬门槛——你没法把截图丢给它说「我要这种感觉」，那这个任务从一开始就不该给它。

---

## 9. 一个 broken image，暴露了闭源和开源之间那道细缝

DeepSeek V4 Pro 均价 37 credits，和 GPT 5.6 Terra 几乎同价，但结果被比下去了。真正扎眼的是这个：

> The middle run also has a broken image: the HTML file points to an image file that does not actually exist in the project, which is a lot less likely to occur nowadays with any of the commercial models from OpenAI, Anthropic, or Google.

HTML 里引用了一个项目里根本不存在的图片文件。这不是审美问题，是**基本的自我校验缺失**——生成完没有回头检查自己引用的资源在不在。

而 V4 Flash 0731 用 2.4 credits 创下全场最低纪录，反倒给了个意外：

> Interestingly, the middle one doesn't just look the most like what a mid-tier closed model might give you, but also feels the same in terms of language, and has actually consumed the least credits among all runs.

那次只花了 1.3 credits——**全场最便宜的一次跑，长得最像中端闭源模型的产出。**

**My take:** 便宜到离谱的模型，风险不在于难看，在于「漏做」。它不会告诉你哪里没做完。所以敢用它的前提是——你有能力自己验收。

---

## 10. 简单网站之外，整套评判标准会换掉

> First, for anything beyond a simple website or the initial ideation phase for a project, the question shifts from how nice the model design & copy is to:
>
> *   Does it know which platform features to use, when and how, to get the functionality you want? Can it store user data, use AI in your web app, and handle authentication and security?
> *   Does it rigorously validate its own work? Can it validate the frontend aspect of your project (that's where image inputs become crucial)? Can it reliably find and fix issues based on feedback from you, and tell you when your own input is misleading or you've overlooked an important concern?

这段是整篇文章的重心。一个静态咖啡店页面能比的只有审美和文案，一旦上升到真实项目，问题立刻变成三件事：**会不会用平台能力、会不会自我验证、能不能可靠地修**。

最后一句尤其重的一条：能不能在你说错的时候告诉你「你这个需求有问题」。

作者对 Opus 的最终定位也在这里：

> Currently, Opus will probably provide the most clever word games and sleekest design, but you don't necessarily need it to. Of course, Opus will also perform relentless self-validation of its own work (it does not bill itself on good looks alone). But remember there's certainly a higher-than-average credit cost attached to that.

那 519 credits 里，有一部分买的不是好看，是那个 relentless self-validation。

而对预算有限的人，作者没有给标准答案：

> Given a limited budget, would you prefer a _turnkey solution_ that attempts to pre-plan and handle everything for you, or should you go with a simpler model and a more iterative approach, where you guide the model with follow-up prompts towards what you want? No option here is necessarily wrong.

**My take:** 一站式贵在「不用管」，迭代式便宜在「你来管」。这道题的答案不取决于模型，取决于你有多少时间、以及你有没有能力判断它做错了。**你验收能力越强，就越买得起便宜模型。**

---

## 写在最后

这份报告的诚实之处，在于它反复提醒你：这只是一次**设计导向的窄任务**，一次静态咖啡店页面，不代表模型的整体能力。Kimi K3 被拿去做它不擅长的事，作者主动说了；这个测试比内部测试套件主观，作者也主动说了。

但它比一百份榜单有用，因为它把两个平时被分开讨论的东西钉在了同一张表上：**你花了多少，和你拿到了什么。**

如果只带走一句：**没有最好的模型，只有配得上你预算和验收能力的模型。** 想法模糊时用两个不同审美的模型对撞，任务窄且明确时用便宜的多跑几次，真到要碰数据库、认证和 AI 网关的时候——再回头考虑那 519 credits 值不值。

---

*原文：More models, more choice: Comparing 11 different AI models（Netlify Blog，2026-08-13）*
*原文链接：https://www.netlify.com/blog/one-prompt-11-models-very-different-results/*
*全部测试结果站点：https://the-coffee-shop-brief.netlify.app/*
