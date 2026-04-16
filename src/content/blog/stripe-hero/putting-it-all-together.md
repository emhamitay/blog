---
title: "Putting It All Together — Building the Stripe Hero"
description: "Every concept from the sub-articles converges here. We build the full Stripe-like hero from scratch — the gradient mesh background, the diagonal clip, the two-column grid, the bleeding screenshots, and the type hierarchy — with every line explained."
date: 2026-04-16
parent: "stripe-hero"
order: 6
tags: ["CSS", "layout", "clip-path", "gradients", "grid", "typography"]
---

# Putting It All Together — Building the Stripe Hero

You've read the foundations. You know what `clip-path: polygon()` does to the render tree, how the `fr` algorithm distributes free space, why oklch gradients stay vivid through the midpoint, what creates a stacking context, and how `clamp()` produces a headline that scales without breakpoints.

Now we assemble them into a single implementation. Every line in the code below maps directly to something you've already understood. Nothing will be a mystery.

---

## What we're building

A hero section with:
- A full-width gradient mesh background — layered radial gradients over a linear base, in oklch
- A diagonal bottom cut using `clip-path: polygon()`
- A two-column grid layout — text left, screenshots right
- UI screenshots that bleed below the diagonal cut into the section below
- A responsive type hierarchy — headline, sub-headline, two CTA buttons
- Fully responsive — collapses to single column on mobile with no media query hacks

---

## The HTML structure

```html
<!-- index.html -->

<div class="hero-wrapper">

  <!-- The background is a separate element so clip-path doesn't clip the content -->
  <div class="hero-bg" aria-hidden="true"></div>

  <!-- The content grid -->
  <section class="hero-content" aria-label="Hero">

    <!-- Left column: text -->
    <div class="hero-text">
      <h1 class="hero-headline">
        Financial infrastructure<br>for the internet
      </h1>
      <p class="hero-subheadline">
        Millions of companies — from startups to the world's largest enterprises
        — use Stripe to accept payments, send payouts, and manage their businesses online.
      </p>
      <div class="hero-ctas">
        <a href="#" class="cta-primary">Start now →</a>
        <a href="#" class="cta-secondary">Contact sales</a>
      </div>
    </div>

    <!-- Right column: screenshots -->
    <div class="hero-screenshots" aria-hidden="true">
      <div class="screenshot-card screenshot-card--back">
        <img src="/screenshots/dashboard-2.png" alt="" width="480" height="300" />
      </div>
      <div class="screenshot-card screenshot-card--front">
        <img src="/screenshots/dashboard-1.png" alt="" width="480" height="300" />
      </div>
    </div>

  </section>
</div>
```

**Why the `.hero-bg` is separate from `.hero-content`:**
If we applied `clip-path` directly to the wrapper, the screenshots inside would also be clipped — they couldn't bleed below the diagonal line. By separating the background into its own element and clipping only that, the content (including the bleeding screenshots) is free to overflow.

---

## Layer 1 — The wrapper

```css
/* ─── Wrapper ───────────────────────────────────────────── */

.hero-wrapper {
  position: relative;      /* establishes positioning context for .hero-bg */
  overflow: visible;        /* default — DO NOT change to hidden */
                            /* overflow: hidden would clip the screenshot bleed */
}
```

This is intentionally minimal. Its only job is to be a positioning parent for the background layer. No `overflow: hidden` — that would prevent the screenshots from bleeding below.

---

## Layer 2 — The gradient mesh background

