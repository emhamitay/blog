---
title: "Overflow, z-index, and Stacking Contexts"
description: "The Stripe UI screenshots bleed outside their hero section, overlapping the content below. Making this work requires understanding overflow, stacking contexts, and z-index — three CSS concepts that interact in non-obvious ways."
date: 2026-04-16
parent: "stripe-hero"
order: 4
tags: ["CSS", "overflow", "z-index", "stacking-context", "layout"]
---

# Overflow, z-index, and Stacking Contexts

The Stripe hero has UI screenshots on the right side that extend *below* the diagonal cut, visually overlapping the section below. This effect is not an SVG trick or an image crop. It's a combination of three CSS concepts working together: `overflow: visible` on the right container, `position: relative` on the hero, and `z-index` to control what renders on top.

Understanding these three things deeply — especially how stacking contexts work — is what separates developers who fight layout bugs from developers who understand what's happening.

---

## overflow — what it really controls

`overflow` controls what happens to content that extends outside its element's box.

```css
overflow: visible;  /* content renders outside the box — default */
overflow: hidden;   /* content outside the box is clipped, not rendered */
overflow: scroll;   /* always shows scrollbars, clips overflow */
overflow: auto;     /* scrollbars appear when needed, clips overflow */
overflow: clip;     /* clips like hidden but no scroll mechanism */
```

The default is `visible`. This means: by default, content *can* extend outside any element and it will render correctly. This is not a bug — it's by design.

The Stripe screenshots bleeding below the hero section are just content with `overflow: visible` on their container (or more precisely, no `overflow: hidden` blocking them). The screenshots are positioned so they extend beyond the element bounds, and because there's no `overflow: hidden` in the ancestor chain, they render visibly.

---

## What blocks overflow: visible

**Here's the critical rule:** `overflow: visible` only works if none of the ancestors have `overflow: hidden` (or `scroll`, or `auto`, or `clip`).

If any ancestor in the chain has `overflow: hidden`, all descendants are clipped at that ancestor's boundary — even if those descendants have `overflow: visible` themselves.

```html
<section class="hero">           <!-- overflow: hidden ❌ blocks the bleed -->
  <div class="screenshots">      <!-- overflow: visible — doesn't matter -->
    <img class="screenshot" />   <!-- this gets clipped at .hero boundary -->
  </div>
</section>
```

For the Stripe bleed effect, the `.hero` section must NOT have `overflow: hidden`. It should be `overflow: visible` (the default). The screenshots can then extend outside the hero's painted area.

---

## How the Stripe bleed is set up

```css
.hero {
  position: relative;
  overflow: visible;  /* default — don't change this */
  /* clip-path handles the visual shape of the background */
  clip-path: polygon(0 0, 100% 0, 100% 88%, 0 100%);
}

.hero__screenshots {
  position: relative;
  /* screenshots extend below the hero's 88% clip line */
  /* but they're not clipped because the hero doesn't have overflow: hidden */
}
```

Wait — doesn't `clip-path` on `.hero` clip the screenshots too?

Yes it does, which is the issue. In the real Stripe implementation, the screenshots are likely in a separate element that is a sibling of the hero section, positioned absolutely relative to a parent wrapper — not a child of the clipped hero. Or the screenshots are children, but the clip-path is on a pseudo-element background layer, not the section element itself.

The cleaner approach: put the gradient background in a `::before` pseudo-element and apply `clip-path` to that, leaving the section itself unclipped:

```css
.hero {
  position: relative;
  /* No clip-path here */
}

.hero::before {
  content: '';
  position: absolute;
  inset: 0;
  z-index: 0;
  background: /* the gradient mesh */;
  clip-path: polygon(0 0, 100% 0, 100% 88%, 0 100%);
}

.hero__content {
  position: relative;
  z-index: 1;  /* above the ::before background */
}
```

Now the background is clipped, but the content (including screenshots) is not. The screenshots can freely extend below the hero.

