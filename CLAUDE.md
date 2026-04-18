# Terminal HUD Style Guide

This project uses a **retro CRT terminal / military HUD** aesthetic. Every new HTML page in this repo must follow this style.

---

## Visual Aesthetic

- Near-black background, single monochrome accent color
- Feels like a 1980s radar/dispatch terminal or aircraft HUD
- All text in monospace; labels ALL_CAPS; values highlighted brighter
- Content types character-by-character as if being transmitted live
- Decorative: scanlines, vignette, subtle flicker, blinking cursor
- Structural decoration: ASCII box borders, separator lines, crosshairs, corner brackets
- Background layers: SVG fractal noise texture + wireframe terrain SVG at bottom

---

## Color Palette (base hue: #35AEB3 teal)

```css
:root {
  --bg:           #020a0a;   /* page background */
  --bg-panel:     #041010;   /* panel/card background */
  --amber-dim:    #1A5759;   /* borders, secondary text */
  --amber:        #35AEB3;   /* primary foreground / body text */
  --amber-bright: #5DE0E6;   /* highlighted values, headings */
  --amber-glow:   #35AEB355; /* text-shadow / glow color */
}
```

Question/uncertain color is hardcoded (not a variable): `#2CF5C8` with glow `#2CF5C844`.

> To shift to orange/amber variant (as in the Philips HUD screenshot), swap the hue to ~38° and adjust: bg `#0a0600`, dim `#5c3200`, main `#d4820a`, bright `#f5a623`, glow `#d4820a55`.

---

## Typography

Load via `@import` inside `<style>` (not a `<link>` tag):

```css
@import url('https://fonts.googleapis.com/css2?family=Share+Tech+Mono&display=swap');

font-family: 'Share Tech Mono', monospace;
```

---

## Required CSS Effects

### CRT Scanlines + Vignette
```css
body::before { /* scanlines */
  content: '';
  position: fixed; inset: 0;
  background: repeating-linear-gradient(
    0deg,
    transparent, transparent 2px,
    rgba(0,0,0,0.15) 2px, rgba(0,0,0,0.15) 4px
  );
  pointer-events: none; z-index: 1000;
}
body::after { /* vignette */
  content: '';
  position: fixed; inset: 0;
  background: radial-gradient(ellipse at center, transparent 50%, rgba(0,0,0,0.6) 100%);
  pointer-events: none; z-index: 999;
}
```

### Screen Flicker
```css
@keyframes flicker {
  0%, 100% { opacity: 1; }
  92% { opacity: 1; } 93% { opacity: 0.8; }
  94% { opacity: 1; } 96% { opacity: 0.9; } 97% { opacity: 1; }
}
.screen { animation: flicker 6s infinite; }
```

### Blinking Cursor
```css
@keyframes blink { 0%, 100% { opacity: 1; } 50% { opacity: 0; } }
.cursor {
  display: inline-block; width: 10px; height: 17px;
  background: var(--amber-bright); vertical-align: text-bottom;
  animation: blink 0.7s step-end infinite;
  box-shadow: 0 0 6px var(--amber-glow);
}
```

### Ambient Noise Texture
```html
<div class="noise"></div>
```
```css
.noise {
  position: fixed; inset: 0; opacity: 0.03;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
  pointer-events: none; z-index: 998;
}
```

### Wireframe Terrain (bottom decoration)
```html
<svg class="terrain-deco" viewBox="0 0 1200 180" preserveAspectRatio="none" xmlns="http://www.w3.org/2000/svg">
  <g stroke="#35AEB3" fill="none" stroke-width="0.8">
    <!-- 5 polylines at increasing y offsets to create layered terrain -->
  </g>
</svg>
```
```css
.terrain-deco {
  position: fixed; bottom: 0; left: 0; right: 0;
  height: 180px; pointer-events: none; z-index: 0; opacity: 0.08;
}
```

---

## Typing Animation (JS)

Lines are defined as objects with optional per-segment markup:

```js
const lines = [
  { text: 'LOCATION: COPENHAGEN', markup: [
    { text: 'LOCATION: ', cls: 'key' },
    { text: 'COPENHAGEN', cls: 'val' }
  ]},
  { text: 'STATUS: TBD', markup: [
    { text: 'STATUS: ', cls: 'key' },
    { text: 'TBD', cls: 'question' }
  ]},
  { text: '[ITEM_A, ITEM_B]', markup: [
    { text: '[', cls: 'bracket' },
    { text: 'ITEM_A, ITEM_B', cls: 'val' },
    { text: ']', cls: 'bracket' }
  ]},
  { text: '──────────────────────', cls: 'separator' },
  { text: '' }, // blank line — renders as &nbsp;
];
```

Character-by-character render with variable speed:
```js
const delay = plainText[currentChar - 1] === ' ' ? 15 : (Math.random() * 25 + 12); // 12–37ms
```

CSS classes used in markup segments:
- `.key` → `var(--amber-dim)` — label text
- `.val` → `var(--amber-bright)` with glow — data values
- `.highlight` → bright + double glow — important callouts
- `.question` → `#2CF5C8` with glow `#2CF5C844` — uncertain / TBD items
- `.bracket` → `var(--amber-dim)` — `[` and `]` around lists
- `.separator` → `var(--amber-dim)` — `───` dividers, box borders
- `.section-label` → `var(--amber-bright)` + underline — section headings

---

## Content Conventions

- Labels: `ALL_CAPS_WITH_UNDERSCORES:`
- Values follow immediately on same line or indented below
- Unknown/uncertain values: `Y/N`, `TBD`, `???`, trailing `?` on names
- Lists: `[ITEM_A, ITEM_B, ITEM_C]` — brackets styled with `.bracket`, items with `.val`
- ASCII borders for major sections:
  ```
  ┌──────────────────────────┐
  │  SECTION TITLE           │
  └──────────────────────────┘
  ```
- Separator lines: `─────────────────────`
- Coordinates: `55.6761°N  12.5683°E`
- Operation codes: `OMEGA_CHARLIE_SAGE_PREM` style

---

## HTML Skeleton

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>TITLE // OPERATION_CODE</title>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Share+Tech+Mono&display=swap');
    /* paste :root, *, html/body, body::before/after, @keyframes flicker/blink,
       .screen, .noise, .terrain-deco, .cursor, .line, .topbar, .bottombar,
       .key, .val, .highlight, .question, .bracket, .separator, .section-label */
  </style>
</head>
<body>
  <div class="noise"></div>
  <svg class="terrain-deco" viewBox="0 0 1200 180" preserveAspectRatio="none" xmlns="http://www.w3.org/2000/svg">
    <g stroke="#35AEB3" fill="none" stroke-width="0.8"><!-- polylines --></g>
  </svg>
  <div class="screen">
    <div class="topbar">
      <span>SYS.DISPATCH.v4.2.1</span>
      <span class="title">PAGE TITLE</span>
      <span>SEC: <span style="color:var(--amber-bright)">CLASSIFIED</span></span>
    </div>
    <div class="terminal" id="terminal"></div>
    <div class="bottombar" id="bottombar">
      <span><span class="status-dot"></span>TRANSMISSION ACTIVE</span>
    </div>
  </div>
  <script>/* lines array + typing engine */</script>
</body>
</html>
```
