---
title: "They Ran a Lights-Off Software Factory for Four Months. Then a Cofounder Rewrote It by Hand"
slug: lights-off-software-factory-postmortem-en
summary: "An essay titled 'Why Software Factories Fail' hit 390 points on Hacker News. Author Dex Horthy is no AI skeptic — he built his reputation teaching people to use coding agents. In July 2025 he ran a no-human-reads-the-code factory at his own company; after the third incident, his cofounder spent two weeks rebuilding the codebase by hand. His root cause: in the RL loop that trains coding models, eroding maintainability carries no penalty. A model-training problem, not a skill issue."
category: ai-tutorials
tags: [software factory, AI coding, coding agents, reinforcement learning, RLVR, code quality, SWE-bench, comprehension debt, Claude Code]
coverImage: ""
status: published
locale: en
source: authored
translationSlug: lights-off-software-factory-postmortem
---

# They Ran a Lights-Off Software Factory for Four Months. Then a Cofounder Rewrote It by Hand

In late July, a long essay titled *Why Software Factories Fail* climbed the Hacker News front page, sitting at **390 points** as I write this, with hundreds of comments. It expands on the author's keynote at AI Engineer World's Fair 2026, and its subtitle is the real thesis: **harness engineering is not enough**.

What makes the essay worth your time is the opposite of what you'd expect from a "AI can't code" piece:

**The author is not an AI skeptic. He is one of the people who taught the industry how to use coding agents in the first place.**

Dex Horthy founded HumanLayer, wrote *12-Factor Agents*, and his talks on advanced context engineering have racked up about a million cumulative views on YouTube. Teams everywhere have built agent workflows on his playbook.

That person is now on record saying: I actually ran the "nobody reads the code" factory at my own company — and it ended with my cofounder spending **two full weeks in VS Code**, rebuilding our patterns line by line.

## 1. What a "lights-off factory" actually means

"Software factory" is not a 2026 coinage. The term traces back to 1968 — the same NATO conference that gave us "software engineering." For half a century people have dreamed of turning software into a repeatable production line instead of artisanal craft.

The AI-era version looks like this: work goes into a queue; agents pick up tickets and build; automated tests and review bots gate the output; incidents and user feedback flow back into the same queue. The human job reduces to two questions: how much can you stuff into the queue, and how fast can you review what comes out.

Then someone notices that review is the bottleneck — so why not delete it?

That is the **lights-off factory**. The term is borrowed from manufacturing: FANUC in Japan has run factory floors with the lights physically off since 2001, because the only workers are robots and robots don't need light. The software equivalent: **code ships to production that no human has ever read**, verified only by machines.

This isn't hypothetical. StrongDM publicly operates a factory where "no human writes code and no human reads code" — Simon Willison covered it in February. Ryan Lopopolo at OpenAI wrote about harness engineering the same month and gave a talk about Symphony, OpenAI's internal software factory. Ramp, Stripe, WorkOS, and Brex have all spent this year explaining how agents now produce around 75% of their code.

The narrative underneath it all takes four sentences: You are the bottleneck. The models are good enough. Code is free. Just ship more stuff.

## 2. He actually tried it: lights off in July 2025, hand rewrite by November

Dex's team went fully lights-off in July 2025. Humans read specs and tickets only; background agents handled everything small and medium; nobody read the output.

Here is how his postmortem says it played out.

**The first time**, they hit a bug gnarly enough that no agent could crack it — not with deep context-aware research, not with ten different reproduction strategies. Eventually he had to climb back into a codebase nobody had read for three months and debug it by hand. Meanwhile: the site was down, users were furious, and he was — his words — miserable, reading all the slop code he had let slip into the system.

He shook that one off. "The downside risk was worth the velocity."

**By the third time, in November**, the team made a different call: rewriting from scratch would be easier than fixing. His cofounder spent **two whole weeks** in VS Code — he pointedly notes "not even Cursor" — plumbing out all the patterns by hand.

