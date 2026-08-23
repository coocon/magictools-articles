# Three Clocks: Why Kimi K3's Weights Still Are Not Out

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/kimi-k3-weights-three-clocks-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/kimi-k3-weights-three-clocks-en?utm_source=github&utm_medium=referral)**

Facts first, interpretation second.

Over the past seven-plus hours I queried the Hugging Face API four times. The record, all times UTC:

| Time | Newest repo under `author=moonshotai` | Official K3 repo |
|---|---|---|
| 07-26 17:45 | `Kimi-K2.7-Code` (2026-06-15) | Does not exist |
| 07-26 19:37 | Same | Does not exist |
| 07-26 21:37 | Same | Does not exist |
| **07-27 01:08** | Same | **Still does not exist** |

The release moment reported almost everywhere was **00:00 UTC on July 27, 2026**. As I write this, that moment passed 68 minutes ago.

## First, calibrate the word "late"

Before writing "Moonshot missed its deadline," it is worth checking the wording — because this is precisely the class of error I criticized in an earlier piece: **a number lifted out of its context and treated as a conclusion.**

Moonshot's own technical blog says the full weights will be released **by July 27, 2026**.

In secondary coverage that became "shipping at 00:00 UTC on July 27."

Those are not the same sentence. By Moonshot's own wording, the deadline is the **end of July 27** — 23:59 UTC. So at the moment I write this they have nearly 23 hours left and **have not missed anything**.

This is therefore not a gotcha piece. What is actually worth writing is something else: **inside this waiting window you can see more than the release itself would show you.**

Because the timing of this release has stopped being an engineering question.

## Clock one: Moonshot's own engineering schedule

First, rule out "something happened to the company."

I checked the MoonshotAI organization on GitHub: the `kimi-code` repository was last pushed at **2026-07-26 16:38 UTC** — an hour before my first query — at 5,192 stars. The org is not dark. People are working.

The delivery difficulty is also real. This is roughly a **1.4TB** weight release (2.8 trillion parameters stored natively at MXFP4 4-bit), alongside a promised technical report. And the KDA prefill-cache implementation Moonshot contributed to vLLM is expected to land with the weights — KDA's hybrid attention is incompatible with conventional prefix caching, so shipping weights without the matching inference path means shipping a file most people cannot run.

All legitimate reasons for delay. None of them explain the next two clocks.

## Clock two: Beijing — draft export controls on weights

On July 21, the *Financial Times* reported that China's Ministry of Commerce has been consulting **Alibaba, ByteDance, Zhipu** and other domestic AI and chip companies on possible controls covering **model weights, key training data, and chip designs**. Reuters followed the same day but said it could not immediately verify the report; MOFCOM and the named companies did not respond to requests for comment.

...

---

**[👉 Continue reading: Three Clocks: Why Kimi K3's Weights Still Are Not Out](https://tools.cooconsbit.com/en/articles/kimi-k3-weights-three-clocks-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
