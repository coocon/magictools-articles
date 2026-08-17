---
slug: "claude-text-watermark-en"
title: "Claude's Text Watermark: A Signature Hidden in the Dice Rolls"
summary: "Anthropic published the full mechanics of Claude's text watermark — nothing added to the text, no hidden characters, no extra tokens, no price change. What's more interesting is the other half of the document: the long list of cases where the watermark barely works at all."
tags: [Claude, Anthropic, watermarking, SynthID, EU AI Act, AI detection, LLM]
---

# Claude's Text Watermark: A Signature Hidden in the Dice Rolls

> "Future Claude models will generate text that contains a watermark."

---

Since August 2, 2026, the EU requires AI providers serving its market to mark AI-generated content. Anthropic and around 190 other signatories signed the EU Code of Practice on Transparency of AI-Generated Content in July 2026. Every major lab is shipping some version of this. Anthropic just published how theirs works.

The document is unusually candid — roughly half of it is spent explaining where the watermark *fails*. Ten things worth pulling out.

---

## 1. The Watermark Isn't in the Text. It's in the Dice.

> "When watermarking is used, choices are still made at random, but the _source_ of the randomness is different."

If your mental model of a watermark is a faint pattern on a banknote, a logo in the corner of an image, or zero-width Unicode characters smuggled between words — throw all three away.

Here's the actual mechanism. An LLM writes one word at a time, picking from a list of candidates. Take "The weather today was cold and…". "Sugary" is out. "Overcast" and "grey" are both fine, and the reader does not care which one lands. Normally a random number settles it.

Watermarking swaps out where that random number comes from:

> "Instead of using an arbitrary random number generator to pick the next word, watermarking uses the key and a few words that come before to settle what word the model should pick."

Key plus preceding context decides the coin flip. The words are the same words. The sentence means the same thing. But the *sequence* of those coin flips forms a pattern that anyone holding the key can check for.

**My take:** The elegance here is that it's subtractive, not additive. Nothing gets encoded into the output — an existing source of randomness just gets a new seed. Which means the entire class of attacks aimed at stripping hidden payloads has nothing to strip. You can't delete a character that was never inserted.

---

## 2. The Monopoly Analogy Is the Whole Idea

Anthropic's best paragraph is a board game:

> "For all intents and purposes, the moves are still random: it makes no difference to the players—or to the outcome of the game—whether the randomness comes from pi or from dice rolls each time."

Play Monopoly, but stop rolling dice. Instead, open a book of the digits of pi, start at some arbitrary position, and read off successive digits as each player's move.

Nothing changes for anyone at the table. The game feels random because it *is* random. But hand someone the full move history afterward, and — if they know what pi looks like — they can work out that this particular game was probably played out of the book.

**My take:** What this analogy really does is pry apart two ideas we normally treat as one: *random* and *untraceable*. Randomness can be perfectly reproducible if you know the seed. Cryptographers have known this forever; the novelty is applying it at the token level and letting the model provider hold the seed. Note what that implies, though — Anthropic's key is now a genuinely load-bearing secret. The paper doesn't dwell on key management. I would have liked a paragraph on it.

---

## 3. The Quality Question Was Answered With Live Traffic

> "In internal testing, we've seen no impact of watermarking on the content, level of creativity, or readability of Claude's text."

Internal testing alone would be worth a raised eyebrow. But the SynthID-Text paper backs it with something harder to wave away: Google DeepMind shipped a watermarked model to a slice of real Gemini traffic and compared thumbs-up and thumbs-down ratings.

> "They found no statistically significant differences from the unwatermarked model."

Plus a controlled version:

> "And in a controlled study, human raters comparing watermarked and unwatermarked answers side-by-side saw no difference in quality."

One misconception the document heads off directly:

> "Importantly, it isn't that the model will now always be biased toward overcast or grey."

The watermark doesn't develop a favorite word, and it won't push Claude toward vocabulary it would never have reached for otherwise. It only operates inside the set of choices that were already interchangeable.

**My take:** A live A/B on production traffic is worth more than any eval suite, because it measures what users actually felt rather than what a benchmark can score. For anyone building on the API, the practical answer is in one sentence: no extra tokens, same price to serve and use, negligible speed impact. "Will this raise my bill?" is the first question most developers ask, and the answer is no.

