---
title: "用 Cloudflare R2 自建图床：从建桶到预签名直传的完整实战"
slug: cloudflare-r2-image-hosting-guide
category: developer
locale: zh
translationSlug: cloudflare-r2-image-hosting-guide-en
tags: [Cloudflare, R2, 对象存储, 图床, S3, Next.js, 预签名 URL, CORS]
summary: "R2 出站流量免费，这一条就足以让它成为图床的默认选项。本文完整走一遍：建桶、绑自定义域、申请 S3 凭据、配 CORS、写预签名直传代码，以及复制直链 / Markdown / HTML 的落地细节。含签名把 Content-Type 签进去导致 403 这类真实坑位。"
status: published
---

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

- 文件走两遍网络，用户等待时间翻倍；
- 你的服务器要扛住上传带宽和内存，Next.js 的 body 体积上限还得往上调；
- 大文件容易把内存打爆，或者被迫落临时盘。

正确做法是**预签名 URL**：服务端用密钥算出一条"允许在 5 分钟内往这个 key 写一次"的临时链接，返回给浏览器，浏览器拿着它直接 PUT 到 R2。文件字节一次都不经过你的服务器，服务端只做鉴权、限额和签名。

### 签名怎么算

R2 用的是标准 AWS SigV4 query 签名。现成轮子有 `@aws-sdk/s3-request-presigner`（功能全，但会给打包体积加几 MB）和 `aws4fetch`（65KB）。我们这次只需要 PUT 和 DELETE 两个动作，索性用 `node:crypto` 直接写，一个依赖都不加：

```ts
import { createHash, createHmac } from "node:crypto";

const ALGORITHM = "AWS4-HMAC-SHA256";

function hmac(key: Buffer | string, data: string): Buffer {
  return createHmac("sha256", key).update(data, "utf8").digest();
}

function getSigningKey(secret: string, dateStamp: string, region: string) {
  const kDate = hmac(`AWS4${secret}`, dateStamp);
  const kRegion = hmac(kDate, region);
  const kService = hmac(kRegion, "s3");
  return hmac(kService, "aws4_request");
}

export function buildPresignedUrl({ method, key, config, expires = 300 }) {
  const amzDate = new Date().toISOString().replace(/[:-]|\.\d{3}/g, "");
  const dateStamp = amzDate.slice(0, 8);

  const host = `${config.accountId}.r2.cloudflarestorage.com`;
  const canonicalUri = `/${config.bucket}/${key}`;
  // R2 的 region 固定写 auto
  const credentialScope = `${dateStamp}/auto/s3/aws4_request`;

  const query = {
    "X-Amz-Algorithm": ALGORITHM,
    "X-Amz-Credential": `${config.accessKeyId}/${credentialScope}`,
    "X-Amz-Date": amzDate,
    "X-Amz-Expires": String(expires),
    "X-Amz-SignedHeaders": "host",
  };

  const canonicalQueryString = Object.keys(query)
    .sort()
    .map((k) => `${encodeURIComponent(k)}=${encodeURIComponent(query[k])}`)
    .join("&");

  const canonicalRequest = [
    method,
    canonicalUri,
    canonicalQueryString,
    `host:${host}\n`,
    "host",
    "UNSIGNED-PAYLOAD",
  ].join("\n");

  const stringToSign = [
    ALGORITHM,
    amzDate,
    credentialScope,
    createHash("sha256").update(canonicalRequest, "utf8").digest("hex"),
  ].join("\n");

  const signature = createHmac(
    "sha256",
    getSigningKey(config.secretAccessKey, dateStamp, "auto")
  )
    .update(stringToSign, "utf8")
    .digest("hex");

  return `https://${host}${canonicalUri}?${canonicalQueryString}&X-Amz-Signature=${signature}`;
}
```

三个容易写错的地方：

**① `X-Amz-SignedHeaders` 只写 `host`。** 如果把 `content-type` 也签进去，浏览器 PUT 时发出的 `Content-Type` 必须和签名时用的字符串**一字不差**，包括 `charset` 部分和大小写。浏览器对某些类型会自己补参数，于是线上随机出现 403 SignatureDoesNotMatch，本地却复现不出来。只签 host 就没有这个耦合。

**② region 写 `auto`。** R2 没有 AWS 那套区域概念，凭据范围里写 `auto`，写成 `us-east-1` 会签名不匹配。

**③ payload 哈希写 `UNSIGNED-PAYLOAD`。** 预签名场景下服务端不可能提前知道文件内容的 SHA-256，用这个常量占位。

自己手写签名一定要有测试兜底。AWS 官方文档给了一组 query 签名的测试向量（`examplebucket.s3.amazonaws.com/test.txt`，固定 key 和时间，期望签名 `aeeed9bb...`），把它跑通，签名的数学部分就可以放心了：

```ts
test("与 AWS 官方测试向量一致", () => {
  const url = signQueryUrl({
    method: "GET",
    host: "examplebucket.s3.amazonaws.com",
    canonicalUri: "/test.txt",
    region: "us-east-1",
    service: "s3",
    accessKeyId: "AKIAIOSFODNN7EXAMPLE",
    secretAccessKey: "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
    amzDate: "20130524T000000Z",
    expires: 86400,
  });
  assert.match(url, /X-Amz-Signature=aeeed9bbccd4d02ee5c0109b86d86835f995330da4c265957d157751f604d404$/);
});
```

还有一个细节：对象 key 里的空格和中文要按 RFC 3986 编码，但 `/` 不能编（它是路径分隔符）。`encodeURIComponent` 会漏掉 `!'()*` 这几个字符，需要补一手替换，否则文件名里带括号的图片会签名失败。

