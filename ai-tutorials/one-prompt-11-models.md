# 同一个提示词，11 个模型差出 200 倍

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/one-prompt-11-models?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/one-prompt-11-models?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：同一个提示词，11 个模型差出 200 倍](https://tools.cooconsbit.com/zh/articles/one-prompt-11-models?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
