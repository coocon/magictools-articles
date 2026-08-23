# Auto-Generating Daily WeChat Covers with MiniMax: A Production Pipeline Postmortem

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/minimax-ai-cover-pipeline-wechat-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/minimax-ai-cover-pipeline-wechat-en?utm_source=github&utm_medium=referral)**

Our WeChat newsletter "Coder's Breakfast" ships a tech-news digest every morning at 8. The content side has been fully automated for months, but the cover image remained the weak link: either an operator hunted for a picture by hand, or an SVG template spat out an information card — tidy, but flat, with zero click appeal in a subscription feed.

This week we automated the cover too: **MiniMax generates the background, code composites the headline, zero manual steps**. This post is the full postmortem of the pipeline, with all three generations of real output below — not concept mockups, but images produced on the same day from the same top news item.

## 1. Start with the Two Hard Constraints

Before writing any code, we ruled out the shortest path — "just ask the AI for a finished cover with the title on it" — because it hits two walls that no amount of prompting can fix:

**Wall one: AI-rendered CJK text is always garbled.** This is a limitation of current text-to-image models, not a prompt-engineering problem. And the single most important element of a news cover is its Chinese headline — if the title is mush, the cover is dead.

**Wall two: the aspect ratios don't match.** WeChat headline covers require **2.35:1** (a little-known spec — upload 16:9 and it gets cropped), while the widest ratio MiniMax image-01 supports is 21:9, roughly 2.33:1.

Both constraints point to the same architecture, summed up in one sentence: **AI owns the mood, code owns the information.**

```
MiniMax renders a 21:9 marketing-style background (prompt bans all text)
    ↓
sharp center-crops to exactly 2.35:1 (940×400)
    ↓
SVG text layer composited on top: gradient scrim + brand badge + headline
    ↓
Upload to Tencent COS → Playwright pushes it into the WeChat draft
```

The headline is drawn as SVG, so it is always razor-sharp, never garbled, and automatically tracks each day's top story.

## 2. First Attempt: One Adjective Shoved the Subject Out of Frame

Our first background prompt included `generous negative space` — anyone who has done design work knows white space reads as premium. The result:

![Version 1: subject crammed to one side, the rest of the frame empty](https://cdn.tools.cooconsbit.com/article-images/wechat-ai-cover/v1-offcenter.png)

The entire subject got crammed into one side, leaving 60% of the frame as flat empty color. On a square canvas, negative space is composition; on a 21:9 banner, it is a disaster — the model interprets "negative space" as "pile everything into one corner."

...

---

**[👉 Continue reading: Auto-Generating Daily WeChat Covers with MiniMax: A Production Pipeline Postmortem](https://tools.cooconsbit.com/en/articles/minimax-ai-cover-pipeline-wechat-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
