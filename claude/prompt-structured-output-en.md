---
title: "Structured Output and Multimodal: Formatted Responses and Vision"
slug: prompt-structured-output-en
category: claude
tags: [prompt-engineering, structured-output, multimodal, vision, advanced]
summary: "Learn to get Claude to output structured formats like JSON and XML, and leverage Claude's vision capabilities for understanding images, documents, and charts."
status: published
---

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

XML tags support nesting and attributes, making them more flexible than JSON for expressing hierarchical relationships.

## Handling Edge Cases

Prevent common issues in structured output with explicit constraints:

```
Output requirements:
- Always return valid JSON, even if input data is unusual
- Use null for fields that cannot be analyzed, not empty strings
- Return [] for empty arrays, do not omit the field
- Do not add any explanatory text outside the JSON
```

These constraints ensure your code can reliably parse Claude's output.

## Multimodal: Image Understanding

Claude has powerful vision capabilities. When sending images via API, combine them with text prompts for analysis:

```
Analyze the UI in this screenshot:
1. List all visible UI components
2. Identify design guideline violations
3. Provide improvement suggestions

Output as a JSON array with component, issue, and suggestion fields per item.
```

### Common Image Analysis Scenarios

| Scenario | Prompt Example |
|----------|---------------|
| OCR text extraction | "Extract all text from the image, preserving layout" |
| Chart data reading | "Read data from this bar chart and output as a table" |
| UI description | "Describe the layout structure and interactive elements" |
| Document parsing | "Extract key info from this invoice: date, amount, vendor" |

## Combining Vision with Structured Output

Merging vision capabilities with structured output is the most powerful application pattern:

```
Analyze this product screenshot and output in JSON format:
{
  "product_name": "name",
  "price": "price",
  "rating": "rating",
  "key_features": ["feature list"],
  "visible_issues": ["UI issues"]
}
```

This combination is ideal for building automated data extraction pipelines.

## Frequently Asked Questions

### Can Claude guarantee valid JSON output 100% of the time?

In most cases yes, especially with the prefilling technique. However, in edge cases such as very long outputs being truncated, JSON may be incomplete. Always wrap JSON parsing in try-catch blocks in your code and implement retry logic.

### What image types can Claude process?

Claude supports JPEG, PNG, GIF, and WebP formats. It can understand photos, screenshots, charts, scanned documents, and handwritten notes. Accuracy decreases with very small text, blurry images, or highly abstract artwork.

### Is structured output better than free-form output?

It depends on the use case. If output needs to be parsed by code (API integrations, data pipelines), always use structured output. If the content is meant for human reading (articles, emails, reports), free-form is usually more natural. You can also combine both — have Claude place free-form text within specific JSON fields.

### Are there limitations when sending images?

A single API request can include up to 20 images. Images consume token quota, with higher resolution images using more tokens. It is recommended to compress images before sending — keep key details clear without requiring ultra-high resolution.
