# Four Numbers Everyone Is Misreading About Kimi K3

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/kimi-k3-open-weights-four-numbers-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/kimi-k3-open-weights-four-numbers-en?utm_source=github&utm_medium=referral)**

Moonshot AI has committed to releasing the full weights of Kimi K3 at 00:00 UTC on July 27, 2026. As I write this, that moment is a few hours away.

In those few hours, four numbers about K3 have already made a full lap around the internet: **2.8 trillion parameters**, a **51% hallucination rate**, **six leaderboard wins**, and **$3.3 trillion wiped from chip stocks**.

All four numbers are real. The most popular reading of all four is wrong.

They are also wrong in the same way each time — a number that describes a **model** gets treated as a conclusion about a **system**. Let us take the denominators apart one at a time.

## Number one: 2.8 trillion parameters — "It is open, so I can finally run it locally"

This is the most-shared claim and the one reality will correct fastest.

First, what K3 actually is. A sparse mixture of experts with 2.8 trillion total parameters, activating roughly 16 of 896 experts per token, so per-token compute is far below a dense model of the same size. A 1-million-token context window. Native visual understanding. Two architectural additions: KDA (Kimi Delta Attention, a hybrid of linear-cost and standard quadratic attention) and Attention Residuals, reportedly delivering up to 6.3× faster decoding at million-token lengths. Training used quantization-aware training with MXFP4 weights and MXFP8 activations, so the released weights are **natively 4-bit**, not post-hoc compressed.

The problem is the phrase "run it locally."

Per public information, loading the weights alone requires **more than 1.5 TB of HBM**, and Moonshot's own serving guidance calls for a supernode cluster of **at least 64 accelerators**. One RTX 4090 cannot do it. One Mac Studio cannot do it. Eight H100s is the **floor**, not the comfortable configuration.

There is also an arithmetic inconsistency worth flagging: multiple outlets report a download size of roughly **594 GB**, but 2.8 trillion parameters at 4 bits should be about **1.4 TB**. Those two numbers do not reconcile. Possible explanations include shard compression, some layers at lower precision, or a reporting error somewhere. **After release, size your storage from the actual files in the repository**, not from a secondhand figure.

The real conclusion is not "K3 is too big." It is this: **with K3, "open weights" and "usable by an individual developer" have been fully decoupled for the first time.**

The old chain of meaning for open models was intact end to end. Weights are published, I download them, I run them on my own hardware, I no longer depend on any vendor. K3 breaks the middle two links. The weights are genuinely public, but the parties who can consume that publication are institutions with multi-node GPU clusters: cloud providers, platform teams at large companies, well-funded research labs. What an individual developer ends up with is still an API endpoint.

...

---

**[👉 Continue reading: Four Numbers Everyone Is Misreading About Kimi K3](https://tools.cooconsbit.com/en/articles/kimi-k3-open-weights-four-numbers-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
