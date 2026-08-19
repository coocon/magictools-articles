---
title: "Claude Designed Proteins From Scratch — And the Prompt Is Public"
slug: claude-protein-binder-design-en
summary: "Anthropic handed Claude a 30,000-token prompt, a GPU budget, and 15 protein targets, then walked away. It hit 14 of them at more than double the field's typical success rate, verified in two independent wet labs. The prompt and every data point are now on HuggingFace."
category: ai-tutorials
tags: [Claude, Anthropic, protein design, AI for science, open source, Opus, drug discovery]
coverImage: ""
status: published
locale: en
source: authored
translationSlug: claude-protein-binder-design
---

# Claude Designed Proteins From Scratch — And the Prompt Is Public

> Based on Anthropic's 2026 research post, *How Claude is accelerating protein design and analytical chemistry*. All block quotes are verbatim from the original.
> Source: https://www.anthropic.com/research/Claude-accelerates-protein-design

---

AI beating benchmarks in math stopped being interesting a while ago. Math verifies cheaply — run the checker, get an answer in seconds.

The hard frontier is the domains where verification is slow and expensive. You cannot grade a protein design with a script. You have to synthesize it, ship it to a wet lab, and wait weeks.

That's what this result is about. Anthropic gave Claude a protein binder design campaign, let it run unsupervised, sent the output to two outside labs for physical testing, and then published the prompt and every data point.

Ten things worth pulling out.

---

## 1. 22–35% Against a 10–15% Baseline

> "Claude (Mythos Preview and Opus 4.8) designed protein binders against 15 targets, and succeeded against 14 of them. Between 22% and 35% of its individual designs bound successfully, depending on the setup, compared to the 10-15% that is typical in protein design campaigns today."

Hit rate is the currency of this field. Design 100 candidates, count how many actually bind.

The field runs at 10–15%. Claude got 26.7% (Mythos Preview) and 22.6% (Opus 4.8) designing against all targets at once in a 48-hour session. Switch to single-target mode — one 24-hour session per target, closer to how a human protein engineer actually works — and Mythos Preview hits 35.1%.

Final tally: 354 confirmed binders against 14 of 15 targets, out of 1,320 designs.

Note what that isn't. It isn't nudging 10% to 12%. It's a doubling, in a field where every candidate costs real money to synthesize and every validation round costs weeks of lab time.

**My take:** Most AI-for-science announcements report either a benchmark number or "assisted an expert with X." This one reports the metric the industry already argues about, measured in a physical experiment. Picking the right metric is half the credibility.

---

## 2. Nobody Was Steering

> "After giving Claude the prompt, we left the model to execute autonomously. We provided no additional scientific, technical, or operational guidance after we initiated the campaigns."

This is the sentence that matters most.

Human involvement consisted of: approving network access requests, keeping the infrastructure alive, and placing the orders for lab validation. Scientific guidance: none. Technical guidance: none.

Claude picked which epitope on each target to attack. It orchestrated multiple structure design, sequence design, and co-folding models to generate candidates. It ran rounds of *in silico* optimization. It screened for designs that were novel, diverse, expressible, soluble, and likely to bind.

> "Claude conducted all of the work that goes into designing a binder, which can take a human operator weeks."

The budget was published too: 48 hours of wall time and up to 12,500 NVIDIA H100 hours for multi-target mode; 24 hours and up to 2,500 H100 hours per target in single-target mode. No token or subagent limits, fast mode on.

**My take:** The operative word is *orchestrating*. Claude didn't invent a new algorithm — it drove the same open specialist models the field already uses. What it replaced was the computational biologist who spends weeks deciding which model to run, in what order, and when a round is bad enough to redo. That job exists in every technical field, under different names.

---

## 3. The Graders Were Outsiders

> "Our external evaluators, Adaptyv Bio and Twist Bioscience, independently produced and tested Claude's designs in the lab, finding that of the 15 targets we designed against, Claude successfully designed binders against 14 of them."

Adaptyv Bio and Twist Bioscience built and tested the designs. Anthropic didn't touch the wet lab step.

The outcome: high-affinity binders against at least six targets (defined as KD < 10 nM, single-digit nanomolar), and binders matching or exceeding the best previously reported affinity against at least four.

Affinity isn't a vanity metric. It sets the dose — tighter binding means less drug, which means fewer side effects and lower manufacturing cost.