---

## Stacking contexts — the real z-index explanation

`z-index` controls the paint order of elements on the Z axis (toward the viewer). Higher `z-index` = painted on top. But `z-index` only works between elements that share the same **stacking context**.

A stacking context is a scope. Elements within a stacking context are z-ordered *among themselves*, independently from elements outside. The stacking context as a whole has its own position in its parent context.

**What creates a stacking context:**
- `position: relative/absolute/fixed/sticky` + any `z-index` value other than `auto`
- `opacity` less than 1
- `transform` (any value except `none`)
- `filter` (any value except `none`)
- `clip-path` (any value except `none`)
- `will-change` with any of the above values
- `isolation: isolate`
- Others: `mix-blend-mode`, `perspective`, `contain`

> ⚠️ **clip-path creates a stacking context.** This means that when you apply `clip-path` to `.hero`, it becomes a stacking context. Every child's `z-index` is resolved only within `.hero` — not compared to elements outside it. This is a subtle but important consequence.

---

## The z-index trap — when z-index "doesn't work"

The most common z-index confusion:

```html
<section class="hero" style="z-index: 1; position: relative; transform: rotate(0)">
  <!-- transform creates stacking context -->
  <div style="z-index: 999">This text</div>
</section>

<div style="z-index: 2; position: relative">This covers the hero</div>
```

The `<div>` with `z-index: 2` will render on top of everything inside `.hero` — including the `z-index: 999` div — because the entire `.hero` stacking context has a lower z-index (`1`) than the sibling div (`2`). The internal `z-index: 999` is meaningless outside of `.hero`'s context.

This is why increasing `z-index` to absurd numbers (`z-index: 9999`) often "doesn't work" — the element is in a stacking context that itself has a lower z-index than the element you're trying to cover.

**The fix:** trace the stacking context hierarchy. Usually the problem is a parent with `transform`, `opacity`, or `filter` creating an unintended stacking context.

---

## The Stripe bleed — complete implementation

```css
/* Wrapper — no overflow: hidden, creates positioning context */
.hero-wrapper {
  position: relative;
  overflow: visible;
}

/* Background — clipped, below the content */
.hero-wrapper::before {
  content: '';
  position: absolute;
  inset: 0;
  z-index: 0;
  background:
    radial-gradient(ellipse at 20% 50%, oklch(0.58 0.25 278° / 0.5) 0%, transparent 55%),
    linear-gradient(135deg, oklch(0.585 0.233 264°) 0%, oklch(0.68 0.18 195°) 100%);
  clip-path: polygon(0 0, 100% 0, 100% 88%, 0 100%);
}

/* Content grid — above background, not clipped */
.hero-content {
  position: relative;
  z-index: 1;
  display: grid;
  grid-template-columns: 3fr 2fr;
  align-items: center;
  gap: 4rem;
  padding: 6rem 4rem 10rem;  /* extra bottom padding for bleed space */
}

/* Screenshots — extend below the 88% clip line visually */
.hero-screenshots {
  position: relative;
  transform: translateY(60px);  /* push down into overlap territory */
}
```

The screenshots extend visually into the section below because `.hero-wrapper` has `overflow: visible` (default) and the `clip-path` is only on the `::before` pseudo-element. The `z-index: 1` on `.hero-content` ensures the screenshots render above the white background section that follows.

---

## Quick reference

| Concept | Rule |
|---|---|
| `overflow: visible` | Default — content can extend outside the box |
| Ancestors block overflow | Any ancestor with `overflow: hidden/scroll/auto/clip` clips all descendants |
| clip-path creates stacking context | Children's z-index is scoped inside the clipped element |
| z-index scope | z-index only compares elements within the same stacking context |
| Stacking context creators | transform, opacity < 1, filter, clip-path, position + z-index, isolation |
| "z-index doesn't work" fix | Find the ancestor creating the stacking context, adjust its z-index |
