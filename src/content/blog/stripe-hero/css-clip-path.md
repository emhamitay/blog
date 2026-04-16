---
title: "CSS clip-path: Cutting Shapes Like a Pro"
description: "Master CSS clip-path from first principles — learn how to create complex shapes, understand coordinate systems, and build the perfect masking effects."
date: 2026-04-11
parent: "stripe-hero"
order: 1
tags: ["CSS", "clip-path"]
---

# CSS clip-path: Cutting Shapes Like a Pro

The `clip-path` property is one of CSS's most powerful tools for creating non-rectangular shapes. Unlike `border-radius` which only gives you rounded corners, `clip-path` lets you define arbitrary polygons, circles, ellipses, and even complex SVG paths.

In the Stripe hero, clip-path is used to create those smooth, flowing edge transitions between gradient regions. The effect feels organic and dynamic, but it's built on a surprisingly simple foundation: defining points in a coordinate system.

## Understanding the Coordinate System

Before we dive into complex shapes, let's understand how clip-path thinks about space. When you use `clip-path: polygon()`, you're defining points on a 2D plane where:

- The top-left corner is `0% 0%`
- The top-right corner is `100% 0%`
- The bottom-left corner is `0% 100%`
- The bottom-right corner is `100% 100%`

You can use percentages, pixels, or any CSS length unit. Percentages are usually best because they scale naturally with the element's size.

## Creating Your First Polygon

Let's start simple. A triangle pointing up:

```css
.triangle {
  clip-path: polygon(50% 0%, 0% 100%, 100% 100%);
}
```

This creates three points: one at the top center, and two at the bottom corners. The browser connects these points in order and clips everything outside.

Want to go deeper? Check out the polygon syntax guide for advanced techniques like creating multi-point paths and understanding winding order.
