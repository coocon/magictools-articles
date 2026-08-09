---
title: "Google 说我的页面'被 robots.txt 屏蔽'？——别急着改代码，先用 URL Inspection API 查官方判定"
slug: seo-gsc-url-inspection-pitfalls
category: pitfalls
locale: zh
tags: [SEO, Google Search Console, 多语言站点, Bug 复盘]
summary: "一条'页面未收录、被 robots.txt 屏蔽'的反馈，差点让我们推翻整个多语言 301 架构。用 GSC URL Inspection API 一查：robots.txt 状态 ALLOWED，真实状态是'网页会自动重定向'——内容早已在目标 URL 正常收录。本文完整复盘排查链路、多语言站的 SEO 架构取舍，以及一套可复用的'查 Google 官方判定'方法论。"
status: published
---

我们的站点是中英双语架构：URL 带 `/zh/`、`/en/` 前缀，每篇文章只有一种语言，中英对版是两条独立记录、两个独立 slug。前几天收到反馈：

> `https://example.com/en/articles/guide-cron-20260304` 会自动跳转，Google 说没有收录，**已被 robots.txt 屏蔽**。

紧接着一个架构级的提议：既然英文前缀会触发跳转，是不是应该升级路由——让 `zh/en` 前缀只控制界面语言，文章不再自动跳转？

如果当时直接照做，我们就会为一个**不存在的问题**推翻一套教科书级正确的 SEO 架构。这篇文章复盘完整排查过程：三个小坑、一个关键方法论、一次架构取舍讨论。

## 第一步：核对 robots.txt——矛盾出现

"被 robots.txt 屏蔽"是个可以立刻证伪的判定。我们的 robots 规则由 Next.js 的 `robots.ts` 生成，屏蔽列表只有这些：

```ts
const disallow = [
  "/api/",
  // 同时列出带/不带尾斜杠两种形态："/*/dashboard/" 拦不住 "/en/dashboard" 本身
  "/*/dashboard",
  "/*/dashboard/",
  "/*/auth/",
  "/*/tweets",
  "/*/tweets/*",
];
```

`/en/articles/guide-cron-20260304` 不匹配任何一条。拉取线上 `robots.txt` 逐行核对，结论一致：**robots.txt 不可能拦这个 URL**。

到这里就出现了排查中最有价值的信号——**转述的结论和一手证据对不上**。这时候不要试图"调和"两边（比如猜"也许 Google 缓存了旧版 robots.txt"），而是去找更权威的一手数据源。

## 第二步：curl 探测——先踩了自己埋的坑

想看这个 URL 到底怎么跳转，最直接的是 curl：

```bash
$ curl -sI "https://example.com/en/articles/guide-cron-20260304"
HTTP/2 403
```

403？页面浏览器里明明能打开。

原因是我们自己几个月前部署的反爬 middleware：非浏览器 User-Agent（`curl`、`python-requests`、`HeadlessChrome` 等）会被直接拒绝。**测自己的站，先想想自己给自己设过什么门禁。** 带上浏览器 UA 再测：

```bash
$ curl -sI -A "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) ... Chrome/127.0" \
    "https://example.com/en/articles/guide-cron-20260304"
HTTP/2 308
location: https://example.com/en/articles/guide-cron-20260304-en
```

跳转链路清楚了：这是一篇**中文**文章的 slug，用 `/en/` 前缀访问时，服务端发现它有已发布的英文对版 `guide-cron-20260304-en`，于是 308 永久重定向过去。这正是设计行为，对应的代码：

```ts
// 跨 locale 访问 → 301 永久重定向
// 1. 文章有对版且对版已发布 → 重定向到对版 URL（用户看到自己语言的对应内容）
// 2. 否则                   → 重定向到原文真实 locale 的 URL（用户看到原文）
if (requestLocale !== articleLocale) {
  if (article.translationSlug) {
    const statusResult = await getArticleStatusBySlug(article.translationSlug);
    if (statusResult.code === 0 && statusResult.data?.status === "published") {
      permanentRedirect(
        buildLocalizedUrl(requestLocale, `/articles/${article.translationSlug}`),
      );
    }
  }
  permanentRedirect(
    buildLocalizedUrl(articleLocale, `/articles/${article.slug}`),
  );
}
```

行为符合预期。那 Google 那边到底怎么判定的？

## 第三步（关键）：别看界面转述，直接问 Google——URL Inspection API

Search Console 的界面报告有两个天然的失真源：

