# 05. Drawing in 2D: SVG schematics

**Time:** 40 min · **Difficulty:** 2 of 5 · Previous: [04](04-quiz-and-calculator.md) · Next: [06 A 3D scene](06-hello-3d.md)

## Goal

Read and edit a vector drawing, and meet your first coordinate system. (It is **not** the same as the 3D
one in lesson 06.)

## Read these lines

- `schematicSVG`, lines 1078-1111: read the first 10 lines carefully and skim the rest.
- `jointSVG`, lines 1116-1123 (the five weld joints in module 4).

## The ideas

### SVG: drawings written as tags

SVG is a way to write a drawing as text: each shape is a tag with numbers. The browser draws it sharp at
any size. `schematicSVG()` just returns one long template string of SVG (line 1080):

```html
<svg viewBox="0 0 820 300" width="820" height="300" role="img" aria-label="Starting and charging circuit schematic" style="color:var(--ink)">
```

`viewBox="0 0 820 300"` says "my drawing uses coordinates from (0, 0) to (820, 300)". The browser scales
that box to fit the space on screen.

### The y axis points down

```
(0,0) ----------------------------> x  (820)
  |
  |      rect x=30 y=110  (BATTERY)
  |
  |                  bigger y = LOWER on screen
  v
  y (300)
```

Origin top-left, **y grows downward** (like text on a page). The 3D scene in lesson 06 is the other way
round: y points up.

### Shapes

| Tag | Example from the file | Draws |
|---|---|---|
| `rect` | `<rect class="bx" x="30" y="110" width="110" height="70" rx="4"/>` (line 1083) | box (`rx` rounds the corners) |
| `circle` | `<circle class="bx" cx="250" cy="210" r="36"/>` (line 1100) | the alternator |
| `text` | `<text class="t" x="85" y="140" text-anchor="middle">BATTERY</text>` (line 1083) | label |
| `path` | `<path class="h" d="M50 110 V60 H600 V110"/>` (line 1086) | a line with corners |

### The path mini-language

| Command | Means |
|---|---|
| `M x y` | move the pen to (x, y) without drawing |
| `H x` / `V y` | draw a horizontal / vertical line to that x / y |
| `h dx` / `v dy` | same, but relative to where the pen is |
| `L x y` | straight line to (x, y) |
| `Z` | close the shape back to the start |

Walk line 1086 on graph paper: `M50 110` (top of the battery's + post), `V60` (up to y = 60),
`H600` (right to x = 600), `V110` (down to the solenoid). That's the heavy red cable.

### Colours that follow the theme

The SVG has its own little stylesheet (line 1081):

```css
.w{stroke:currentColor;stroke-width:2;fill:none}
.h{stroke:var(--bad);stroke-width:4;fill:none}
.g{stroke:var(--muted);stroke-width:2;fill:none;stroke-dasharray:5 4}
.bx{fill:var(--panel);stroke:currentColor;stroke-width:1.5}
```

`.w` is a wire, `.h` a heavy cable, `.g` a dashed ground, `.bx` a box, `.t` a label. `currentColor` means
"the text colour", which is `var(--ink)` (set on the `<svg>`). `var(--bad)` and the others are the CSS
variables from the top of the file, which have dark-mode versions (lines 17-26). So the drawing changes
with the light/dark theme, with no extra code.

## Exercise: add a horn

Inside the template string in `schematicSVG`, just before `</svg>` (line 1110), add a box labelled HORN
and a wire from the heavy positive cable:

```html
<!-- horn -->
<rect class="bx" x="660" y="20" width="80" height="30" rx="3"/><text class="t" x="700" y="40" text-anchor="middle">HORN</text>
<path class="w" d="M600 60 H620 V35 H660"/>
```

Reload and open Electrical. Then run this in the Console:

```js
document.documentElement.dataset.theme = "dark"
```

**You should now see:** a HORN box at the top right of the schematic, wired to the corner of the red
cable. After the Console line the whole page and the schematic switch to dark colours (your box too,
because it uses the same classes). If you visit the Engine tab, its 3D background is dark as well.

<details><summary>Answer: reading the new path</summary>

`M600 60` starts at the corner of the heavy cable (where `H600` ended). `H620` goes 20 right,
`V35` goes **up** to y = 35 (smaller y = higher), `H660` reaches the left edge of the box (x = 660),
level with the middle of the box (y 20 to 50, middle 35).

`dataset.theme = "dark"` sets `data-theme="dark"` on `<html>`, which switches on the CSS at line 22.
A `MutationObserver` (line 1316) notices the change and repaints the 3D backgrounds too. To go back:
`document.documentElement.dataset.theme = "light"`.

</details>

## Traps

- **Bigger y is lower,** not higher.
- **Outside 0-820 × 0-300** the drawing is cut off.
- **Inside the template string,** an HTML `<!-- comment -->` is fine, but `${` starts a JavaScript hole and
  breaks things, and a stray backtick ends the string.

## New words

[SVG](GLOSSARY.md#svg) · [viewBox](GLOSSARY.md#viewbox) · [SVG path](GLOSSARY.md#svg-path) ·
[CSS variable](GLOSSARY.md#css-variable) · [MutationObserver](GLOSSARY.md#mutationobserver)