```css
/* ─── Background ────────────────────────────────────────── */

.hero-bg {
  position: absolute;
  inset: 0;                /* fills .hero-wrapper exactly */
  z-index: 0;              /* below the content */

  background:
    /* Pink blob — upper right area */
    radial-gradient(
      in oklch ellipse at 78% 20%,
      oklch(0.72 0.25 330° / 0.38) 0%,
      transparent 48%
    ),
    /* Cyan blob — right middle */
    radial-gradient(
      in oklch ellipse at 90% 65%,
      oklch(0.75 0.18 200° / 0.42) 0%,
      transparent 45%
    ),
    /* Purple blob — left center */
    radial-gradient(
      in oklch ellipse at 12% 55%,
      oklch(0.58 0.28 275° / 0.52) 0%,
      transparent 52%
    ),
    /* Subtle blue blob — center */
    radial-gradient(
      in oklch ellipse at 50% 40%,
      oklch(0.62 0.20 240° / 0.25) 0%,
      transparent 55%
    ),
    /* Base linear gradient */
    linear-gradient(
      in oklch 135deg,
      oklch(0.585 0.233 264°) 0%,    /* #6366f1 — vivid indigo */
      oklch(0.62 0.20 230°) 45%,     /* blue-indigo midpoint */
      oklch(0.68 0.18 195°) 100%     /* #0ea5e9 — bright cyan */
    );

  /* The diagonal cut — this is the Stripe slope */
  /* bottom-left stays at 100%, bottom-right rises to 88% */
  /* the difference over full width creates the ~6.5° angle */
  clip-path: polygon(0 0, 100% 0, 100% 88%, 0 100%);
}
```

**Why `in oklch` on the radial gradients:**
Without it, the browser blends through sRGB — vivid colors pass through a gray zone. With `in oklch`, blending stays perceptually vivid. At these alpha levels and blob sizes, the difference is visible, especially in the overlapping regions where blobs mix.

**Why four radial layers:**
Three is enough for a basic mesh. The fourth (center blob) adds density in the middle of the hero, preventing the base gradient from looking flat. Stripe's actual implementation uses at least 4–5 radial layers.

**The clip-path numbers:**
`polygon(0 0, 100% 0, 100% 88%, 0 100%)` — top edge is flat, bottom-right is at 88% of the element's height, bottom-left is at 100%. The 12-percentage-point vertical difference across the full element width creates approximately a 6.5° slope. Adjust `88%` to change the angle — lower = steeper, higher = shallower.

---

## Layer 3 — The content grid

```css
/* ─── Content Grid ──────────────────────────────────────── */

.hero-content {
  position: relative;
  z-index: 1;              /* above .hero-bg (z-index: 0) */

  display: grid;
  grid-template-columns: 3fr 2fr;   /* text takes 3/5, screenshots take 2/5 */
  align-items: center;              /* vertically center both columns */
  gap: clamp(2rem, 5vw, 5rem);     /* responsive gap — not a fixed value */

  /* Padding: generous top and bottom, extra bottom for bleed space */
  padding:
    clamp(4rem, 8vw, 7rem)          /* top */
    clamp(1.5rem, 5vw, 5rem)        /* right */
    clamp(6rem, 12vw, 10rem)        /* bottom — extra room for screenshot bleed */
    clamp(1.5rem, 5vw, 5rem);       /* left */

  max-width: 1280px;
  margin: 0 auto;          /* center the content within the full-width wrapper */
}

@media (max-width: 820px) {
  .hero-content {
    grid-template-columns: 1fr;     /* single column */
    gap: 3rem;
    padding-bottom: 4rem;           /* less bottom padding on mobile */
  }
}
```

**Why `3fr 2fr` and not `60% 40%`:**
`fr` units subtract the `gap` before distributing. Percentages don't — they'd overflow. Also, `fr` responds to the actual available space, which is important when the container has padding.

**Why `clamp()` on padding and gap:**
On very large viewports, fixed padding (say `5rem`) leaves the content cramped relative to the hero width. On small viewports, fixed padding wastes too much space. `clamp()` creates a smooth proportional relationship — the hero breathes correctly at every viewport width.

---

## Layer 4 — The text block

