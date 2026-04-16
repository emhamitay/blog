---
title: "Layering Multiple Backgrounds — The Stacking Order"
description: "CSS background can take multiple comma-separated values — gradients, images, or both. Understanding the stacking order and how each layer's properties work independently is what makes complex background compositions possible."
date: 2026-04-16
parent: "stripe-hero/gradient-mesh"
order: 2
tags: ["CSS", "background", "gradients", "layering"]
---

# Layering Multiple Backgrounds — The Stacking Order

The Stripe mesh effect is three radial gradients and a linear gradient all sitting on a single element. That's four `background` layers. CSS supports this natively — you can comma-separate any number of gradient or image values in the `background` property.

Understanding how the layers stack and how to control each one independently is the key skill here.

---

## The basic syntax

```css
.element {
  background:
    url('top-image.png'),         /* layer 1 — on top */
    radial-gradient(red, blue),   /* layer 2 */
    linear-gradient(135deg, #333, #666); /* layer 3 — on bottom */
}
```

The golden rule: **the first item in the list is the topmost layer.** The last item is the bottommost layer. This is the opposite of how most stacking systems feel (you might expect the "first" to be the base), so it trips people up.

Think of it as the CSS telling you what's *painted first* vs *painted on top*. The last layer is painted first (it's the base). Each earlier layer is painted on top.

---

## Why this matters for the Stripe mesh

```css
background:
  radial-gradient(ellipse at 75% 55%, rgba(232, 121, 249, 0.35) 0%, transparent 45%),  /* pink blob — top */
  radial-gradient(ellipse at 85% 15%, rgba(56, 189, 248, 0.4) 0%, transparent 50%),    /* cyan blob */
  radial-gradient(ellipse at 15% 60%, rgba(139, 92, 246, 0.5) 0%, transparent 55%),    /* purple blob */
  linear-gradient(135deg, #6366f1 0%, #3b82f6 60%, #0ea5e9 100%);                      /* base — bottom */
```

The linear gradient is the solid base color. The radial blobs are on top of it. Each blob uses `rgba()` with alpha — so the layers beneath show through. The topmost blob shows through to the blobs below it, which show through to the linear base. This is what creates the blended mesh look.

If the blobs were opaque instead of transparent, each one would completely hide the layers beneath it. No mesh, just colored rectangles.

---

## Independent background properties per layer

Each background layer can have its own `background-size`, `background-position`, `background-repeat`, and `background-origin`. You control these with comma-separated values in the same order as the layers.

```css
.element {
  background-image:
    url('logo.png'),
    radial-gradient(red, blue);
  background-size:
    100px 100px,   /* for logo.png */
    cover;         /* for radial-gradient */
  background-position:
    top right,     /* for logo.png */
    center;        /* for radial-gradient */
  background-repeat:
    no-repeat,     /* for logo.png */
    no-repeat;     /* for radial-gradient */
}
```

The number of values must match the number of layers, or CSS wraps around (cycles through the list). For gradients specifically, `background-size` defaults to the element's full size and `background-position` defaults to `0 0` (top-left), so you usually don't need to override them.

> ⚠️ **The shorthand `background:` resets all sub-properties.** If you set `background-size` separately after a `background:` shorthand, it works. But if you use the `background:` shorthand again later, it resets `background-size` back to default. Order of declarations matters.

---

## Gradients vs images in layers

Both CSS gradients (`linear-gradient`, `radial-gradient`, `conic-gradient`) and image URLs (`url()`) work as background layers. They're all "images" from CSS's perspective.

**Gradients don't have an intrinsic size.** They fill the element (or whatever `background-size` tells them) by default. This is why `background-size: cover` and `background-size: 100% 100%` mean the same thing for gradients — they already fill the space.

**Images (url()) have intrinsic dimensions.** A 200×100px image, by default, is placed at `0 0` and repeats. You need to set `background-size: cover` or `background-size: contain` if you want it to scale to the element.

---

## The final base color — `background-color`

One more layer exists that doesn't appear in the `background-image` list: `background-color`. It's always the absolute bottommost layer — below all gradient and image layers.

```css
.element {
  background-color: #0a0a1a;  /* darkest base */
  background-image:
    radial-gradient(ellipse at 20% 50%, rgba(99, 102, 241, 0.4) 0%, transparent 60%),
    radial-gradient(ellipse at 80% 80%, rgba(56, 189, 248, 0.3) 0%, transparent 55%);
}
```

This is useful when your topmost gradient layers don't fully cover the element (e.g., they're positioned blobs). The `background-color` fills the uncovered areas.

For the Stripe mesh, using a `background-color` as the base instead of a linear gradient is a valid simplification — the blobs appear on top of a solid base.

---

## Managing complex multi-layer backgrounds

When you have 4+ layers, the CSS can get hard to read. A few patterns help:

**Custom properties per layer:**
```css
:root {
  --blob-purple: radial-gradient(ellipse at 15% 60%, rgba(139, 92, 246, 0.5) 0%, transparent 55%);
  --blob-cyan:   radial-gradient(ellipse at 85% 15%, rgba(56, 189, 248, 0.4) 0%, transparent 50%);
  --blob-pink:   radial-gradient(ellipse at 75% 55%, rgba(232, 121, 249, 0.35) 0%, transparent 45%);
  --base-grad:   linear-gradient(135deg, #6366f1 0%, #3b82f6 60%, #0ea5e9 100%);
}

.hero {
  background: var(--blob-pink), var(--blob-cyan), var(--blob-purple), var(--base-grad);
}
```

This makes the stacking order readable and lets you easily reuse or recombine layers.

---

## Quick reference

| Concept | Rule |
|---|---|
| Stacking order | First in list = topmost layer |
| Transparency | Use `rgba()` alpha to let lower layers show through |
| Per-layer control | Comma-separate `background-size`, `background-position`, etc. to match layers |
| Solid base | `background-color` is always below all image/gradient layers |
| Number of layers | No hard limit — browser performance degrades beyond ~8–10 layers |
| Gradients vs images | Both work as layers; gradients default to full-size fill |
