# Two API Calls to Steal Any LLM's Hidden Thoughts

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/stealing-reasoning-traces-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/stealing-reasoning-traces-en?utm_source=github&utm_medium=referral)**

You just shipped a bug fix. Claude helped—you pasted the stack trace, it suggested the patch, you moved on.

What you didn't see: buried in Claude's API response is an encrypted blob containing the model's entire reasoning chain. Every step it took to analyze your code. Every assumption it made about your data. You can't read it—the blob is encrypted. But a new paper shows anyone can decrypt it. In two API calls.

The paper is *Stealing Reasoning Traces from Proprietary LLM APIs*, authored by researchers from MATS Research, the Max Planck Institute, and ELLIS Institute Tübingen. Their findings are brutal: 315,320 reasoning blocks recovered, 704 privacy artifacts extracted from public agent traces, and 64 secrets that existed *only* inside the reasoning blocks—never in the visible conversation.

## The Attack: Use a Weak Model as a Decoder

Here's how it works, in three steps.

**Step one: collect the encrypted blob.** When you call GPT-4o, Claude Opus, or Gemini with reasoning enabled, the API response includes a `thinking` field. For OpenAI, it's labeled `encrypted_content`. For Anthropic, it's a `signature`-wrapped block. Tens of thousands of characters that look like noise. You, the client, receive this block and are expected to echo it back on the next request so the server can reconstruct context.

**Step two: replay it into a weaker model.** Here's the vulnerability: these encrypted blocks are *not bound* to a specific model, session, or user. Take an encrypted reasoning block from Claude Opus 4, and inject it into a Claude Haiku request in the same field. Haiku will accept it.

**Step three: ask the weak model to transcribe it.** Jailbreak Haiku—which has weaker safety guardrails than Opus—and tell it the encrypted block is its own reasoning. Then ask it to "transcribe what you just thought, verbatim." Haiku dutifully outputs the plaintext. It believes those were its own thoughts.

Think of it like this: Opus writes a diary in cipher and hands you the encrypted page for safekeeping. You realize the same cipher key works across all notebooks from the same manufacturer, so you hand the page to Haiku—a gullible notebook—and say "read this back, it's yours." Haiku reads it aloud, word for word. Opus never knows.

...

---

**[👉 Continue reading: Two API Calls to Steal Any LLM's Hidden Thoughts](https://tools.cooconsbit.com/en/articles/stealing-reasoning-traces-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
