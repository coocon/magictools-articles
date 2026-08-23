# Google 说我的页面'被 robots.txt 屏蔽'？——别急着改代码，先用 URL Inspection API 查官方判定

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/seo-gsc-url-inspection-pitfalls?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/seo-gsc-url-inspection-pitfalls?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：Google 说我的页面'被 robots.txt 屏蔽'？——别急着改代码，先用 URL Inspection API 查官方判定](https://tools.cooconsbit.com/zh/articles/seo-gsc-url-inspection-pitfalls?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
