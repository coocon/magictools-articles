# 用 MiniMax 生图给公众号做每日封面：一条全自动管线的实战复盘

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/minimax-ai-cover-pipeline-wechat?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/minimax-ai-cover-pipeline-wechat?utm_source=github&utm_medium=referral)**

我们的公众号「码农早餐」每天早上 8 点推送一期技术圈日报。内容早就全自动了，但封面一直是短板：要么运营手动找图，要么用 SVG 模板生成一张信息卡片——工整，但太素，在订阅列表里毫无点击欲。

这周我们把封面也自动化了：**MiniMax 生图出底图，程序合成标题文字，全程零人工**。这篇文章复盘整条管线的设计与踩坑，三版真实产出图都贴在下面——不是概念稿，是同一天、同一条头条新闻跑出来的东西。

## 一、先想清楚：AI 生图的两个死穴

动手前先做减法。「让 AI 直接出一张带标题的封面」这条最短路径，有两个绕不过去的死穴：

**死穴一：AI 画中文必乱码。** 这不是调 prompt 能解决的问题，是当前文生图模型的通病。封面的核心信息恰恰是中文标题——标题糊了，封面就废了。

**死穴二：比例对不上。** 微信公众号头条大图封面要求 **2.35:1**（这是很多人不知道的冷知识，传 16:9 会被裁），而 MiniMax image-01 支持的最宽比例是 21:9，约 2.33:1。

对策也就顺理成章了，一句话概括：**AI 只负责氛围，程序负责信息**。

```
MiniMax 出 21:9 营销风底图（prompt 里明令禁止出现文字）
    ↓
sharp 居中裁切到精确 2.35:1（940×400）
    ↓
SVG 文字层合成：压暗渐变 + 品牌徽章 + 标题大字
    ↓
上传腾讯云 COS → Playwright 自动灌进公众号草稿
```

标题是 SVG 画的，所以永远锐利、永远不会乱码，还能每天自动跟随当日头条变化。

## 二、第一版翻车：一个形容词把主体挤出了画面

第一版底图 prompt 我们写了 `generous negative space`（大量留白）——做过设计的人都懂，留白显高级。结果：

![第一版：主体被挤到一侧，另一侧大片空白](https://cdn.tools.cooconsbit.com/article-images/wechat-ai-cover/v1-offcenter.png)

主体整个被挤到一侧，剩下 60% 的画面是纯色空白。在方图上「留白」是构图，在 21:9 宽幅上「留白」就是灾难——模型会把 negative space 理解成「把东西堆到一边去」。

...

---

**[👉 继续阅读全文：用 MiniMax 生图给公众号做每日封面：一条全自动管线的实战复盘](https://tools.cooconsbit.com/zh/articles/minimax-ai-cover-pipeline-wechat?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