July to November: the experiment lasted roughly four months. Keep that number in mind, because it matches another observation in the essay: at today's shipping pace, an agent-built codebase starts to struggle after maybe **three to six months**. "Brownfield" used to mean a decade-old Java system. Now it can mean the project you vibed into existence last quarter.

## 3. The root cause is not in your harness. It's in the training loop

The essay's real contribution is not the war story — it's the layer underneath: **why do models behave this way?**

The answer starts with how coding models are trained. The reinforcement learning loop (RLVR) goes roughly: generate agent traces that attempt a task, score each trace with a verifier, then update the weights to make high-scoring traces more likely. You run this millions of times, which means scoring must be fast and reliable.

That's where the problem lives. Take SWE-bench Multilingual: tasks mined from real issues in open-source repos like Redis, Django, and fastlane, each about fifteen minutes of human work. The reward is binary:

- `FAIL_TO_PASS` — did you fix the thing you were asked to fix?
- `PASS_TO_PASS` — did you avoid breaking anything else?

Pass gets a 1, fail gets a 0. He walks through a real task: fastlane's zip action calls `.empty?` on two optional params and crashes when they're omitted. The human fix that closed the issue was **two lines** — default the params to empty arrays. How the model gets to a passing state — whether the patch is elegant, whether it leaves behind defensive try-catch wrappers and type casts that gut the type system — **has zero effect on the score**.

He sets one line in giant type:

> **there is no penalty for eroding codebase maintainability**

Worse, the timescales don't match:

> Tests give you feedback in seconds, but the cost function of bad architecture is measured in weeks, months, maybe even years.

A bad design decision detonates months later as an incident — and nothing exists to backpropagate that incident to the decision that caused it. RL needs a fast, reliable oracle, and maintainability has none. Which produces the sharpest sentence in the piece:

> If a model could reliably tell good code from bad, it might have written the good version to begin with.

As an aside, the same logic explains something else: how Claude Code went from zero to roughly $9 billion in run-rate revenue in about a year, blowing past earlier CLI agents like aider and cline that had genuinely good context engineering. The canonical explanation is that Anthropic ran RL **inside the harness** — the first time a lab trained a model against the exact tools it would ship with. RL decides what a model is good at. What RL never taught, no amount of loops and prompts will patch in later.

## 4. The supporting data — with the honesty label attached

Faros AI published a report on what happened after teams adopted AI coding tools at scale early this year: review comments up 25%, comment length up 22.7%, and **31.3% of pull requests merged with no review at all**; incidents per PR up 242.7%, monthly incidents up 57.9%, bugs per developer up 54%.

To be fair — and Dex flags this himself — this is **a correlation signal, not a smoking gun**. The AI adoption wave coincided with teams deliberately shipping faster, and the causality is tangled. His assessment: "directionally valid based on what I've seen."

## 5. The counterarguments: Hacker News did not simply nod along

The HN thread pushed back hard on several fronts, and the objections deserve to be presented straight:

**"Your experiment is stale."** A highly upvoted comment noted the widely accepted step-change in model capability between fall 2025 and spring 2026: experience from a July 2025 failure may simply not apply to current models. Dex pre-empts this in the essay. His answer: models got dramatically better at *one-off problem solving*, but as far as he can tell, not much better at *maintaining codebase quality over time* — and he concedes he **cannot prove it, because no benchmark measures that ability**. Neither can you disprove it, for the same reason.

**"StrongDM seems to be doing fine."** One commenter pointed out that people from StrongDM's lights-off factory have since launched a consulting firm on the back of it — suggesting the approach didn't fail, and that the real variable is rigor and discipline, not possibility.

**"Why not just add an RL penalty for bad design?"** The right question, and the industry is trying: Abundant AI's SWE-Marathon builds ~400-hour tasks with a compound reward channel instead of a single pass/fail bit; Cognition's Frontier Code penalizes tests that don't fail on pre-patch code (a mutation-testing move) and runs a judge model over diffs with code-quality rules. Dex's verdict: pointed the right way, but he still wouldn't bet a codebase on any of them — because a judge model runs into the same paradox: a model that could reliably judge quality would already write quality.

