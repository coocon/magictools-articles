# Next.js 全站 hydration mismatch：根因是 GTM 载入的 AdSense，不是你的代码

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/nextjs-hydration-mismatch-gtm-adsense?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/nextjs-hydration-mismatch-gtm-adsense?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：Next.js 全站 hydration mismatch：根因是 GTM 载入的 AdSense，不是你的代码](https://tools.cooconsbit.com/zh/articles/nextjs-hydration-mismatch-gtm-adsense?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