**My take:** I discount self-reported AI benchmarks by default. What makes this different is that the scoring was done by two commercial protein companies measuring how molecules behave in a tube. Third-party physical validation is still rare in AI announcements. It shouldn't be.

---

## 4. RBX1: 40% Versus 3.7%

> "Against RBX1 (a small protein that drives the targeted destruction of specific regulatory proteins), Mythos Preview in single-target mode achieved a 40% hit rate, compared to a 3.7% hit rate among participants."

Adaptyv Bio runs public protein design competitions. The participants are human teams with real tooling.

On RBX1, the entire field of participants managed 3.7%. Claude managed 40%. An order of magnitude.

And its top-ranked design outperformed the winning entry — the winner having been selected from 245 submissions.

**My take:** The 3.7% is the important half of that comparison. It tells you RBX1 is genuinely hard, not a target cherry-picked for being easy. A 10× gap isn't "AI does it slightly better." It's a different solution regime. This is also the argument for competition-style benchmarks: without a human scoreline on the same problem, an AI number has no coordinates.

---

## 5. TNFα: The Weaker Model Won

> "We're not sure why Opus 4.8 was successful on this target and Mythos Preview was not. When we assess our models capabilities, we do so holistically."

TNFα is a heavyweight. Blocking it is the mechanism behind Humira and some of the most commercially successful drugs ever made. It's hard to design against because it's multimeric — the binding site sits in a groove formed between two proteins.

The counterintuitive part: Mythos Preview, the stronger model overall, failed. Opus 4.8, the weaker one, succeeded — and produced cross-reactive binders that hit human, cynomolgus monkey, and mouse TNFα. Cross-species reactivity is what makes animal studies possible downstream.

Anthropic's explanation, in full: they don't know.

**My take:** This generalizes far beyond protein design. "Which model is better" is a scalar on a leaderboard and a distribution in reality. When the task space is complex enough, a weaker model winning some sub-region is the expected outcome, not an anomaly. Practical version: run both on *your* task. It costs less than you think, and the leaderboard can't answer the question for you.

---

## 6. β-Sheets Are Reasoning, Not Retrieval

> "Claude designed 15 confirmed binders across six targets that contain at least 20% β-strand, demonstrating its ability to reason about protein structure."

Nearly every computationally designed binder is a bundle of α-helices. There's a reason: helices are easy to design, fold reliably, and forgive mistakes.

β-sheets require extended strands of amino acids to line up side by side. They're harder to design and far more prone to misfolding and aggregation — protein molecules clumping together instead of folding properly on their own.

Claude produced 15 confirmed β-sheet binders across six targets, built on 10 distinct backbones.

**My take:** This is the most informative result in the post. A model that's pattern-matching its training distribution outputs α-helix bundles all day, because that's what the literature is full of. Reliably producing the harder, rarer, more failure-prone fold is evidence of reasoning under physical constraints rather than sampling by frequency. When you want to know whether a model actually understands a domain, watch whether it will take the unpopular path.

---

## 7. A 30,000-Token Prompt, Released in Full

> "In the meantime, we are sharing the prompts we used for these campaigns, as well as all _in vitro_ and _in silico_ data we generated."

Three things went public on HuggingFace (`Anthropic/claude-protein-binder-design`): the protein design prompt — roughly 30,000 tokens — the computational models of the designed complexes, and all *in vitro* and *in silico* data.

Thirty thousand tokens is a short book. What's in it is a field's methodology, evaluation criteria, known failure modes, and the checks that need running.

**My take:** This is the part I'd defend hardest. Hit-rate definitions are arguable; a published prompt and raw dataset are not. Any team with GPUs and a lab budget can now reproduce this or knock it down. That's science, as opposed to a launch event.

Secondary point: those 30,000 tokens are one of the best public artifacts of prompt engineering for expert work that exists right now. Encoding tacit domain expertise as executable instructions is a transferable skill. Reading that file will teach you more than ten listicles about prompting.

---

## 8. Analytical Chemistry: 19 Minutes Versus Four Days

> "Claude worked out how the data was encoded, then confirmed it had read the file correctly by reproducing the instrument's own recorded totals for all 2,664 scans before analyzing anything."

The second experiment used Opus 5 — a generally available model anyone can access.

The setup: raw instrument files from a contract lab (NMR and LC-MS), plus a two-sentence plain-language prompt. No vendor software. No operator.

The result: NMR processed in 23 minutes, LC-MS in 19, running in parallel. Hydrogen counts per peak landed within 0.08 ¹H of the lab's own numbers. Purity: 96.4% versus the lab's 96.33%.

