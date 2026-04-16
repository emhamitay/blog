---
title: "Building the Stripe Hero"
description: "Reverse-engineering one of the most iconic hero sections on the web — the diagonal gradient mesh, the clip-path tilt, the grid layout, and the typography hierarchy that makes it feel premium."
date: 2026-04-16
tags: ["CSS", "layout", "clip-path", "gradients", "typography"]
order: 1
---

# Building the Stripe Hero

If you look at [stripe.com](https://stripe.com), the first thing you see is one of the most studied hero sections on the web. It looks deceptively simple — some text, a diagonal colored background, a couple of UI screenshots. But underneath that, there are at least five distinct CSS concepts working in harmony, each one something a lot of developers have "seen" but never deeply understood.

This is not a tutorial where you copy the code. This is a deep map — we break each concept into its own article, take it from nothing to real understanding, and then at the end we assemble them all into the actual Stripe-like hero. When you're done, you'll *understand* why it looks the way it does, not just how to reproduce it.

---

## What's actually in the Stripe hero?

Here's a fast visual inventory of what's happening:

1. **The diagonal tilt** — the background isn't a rectangle. It's been cut with `clip-path: polygon()` to create that angled bottom edge.
2. **The gradient mesh** — it's not a flat color. It's a layered radial gradient system that produces those soft, blended color blobs (blue, purple, cyan, pink) behind the content.
3. **The layout** — the text and the right-side UI screenshots sit in a grid. It's not flexbox. Understanding why grid is the right tool here is its own lesson.
4. **The stacking and overflow** — the UI screenshots on the right bleed *outside* their container, overlapping the section below. This is deliberate and requires knowing how `overflow`, `z-index`, and stacking contexts work.
5. **The typography hierarchy** — the headline, sub-headline, the CTA buttons, and the supporting text each have a very specific role. The size jumps, weights, and line-height aren't random.

---

## The map of articles

Each concept below is a standalone article — read them in any order, but the foundations come first.

### 1. [CSS clip-path — Cutting Shapes Out of Rectangles](./stripe-hero/css-clip-path)
The diagonal cut at the bottom of the Stripe hero is done with `clip-path: polygon()`. This article teaches the full concept from scratch — what clip-path actually does to the render tree, the polygon coordinate system, and how to build the exact Stripe angle.

- [The polygon() coordinate system — how the numbers actually work](./stripe-hero/css-clip-path/polygon-syntax)
- [Clipping vs masking — what's the difference and when to use each](./stripe-hero/css-clip-path/clip-vs-mask)

### 2. [CSS Radial Gradients — Building the Mesh](./stripe-hero/gradient-mesh)
The colored blob effect is a layered set of `radial-gradient()` backgrounds stacked on one element. This article explains radial gradients from the math up — center point, shape, size keywords, color stops — then builds the Stripe mesh layer by layer.

- [Radial gradient syntax — every parameter explained](./stripe-hero/gradient-mesh/radial-syntax)
- [Layering multiple backgrounds — the stacking order](./stripe-hero/gradient-mesh/background-layering)
- [Color spaces and oklch — why Stripe colors look so smooth](./stripe-hero/gradient-mesh/color-spaces)

### 3. [CSS Grid — The Two-Column Hero Layout](./stripe-hero/css-grid)
The text-left / screenshots-right split is CSS grid. This article covers grid from the track model up — what `fr` actually means, explicit vs implicit tracks, and how to align the Stripe layout responsively.

- [fr units — what they really are](./stripe-hero/css-grid/fr-units)
- [Grid placement — named lines vs span](./stripe-hero/css-grid/placement)

### 4. [Overflow, z-index, and Stacking Contexts](./stripe-hero/layout-composition)
The UI screenshots on the right side of the Stripe hero bleed outside their parent. That requires understanding what `overflow: visible` means in context, how stacking contexts are created, and how to control what appears on top of what without fighting the browser.

### 5. [Typography Hierarchy — Why the Text Feels Premium](./stripe-hero/typography)
The Stripe headline uses a very specific type scale. This article covers the principles: contrast ratio between heading sizes, optical sizing, line-height math, and how to make a `clamp()` responsive headline that doesn't break on mobile.

---

## [Putting it all together](./stripe-hero/putting-it-all-together)

Once you've read through the individual concepts, the final article assembles all of them into a single HTML/CSS implementation — the full Stripe-like hero, built from scratch, every line traced back to a concept from the sub-articles.

The table at the end maps each CSS declaration to the article that explains it. If any line surprises you, there's exactly one article to re-read.
