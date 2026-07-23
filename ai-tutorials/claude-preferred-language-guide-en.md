---
title: "Using Claude in Your Preferred Language: A Practical Workflow"
slug: "claude-preferred-language-guide-en"
category: ai-tutorials
tags:
  - claude
  - anthropic
  - language
  - localization
summary: "How to use Claude across languages, adjust the interface language, and write prompts that make bilingual workflows smoother."
coverImage: ""
status: published
scheduledAt: ""
---

Claude can work in more than one language, but the best results still come from being explicit about what language you want and where language switching should happen. Anthropic's help center notes that Claude is available in several interface languages and can still converse in the language you use, even if the app UI is set differently.

That distinction matters. The interface language controls the app experience. The prompt language controls the output. If you need bilingual workflows, treat them as separate decisions.

## Set the interface language first

If you want the Claude app itself to feel local, change the interface language in your account settings. Anthropic's help center says supported web and desktop languages include English, French, German, Hindi, Indonesian, Italian, Japanese, Korean, Portuguese, and Spanish variants.

Changing the interface language is useful when:

- Your team prefers a non-English UI
- You are onboarding non-English speakers
- You want menus and settings to match the user's comfort level

But changing the UI does not force Claude to answer in that same language.

## Tell Claude the output language explicitly

If you want a response in Chinese, Japanese, English, or any other supported language, say so in the prompt. Do not rely on the interface language alone.

Examples:

```text
Summarize this meeting note in English, even if the source text is in Chinese.
```

```text
请用中文回答，并保留英文产品名不翻译。
```

```text
Translate this draft into Spanish for a customer-facing email. Keep the tone polite and concise.
```

The more important the language choice is, the more explicit you should be.

## Use Claude as a bilingual workflow tool

Claude is especially useful when one language is the source and another is the delivery format. A few practical patterns:

- Translate customer support replies before sending them
- Draft documentation in English, then localize it
- Summarize interviews or notes in the language your team actually uses
- Rewrite mixed-language drafts so the final version is consistent

When translating, include the audience and desired tone, not just the target language.

## Prompt patterns that work well

Use prompts that define both language and behavior:

```text
Rewrite this announcement in English for a global product team.
Keep the meaning intact.
Use simple wording.
Do not over-localize brand names.
```

```text
请把下面这段内容改写成自然的中文产品公告。
受众是普通用户，不是内部员工。
语气要友好、清晰、简洁。
```

If the output feels unnatural, add one more constraint: whether to preserve names, whether to localize dates, and whether to keep technical terms in English.

## Common mistakes

- Assuming the UI language changes the answer language
- Asking for translation without naming the audience
- Mixing too many languages in one prompt without instructions
- Forgetting that some languages may still require extra editing for tone or terminology

Claude can handle multilingual work well, but it is still worth reviewing the final copy, especially for customer-facing text.

## Practical checklist

1. Set the UI language if you want the interface localized.
2. State the target language in the prompt.
3. Specify audience and tone.
4. Tell Claude what to preserve and what to translate.
5. Review the result before publishing.

That small amount of structure is usually enough to make bilingual work much smoother.

## Official References

- [Using Claude in Your Preferred Language](https://support.anthropic.com/en/articles/10769299-using-claude-in-your-preferred-language)
- [Can I use Claude in different languages?](https://support.anthropic.com/en/articles/7996851-can-i-use-claude-in-different-languages)
- [Getting started with Claude](https://support.anthropic.com/en/articles/8114491-how-do-i-get-started-with-claude-ai)
- [What are some things I can use Claude for?](https://support.anthropic.com/en/articles/7996845-what-are-some-things-i-can-use-claude-for)

Sources reviewed on March 29, 2026. Interface language support and regional availability can change, so confirm current behavior in the linked official Anthropic resources.