1. **分组名相近**。"未编入索引"下面有十几种原因分组：'网页会自动重定向'、'已被 robots.txt 屏蔽'、'已抓取但尚未编入索引'……人工转述时很容易串行。
2. **数据是快照**。界面报告更新有延迟，看到的可能是几周前的状态。

而 [URL Inspection API](https://developers.google.com/webmaster-tools/v1/urlInspection.index/inspect) 返回的是 Google 索引库里对这个 URL 的**当前官方判定**，字段是结构化的，没有翻译损耗。一个可复用的最小脚本（Node.js + Service Account）：

```ts
// gsc-inspect-url.ts —— 查询任意 URL 的 Google 官方收录判定
// 用法：npx tsx gsc-inspect-url.ts <url>
import { GoogleAuth } from "google-auth-library";

const url = process.argv[2];

async function main() {
  const auth = new GoogleAuth({
    keyFile: ".secrets/gsc-sa-key.json", // Service Account 密钥
    scopes: ["https://www.googleapis.com/auth/webmasters.readonly"],
  });
  const client = await auth.getClient();

  // 注意：用 client.request()（gaxios），不要用 Node 原生 fetch——
  // 原生 fetch 不遵守 https_proxy 环境变量，代理环境下会 ConnectTimeout
  const res = await client.request({
    url: "https://searchconsole.googleapis.com/v1/urlInspection/index:inspect",
    method: "POST",
    data: {
      inspectionUrl: url,
      // 必须与 GSC 属性类型严格一致：
      // URL-prefix 属性 → "https://example.com/"（带尾斜杠）
      // Domain 属性     → "sc-domain:example.com"
      // 填错会报 "You do not own this site"
      siteUrl: "https://example.com/",
    },
  });
  console.log(JSON.stringify(res.data, null, 2));
}

main().catch((e) => { console.error(e); process.exit(1); });
```

配置只有三步：GSC 后台把 Service Account 邮箱加为属性用户（完整权限）、下载 SA 密钥、装 `google-auth-library`。脚本里埋着两个我们实际踩过的坑，都写在注释里了：

- **坑 A：Node 原生 fetch 不认代理。** `fetch` 底层是 undici，不读 `https_proxy` 环境变量；在需要代理访问 Google API 的网络环境下会一直 `ConnectTimeout`。换成 google-auth-library 自带的 `client.request()`（底层 gaxios）即可。
- **坑 B：`siteUrl` 必须和属性类型精确匹配。** 我们第一次填了 `sc-domain:example.com`，报错 "You do not own this site"——实际属性是 URL-prefix 类型，必须写成 `https://example.com/`，尾斜杠都不能少。不确定时先调 `GET /webmasters/v3/sites` 列出 SA 有权限的属性。

## 判定结果：不是 robots.txt，是"网页会自动重定向"——而这是正常态

对出问题的 URL 跑一遍：

```json
{
  "indexStatusResult": {
    "verdict": "NEUTRAL",
    "coverageState": "Page with redirect",
    "robotsTxtState": "ALLOWED",
    "indexingState": "INDEXING_ALLOWED",
    "pageFetchState": "SUCCESSFUL",
    "googleCanonical": "https://example.com/en/articles/guide-cron-20260304-en",
    "referringUrls": [
      "https://example.com/en/articles/guide-cron-20260304-en",
      "https://example.com/en/articles"
    ]
  }
}
```

三个字段直接终结了"robots.txt 屏蔽"的假设：

- `robotsTxtState: "ALLOWED"` —— robots.txt 没有拦它，Google 亲口说的；
- `coverageState: "Page with redirect"` —— 真实分组是"网页会自动重定向"；
- `googleCanonical` 指向 `-en` 英文对版 —— Google 完全理解了我们的意图。

再查跳转目标 `guide-cron-20260304-en`：

```json
{
  "verdict": "PASS",
  "coverageState": "Submitted and indexed",
  "sitemap": ["https://example.com/sitemap.xml"]
}
```

**内容根本没有丢收录。** 英文文章在它自己的 URL 上"已提交并编入索引"，富结果校验也通过。GSC 把重定向源 URL 列在"未编入索引"里，只是如实陈述"这个 URL 跳到了别处，我收录了目标页"——这不是故障告警，是系统按设计工作的日志。

## 那要不要改成"不跳转"？——多语言架构的 SEO 取舍

回到最初的提议：`zh/en` 前缀只控制界面语言，`/en/` 前缀访问中文文章时直接渲染、不再 301。从 SEO 角度这是个**负优化**，三个理由：

**1. 制造重复内容。** 同一篇中文文章会在 `/zh/articles/x` 和 `/en/articles/x` 两个 URL 下都返回 200。即使 canonical 都指向 `/zh/`，也是在让 Google 抓双倍页面、再靠推断合并——canonical 是"建议"，301 是"指令"，信号强度差一个量级。

**2. 污染语言信号。** URL 前缀声明 `en`，`html lang` 和正文却是中文，hreflang 体系里出现自相矛盾的数据。多语言 SEO 的全部前提是"每个 URL 的语言身份唯一且自洽"。

**3. 现状就是 Google 官方推荐的架构。** 一语言一 URL、自引用 canonical、双向 hreflang、跨语言访问 301 到对版——我们逐项核对了两个页面的实际输出：

```html
<!-- 中文页 /zh/articles/guide-cron-20260304 -->
<link rel="canonical" href="https://example.com/zh/articles/guide-cron-20260304"/>
<link rel="alternate" hreflang="zh-CN" href="https://example.com/zh/articles/guide-cron-20260304"/>
<link rel="alternate" hreflang="en" href="https://example.com/en/articles/guide-cron-20260304-en"/>

<!-- 英文页 /en/articles/guide-cron-20260304-en -->
<link rel="canonical" href="https://example.com/en/articles/guide-cron-20260304-en"/>
<link rel="alternate" hreflang="en" href="https://example.com/en/articles/guide-cron-20260304-en"/>
<link rel="alternate" hreflang="zh-CN" href="https://example.com/zh/articles/guide-cron-20260304"/>
```

canonical 有个容易做错的细节：它必须指向**文章真实语言**的 URL，与请求路径无关。用户拿错误前缀访问时，metadata 里的 canonical 和 301 的目标保持一致，Google 收到的两路信号互相印证：

```ts
// canonical 必须指向文章真实 locale 的 URL，与 request locale 无关。
// 即使用户访问了错误 locale 的 URL，页面也会 301 到真实 locale；
// 此时 metadata 的 canonical 和 301 目标保持一致，信号最干净。
const canonicalUrl = buildLocalizedUrl(articleLocale, `/articles/${article.slug}`);
```

还有一条防御性规则：**hreflang 只在对版"已发布"时才声明**（sitemap 和页面 metadata 双重校验）。对版还是草稿时就声明 alternate，等于给 Google 递一个会 301 回源或 404 的 URL，是主动送出去的错误信号。

结论：**架构不动。** 唯一值得做的是下一节。

## 意外收获：referringUrls 暴露了一个真问题

API 返回的 `referringUrls` 字段告诉你 Google 是**从哪里发现**这个 URL 的。上面的结果显示，它是从 `/en/articles` 列表页链过来的——说明历史上某个版本的英文列表页，曾经输出过"英文前缀 + 中文 slug"的内部链接。

内部链接指向一个注定要 301 的 URL，虽然不致命（Google 会跟随），但每个这样的链接都在浪费一次跳转、稀释一点权重。正确姿势是**内链永远直指最终 URL**。我们实测当前线上列表页已经不再输出这种链接（文章列表和相关文章推荐都按 locale 过滤），所以这些 redirect 条目会随 Google 重抓自然从报告里淡出，无需额外处理。

顺带一提，`referringUrls` 也是排查"Google 为什么会抓到这个奇怪 URL"类问题的最短路径——比在全站代码里 grep 链接快得多。

## 方法论沉淀

1. **"Google 说"要拿到 Google 的原话。** GSC 界面分组名相近、数据有延迟、人工转述会失真。收录疑问一律走 URL Inspection API，看 `coverageState` / `robotsTxtState` / `googleCanonical` 三个结构化字段，五分钟拿到官方判定。
2. **转述与一手证据矛盾时，升级数据源而不是调和矛盾。** 本例中 robots.txt 核对结果与反馈直接冲突，正确动作是去找更权威的判定（API），而不是给"也许缓存了旧 robots.txt"这类猜想打补丁。
3. **"未编入索引"不全是坏消息。** "Page with redirect"、"Alternate page with proper canonical tag" 这类分组是系统按设计工作的日志。改架构之前先确认：内容本体（canonical 目标）是否已收录？是，就什么都不用做。
4. **测自己的站点时，记住自己设过的门禁。** 反爬 UA 黑名单、地域封锁、登录墙——curl 返回的 403 不一定是新问题，可能是几个月前的自己。
5. **多语言站的铁律：一语言一 URL。** 跨语言访问用 301 收敛，canonical 指向真实语言版本，hreflang 只声明已发布的对版，内链直指最终 URL。任何"同一内容多 URL 返回 200"的设计，无论动机多合理，都是在给搜索引擎出难题。