## 第六步：Next.js 里的两端代码

### 服务端：签名接口

```ts
// src/app/api/upload/r2/presign/route.ts
export async function POST(request: NextRequest) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ code: 401, msg: "请先登录" }, { status: 401 });
  }

  const { fileName, fileSize, fileType } = await request.json();

  // 类型白名单 + 体积上限
  const invalid = validateUploadRequest({ fileName, fileSize, fileType });
  if (invalid) {
    return NextResponse.json({ code: invalid.code, msg: invalid.msg }, { status: 422 });
  }

  // 每日配额（管理员跳过）
  if (!isAdmin(session.user.email)) {
    const todayStart = new Date();
    todayStart.setHours(0, 0, 0, 0);
    const count = await prisma.upload.count({
      where: { userId: session.user.id, createdAt: { gte: todayStart } },
    });
    if (count >= DAILY_UPLOAD_LIMIT) {
      return NextResponse.json({ code: 403, msg: "今日上传次数已用完" }, { status: 403 });
    }
  }

  const key = generateKey(session.user.id, fileName, "r2");
  return NextResponse.json({
    code: 0,
    msg: "签名生成成功",
    data: { key, uploadUrl: await getPresignedUrl(key), fileUrl: getFileUrl(key) },
  });
}
```

对象 key 的设计值得多说一句。我们用的是：

```
r2/{userId}/{yyyy-MM-dd}/{时间戳}-{8位随机}.{扩展名}
```

- 带 `userId` 便于按人清理和排查；
- 带日期天然分片，桶里几十万个对象也不会挤在一个前缀下；
- 时间戳 + 随机串保证不撞、且**不可猜**——如果直接用原文件名，别人可以顺着链接遍历你的桶；
- 顶上留一层 `r2/` 前缀，是因为我们的上传记录表和另一套腾讯云 COS 图床共用，靠 key 前缀就能分辨对象在哪个后端，不用给数据库加列。

### 浏览器端：带进度条的直传

`fetch` 目前拿不到上传进度，要进度条就还得用 `XMLHttpRequest`：

```ts
const presign = await fetch("/api/upload/r2/presign", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ fileName: file.name, fileSize: file.size, fileType: file.type }),
}).then((r) => r.json());

if (presign.code !== 0) throw new Error(presign.msg);
const { key, uploadUrl, fileUrl } = presign.data;

await new Promise<void>((resolve, reject) => {
  const xhr = new XMLHttpRequest();
  xhr.upload.onprogress = (e) => {
    if (e.lengthComputable) setProgress(Math.round((e.loaded / e.total) * 100));
  };
  xhr.onload = () =>
    xhr.status >= 200 && xhr.status < 300
      ? resolve()
      : reject(new Error(`上传失败 HTTP ${xhr.status}`));
  xhr.onerror = () => reject(new Error("网络错误"));
  xhr.open("PUT", uploadUrl);
  xhr.setRequestHeader("Content-Type", file.type);
  xhr.send(file);
});

// 上传成功后再入库，避免"数据库里有记录、桶里没文件"
await createUploadRecord({ fileName: file.name, fileKey: key, fileUrl, fileSize: file.size, fileType: file.type });
```

注意入库的时机：**先传完，再写数据库**。反过来的话，用户中途取消或网络断了，数据库里就留下一条指向 404 的记录。

## 第七步：预览与三种复制格式

