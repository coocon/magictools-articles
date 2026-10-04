# Claude Code 429 \"Request rejected (429)\": How Long It Retries, and Why retry-after Over 60 Seconds Fails Instantly (Tested)

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-429-rate-limit-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-429-rate-limit-en?utm_source=github&utm_medium=referral)**

> **Short answers first**
>
> - **What `API Error: Request rejected (429)` means**: the backend (Anthropic, or your gateway / relay) says you are sending too much. The part after `·` is the backend's own message — that tells you who is rate-limiting you.
> - **Should you retry by hand?** Usually no. Claude Code has already retried up to 10 times over about 3 minutes; seeing the error means you were limited the whole time.
> - **It failed instantly without waiting**: the backend sent a `retry-after` above 60 seconds, and Claude Code gave up (the threshold measured exactly between 60 and 61 seconds). Wait that long and send again, or ask your gateway to raise the limit.
> - **Interactive mode keeps showing `Retrying in Ns · attempt N/10`**: normal backoff, not a hang. Press `Esc` to interrupt.
> - **Monthly spend-cap 429s**: per the official docs these carry no `retry-after` and keep failing until access resumes. Retrying will not help; check your usage in the Console.

## Background

On long tasks, or behind a third-party gateway, Claude Code sometimes prints:

```
API Error: Request rejected (429) · Number of request tokens has exceeded your per-minute rate limit
```

People searching for this want to know who is limiting them, whether Claude Code retries on its own, and how long to wait. The docs say the SDK retries with exponential backoff and honors `retry-after`, but not how many times Claude Code retries, how long it waits, or what happens when `retry-after` is large.

Our [previous article](/en/articles/claude-code-invalid-api-key-en) measured the basic 429 retry curve but left gaps: recovery after a 429, the interactive display, large `retry-after` values, and what gateway 429s look like. This one fills them in.

## Where 429s come from

All from primary sources:

| Source | What the 429 looks like | Reference |
|---|---|---|
| Anthropic API | `{"type":"error","error":{"type":"rate_limit_error","message":"…"}}`, usually with `retry-after` | [API errors](https://docs.claude.com/en/api/errors) |
| Anthropic monthly spend cap | Also `rate_limit_error`, but **no `retry-after`**, keeps failing until access resumes | Same page: *A tier spend-cap 429 has no `retry-after` header and keeps failing until access resumes* |
| new-api per-model limit | OpenAI-style: `{"error":{"message":"您已达到请求数限制：N分钟内最多请求M次 (request id: …)","type":"new_api_error","code":""}}` | new-api source `middleware/model-rate-limit.go`, `middleware/utils.go` (commit `1a4166d8e8`) |
| new-api global limit | **Empty body**, only a `Retry-After` header set to the full window in seconds | `writeRateLimited` in `middleware/rate-limit.go` |
| nginx `limit_req` | HTML error page `<title>429 Too Many Requests</title>` | nginx default error page |
| DeepSeek | Docs list `429 - Rate Limit Reached` without a body format | [DeepSeek error codes](https://api-docs.deepseek.com/quick_start/error_codes) |

...

---

**[👉 Continue reading: Claude Code 429 \"Request rejected (429)\": How Long It Retries, and Why retry-after Over 60 Seconds Fails Instantly (Tested)](https://tools.cooconsbit.com/en/articles/claude-code-429-rate-limit-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
