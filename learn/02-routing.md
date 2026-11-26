# 02. Tabs and routing: the program's `main()`

**Time:** 45 min · **Difficulty:** 2 of 5 · Previous: [01](01-js-for-cpp.md) · Next: [03 Content as data](03-content-as-data.md)

## Goal

Know where execution starts, how a click on a tab becomes a visible page, and why each tab is set up
only once.

## Read these lines

- The tab links, lines 168-174.
- The five section openings: lines 180, 207, 235, 247, 256.
- `route()` and its variables, lines 1281-1306.
- `loop()`, lines 1307-1312 (only notice that it draws the active 3D view; lesson 06 explains it).
- The start-up, lines 1317-1318.

## The ideas

### The URL hash

The **hash** is the part of the address after `#`: in `.../index.html#build` it is `#build`. Changing
it does **not** reload the page. The browser only fires a `hashchange` event. The tab links are plain
links to hashes (lines 168-174):

```html
<nav id="nav">
  <a href="#explore">Engine</a>
  <a href="#build">Build a Car</a>
  ...
```

Each view is a `<section>` whose `id` is `v-` plus the same name (line 207):

```html
<section class="view" id="v-build" hidden>
```

The `hidden` attribute hides an element. Showing a tab = removing `hidden` from one section and adding
it to the others.

### `route()`: the dispatcher

Lines 1281-1284 set up four variables:

```js
const VIEWS = ["explore", "build", "maintain", "electrical", "weld"];
const inited = {};      // which tabs have been set up already
const view3d = {};      // the 3D viewer of each 3D tab
let active3d = null;    // the viewer to draw right now (or null)
```

Then `route()` (lines 1285-1305), in four steps:

```js
const h = location.hash.slice(1), name = VIEWS.includes(h) ? h : "explore";   // 1. which tab?
VIEWS.forEach(v => { $("#v-" + v).hidden = v !== name; });                     // 2. show one, hide the rest
```

3. Mark the active link: `a.setAttribute("aria-current", "page")` (line 1289). Screen readers announce
   it, and the CSS (line 46) colours the link with `nav a[aria-current="page"]`.
4. **Lazy initialisation**: build the tab's content the first time only (lines 1292-1301):

```js
if (!inited[name]) {
  inited[name] = true;
  ...
  if (name === "maintain") initMaintain();
```

Why lazy? Building a 3D scene is expensive. If you never open Build a Car, its scene is never built.
And because `inited[name]` is set, coming back to a tab only shows and hides; the tab keeps its state.

Line 1306 makes `route` run on every hash change:

```js
window.addEventListener("hashchange", route);
```

### Only one 3D view is drawn

Line 1303: `active3d = view3d[name] || null;`. The render loop draws only `active3d`, so a hidden 3D tab
costs nothing:

```js
function loop(now) {
  requestAnimationFrame(loop);
  const v = active3d; if (!v) return;
```

### The life of the page

Put together, this is the whole program flow (also in the [lesson list](README.md#life-of-the-page)):

```
open file -> CSS/HTML read -> 3D library downloads -> script defines everything
  -> route()                    (line 1317)  show #v-explore, initEngine() once
  -> requestAnimationFrame(loop) (line 1318) draw active3d ~60 times a second
click a tab -> hash changes -> "hashchange" -> route() -> show/hide, init on first visit
```

### Two things you can ignore, one you must not

- Line 1304 sends a fake window `resize` event after a 3D tab appears. Nothing in this file listens for
  it (the viewers use a `ResizeObserver`, lesson 06), so it is harmless.
- Lines 1313-1316 repaint 3D backgrounds when the theme changes (lesson 05).
- **Must not ignore:** line 1297 wraps the 3D set-up in `try`/`catch` and, on *any* error, shows
  "3D view could not start (WebGL may be disabled in this browser)". When your own edit has a bug in a
  3D tab, that message is wrong about the cause. **Always open the Console for the real error.**

## Exercise: add a sixth tab

1. In the nav (after line 173) add `<a href="#about">About</a>`.
2. Before `</main>` (line 266) add:
   ```html
   <section class="view" id="v-about" hidden><div class="view-head"><h2>About</h2></div><p>My notes.</p></section>
   ```
3. Add `"about"` to the end of `VIEWS` (line 1281).
4. Add `console.log("maintain init");` as the first line inside `initMaintain` (after line 967).
5. Save, reload, open the Console.

**You should now see:** clicking About shows only your section, the address ends in `#about`, the About
link is highlighted, and reloading keeps you on About. Visiting Maintenance three times (with other tabs
in between) prints "maintain init" only once.

<details><summary>Answer and why it works</summary>

```js
const VIEWS = ["explore", "build", "maintain", "electrical", "weld", "about"];
```

`route()` finds `"about"` in `VIEWS`, so `$("#v-about")` gets un-hidden and the link with
`href="#about"` gets `aria-current`. There is no `init` for "about", which is fine: the `if` lines
simply don't match. Reload keeps you there because the hash is part of the address. The log appears
once because `inited.maintain` is `true` after the first visit.

</details>

## Traps

- **Forgot `VIEWS`:** the address changes but `route()` falls back to the Engine tab.
- **Wrong `id`:** it must be exactly `v-` + the name. Otherwise `$("#v-about")` is `null` and the Console
  shows "Cannot set properties of null".
- **A 3D tab says "WebGL may be disabled" after your edit:** read the Console (see above).

## New words

[URL hash and hash routing](GLOSSARY.md#url-hash-and-hash-routing) ·
[Lazy initialisation](GLOSSARY.md#lazy-initialisation) · [Event and listener](GLOSSARY.md#event-and-listener) ·
[hidden attribute](GLOSSARY.md#hidden-attribute) · [aria-current](GLOSSARY.md#aria-current) ·
[requestAnimationFrame](GLOSSARY.md#requestanimationframe)
