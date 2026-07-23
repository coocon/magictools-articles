---
title: "Color Converter Guide: HEX, RGB, HSL, and Modern CSS Colors (2026)"
slug: "color-converter-hex-rgb-hsl-guide-en"
category: developer
tags:
  - Color Conversion
  - CSS
  - Design Systems
  - OKLCH
  - Developer Tools
summary: "Front-end engineers convert colors every day. This guide compares HEX, RGB, HSL, and OKLCH — strengths, weaknesses, conversion formulas, and JavaScript snippets. It covers real scenarios from Figma hand-off and dark mode to theme scales and WCAG contrast, with engineering best practices for modern 2026 design systems."
coverImage: ""
status: published
scheduledAt: ""
---

## Color Conversion: A Daily Front-End Reality

Open DevTools on any modern site and you'll see a color cocktail like this:

```css
.button {
  background: #3b82f6;             /* HEX */
  color: rgb(255 255 255);         /* RGB */
  border-color: hsl(217 91% 60%);  /* HSL */
  box-shadow: 0 4px 12px oklch(0.7 0.15 230 / 0.4);  /* OKLCH */
}
```

Each format has its sweet spot: HEX for syncing with design files, RGB for programmatic computation, HSL for theme tweaking, OKLCH for the modern design systems of 2026. This guide untangles the relationships between them and gives you efficient conversion workflows.

---

## Format-by-Format Comparison

### HEX: The Designer's Native Tongue

The oldest and most ubiquitous format: `#RRGGBB`, where each pair is a hex digit (00-FF) for red, green, and blue.

```css
#3b82f6     /* blue */
#3B82F6     /* same — case doesn't matter */
#3b82f6cc   /* 8-digit = RGB + alpha */
#3bf        /* shorthand for #33bbff */
```

**Pros:** compact, copy-pasteable, native to Photoshop and Figma
**Cons:** doesn't reveal color relationships; lightening/darkening requires fresh math

### RGB / RGBA: The Browser's Internal Format

```css
rgb(59 130 246)         /* modern syntax (recommended) */
rgb(59, 130, 246)       /* legacy syntax with commas */
rgba(59 130 246 / 0.8)  /* alpha after slash */
```

**Pros:** maps directly to pixels, easy to generate programmatically, simple linear blending
**Cons:** humans can't eyeball a color from numbers

### HSL: The Theming Swiss Army Knife

```css
hsl(217 91% 60%)  /* hue / saturation / lightness */
```

- **H (Hue):** 0-360 degrees, walking the rainbow
- **S (Saturation):** 0% (gray) to 100% (most vivid)
- **L (Lightness):** 0% (black) to 100% (white); 50% is the most saturated pure color