---

## 4. This Is Compliance, and It Went Global by Default

> "We're implementing watermarking to comply with the EU AI Act."

No framing about voluntary leadership in AI safety. The first line of the "why" section is a legal obligation, stated plainly.

The more consequential sentence is this one:

> "We're applying watermarking globally at launch because we don't yet have a durable way to scope it by region."

An EU statute, applied to every user on earth. Anthropic says it will keep evaluating alternatives and share updates.

**My take:** This is the Brussels Effect in its purest textbook form — one bloc legislates, the global default shifts. Region-scoping is technically feasible on paper (IP, account jurisdiction, API region), so "no durable way" reads to me as a judgment that any such boundary would be trivially circumvented, and a leaky watermark is worse than a universal one. That's a defensible call. It's also the convenient one, and it's worth naming both things at once.

---

## 5. Nobody Invented This Last Quarter

> "Claude's text watermark is a version of the SynthID-Text approach published by Google DeepMind in a *Nature* paper in 2024."

Not in-house research. A version of Google DeepMind's SynthID-Text, published in *Nature* in 2024. And the lineage runs back further:

> "It belongs to a family of approaches that go back to a proposal by Scott Aaronson in 2022, all of which share the same design principle that we described above—the watermark only changes the source of the randomness used to pick among words."

A 2022 proposal from Scott Aaronson, formalized by a rival lab in 2024, shipped as regulatory compliance by a third lab in 2026.

**My take:** Read this as an industry signal rather than a technical detail. Compliance is exactly the wrong place for proprietary cleverness — a shared method with per-provider keys gives you a reusable detection ecosystem and clean privacy boundaries in one move, because Anthropic's key tells you nothing about OpenAI's text. Convergence here is a feature. Convergence on *keys* would not be, and nobody is proposing that.

---

## 6. The Watermark Answers Exactly One Question

> "Using our key, one can only answer the question "What is the likelihood this was partly written by Claude?""

Read every qualifier. *Likelihood* — a probability, not a verdict. *Partly* — some involvement, extent unspecified. *By Claude* — this key sees Claude and nothing else. It cannot confirm that text was human-written, and it cannot identify a different AI, which may hold a different key or use an entirely different watermarking scheme.

Then there's the length floor:

> "Detecting a watermark also doesn't work well on small samples, where there are fewer word choices and thus less information to go on."

More text, more forks in the road, more confidence. A paragraph or a social post is close to hopeless.

And the sharpest sentence in the whole document:

> "A watermark can only determine that Claude was likely involved with the content at some point. It cannot distinguish "Claude wrote this" from "Claude heavily edited this.""

**My take:** This is the paragraph that should be stapled to every policy built on top of watermark detection. A signal that outputs a probability, tops out at "partly," and cannot separate authorship from editing is not adjudication evidence. Not for academic misconduct panels, not for automated platform enforcement, not for hiring screens. Anthropic draws the line explicitly elsewhere in the document: the watermark says nothing about ownership, authorship, or a user's rights under their terms. Institutions will still try. They should read this section first.

---

## 7. Where There's Only One Right Answer, There's No Watermark

> "Watermarking is sparser on factual passages where there are fewer choices that can be made without decreasing the accuracy of the text."

The example is well chosen. "Isaac Newton's most famous work was called *Principia*…" — the next word being *Mathematica* is a correctness question, not a stylistic one. The watermark has nothing to work with.

Same for "2 + 2 =". No second choice is as good as "4" (unless, as the document notes with a straight face, you're discussing George Orwell's *Nineteen Eighty-Four*, in which case nothing beats "5"). The watermark's nudge simply isn't applied.

Which leads somewhere developers should notice:

> "For the same reason, code—which in very many cases has to be exact—has generally less watermarking than some other forms of text."

The exceptions are the parts of code where wording is genuinely free:

> "Having said that, in areas where there _is_ an arbitrary choice between particular words or terms within the code, the watermark can be used, such as comments within code."

And Anthropic concedes the effect on the actual code produced is negligible.

**My take:** Good news and bad news, same fact. Good: the watermark will never rename a variable or restructure a function to carry a bit. Your generated code is your generated code. Bad: if you were hoping watermark detection could tell you whether a contractor's pull request came out of a model, forget it — and stripping the comments removes what little signal remained. This is structural, not a gap someone can engineer away. **The higher the information density of the text, the thinner the watermark.** That trade-off is baked into the design.

