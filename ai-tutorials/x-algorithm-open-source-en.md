---
title: "X Open-Sourced Its Feed. Reading It Is the Hard Part."
slug: x-algorithm-open-source-en
summary: "On August 13, 2026, xAI pushed the ranking weights, visibility filters and Phoenix training code behind X's For You feed to GitHub — roughly 10-15x more code than before. The next day it shipped another commit that changed no logic at all: comments explaining that -234 does not mean one report cancels 468 likes."
category: ai-tutorials
tags: [recommender systems, X, Twitter, xAI, open source, Rust, ranking, algorithmic transparency]
coverImage: ""
status: draft
locale: en
source: authored
translationSlug: x-algorithm-open-source
---

# X Open-Sourced Its Feed. Reading It Is the Hard Part.

> "This is the kind of thing that I think people will be fairly shocked that we are releasing."
> — Keith Coleman, VP of Product at X, to TechCrunch, August 13, 2026

The repo itself isn't news. `xai-org/x-algorithm` was created on January 19, 2026 and got its first commit the next day. What's worth reading are the commits from August 13th and 14th: the actual numeric ranking weights, the visibility-filtering system that decides whether a post can be shown at all, and the real Phoenix training code with synthetic data generation to go with it. TechCrunch reports X's own estimate that the codebase grew 10 to 15 times.

As of this writing (2026-08-15), the repo sits at 29,347 stars, 4,918 forks and 54 open issues, Rust-first, Apache-2.0. For contrast, the Scala-based `twitter/the-algorithm` from March 2023 has 73,738 stars — more attention, considerably less substance.

Seven things stood out after reading the README, `param.rs`, `brazil_2026_election_filter.rs`, `docs/BIDIRECTIONAL_BOOST_CHANGE.md` and `phoenix/QUICKSTART.md`.

---

## 1. The real difference from 2023: the model actually runs

> "All of the code here is inspectable, and some of the code is even designed to be runnable end-to-end — e.g. training and running the Phoenix scoring model."
> — README, *What's not in this repo?*

In 2023, Twitter published the heavy ranker's plumbing but no training path. You could read it; you couldn't run it. This time `phoenix/` ships a Cargo workspace, a `pyproject.toml`, a QUICKSTART and deterministic synthetic data generation — enough to train a small model and serve it. TechCrunch notes that X previewed the release to outside recommender-systems researchers who independently trained and ran Phoenix.

`QUICKSTART.md` is refreshingly blunt about the ceiling:

> "It is not a production-quality model or a production-scale setup. Production data, checkpoints, orchestration, and scale are not included."

The entry cost isn't trivial either — Linux, an NVIDIA GPU, CUDA 12, a Rust toolchain — and the walkthrough trains for six steps, with an explicit warning not to read six steps as evidence of model quality.

**My take:** runnable and reproducible are different properties, and this release delivers the first one. You can confirm the architecture learns; you cannot reconstruct the behavior of the model serving your timeline, because neither the data nor the checkpoints are here. That's still a real step up — a skeleton you can execute beats a pile of unexplained class definitions. Just don't mistake a demo for an audit tool.

---

## 2. It doesn't predict whether you'll like a post. It predicts what you'll do.

> ```
> Final Score = Σ (weight_i × P(action_i))
> ```
> — README, *Scoring and Ranking*

Phoenix doesn't emit a relevance score. It emits probabilities for twenty-odd distinct actions: favorite, reply, repost, quote, share via DM, share via copy link, photo expand, video open, link click, profile click, dwell, dwell time, active seconds, follow author — plus the negatives: not interested, mute, block, report, not dwelled. `RankingScorer` collapses them with a set of weights that live in source.

The payoff is architectural: **the model estimates likelihood, and the value judgments are quarantined in a table of human-readable constants.** Want to know how much more X cares about a quote tweet than a like? You don't reverse-engineer embeddings. You open `home-mixer/params/param.rs` and read `QuoteWeight = 5.0` next to `FavoriteWeight = 0.5`.

