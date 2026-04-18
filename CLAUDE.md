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
  --question:     #2CF5C8;   /* uncertain fields: Y/N, TBD, ??? */
}
```

> To shift to orange/amber variant (as in the Philips HUD screenshot), swap the hue to ~38° and adjust: bg `#0a0600`, dim `#5c3200`, main `#d4820a`, bright `#f5a623`, glow `#d4820a55`.

---

## Typography

```html
<link href="https://fonts.googleapis.com/css2?family=Share+Tech+Mono&display=swap" rel="stylesheet">
```

```css
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
  { text: '──────────────────────', cls: 'separator' },
  { text: '' }, // blank line — renders as &nbsp;
];
```

Character-by-character render with variable speed:
```js
const delay = char === ' ' ? 15 : (Math.random() * 25 + 12); // 12–37ms
```

CSS classes used in markup segments:
- `.key` → `var(--amber-dim)` — label text
- `.val` → `var(--amber-bright)` with glow — data values
- `.highlight` → bright + double glow — important callouts
- `.question` → `#2CF5C8` — uncertain / TBD items
- `.separator` → `var(--amber-dim)` — `───` dividers, box borders
- `.section-label` → bright + underline — section headings

---

## Content Conventions

- Labels: `ALL_CAPS_WITH_UNDERSCORES:`
- Values follow immediately on same line or indented below
- Unknown/uncertain values: `Y/N`, `TBD`, `???`, trailing `?` on names
- Lists: `[ITEM_A, ITEM_B, ITEM_C]` with bracket styling
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
  <link href="https://fonts.googleapis.com/css2?family=Share+Tech+Mono&display=swap" rel="stylesheet">
  <style>
    /* paste :root, *, html/body, body::before/after, @keyframes, .screen, .cursor, .line, .key, .val, .highlight, .question, .separator, .section-label */
  </style>
</head>
<body>
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
