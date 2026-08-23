# Regular Expressions for Beginners: 10 Practical Examples You'll Actually Use

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/regex-guide-beginners-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/regex-guide-beginners-en?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 Continue reading: Regular Expressions for Beginners: 10 Practical Examples You'll Actually Use](https://tools.cooconsbit.com/en/articles/regex-guide-beginners-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
