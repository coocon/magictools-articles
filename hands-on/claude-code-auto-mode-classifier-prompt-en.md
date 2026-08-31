# Cracking Open Claude Code's Auto-Mode Classifier: A 116K-Char System Prompt, Dissected Line by Line

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-auto-mode-classifier-prompt-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-auto-mode-classifier-prompt-en?utm_source=github&utm_medium=referral)**

My [earlier retest](/en/articles/claude-code-auto-mode-permission-trap-en) confirmed one thing: before auto mode runs a Bash command, it makes an extra call to **the very session model you're using**, asking it to judge whether that command is safe. Back then I captured the outer shell of the classifier request — `model`, `max_tokens`, message count — but the intimidatingly long system prompt inside the request body, I only quoted its first line, `You are a security monitor for autonomous AI coding agents`, without unpacking it.

This article lays that whole prompt out. It's a **116,879-character** security-policy spec that dictates what the classifier must block, what it must allow, and in what order it decides. Every dissection below quotes the capture directly — no paraphrasing.

## The Problem: the classifier is a black box, but its rules define auto mode's real boundary

Auto mode's pitch is "one model call in exchange for dozens of manual confirmations." But that sentence dodges a question: by **what standard** does that model call allow or block?

The official docs only say auto mode "evaluates command safety" — the evaluation rules aren't published. So a user's mental model of auto mode's boundary is guesswork: will `rm -rf` get blocked? What about `curl | bash`? Editing `.claude/settings.json`? Without knowing the rules, you can't predict which command will interrupt you, nor judge what this layer of protection actually stops versus lets through.

Turning the black box white has exactly one path: obtain the complete input the classifier receives. And the classifier request is just an ordinary HTTP call from Claude Code to the model API — intercept that request body and the rules are all right there.

## Analysis: the guard path is two independent model calls

First, the call structure. In auto mode, a Bash command under the classifier's watch triggers **two independent model-call chains**:

1. **Main loop**: reads your message, decides which tool to call, generates the reply — the model doing the work.
2. **Safety classification**: after the main loop decides to run some Bash, and before it actually runs, **a separate request** asks "is this command dangerous?"

The classifier request and the main-loop request hit the same model ID, but their shapes differ sharply (both from this session's `bodies/7.json`):

| | Classifier request | Main-loop request |
|---|---|---|
| `model` | `claude-sonnet-5` | `claude-sonnet-5` |
| System prompt length | **116,879 chars** | 27,702 chars |
| `max_tokens` | **64** | normal |
| `tools` | **none** (field absent) | 23 |
| `thinking` | `disabled` | normal |
| `stop_sequences` | `["</severity>"]` | none |

...

---

**[👉 Continue reading: Cracking Open Claude Code's Auto-Mode Classifier: A 116K-Char System Prompt, Dissected Line by Line](https://tools.cooconsbit.com/en/articles/claude-code-auto-mode-classifier-prompt-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
