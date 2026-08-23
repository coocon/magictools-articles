# Claude XML Tags Guide: Structure Prompts for Better Output

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-xml-tags-guide-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-xml-tags-guide-en?utm_source=github&utm_medium=referral)**

XML tags are one of the most useful structure tools in Anthropic's prompt engineering guidance. They help Claude separate instructions, examples, context, and output constraints so the model does not mix them together.

The main advantage is not flashy. It is reliability. When your prompt has several moving parts, a clear structure makes it much easier for Claude to interpret the request correctly.

## Why XML tags help

Use XML tags when a prompt has multiple sections or when you want predictable parsing:

- Separate instructions from source material
- Mark examples clearly
- Isolate output format requirements
- Make complex prompts easier to edit later

You do not need exotic tag names. Use names that match the content and stay consistent across prompts.

## A simple pattern

```text
<instructions>
Rewrite the note for a leadership audience.
</instructions>

<context>
This is a weekly project update for executives.
</context>

<output_format>
Use 3 bullets: status, risk, next step.
</output_format>
```

That structure gives Claude a clean map of the task. It also helps you read the prompt later and see what each part does.

...

---

**[👉 Continue reading: Claude XML Tags Guide: Structure Prompts for Better Output](https://tools.cooconsbit.com/en/articles/claude-xml-tags-guide-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
