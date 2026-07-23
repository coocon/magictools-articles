---
title: "Unix Timestamp Converter: The Developer's Guide to Epoch Time"
slug: "timestamp-converter-guide-en"
category: developer
tags:
  - timestamp
  - unix time
  - developer tools
  - date conversion
summary: "A complete developer's guide to Unix timestamps — covering what epoch time is, why developers use it, the critical difference between seconds and milliseconds, code examples in JavaScript, Python, and Shell, the Year 2038 problem, common timezone pitfalls, and how to compare timestamps correctly across time zones."
coverImage: ""
status: published
scheduledAt: ""
---

# Unix Timestamp Converter: The Developer's Guide to Epoch Time

Dates and times are deceptively complicated in software. Timezones, daylight saving time, leap seconds, locale-specific formatting — any one of these can introduce bugs that are notoriously difficult to reproduce. The Unix timestamp cuts through this complexity by reducing any moment in time to a single integer. Understanding it is fundamental to writing correct, timezone-safe applications.

## What Is a Unix Timestamp?

A **Unix timestamp** (also called Epoch time, POSIX time, or Unix time) represents a moment in time as the number of **seconds elapsed since January 1, 1970, 00:00:00 UTC**.

This reference point — midnight UTC on January 1, 1970 — is called the **Unix Epoch**. It was chosen arbitrarily when Unix was being designed, but it has become universal across operating systems and programming languages.

Key characteristics:
- It is always in **UTC** (Coordinated Universal Time). No timezone is embedded in the number itself.
- It is a single integer (or floating-point for sub-second precision).
- It increases monotonically — you can always determine which of two timestamps comes first simply by comparing their values.
- It is compact: just 10 digits covers the entire 21st century (for second-precision timestamps).

For example, the timestamp `1700000000` corresponds to **Saturday, November 14, 2023, 22:13:20 UTC**.

## Why Developers Use Timestamps

### Timezone-Agnostic Storage

When you store `1700000000` in a database, it unambiguously means the same moment regardless of where the server, the database, or the user is located. Converting to a local display time is a presentation concern, not a storage concern.

Compare this to storing `"2023-11-14 22:13:20"` — is that UTC? The server's local time? The user's local time? Without explicit timezone information, you don't know.

### Trivial Arithmetic

Time arithmetic is simple with timestamps:

```javascript
// How many seconds ago did this event happen?
const ageInSeconds = Math.floor(Date.now() / 1000) - eventTimestamp;

// When does a 30-day trial expire?
const trialExpiry = signupTimestamp + (30 * 24 * 60 * 60);

// Is this token still valid?
const isValid = tokenExpiry > Math.floor(Date.now() / 1000);
```

Doing the same arithmetic with datetime strings requires parsing, timezone handling, and careful edge-case management around daylight saving transitions.

### Compact Storage

A Unix timestamp fits in a 4-byte integer (32-bit) or 8-byte integer (64-bit). A formatted datetime string takes 19–25 bytes as text. At scale, this storage difference is meaningful.

### Universal Compatibility

Every programming language, database, and operating system understands Unix timestamps. There's no parsing ambiguity — `MM/DD/YYYY` vs `DD/MM/YYYY` is a classic source of bugs that timestamps eliminate entirely.

## Milliseconds vs Seconds: A Critical Distinction

This is the source of more bugs than almost any other timestamp-related issue. Different systems use different precision:

| System | Unit | Example Value |
|--------|------|---------------|
| Unix tradition (C, Python, shell) | Seconds | `1700000000` |
| JavaScript, Java | Milliseconds | `1700000000000` |
| MySQL, PostgreSQL (TIMESTAMP) | Seconds | `1700000000` |
| MongoDB (Date object) | Milliseconds | `1700000000000` |
| Windows FILETIME | 100-nanosecond intervals since 1601 | (different epoch entirely) |

A common bug: taking a JavaScript timestamp (`Date.now()` returns milliseconds) and passing it directly to a function that expects seconds, or vice versa. The result is a date either 1,000 seconds off or 1,000× too far in the future.

```javascript
// WRONG: treats milliseconds as seconds
const date = new Date(1700000000); // Year 1970 + 1.7 million seconds ≈ 1990
// This is only 1.7 million milliseconds after epoch — January 20, 1970!

// CORRECT: convert seconds to milliseconds for JavaScript Date
const date = new Date(1700000000 * 1000); // November 14, 2023
```

The rule: **always check your units.** When in doubt, check whether your value is ~10 digits (seconds) or ~13 digits (milliseconds) and convert accordingly.

## How to Use the Timestamp Converter Tool

Our tool provides four operations:

1. **Current timestamp (live)**: Shows the current Unix time in both seconds and milliseconds, updating every second. Useful for quickly grabbing "now" for testing.
2. **Timestamp → Datetime**: Enter any Unix timestamp (seconds or milliseconds — the tool detects automatically) and select a timezone. The tool displays the corresponding human-readable date and time.
3. **Datetime → Timestamp**: Enter a date/time in any timezone and convert it to a Unix timestamp.
4. **Time difference calculator**: Enter two timestamps and see the difference in days, hours, minutes, and seconds.

## Code Examples

### JavaScript

```javascript
// Get current timestamp in milliseconds (JavaScript default)
Date.now(); // e.g., 1700000000000

// Get current timestamp in seconds
Math.floor(Date.now() / 1000); // e.g., 1700000000

// Convert timestamp (seconds) to a Date object
const date = new Date(1700000000 * 1000);
console.log(date.toISOString()); // "2023-11-14T22:13:20.000Z"

// Format with Intl.DateTimeFormat (respects locale and timezone)
const formatter = new Intl.DateTimeFormat("en-US", {
  timeZone: "America/New_York",
  dateStyle: "full",
  timeStyle: "long",
});
console.log(formatter.format(date));
// "Tuesday, November 14, 2023 at 5:13:20 PM EST"

// Convert a specific datetime string to timestamp
const ts = Math.floor(new Date("2023-11-14T22:13:20Z").getTime() / 1000);
console.log(ts); // 1700000000
```

