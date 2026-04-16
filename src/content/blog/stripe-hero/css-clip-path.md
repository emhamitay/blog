---
title: "CSS clip-path — Cutting Shapes Out of Rectangles"
description: "clip-path is the CSS property that lets you cut any element into a custom shape. Understanding it properly means understanding what 'clipping' actually does to the render tree — not just learning the polygon syntax."
date: 2026-04-16
parent: "stripe-hero"
order: 1
tags: ["CSS", "clip-path", "layout"]
---

# CSS clip-path — Cutting Shapes Out of Rectangles

Every HTML element, by default, renders as a rectangle. A `<div>` is a box. An `<img>` is a box. Even if you add `border-radius`, the element is still a rectangle internally — you're just hiding the corners visually. The actual *bounding box* is still rectangular.

`clip-path` breaks this. It lets you define a shape, and anything outside that shape becomes invisible — not hidden with `overflow: hidden`, not transparent with `opacity: 0`, but literally cut away from the painted output. The browser paints the element, and then clips the result to the shape you defined.

This is the property that creates the diagonal bottom edge of the Stripe hero.

---

## What clip-path actually does

Before learning the syntax, get the mental model right.

**Without clip-path — the problem:**

Your element is a full rectangle. If you want a diagonal bottom edge, you might try:
- A skewed `<div>` underneath (creates layout side effects, complex to maintain)
- An SVG background image (hard to animate, not responsive to content)
- A pseudo-element with `transform: skewY()` (possible but brittle)

None of these actually change the element's shape. They just create the *illusion* of a shape by layering extra things.

**With clip-path — the solution:**

```css
.hero {
  clip-path: polygon(0 0, 100% 0, 100% 85%, 0 100%);
}
```

The element is drawn in full. Then the browser cuts it — anything outside the four polygon points is removed from the final paint. No extra elements. No layout tricks. The shape *is* the element.

---

## The clipping region

The key concept: **clip-path defines a region. Anything inside the region is visible. Anything outside is invisible.**

The region is defined in the element's own coordinate space — the top-left corner is `0 0`, the bottom-right is `100% 100%`. You can use percentages (relative to element size) or fixed lengths (`px`, `em`, etc.).

This is important: the element still takes up its full layout space. If a `<section>` is 600px tall, it still occupies 600px in the document flow — even if `clip-path` is cutting away the bottom 100px visually. The surrounding elements don't "see" the clip.

> ⚠️ **Gotcha:** `clip-path` does not affect layout. The element's box model is unchanged. Only the painted output is clipped. If you need the actual space to change, you still need to adjust `padding`, `margin`, or `height`.

---

## clip-path values — the full list

`clip-path` accepts several shape functions. Here's the complete picture:

| Value | What it creates | Use when |
|---|---|---|
| `polygon(x1 y1, x2 y2, ...)` | Any polygon from 3+ points | Diagonal cuts, custom shapes |
| `circle(r at cx cy)` | A circle | Profile pictures, circular reveals |
| `ellipse(rx ry at cx cy)` | An ellipse | Oval crops |
| `inset(top right bottom left round radius)` | A rectangle with optional rounded corners | Rectangular crops with border-radius |
| `path("SVG path string")` | Any SVG path | Complex curves, logos |
| `url(#svgClipPath)` | References an SVG `<clipPath>` element | Reusable complex shapes |

For the Stripe hero, `polygon()` is the right tool — it's simple, readable, and responsive since we use percentages.

---

## How clip-path interacts with the browser

A few things that matter in practice:

**clip-path creates a stacking context.** Like `transform`, `opacity < 1`, and `filter`, applying `clip-path` on an element makes it a stacking context. This affects `z-index` layering of its children. Worth knowing before you fight mysterious z-index bugs.

**clip-path is GPU-accelerated.** The browser composites clipped elements on the GPU, so animating `clip-path` (e.g. expanding a shape on hover) is performant — as long as you use `transition` or `animation`, not JavaScript layout loops.

**clip-path clips pointer events too.** If part of your element is clipped away, that area is not clickable. An invisible clipped corner won't respond to mouse events. Useful to know, and occasionally surprising.

---

## The two sub-topics to go deep on

The `polygon()` syntax has its own article — the coordinate system and how to calculate the exact Stripe angle is worth its own treatment:

→ [The polygon() coordinate system — how the numbers actually work](./polygon-syntax)

And clipping vs masking is a separate concept that often gets confused with clip-path:

→ [Clipping vs masking — what's the difference and when to use each](./clip-vs-mask)

---

## The Stripe clip-path — preview

Once you understand polygon() coordinates fully, the Stripe hero cut is:

```css
.hero {
  /* 0 0 = top-left, 100% 0 = top-right */
  /* 100% 88% = bottom-right (slightly up) */
  /* 0 100% = bottom-left (full height) */
  clip-path: polygon(0 0, 100% 0, 100% 88%, 0 100%);
}
```

The bottom-right point is at 88% height, the bottom-left at 100% — this creates the diagonal. Read the polygon syntax article to understand *why* these numbers produce that specific angle and how to adjust it.
