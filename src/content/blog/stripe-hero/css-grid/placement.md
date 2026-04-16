---
title: "Grid Placement — Named Lines vs Span"
description: "Auto-placement puts items into the next available cell. Explicit placement lets you put any item into any cell — spanning multiple columns, overlapping cells, or naming grid lines for semantic placement. This is what enables complex hero layouts."
date: 2026-04-16
parent: "stripe-hero/css-grid"
order: 2
tags: ["CSS", "grid", "placement", "layout"]
---

# Grid Placement — Named Lines vs Span

When you write `display: grid` on a container, its children flow automatically into cells — first child into cell 1, second into cell 2, and so on. This is **auto-placement**, and it's the right default for most grid layouts including the basic Stripe hero.

But more complex layouts need explicit control. When a UI screenshot card needs to span 2 columns, or an element needs to start in a specific column regardless of its position in the HTML, you need explicit placement. This article covers the full system.

---

## The grid line numbering system

CSS grid tracks are defined by **grid lines**. A three-column grid has four vertical lines:

```
│  col 1  │  col 2  │  col 3  │
1         2         3         4
```

Lines are numbered starting at `1` on the left (or top for rows). The rightmost line is `4` (for 3 columns). Negative numbers count from the opposite end: line `-1` is the rightmost line, `-2` is the second from the right.

> ⚠️ Grid lines start at **1**, not 0. This is a common source of off-by-one errors.

---

## grid-column and grid-row

The fundamental placement properties:

```css
.item {
  grid-column: 2;         /* starts at line 2, spans 1 column */
  grid-column: 2 / 4;    /* starts at line 2, ends at line 4 (spans 2 columns) */
  grid-column: 1 / -1;   /* from first line to last line (full width) */

  grid-row: 1;            /* starts at row line 1 */
  grid-row: 1 / 3;       /* spans 2 rows */
}
```

The shorthand `grid-column: start / end` — the numbers are **grid line numbers**, not column numbers. `grid-column: 2 / 4` starts at the second vertical line and ends at the fourth — this places the item in columns 2 and 3 (spanning 2 columns).

---

## span keyword

Instead of specifying a line number for the end, use `span`:

```css
.item {
  grid-column: 2 / span 2;   /* start at line 2, span 2 columns */
  grid-column: span 3;        /* span 3 columns wherever auto-placed */
}
```

`span N` is often more readable than computing the end line number. For the Stripe hero screenshots that bleed slightly, a `span 1` with careful alignment and `overflow: visible` is the pattern (the bleeding is handled with `translate` rather than spanning extra columns).

---

## Named grid lines

Instead of numbering, you can name grid lines in `grid-template-columns`:

```css
.hero {
  grid-template-columns:
    [content-start] 3fr [content-end screenshots-start] 2fr [screenshots-end];
}

.hero__text {
  grid-column: content-start / content-end;
}

.hero__screenshots {
  grid-column: screenshots-start / screenshots-end;
}
```

Named lines make the placement semantically clear. You're placing items into named regions rather than numbered positions. This matters for maintainability — if you change the column count or order, you update the names, not dozens of `grid-column: 2` declarations.

---

## grid-area — shorthand for row + column

```css
.item {
  grid-area: row-start / column-start / row-end / column-end;
  /* example: */
  grid-area: 1 / 1 / 2 / 3;  /* row 1, columns 1-2 */
}
```

Or with named areas:

```css
.hero {
  display: grid;
  grid-template-areas:
    "text  screenshots"
    "cta   screenshots";
  grid-template-columns: 3fr 2fr;
  grid-template-rows: 1fr auto;
}

.hero__text       { grid-area: text; }
.hero__cta        { grid-area: cta; }
.hero__screenshots { grid-area: screenshots; }
```

`grid-template-areas` uses ASCII art to define the layout. Each quoted string is a row. Same name in multiple cells = that item spans those cells. A `.` = an empty cell.

This is the most readable placement system for fixed layouts. You can see the layout directly in the CSS without mentally simulating line numbers.

---

## Auto-placement algorithm — how it works

When items are NOT explicitly placed, the browser uses the auto-placement algorithm:

1. Explicitly placed items are placed first.
2. Remaining items are placed into the next available cell, scanning left-to-right, top-to-bottom.
3. If an item spans multiple columns and won't fit in the current row, it moves to the next row.

You can change the scanning direction with `grid-auto-flow`:

```css
grid-auto-flow: row;     /* default — fill rows before starting new columns */
grid-auto-flow: column;  /* fill columns first */
grid-auto-flow: dense;   /* backfill gaps left by large items */
```

`grid-auto-flow: dense` is useful for masonry-like layouts where you have items of different sizes and want gaps to be backfilled by smaller items. Not used in the Stripe hero, but important to know.

---

## Overlapping cells

Grid items can be explicitly placed into the same cell — unlike flexbox, which can't do this without absolute positioning.

```css
.hero {
  display: grid;
  grid-template-columns: 3fr 2fr;
}

/* Place both items in column 2 */
.hero__screenshots-back {
  grid-column: 2;
  grid-row: 1;
  transform: translateY(20px) translateX(20px);
}

.hero__screenshots-front {
  grid-column: 2;
  grid-row: 1;  /* same cell */
  z-index: 1;
}
```

This technique is used to stack UI screenshot cards at slight offsets in the Stripe hero — giving the layered card appearance without making the HTML structure overly complex.

---

## Quick reference

| Declaration | What it does |
|---|---|
| `grid-column: 2` | Item starts at line 2, spans one column |
| `grid-column: 2 / 4` | Item from line 2 to line 4 (spans 2 columns) |
| `grid-column: 1 / -1` | Full-width (line 1 to last line) |
| `grid-column: span 2` | Auto-placed, spans 2 columns |
| `grid-area: text` | Placed into the named area "text" |
| `grid-template-areas` | ASCII-art layout definition |
| `grid-auto-flow: dense` | Backfill gaps in auto-placement |
