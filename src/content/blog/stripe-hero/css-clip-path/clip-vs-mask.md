---
title: "Clipping vs Masking — What's the Difference and When to Use Each"
description: "clip-path and mask are both about making parts of an element invisible — but they work differently, support different shapes, and have different performance characteristics. This article draws the line clearly."
date: 2026-04-16
parent: "stripe-hero/css-clip-path"
order: 2
tags: ["CSS", "clip-path", "mask", "rendering"]
---

# Clipping vs Masking — What's the Difference and When to Use Each

Both `clip-path` and `mask` make parts of an element invisible. The surface-level goal is the same. But the mechanism, the kind of shapes they support, and when to reach for each are completely different.

This article exists because people often discover `mask` after learning `clip-path` and get confused about which tool to use. They're not interchangeable.

---

## The core distinction

**Clipping** is binary. Every pixel is either fully visible (inside the clip region) or fully invisible (outside it). No partial transparency. Hard edges only.

**Masking** uses a *mask image* where each pixel's **luminance or alpha** controls how transparent the corresponding element pixel is. A white mask pixel = fully visible. A black mask pixel = fully invisible. A gray mask pixel = partially transparent. Gradients in the mask produce smooth fade-outs.

```
Clipping:  pixel is IN or OUT. Binary.
Masking:   pixel is fully visible, fully invisible, or anything in between.
```

---

## clip-path

```css
.element {
  clip-path: polygon(0 0, 100% 0, 100% 88%, 0 100%);
}
```

You define a geometric shape. The browser uses it as a hard boundary. Everything inside: fully painted. Everything outside: gone.

**Supports:**
- `polygon()` — any polygon from points
- `circle()` — a circle
- `ellipse()` — an ellipse
- `inset()` — a rectangle with optional rounded corners
- `path()` — an SVG path (complex curves)
- `url(#svgClipPath)` — references an SVG `<clipPath>` element

**Always hard edges.** You cannot make a soft, feathered edge with `clip-path`. If you need a fade, you need `mask`.

**Performance:** GPU-accelerated. Smooth to animate.

---

## mask

```css
.element {
  mask-image: linear-gradient(to bottom, black 60%, transparent 100%);
}
```

The `mask-image` can be any image — a PNG with transparency, an SVG, or a CSS gradient. The browser uses the image's alpha (or luminance, depending on `mask-mode`) to determine transparency.

**Key properties:**

| Property | What it controls |
|---|---|
| `mask-image` | The source image (gradient, url, etc.) |
| `mask-mode` | `alpha` (use alpha channel) or `luminance` (use brightness) |
| `mask-size` | Like `background-size` — `cover`, `contain`, px, % |
| `mask-position` | Like `background-position` |
| `mask-repeat` | Like `background-repeat` |
| `mask-composite` | How multiple mask layers combine (`add`, `subtract`, `intersect`, `exclude`) |

**Supports:** Anything that can be an image — gradients, PNGs, SVGs, even CSS-generated images via `element()` in Firefox.

**Allows soft, feathered edges.** A gradient mask produces smooth fade-outs. This is what `clip-path` cannot do.

> ⚠️ **Browser prefix:** `mask` properties still need `-webkit-` prefix in most cases for WebKit-based browsers. Write: `mask-image: ...` and `-webkit-mask-image: ...` together, or use the shorthand `mask: ...` with `-webkit-mask: ...`.

---

## Side-by-side: when to use each

| Situation | Use |
|---|---|
| Hard geometric shape (diagonal, polygon, circle) | `clip-path` |
| Soft fade-out edge | `mask` with gradient |
| Reveal animation on a shape | `clip-path` (animates cleanly) |
| Fade an image into its background | `mask` with gradient |
| Complex curved shape (SVG path) | Either — `clip-path: path()` or SVG `mask` |
| Text masking / text revealing | `mask` |
| Performance matters, simple shape | `clip-path` |

---

## Common mistake: using clip-path when you needed mask

```css
/* ❌ This gives a hard diagonal cut — the text will be cut off sharply */
.hero-text {
  clip-path: polygon(0 0, 100% 0, 100% 80%, 0 100%);
}

/* ✅ This fades the text out softly toward the bottom */
.hero-text {
  mask-image: linear-gradient(to bottom, black 60%, transparent 100%);
  -webkit-mask-image: linear-gradient(to bottom, black 60%, transparent 100%);
}
```

---

## For the Stripe hero

The Stripe hero uses `clip-path` on the section container — the diagonal cut is a hard geometric shape, no softness needed. The background is cut cleanly.

If you wanted the Stripe mesh gradient to *fade* into the white background below instead of cutting sharply, you'd switch to a mask on the section. Different visual goal, different tool.

The choice always comes back to the mental model: **hard edge → clip-path. Soft/variable transparency → mask.**
