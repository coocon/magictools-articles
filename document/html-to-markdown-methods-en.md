---
title: "How to Convert HTML to Markdown: 3 Easy Methods (With Examples)"
slug: "html-to-markdown-methods-en"
category: document
tags:
  - html
  - markdown
  - conversion
  - web scraping
summary: "Learn three practical methods to convert HTML to Markdown using an online tool — paste raw HTML code, fetch a URL automatically, or paste rich text from any webpage — with real-world examples and troubleshooting tips."
coverImage: ""
status: published
scheduledAt: ""
---

Every developer who has migrated a website, scraped content for documentation, or tried to reuse existing web content in a static site generator has faced the same problem: the source is HTML, but the destination expects Markdown.

Hand-converting HTML to Markdown is tedious, error-prone, and frankly unnecessary. The [HTML to Markdown converter](/tools/html2md) at MagicTools handles the transformation automatically — and it offers three distinct input methods to match how you actually work.

## Why Convert HTML to Markdown?

Before diving into the how, it's worth being clear on the why. Here are the most common scenarios:

**Static site migration.** Moving from a WordPress or Drupal site to a static site generator like Hugo or Jekyll requires converting stored HTML content into Markdown files. Doing this manually for hundreds of posts is not realistic.

**Content reuse and repurposing.** Copying web content for internal documentation, training datasets, or knowledge bases is much cleaner in Markdown than in raw HTML.

**Version control for content.** Markdown diffs meaningfully in Git — HTML diffs are often impossible to read due to generated attributes, inline styles, and auto-closed tags.

**Cleaning up pasted content.** When writers paste content from the web into CMS editors, they bring invisible HTML baggage — `<span>` wrappers, inline styles, and Microsoft Word-specific tags. Converting to Markdown strips all of that.

## Method 1: Paste HTML Code Directly

This is the most direct method. Copy any HTML snippet from your source, paste it into the input area, and the tool converts it instantly.

**Before (HTML):**
```html
<h2>Getting Started</h2>
<p>Install the package using <strong>npm</strong>:</p>
<pre><code class="language-bash">npm install my-package --save</code></pre>
<ul>
  <li>Requires Node.js 18+</li>
  <li>Works on Linux, macOS, and Windows</li>
</ul>
```

**After (Markdown):**
```markdown
## Getting Started

Install the package using **npm**:

```bash
npm install my-package --save
```

- Requires Node.js 18+
- Works on Linux, macOS, and Windows
```

Notice what the converter handles automatically: the `<strong>` tag becomes `**bold**`, the `<pre><code>` block becomes a fenced code block with the language preserved, and the `<ul><li>` structure becomes a simple dash list.

**Best used for:** Converting snippets from HTML templates, CMS exports, email HTML, or documentation source files.

## Method 2: Enter a URL — Automatic Fetch and Convert

Enter any publicly accessible URL into the input field and click fetch. The tool sends the request from a server-side proxy (not your browser), retrieves the page HTML, strips navigation, headers, and footers using content-extraction heuristics, and converts the main body content to Markdown.

For example, entering a Wikipedia article URL returns the article body in clean Markdown — headings, links, and lists intact — without the sidebar, navigation, or cookie banners.

**Technical details worth knowing:**