**Killer feature:** lock H and S, vary L only, and you instantly generate a theme color scale (this is exactly how Tailwind's blue-50 → blue-900 works).

### OKLCH: The 2026 Standard

A modern format from CSS Color Module Level 4, **perceptually uniform** — visual change scales with the numbers:

```css
oklch(0.7 0.15 230)  /* lightness / chroma / hue */
```

- **L:** perceptual lightness, 0 (black) to 1 (white)
- **C:** chroma (close to saturation), 0 to roughly 0.4
- **H:** hue, 0-360 degrees

**Why switch?** HSL "lightness" is mathematically defined, not perceptually accurate. A yellow at HSL `lightness: 50%` looks bright; purple at the same value looks dark. OKLCH actually delivers "same number = same visual brightness" — a gift to design systems.

---

## Online Conversion in 3 Steps

No formulas to memorize. Open MagicTools' color converter:

1. **Input any format** — paste HEX, RGB, HSL, or a color name (red/blue/...)
2. **Auto-convert** — every format updates simultaneously, hit copy
3. **Visual preview** — a swatch on the right confirms the color visually

Supports 8 mainstream formats: HEX, RGB, HSL, HSV, OKLCH, LAB, CMYK, and CSS color names.

---

## Cheat Sheet for Common Conversions

### HEX ↔ RGB

Convert each pair of hex digits to decimal:

```
#3b82f6
3b = 59
82 = 130
f6 = 246
→ rgb(59, 130, 246)
```

One-liner JavaScript:

```javascript
const hexToRgb = hex => hex.match(/\w\w/g).map(x => parseInt(x, 16));
hexToRgb('3b82f6');  // [59, 130, 246]

const rgbToHex = (r, g, b) =>
  '#' + [r, g, b].map(x => x.toString(16).padStart(2, '0')).join('');
rgbToHex(59, 130, 246);  // '#3b82f6'
```

### RGB ↔ HSL

The formula is non-trivial — use a tool or library (chroma.js, color, culori). Conceptually:

- Find max and min of R/G/B
- Average → L (lightness)
- (max - min) → S (saturation)
- Which channel is the max determines H (hue)

### CSS Color Names → HEX

The CSS spec defines 147 named colors:

| Name | HEX | Use case |
|------|-----|----------|
| `tomato` | #FF6347 | Alert / accent |
| `slategray` | #708090 | Neutral background |
| `forestgreen` | #228B22 | Nature theme |
| `dodgerblue` | #1E90FF | Link color |

Full list at [MDN Named Colors](https://developer.mozilla.org/en-US/docs/Web/CSS/named-color).

---

## Real-World Scenarios

### Scenario 1: From Figma Hand-Off to Code

Designers usually deliver HEX values, but your design system uses OKLCH:

```
Figma: #3b82f6
↓ convert
oklch(0.624 0.198 259.81)
```

Storing OKLCH in CSS variables lets you adjust colors perceptually:

```css
:root {
  --primary: oklch(0.624 0.198 259.81);
  --primary-light: oklch(0.75 0.15 259.81);  /* lift lightness only */
  --primary-dark: oklch(0.45 0.2 259.81);
}
```

### Scenario 2: Dynamic Alpha Adjustment

Need a 30%-transparent version of a color:

```css
/* HEX 8-digit form */
background: #3b82f64d;  /* 4d = 77/255 ≈ 30% */

/* RGB form (more readable) */
background: rgb(59 130 246 / 30%);

/* HSL/OKLCH similarly */
background: oklch(0.624 0.198 259.81 / 0.3);
```

### Scenario 3: Generating a Theme Color Scale

Tailwind's 50/100/200/.../900 scale is essentially fixed hue/saturation with varying lightness. One HSL line generates it:

```javascript
const generateScale = (hue, sat) => {
  const lightnesses = [97, 92, 85, 75, 60, 47, 38, 30, 22];  // 9 stops
  return lightnesses.map(l => `hsl(${hue} ${sat}% ${l}%)`);
};

generateScale(217, 91);  // matches Tailwind blue
```

### Scenario 4: Picking Colors for Dark Mode

Dark mode is not just inverted colors. Mirror the lightness with OKLCH for natural results:

```css
/* light mode */
:root {
  --bg: oklch(0.98 0.01 240);
  --text: oklch(0.2 0.02 240);
}

/* dark mode: flip L */
[data-theme="dark"] {
  --bg: oklch(0.18 0.01 240);
  --text: oklch(0.92 0.02 240);
}
```

---

## Color Accessibility: Contrast Ratios You Must Hit

WCAG 2.1 minimums:

| Content type | AA | AAA |
|--------------|-----|------|
| Body text (< 18pt) | 4.5:1 | 7:1 |
| Large text (≥ 18pt) | 3:1 | 4.5:1 |
| Graphics / UI controls | 3:1 | — |

**Practical tools:**

- Browser DevTools' built-in contrast checker (hover an element)
- WebAIM Contrast Checker (online)
- VS Code `Color Highlight` extension

**Common pitfall:** purple buttons with white text and gray backgrounds with light gray text are the two most-cited contrast violations. A pretty design file does not guarantee shippable contrast — always run the checker.

---

## Engineering Discipline for Color Management

### 1. CSS Variables Only — No Hardcoding

```css
/* ❌ bad */
.button { background: #3b82f6; }
.link { color: #3b82f6; }

/* ✅ good */
:root { --primary: #3b82f6; }
.button { background: var(--primary); }
.link { color: var(--primary); }
```

### 2. Name by Purpose, Not Appearance

```css
/* ❌ unmaintainable */
--blue-500: #3b82f6;
.cta-button { background: var(--blue-500); }

/* ✅ maintainable */
--color-primary: #3b82f6;
--color-cta-bg: var(--color-primary);
.cta-button { background: var(--color-cta-bg); }
```

Future rebrand = one-line change, no global find-and-replace.

### 3. Sync With Figma Variables

Figma 2025's Variables can two-way sync with code tokens. Designer changes color → an automated PR is created — no lossy human re-typing.

### 4. Centralize in a Design Tokens File

```json
{
  "color": {
    "primary": { "value": "#3b82f6" },
    "primary-hover": { "value": "{color.primary}" }
  }
}
```

Tools like Style Dictionary turn one token file into CSS, SCSS, iOS, and Android outputs.

---

## Frequently Asked Questions

**Q: How well-supported is 8-digit HEX (with alpha)?**
A: Modern browsers (Chrome 62+, Firefox 49+, Safari 9.1+) support it fully. IE 11 doesn't, but IE 11 is fully retired in 2026.

**Q: Is OKLCH safe to use in production today?**
A: Yes — Chrome/Edge/Firefox/Safari all support it (Safari 15.4+ and other major browsers since 2023). If you must support legacy devices, use a PostCSS `oklab` plugin to auto-fall-back to RGB.

**Q: Why do same-lightness HSL colors look different in brightness?**
A: HSL "lightness" is a mathematical average, not perceptual brightness. `hsl(60 100% 50%)` (yellow) looks much brighter than `hsl(240 100% 50%)` (blue). Hence OKLCH.

**Q: How do I convert between CMYK and RGB?**
A: CMYK targets print, RGB targets screens — the gamuts differ and **there is no perfect conversion**. For print-bound screen designs, do color management in Photoshop with an ICC profile rather than naive math.

**Q: Color names vs HEX — which performs better?**
A: Identical. The CSS parser converts both to internal RGB. Use names for readability, HEX/HSL/OKLCH for control.

---

## Wrap-Up

Color conversion looks trivial but sits at the intersection of color science, design systems, and accessibility engineering. 2026 best practices:

- **Design hand-off:** HEX
- **Programmatic math:** RGB
- **Theme scales:** HSL (legacy) or OKLCH (modern)
- **Design system:** OKLCH, exploit its perceptual uniformity
- **Always centralize via CSS variables**

Need to convert formats right now? Open the [MagicTools color converter](/tools/color-converter) — all mainstream formats sync in real time.
