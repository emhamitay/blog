---
title: "Polygon Syntax Deep Dive"
description: "Everything you need to know about the polygon() function in CSS clip-path — from basic syntax to advanced techniques for creating complex shapes."
date: 2026-04-12
parent: "stripe-hero/css-clip-path"
order: 1
tags: ["CSS", "clip-path", "polygon"]
---

# Polygon Syntax Deep Dive

The `polygon()` function is the workhorse of `clip-path`. It's deceptively simple on the surface — just a list of coordinate pairs — but there's nuance in how you structure these points and what effects you can achieve.

## The Basic Syntax

At its core, a polygon is defined like this:

```css
clip-path: polygon(x1 y1, x2 y2, x3 y3, ...);
```

Each pair of values represents a point. The browser draws straight lines connecting these points in the order you specify, and automatically closes the path from the last point back to the first.

## Coordinate Units

You can mix and match units within a single polygon:

```css
clip-path: polygon(
  0 0,           /* top-left corner */
  100% 0,        /* top-right corner */
  100% calc(100% - 50px),  /* almost bottom-right */
  50% 100%,      /* bottom-center */
  0 calc(100% - 50px)      /* almost bottom-left */
);
```

This creates a pentagon with an angled bottom. Notice how we can use `calc()` within the coordinate values — this is incredibly powerful for creating responsive, dynamic shapes.

## The Fill Rule

Polygons follow the "nonzero winding rule" by default, but you can specify the fill rule explicitly:

```css
clip-path: polygon(evenodd, 0 0, 100% 0, 100% 100%, 0 100%);
```

The `evenodd` keyword changes how self-intersecting paths are filled. For most use cases, you won't need this, but it's essential when creating complex shapes with holes or overlapping regions.

Understanding these fundamentals means you can create any shape imaginable. In the Stripe hero, we use carefully crafted polygons to create those smooth, organic dividing lines between sections.
