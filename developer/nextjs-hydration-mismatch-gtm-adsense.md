---
title: "Next.js 全站 hydration mismatch：根因是 GTM 载入的 AdSense，不是你的代码"
slug: "nextjs-hydration-mismatch-gtm-adsense"
category: developer
locale: zh
source: authored
tags:
  - nextjs
  - react
  - hydration
  - gtm
  - adsense
  - troubleshooting
summary: "Next.js 站点控制台随机出现 A tree hydrated but some attributes didn't match，且冷缓存首个页面干净、之后页面必报？根因不在你的组件，而是 GTM 拉起的 AdSense 自动广告在 React 水合前向 head 注入脚本，位置撞上了内联 GTM script。本文讲清这个时序谜题、一行修复方案，以及用 Playwright 抓完整报错 diff 的诊断技巧。"
coverImage: ""
status: published
scheduledAt: ""
---

## 一、问题描述

Next.js（App Router）站点的浏览器控制台随机冒出 hydration 警告：

```
A tree hydrated but some attributes of the server rendered HTML
didn't match the client properties.
```

说它"随机"，是因为复现规律极其怪异：

- **冷缓存**（首次访问 / 硬刷新）打开的第一个页面：控制台干净
- 之后再点开的**任何页面：必报**
- 报错和具体页面无关——工具页、文章页、首页都能触发
- 本地 dev 和线上都能复现

按常规思路把嫌疑组件查了个遍：没有 `Date.now()` / `Math.random()` 直接进渲染，没有 `typeof window` 分支渲染，没有浏览器插件注入（无痕模式照样报）。**页面代码是清白的。**

## 二、环境

| 项目 | 详情 |
|------|------|
| 框架 | Next.js 16（App Router，React 19） |
| 打点 | Google Tag Manager 内联片段（layout.tsx 直出，服务于 GSC 站点验证） |
| 广告 | AdSense 自动广告，经由 GTM 加载 |

## 三、排查过程

**第一步：拿到完整的报错 diff。** 控制台里的警告是折叠的，肉眼看不出到底哪个属性不匹配。用 Playwright 无头浏览器抓：

```js
page.on("console", async (msg) => {
  // 注意：msg.text() 只有 "%s" 占位符，拿不到实际内容
  for (const arg of msg.args()) {
    console.log(await arg.evaluate(String));
  }
});
await page.goto(url, { waitUntil: "load" }); // dev 模式 networkidle 永不触发
```

两个容易浪费时间的小坑：

- `msg.text()` 对 React 这类多参数 console 调用只返回 `%s` 占位符，必须遍历 `msg.args()` 逐个 evaluate 才能拿到完整 diff
- dev 模式下 HMR 的长连接让 `networkidle` 永远不满足，等待条件要用 `load`

**第二步：读 diff。** 完整输出显示不匹配发生在 `<head>` 里的 **GTM 内联 script** 上——React 期望那个位置是我们 layout.tsx 里直出的 GTM 片段，实际 DOM 里那个位置却是一个 `pagead2.googlesyndication.com` 的脚本标签。

**第三步：解释冷热缓存的时序差。** 这是整个问题最有意思的部分：

1. GTM 片段在 SSR HTML 里直出（为了 Google 站点验证，必须直出）
2. GTM 启动后拉起 AdSense 自动广告，AdSense 会**向 `<head>` 动态注入**自己的 script
3. React 水合时对 head 内的 script 按**位置**匹配
4. 冷缓存时：AdSense 脚本要现下载，注入发生在**水合之后** → 匹配正常，控制台干净
5. 热缓存时：AdSense 脚本秒加载，注入发生在**水合之前** → head 里多出一个不在服务端 HTML 里的脚本，位置恰好撞上 React 要匹配的内联 GTM script → 报错

"冷缓存首页干净、之后必报"这个特征完美对应"脚本是否已在浏览器缓存里"——**这不是代码 bug，是第三方脚本注入与 React 水合的竞速**，谁快谁赢。

## 四、修复：一行属性，但要修对地方

给 layout.tsx 里的 GTM 内联 script 加 `suppressHydrationWarning`：

```tsx
<script
  id="gtm"
  suppressHydrationWarning
  dangerouslySetInnerHTML={{ __html: gtmSnippet }}
/>
```

这个属性只是告诉 React"这个节点的不匹配是预期的，别警告"——**SSR 输出的 HTML 一个字节都不会变**，GSC 站点验证不受任何影响。

### 一个看似更"正规"、实则错误的方案

把 GTM 换成 `next/script` 的 `strategy="afterInteractive"` 也能消掉警告，但**不要这么做**：afterInteractive 意味着 GTM 片段不再出现在 SSR HTML 里，而 Google 站点所有权验证（HTML 标签方式）和部分爬虫检测依赖**首屏 HTML 里直出的片段**。为了消一个无害警告丢掉站点验证，得不偿失。

## 五、验证

修复前后用同一套 Playwright 脚本扫全站主要页面：

- 修复前：5 个页面中 4 个报 hydration 警告（唯一干净的是冷缓存首个页面）
- 修复后：5/5 全部干净，SSR HTML diff 为零

## 六、这个问题值得修吗？

hydration mismatch 警告本身不影响功能——React 会以客户端为准继续跑。但放着不修有两个实际代价：

1. **它会淹没真警告。** 全站常态化报错后，哪天你自己的组件真写出了水合问题，没人会注意到多了一条
2. **排查成本会转嫁给未来。** 每个新同事（或未来的你）看到控制台红字都会重新排查一轮，而这个根因藏得足够深，每轮都不便宜

## 七、排查口诀

- hydration 警告先看**复现模式**：所有页面都报、和页面代码无关 → 查全局注入（打点、广告、浏览器插件），别逐个组件排查
- **冷缓存干净、热缓存必报** → 竞速类问题，嫌疑人是"加载速度会变的第三方脚本"
- 折叠的 React 警告用 Playwright `msg.args()` 逐个 evaluate 拿全文，别对着省略号猜
- `suppressHydrationWarning` 只该用在**已确认根因、且不匹配确属预期**的节点上——它是精确豁免，不是消音器
