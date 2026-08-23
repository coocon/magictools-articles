# On July 27, AI Security Split Into Two Camps. Microsoft Bet on Both.

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/ai-security-two-camps-microsoft-both-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/ai-security-two-camps-microsoft-both-en?utm_source=github&utm_medium=referral)**

On July 27, 2026, the same question received two directly opposed answers.

The question: **who should hold security capability in the AI era?**

The first answer came from San Francisco. Microsoft unveiled [MAI-Cyber-1-Flash](https://microsoft.ai/news/introducing-mai-cyber-1-flash-inside-mdash/) — its first security-specialized, internally developed model: closed, private preview, available only to vetted customers. Microsoft AI chief Mustafa Suleyman's framing was unapologetic: the data, the harness, the domain expertise — that's a moat, and "this is just the tip of the iceberg."

The second answer came from Nvidia's official blog. The [Open Secure AI Alliance](https://blogs.nvidia.com/blog/open-secure-ai-alliance/) launched with 37 founding members (reports vary from 37 to 52 depending on the outlet; Nvidia's official list is the reference), including IBM, CrowdStrike, Palo Alto Networks, Cloudflare, Hugging Face, and the Linux Foundation. The alliance's core claim fits in one sentence: **defenders need security AI they can inspect, modify, and run locally** — with the unstated corollary that closed APIs cannot deliver that.

The interesting part is in the member list: **Microsoft is on both sides.** It is a founding member of the alliance, and on the same day it shipped a closed model that contradicts the alliance's central thesis.

That is not an editorial coincidence. It is the industry laying all its cards on the table in a single day. Let's take each camp's logic apart, then ask whether Microsoft's double bet is schizophrenia or something more deliberate.

## Microsoft's answer: security capability is a moat

Start with what Microsoft actually shipped, because the secondhand coverage loses detail fast.

MAI-Cyber-1-Flash is built on Microsoft's in-house MAI-Thinking-1 reasoning model and designed to work inside **MDASH** — the multi-agent vulnerability identification and remediation harness Microsoft introduced in May, made up of 100+ agents tuned by security experts. The division of labor is explicit: MAI-Cyber-1-Flash handles up to 90% of MDASH's tasks, with GPT-5.4 reserved for the hardest 10%. Microsoft claims the combination runs at **50% lower cost** than its previous configuration of GPT-5.4, GPT-5.4 mini, and GPT-5.3 Codex.

The report card: **95.95%** on [CyberGym](https://thehackernews.com/2026/07/microsoft-says-new-cybersecurity-ai.html) — 1,507 real-world vulnerability-reproduction tasks drawn from 188 open-source projects in Google's OSS-Fuzz program — roughly 12 percentage points ahead of Anthropic's Mythos system, and well above GPT-5.5-Cyber's ~83%.

...

---

**[👉 Continue reading: On July 27, AI Security Split Into Two Camps. Microsoft Bet on Both.](https://tools.cooconsbit.com/en/articles/ai-security-two-camps-microsoft-both-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