- The proxy has a **5MB response size limit** — sufficient for nearly all text-based pages
- **10-second timeout** — pages that respond slowly will fail with a timeout error
- JavaScript-rendered content (single-page apps) may not work correctly, because the proxy fetches the initial HTML response, not the fully rendered DOM
- The conversion uses the [Turndown](https://github.com/mixmark-io/turndown) library under the hood

**Best used for:** Blog posts, Wikipedia articles, documentation pages, product pages with text-heavy content, news articles.

## Method 3: Paste Rich Text from Your Browser

This method is less obvious but often the most convenient. In your browser, select text on any webpage (including formatted text with bold, headings, and lists), copy it (Ctrl+C / Cmd+C), then click inside the rich text input area and paste.

The tool receives the HTML representation of your clipboard content (which browsers provide automatically when copying formatted text) and converts it to Markdown.

**Why this works:** When you copy text from a webpage, your clipboard stores both a plain text version and an HTML version. The rich text input reads the HTML version, preserving formatting that the plain text version would lose.

**Best used for:** Copying specific sections from web pages without loading the full URL, capturing formatted content from web apps that block URL access, or quickly extracting a few paragraphs from a long page.

## Common Conversion Gotchas

No automatic converter is perfect. Here are the most frequent edge cases and how to handle them.

### Tables

HTML tables convert reasonably well for simple cases:

```html
<table>
  <thead><tr><th>Name</th><th>Age</th></tr></thead>
  <tbody><tr><td>Alice</td><td>30</td></tr></tbody>
</table>
```

Converts to:

```markdown
| Name  | Age |
|-------|-----|
| Alice | 30  |
```

However, tables with `rowspan`, `colspan`, or nested tables cannot be represented in standard Markdown. These will either be simplified (losing merged cell information) or converted to a best-effort approximation. Check complex tables manually after conversion.

### Image Alt Text

Images convert to `![alt](src)` syntax, but many websites use empty or auto-generated alt text. After conversion, you'll often need to manually improve alt text for accessibility and SEO.

### Nested Lists

Most nested list structures convert correctly, but deeply nested content (4+ levels) can sometimes produce inconsistent indentation. The Markdown spec allows 2 or 4 spaces for nesting — the converter uses 2 spaces, which not all renderers handle identically.

### Special Characters

HTML entities like `&amp;`, `&lt;`, and `&nbsp;` are decoded correctly. However, `&nbsp;` (non-breaking space) sometimes causes unexpected whitespace in the Markdown output. If you see strange spacing, search for `\u00a0` characters and replace with regular spaces.

## Real-World Use Case: WordPress to Hugo Migration

Suppose you're migrating a WordPress blog with 80 posts to Hugo. WordPress exports content as HTML stored in a database. Here's the workflow:

1. Export your WordPress content as XML (Tools → Export in WP Admin)
2. Extract HTML content from the XML (using a script or WP-CLI)
3. For each post, paste the HTML body into the converter and copy the Markdown output
4. Create a Hugo content file with the correct frontmatter and paste the Markdown body
5. Fix any tables, complex images, or shortcodes that didn't convert cleanly

For very large migrations, this workflow can be scripted using the Turndown library directly in Node.js — the online tool is most useful for reviewing and spot-checking individual posts before committing to a full automated conversion.

## Pro Tips

**Strip `<div>` wrappers before converting.** Many CMS editors wrap content in meaningless `<div>` containers with class names. These convert to nothing in Markdown but can confuse the parser. Strip outer wrappers manually or with a quick regex before pasting.

**Validate output before using it.** Paste the Markdown output back into a Markdown preview (like the [Markdown editor](/tools/markdown)) to verify the rendering looks correct before committing to a file.

**For URL fetching, try the article URL not the homepage.** The content extractor works best on single-article pages. Homepages and category pages contain mixed content that's harder to extract cleanly.

## Frequently Asked Questions

**What if URL fetching fails with a timeout or error?**

This usually happens because the target site uses JavaScript rendering (the initial HTML response is just a shell with no content), the server responded slowly, or the site blocks server-side requests. In these cases, use Method 3: open the page in your browser, select the content you want, copy it, and paste it using the rich text input.

**How do I handle conversion errors where the output looks garbled?**

If the output contains repeated tags, escaped characters, or broken structure, the source HTML is likely malformed or deeply nested in ways the parser can't handle. Try pasting a smaller, cleaner subset of the HTML. If the source HTML uses non-standard attributes or vendor-specific tags (common in email HTML), strip those first using an HTML cleaner before converting.

**Does the converter preserve links?**

Yes. `<a href="https://example.com">link text</a>` converts correctly to `[link text](https://example.com)`. Relative links (e.g., `/about`) are preserved as-is — they won't be converted to absolute URLs, so check those if you're moving content to a different domain.

**Can it handle HTML with inline styles?**

Inline styles (`style="color: red;"`) are stripped during conversion — Markdown has no way to represent arbitrary CSS. If you need to preserve some styling, you can embed raw HTML inside Markdown (most Markdown renderers support it), but this defeats the purpose of converting to Markdown in the first place.

## Conclusion

Converting HTML to Markdown doesn't have to be a manual, error-prone process. With three flexible input methods — direct code paste, URL fetch, and rich text clipboard — the converter covers the vast majority of real-world scenarios.

Start with the method that matches your source: paste the code if you have it, fetch the URL if the page is public, or copy-paste from the browser if you just need a section. Then use the Markdown preview to verify the output before saving or committing.

For large-scale migrations, the online tool works best as a validation tool alongside a scripted conversion pipeline using Turndown directly.
