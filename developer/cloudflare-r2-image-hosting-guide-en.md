# Build Your Own Image Host on Cloudflare R2: Buckets, Presigned Uploads, and the Gotchas

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/cloudflare-r2-image-hosting-guide-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/cloudflare-r2-image-hosting-guide-en?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 Continue reading: Build Your Own Image Host on Cloudflare R2: Buckets, Presigned Uploads, and the Gotchas](https://tools.cooconsbit.com/en/articles/cloudflare-r2-image-hosting-guide-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
