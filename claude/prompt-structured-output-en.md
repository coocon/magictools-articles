# Structured Output and Multimodal: Formatted Responses and Vision

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/prompt-structured-output-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/prompt-structured-output-en?utm_source=github&utm_medium=referral)**

## Getting JSON Output

The most reliable way to get structured JSON from Claude is to provide a clear schema:

```
Analyze the sentiment of the following user review and output in JSON format:
{
  "sentiment": "positive | negative | neutral",
  "confidence": 0.0-1.0,
  "keywords": ["keyword array"],
  "summary": "one-sentence summary"
}

Review: The steak at this restaurant was excellent, but the wait time was way too long and the service was mediocre.
```

Claude will return results precisely matching the schema.

## Prefilling to Guarantee Format

When using the API, prefilling the assistant response guarantees output format with 100% reliability:

```python
messages = [
    {"role": "user", "content": "Analyze the sentiment of this text"},
    {"role": "assistant", "content": "{"}  # prefill
]
```

Claude continues from `{`, ensuring pure JSON output without preambles like "Sure, here is the analysis:".

## XML Tags for Structured Sections

For complex outputs with multiple sections, XML tags are an excellent structuring tool:

```
Output the code review results in the following XML format:

<review>
  <issues>
    <issue severity="high|medium|low">
      <description>Issue description</description>
      <location>File and line number</location>
      <fix>Suggested fix</fix>
    </issue>
  </issues>
  <summary>Overall assessment</summary>
  <score>1-10</score>
</review>
```

...

---

**[👉 Continue reading: Structured Output and Multimodal: Formatted Responses and Vision](https://tools.cooconsbit.com/en/articles/prompt-structured-output-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
