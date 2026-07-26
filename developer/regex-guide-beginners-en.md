---
title: "Regular Expressions for Beginners: 10 Practical Examples You'll Actually Use"
slug: "regex-guide-beginners-en"
locale: en
category: developer
tags:
  - regex
  - regular expressions
  - programming
  - text processing
summary: "Regular expressions look intimidating at first glance, but mastering a small core syntax unlocks powerful text matching in every programming language and editor — this guide teaches the essentials through 10 real-world patterns you will reach for repeatedly."
coverImage: ""
status: published
scheduledAt: ""
---

## Introduction

The first time most developers encounter a regular expression, the reaction is the same: what is this line noise? A pattern like `^[\w.-]+@[\w-]+\.[a-zA-Z]{2,}$` looks more like a cat walked across the keyboard than a coherent instruction.

The reality is that regex has a small, learnable core. Once you internalize about 20 syntax elements, you can read and write patterns confidently. And the payoff is enormous: regex is supported in every major programming language, in VS Code, Vim, grep, sed, SQL `LIKE` patterns, spreadsheet formulas, and even browser DevTools. It is the closest thing to a universal text-processing superpower.

This guide skips the theory and focuses on 10 practical patterns with full explanations, test strings, and notes on language differences.

---

## Quick Reference: Core Syntax

### Character Classes

| Syntax | Matches |
|---|---|
| `.` | Any character except newline |
| `\d` | Any digit (0–9) |
| `\D` | Any non-digit |
| `\w` | Word character (a–z, A–Z, 0–9, underscore) |
| `\W` | Non-word character |
| `\s` | Whitespace (space, tab, newline) |
| `\S` | Non-whitespace |
| `[abc]` | Any of: a, b, or c |
| `[^abc]` | Anything except a, b, or c |
| `[a-z]` | Any lowercase letter |
| `[a-zA-Z0-9]` | Alphanumeric |

### Quantifiers

| Syntax | Meaning |
|---|---|
| `*` | Zero or more |
| `+` | One or more |
| `?` | Zero or one (optional) |
| `{n}` | Exactly n times |
| `{n,m}` | Between n and m times |
| `{n,}` | n or more times |

### Anchors and Boundaries

| Syntax | Meaning |
|---|---|
| `^` | Start of string (or line with multiline flag) |
| `$` | End of string (or line with multiline flag) |
| `\b` | Word boundary |
| `\B` | Non-word boundary |

### Groups and Alternation

| Syntax | Meaning |
|---|---|
| `(abc)` | Capturing group |
| `(?:abc)` | Non-capturing group |
| `(?<name>abc)` | Named capturing group |
| `a\|b` | Alternation: a or b |

---

## 10 Practical Regex Examples

### 1. Email Validation

```
Pattern: ^[\w.+-]+@[\w-]+\.[a-zA-Z]{2,}$
```

**Explanation:**
- `^[\w.+-]+` — one or more word chars, dots, plus, or hyphen (local part)
- `@` — literal @ symbol
- `[\w-]+` — domain name (alphanumeric and hyphens)
- `\.` — literal dot
- `[a-zA-Z]{2,}$` — TLD of 2 or more letters

**Matches:** `alice@example.com`, `user.name+filter@sub.domain.org`
**Does not match:** `@example.com`, `user@`, `user@.com`

*Note: True RFC 5322 email validation is extraordinarily complex. This pattern catches 99% of real-world cases without false positives.*

---

### 2. URL Matching (HTTP/HTTPS)

```
Pattern: https?://[\w.-]+(?:\.[a-zA-Z]{2,})(?:/[\w./?=%&-]*)?
```

**Explanation:**
- `https?://` — http or https (the `s?` makes the `s` optional)
- `[\w.-]+` — domain name characters
- `(?:\.[a-zA-Z]{2,})` — TLD (non-capturing group)
- `(?:/[\w./?=%&-]*)?` — optional path, query string

**Matches:** `https://example.com`, `http://sub.domain.org/path?q=1&p=2`

---

### 3. US Phone Numbers

```
Pattern: (?:\+1[-.\s]?)?\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}
```

**Matches:** `(555) 867-5309`, `555.867.5309`, `+1-555-867-5309`, `5558675309`

This pattern accommodates the many formats Americans actually use: parentheses around area code, hyphens, dots, spaces, and international prefix.

---

### 4. Replace Multiple Spaces with Single Space

```
Pattern: \s+
Replace with: (single space)
```

**Example:**
```javascript
const cleaned = "Hello    world   from  regex".replace(/\s+/g, ' ');
// Result: "Hello world from regex"
```

`\s+` matches one or more whitespace characters (spaces, tabs, newlines). The `g` flag makes the replacement global across the entire string.

---

### 5. Extract All Numbers from Text

```
Pattern: -?\d+(?:\.\d+)?
```

**Explanation:**
- `-?` — optional negative sign
- `\d+` — one or more digits
- `(?:\.\d+)?` — optional decimal portion

**Example:**
```python
import re
text = "The temperature is -12.5°C and altitude 3500m"
numbers = re.findall(r'-?\d+(?:\.\d+)?', text)
# Result: ['-12.5', '3500']
```

---

### 6. Validate Date Format (YYYY-MM-DD)

```
Pattern: ^\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12]\d|3[01])$
```

**Explanation:**
- `\d{4}` — four-digit year
- `(0[1-9]|1[0-2])` — month 01–12 (not 00 or 13)
- `(0[1-9]|[12]\d|3[01])` — day 01–31

**Matches:** `2024-01-15`, `2000-12-31`
**Does not match:** `2024-13-01`, `2024-1-5`, `24-01-15`

*Note: Regex cannot validate that February 30 is invalid — use date library parsing for semantic validation.*

---

### 7. Password Strength Check