The LC-MS file was in an undocumented proprietary binary format that only the manufacturer's software is meant to read. Claude reverse-engineered the encoding — and then, before analyzing anything, validated its own reader by reproducing the instrument's recorded totals across all 2,664 scans.

For comparison: a chemist doing this by hand typically spends 30 to 60 minutes per sample, and the lab's finished report for this sample arrived four days after the first spectrum was acquired.

It also showed something closer to judgment. It flagged four broad peaks as probably hydrogens bound to nitrogen or oxygen, and proposed the standard follow-up — add heavy water and see which peaks shrink. The lab had independently run that exact experiment three days later. Given the heavy-water file, Claude caught an overstatement in its own first pass (it had claimed all four peaks vanished; its self-check showed only two did) and corrected to the lab's conclusion.

**My take:** Cracking the vendor format is the flashy part. Validating the reader against the instrument's own totals *before* drawing conclusions is the professional part — and it's a step plenty of human analysts skip. As for four days versus 25 minutes: that's not an efficiency gain, it's a phase change in research tempo. When feedback moves from days to minutes, the way you design experiments changes too.

---

## 9. MBP: 90 Designs, Zero Binders

> "Against MBP, however, none of the 90 designs was confirmed to have bound to the target, although one demonstrated a weak, reproducible binding signal."

Two targets gave Claude real trouble: BBF-14 and maltose-binding protein.

BBF-14 is a β-barrel that doesn't exist in nature — it was itself de novo designed, which is precisely why it's used as a benchmark. Claude still produced three independent binders, one from each design arm and each on a different backbone, but with only modest affinities.

MBP was a shutout. Ninety designs, none confirmed, one weak reproducible signal. MBP is large, flexible, and bacterial, with a smooth hydrophilic surface that leaves a binder almost nothing to grip.

**My take:** Including this section is what makes the rest believable. They also disclosed that GDF-8 (Mature) was dropped from the results because target aggregation and non-specific stickiness made the data inconclusive — which is why 16 targets became 15. A report that publishes its failures *and* its data-exclusion reasoning is worth several that publish only clean numbers.

---

## 10. Dual-Use: The Strongest Capability Is the One You Can't Have

> "As we work to deliver these capabilities safely via trusted access programs, protein design and other dual-use research biology capabilities remain unavailable for general access in Claude Fable 5."

Autonomous biological research cuts both ways. The same capability that compresses drug discovery can help a bad actor develop a bioweapon.

So the deployment looks like this: life science research tasks are blocked in the most capable model, protein design isn't available to general users, and access is delivered through trusted access programs instead. Anthropic says launching a scientist access program is one of its highest priorities. In the meantime, Opus 5 is the strongest generally available model.

And then, in the conclusion, they talk themselves down:

> "Protein minibinders are not a standard therapeutic modality for drugs and even for the common drug modalities, such as monoclonal antibodies and small molecules, designing a high-affinity binder is just the first step in the process of generating a drug-like molecule."

Minibinders aren't a standard drug format. Even for antibodies and small molecules, a high-affinity binder is step one. Toxicity, pharmacokinetics, and clinical trials are where drug programs actually die.

**My take:** Both things are true at once: this is a real technical result, and it's a long way from "AI made a drug." Not many organizations write the second sentence into the last paragraph of their own scorecard.

On the dual-use side — capability-tiered release and trust-graded access is probably the default shape for frontier models in high-risk domains from here on. Which means "the strongest model" and "the strongest model you can use" are now two different things, and the gap will widen.

---

## The Through-Line

Put both experiments together and they say the same thing: **AI is moving into domains where verification is expensive.**

AI moves fast in math and code because feedback is nearly free — you find out immediately when you're wrong. Life science inverts that. One verification round costs weeks, real money, and a whole lab. In that regime, the value of AI isn't "try more things." It's raising the hit rate of each expensive attempt.

Going from 10% to 35% doesn't just improve accuracy. It doubles what a research budget buys.

There's a narrower lesson for anyone building tools. Claude didn't replace the specialist protein models — it conducted them. The role that got automated was the one in the middle: deciding what to run, in what sequence, and when a result is bad enough to start over.

That role exists in every industry.

---

*Source: Anthropic research post, How Claude is accelerating protein design and analytical chemistry*
*Open data and prompts: https://huggingface.co/datasets/Anthropic/claude-protein-binder-design*
