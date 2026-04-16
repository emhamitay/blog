---
title: "CSS Grid — The Two-Column Hero Layout"
description: "The Stripe hero's text-left / screenshots-right layout is CSS Grid. Understanding grid from the track model up — fr units, explicit vs implicit tracks, alignment — is what makes this kind of layout feel effortless instead of fought against."
date: 2026-04-16
parent: "stripe-hero"
order: 3
tags: ["CSS", "grid", "layout"]
---

# CSS Grid — The Two-Column Hero Layout

The Stripe hero has two main columns: the left side contains the headline, sub-text, and CTA buttons. The right side has a stack of UI screenshots. At some viewport width, they collapse to a single column on mobile.

This is CSS grid. Specifically, it's a two-column grid where the columns have different sizes, and the content is placed into those columns with no explicit placement needed — it flows naturally.

If you've used flexbox for everything, this article will shift your mental model. Grid and flexbox solve different problems, and this layout is where grid wins.

---

## Why grid and not flexbox?

Flexbox works along **one axis** — either a row or a column, and the flex items share space in that axis. The layout is *content-driven* — the items tell the container how much space they need, and the container distributes the remaining space.

Grid works in **two axes simultaneously**. You define the tracks (columns and rows) on the container, and then items are placed *into those tracks*. The layout is *container-driven* — the container defines the space, and items fill it.

For the Stripe hero:
- The left column needs to be flexible and text-driven — it grows with content
- The right column needs to be a specific proportion of the viewport, or take up the remaining space
- The two columns need to maintain their size relationship regardless of content

Grid is the right tool because we're defining *structure* first and placing content into it. Flexbox would require nesting, percentage hacks, and min-width fights to achieve the same result.

---

## The basic grid setup

```css
.hero {
  display: grid;
  grid-template-columns: 1fr 1fr;  /* two equal columns */
  gap: 4rem;
  padding: 6rem 4rem;
  align-items: center;
}
```

`display: grid` activates grid on the container. `grid-template-columns: 1fr 1fr` creates two equal columns. Every direct child of `.hero` is automatically a grid item and gets placed into cells.

The first child (the text block) goes into column 1. The second child (the screenshots block) goes into column 2. That's the layout.

But Stripe's two columns aren't equal. The text column is slightly wider. The screenshots column is slightly narrower. The actual Stripe ratio is closer to:

```css
grid-template-columns: 1.2fr 0.8fr;
/* or equivalently: */
grid-template-columns: 3fr 2fr;
```

The `fr` unit is the key to understanding this. See the sub-article for a complete explanation of what `fr` actually is.

---

## grid-template-columns — the full syntax

You can define columns with any combination of values:

```css
/* Fixed widths */
grid-template-columns: 300px 600px;

/* fr units — distribute remaining space */
grid-template-columns: 1fr 2fr;        /* 1/3 + 2/3 */

/* Mix of fixed and flexible */
grid-template-columns: 250px 1fr;      /* sidebar + flexible main */

/* repeat() — shorthand for repeating patterns */
grid-template-columns: repeat(3, 1fr); /* three equal columns */
grid-template-columns: repeat(4, minmax(200px, 1fr)); /* responsive grid */

/* Named lines */
grid-template-columns: [sidebar-start] 250px [sidebar-end content-start] 1fr [content-end];
```

For the Stripe hero, `1.2fr 0.8fr` (or `3fr 2fr`) is the correct tool — two flexible columns in a specific ratio.

---

## gap — the space between tracks

```css
gap: 4rem;           /* same gap between columns and rows */
column-gap: 4rem;    /* only between columns */
row-gap: 2rem;       /* only between rows */
```

`gap` is cleaner than margins on grid items because it only applies *between* cells — never on the outer edges. You don't fight the "first/last item no margin" problem that flexbox layouts often have.

---

## align-items — vertical alignment within cells

```css
.hero {
  align-items: center;   /* vertically center items in their cells */
}
```

In the Stripe hero, the text block and the screenshots block might have different heights. `align-items: center` vertically centers both within their respective cells, so neither looks pinned to the top while the other is shorter.

Options:

| Value | What it does |
|---|---|
| `stretch` | Items stretch to fill cell height. Default. |
| `center` | Items centered vertically in cell. |
| `start` | Items aligned to top of cell. |
| `end` | Items aligned to bottom of cell. |
| `baseline` | Items aligned on their first text baseline. |

---

## The responsive collapse

On mobile, the two columns should stack vertically. The standard pattern:

```css
.hero {
  display: grid;
  grid-template-columns: 3fr 2fr;
  gap: 4rem;
  align-items: center;
}

@media (max-width: 768px) {
  .hero {
    grid-template-columns: 1fr;  /* single column */
    gap: 2rem;
  }
}
```

When `grid-template-columns: 1fr`, there's only one column. Both children stack vertically — text on top, screenshots below. No additional changes needed.

---

## Sub-topics in this section

→ [fr units — what they really are](./fr-units)
The `fr` unit is not just "remaining space divided equally." Understanding the algorithm for how `fr` distributes space — including how it interacts with `min-content`, fixed sizes, and content-sized items — is what lets you use it predictably.

→ [Grid placement — named lines vs span](./placement)
By default, grid items are auto-placed. But you can take explicit control with `grid-column`, `grid-row`, `grid-area`, named lines, and span values. For more complex hero layouts where the screenshots overlap between two cells, explicit placement is required.

---

## The complete hero grid

```css
.hero {
  display: grid;
  grid-template-columns: 3fr 2fr;
  gap: clamp(2rem, 5vw, 6rem);  /* responsive gap */
  align-items: center;
  padding: clamp(4rem, 10vw, 8rem) clamp(1.5rem, 5vw, 4rem);
  min-height: 80vh;
}

@media (max-width: 800px) {
  .hero {
    grid-template-columns: 1fr;
    gap: 3rem;
  }

  /* On mobile, screenshots go below the text */
  .hero__screenshots {
    order: 2;
  }
}
```

The `clamp()` on padding and gap means these values scale smoothly with viewport width — no abrupt jumps between breakpoints. This is the Stripe approach to responsive design.