### Python

```python
import time
from datetime import datetime, timezone

# Get current timestamp in seconds
int(time.time())  # e.g., 1700000000

# Convert timestamp to UTC datetime
datetime.utcfromtimestamp(1700000000)
# datetime(2023, 11, 14, 22, 13, 20)

# Timezone-aware conversion (recommended)
dt = datetime.fromtimestamp(1700000000, tz=timezone.utc)
print(dt.isoformat())  # "2023-11-14T22:13:20+00:00"

# Convert datetime to timestamp
dt = datetime(2023, 11, 14, 22, 13, 20, tzinfo=timezone.utc)
int(dt.timestamp())  # 1700000000

# Display in a specific timezone (requires pytz or zoneinfo)
from zoneinfo import ZoneInfo
dt_local = dt.astimezone(ZoneInfo("America/New_York"))
print(dt_local)  # 2023-11-14 17:13:20-05:00
```

### Shell (Bash)

```bash
# Get current timestamp (seconds)
date +%s
# e.g., 1700000000

# Get current timestamp in milliseconds
date +%s%3N

# Convert timestamp to human-readable date (Linux)
date -d @1700000000
# Tue Nov 14 22:13:20 UTC 2023

# Convert timestamp to human-readable date (macOS)
date -r 1700000000
# Tue Nov 14 22:13:20 UTC 2023

# Convert a datetime string to timestamp (Linux)
date -d "2023-11-14 22:13:20 UTC" +%s
# 1700000000
```

## Common Pitfalls

### Timezone Confusion

The most common mistake: storing or comparing datetimes in local time instead of UTC.

**Rule**: Always store timestamps in UTC. Convert to local time only at the point of display to the user. Never store times that depend on the local timezone of the server or application.

A server that stores "November 14, 2023 at 5:13 PM" without timezone information is storing ambiguous data. If the server moves to a different data center in a different timezone, all those historical records become wrong.

### Mixing Milliseconds and Seconds

As discussed above, this is a frequent bug. To defend against it:
- Adopt a convention and document it explicitly (e.g., "all timestamps in our codebase are in seconds")
- Use descriptive variable names: `createdAtMs` vs `createdAtSec`
- Write a type or class that enforces the unit in strongly-typed languages

### The Year 2038 Problem

Unix timestamps stored as **signed 32-bit integers** can only represent values up to `2,147,483,647` — which corresponds to **January 19, 2038, 03:14:07 UTC**. After this moment, a 32-bit signed integer overflows to a large negative number, causing date calculation errors.

Modern 64-bit systems are not affected — a 64-bit signed integer can represent timestamps billions of years into the future. However, legacy embedded systems, some databases with `INT(11)` timestamp columns, and old C code may still be vulnerable. If your system stores timestamps as 32-bit integers, migrate to 64-bit before 2038.

## Pro Tips

- **Use ISO 8601 for human-readable dates in APIs**: `2023-11-14T22:13:20Z` is unambiguous, sortable, and parseable by every language. The trailing `Z` means UTC.
- **In logs, always include timezone**: `2023-11-14 22:13:20 UTC` is clear. `2023-11-14 22:13:20` alone is ambiguous and will cause confusion during incident response.
- **When comparing "is event X within the last 24 hours?", use timestamps** not datetime strings. `currentTimestamp - eventTimestamp < 86400` is bulletproof.
- **Cache your `Date.now()` / `time.time()` calls.** Calling them in a tight loop is expensive in some contexts. Capture once at the start of a request cycle.

## FAQ

**Should I store dates as timestamps or strings in a database?**

Use your database's native `TIMESTAMP` or `DATETIME` type with explicit UTC timezone, rather than storing raw integers or strings. Native types give you built-in functions for date arithmetic, proper indexing, and automatic timezone handling. If your ORM or query builder handles UTC automatically (which most modern ones do), you get the benefits of timestamps with better readability. Use raw Unix timestamp integers when you need maximum portability, minimum storage, or interoperability with systems that only understand integers.

**How do I compare timestamps across timezones?**

Always convert to UTC before comparing. If timestamp A is `1700000000` (UTC) and timestamp B is `"2023-11-14 17:13:20"` in New York time (UTC−5), convert B to UTC first: it becomes `1700000000`. They're the same moment. Never compare a UTC timestamp to a local-time timestamp directly. Unix timestamps are inherently UTC, so two Unix timestamps are always directly comparable with `<`, `>`, or `===`.

**What's the maximum Unix timestamp?**

For 64-bit signed integers: `9,223,372,036,854,775,807` seconds, which is approximately the year 292,277,026,596 — effectively unlimited for any practical purpose. For 32-bit signed integers: `2,147,483,647` seconds — January 19, 2038 (the Year 2038 problem). For JavaScript's `Date` object: it uses 64-bit floats internally, supporting dates up to approximately ±8,640,000,000,000,000 milliseconds from epoch, which covers 275,760 years in each direction.

## Conclusion

The Unix timestamp is one of the most practical abstractions in computer science — a single integer that represents any moment in time, unambiguously, in every programming language and system. Mastering timestamps means fewer timezone bugs, simpler time arithmetic, and more portable code. Use our [Timestamp Converter tool](/tools/timestamp) to quickly convert between epoch time and human-readable dates, or to grab the current timestamp for testing.
