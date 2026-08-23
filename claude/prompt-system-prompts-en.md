# System Prompt Design: Make Claude Understand Exactly What You Need

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/prompt-system-prompts-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/prompt-system-prompts-en?utm_source=github&utm_medium=referral)**

## System Prompts vs User Messages

In Claude's API, messages exist at two levels: the system prompt and user messages. The system prompt defines Claude's "identity" and "rules of behavior" — it stays active throughout the entire conversation. User messages contain specific task requests.

Placing stable instructions in the system prompt and variable content in user messages is the key pattern for using Claude effectively.

## Role Definition Patterns

Role definition is the most powerful technique in system prompts. It sets not only Claude's domain expertise but also the implied depth and style of responses.

**Basic role definition:**

```
You are a data analyst with 10 years of experience, skilled at explaining data insights in plain language to non-technical stakeholders.
```

**Advanced role definition with behavioral constraints:**

```
You are a meticulous legal advisor.
- Always base answers on current applicable laws
- If a question falls outside your certain knowledge, clearly advise the user to consult a licensed attorney
- Never make legal commitments or guarantees
- Cite specific statutes with references when applicable
```

## Output Format Control

System prompts are ideal for defining fixed output formats. Here is a practical JSON output example:

...

---

**[👉 Continue reading: System Prompt Design: Make Claude Understand Exactly What You Need](https://tools.cooconsbit.com/en/articles/prompt-system-prompts-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
