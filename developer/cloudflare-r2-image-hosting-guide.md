# 用 Cloudflare R2 自建图床：从建桶到预签名直传的完整实战

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/cloudflare-r2-image-hosting-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/cloudflare-r2-image-hosting-guide?utm_source=github&utm_medium=referral)**

## 为什么图床值得换到 R2

图床的成本大头从来不是存储，是**出站流量**。一张 200KB 的配图放进一篇被读了 10 万次的文章，产生的存储费不到一分钱，流量费却是它的几百倍。

写这篇时几家的价目（以各家官网为准）：

| | 存储 | 出站流量 | 免费额度 |
|---|---|---|---|
| Cloudflare R2 | $0.015/GB/月 | **$0** | 10GB 存储 / 月 |
| AWS S3 Standard | $0.023/GB/月 | ~$0.09/GB（前 10TB） | 新账号首年少量 |
| 各家 CDN 回源方案 | 存储另计 | $0.02–0.15/GB 不等 | 视套餐 |

R2 的出站是真的不收钱——不是"前多少 GB 免费"，是这一项在账单上根本不存在。对图床这种"写一次、读无数次"的场景，成本模型直接从"随访问量线性增长"变成"随文件总量增长"。

另外两点也重要：

- **S3 兼容**。R2 说的是 S3 API，任何 S3 客户端、任何写好的 SigV4 签名代码都能直接用，将来想迁走不用重写。
- **自定义域走 Cloudflare 边缘网络**。绑上 `img.example.com` 之后，文件由 Cloudflare 的 CDN 分发，全球访问都快，而且域名是你自己的——图床换后端时不用满世界改老链接。

代价是：R2 只做存储和分发，不做图片处理。要缩略图、格式转换、水印，得自己在上传前处理，或者叠加 Cloudflare Images / Workers。

## 第一步：建桶

Cloudflare Dashboard → R2 → Create bucket。名字随便取（本文用 `tingease`），位置选 Automatic 就行，R2 会按访问情况自动选。

建完先记下 **Account ID**——它在 dashboard URL 里：`dash.cloudflare.com/{ACCOUNT_ID}/r2/...`。S3 endpoint 是拿它拼出来的：

```
https://{ACCOUNT_ID}.r2.cloudflarestorage.com
```

## 第二步：公开访问，别用 r2.dev

新建的桶默认完全私有，直接访问对象会 401。开放读有两条路：

**① r2.dev 开发域名** —— 桶设置里一键开启，拿到一个 `pub-xxxx.r2.dev` 的地址。**只适合本地调试**：Cloudflare 明确说它带速率限制、不保证可用性，也不进 CDN 缓存策略。图床链接一旦发出去就是永久的，用它等于给自己埋雷。

**② 自定义域（生产用这个）** —— Settings → Public access → Connect Domain，填一个托管在 Cloudflare 上的域名，比如 `img.example.com`。CF 会自动建好 DNS 记录并挂上证书，之后这个域名的请求就直接命中 R2，并享受完整的 CDN 缓存。

绑完之后，一个对象的公开地址就是：

```
https://img.example.com/{对象 key}
```

## 第三步：申请 S3 凭据，权限给到刚好够用

R2 → API → Create API token：

- **权限选 Object Read & Write**，不要给 Admin。图床只需要读写对象，不需要建桶删桶。
- **范围限定到具体的桶**，别给"所有桶"。
- 创建后会显示 **Access Key ID** 和 **Secret Access Key**，Secret 只显示这一次。

存进环境变量，别提交进仓库：

```env
R2_ACCOUNT_ID=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
R2_ACCESS_KEY_ID=xxxxxxxx
R2_SECRET_ACCESS_KEY=xxxxxxxx
R2_BUCKET=tingease
R2_PUBLIC_DOMAIN=img.example.com
```

顺带一提：这套 Access Key 只能待在服务端。下面的直传方案之所以成立，正是因为浏览器全程拿不到它，只拿到一条有效期几分钟的签名链接。

## 第四步：CORS 必须配，否则浏览器直传必挂

浏览器向 `*.r2.cloudflarestorage.com` 发 PUT，属于跨域请求，会先发 `OPTIONS` 预检。桶没配 CORS 的话预检就被拒，控制台报的是"CORS policy"错误，跟签名没关系——很多人卡在这里去反复检查签名，方向就错了。

桶 Settings → CORS Policy，贴这段（域名换成你自己的）：

```json
[
  {
    "AllowedOrigins": [
      "https://tools.example.com",
      "http://localhost:3001"
    ],
    "AllowedMethods": ["PUT", "GET"],
    "AllowedHeaders": ["content-type"],
    "ExposeHeaders": ["ETag"],
    "MaxAgeSeconds": 3600
  }
]
```

要点：

- `AllowedOrigins` 写具体来源，别图省事写 `*`——写了 `*` 等于任何网站都能拿你的签名链接往桶里写。
- `AllowedHeaders` 要包含 `content-type`，因为上传时会带这个头。
- 本地开发端口要单独列进去，`localhost` 和线上域名是两个来源。

## 第五步：预签名直传，别让文件过服务器

最容易想到的做法是"浏览器 POST 给自己的服务器，服务器再转存到 R2"。别这么干：

...

---

**[👉 继续阅读全文：用 Cloudflare R2 自建图床：从建桶到预签名直传的完整实战](https://tools.cooconsbit.com/zh/articles/cloudflare-r2-image-hosting-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
