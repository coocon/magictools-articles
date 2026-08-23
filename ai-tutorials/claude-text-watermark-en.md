# Claude's Text Watermark: A Signature Hidden in the Dice Rolls

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-text-watermark-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-text-watermark-en?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 Continue reading: Claude's Text Watermark: A Signature Hidden in the Dice Rolls](https://tools.cooconsbit.com/en/articles/claude-text-watermark-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
