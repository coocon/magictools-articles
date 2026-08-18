---
title: "The Day GitHub Went Down for 7.5 Hours, Cursor Launched Its Own Code Host"
slug: github-outage-cursor-origin-en
summary: "On August 17, GitHub suffered a 7-hour-35-minute critical incident — its 9th critical in two months. The same day, Cursor launched Origin, a git forge 'designed for agent scale,' three days after SpaceX closed its $60B acquisition. Is the moat cracking? We fact-checked both stories."
category: developer
tags: [GitHub, Cursor, code hosting, Git, outage, developer tools]
coverImage: ""
status: published
locale: en
source: authored
translationSlug: github-outage-cursor-origin
---

# The Day GitHub Went Down for 7.5 Hours, Cursor Launched Its Own Code Host

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

**What it isn't**: there's no issue tracker, no wiki, no standalone pricing, and no free tier — the docs state plainly, "It is not available on free plans." The "$20/month" figure floating around is Cursor Pro's subscription price attached to Origin by mistake (Origin comes bundled with paid plans; it isn't sold separately). The "3 private repos on the free tier" claim is pure fabrication.

**Two pieces of context**: Cursor acquired the code review company Graphite in December 2025, and Graphite founder Tomas Reimers is one of Origin's developers — he showed up in the HN launch thread to claim it. The bigger backdrop: three days before the launch (August 14), SpaceX completed its **$60 billion** all-stock acquisition of Cursor's parent Anysphere, confirmed in an SEC 8-K.

And one detail no screenwriter would dare pitch: **Origin was knocked over by the GitHub outage on its own launch day** — its GitHub sync depends on GitHub, and Cursor posted its own incident. The challenger got taken down, on debut day, by the incumbent it's challenging.

## What HN Is Fighting About: Blame, Migration, and Why Migration Is Hard

**The blame war.** One camp says LLM-era traffic is crushing traditional forges — "usage is up tens of x, agents are pushing slop" (130 replies under that comment). The rebuttal: "LLM code isn't the problem; Microsoft mismanagement is." Someone asked the practical question: why not solve this with pricing and rate limits?

**Migration testimonies.** The loudest destination is Forgejo/Codeberg: "We ditched GitHub for Forgejo and couldn't be happier — the migration took a few hours and we treated it as a hackathon." "Moved everything to Codeberg months ago and set up an annual donation; what broke me was GitHub force-feeding Copilot." One commenter offered a decision framework: want the GitHub experience → Forgejo/Gitea; just want managed hosting → GitLab/Codeberg; have your own infra → gitolite+CGit. The founder of federated newcomer tangled.org was in the thread recruiting.

**Why people stay** may be the most sober part of the discussion. A user who has self-hosted GitLab for six years testified it's "not always smooth sailing" — upgrade rollbacks and maintenance costs are real. The deeper counterargument: GitHub's value is the **unity of the entire open-source world** — cross-project search, shared Actions, a single contribution graph; forge balkanization destroys the public good of "everyone in one place." Organizations deeply bound to the Actions ecosystem effectively have no options — "the real problem is replacing Actions."

## Verdict: The Beginning of Cracks, Not a Cracked Moat

Weighing the evidence:

**For the "moat is cracking" case**: 50 incidents and 9 criticals in two months is hard data from the official API, not vibes; the status-page whitewashing is documented trust erosion; community discussion shifted for the first time from "complain and carry on" to serious evaluation with real completed migrations; and Cursor's timing was surgical — "agent scale" targets exactly the pain point HN spent all day arguing about, with SpaceX's resources behind it.

**The counter-evidence is just as hard**: Origin today is an early beta behind a paywall, with no issues and no free tier — it cannot substitute for GitHub's open-source ecosystem. One HN commenter put it in a sentence: "this is a beta for paid plans, not a GitHub alternative." Right now it's a **parasitic layer** on GitHub (it lives off GitHub sync), not a replacement. And "editor + hosting + models" consolidating under one Musk-affiliated company is already a stated dealbreaker for some developers.

So the accurate framing: GitHub's reliability crisis and a credibly funded, AI-native challenger showed up on the same day for the first time. But the challenger is nowhere near ready to absorb an exodus — and GitHub's deepest moat was never uptime; it's the network effect of the ecosystem.

For the individual developer, there's exactly one actionable takeaway, and it's the cheapest insurance this outage offers: spend ten minutes adding a second remote to your critical repos (GitLab, Codeberg, or self-hosted — any will do); it's one `git remote set-url --add --push` command. You can watch the platform war from the sidelines. An afternoon of failed pushes is something you don't want twice.

---

*Sources:*
*GitHub status page: [Incident zkxwbgr0cnmx](https://www.githubstatus.com/incidents/zkxwbgr0cnmx) (frequency stats from the [githubstatus API](https://www.githubstatus.com/api/v2/incidents.json))*
*Cursor official: [Origin changelog](https://cursor.com/changelog/origin-code-hosting) / [Origin docs](https://cursor.com/docs/origin)*
*Hacker News: [outage main thread (529 pts)](https://news.ycombinator.com/item?id=49330597) / [Ask HN: Alternatives to GitHub (491 pts)](https://news.ycombinator.com/item?id=49331033) / [Origin launch thread](https://news.ycombinator.com/item?id=49334209)*
*SpaceX–Cursor acquisition: [Seeking Alpha (SEC 8-K)](https://seekingalpha.com/news/4633335-spacex-completes-60b-acquisition-of-cursor-as-musk-led-firm-tries-to-gain-edge-in-ai-coding)*
*Opinion piece: [GitHub has an availability problem](https://dhruv2038.bearblog.dev/github-has-an-availability-problem-is-it-time-to-look-elsewhere/)*
