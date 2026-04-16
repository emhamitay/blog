---
title: "Color Spaces and oklch — Why Stripe Colors Look So Smooth"
description: "Stripe uses oklch() colors, not hex or rgba(). Understanding perceptual color spaces explains why their gradients look smoother and more vibrant — and how to use oklch in your own CSS."
date: 2026-04-16
parent: "stripe-hero/gradient-mesh"
order: 3
tags: ["CSS", "color", "oklch", "gradients"]
---

# Color Spaces and oklch — Why Stripe Colors Look So Smooth

If you've ever built a gradient between two saturated colors — say, from a vivid blue to a vivid pink — and it looked muddy and desaturated in the middle, that's a color space problem. The browser was blending those colors through sRGB, and sRGB doesn't blend "through" colors the way human perception does.

Stripe's actual CSS uses `oklch()` colors. Not `rgba()`, not hex. Understanding why explains both the visual quality of their hero and how to use modern CSS color functions properly.

---

## The problem with sRGB gradients

sRGB is the color space CSS has used for decades. `#ff0000` is red, `#0000ff` is blue, and a 50% blend is calculated by averaging the R, G, B channels: `#800080` — which is a muted purple.

The problem is that sRGB doesn't model *perceptual* color. The "same" amount of numeric difference in sRGB doesn't produce the same *visual* difference to a human eye. Some regions of sRGB are very dense (lots of visible colors in a small numeric range), and some are sparse. Blending through sRGB interpolates the numbers, not the perception.

The result: gradients through sRGB often pass through a "gray zone" in the middle — a band of desaturated, muddy color between two vivid endpoints. You've seen this on a hundred websites.

---

## What perceptual color spaces solve

Perceptual color spaces are designed so that equal numeric differences produce equal *visual* differences. When you blend between two vivid colors in a perceptual space, the midpoint is *also* vivid — it stays on the vivid outer edge of the color gamut, not the muddy interior.

The main perceptual color space supported in modern CSS is **OKLab** and its polar coordinate version **OKLCH**.

---

## oklch() — the syntax

```css
color: oklch(L C H);
color: oklch(L C H / alpha);
```

| Parameter | What it is | Range |
|---|---|---|
| `L` — Lightness | Perceptual lightness | `0` (black) to `1` (white) |
| `C` — Chroma | Colorfulness / saturation | `0` (gray) to ~`0.4` (max vivid) |
| `H` — Hue | The color angle | `0°` to `360°` |
| `alpha` | Transparency | `0` to `1` |

**Hue angles (approximate):**
- `0°` / `360°` — red
- `30°` — orange
- `60°` — yellow
- `120°` — green
- `180°` — cyan
- `240°` — blue
- `270°` — purple
- `300°` — pink/magenta

---

## oklch vs rgba — a direct comparison

The same Stripe blue in both systems:

```css
/* rgba */
color: rgba(99, 102, 241, 1);   /* #6366f1 — a vivid blue-purple */

/* oklch */
color: oklch(0.585 0.233 264°);
```

The oklch version tells you directly: lightness is 58.5%, chroma (saturation) is 0.233, hue is at 264° (the blue-purple region). If you want it lighter, increase L. If you want it more vivid, increase C. If you want it to shift toward blue, increase H.

With rgba, none of those operations are obvious. Changing just the "saturation" means recalculating all three channels.

---

## Why oklch gradients look smoother

When CSS interpolates a gradient, it blends the colors in whatever color space they're specified in. `rgba()` gradients blend in sRGB. `oklch()` gradients blend in OKLab space.

Consider a gradient from vivid purple to vivid cyan:

```css
/* sRGB gradient — goes through a muddy gray-blue middle */
background: linear-gradient(to right, #a855f7, #06b6d4);

/* oklch gradient — stays vivid the whole way */
background: linear-gradient(in oklch, oklch(0.62 0.27 300°), oklch(0.68 0.18 195°));
```

The `in oklch` interpolation hint tells the browser to blend these colors through OKLab space. The midpoint stays on the vivid arc of the color gamut. No gray zone.

This is exactly why Stripe gradients look so rich — they're interpolating through a perceptually uniform space.

---

## The `color-interpolation-method` in gradients

```css
linear-gradient(in oklch, color1, color2)
radial-gradient(in oklch, color1, color2)
```

The `in <color-space>` token tells the browser which color space to use for blending. Options include:

| Value | Color space | Notes |
|---|---|---|
| `in srgb` | sRGB | Default for most gradients |
| `in oklch` | OKLab polar | Best for vivid, perceptually smooth gradients |
| `in oklab` | OKLab rectangular | Same quality as oklch, different coordinate form |
| `in hsl` | HSL | Stays on the hue arc, but HSL has other perceptual issues |
| `in display-p3` | Display P3 | Wide gamut, good for newer displays |

For the Stripe mesh effect, using `in oklch` for the radial gradients is what produces the smooth transitions between blob colors.

---

## The wide gamut advantage

sRGB covers only a subset of the colors your monitor can display. Modern displays (most laptops and phones since ~2018) support the **Display P3** gamut, which is about 25% wider than sRGB. Colors in P3 that sRGB can't represent are called "out of gamut" for sRGB.

oklch can express these wide-gamut colors:

```css
/* This vivid green is out of sRGB gamut — can't be expressed in hex */
color: oklch(0.7 0.35 145°);
```

Stripe uses this — some of their blues and cyans are genuinely in the P3 gamut, which is why they look more vivid than anything you could write as a hex color.

> ⚠️ **Browser support:** `oklch()` is supported in all modern browsers (Chrome 111+, Firefox 113+, Safari 15.4+). For older browser fallbacks, always specify a `rgba()` or hex color first:
> ```css
> color: #6366f1;  /* fallback for older browsers */
> color: oklch(0.585 0.233 264°);  /* override for modern browsers */
> ```

---

## Building the Stripe mesh in oklch

Converting the Stripe blob colors to oklch:

```css
.hero {
  background:
    /* pink blob */
    radial-gradient(in oklch, ellipse at 75% 55%,
      oklch(0.72 0.25 330° / 0.35) 0%,
      transparent 45%),
    /* cyan blob */
    radial-gradient(in oklch, ellipse at 85% 15%,
      oklch(0.75 0.17 200° / 0.4) 0%,
      transparent 50%),
    /* purple blob */
    radial-gradient(in oklch, ellipse at 15% 60%,
      oklch(0.58 0.28 270° / 0.5) 0%,
      transparent 55%),
    /* base */
    linear-gradient(in oklch, 135deg,
      oklch(0.585 0.233 264°) 0%,
      oklch(0.62 0.20 230°) 50%,
      oklch(0.68 0.18 195°) 100%);
}
```

The blending between blob colors is now visually smooth — no muddy transitions in the midpoints.

---

## Quick reference — oklch conversions

Converting common web colors to oklch (approximate):

| Color | hex | oklch |
|---|---|---|
| Stripe blue | `#6366f1` | `oklch(0.585 0.233 264°)` |
| Stripe cyan | `#0ea5e9` | `oklch(0.67 0.18 200°)` |
| Stripe purple | `#8b5cf6` | `oklch(0.58 0.25 278°)` |
| Stripe pink | `#ec4899` | `oklch(0.64 0.26 328°)` |

Tools for converting: [oklch.com](https://oklch.com) (interactive picker), or use Chrome DevTools color picker which now supports oklch.
