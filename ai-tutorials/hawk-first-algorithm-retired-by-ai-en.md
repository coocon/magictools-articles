---
title: "The AI Found the Weakness in 60 Hours. Humans Took a Month to Verify It. HAWK Is the First NIST Candidate Retired by an AI."
slug: hawk-first-algorithm-retired-by-ai-en
summary: "Anthropic's unreleased Claude Mythos model spent 60 hours and roughly $100,000 of compute halving the best known analysis of post-quantum candidate HAWK. The next day, HAWK's team withdrew from NIST standardization. Inside the 48 hours: why attacking the 256-bit version killed the whole scheme, the model that refused to try until encouraged for three days, the sober expert assessments — and the real story: discovery took 60 hours, verification took a month."
category: ai-tutorials
tags: [AI, cryptography, post-quantum, NIST, HAWK, Claude Mythos, Anthropic, AES, LLM]
coverImage: ""
status: published
locale: en
source: authored
translationSlug: hawk-first-algorithm-retired-by-ai
---

# The AI Found the Weakness in 60 Hours. Humans Took a Month to Verify It. HAWK Is the First NIST Candidate Retired by an AI.

The full arc of this story spans 48 hours.

On July 28, Anthropic [published research](https://www.anthropic.com/research/discovering-cryptographic-weaknesses) showing that its unreleased Claude Mythos Preview model, working semi-autonomously for roughly 60 hours and consuming about $100,000 in compute, had compressed the best known analysis of the NIST post-quantum signature candidate HAWK from 2^64 down to 2^38 — cutting its effective security strength in half, and making HAWK-256 keys recoverable on a single server in hours.

On July 29, HAWK's team confirmed the attack on the [official NIST mailing list](https://groups.google.com/a/list.nist.gov/g/pqc-forum/c/2r2u6SbHun4) and **formally withdrew the algorithm from NIST standardization**.

A candidate that had survived two rounds and two years of expert review went from "AI finds weakness" to "voluntary exit" in under two days. A first for the history of cryptographic standardization: an algorithm eliminated with an AI as the lead author of its demise.

The story worth writing is not "AI breaks encryption" — independent cryptographers' first reaction was, in fact, to tell everyone to calm down. The story worth writing hides in another pair of numbers on the timeline: **the AI found the weakness in 60 hours; Anthropic's human researchers needed nearly a month to verify it.** That inversion is the genuinely new thing here.

## First, be precise about what got knocked out

HAWK needs context. NIST's mainline post-quantum standards are already finalized — ML-KEM (FIPS 203), ML-DSA (FIPS 204), SLH-DSA (FIPS 205); those are what your TLS connections are gradually migrating to, and **none of them are affected by this**. HAWK was competing in NIST's *additional* signature process, which entered its third round in May 2026 with nine candidates — of which HAWK was the **only lattice-based scheme**. Hold onto that detail; we'll need it.

The attack's mathematical shape is summarized precisely by the NIST forum thread's title: "HAWK-n Key Recovery Reduces to SVP in Dimension n/2 + 1." In plain language: Mythos found a path that halves the scale of the lattice-reduction computation needed to recover a key. Johns Hopkins cryptographer Matthew Green, [in his post-mortem](https://blog.cryptographyengineering.com/2026/07/29/some-notes-about-anthropics-new-results/), highlighted the detail most coverage skips: **Mythos invented no new mathematics.** It combined known cryptanalytic tools in a way nobody had tried, aimed at HAWK's mathematical foundation, the Lattice Isomorphism Problem.

And here is the pivot almost no report unpacks: **Anthropic attacked only HAWK-256, while the NIST submission concerns the 512- and 1024-bit versions.** By the usual playbook, the team could have argued "our parameters are big enough." So why withdraw everything the next day?

Because HAWK's own team ran the numbers: the attack halves the block size required for lattice reduction, and the naive fix — doubling parameters — would erase **all of HAWK's performance advantages** against the other eight candidates. It wasn't killed by a full break. It was killed by economics: the repaired version of itself no longer deserved to exist. That is a more general lesson than "algorithm broken" — most retired cryptographic schemes don't collapse; they reach the point where the cost of patching exceeds the cost of replacement.

## The second front: refusal, encouragement, and a "Möbius Bridge"

The same research contains a second result with better drama.

Mythos accelerated the best known analysis of 7-round reduced AES (full AES-128 runs 10 rounds; attacking reduced-round variants is a standard research target) by a factor of 200 to 800, breaking a record that had stood since 2013. By Anthropic's own account, the result was achieved almost entirely autonomously — but its starting point was anything but autonomous: **the model initially refused to try, on the grounds that the existing result was optimal and "improvement was impossible." Researchers sent encouraging prompts for three days before it engaged**, eventually producing roughly a billion output tokens and naming its self-invented technique the "Möbius Bridge."

Most retellings treat this as a charming anecdote. I think it's one of the most information-dense paragraphs in the research. It reveals how AI research capability currently gets triggered: not by a model's own ambition, but by **humans pushing it past its "professional judgment" into unexplored territory.** The model's "impossible to improve" verdict was precisely the consensus it had absorbed from human literature — and the breakthrough lived outside the consensus. Who decides where to push and for how long? Still humans. Read that as evidence of a capability ceiling or as evidence that control remains with people — both readings are available.

## Independent calibration: don't panic, do stay sober

You can't calibrate this story on Anthropic's account alone. Several independent voices belong side by side:

- **Matthew Green**: overall verdict "very impressive," followed by line-item cooling — HAWK is not a deployed scheme; it is related to Falcon signatures but the attack **does not transfer**; reduced-round AES is a standard research target, and the new result is "better than the state of the art but not wildly so"; this is not an "AI breaks Curve25519" step change.
- **Google's cryptography lead Sophie Schmieg** was blunter: "Basically with this paper, HAWK is dead."
- One line worth recording separately: **almost simultaneously, another group of researchers independently reached a similar attack via a different mathematical path** (Green confirmed this, noting Claude's version is a little better; some reports say the second team used GPT-5.6). Two rival models, two different routes, one target, hit within days of each other — what was crossed here is not one lab's threshold but **the field's**.
- Keyfactor's Ellen Boehm offered the optimistic reading: a candidate getting eliminated *during* standardization is the NIST review process working as designed — the weakness was found **before** deployment.

The optimistic reading holds, but Green attached a shadow to it: mathematical cryptanalysis is severely understaffed, and lattice-based cryptography is, in his view, **understudied**. A weakness that two years of human review missed fell in 60 hours to an AI — flip that sentence around and it reads: **how many other "unfound for two years" weaknesses exist simply because nobody (and no AI) has looked hard?** NIST's remaining eight candidates, and much of the deployed landscape, will likely get "scanned" next. At $100,000 per result, it's dramatically cheaper than hiring a top cryptanalysis team for two years.

## The real news: the bottleneck moved

Now return to the opening numbers. Discovery: 60 hours. Verification: nearly a month, with Anthropic staff spending hundreds of hours confirming the AES result was real. Anthropic wrote the key sentence itself: at this rate of progress, **"human researchers may become the bottleneck."**

Last week we wrote about [how benchmarks stopped measuring](/articles/benchmarks-stopped-measuring-en) — the evaluation ecosystem's crisis of measurement capacity falling behind generation capacity. This event is the same structure replaying in research: **the rate of producing findings has outrun the rate of digesting them.** The default assumption of the scientific pipeline has always been that discovery is hard and verification is comparatively cheap (peer review costs far less than original research). Once AI drives the marginal cost of discovery down to $100,000 a shot, that assumption inverts for the first time. Verification capacity — qualified human expert time that can say "this result is real" — is becoming the scarcest resource on the line.

Anthropic's response is CryptanalysisBench, built with ETH Zurich, Tel Aviv University, and others, an attempt to make "AI finds cryptographic weaknesses" a measurable, standardized activity. Right direction — but it addresses evaluation on the discovery side, not capacity on the verification side. The verification side has no good answer yet, which is why, despite this disclosure following the full responsible-disclosure playbook (NIST mailing list coordination, contact with HAWK's authors), some outlets still cautioned that the external reviewing cryptographers were unnamed and independent reproduction was incomplete. Even the news itself is stuck in the verification bottleneck.

## What this means for developers

Three practical calls:

1. **Don't touch your production systems.** Finalized FIPS 203/204/205 are unaffected; full-round AES is unaffected. What fell was an undeployed candidate and a research target. When you see an "AI breaks encryption" headline, check two things first: was the target a reduced variant, and was the scheme deployed anywhere.

2. **When choosing cryptography for long horizons, weight "years under scrutiny" heavily.** HAWK's lesson is that two years of human review carries less assurance than we assumed. If your data must stay confidential for a decade or more (medical, legal, infrastructure), prefer the most-studied schemes — mainline standards like ML-KEM have absorbed analysis on a different order of magnitude than additional-competition candidates. And a scheme's status as "the only lattice-based candidate" — the lone specimen — is itself a risk signal.

3. **Start imagining AI-assisted auditing of your own cryptographic assets.** Offense and defense draw on the same capability. Spending $100,000 to sweep the protocol stack you depend on sounds extravagant today; ride the compute price curve two years forward and it's a routine security budget item. Teams that inventory their non-mainstream cryptographic dependencies now will be far calmer than teams that wait for the CVE notification.

## Coda

The July 29 withdrawal email will end up in the historical record — the first standard candidate eliminated primarily on an AI's findings. But the more archival item on this timeline is the unglamorous ratio sitting next to it: 60 hours versus a month.

Discovery got cheap. Verification is still full price. For the next several years, science's scarce resource won't be ideas — it will be people qualified to say "this is true."

Last time we covered a measurement crisis, the subject was benchmark scores. This time, it's mathematical proof. Same bottleneck, climbing upstream.
