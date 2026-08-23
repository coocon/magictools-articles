# The Day GitHub Went Down for 7.5 Hours, Cursor Launched Its Own Code Host

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/github-outage-cursor-origin-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/github-outage-cursor-origin-en?utm_source=github&utm_medium=referral)**

Two things happened on August 17, 2026. Neither is earth-shattering on its own; on the same day, they make a narrative.

At 13:40 UTC, GitHub's status page opened incident zkxwbgr0cnmx, severity critical. That same afternoon, Cursor launched **Origin**, its own code hosting platform, with the tagline "designed for agent scale."

A platform's reliability crisis and a challenger armed with capital and an AI story, in the same news cycle. Here are the facts on both — including several details that got mangled as the story spread.

---

## The Outage: 7 Hours 35 Minutes, the 9th Critical in Two Months

From the official githubstatus API, the August 17 timeline (all UTC):

- **13:40** incident opened; at 13:45 GitHub acknowledged ~20% error rates affecting PRs, Issues, and other core features
- **14:04–16:16** peak: ~20% error rates on Web/API, ~50% on archive downloads and raw content, SAML/OIDC/SCIM affected; Actions, Webhooks, Pages, Git operations, and Copilot successively marked degraded
- **16:59** first mitigation, followed by relapses — Git operations, Issues, and the API degraded again, then intermittent Copilot authentication failures
- **21:15** resolved. Total duration: **7 hours 35 minutes**. No root cause published yet; GitHub has only promised that "a detailed root cause analysis will be shared as soon as it is available"

One detail got roasted on Hacker News: users measured the outage starting around 13:35 — five minutes before the incident was opened — while the status page was still all green. The more structural criticism: site-wide failures get labeled "degraded performance," and degraded time doesn't count against uptime, which is how Issues shows 100% historical availability.

A single incident isn't news; frequency is. We pulled the most recent 50 incidents from the status page API — **they only span about two months**, June 16 to August 17: 9 in June, 26 in July, 15 in the first 17 days of August. By severity: 9 critical, 11 major, 28 minor — with 6 criticals in July alone.

That's why the mood on HN was different this time. The main thread hit 529 points and 888 comments, with one user writing "today is the tipping point… The hope is dead." A separate "Ask HN: Alternatives to GitHub" thread reached 491 points — the complaint thread became a serious procurement thread.

## Origin: What It Is, and What It Isn't

Cursor's Origin is real, but much of what circulated about it is wrong. Per the official changelog and docs:

**What it is**: Cursor's own git forge, in early beta, rolling out to all **paid** plans starting August 17. Current features: repository hosting (standard git clone/push/pull), opening/reviewing/merging PRs, web-based code browsing and search, two-way GitHub mirror sync (GitHub remains the source of truth, with PR comments syncing both ways in real time), an Origin CLI, and third-party apps (Vercel preview deploys; Depot/Buildkite CI — the latter can run your existing GitHub Actions workflows as-is). The AI-native part: every repo has a built-in agent you can ask about the code, have edit code, update PRs, and push branches. More "agent-native features ship soon."

...

---

**[👉 Continue reading: The Day GitHub Went Down for 7.5 Hours, Cursor Launched Its Own Code Host](https://tools.cooconsbit.com/en/articles/github-outage-cursor-origin-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