```css
/* ─── Text Block ────────────────────────────────────────── */

.hero-text {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 0;                           /* spacing handled with margin-bottom per element */
}

.hero-headline {
  font-size: clamp(2.25rem, 4.5vw + 0.5rem, 4.5rem);
  font-weight: 800;
  line-height: 1.08;                /* tight — headline reads as one visual block */
  letter-spacing: -0.025em;        /* tighten at large sizes */
  color: #ffffff;
  margin: 0 0 1.25rem 0;
}

.hero-subheadline {
  font-size: clamp(1rem, 1.2vw + 0.5rem, 1.2rem);
  font-weight: 400;
  line-height: 1.65;               /* comfortable for 2–4 line reading */
  color: rgba(255, 255, 255, 0.82);
  max-width: 50ch;                  /* ~50 characters — comfortable reading measure */
  margin: 0 0 2.25rem 0;
}

.hero-ctas {
  display: flex;
  align-items: center;
  gap: 1rem;
  flex-wrap: wrap;                  /* wraps on narrow screens without breaking layout */
}

.cta-primary {
  display: inline-flex;
  align-items: center;
  gap: 0.375rem;
  font-size: 1rem;
  font-weight: 600;
  line-height: 1;
  padding: 0.875rem 1.625rem;
  border-radius: 6px;
  background: #ffffff;
  color: oklch(0.585 0.233 264°);  /* matches the indigo from the gradient */
  text-decoration: none;
  transition: background 0.15s ease, transform 0.1s ease;
}

.cta-primary:hover {
  background: rgba(255, 255, 255, 0.92);
  transform: translateY(-1px);
}

.cta-secondary {
  display: inline-flex;
  align-items: center;
  gap: 0.375rem;
  font-size: 1rem;
  font-weight: 500;
  line-height: 1;
  padding: 0.875rem 1.25rem;
  color: rgba(255, 255, 255, 0.88);
  text-decoration: none;
  border-radius: 6px;
  transition: color 0.15s ease, background 0.15s ease;
}

.cta-secondary:hover {
  color: #ffffff;
  background: rgba(255, 255, 255, 0.1);
}
```

**Why `letter-spacing: -0.025em` on the headline:**
At `72px`, the default inter-character spacing of most typefaces feels slightly loose. `-0.025em` = `-1.8px` at that size. Imperceptible when called out, but immediately noticeable if removed. Stripe applies this consistently.

**Why `max-width: 50ch` on the subheadline:**
At desktop widths, the left column is 3/5 of 1280px ≈ 730px of usable space. A `1.2rem` (~19px) font at 730px wide would produce 70+ characters per line — too wide to read comfortably. `50ch` limits it to ~50 characters and creates the visual sense of a tightly controlled text block.

**Why `rgba(255,255,255,0.82)` for the sub-headline color:**
Pure white (`#fff`) creates too much contrast — it competes with the headline. 82% opacity white is still highly legible on the gradient background but reads as clearly secondary to the full-white headline.

---

## Layer 5 — The bleeding screenshots

```css
/* ─── Screenshots ───────────────────────────────────────── */

.hero-screenshots {
  position: relative;
  /* Push the screenshot block downward so it extends past the hero's clip boundary */
  /* This is the bleed — the screenshots overlap the section below */
  transform: translateY(clamp(2rem, 5vw, 5rem));
}

.screenshot-card {
  position: absolute;
  border-radius: 12px;
  overflow: hidden;
  box-shadow:
    0 20px 60px rgba(0, 0, 0, 0.3),
    0 4px 16px rgba(0, 0, 0, 0.15);
}

.screenshot-card img {
  display: block;
  width: 100%;
  height: auto;
}

/* The back card — offset behind the front card */
.screenshot-card--back {
  top: 0;
  left: 0;
  width: 88%;
  transform: translateX(14%) translateY(8%) rotate(-1.5deg);
  z-index: 0;
  opacity: 0.85;
}

/* The front card — the main featured screenshot */
.screenshot-card--front {
  position: relative;   /* in flow — sets the natural size of the container */
  width: 100%;
  z-index: 1;
  transform: rotate(0.5deg);  /* subtle tilt */
}

@media (max-width: 820px) {
  .hero-screenshots {
    transform: none;           /* no bleed on mobile */
    margin-top: 1rem;
    padding-bottom: 2rem;
  }

  .screenshot-card--back {
    display: none;             /* simplify to single card on mobile */
  }
}
```

**How the bleed works:**
`transform: translateY(clamp(2rem, 5vw, 5rem))` pushes the entire screenshot block downward — into the space below the diagonal clip. The `.hero-wrapper` has `overflow: visible`, and `.hero-bg` is the only element with `clip-path`. The content (`.hero-content`) is not clipped, so the screenshots can extend as far as the transform pushes them.

