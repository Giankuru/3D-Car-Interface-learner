# 00. Run it and find your way

**Time:** 30 min · **Difficulty:** 1 of 5 · [All lessons](README.md) · Next: [01 JavaScript for C++ programmers](01-js-for-cpp.md)

## Goal

Open the app, find the three parts of `index.html` (CSS, HTML, JavaScript), and use the browser's
DevTools to look inside the running program.

## Read these lines

- The [map of `index.html`](README.md#map-of-indexhtml) in the lesson list.
- Lines 1-5 (the top), lines 168-174 (the tab links), lines 269-281 (the libraries and helpers) and
  lines 1317-1318 (the last two lines of code).

## The ideas

### One file, three languages

A web page is made of three things, and this file holds all three:

```
 +------------------------------+
 | CSS   lines 6-163            |  how it LOOKS   (colours, layout, fonts)
 +------------------------------+
 | HTML  lines 165-267          |  what is ON the page (tabs, buttons, panels)
 +------------------------------+
 | JS    lines 269-1319         |  what it DOES   (3D, quizzes, saving progress)
 +------------------------------+
```

The HTML is mostly empty boxes. For example the Maintenance tab is just this
(`<section class="view" id="v-maintain" hidden>`, lines 235-245):

```html
<aside class="side" id="mSide"></aside>
<article class="doc" id="mDoc"></article>
```

JavaScript fills those boxes in later. So: **if you want to know what the page shows, read the
JavaScript, not the HTML.**

### Opening it

Double-click `index.html`, or drag it onto a Chrome, Edge or Firefox window. No install, no build step,
no server.

You need **internet**: lines 269-270 load the 3D library (Three.js) from a CDN, a public server that
hosts libraries. Offline, the Engine and Build a Car tabs show "The 3D library could not load"; the other
three tabs still work.

The file starts with `<title>`, not `<!doctype html><html><body>`. That's fine: browsers add the missing
wrapper themselves. (The file was written to be published as a page fragment.)

### DevTools: your window into the running program

Press **F12** (or right-click > Inspect). Three tabs matter:

| DevTools tab | What it's for |
|---|---|
| **Console** | Type JavaScript and press Enter to run it. Red errors appear here. |
| **Elements** | The live HTML, including everything JavaScript added. |
| **Application > Local Storage** | What the app has saved in your browser (lesson 03). |

The names the script creates at its top level, such as `GUIDES` or `ENG`, can be typed straight into the
Console. That lets you inspect the program's data while it runs.

### Where does it start?

At the very bottom (lines 1317-1318):

```js
route();
requestAnimationFrame(loop);
```

Everything above those two lines only *defines* things. Lesson 02 follows what these two calls do.

## Exercise: poke the data

1. Open `index.html` in your browser. You land on the Engine tab.
2. Open the Console (F12 > Console).
3. Type each line and press Enter:

```js
GUIDES.length
STEPS.length
MODULES.length
ENG.parts.length
location.hash = "#weld"
```

**You should now see:** `14`, `14`, `7` and `21`, and the last line switches the page to the Weld & Fab tab.

<details><summary>What were those numbers?</summary>

- `GUIDES` is the list of 14 maintenance jobs (line 884).
- `STEPS` is the 14 Build a Car steps (line 588).
- `MODULES` is the 7 welding modules (line 1125).
- `ENG.parts` is the 21 engine parts, filled in when the Engine tab is built on page load.
- `location.hash` is the `#...` part of the address. Changing it changes the tab (lesson 02).

</details>

## Traps

- **`ENG.parts.length` is `0`:** the 3D library didn't load (no internet, or WebGL is turned off). Lessons
  01-05 still work; fix the connection before lesson 06.
- **"`GUIDES` is not defined":** you are in the Console of a different tab or page. Reload `index.html`
  and try again.
- **Getting lost in the file:** search for `/* ====` to jump between sections, and turn on word-wrap.

## New words

[CDN](GLOSSARY.md#cdn) · [DevTools](GLOSSARY.md#devtools) · [HTML, CSS, JS](GLOSSARY.md#html-css-js) ·
[URL hash](GLOSSARY.md#url-hash-and-hash-routing)