One easy-to-miss design note (Key Design Decisions #2): during transformer inference, candidates cannot attend to each other — only to viewer context. A post's score therefore doesn't depend on what else is in the batch, which makes scores stable, cacheable and independently recomputable.

**My take:** this is the part other teams should copy. Multi-action prediction plus explicit weights cleanly separates "what the model learned" from "what we want." Shifting priorities means editing an f64, not retraining. That's where the interpretability comes from — and, as point 5 shows, also where the trouble starts.

---

## 3. Ranking and visibility are separate systems — the mechanics of "limited, not banned"

> "Ranking decides the order. Visibility filtering decides whether a post can be shown at all. Different services, different inputs, different rules."
> — README, *Key Design Decisions #4*

`visibility-filtering/` answers one of three ways per (post, viewer) pair: ALLOW, INTERSTITIAL (shown behind a tap-through screen), or DROP. It runs *after* ranking, reading labels produced by the labeling path — `scarecrow/`, `botmaker/`, `agatha/`, `bdsm/`, `user-cred-v2/` — plus the viewer's own blocks, mutes, follows, country and settings.

The single most clarifying sentence in the README:

> "Some rules drop a post only when it is a recommendation from an account the viewer does not follow — spam caught at high recall, for instance. The same post is allowed to a follower."

Same post, allowed to your followers, dropped for strangers. That is what years of shadowban argument look like as code: not a ban switch, but a rule set that evaluates recommendation-visibility separately from subscription-visibility, in order, stopping at the first DROP.

**My take:** publishing this matters more than publishing the ranking weights. Ranking is a taste question; visibility filtering is a power question. The design is defensible — apply high-recall spam heuristics only to strangers, where false positives cost least — but defensible things still need outside review. `visibility-filtering/rules/registry.rs`, which lists both rule sets in evaluation order, is the file I'd most want regulators to actually read.

---

## 4. The 468x story: open code is not the same as public understanding

> "One common misinterpretation is that you can read these weight ratios as count equivalences, e.g. the incorrect statement that 'one report cancels 468 likes'."
> — `home-mixer/params/param.rs`, comment added August 14, 2026

The number is real. `FavoriteWeight = 0.5`, `ReportWeight = -234.0`, and 234 ÷ 0.5 = 468. The ratio is right there in the file.

The reading is what's wrong. Weights multiply **predicted probabilities**, not **observed counts**. The baseline probability of a report is more than 1000x lower than a like — the comment says so explicitly — so the large magnitude exists to let that prediction register in the final score at all, not to make one real report erase 468 real likes. And because recommendations are personalized, brigaded reports mostly move rankings for users similar to the brigaders rather than suppressing a post globally.

The August 14 commit added this explanation to both `param.rs` and `ranking_scorer.rs`. The README's stated reason is striking: "so that LLMs or people reading it are more likely to understand it correctly."

That wasn't paranoia. On August 14, explainx.ai published a piece headlined *"X open-sources For You ranking weights: a Report costs 468 likes of reach."* X was writing the correction on roughly the same day the misreading went to press.

**My take:** this is the most interesting item on the list, because it exposes what transparency actually costs. Publishing code is step one; step two is a permanent race against misinterpretation, and half your readers are now language models. The fact that X's stated motivation for a code comment is *keeping LLMs from getting it wrong* says more about 2026 than the release itself. Worth noting, though: those comments are also an argument, not just an explanation. X asserts that coordinated reporting can't meaningfully suppress reach — validating that claim requires studying how `agatha/` and `bdsm/` actually behave, not taking the comment's word for it.

---

## 5. What you're reading is a snapshot — refreshed roughly every three months

> "On July 13, 2026, after seeing great initial results from the A/B test, we rolled out a boost value of 20 to many users... Then, on July 24, 2026... we set the bidirectional follow reply boost value to 15 instead of 20."
> — `docs/BIDIRECTIONAL_BOOST_CHANGE.md`

This is the most underrated file in the repo. It walks through one parameter's full life: on July 10 an A/B test randomly assigned `BidirectionalFollowReplyWeightBoost` values of 5, 10, 15 or 20 (most users at 0). On July 13, results looked good and 20 shipped broadly. On July 24 — and the reason given is unusually concrete, that the World Cup was on and people weren't seeing enough of the conversation because much of it came from accounts they don't follow — it dropped from 20 to 15. In the repo that's one line: `-20.0` / `+15.0`.

The README also concedes that live tunables are read from a configuration system, that cron scripts periodically rewrite the in-repo defaults to match production, and that the commitment covers experiments running at "a notable share of traffic — e.g. 10% or more."

Then check the cadence. Pull the commit list from the GitHub API and the entire history is **six commits**: 2026-01-20, 2026-05-15 (x2), 2026-08-13 (x2), 2026-08-14. The January announcement promised updates every four weeks. What shipped was January, then May, then August.

**My take:** "read the code and you'll know the algorithm" needs an asterisk. Parameters move daily; the repo syncs quarterly. You are reading a cron-refreshed snapshot, not live state. `BIDIRECTIONAL_BOOST_CHANGE.md` proves they can narrate a change well when they choose to — which means the question worth pressing isn't whether to open-source, but how often it syncs and when experiment-layer data becomes public. Open-sourcing is the promise; cadence is the delivery.

---

## 6. What's withheld, and what's offered in exchange

> "To reduce the risk of this, there are a limited set of files not currently published in the repository, e.g.: Grox prompts... Some botmaker rules"
> — README, *What's not in this repo?*

Two categories stay out: the specific LLM prompts (j2 files) used by Grox, the content-understanding system, and some botmaker labeling rules. The rationale is anti-gaming — knowing the prompts means knowing how to slip past spam classification. TechCrunch frames it as protection against bad actors working around the rules to flood the network.

X's compensation is Under the Hood: you don't get the rules, you get **the rules' output on you**. Eligible users download a JSON file showing which visibility-affecting labels were applied to their account and posts over the previous calendar month. It's a pilot — a randomized group of accounts at least a year old with 10+ posts in the prior month. TechCrunch notes the intended workflow for non-technical users: drop the JSON into an LLM, point it at the GitHub repo, ask for an interpretation.

**My take:** I'll grant the tradeoff and name its limit. "Code plus outputs" lets you verify *how the rules apply to me*. It cannot verify *whether the rules apply fairly across everyone* — you see your own labels, never the distribution. Real algorithmic audit needs cross-account aggregates, and Under the Hood is per-account by construction. There's also something quietly telling in "just ask an LLM to interpret it," sitting alongside code comments written specifically so LLMs read the weights correctly: the working audience for this transparency stack is models, with humans as the relay.

---

## 7. Brazil2026ElectionFilter: a law, compiled

> "Application providers that use a recommendation system for users must exclude from the results the channels and profiles reported to the Electoral Court..."
> — header comment in `home-mixer/filters/brazil_2026_election_filter.rs`, quoting Brazilian electoral law

Open the file and you find a hardcoded set of 665 user IDs — a test literally asserts `BRAZIL_2026_ELECTION_USER_IDS.len() == 665`. Each ID carries a comment with its @handle, and the file explains why: "User ids below are obfuscated; usernames are included for transparency." There's even a note that "OmarAzizSenador deleted his account at the time this code was written."

The behavior is one line of policy: posts from these accounts don't enter For You unless the viewer explicitly follows them. The README's framing:

> "A benefit of open-source is that you can see that changes like this exist, and exactly how they work."

**My take:** here I think X is straightforwardly right, and this may be the strongest single argument in the whole release. Compliance-driven content restrictions exist on every platform; the only variable is whether you can see them. Here, an election statute has become a Rust file, 665 integers and an `unless the viewer follows` clause. You can dispute how the list was assembled, whether it exceeds what the law requires, what the removal process is — but you have something concrete to dispute. That's what open source changed. It didn't make the restriction disappear; it made it reviewable line by line.

---

## Closing

Seven points in, my read is that this release is technically generous and institutionally incomplete.

The generous part is substantive: multi-action prediction with explicit weights, a clean split between ranking and visibility, training code that actually executes, and a postmortem document that narrates a parameter change honestly. Any team building recommenders can learn from this. It is not a press-release open-sourcing.

The incomplete part is verifiability. The repo syncs every few months, production checkpoints aren't included, Under the Hood only offers a per-account view, and the central question — whether the published code matches what's running — still rests on trust. Coleman's stated dream is that "anyone in the public can be able to assess how posts are distributed on the platform, vet that it's a level playing field." Code is necessary for that. It isn't sufficient.

That said: Meta, YouTube and TikTok have published exactly zero ranking weights. The right yardstick for this release is how far it still is from auditable, not how much better it is than closed — but both halves of that sentence deserve saying.

---

**Sources**

- [xai-org/x-algorithm](https://github.com/xai-org/x-algorithm) — README, `home-mixer/params/param.rs`, `home-mixer/scorers/ranking_scorer.rs`, `home-mixer/filters/brazil_2026_election_filter.rs`, `docs/BIDIRECTIONAL_BOOST_CHANGE.md`, `phoenix/QUICKSTART.md` (repo stats and commit history verified via GitHub API on 2026-08-15)
- [X open sources its ranking algorithm, letting users see if they've been 'shadowbanned'](https://techcrunch.com/2026/08/13/x-open-sources-its-ranking-algorithm-letting-users-see-if-theyve-been-shadowbanned/) — TechCrunch, 2026-08-13
- [X open-sources For You ranking weights: a Report costs 468 likes of reach](https://explainx.ai/blog/x-algorithm-for-you-timeline-open-source-ranking-weights-august-2026) — explainx.ai, 2026-08-14 (cited as an instance of the misreading)
- [X shares new insights into transparency and shadowbanning](https://www.socialmediatoday.com/news/x-shares-new-insights-into-transparency-and-shadowbanning/827858/) — Social Media Today
- [twitter/the-algorithm](https://github.com/twitter/the-algorithm) — the March 2023 release, for comparison