**Why `position: relative` on `.screenshot-card--front`:**
The back card is `position: absolute` — it's taken out of flow and doesn't contribute to the container's height. The front card is `position: relative` (in flow) so `.hero-screenshots` has a natural size to work with. Without this, the container would collapse to zero height and the layout would break.

**The subtle rotations:**
`rotate(-1.5deg)` on the back card and `rotate(0.5deg)` on the front. Small enough to be subliminal. Large enough that the eye perceives depth and layering. The Stripe hero uses exactly this technique — it's what makes the screenshots feel like physical objects rather than flat images.

---

## The section below — accounting for the bleed

The section immediately following `.hero-wrapper` needs top padding to account for the screenshots bleeding into it:

```css
.section-after-hero {
  padding-top: clamp(8rem, 15vw, 14rem);
  /* Extra top padding so the screenshots don't cover the section's own content */
}
```

Alternatively, if you know the exact bleed amount, you can set a negative `margin-top` on `.hero-wrapper` equal to the bleed. But `padding-top` on the following section is simpler and more maintainable.

---

## The complete stylesheet

```css
/* ─── Reset & Base ──────────────────────────────────────── */

*, *::before, *::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Inter', 'Segoe UI', sans-serif;
  background: #ffffff;
  color: #1a1a2e;
  -webkit-font-smoothing: antialiased;
}

/* ─── Wrapper ───────────────────────────────────────────── */

.hero-wrapper {
  position: relative;
  overflow: visible;
}

/* ─── Background ────────────────────────────────────────── */

.hero-bg {
  position: absolute;
  inset: 0;
  z-index: 0;
  background:
    radial-gradient(in oklch ellipse at 78% 20%,
      oklch(0.72 0.25 330° / 0.38) 0%, transparent 48%),
    radial-gradient(in oklch ellipse at 90% 65%,
      oklch(0.75 0.18 200° / 0.42) 0%, transparent 45%),
    radial-gradient(in oklch ellipse at 12% 55%,
      oklch(0.58 0.28 275° / 0.52) 0%, transparent 52%),
    radial-gradient(in oklch ellipse at 50% 40%,
      oklch(0.62 0.20 240° / 0.25) 0%, transparent 55%),
    linear-gradient(in oklch 135deg,
      oklch(0.585 0.233 264°) 0%,
      oklch(0.62 0.20 230°) 45%,
      oklch(0.68 0.18 195°) 100%);
  clip-path: polygon(0 0, 100% 0, 100% 88%, 0 100%);
}

/* ─── Content Grid ──────────────────────────────────────── */

.hero-content {
  position: relative;
  z-index: 1;
  display: grid;
  grid-template-columns: 3fr 2fr;
  align-items: center;
  gap: clamp(2rem, 5vw, 5rem);
  padding:
    clamp(4rem, 8vw, 7rem)
    clamp(1.5rem, 5vw, 5rem)
    clamp(6rem, 12vw, 10rem)
    clamp(1.5rem, 5vw, 5rem);
  max-width: 1280px;
  margin: 0 auto;
}

/* ─── Text Block ────────────────────────────────────────── */

.hero-text {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}

.hero-headline {
  font-size: clamp(2.25rem, 4.5vw + 0.5rem, 4.5rem);
  font-weight: 800;
  line-height: 1.08;
  letter-spacing: -0.025em;
  color: #ffffff;
  margin-bottom: 1.25rem;
}

.hero-subheadline {
  font-size: clamp(1rem, 1.2vw + 0.5rem, 1.2rem);
  font-weight: 400;
  line-height: 1.65;
  color: rgba(255, 255, 255, 0.82);
  max-width: 50ch;
  margin-bottom: 2.25rem;
}

.hero-ctas {
  display: flex;
  align-items: center;
  gap: 1rem;
  flex-wrap: wrap;
}

.cta-primary {
  display: inline-flex;
  align-items: center;
  gap: 0.375rem;
  font-size: 1rem;
  font-weight: 600;
  line-height: 1;
  padding: 0.875rem 1.625rem;
  border-radius: 6px;
  background: #ffffff;
  color: oklch(0.585 0.233 264°);
  text-decoration: none;
  transition: background 0.15s ease, transform 0.1s ease;
}

.cta-primary:hover {
  background: rgba(255, 255, 255, 0.92);
  transform: translateY(-1px);
}

.cta-secondary {
  display: inline-flex;
  align-items: center;
  gap: 0.375rem;
  font-size: 1rem;
  font-weight: 500;
  line-height: 1;
  padding: 0.875rem 1.25rem;
  color: rgba(255, 255, 255, 0.88);
  text-decoration: none;
  border-radius: 6px;
  transition: color 0.15s ease, background 0.15s ease;
}

.cta-secondary:hover {
  color: #ffffff;
  background: rgba(255, 255, 255, 0.1);
}

/* ─── Screenshots ───────────────────────────────────────── */

.hero-screenshots {
  position: relative;
  transform: translateY(clamp(2rem, 5vw, 5rem));
}

.screenshot-card {
  position: absolute;
  border-radius: 12px;
  overflow: hidden;
  box-shadow:
    0 20px 60px rgba(0, 0, 0, 0.3),
    0 4px 16px rgba(0, 0, 0, 0.15);
}

.screenshot-card img {
  display: block;
  width: 100%;
  height: auto;
}

.screenshot-card--back {
  top: 0;
  left: 0;
  width: 88%;
  transform: translateX(14%) translateY(8%) rotate(-1.5deg);
  z-index: 0;
  opacity: 0.85;
}

.screenshot-card--front {
  position: relative;
  width: 100%;
  z-index: 1;
  transform: rotate(0.5deg);
}

/* ─── Responsive ────────────────────────────────────────── */

@media (max-width: 820px) {
  .hero-content {
    grid-template-columns: 1fr;
    gap: 3rem;
    padding-bottom: 4rem;
  }

  .hero-screenshots {
    transform: none;
    padding-bottom: 2rem;
  }

  .screenshot-card--back {
    display: none;
  }
}
```

