# Kimi K3's Weights Landed Right on Deadline. The Same Day, Anthropic's CEO Published a Denial

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/kimi-k3-weights-land-dario-denial-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/kimi-k3-weights-land-dario-denial-en?utm_source=github&utm_medium=referral)**

First, let's close out the cliffhanger.

Two days ago, in [Three Clocks](/articles/kimi-k3-weights-three-clocks-en), I logged the state of play: as of 01:08 UTC on July 27, the K3 repo under `moonshotai` on Hugging Face still did not exist — but Moonshot's own wording had always been that weights would ship *by* July 27, which gave them another 23 hours before anyone could fairly say "late."

They made it. **Right on the wire.**

On July 27, `moonshotai/Kimi-K3` went live, and the Hacker News post climbed to **1,314 points and 516 comments** — the top story of the weekend. People in the thread were literally sharing a countdown page, waiting for the gates to open like it was New Year's Eve.

And then, the same day, the other thing happened: **Dario Amodei published a personally signed post on Anthropic's site** titled *Our position on open-weights models*. The first section gets straight to the point:

> "Anthropic has never advocated for a ban on open-weights models."

The CEO of the industry's most safety-forward frontier lab, publishing a denial letter on the very day of the largest open-weights release in history — those two events, side by side, are the whole plot of this weekend in AI.

This piece does two jobs: verify what actually shipped and how to read K3's benchmark table, then faithfully report what Dario actually said — and why 665 HN comments largely refused to take yes for an answer.

## 1. What you actually get in the download

Per the model card and technical report, this is more than a weights file:

- **Scale**: a 2.8-trillion-total-parameter MoE with 104B activated per token; 896 experts with 16 selected (plus 2 shared). Moonshot calls it "the world's first open 3T-class model."
- **Architecture**: 93 layers — 69 running Kimi Delta Attention (KDA) and 24 running gated MLA — plus Attention Residuals; natively multimodal via a 401M-parameter MoonViT-V2 vision encoder; a 1-million-token context window.
- **Precision**: MXFP4 weights / MXFP8 activations from quantization-aware training — native, not a post-hoc squeeze. The full download is roughly 1.5TB.
- **Infrastructure shipped alongside**: MoonEP (expert parallelism), FlashKDA (KDA kernels), and AgentEnv. *Three Clocks* noted that KDA is incompatible with ordinary prefix caching, and that shipping weights without the serving stack would mean shipping a file most people can't run. They shipped the stack too.

**The license has one clause that is not MIT**, and hosts should read it in the original: if you operate a model-as-a-service business on K3 and your aggregate revenue exceeds **$20 million** over any consecutive 12 months, you need a separate agreement with Moonshot; products over 100M MAU or $20M revenue must display "Kimi" attribution. Irrelevant to almost every researcher and small team. Not irrelevant to inference providers.

...

---

**[👉 Continue reading: Kimi K3's Weights Landed Right on Deadline. The Same Day, Anthropic's CEO Published a Denial](https://tools.cooconsbit.com/en/articles/kimi-k3-weights-land-dario-denial-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
