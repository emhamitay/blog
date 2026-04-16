---
title: "The polygon() Coordinate System — How the Numbers Actually Work"
description: "polygon() takes a list of x y coordinate pairs. Understanding the coordinate space — and how to calculate angles, percentages, and responsive shapes — is what separates copying clip-path values from actually knowing them."
date: 2026-04-16
parent: "stripe-hero/css-clip-path"
order: 1
tags: ["CSS", "clip-path", "polygon", "geometry"]
---

# The polygon() Coordinate System — How the Numbers Actually Work

When you look at a `clip-path: polygon()` value for the first time, the list of numbers looks arbitrary:

```css
clip-path: polygon(0 0, 100% 0, 100% 88%, 0 100%);
```

It's not arbitrary. Every pair of numbers is an `x y` coordinate inside the element's own coordinate space. Once you understand that space, you can write or adjust any polygon without trial and error.

---

## The coordinate space

The element has its own local coordinate system:

```
(0, 0) ─────────────────── (100%, 0)
  │                              │
  │         The element          │
  │                              │
(0, 100%) ──────────── (100%, 100%)
```

- **X axis** goes left → right. `0` is the left edge, `100%` is the right edge.
- **Y axis** goes top → bottom. `0` is the top edge, `100%` is the bottom edge.

This is the same axis direction as screen coordinates — Y increases downward, not upward like math class. This trips people up.

You can use:
- **Percentages** — relative to the element's width (for X) or height (for Y)
- **px, em, rem** — fixed sizes
- **Mix of both** — `polygon(0 0, 100% 0, 100% calc(100% - 80px), 0 100%)`

For the Stripe hero we use percentages because the hero is full-width and needs to be responsive.

---

## Reading polygon() as a path

`polygon()` takes **three or more** coordinate pairs, separated by commas. The browser draws a straight line connecting them in order, then closes the shape back to the first point automatically.

Think of it as "connect the dots" — you're placing dots on the element, and the browser draws lines between them.

```
polygon(x1 y1, x2 y2, x3 y3, x4 y4)
```

For a rectangle (same as no clip-path):
```css
polygon(0 0,      /* top-left */
        100% 0,   /* top-right */
        100% 100%, /* bottom-right */
        0 100%)   /* bottom-left */
```

For a triangle:
```css
polygon(50% 0,    /* top-center */
        100% 100%, /* bottom-right */
        0 100%)   /* bottom-left */
```

The minimum is 3 points (triangle). There's no maximum.

---

## Building the Stripe diagonal — step by step

The Stripe hero has a diagonal cut on the bottom. The top is a straight horizontal line, the left side is straight, the right side is straight, but the bottom-right corner is *higher* than the bottom-left corner. This creates the slope.

**Goal:** keep the top-left, top-right, and bottom-left corners exactly at the element's corners. Raise the bottom-right corner to create the slope.

Start with a full rectangle:
```css
polygon(0 0, 100% 0, 100% 100%, 0 100%)
```

Now raise the bottom-right point — move it from `100% 100%` to `100% 85%`:
```css
polygon(0 0, 100% 0, 100% 85%, 0 100%)
```

This means: bottom-right corner is at 85% of the element's height. Bottom-left is still at 100%. The line between them slopes upward from left to right.

**The steepness of the slope is controlled by how much you raise or lower the bottom-right Y value.** `100% 90%` = shallow slope. `100% 70%` = steep slope.

---

## Calculating the exact angle

If you want to know the angle of the Stripe slope in degrees:

- Bottom-left point: `(0, 100%)` — x=0, y=100%
- Bottom-right point: `(100%, 85%)` — x=100%, y=85%

Let's say the element is `800px` wide and `600px` tall.

In pixels:
- Bottom-left: `(0px, 600px)`
- Bottom-right: `(800px, 510px)` — 85% of 600 = 510px

Vertical difference: `600 - 510 = 90px` over `800px` horizontal distance.

```
angle = arctan(90 / 800) ≈ 6.4°
```

Stripe's angle is approximately **6–7 degrees**. Subtle — which is why it feels elegant rather than aggressive.

---

## The fill-rule — when polygons self-intersect

`polygon()` accepts an optional first argument: the fill rule.

```css
polygon(evenodd, x1 y1, x2 y2, ...)
polygon(nonzero, x1 y1, x2 y2, ...)  /* default */
```

This only matters for **self-intersecting polygons** — shapes where the lines cross each other. For the Stripe hero (a simple convex shape), the default `nonzero` rule is fine and you never need to touch this.

But if you ever try to make a polygon with a hole in it (like a donut shape), you need `evenodd` to make the inner region invisible. Worth knowing it exists.

---

## Making the polygon responsive

Since polygon coordinates use percentages by default, the shape already scales with the element. The angle stays consistent because both the element width and height scale proportionally.

But sometimes you want the **drop** to be a fixed number of pixels regardless of viewport width — so the angle becomes shallower on wide screens and steeper on narrow ones, like Stripe's actual hero:

```css
.hero {
  clip-path: polygon(0 0, 100% 0, 100% calc(100% - 80px), 0 100%);
}
```

Here the bottom-right point is always `80px` above the element's bottom, regardless of height. As the hero gets taller (more content), the `80px` drop becomes a smaller percentage, so the angle gets shallower. This is the pattern Stripe uses.

---

## Common mistakes

> 🚫 **Wrong axis direction** — Y goes *down*, not up. `0 0` is top-left, not bottom-left. If your shape is flipped vertically, this is why.

> ⚠️ **Forgetting the clip doesn't affect layout** — The element still occupies its full original height. Clip only removes paint. If the diagonal is cutting away space you still want to use, adjust padding instead.

> 💡 **Use browser DevTools to live-edit** — In Chrome/Firefox DevTools, click the small polygon icon next to a `clip-path` value to drag the points visually. This is the fastest way to find the right values before committing them to code.

> ⚠️ **Percentage X vs percentage Y** — In `polygon()`, each coordinate pair is `x y`. The X percentage is relative to element **width**, the Y percentage is relative to element **height**. They're not the same number even if the element is a square.

---

## Quick reference

| Shape | polygon() value |
|---|---|
| Full rectangle (no clip) | `polygon(0 0, 100% 0, 100% 100%, 0 100%)` |
| Triangle pointing up | `polygon(50% 0, 100% 100%, 0 100%)` |
| Triangle pointing right | `polygon(0 0, 100% 50%, 0 100%)` |
| Stripe-style diagonal (slope left→right) | `polygon(0 0, 100% 0, 100% 85%, 0 100%)` |
| Parallelogram | `polygon(10% 0, 100% 0, 90% 100%, 0 100%)` |
| Pentagon | `polygon(50% 0, 100% 38%, 82% 100%, 18% 100%, 0 38%)` |
| Fixed pixel drop diagonal | `polygon(0 0, 100% 0, 100% calc(100% - 80px), 0 100%)` |
