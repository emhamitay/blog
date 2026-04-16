---
title: "Typography Hierarchy — Why the Text Feels Premium"
description: "Stripe's headline, sub-headline, and CTA buttons work together through a precise type scale. Understanding the principles — size contrast, weight, line-height math, and responsive clamp() — is what makes type feel intentional rather than arbitrary."
date: 2026-04-16
parent: "stripe-hero"
order: 5
tags: ["CSS", "typography", "type-scale", "clamp", "line-height"]
---

# Typography Hierarchy — Why the Text Feels Premium

Open Stripe's homepage and look at the text in the hero. There's a large headline, a smaller sub-headline beneath it, and two CTA buttons. Four elements, but the visual weight difference between them communicates a clear reading order without any effort from the reader.

This doesn't happen by accident. It's a deliberate type scale with specific size jumps, weight contrast, line-height math, and responsive sizing. Each decision is grounded in a principle.

---

## The type scale — the size hierarchy

A type scale is a set of font sizes that have a consistent mathematical relationship. The classic approach uses a ratio — multiply each level by a fixed number to get the next:

Common ratios:
| Name | Ratio | Feel |
|---|---|---|
| Minor Second | 1.067 | Very tight — good for dense UIs |
| Major Second | 1.125 | Tight — good for body text and small headings |
| Minor Third | 1.200 | Moderate — commonly used |
| Major Third | 1.250 | Clear jump — good for landing pages |
| Perfect Fourth | 1.333 | Strong — expressive hierarchy |
| Golden Ratio | 1.618 | Very dramatic — use sparingly |

Stripe's hero uses something close to a Perfect Fourth scale. The approximate sizes:

| Element | Size | Ratio from body |
|---|---|---|
| Body / label text | 16px | base |
| Sub-headline | 20px | ~1.25× |
| CTA button | 16px (same as body, but bold) | — |
| Headline | 56–72px | ~3.5–4.5× body |

The headline is not just "a bit bigger." It's dramatically larger — 3.5× or more. This is what makes it a *headline* rather than a big paragraph. Small ratios produce text that feels like multiple paragraphs at slightly different sizes. Large ratios produce clear visual priority.

---

## Responsive headline — clamp()

Stripe's headline scales with the viewport. On mobile it might be 36px; on desktop it might be 72px. Rather than media query breakpoints, the modern approach is `clamp()`:

```css
.hero__headline {
  font-size: clamp(2.25rem, 5vw + 1rem, 4.5rem);
}
```

`clamp(min, preferred, max)`:
- **min (`2.25rem` / 36px)** — the smallest it will ever get (mobile)
- **preferred (`5vw + 1rem`)** — scales with viewport width
- **max (`4.5rem` / 72px)** — the largest it will ever get (desktop)

The preferred value `5vw + 1rem` means: 5% of the viewport width, plus 1rem. On a 320px viewport: `5vw = 16px`, plus `1rem = 16px` = `32px` — but `clamp` enforces the minimum of `36px`. On an 800px viewport: `5vw = 40px` + `1rem = 16px` = `56px`. On a 1440px viewport: `5vw = 72px` + `1rem = 16px` = `88px`, but clamped to max `72px`.

This single declaration handles the entire responsive font size range with no breakpoints.

> 💡 **The golden rule for clamp() on type:** your minimum should be readable on the smallest screen, your maximum should look right on the largest, and your preferred formula should create smooth interpolation between them. A common formula is `preferred = Xvw + Yrem` where you tune X and Y until the sizes feel right at a few key viewport widths.

---

## Weight contrast — not just size

Stripe's headline uses a heavy weight — `font-weight: 700` or higher (`800`, `900`). The sub-headline is regular weight (`400`). The CTA is `600`.

Weight contrast amplifies the size difference. Two elements at different sizes but the same weight feel like siblings. An element at 3× the size and 300 weight units heavier reads as a different *kind* of content — a true heading rather than just larger text.

The rule: **if you're going for dramatic size contrast (3× or more), use weight contrast to reinforce it.** If the sizes are close, use weight contrast more carefully — it can create visual noise if everything is bold.

---

## Line-height — the math

Line-height (`line-height`) is the vertical space given to each line of text. It affects readability dramatically.

