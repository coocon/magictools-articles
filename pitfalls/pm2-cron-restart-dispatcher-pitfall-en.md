# PM2 cron_restart Kills Before It Starts: How My Scheduler Silently Died for Two Days

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/pm2-cron-restart-dispatcher-pitfall-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/pm2-cron-restart-dispatcher-pitfall-en?utm_source=github&utm_medium=referral)**

## The Symptom

A content pipeline that publishes every morning (generate → WeChat post → email) went completely silent on 08-28 and 08-29 — two days in a row, with identical log signatures:

- The generate step's output stopped at **07:04:xx**. No error, no stack trace. Just nothing after that.
- At **07:05** sharp, two dispatcher startup banner lines appeared in the log.
- For the rest of the day, every dispatcher wake-up printed the same line: "record exists for today (running), skipping."
- At 07:40, the downstream WeChat draft step failed with `guard:no-today-queue` — there was no article to publish.

No crash logs. No OOM. The dispatcher showed up healthy in the process list. The task had simply evaporated.

## The Root Cause

The pipeline's scheduler, pipeline-dispatcher, was at the time a **one-shot PM2 process**: run one round of checks, execute whatever steps are due, exit. PM2's `cron_restart: '*/5 * * * *'` woke it up every five minutes.

Here's the trap: **when the cron fires and the instance is still running, `cron_restart` doesn't wait for it, and doesn't skip the round. It kills first, then starts** — SIGINT to the running process, its spawned children included, then a fresh instance.

This was invisible in normal operation because a generate round took 42–92 seconds, always finishing well before the next 5-minute boundary. On those two mornings, the candidate pool's 24-hour cap had rolled old entries out of the window around 07:00, quota suddenly freed up, and the inline curation step ballooned (one LLM clustering call took 1m43s, plus fact-checking six briefs). Generate got dragged past 07:05 — and then:

1. At 07:05, PM2 killed the dispatcher along with the generate child that was mid-write.
2. The execution record in the database (PipelineRun) was left stuck in `running`. Nobody cleaned it up.
3. The idempotency key (step + date) saw "a record already exists for today" and **refused to retry for the rest of the day**.
4. Every downstream step waited for upstream output that would never come. The whole chain failed silently.

One sentence: **putting cron_restart on a "wake every N minutes" scheduler gives every one of its subtasks a hard N-minute timeout** — a timeout that raises no error, fires no alert, and leaves behind a zombie `running` record that blocks the retry path too.

...

---

**[👉 Continue reading: PM2 cron_restart Kills Before It Starts: How My Scheduler Silently Died for Two Days](https://tools.cooconsbit.com/en/articles/pm2-cron-restart-dispatcher-pitfall-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
