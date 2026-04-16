---
title: "Radial Gradient Syntax — Every Parameter Explained"
description: "A complete reference to radial-gradient() — shape, size keywords, center position, color stops, and the less-obvious behaviors that bite you in production."
date: 2026-04-16
parent: "stripe-hero/gradient-mesh"
order: 1
tags: ["CSS", "gradients", "background"]
---

# Radial Gradient Syntax — Every Parameter Explained

The `radial-gradient()` function signature looks simple on the surface. In practice, each parameter has edge cases and interactions that aren't obvious until they break something. This is the full reference.

---

## Full signature

```css
radial-gradient(
  [ <ending-shape> || <size> ]? [ at <position> ]?,
  <color-stop> , <color-stop> [, <color-stop>]*
)
```

In plain terms:
```css
radial-gradient(shape size at x-position y-position, color1 stop1, color2 stop2)
```

Everything before the first comma is the "configuration" of the gradient. Everything after the first comma is the color list.

---

## shape — circle vs ellipse

```css
radial-gradient(circle, red, blue)
radial-gradient(ellipse, red, blue)  /* default */
```

**circle:** The gradient radiates equally in all directions. The radius is a single value (same in X and Y). Even if the element is a wide rectangle, the gradient circle stays circular.

**ellipse:** The gradient's shape matches the element's aspect ratio by default. On a wide element, the ellipse is wider than tall. This is why blobs usually look more natural with `ellipse` — they adapt to the container.

> ⚠️ You can only combine `circle` with a single size value. `ellipse` takes two values (X radius, Y radius). Mixing them wrong causes the gradient to silently fall back to default.

```css
/* ✅ Valid */
radial-gradient(circle 80px, red, blue)
radial-gradient(ellipse 80px 40px, red, blue)
radial-gradient(circle farthest-side, red, blue)

/* ❌ Invalid — circle cannot take two size values */
radial-gradient(circle 80px 40px, red, blue)
```

---

## size — the 4 keywords + explicit values

### `farthest-corner` (default)
The gradient's ending shape (where the last color stop lands) is sized so that it exactly reaches the **farthest corner** of the element from the center point.

If the center is at `50% 50%` in a 400×200px element, the farthest corner is any corner — they're all equidistant. So the gradient reaches those corners.

If the center is at `10% 10%` (near top-left), the farthest corner is bottom-right. The gradient grows large enough to reach it.

This is the most common keyword because it guarantees the gradient covers the entire element no matter where the center is.

### `closest-corner`
The gradient ends at the **closest corner** to the center. Results in a smaller, tighter gradient. The corners that are farther away will show the last color stop past the gradient edge (color clamps at the final stop value).

### `farthest-side`
Same idea as `farthest-corner` but uses the farthest **side** (not corner). Results in a slightly smaller gradient than `farthest-corner` in most cases.

### `closest-side`
Gradient ends at the closest side. For circles, this makes a circle that exactly touches one side. Useful for "spotlight" effects.

### Explicit sizes
```css
radial-gradient(circle 100px, red, blue)        /* circle, 100px radius */
radial-gradient(ellipse 200px 80px, red, blue)  /* ellipse, 200px wide, 80px tall */
radial-gradient(ellipse 50% 30%, red, blue)     /* percentages work too */
```

For the Stripe blob effect, explicit sizes give you precise control over how large each blob is, independent of element size.

---

## position — the `at` keyword

```css
radial-gradient(at 50% 50%, ...)   /* center — default */
radial-gradient(at 0 0, ...)       /* top-left corner */
radial-gradient(at top, ...)       /* top-center */
radial-gradient(at 20% 80%, ...)   /* left-ish, low */
radial-gradient(at 300px 150px, ...)  /* fixed pixel offset */
```

Position accepts the same values as `background-position`. It's the center point of the radial gradient.

**Important:** The position is relative to the element's box — the same coordinate system as `background-position`. `50% 50%` is the element's center regardless of element size.

> 💡 **For the mesh effect**, this is the main tuning knob. Moving a blob's `at` position changes where on the hero that blob appears. Positioning multiple blobs at different corners and edges builds up the layered look.

---

## Color stops — the full syntax

Color stops define where the gradient transitions happen.

```css
/* Basic: two color stops */
radial-gradient(circle, red, blue)
/* → red at center (0%), blue at edge (100%) */

/* With explicit stop positions */
radial-gradient(circle, red 0%, orange 30%, transparent 70%)
/* → red at center, orange at 30% out, transparent at 70% */
/* → from 70% to edge: stays transparent (color clamps) */

/* Overlapping stops = hard edge */
radial-gradient(circle, red 50%, blue 50%)
/* → red from center to halfway, then immediately blue — no blend */

/* Using px or em for stop positions */
radial-gradient(circle 200px, red, transparent 120px)
/* → fades from red at center to transparent at 120px out */
```

For the Stripe blob, the standard pattern is:
```css
radial-gradient(ellipse at 25% 40%, rgba(99, 102, 241, 0.45) 0%, transparent 60%)
```

- `0%` — full color (at center)
- `60%` — fully transparent (at 60% of the gradient's size)
- From 60% to edge — stays transparent

The gap between the last color stop and 100% is clamped to the last stop's value. That's why you don't need to write `transparent 60%, transparent 100%` — `transparent 60%` is enough.

---

## Less-obvious behaviors

**The center point can be outside the element.** `at -20% 50%` is valid. The gradient center is to the left of the element. You only see the part that falls within the element's bounds. This is useful for effects where you want a strong color along one edge with the gradient fading out.

**`closest-side` on a circle with a centered position produces a circle that touches all sides** — because all sides are equidistant from center. Useful for perfect inscribed-circle effects.

**Multiple radial gradients are composited with alpha blending.** When you layer several `radial-gradient()` values using multiple backgrounds, each layer's alpha value controls transparency, and the browser alpha-composites them together. A blob at `0.4` opacity lets 60% of the layers beneath show through.

**Repeating variant:** `repeating-radial-gradient()` tiles the color stops repeatedly outward. Not used for the Stripe effect but good to know it exists.

---

## Quick reference table

| Parameter | Options | Default |
|---|---|---|
| `shape` | `circle`, `ellipse` | `ellipse` |
| `size` | `farthest-corner`, `closest-corner`, `farthest-side`, `closest-side`, or `px`/`%` values | `farthest-corner` |
| `position` | Any `background-position` value | `center` (`50% 50%`) |
| Color stops | Colors with optional `%` or `px` positions | Evenly distributed |