图床好不好用，八成体验在"传完之后那一下"。至少给三种复制格式：

```ts
function getCopyLinks(fileName: string, fileUrl: string, fileType: string) {
  const isImage = fileType.startsWith("image/");
  return [
    { label: "链接", value: fileUrl },
    {
      label: "MD",
      value: isImage ? `![${fileName}](${fileUrl})` : `[${fileName}](${fileUrl})`,
    },
    {
      label: "HTML",
      value: isImage
        ? `<img src="${fileUrl}" alt="${fileName}" />`
        : `<a href="${fileUrl}" download>${fileName}</a>`,
    },
  ];
}
```

几个体验细节：

- **Markdown 要区分图片和普通文件**：图片用 `![]()`，其他用 `[]()`，否则拖一个 zip 进去会生成一个渲染不出来的图片语法。
- **`navigator.clipboard` 要有降级**。它只在 HTTPS 和 localhost 下可用，内网 IP 访问时会直接抛异常，得 fallback 到 `document.execCommand("copy")`。
- **复制成功要有反馈**，按钮图标切成对勾停 2 秒就够，没有反馈用户会连点好几次。
- **图片列表用缩略图 + 点击放大**，比一列文件名有用得多。

## 第八步：删除、配额与防滥用

**删除要删两处。** 数据库记录和 R2 对象都得删，只删记录的话对象会永远躺在桶里付费。R2 的删除同样可以用预签名 DELETE，不用再单独签一次 Authorization 头：

```ts
export async function deleteObject(key: string): Promise<void> {
  const url = buildPresignedUrl({ method: "DELETE", key, config: getR2Config(), expires: 60 });
  const res = await fetch(url, { method: "DELETE" });
  // 成功返回 204；对象本来就不存在（404）也按成功处理
  if (!res.ok && res.status !== 404) {
    throw new Error(`R2 删除失败 ${res.status}`);
  }
}
```

顺便说一个我们自己踩过的坑：如果站点同时接了两套对象存储，删除逻辑一定要**按对象 key 分派到对应的后端**。我们早期的代码无条件调腾讯云 COS 的删除接口，而且异常被 `try/catch` 吞掉只打了日志——表现是"页面上删掉了，用户什么都看不出来，桶里的文件一个没少"。这类静默失败比直接报错难查得多。

**配额与防滥用**，服务端至少要做这几件事：

- **登录才能传**，匿名图床迟早变成别人的免费 CDN；
- **每日次数上限**，按用户统计，管理员豁免；
- **类型白名单 + 体积上限**，在签名接口里就拦掉，别等传完再说；
- **限额检查放在服务端**，前端那份校验只是提前给用户反馈，不能当安全边界。

如果想更严，还可以在自定义域上挂 Cloudflare 的 WAF 规则或 Rate Limiting，对热点链接限速。

## 成本估算，以及什么时候别用 R2

一个中等体量的博客图床，假设 5000 张图、平均 300KB，共 1.5GB，每月新增 500 张：

- 存储：1.5GB × $0.015 = **$0.023/月**（还在 10GB 免费额度内，实际 $0）
- Class A 操作（上传/删除）：500 次，免费额度 100 万/月，**$0**
- Class B 操作（读取）：假设 50 万次，免费额度 1000 万/月，**$0**
- 出站流量：无论多少，**$0**

也就是说这个量级基本白嫖。同样的流量放在 S3 上，光出站就是几十美元一个月。

什么时候不该用 R2：

- **需要图片处理**（缩略图、裁剪、WebP 转换、水印）。R2 只存不处理，得自己在上传前做，或者额外上 Cloudflare Images / Workers。
- **国内访问为主**。Cloudflare 免费版在国内的线路质量不稳定，对国内用户为主的站点，国内云厂商的 CDN 更稳。
- **强合规要求的数据落地**。R2 的位置只能给到大区级别，做不到精确到某国某地域。
- **要用 S3 的高级特性**。版本控制、生命周期规则、事件通知这些，R2 的支持程度和 S3 有差距，上之前先确认。

## 小结

搭完一整套之后回头看，真正卡人的就那么几处：公开访问必须绑自定义域、CORS 必须提前配、签名别把 `Content-Type` 签进去、region 写 `auto`、删除要同时删对象和记录。剩下的都是常规的鉴权与限额工程。

不想自己搭的话，可以直接用我们做好的 [Cloudflare R2 图床](/tools/r2-upload)：登录后拖进去就传，出来直链、Markdown、HTML 三种格式一键复制，文件从浏览器直传 R2，不经过中转服务器。
