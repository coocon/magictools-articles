---
title: "Build Your Own Image Host on Cloudflare R2: Buckets, Presigned Uploads, and the Gotchas"
slug: cloudflare-r2-image-hosting-guide-en
category: developer
locale: en
translationSlug: cloudflare-r2-image-hosting-guide
tags: [Cloudflare, R2, object storage, image hosting, S3, Next.js, presigned URL, CORS]
summary: "R2 charges nothing for egress, which alone makes it the default choice for an image host. This is the full walkthrough: create the bucket, attach a custom domain, mint S3 credentials, configure CORS, and write presigned direct-upload code — including the 403 you get when you sign Content-Type into the signature."
status: published
---

## Why an image host belongs on R2

The expensive part of image hosting has never been storage. It is **egress**. A 200KB illustration inside an article that gets 100,000 reads costs a fraction of a cent to store and several hundred times that to serve.

Prices at the time of writing (always check the vendor's page):

| | Storage | Egress | Free tier |
|---|---|---|---|
| Cloudflare R2 | $0.015/GB/mo | **$0** | 10GB storage/mo |
| AWS S3 Standard | $0.023/GB/mo | ~$0.09/GB (first 10TB) | small, first year |
| CDN-in-front-of-bucket setups | storage extra | $0.02–0.15/GB | varies |

R2's egress is genuinely free — not "the first N gigabytes," the line item does not exist on the bill. For a write-once-read-forever workload like image hosting, that flips the cost model from "grows with traffic" to "grows with total bytes stored."

Two other things matter:

- **S3 compatibility.** R2 speaks the S3 API, so any S3 client or SigV4 signing code works as-is, and migrating away later does not mean a rewrite.
- **A custom domain on Cloudflare's edge.** Attach `img.example.com` and files are served from Cloudflare's CDN under a hostname you own — so swapping the backend later does not break links you have already published.

The tradeoff: R2 stores and serves, it does not transform. Thumbnails, format conversion, and watermarks are on you (before upload) or on Cloudflare Images / Workers.

## Step 1: create the bucket

Cloudflare Dashboard → R2 → Create bucket. Any name works (this guide uses `tingease`); leave the location on Automatic and R2 picks based on access patterns.

Write down your **Account ID** — it is right there in the dashboard URL: `dash.cloudflare.com/{ACCOUNT_ID}/r2/...`. The S3 endpoint is built from it:

```
https://{ACCOUNT_ID}.r2.cloudflarestorage.com
```

## Step 2: public access — do not ship r2.dev

A fresh bucket is fully private; hitting an object directly returns 401. Two ways to open reads:

**① The r2.dev development URL** — one toggle in bucket settings gives you a `pub-xxxx.r2.dev` hostname. **Debugging only.** Cloudflare explicitly rate-limits it and makes no availability guarantee, and it sits outside your normal caching setup. Image links live forever once published, so shipping these is a trap you set for yourself.

**② A custom domain (use this in production)** — Settings → Public access → Connect Domain, then enter a hostname on a zone you have in Cloudflare, e.g. `img.example.com`. Cloudflare creates the DNS record and certificate for you, and requests to that hostname hit R2 with full CDN caching.

After that, an object's public address is simply:

```
https://img.example.com/{object key}
```

## Step 3: credentials, scoped to exactly what you need

R2 → API → Create API token:

- **Pick Object Read & Write**, not Admin. An image host reads and writes objects; it never needs to create or destroy buckets.
- **Scope the token to the one bucket**, not "all buckets."
- You get an **Access Key ID** and a **Secret Access Key**. The secret is shown once.

Put them in environment variables, never in the repo:

```env
R2_ACCOUNT_ID=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
R2_ACCESS_KEY_ID=xxxxxxxx
R2_SECRET_ACCESS_KEY=xxxxxxxx
R2_BUCKET=tingease
R2_PUBLIC_DOMAIN=img.example.com
```

These keys stay server-side, always. The direct-upload design below works precisely because the browser never sees them — it only ever gets a signed URL that expires in minutes.

## Step 4: CORS is mandatory for browser uploads

A `PUT` from your page to `*.r2.cloudflarestorage.com` is cross-origin, so the browser sends an `OPTIONS` preflight first. Without a CORS policy on the bucket the preflight is rejected and the console reports a CORS error — which has nothing to do with your signature. Plenty of people burn an afternoon re-checking signing code here.

Bucket Settings → CORS Policy, paste this (swap in your own origins):

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

Notes:

- List explicit origins. `*` means any website can use a leaked signed URL to write into your bucket.
- `AllowedHeaders` must include `content-type`, since the upload sends it.
- Add your local dev port separately — `localhost` and your production domain are different origins.

## Step 5: presigned direct upload — keep files off your server

The obvious design is "browser POSTs to my API, my API forwards to R2." Do not do that:

- The bytes cross the network twice, so the user waits twice as long.
- Your server has to absorb upload bandwidth and memory, and you end up raising Next.js body-size limits.
- Big files either blow up memory or force you to spool to disk.

Use a **presigned URL** instead: the server uses the secret to compute a link that means "you may write to this one key, once, within the next five minutes," and hands it to the browser, which PUTs straight to R2. Not a single byte of the file touches your server — it only does auth, quota, and signing.

### Computing the signature

R2 uses standard AWS SigV4 query signing. Off-the-shelf options are `@aws-sdk/s3-request-presigner` (complete, but adds megabytes to your bundle) and `aws4fetch` (65KB). We only needed PUT and DELETE, so we wrote it directly against `node:crypto` and added no dependency at all:

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
  // R2's region is always the literal string "auto"
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

Three things people get wrong:

**① Sign `host` only.** If you add `content-type` to `X-Amz-SignedHeaders`, the `Content-Type` the browser actually sends must match the signed string **byte for byte**, including any `charset` parameter and casing. Browsers add parameters on their own for some types, and you get intermittent 403 SignatureDoesNotMatch in production that you cannot reproduce locally. Signing only `host` removes the coupling entirely.

**② The region is `auto`.** R2 has no AWS-style regions; the credential scope uses the literal `auto`. Writing `us-east-1` produces a signature mismatch.

**③ The payload hash is `UNSIGNED-PAYLOAD`.** When presigning, the server cannot know the file's SHA-256 ahead of time, so that constant stands in for it.

If you hand-roll signing, back it with a test. AWS publishes a query-signing test vector (`examplebucket.s3.amazonaws.com/test.txt`, fixed key and timestamp, expected signature `aeeed9bb...`). Get that green and the math is settled:

```ts
test("matches the AWS reference vector", () => {
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

One more detail: spaces and non-ASCII characters in the object key need RFC 3986 encoding, but `/` must stay literal since it is the path separator. `encodeURIComponent` leaves `!'()*` unencoded, so patch those manually — otherwise a filename with parentheses fails to sign.

## Step 6: the Next.js code, both ends

### Server: the signing endpoint

```ts
// src/app/api/upload/r2/presign/route.ts
export async function POST(request: NextRequest) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ code: 401, msg: "Sign in required" }, { status: 401 });
  }

  const { fileName, fileSize, fileType } = await request.json();

  // type allowlist + size cap
  const invalid = validateUploadRequest({ fileName, fileSize, fileType });
  if (invalid) {
    return NextResponse.json({ code: invalid.code, msg: invalid.msg }, { status: 422 });
  }

  // daily quota (admins exempt)
  if (!isAdmin(session.user.email)) {
    const todayStart = new Date();
    todayStart.setHours(0, 0, 0, 0);
    const count = await prisma.upload.count({
      where: { userId: session.user.id, createdAt: { gte: todayStart } },
    });
    if (count >= DAILY_UPLOAD_LIMIT) {
      return NextResponse.json({ code: 403, msg: "Daily limit reached" }, { status: 403 });
    }
  }

  const key = generateKey(session.user.id, fileName, "r2");
  return NextResponse.json({
    code: 0,
    msg: "ok",
    data: { key, uploadUrl: await getPresignedUrl(key), fileUrl: getFileUrl(key) },
  });
}
```

The key layout deserves a paragraph. Ours is:

```
r2/{userId}/{yyyy-MM-dd}/{timestamp}-{8 random chars}.{ext}
```

- `userId` makes per-user cleanup and debugging trivial;
- the date shards naturally, so hundreds of thousands of objects never pile up under one prefix;
- timestamp plus random suffix avoids collisions and, more importantly, makes keys **unguessable** — reuse the original filename and anyone can walk your bucket from a single link;
- the leading `r2/` segment exists because our upload table is shared with a second, Tencent COS-backed image host: the prefix tells us which backend an object lives in without adding a database column.

### Browser: uploading with a progress bar

`fetch` still cannot report upload progress, so a progress bar means `XMLHttpRequest`:

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
      : reject(new Error(`Upload failed: HTTP ${xhr.status}`));
  xhr.onerror = () => reject(new Error("Network error"));
  xhr.open("PUT", uploadUrl);
  xhr.setRequestHeader("Content-Type", file.type);
  xhr.send(file);
});

