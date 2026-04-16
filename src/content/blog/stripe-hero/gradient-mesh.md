---
title: "Creating Gradient Meshes with CSS"
description: "Learn how to build complex, multi-stop gradient meshes that bring depth and dimension to your designs — the secret sauce behind Stripe's flowing backgrounds."
date: 2026-04-13
parent: "stripe-hero"
order: 2
tags: ["CSS", "Gradients", "Design"]
---

# Creating Gradient Meshes with CSS

If you've ever wondered how Stripe achieves those silky-smooth, multi-colored gradient backgrounds, you're in the right place. The technique is called a gradient mesh — layering multiple gradients with different angles, colors, and opacity levels to create depth that feels almost three-dimensional.

## What Is a Gradient Mesh?

A gradient mesh isn't a single CSS property — it's a composition technique. You layer multiple `background-image` gradients on top of each other, each contributing a different color or tone to the final result.

Think of it like painting with translucent layers. Each gradient is semi-transparent, and where they overlap, the colors blend naturally to create new hues and transitions.

## The Foundation: Multiple Backgrounds

CSS lets you stack multiple backgrounds on a single element:

```css
.gradient-mesh {
  background-image:
    radial-gradient(circle at 20% 80%, rgba(120, 100, 250, 0.3) 0%, transparent 50%),
    radial-gradient(circle at 80% 20%, rgba(255, 100, 150, 0.3) 0%, transparent 50%),
    linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}
```

The browser renders these from top to bottom, so the first gradient in the list appears on top. In this example, we have two radial gradients creating "hotspots" of color, sitting on top of a base linear gradient.

## Controlling Opacity for Depth

The key to a great mesh is controlling opacity. Each layer should be semi-transparent so the layers beneath can show through. That's why we use `rgba()` or `hsla()` color values with alpha channels less than 1.0.

Experiment with opacity values between 0.2 and 0.5 for the overlay gradients. Too high, and you lose the blending effect; too low, and the gradient becomes invisible.

In the next section, we'll explore how to animate these meshes smoothly, creating that signature flowing effect you see on the Stripe homepage.
