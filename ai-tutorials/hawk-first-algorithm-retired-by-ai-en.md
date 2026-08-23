# The AI Found the Weakness in 60 Hours. Humans Took a Month to Verify It. HAWK Is the First NIST Candidate Retired by an AI.

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/hawk-first-algorithm-retired-by-ai-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/hawk-first-algorithm-retired-by-ai-en?utm_source=github&utm_medium=referral)**

The full arc of this story spans 48 hours.

On July 28, Anthropic [published research](https://www.anthropic.com/research/discovering-cryptographic-weaknesses) showing that its unreleased Claude Mythos Preview model, working semi-autonomously for roughly 60 hours and consuming about $100,000 in compute, had compressed the best known analysis of the NIST post-quantum signature candidate HAWK from 2^64 down to 2^38 — cutting its effective security strength in half, and making HAWK-256 keys recoverable on a single server in hours.

On July 29, HAWK's team confirmed the attack on the [official NIST mailing list](https://groups.google.com/a/list.nist.gov/g/pqc-forum/c/2r2u6SbHun4) and **formally withdrew the algorithm from NIST standardization**.

A candidate that had survived two rounds and two years of expert review went from "AI finds weakness" to "voluntary exit" in under two days. A first for the history of cryptographic standardization: an algorithm eliminated with an AI as the lead author of its demise.

The story worth writing is not "AI breaks encryption" — independent cryptographers' first reaction was, in fact, to tell everyone to calm down. The story worth writing hides in another pair of numbers on the timeline: **the AI found the weakness in 60 hours; Anthropic's human researchers needed nearly a month to verify it.** That inversion is the genuinely new thing here.

## First, be precise about what got knocked out

HAWK needs context. NIST's mainline post-quantum standards are already finalized — ML-KEM (FIPS 203), ML-DSA (FIPS 204), SLH-DSA (FIPS 205); those are what your TLS connections are gradually migrating to, and **none of them are affected by this**. HAWK was competing in NIST's *additional* signature process, which entered its third round in May 2026 with nine candidates — of which HAWK was the **only lattice-based scheme**. Hold onto that detail; we'll need it.

The attack's mathematical shape is summarized precisely by the NIST forum thread's title: "HAWK-n Key Recovery Reduces to SVP in Dimension n/2 + 1." In plain language: Mythos found a path that halves the scale of the lattice-reduction computation needed to recover a key. Johns Hopkins cryptographer Matthew Green, [in his post-mortem](https://blog.cryptographyengineering.com/2026/07/29/some-notes-about-anthropics-new-results/), highlighted the detail most coverage skips: **Mythos invented no new mathematics.** It combined known cryptanalytic tools in a way nobody had tried, aimed at HAWK's mathematical foundation, the Lattice Isomorphism Problem.

...

---

**[👉 Continue reading: The AI Found the Weakness in 60 Hours. Humans Took a Month to Verify It. HAWK Is the First NIST Candidate Retired by an AI.](https://tools.cooconsbit.com/en/articles/hawk-first-algorithm-retired-by-ai-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