// record the upload only after the PUT succeeds
await createUploadRecord({ fileName: file.name, fileKey: key, fileUrl, fileSize: file.size, fileType: file.type });
```

Order matters: **upload first, write the database row second**. Do it the other way around and every cancelled or failed upload leaves a row pointing at a 404.

## Step 7: preview and three copy formats

Most of the perceived quality of an image host lives in the moment right after the upload finishes. Offer at least three formats:

```ts
function getCopyLinks(fileName: string, fileUrl: string, fileType: string) {
  const isImage = fileType.startsWith("image/");
  return [
    { label: "Link", value: fileUrl },
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

Details that matter:

- **Markdown must branch on type.** Images get `![]()`, everything else gets `[]()`, or dropping a zip produces image syntax that renders as nothing.
- **`navigator.clipboard` needs a fallback.** It only exists on HTTPS and localhost; over a LAN IP it throws, so fall back to `document.execCommand("copy")`.
- **Confirm the copy visually** — swap the icon to a checkmark for two seconds. Without feedback people click three more times.
- **Show thumbnails with click-to-zoom** rather than a column of filenames.

## Step 8: deletion, quotas, and abuse

**Delete in both places.** The database row and the R2 object. Drop only the row and the object sits in the bucket accruing charges forever. Deletion can also go through a presigned URL, so there is no second signing path to maintain:

```ts
export async function deleteObject(key: string): Promise<void> {
  const url = buildPresignedUrl({ method: "DELETE", key, config: getR2Config(), expires: 60 });
  const res = await fetch(url, { method: "DELETE" });
  // 204 on success; treat "already gone" (404) as success too
  if (!res.ok && res.status !== 404) {
    throw new Error(`R2 delete failed: ${res.status}`);
  }
}
```

A bug worth stealing the lesson from: if your site ever runs two object stores, the delete path **must dispatch on the object key**. Ours unconditionally called the Tencent COS delete API, with the exception swallowed by a `try/catch` that only logged — so the UI said "deleted," the user saw nothing wrong, and not one file actually left the bucket. Silent failures like that are far harder to find than loud ones.

**Quotas and abuse controls**, minimum viable set on the server:

- **Require sign-in.** An anonymous image host eventually becomes somebody else's free CDN.
- **Cap uploads per day** per user, with an admin exemption.
- **Enforce the type allowlist and size cap in the signing endpoint**, not after the bytes have moved.
- **Treat client-side validation as UX only.** It is a fast error message, never a security boundary.

If you want more, put Cloudflare WAF rules or Rate Limiting on the custom domain to throttle hot links.

## What it costs, and when not to use R2

Take a mid-sized blog image host: 5,000 images averaging 300KB (1.5GB total), 500 new uploads a month.

- Storage: 1.5GB × $0.015 = **$0.023/mo** — inside the 10GB free tier, so $0 in practice
- Class A operations (uploads, deletes): 500, against 1M free per month → **$0**
- Class B operations (reads): say 500,000, against 10M free per month → **$0**
- Egress: any amount → **$0**

At that scale it is effectively free. The same traffic on S3 runs tens of dollars a month in egress alone.

When R2 is the wrong answer:

- **You need image transformation** — thumbnails, cropping, WebP conversion, watermarks. R2 stores, it does not transform; do it before upload or add Cloudflare Images / Workers.
- **Your audience is mostly in mainland China.** Cloudflare's free-tier routing there is inconsistent; a domestic CDN will serve those users better.
- **You have strict data-residency requirements.** R2 location hints are region-level, not country- or datacenter-precise.
- **You depend on advanced S3 features.** Versioning, lifecycle rules, and event notifications are not at S3 parity — verify before you commit.

## Wrapping up

Having built the whole path, only a handful of things actually block you: public access needs a custom domain, CORS has to be configured up front, do not sign `Content-Type`, the region is `auto`, and deletion has to hit both the object and the row. Everything else is ordinary auth-and-quota work.

If you would rather not build it, use the one we shipped: [Cloudflare R2 Image Hosting](/tools/r2-upload). Sign in, drop a file, and copy the result as a URL, Markdown, or HTML — the file goes from your browser straight to R2 with no server in the middle.
