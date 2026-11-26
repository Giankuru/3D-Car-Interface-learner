# Learn how Teardown Garage is built

This folder is a step-by-step guide to the code behind Teardown Garage. The whole app is one file,
`index.html`: a web page with a 3D engine you can take apart, a car you build piece by piece, and three
reading sections (maintenance, electrical, welding). By the end you will have read every important
part of it and changed most of them.

## Who this is for

You can already program a little in some language (C++ up to classes and arrays is plenty): you know
variables, functions, loops, arrays and structs. You don't need to know JavaScript, web pages or 3D
graphics. The guide teaches those, always using real lines from `index.html`.

> **Never written code before?** Do a beginner course in HTML, CSS and JavaScript first (roughly
> 15-20 hours, for example MDN's [Learn web development](https://developer.mozilla.org/en-US/docs/Learn)
> and the first part of [javascript.info](https://javascript.info/)). Then come back: this guide
> assumes you know what a variable, a function and a loop are.

## What you need first

| What | Why | How long |
|---|---|---|
| Programming basics (variables, functions, loops, arrays, structs or classes) | Everything builds on it | already have it |
| A skim of HTML and CSS: tags, `id`, `class`, attributes, CSS variables like `var(--ink)` | The page is HTML + CSS | ~1 hour (MDN "HTML basics" and "CSS basics"; skim, don't study) |
| A modern browser (Chrome, Edge or Firefox) **with internet** | The 3D library loads from the internet | - |
| A text editor with "Go to line" and word-wrap (VS Code is fine) | You will jump to line numbers a lot | - |
| Vectors, x/y/z axes, radians | The 3D lessons use them | school physics is enough |

Modern JavaScript and the browser's DevTools are **not** prerequisites: lessons 00 and 01 cover them.

## Start here

1. Read this page, especially the [map of `index.html`](#map-of-indexhtml) below.
2. Do [lesson 00](00-start-here.md): open the app, open DevTools, poke the data.
3. Do [lesson 01](01-js-for-cpp.md) (JavaScript for C++ programmers) even if you are impatient. Every
   later lesson assumes it.
4. Then go in order. Lessons 02-05 are the gentle part (the web page, data, 2D drawing). Lessons 06-11
   are the 3D part. **Lesson 06 is the hardest climb**; after lesson 10 it is all reuse.
5. One rule for the whole guide: **read only the lines a lesson names, and skim the rest.** Most of
   the file is content (car text, quiz text). You never need to read all 215 lines of `STEPS`.

Every lesson has the same parts: a goal, the lines to read, the ideas explained, one exercise with a
**"you should now see"** checkpoint and a folded answer, the traps, and new words (all collected in the
[glossary](GLOSSARY.md)).

Line numbers in this guide match `index.html` as it is in this repository. If you edit the file, the
numbers below your edit shift; undo your experiments (or use `git stash`) before the next lesson.

## The lessons

| # | Lesson | What you learn | Minutes |
|---|---|---|---|
| 00 | [Run it and find your way](00-start-here.md) | Open the app, map the one big file, poke the data from DevTools | 30 |
| 01 | [JavaScript for C++ programmers](01-js-for-cpp.md) | Every JS idiom the file uses, each shown on a real line | 75 |
| 02 | [Tabs and routing: the program's main()](02-routing.md) | URL hash, show/hide views, lazy init, where execution starts | 45 |
| 03 | [Content as data: maintenance guides](03-content-as-data.md) | Lists of objects turned into pages; search, checklists, localStorage | 60 |
| 04 | [Weld course quizzes and the MIG calculator](04-quiz-and-calculator.md) | Positional quiz data, pass tracking and a one-formula calculator | 50 |
| 05 | [Drawing in 2D: SVG schematics](05-svg-schematics.md) | viewBox, y-down coordinates, path commands, theme-aware colours | 40 |
| 06 | [A 3D scene: renderer, camera, lights, loop](06-hello-3d.md) | Hello-cube first, then the first half of `createViewer` and the render loop | 100 |
| 07 | [Orbit and zoom](07-orbit-controls.md) | OrbitControls, damping, `update()` every frame, `lerp` to refocus | 35 |
| 08 | [Models from basic shapes](08-models-from-shapes.md) | Meshes, materials, groups and `userData` in `buildEngine` | 90 |
| 09 | [The teardown slider](09-teardown-slider.md) | Stages, smoothstep and the per-frame chase behind `explodeProgress` | 60 |
| 10 | [Clicking a part: raycasting and highlight](10-clicking-parts.md) | Pixels to NDC to a ray, hit to part, emissive and isolation | 85 |
| 11 | [Capstone: Build a Car](11-build-a-car.md) | Steps as data plus groups, `dt`-based drop-in animation, a real bug to fix | 80 |
| | **Total** | | **750 (12.5 h)** |

The minutes are for someone with the background above, exercise included.

## Map of `index.html`

The file has three parts stacked on top of each other:

```
 index.html (1,319 lines)
 +-------------------------------------------+
 | lines 6-163    CSS  <style>   (the look)  |
 +-------------------------------------------+
 | lines 165-267  HTML markup (the structure)|
 +-------------------------------------------+
 | lines 269-1319 JavaScript <script>        |
 |                (the behaviour)            |
 +-------------------------------------------+
```

In detail ("read in" = the lesson where you read it):

| Lines | What | Read in |
|---|---|---|
| 1-5 | `<title>` and fonts. There is no `<!doctype html>`, `<html>` or `<body>`: the file is a page fragment and the browser fills those in. Not an error. | 00 |
| 6-163 | CSS. Colour variables such as `--accent`, `--ink`, `--stage`; dark-mode versions at 17-26 | skim |
| 165-176 | Header and the tab links in `<nav id="nav">` (168-174) | 02 |
| 178-266 | `<main>` with the five views, `<section class="view" id="v-explore">` to `v-weld` (all but the first start `hidden`) | 02 |
| 267 | `<div class="tip" id="tip" hidden>`, the hover tooltip | 10 |
| 269-270 | Three.js r128 and `OrbitControls.js`, loaded from the internet (CDN) | 06, 07 |
| 272-281 | Helpers: `$`, `cssVar`, `store`, `esc`, `tip`, `HAS3D` | 01 |
| 283-343 | 3D VIEWER: `createViewer()` 287-342, `isShown()` 343 | 06, 07, 10 |
| 345-365 | Geometry helpers: `M`, `add`, `box`, `cylY`, `cylX`, `cylZ`, `tube`, `helix` | 08 |
| 367-583 | ENGINE EXPLORER: `ENGINE_INFO` 370-392 (text), `explodeProgress` 397-402, `ENG` 404, `buildEngine` 405-522 (`part()` 412-417, the 21 parts 418-520, `v.onClick` 521), `selectEnginePart` 524-548, `applyIsolation` 549-555, `initEngine` 557-582 (its `v.tick` 570-580) | 08, 09, 10 |
| 584-880 | BUILD A CAR: `WB`, `TR`, `WY` 587, `STEPS` 588-802 (14 steps, mostly text), `CAR` 804, `initBuild` 805-848, `stopPlay` 849, `goStep` 850-864, `renderStep` 865-879 | 11 |
| 881-1015 | MAINTENANCE: `GUIDES` 884-965 (14 jobs), `initMaintain` 967-1014 | 03 |
| 1016-1112 | ELECTRICAL: `initElectrical` 1019-1077 (lists `comps` 1020, `sensors` 1030, `codes` 1042, `tests` 1043), `schematicSVG` 1078-1111 | 03 (skim), 05 |
| 1113-1277 | WELD & FAB: `jointSVG` 1116, `MODULES` 1125-1225 (7 modules), `initWeld` 1227-1276 (quiz logic 1244-1259, calculator 1262-1271) | 04, 05 |
| 1278-1319 | ROUTING & LOOP: `VIEWS` 1281, `route()` 1285-1305, `loop()` 1307-1312, theme repaint 1313-1316, start-up 1317-1318 | 02, 06 |

**Content you can skim** (it's words, not logic): `ENGINE_INFO`, `STEPS` (except the two steps lesson 11
names), `GUIDES`, `MODULES`, the electrical lists. `tube` and `helix` (354-365) are optional.

**Finding your way in a 135 KB file:** search for the banner comments (`/* ====`), use "Go to line"
(Ctrl+G in VS Code), and turn on word-wrap (four lines are over 1,000 characters long).

## Life of the page

What happens, in order, from opening the file to the first picture on screen. Lesson 02 walks
through it; come back to it whenever you wonder "who calls this?".

```
browser opens index.html
  |
  +-> CSS + HTML are read. All five <section>s exist; all but #v-explore are hidden.
  |
  +-> line 269-270: three.min.js and OrbitControls.js download from the CDN
  |                 (HAS3D, line 281, is true only if both arrived)
  |
  +-> the big <script> runs top to bottom (lines 271-1316):
  |     it only DEFINES helpers, data and functions. Nothing visible happens yet.
  |
  +-> line 1317: route()
  |     reads location.hash  ("" -> "explore")
  |     un-hides <section id="v-explore">, hides the others
  |     first visit to "explore"?  inited.explore = true
  |         -> initEngine() -> createViewer()  (renderer, camera, lights)
  |                         -> buildEngine()   (the 21 parts)
  |     active3d = that viewer
  |
  +-> line 1318: requestAnimationFrame(loop)
        loop(now), about 60 times a second, forever:
            active3d.tick(now)            (move parts)
            controls.update()             (orbit camera)
            renderer.render(scene, camera)  (draw)

user clicks the "Build a Car" tab
  -> URL becomes ...#build -> browser fires "hashchange" -> route() again
       first visit?  initBuild()      later visits: only show/hide
       active3d = the car viewer, so the loop now draws the car
```

## Saved progress

The app stores checkmarks and quiz passes in your browser's localStorage under the keys `tg-maint`,
`tg-maint-sel`, `tg-weld` and `tg-weld-sel` (lesson 03 explains them). To reset, use DevTools >
Application > Local Storage and delete them, or use "Clear checkmarks" and "Retry quiz" in the app.

## Words

All new terms are in the [glossary](GLOSSARY.md).
