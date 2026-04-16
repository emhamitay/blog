---
title: "fr Units — What They Really Are"
description: "The fr unit distributes 'free space' — but free space is not the same as total space. Understanding the algorithm: how fr interacts with content-sized items, fixed tracks, min-content, and the gap — is what makes fr predictable."
date: 2026-04-16
parent: "stripe-hero/css-grid"
order: 1
tags: ["CSS", "grid", "fr", "layout"]
---

# fr Units — What They Really Are

The `fr` unit appears simple: `1fr 1fr` = two equal columns. But `fr` doesn't mean "50% of the container." It means *one fraction of the available free space*. Free space and total space are not the same thing — and this distinction explains every "why isn't this working" moment with `fr`.

---

## The algorithm: how fr is calculated

When the browser lays out a grid, it runs this process:

1. **Measure all non-fr columns first.** Fixed widths (`px`, `em`), content-sized columns (`auto`, `min-content`, `max-content`), and `minmax()` ranges are all resolved first. The browser calculates exactly how much width they consume.

2. **Subtract from the total.** Subtract the used width + all `gap` values from the container's total width. What's left is the **free space**.

3. **Distribute free space to fr columns.** Each `fr` column gets `(N / total_fr) × free_space`.

The critical insight: `fr` only distributes the *leftover* space after everything else has been satisfied.

---

## fr vs percentage: the difference

```css
/* Percentage — 50% of the container total, always */
grid-template-columns: 50% 50%;

/* fr — 50% of the FREE space after fixed items */
grid-template-columns: 1fr 1fr;
```

If the container is `1000px` and there are no fixed items, both produce `500px` columns. They look the same.

But add a gap:

```css
/* Percentage — the math breaks: 50% + 50% + 24px gap = 1024px → overflow */
grid-template-columns: 50% 50%;
gap: 24px;

/* fr — works correctly: free space is (1000 - 24) = 976px, each column = 488px */
grid-template-columns: 1fr 1fr;
gap: 24px;
```

With `fr`, the gap is subtracted from the free space before fr columns are sized. With percentages, the gap is *added on top* and the layout overflows. This is one of the main reasons to use `fr` over percentages in grid layouts.

---

## fr with mixed track types

This is where fr becomes powerful — and where it surprises people.

```css
grid-template-columns: 200px 1fr 1fr;
/* container: 1000px, gap: 0 */
/* → 200px is fixed */
/* → free space = 1000 - 200 = 800px */
/* → each 1fr = 800 / 2 = 400px */
/* result: 200px | 400px | 400px */
```

```css
grid-template-columns: 200px 1fr 2fr;
/* → free space = 800px */
/* → 1fr = 800 / 3 = 266px */
/* → 2fr = 800 × 2/3 = 533px */
/* result: 200px | 266px | 533px */
```

The ratio between fr values is what matters. `1fr 2fr` means the second column is always twice the width of the first. `3fr 2fr` means the first is 3/5 of the free space and the second is 2/5. For the Stripe hero layout, `3fr 2fr` is the ratio — text column slightly wider than screenshot column.

---

## fr with `auto` columns — the real gotcha

`auto` in a grid track means "size to content, with a maximum of the available free space." When `auto` and `fr` columns coexist, `auto` is resolved first — it takes as much space as the content needs. Then the *remaining* space is distributed to `fr`.

```css
grid-template-columns: auto 1fr;
/* 'auto' column sizes to its content */
/* '1fr' gets whatever is left */
```

This is useful for sidebar + main layouts where the sidebar is as wide as it needs to be and the main takes the rest. But it also means `fr` columns can end up unexpectedly small if `auto` columns consume a lot of space.

> ⚠️ **Never mix `auto` and `fr` if you expect `fr` to behave like a percentage.** `auto` can take a significant amount of space before `fr` sees any free space.

---

## fr with `minmax()` — the responsive grid pattern

```css
grid-template-columns: repeat(3, minmax(200px, 1fr));
```

This says: create 3 columns, each with a minimum of `200px` and a maximum of `1fr` (equal share of free space). The columns are flexible — they grow together from `200px` up to equal thirds of the container.

This is the standard pattern for card grids. Combine with `auto-fit` or `auto-fill` and you get a responsive grid without breakpoints:

```css
grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
```

- Items are at least `280px` wide
- They take up to `1fr` of free space
- The browser puts as many columns as fit, so on narrow screens there's 1 column, medium screens 2, wide screens 3 or 4

This pattern has nothing to do with the Stripe hero (which has exactly 2 fixed columns), but it's the most useful responsive grid pattern in CSS and worth knowing.

---

## fr minimum is content size

A critical detail: **fr columns cannot shrink below their content's minimum size.** The `min-width` of a grid track defaults to `auto`, which means "as wide as the narrowest content item in this column."

If you have a `1fr` column with a long unbreakable string (like a URL) or a fixed-size image, the column won't shrink below that content's width — even if `1fr` would calculate to something smaller.

To override this:
```css
grid-template-columns: minmax(0, 1fr) minmax(0, 1fr);
/* OR */
grid-template-columns: 1fr 1fr;
/* + set overflow: hidden or min-width: 0 on the grid item */
```

The `minmax(0, 1fr)` pattern explicitly sets the minimum to `0`, allowing the column to shrink fully. This is the fix for when a `1fr` column doesn't shrink as expected.

---

## Quick reference

| Declaration | What it does |
|---|---|
| `1fr 1fr` | Two equal columns sharing free space |
| `3fr 2fr` | Two columns in 3:2 ratio |
| `200px 1fr` | Fixed sidebar, flexible main |
| `auto 1fr` | Content-sized sidebar, flexible main |
| `minmax(0, 1fr)` | Flexible column that can shrink to zero |
| `repeat(3, 1fr)` | Three equal columns |
| `repeat(auto-fit, minmax(280px, 1fr))` | Responsive card grid |