---

## 8. Ask Claude to Proofread and the Watermark Mostly Vanishes

> "The watermark only applies to words Claude chooses."

That single line defines the whole boundary. Hand Claude your draft, ask for grammar and punctuation only, and almost every word coming back is still yours. The watermark can attach to a handful of corrections at most — often too few to register.

> "The more Claude writes, the more decisions it has to make, and the more space there is for a watermark."

On deliberate removal, the answer is refreshingly direct:

> "Light editing probably won't remove the watermark completely; a complete rewrite where every word is replaced will."

Followed by a rhetorical jab I enjoyed:

> "In the latter case, of course, it's arguable whether the text can any longer be described as AI-generated."

**My take:** Publishing your own bypass is the strongest credibility move in the document, and the retort holds up — if you've replaced every single word by hand, the definitional question is live. But don't over-read it into "watermarks are theater." This was never built to beat a motivated adversary. It's built to put a checkable default marker on AI text moving through the world at volume, most of which nobody is trying to launder. It loses the adversarial case, and Anthropic says so on the record rather than letting someone else discover it.

---

## 9. A Key Beats Vibes: Watermarks vs. Detectors Like Pangram

> "AI detection software uses a different method, because the companies that provide it don't have our key."

Anyone who has ever pasted text into an "AI probability" checker should sit with that sentence. Third-party detectors have no key. All they can do is profile style — and Anthropic names two tells outright:

> "For example, AI models appear to be fond of the construction "this isn't [X], it's [Y]", and use the word "quietly" a lot more than you might expect."

The "this isn't X, it's Y" construction, and a strange fondness for "quietly."

**My take:** One approach checks a cryptographic pattern; the other profiles writing habits. Those are not comparable instruments. The false-positive bill for style-based detection has already been paid by real people — non-native English writers with formal register flagged as machines is a recurring, documented failure. A watermark can't convict you for writing like an AI, because it isn't looking at your prose at all. The price is far narrower coverage: Claude only, long text only, probability only.

Also, take those two tells as free editing advice. If your draft opens paragraph after paragraph with "this isn't X, it's Y," that's worth fixing regardless of who wrote it.

---

## 10. Files Get Metadata, Not a Watermark — Plus the Rollout Schedule

Files take an entirely different route. When Claude produces a .png, .jpg, or .svg, it attaches a small cryptographically signed note in the file's metadata saying Claude was involved. That's C2PA, the open industry standard already used by camera manufacturers and photo-editing software to record provenance.

> "This metadata label is very different from a watermark. Nothing in the file changes—it is not embedded or hidden."

**Metadata is not a watermark.** It rides alongside the file rather than inside it — which means, and this is my inference rather than Anthropic's claim, a re-export through almost any tool will drop it. Any C2PA-aware tool can read it, and Anthropic says it will ship a drop-a-file checker of its own.

Three timelines close out the document:

- Detection API: `"We will soon be offering a watermark detection API."` Implementation details still being worked out.
- Older models: Anthropic models launched before August 2, 2026 fall under an EU transition period, with watermarking added over the coming months.
- Translations: `"Yes. A translation produced by Claude carries a watermark, because in this case every word is chosen by Claude."`

**My take:** Translation is the sleeper item on this list. Most people don't file "run this through Claude" under AI-generated content — but by the mechanism described here, every word in the output was selected by the model, which puts translations at the *high* end of watermark density. Write an article yourself, translate it with Claude, and the translated version is as thoroughly marked as anything Claude wrote from scratch. That gap between intuition and mechanism is where I'd expect the first real disputes to land.

---

## The Part That Actually Matters

The most credible thing about this document isn't the explanation of how watermarking works. It's that roughly half the text is devoted to when it *doesn't*: short samples, factual passages, code, proofreading, rewrites.

A signal that can be evaded, only ever returns a probability, and can't separate writing from editing is still genuinely useful — as long as everyone understands its edges. **The damage won't come from the technology. It'll come from the first institution that reads "likely involved" as "proven."**

Source: [How Claude's text watermarking works](https://www.anthropic.com/news/claude-text-watermark)