**"Modularity routes around it."** Several commenters proposed letting humans fix the module boundaries, then unleashing agents inside modules where maintainability barely matters. Which, as it happens, converges with Dex's own prescription.

## 6. His fix: not lights off — judgment moved upstream

The essay's prescription is almost anticlimactically conservative: put code review back, then **use planning to make review cheap**. Four phases, each human-with-agent rather than fully automatic:

1. **Product review** — one short doc pinning down the problem and what success looks like; an HTML mockup settles arguments three paragraphs would prolong.
2. **System architecture** — how services, endpoints, schemas, and queues talk to each other, aligned with sequence diagrams and contracts.
3. **Program design** — the phase he calls criminally underemphasized: before anyone (human or agent) writes the implementation, align on types, method signatures, call-stack trees, and file-tree diffs. The model drafts them, you argue with it. Every one of those is a decision you would otherwise make implicitly during code review — the most expensive possible moment to change your mind.
4. **Vertical slices** — reject the model's beloved "horizontal plan" (all migrations, then all services, then all APIs, then all frontend) in favor of thin end-to-end slices you can poke with curl, reviewing 100–200 lines and re-steering as you go.

He's not absolutist about it either: roughly **40% of tasks at his company still get one-shotted** to an agent; only medium and large changes go through the full process. The underlying arithmetic: **thirty minutes of planning saves hours of review**. Trade the fantasy of "10–100x faster and nobody reads code" for the reality of "2–3x faster, safely."

His closing advice is four words long: **Read the dang code.**

## 7. My take: this is the other side of the "deleted 80% of the prompt" coin

Google's Addy Osmani wrote a follow-up essay, *Software Factories, Light and Dark*, contributing a term that may outlive the original: **comprehension debt** — the widening gap between how much code exists and how much any human still understands. A dark factory doesn't pay that debt down; it takes it on as fast as it can, **with the tests green the whole way**. And the reckoning, when it comes, won't be dramatic. It will be quiet and late.

Read this alongside our earlier piece on [Anthropic deleting 80% of Claude Code's system prompt](/articles/context-engineering-claude-5-deleted-80-percent-en) and a clean division of labor emerges. Context engineering governs the **ceiling of a single run** — a leaner prompt and a sharper harness optimize the quality of this one output. What Dex is pointing at is a different axis entirely: **the long-term fate of a codebase is governed by how much understanding humans retain**. The first thing can be engineered. The second currently cannot, because it appears in no training loop's reward function.

For working developers, the actionable residue is about three items:

1. **Distrust the comfort of green tests.** Green proves only what the verifier cares about. What is actually happening to your codebase, neither the verifier nor the model knows — only someone who reads the code does.
2. **Move your arguments with the agent to before the code exists.** Types, signatures, call stacks, file layout: a one-sentence correction at planning time becomes a 2,000-line unreadable diff at review time.
3. **Set a three-to-six-month alarm on any agent-heavy codebase.** When "change one thing, break three things" sets in, you haven't gotten worse at your job. The comprehension debt has come due.

One last thing, in fairness: Dex discloses his bias in the first paragraph — HumanLayer sells human-agent collaboration tooling, and this essay is good for business. But his strongest evidence doesn't require trusting him at all. SWE-bench's grading rules are public. Anyone can go check that between the 0 and the 1, there is no slot for good design.

---

**References**

- Original essay: [Why Software Factories Fail (or: harness engineering is not enough)](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/wsff.md)
- [Hacker News discussion (390 points)](https://news.ycombinator.com/item?id=49023019)
- [Addy Osmani: Software Factories, Light and Dark](https://addyo.substack.com/p/software-factories-light-and-dark)
- [AI Engineer World's Fair 2026 keynote video](https://www.youtube.com/watch?v=Ib5GBkD555M)
- [Faros AI: AI Acceleration Whiplash report](https://www.faros.ai/research/ai-acceleration-whiplash)
- [Simon Willison on StrongDM's software factory](https://simonwillison.net/2026/Feb/7/software-factory/)