```
Pattern: ^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).{8,}$
```

**Explanation:**
- `(?=.*[a-z])` — lookahead: must contain at least one lowercase letter
- `(?=.*[A-Z])` — lookahead: must contain at least one uppercase letter
- `(?=.*\d)` — lookahead: must contain at least one digit
- `.{8,}` — minimum 8 characters total

**Example:**
```javascript
const isStrong = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).{8,}$/.test(password);
```

Lookaheads (`(?=...)`) are zero-width assertions — they check without consuming characters, allowing multiple independent conditions on the same string.

---

### 8. Match IPv4 Addresses

```
Pattern: \b(?:(?:25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(?:25[0-5]|2[0-4]\d|[01]?\d\d?)\b
```

**Explanation:** Each octet is constrained to 0–255:
- `25[0-5]` — 250–255
- `2[0-4]\d` — 200–249
- `[01]?\d\d?` — 0–199

**Matches:** `192.168.1.1`, `0.0.0.0`, `255.255.255.255`
**Does not match:** `256.0.0.1`, `192.168.1` (incomplete)

---

### 9. Extract HTML Tag Contents

```
Pattern: <(\w+)[^>]*>(.*?)<\/\1>
```

**Explanation:**
- `<(\w+)` — captures the tag name (group 1)
- `[^>]*>` — any tag attributes
- `(.*?)` — captures content lazily (group 2)
- `<\/\1>` — closing tag matching the captured tag name via backreference

**Example:** Applied to `<h1 class="title">Hello World</h1>` — captures `h1` and `Hello World`.

---

### 10. Strip HTML Tags

```
Pattern: <[^>]+>
Replace with: (empty string)
```

**Example:**
```python
import re
html = "<p>Hello <strong>World</strong></p>"
text = re.sub(r'<[^>]+>', '', html)
# Result: "Hello World"
```

This works for simple cases. It fails on nested angle brackets in attribute values and malformed HTML. See the FAQ below.

---

## Greedy vs Lazy Matching

By default, quantifiers are **greedy** — they consume as much as possible.

```
Input:  <div>first</div><div>second</div>
Greedy  (<div>.*</div>):  matches the ENTIRE string (both divs)
Lazy    (<div>.*?</div>): matches <div>first</div> then <div>second</div>
```

Add `?` after any quantifier to make it lazy: `*?`, `+?`, `{n,m}?`.

---

## Regex Across Languages: Key Differences

| Feature | JavaScript | Python | Java | Go |
|---|---|---|---|---|
| Syntax | `/pattern/flags` | `r"pattern"` | `"pattern"` | `` `pattern` `` |
| Global flag | `g` flag | `re.findall()` | Matcher loop | `FindAllString()` |
| Named groups | `(?<name>...)` | `(?P<name>...)` | `(?<name>...)` | `(?P<name>...)` |
| Lookahead | Yes | Yes | Yes | No |
| Lookbehind | Yes | Yes | Yes | No |
| Backreferences | Yes | Yes | Yes | No |

Go's `regexp` package uses RE2 syntax, which deliberately excludes lookaheads and backreferences to guarantee linear-time matching. This prevents catastrophic backtracking but limits expressiveness.

---

## Common Pitfalls

**Catastrophic Backtracking:** Patterns like `(a+)+` applied to a long string of `a`s followed by a non-matching character can cause exponential time complexity, freezing your application. Avoid nested quantifiers on overlapping character sets.

**Regex for HTML Parsing:** Do not use regex to parse HTML for structural purposes (navigating the DOM, extracting nested elements). Use a proper HTML parser: `BeautifulSoup` in Python, `cheerio` in Node.js, or the browser's built-in `DOMParser`. Regex is suitable for simple pattern extraction in known, controlled HTML fragments.

---

## Testing Tools

- **regex101.com** — the gold standard. Paste a pattern, see matches highlighted in real time, with a detailed explanation of every token. Supports PCRE, Python, JavaScript, Java, Go flavors.
- **RegExr.com** — clean UI, great for visual learning, built-in reference sidebar.
- **VS Code built-in search** — enables regex in Find & Replace with the `.*` icon. Great for one-off text transformations in your codebase.

---

## FAQ

**Should I use regex to parse HTML?**
No, not for structural parsing. HTML is not a regular language — it has recursive nesting that regex cannot handle correctly. Use a dedicated HTML parser. Regex is fine for finding specific patterns within known, simple HTML strings (like stripping tags from a blog excerpt), but not for navigating arbitrary HTML documents.

**How do I make regex case-insensitive?**
Most languages support an `i` flag: `/pattern/i` in JavaScript, `re.IGNORECASE` in Python, `Pattern.CASE_INSENSITIVE` in Java. Go requires `(?i)` inline: `(?i)pattern`.

**Why does my regex work in one language but not another?**
Different languages implement different regex "flavors" with varying feature support. The main divisions are PCRE (Perl-Compatible) and RE2. PCRE supports lookaheads, lookbehinds, and backreferences. RE2 (used by Go, and optionally by RE2 libraries in other languages) does not, in exchange for guaranteed linear-time matching. JavaScript, Python, Java, and .NET all use PCRE-compatible flavors with minor differences in named group syntax.

---

## Conclusion

Regular expressions reward investment. The first hour feels like deciphering an alien language. By the third hour, patterns start reading almost naturally. By the time you have written and debugged a dozen of your own, you start seeing text problems differently — noticing when a quick regex would eliminate twenty lines of string manipulation code.

Start with the 10 examples in this guide. Modify them, break them, test edge cases on regex101.com, and read the explanations the tool generates. That tight feedback loop builds intuition faster than any tutorial.

The core syntax is stable across decades and programming languages. It is one of the few skills where learning it once pays dividends everywhere.
