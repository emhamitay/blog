---
title: "CSS Radial Gradients — Building the Mesh"
description: "The Stripe background isn't a flat color. It's a stack of radial gradients layered on one element, each positioned to create a soft colored blob. Understanding radial gradients from the math up is what lets you build and control this kind of effect."
date: 2026-04-16
parent: "stripe-hero"
order: 2
tags: ["CSS", "gradients", "background", "color"]
---

# CSS Radial Gradients — Building the Mesh

If you inspect the Stripe hero's background in DevTools, you'll find something like this on the section element:

```css
background:
  radial-gradient(ellipse at 20% 50%, rgba(88, 80, 236, 0.4) 0%, transparent 60%),
  radial-gradient(ellipse at 80% 20%, rgba(56, 189, 248, 0.35) 0%, transparent 55%),
  radial-gradient(ellipse at 60% 80%, rgba(232, 121, 249, 0.3) 0%, transparent 50%),
  linear-gradient(135deg, #6366f1 0%, #3b82f6 50%, #06b6d4 100%);
```

That's four gradient layers stacked on a single element. The bottom layer is a linear gradient base — the main background color. The three radial gradients on top are the soft colored "blobs" — the mesh effect.

This article teaches the mechanics of `radial-gradient()` fully, so you can build and customize this kind of effect from scratch.

---

## What a radial gradient is

A linear gradient draws color along a straight line from point A to point B. A radial gradient **radiates outward from a center point** — like light spreading from a lamp. The color at the center transitions to the color at the edge.

The simplest case:
```css
background: radial-gradient(red, blue);
```

This puts red at the center of the element, fades to blue at the edges.

For the Stripe mesh, we use this to create soft "blobs" of color that fade to transparent — placing each one at a specific position to build up the layered effect.

---

## The radial-gradient() syntax — fully broken down

```css
radial-gradient(
  [shape] [size] at [position],
  color-stop-1,
  color-stop-2,
  ...
)
```

Every part is optional except the color stops. Defaults kick in for anything omitted.

### shape
`circle` or `ellipse`. Default is `ellipse`.

- `circle` — equal radius in all directions. Always perfectly circular.
- `ellipse` — different X and Y radii. The default, and more useful for blob effects because the element's own aspect ratio influences the shape.

### size
How large the gradient is. Controls where the last color stop is reached.

| Value | What it does |
|---|---|
| `farthest-corner` | Gradient extends to the corner of the element farthest from center. **Default.** |
| `closest-corner` | Gradient ends at the closest corner. Tighter gradient. |
| `farthest-side` | Gradient ends at the farthest side edge. |
| `closest-side` | Gradient ends at the closest side. Useful for circles touching one side. |
| `200px 150px` | Explicit X radius and Y radius (ellipse only). |
| `80px` | Explicit radius (circle only). |

For the Stripe blob effect, size keywords like `farthest-corner` work well because the blobs naturally reach across the section.

### position (the `at` keyword)
Where the center of the gradient sits. Uses the same syntax as `background-position`.

```css
at 50% 50%    /* center — default */
at 20% 80%    /* left-ish, low */
at top left   /* top-left corner */
at 300px 200px /* fixed offset from top-left */
```

This is the most important parameter for building the mesh. Each radial gradient layer in the Stripe background is positioned at a different point, creating blobs in different areas.

### color stops
Same as linear gradient color stops:

```css
radial-gradient(circle, red 0%, blue 100%)
radial-gradient(circle, red, orange 40%, transparent 70%)
```

For the mesh effect, the pattern is always:
```css
radial-gradient(ellipse at X% Y%, rgba(r,g,b, alpha) 0%, transparent 60%)
```

- Start with a semi-transparent color at the center
- Fade to fully transparent at ~50–70% of the gradient size

The transparency is what lets the layers underneath show through, creating the blended mesh look.

---

## Building the Stripe mesh step by step

Start with the base — a linear gradient for the overall background color:

```css
background: linear-gradient(135deg, #6366f1 0%, #3b82f6 60%, #0ea5e9 100%);
```

This gives you the deep blue-to-cyan base. Now add the first blob — a purple one in the lower-left area:

```css
background:
  radial-gradient(ellipse at 15% 60%, rgba(139, 92, 246, 0.5) 0%, transparent 55%),
  linear-gradient(135deg, #6366f1 0%, #3b82f6 60%, #0ea5e9 100%);
```

Add a cyan blob in the upper-right:

```css
background:
  radial-gradient(ellipse at 85% 15%, rgba(56, 189, 248, 0.4) 0%, transparent 50%),
  radial-gradient(ellipse at 15% 60%, rgba(139, 92, 246, 0.5) 0%, transparent 55%),
  linear-gradient(135deg, #6366f1 0%, #3b82f6 60%, #0ea5e9 100%);
```

Add a pink blob in the middle-right:

```css
background:
  radial-gradient(ellipse at 75% 55%, rgba(232, 121, 249, 0.35) 0%, transparent 45%),
  radial-gradient(ellipse at 85% 15%, rgba(56, 189, 248, 0.4) 0%, transparent 50%),
  radial-gradient(ellipse at 15% 60%, rgba(139, 92, 246, 0.5) 0%, transparent 55%),
  linear-gradient(135deg, #6366f1 0%, #3b82f6 60%, #0ea5e9 100%);
```

This is the mesh. Each layer adds one blob. The order matters: later layers in the list are painted first (lower in the stack). The first item in the list is on top.

---

## Tuning the effect

The variables that control the look:

**Alpha value of the `rgba()`** — Lower = more transparent blob, more subtle. Stripe uses 0.3–0.5 range for most blobs.

**The "stop" percentage** — `transparent 60%` means the blob fades to fully transparent by 60% of its total size. Lower = smaller, tighter blob. Higher = blob spreads across more of the element.

**The `at` position** — Move blobs to overlap more for a blended look, or spread them apart for distinct color zones.

**The color** — Stripe uses colors that share a hue family (blues, purples, cyans) so they blend without muddying. Complementary colors (opposite on the color wheel) layered this way would look brown.

---

## Sub-topics in this section

→ [Radial gradient syntax — every parameter explained](./radial-syntax)
Goes deeper on each parameter with all the edge cases and the less-obvious behaviors.

→ [Layering multiple backgrounds — the stacking order](./background-layering)
The CSS `background` shorthand can take multiple values. How the stacking order works, what the limits are, and how to manage complex multi-layer declarations.

→ [Color spaces and oklch — why Stripe colors look so smooth](./color-spaces)
Stripe uses `oklch()` colors in their actual CSS, not `rgba()`. Understanding perceptual color spaces explains why their gradients look smoother than hex-color gradients — and how to use oklch yourself.
