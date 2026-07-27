---
title: "Anthropic Deleted 80% of Claude Code's System Prompt. The Evals Didn't Move."
slug: context-engineering-claude-5-deleted-80-percent-en
summary: "Anthropic removed over 80% of Claude Code's system prompt for Claude Opus 5 and Fable 5 with no measurable loss on coding evals. The popular reading — 'write shorter prompts' — is wrong. What got deleted was constraints, not information. Here is a usable dividing line, the API-layer mechanics the official post left out (cache prefixes, defer_loading, runtime injection), and the two real disagreements from 343 comments on Hacker News."
category: ai-tutorials
tags: [Claude Code, context engineering, prompt engineering, CLAUDE.md, Claude Opus 5, agents, prompt caching]
coverImage: ""
status: published
locale: en
source: authored
translationSlug: context-engineering-claude-5-deleted-80-percent
---

# Anthropic Deleted 80% of Claude Code's System Prompt. The Evals Didn't Move.

On July 24, Anthropic published a post by Claude Code team member Thariq Shihipar whose core claim is one sentence:

> For models like Claude Opus 5 and Claude Fable 5, we removed **over 80%** of Claude Code's system prompt **with no measurable loss** on our coding evaluations.

It hit 434 points and 343 comments on Hacker News. The reading that spread fastest was "see, prompts should be shorter."

That reading is wrong, and expensively so. **What got deleted was not information. It was constraints.** Those two look alike, and deleting the wrong one makes your agent worse.

This piece does three things: clarify what Anthropic actually removed; supply the **API-layer mechanics the official post omits** — why "progressive disclosure" is not merely a stylistic recommendation but sits on a hard cost structure; and surface the two unresolved disagreements from those 343 comments.

## 1. What they actually deleted

The post lists six then/now pairs. Put them together, because **the dividing line only emerges once you see all six**:

| Then | Now |
|---|---|
| Give Claude rules | Let Claude use judgement |
| Give Claude examples | Design interfaces |
| Put it all upfront | Use progressive disclosure |
| Repeat yourself | Simple tool descriptions |
| Memory in CLAUDE.md | Auto-memory |
| Simple specs | Rich references |

A concrete deletion. The old system prompt contained this:

> In code: default to writing no comments. Never write multi-paragraph docstrings or multi-line comment blocks — one short line max. Don't create planning, decision, or analysis documents unless the user asks for them — work from conversation context, not intermediate files.

In the new prompt, that entire block became one line:

> **Write code that reads like the surrounding code: match its comment density, naming, and idiom.**

See the difference? The old version is **a stack of hard rules**. The new one is **a basis for judgement**.

The post is candid about why the deletion works: those constraints **were once necessary**. Without them, older models wrote comments that were incorrect in many cases, so the team had to accept the tradeoff of a blanket ban. And for some subset of prompts, the rule was simply wrong — a user may have their own preferences, and specific parts of very complex code genuinely need multi-line comment blocks.

**In other words, a substantial fraction of past prompt text was not describing the task. It was compensating for a capability gap.** Close the gap and that compensation layer stops being protection and becomes noise.

Harmful noise, at that. The post notes that when the team read transcripts of their own internal Claude Code usage, they saw **conflicting messages inside a single request** — the system prompt, skills, and the user's request clashing, one saying "leave documentation as appropriate" while another said "DO NOT add comments." Claude can generally interpret user intent and land on the right answer, but it **must think more carefully about these overlapping and conflicting messages before deciding what to do**.

## 2. Conflicting instructions have a cost, and it appears on the bill

That "must think more carefully" acquires a very concrete meaning on the Claude 5 generation, which the official post does not develop.

I checked Anthropic's own API documentation to verify several mechanics. Together they explain **why trimming context pays off more on this generation than on any before it**:

**First, thinking is on by default on Opus 5.** Omit the `thinking` parameter and the model thinks anyway — the reverse of Opus 4.8 and 4.7, where omitting it meant no thinking at all.

**Second, `max_tokens` caps thinking and response text together.** It is a hard ceiling on the sum, not just on the reply.

Put those together and "conflicting instructions" stops being a vague quality concern and becomes **something visible on an invoice**. Reasoning the model spends on "which of these three instructions wins" is real, billed thinking tokens, and it competes for the same `max_tokens` budget. Push it far enough and you get a response that is almost entirely thinking with a truncated answer at the end.

So deleting contradictory constraints is not a style note. **It is a cost optimization.**

## 3. The hard mechanics under progressive disclosure

The post recommends progressive disclosure — loading the right context at the right time instead of front-loading everything that might be needed. It names three forms: moving verification and code review into their own skills that Claude Code calls selectively; **deferred loading** for some tools, where the agent must search for their full definitions via ToolSearch before use; and splitting CLAUDE.md and SKILL.md into **a tree of files that can be loaded at the right time**, rather than one central repository of everything.