The formula that works:
- **Headlines (large text, often 1–2 lines):** `line-height: 1.1` to `1.2`. Tight — the lines feel like one unit.
- **Sub-headlines / medium text:** `line-height: 1.3` to `1.5`. Moderate breathing room.
- **Body text / descriptions:** `line-height: 1.5` to `1.7`. Enough space to read multiple lines comfortably.

Stripe's headline uses approximately `line-height: 1.1`. At 64px font size, that's about 70px of line height — just 6px between lines. This tightness makes the headline feel like a single graphic element rather than multiple lines of text.

```css
.hero__headline {
  font-size: clamp(2.25rem, 5vw + 1rem, 4.5rem);
  font-weight: 800;
  line-height: 1.1;
  letter-spacing: -0.02em;  /* slight tightening at large sizes */
  color: #fff;
}

.hero__subheadline {
  font-size: clamp(1rem, 1.5vw + 0.5rem, 1.25rem);
  font-weight: 400;
  line-height: 1.6;
  color: rgba(255, 255, 255, 0.85);
  max-width: 42ch;  /* ~character limit for comfortable reading width */
}
```

---

## Letter-spacing at large sizes

At large font sizes, the natural letter-spacing (tracking) of most typefaces looks slightly too loose. It's a quirk of how fonts are designed — they're optimized for body text sizes. At 64px, individual letterforms have too much air between them.

The fix: a small negative `letter-spacing` at heading sizes:

```css
.hero__headline {
  letter-spacing: -0.02em;  /* tighten by 2% of the font size */
}
```

At `64px`, `-0.02em` = `-1.28px`. Subtle, but it makes the headline feel more cohesive. Stripe consistently applies this. At body text sizes, never apply negative tracking — it hurts readability.

---

## The `max-width` on sub-headlines

Long sub-headline text that spans full-width on a desktop monitor is hard to read. The human eye tracks back to the start of the next line after reading each one — on very long lines, this tracking becomes tiring and introduces errors.

The comfortable reading width is approximately **50–75 characters per line**. The CSS unit `ch` is exactly the width of the `0` character in the current font — a useful approximation for character count:

```css
.hero__subheadline {
  max-width: 52ch;  /* ~52 characters per line */
}
```

Stripe's sub-headline does this. The text doesn't stretch to full column width — it stops at a comfortable reading measure.

---

## The CTA buttons — not typography, but spacing

The CTA buttons aren't large in font size. They're `16px` — same as body text. What makes them feel substantial is **padding**:

```css
.hero__cta-primary {
  font-size: 1rem;
  font-weight: 600;
  padding: 0.875rem 1.5rem;  /* generous vertical padding */
  border-radius: 6px;
  letter-spacing: 0;         /* neutral — not tight, not loose */
}
```

The tall padding creates a comfortable tap target and makes the button feel weighty without inflating the text size. This is the correct approach — large text in buttons often looks wrong. Large padding in buttons looks premium.

---

## The complete type stack

```css
/* Headline */
.hero__headline {
  font-size: clamp(2.25rem, 5vw + 1rem, 4.5rem);
  font-weight: 800;
  line-height: 1.1;
  letter-spacing: -0.02em;
  color: #ffffff;
  margin-bottom: 1.5rem;
}

/* Sub-headline */
.hero__subheadline {
  font-size: clamp(1rem, 1.5vw + 0.5rem, 1.25rem);
  font-weight: 400;
  line-height: 1.6;
  color: rgba(255, 255, 255, 0.82);
  max-width: 52ch;
  margin-bottom: 2.5rem;
}

/* CTA primary */
.hero__cta-primary {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 1rem;
  font-weight: 600;
  line-height: 1;
  padding: 0.875rem 1.5rem;
  border-radius: 6px;
  background: #ffffff;
  color: #5850ec;
  text-decoration: none;
}

/* CTA secondary */
.hero__cta-secondary {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 1rem;
  font-weight: 500;
  line-height: 1;
  padding: 0.875rem 1.5rem;
  color: rgba(255, 255, 255, 0.9);
  text-decoration: none;
}
```

The four elements together — headline, sub-headline, primary CTA, secondary CTA — hit four distinct visual levels. Clear hierarchy, no confusion about reading order.