---

## What each article contributed

| Concept | Where it's used |
|---|---|
| `clip-path: polygon()` | `.hero-bg` — the diagonal bottom cut |
| Polygon coordinate system | The `88%` value — calculated angle |
| Clipping vs masking | Confirmed: clip-path is right here, mask is wrong |
| Radial gradients | Each `radial-gradient()` blob layer |
| Radial gradient syntax | Every `at X% Y%` position, size keywords, stop positions |
| Background layering | The comma-separated 5-layer `background:` on `.hero-bg` |
| oklch + color spaces | `in oklch` interpolation on every gradient, oklch color values |
| CSS Grid | `.hero-content` — `grid-template-columns: 3fr 2fr` |
| fr units | The `3fr 2fr` ratio — text gets 3/5 of free space after gap |
| Grid placement | Auto-placement — both children naturally go left then right |
| Overflow + stacking contexts | `.hero-wrapper` `overflow: visible`, `.hero-bg` `z-index: 0`, `.hero-content` `z-index: 1` |
| Typography hierarchy | `clamp()` headline, weight contrast, `50ch` max-width, line-height ratios |

---

## What to try next

Now that you have the structure, experiment:

**Change the diagonal angle:** adjust the third polygon point from `100% 88%` toward `100% 75%` for a steeper slope, or `100% 95%` for a shallower one.

**Move the blobs:** change the `at X% Y%` position in any radial gradient. Watch how the mesh shifts. Try placing a blob at `at 100% 100%` — a corner accent.

**Try a different base:** swap the linear gradient for a single solid oklch color. See how the blobs read differently against a flat base vs a gradient base.

**Animate the mesh:** CSS custom properties inside `radial-gradient()` can be transitioned. Try animating a blob's position on hover with `@property` and `transition`. This is how interactive gradient backgrounds work.

**Adjust the type scale:** change the `clamp()` max from `4.5rem` to `6rem` for a more aggressive headline. Notice how `letter-spacing` and `line-height` need recalibrating when the size changes significantly.