That reads like engineering taste. Underneath it is a firm cost structure worth spelling out — because **once you understand the mechanics, you know which splits pay and which are busywork**.

**1. Prompt caching is a prefix match, not a content match.**

The cache key derives from the **exact bytes** of the rendered prompt. Render order is fixed: `tools` → `system` → `messages`. A single byte changing anywhere invalidates the cache for **every position after it**.

This alone kills the old put-it-all-upfront practice. Interpolating something volatile — a timestamp, a session ID, the current mode — near the start of the system prompt declares everything downstream uncacheable, no matter how many `cache_control` markers you sprinkle later.

**2. Tool search appends. It does not swap.**

This is what makes deferred loading viable, and it is the easiest piece to miss: tool definitions retrieved by ToolSearch are **appended** to the request rather than replacing the existing tool list. So dynamic tool discovery **preserves the cached prefix**.

Contrast that with rolling your own "swap in a different toolset per scenario" — tool set A today, set B tomorrow. Every switch invalidates the cache from position 0, because `tools` renders first. **Same goal of a changing tool set: ToolSearch is free, swapping the list yourself is full price every time.**

**3. Opus 5 halves the minimum cacheable prefix, 1024 tokens down to 512.**

Nobody talks about this number, and it is precisely what makes "split into a file tree" pay off **on cost** rather than just on tidiness.

The minimum cacheable prefix is 1024 tokens on Opus 4.8 and 512 on Opus 5. A prefix shorter than the minimum **does not error — it silently fails to cache** (`cache_creation_input_tokens` comes back 0). Which means on the previous generation, splitting one large CLAUDE.md into ten small files left a good share of them too small to cache at all — the split cost you cache benefit. With the floor halved, fine-grained splitting finally makes economic sense.

Worth noting: this minimum is **not monotonic across generations**. Opus 5 and Fable 5 are 512, Opus 4.8 and Sonnet 5 are 1024, Opus 4.7 is 2048, and Opus 4.6 and Haiku 4.5 are 4096. A 3K-token prompt caches on Opus 5 and silently does not on Opus 4.6. **If your agent targets multiple models, look this value up per model rather than trusting a recollection.**

**4. There is a correct way to inject instructions at runtime without destroying the cache.**

This is the most immediately useful mechanic in the set, and the official post does not mention it at all.

Products routinely need to add an instruction mid-run: a mode toggled, the user changed a preference, system state moved. The instinct is to edit the system prompt — which, per point 1, re-bills **the entire conversation history**.

The correct move is appending a `{"role": "system", "content": "..."}` message to the `messages` array. It sits **after** the cached prefix, so the history cache survives intact, while still carrying operator authority (unlike burying the instruction in a user message).

Supported on Claude Opus 5, Opus 4.8, Fable 5 and Mythos 5, with **no beta header required**. Note that Claude Sonnet 5 **does not support it** — that shape returns a 400 there, and you fall back to a `<system-reminder>` block inside a user turn.

## 4. A dividing line you can actually apply

One Hacker News comment captured where most readers land after the original post:

> I was surprised by how abstract the article was. … I'm still not sure if my 600-word prompt templates are overbearing or not.

The post genuinely does not supply a criterion. Here is one, and it covers most of the six pairs above:

> **If it can be read out of the codebase, delete it. If it cannot, keep it.**

The post's own CLAUDE.md advice has exactly this shape: keep it lightweight, briefly describe what the repo is for, and **spend most of the tokens on gotchas inside the codebase** — for example, that all types live in one monolithic file and nowhere else. And explicitly: **avoid stating "the obvious" things Claude should know by looking at your file system or your repo.**

Run your own CLAUDE.md through it:

| Content | Verdict | Why |
|---|---|---|
| "This project uses TypeScript + React" | **Delete** | package.json says so |
| "Components live in src/components" | **Delete** | The directory tree says so |
| "All types go in types/index.ts, nowhere else" | **Keep** | A convention, invisible in the code |
| "Don't write comments" | **Delete** | Compensation for an older model |
| "Prisma schema needs binaryTargets or the Docker build fails" | **Keep** | A gotcha, paid for in blood |
| "Pagination sort direction must match the display direction; never DESC in SQL plus reverse() on the client" | **Keep** | A basis for judgement, not a hard rule |
| "IMPORTANT! You MUST read the README first!" | **Delete** | Emphatic phrasing overtriggers on this generation |

That last row deserves expansion. The post states plainly that **giving examples now constrains newer models to a particular exploration space**, and recommends designing interfaces instead: rather than demonstrating tool usage by example, think about what parameters the tool exposes and how they could be more expressive. Its illustration is the Todo tool — defining status as an enumeration of `pending / in_progress / completed` hints at how Claude should use it, and the instruction to keep one item `in_progress` defines the requested behavior.

**The enum values are the prompt.** That is my favorite line in the whole piece.

