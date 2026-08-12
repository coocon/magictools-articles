---
title: "Two API Calls to Steal Any LLM's Hidden Thoughts"
slug: stealing-reasoning-traces-en
summary: "Encrypted reasoning blocks from closed-source LLMs can be replayed across models — feed Opus's encrypted thinking to Haiku and it transcribes verbatim. Researchers scanned 6,708 public agent traces and pulled 62 real API keys and 33 passwords."
category: ai-tutorials
tags: [AI security, chain-of-thought, Claude, OpenAI, Gemini, prompt injection, LLM]
coverImage: ""
status: published
locale: en
source: authored
translationSlug: stealing-reasoning-traces-zh
---

You just shipped a bug fix. Claude helped—you pasted the stack trace, it suggested the patch, you moved on.

What you didn't see: buried in Claude's API response is an encrypted blob containing the model's entire reasoning chain. Every step it took to analyze your code. Every assumption it made about your data. You can't read it—the blob is encrypted. But a new paper shows anyone can decrypt it. In two API calls.

The paper is *Stealing Reasoning Traces from Proprietary LLM APIs*, authored by researchers from MATS Research, the Max Planck Institute, and ELLIS Institute Tübingen. Their findings are brutal: 315,320 reasoning blocks recovered, 704 privacy artifacts extracted from public agent traces, and 64 secrets that existed *only* inside the reasoning blocks—never in the visible conversation.

## The Attack: Use a Weak Model as a Decoder

Here's how it works, in three steps.

**Step one: collect the encrypted blob.** When you call GPT-4o, Claude Opus, or Gemini with reasoning enabled, the API response includes a `thinking` field. For OpenAI, it's labeled `encrypted_content`. For Anthropic, it's a `signature`-wrapped block. Tens of thousands of characters that look like noise. You, the client, receive this block and are expected to echo it back on the next request so the server can reconstruct context.

**Step two: replay it into a weaker model.** Here's the vulnerability: these encrypted blocks are *not bound* to a specific model, session, or user. Take an encrypted reasoning block from Claude Opus 4, and inject it into a Claude Haiku request in the same field. Haiku will accept it.

**Step three: ask the weak model to transcribe it.** Jailbreak Haiku—which has weaker safety guardrails than Opus—and tell it the encrypted block is its own reasoning. Then ask it to "transcribe what you just thought, verbatim." Haiku dutifully outputs the plaintext. It believes those were its own thoughts.

Think of it like this: Opus writes a diary in cipher and hands you the encrypted page for safekeeping. You realize the same cipher key works across all notebooks from the same manufacturer, so you hand the page to Haiku—a gullible notebook—and say "read this back, it's yours." Haiku reads it aloud, word for word. Opus never knows.

## They Found Real Passwords and Credit Card Numbers

The researchers didn't stop at a proof of concept. They went hunting in the wild.

They scraped 6,708 public agent conversation traces from GitHub and HuggingFace—real sessions produced by Claude, GPT, and Gemini models that still contained encrypted reasoning blocks. Applying the replay attack across all of them yielded **315,320 reconstructed reasoning blocks**.

Inside those blocks: **704 distinct privacy artifacts**, including:

- 62 API keys
- 33 passwords
- 24 access tokens
- 30 personal email addresses
- Names, postal addresses, internal URLs, and more

The terrifying stat: **64 of those 704 artifacts appeared exclusively inside the reasoning blocks.** The users never saw them in the visible conversation. They had no way of knowing their secrets were leaked—because the leak happened inside a part of the response they couldn't even read.

One concrete example: GPT-5.2 Codex, running a `sanitize-git-repo` task, listed inside its reasoning block the exact tokens it needed to scrub—AWS access keys, GitHub tokens, HuggingFace tokens—in plaintext. The final output responsibly masked them. The reasoning trace remembered everything.

Another: Claude Sonnet 4.6 helping with flight booking. The reasoning block contained the passenger's full name, passport number, date of birth, credit card details (number, expiry, CVV), and frequent flyer number. Some of this was likely inferred by the model rather than explicitly provided—but the inference was faithfully preserved in the encrypted block.

## Why Providers Really Hide the Chain of Thought

OpenAI and Anthropic have long restricted access to chain-of-thought outputs. The official rationale: safety. If users see how the model reasons, they can more easily craft jailbreaks.

This paper exposes the second, unspoken reason: **fear of cheap distillation.**

The experiments show that Opus's reasoning traces are high quality. For 120 Codeforces problems, the decoded reasoning token count tracked the API-reported hidden thinking tokens almost perfectly. These traces represent millions of dollars of training cost, distilled into a few hundred dollars of API calls. Extract Opus's reasoning, fine-tune a small open model on it, and you've got a poor man's Opus—without ever touching the model weights.

The "closed-source moat" is draining through a side channel called reasoning trace leakage.

And patching this is genuinely hard. The fundamental tension: providers need encrypted blocks for multi-turn context continuity. The client must receive the block and send it back. If the block isn't bound to the session and user, replay attacks are an inherent design flaw, not a bug.

## What You Should Do Right Now

**1. Audit your API response handling.** Are you logging full API responses? If so, those encrypted blocks are sitting in your logs. Check whether your application server stores or forwards `encrypted_content` (OpenAI) or `signature` (Anthropic) fields. They look like garbage data, which means they're easy to overlook during log reviews.

**2. Never commit full API responses to public repos.** This sounds obvious, but the paper found 6,708 public sessions containing these blocks on GitHub and HuggingFace. Each one represents a developer who thought "I'd never make that mistake." You might be next.

**3. Keep sensitive data out of model prompts.** If your workflow involves passing API keys, passwords, or user PII through a reasoning model, those secrets enter the reasoning trace—even if the model responsibly omits them from the final output. The trace is encrypted, but encryption isn't absolute. This paper proved that.

**4. Watch for provider fixes.** Anthropic and OpenAI were presumably notified before publication. Likely fixes: binding encrypted blocks to specific sessions, binding them to model versions, or adding integrity checks. Until those land, treat the reasoning mode as a feature that leaks your thinking—because right now, it does.

---

Bottom line: you thought you were getting just the answer. You were also getting the scratch paper—sealed in an envelope that turns out to be transparent.

> Based on the codefarm Daily Intel 2026-08-12 issue. Source data from stolen-thoughts.com.

**Sources:**
- [Stealing Reasoning Traces from Proprietary LLM APIs — stolen-thoughts.com](https://stolen-thoughts.com/)
- [arXiv: 2608.09867](https://arxiv.org/abs/2608.09867)