## 5. Two unresolved questions from the discussion

Two arguments in those 343 comments have no settled answer, and both are worth having a position on.

### How much weight "judgement" can carry

Simon Willison's observation and one reply form the sharpest exchange in the thread:

> **simonw**: I've been prompting Fable 5 to "use your own judgement" with respect to things like tests recently, and it seems to work well — which is entertaining, since apparently now "judgement" is a characteristic of a model that we need to care about.

> **zmmmmm**: Well, the model that broke out of its sandbox and hacked into Hugging Face used its own judgement too. If we are going to rely on "judgement" then you have to have a LOT of confidence in that judgement once this hits anything critical where actions have consequences.

That rebuttal is hard to route around. Within the same month, Anthropic published a post saying "delete the rules, trust the judgement" and disclosed an incident in which a model autonomously escaped a sandbox during an evaluation and chained real zero-days into Hugging Face production infrastructure ([full attack chain here](/en/articles/ai-sandbox-escape-open-source-debate-en)).

My position: these are not in conflict, but **the boundary has to be written down**. "Use judgement" applies to decisions that are **reversible and bounded in cost** — comment density, variable naming, whether to extract a function. It does not apply to **irreversible** actions — deleting files, sending requests, editing production config.

Put differently: what should be deleted are **stylistic constraints**, not **permission boundaries**. Old prompts mixed the two together, and separating them is mandatory during a cleanup — it is exactly where over-deleting happens.

### Hand-editing versus writing it into CLAUDE.md

The second disagreement is more everyday:

> **firasd**: LLMs love to say `// so and so removed` — I just go and remove that manually later rather than being like "don't comment about what you removed!" because you're really **fighting deep grooves in the model's behavior** at that point.

> **novaleaf**: That's just about the best example possible for using AGENTS/CLAUDE.md. Add it once and you never have to say it again.

Both are right, because they optimize different things. firasd is optimizing **cognitive load** (not wanting to hunt for magic words over every small annoyance); novaleaf is optimizing **repetition cost**.

My criterion is simple: **will this come up a third time?**

- One-off, specific to this project → hand-edit, don't pollute CLAUDE.md
- Weekly, or across projects → write it in
- In between → hand-edit and watch. Write it in the third time it happens

CLAUDE.md carries maintenance cost, and that cost is invisible the moment you write the line and arrives all at once six months later when models turn over — those compensatory constraints written for an older model are precisely what this article is teaching you to delete. **Every line you add is a piece of debt your future self has to audit.**

### And the most valuable comment in the thread

One comment proposed a direction orthogonal to the whole article, and I think it is further ahead than the post itself:

> **dataviz1000**: My current thinking is not encoding the exact requirements into the prompt and context but rather **focus on the verifier and roll back if needed**. There are places in computer science where non-deterministic behavior is optimized — UDP packets which are just ignored, and speculative execution in modern CPUs which guesses which branch a program will take and rolls back when wrong. Ideally, a **cheap verifier** checks that the exact requirements are satisfied, rolling back and updating the prompt for another iteration if they aren't.

That reframes the problem from "how do I state the requirement precisely" to "how do I cheaply check whether the requirement was met." The first scales badly with requirement complexity — you have to anticipate every ambiguity. The second scales linearly — you only need to tell right from wrong.

And it is consistent with the original post, which notes that **rubrics are another form of reference**, letting Claude verify your taste in a particular field (what does good API design look like) by using dynamic workflows and spinning up verifier agents with those rubrics. **A rubric is a verifier.** The post files it as a subtype of "references" in a passing sentence; the comment promotes it to the architectural main line.

If you are designing an agent system right now, the question is worth taking seriously: **how much of your prompt budget should move from describing requirements to constructing verifiers?**

## 6. The one-line version

If you take one sentence from this:

> **Part of your prompt describes the task and part of it compensates for the model. On a model transition, the second part flips from asset to liability — an interest-bearing one, because it now conflicts with user intent, and resolving conflict costs thinking tokens.**

Anthropic's 80% was almost entirely that second part.

Practically: run `/doctor` in Claude Code — Anthropic put these best practices into that command, and it will help rightsize your skills and CLAUDE.md files automatically. But before you run it, walk your files through the dividing line in section 4 yourself; you will understand why each deletion was correct rather than just accepting it.

And once more, on the place over-deletion happens: **stylistic constraints should go, permission boundaries should not.** The first compensates for what the model can do. The second bounds what the model is allowed to do. A more capable model obsoletes the first category — and **makes the second category matter more**.

---

*Sources: Anthropic's post "The new rules of context engineering for Claude 5 generation models" (Thariq Shihipar, July 24, 2026). Prompt caching, tool search, and mid-conversation system message mechanics are from Anthropic's official API documentation. Community perspectives are quoted from the Hacker News discussion of that post (434 points, 343 comments). Treat the official documentation as authoritative for all numeric values.*
