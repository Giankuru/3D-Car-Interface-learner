# Teardown Garage

An interactive, single-page learning app for cars: engines, building a car, maintenance, electrical and welding/fabrication.

Open `index.html` in any modern browser. No install or build step needed (Three.js loads from a CDN).

You need an internet connection for the two 3D tabs (Engine and Build a Car), because Three.js is downloaded when the page opens. Offline, those tabs show "The 3D library could not load"; Maintenance, Electrical and Weld & Fab still work. The file starts at `<title>` without `<!doctype html>`, `<html>` or `<body>`: it is a page fragment, and browsers fill those in, so opening it directly is fine.

## Sections

- **Engine Explorer:** a 3D inline-4 DOHC engine. Orbit, zoom, and drag the teardown slider to pull parts off in the order a mechanic removes them. Click any part for what it does, how it works, failure symptoms and maintenance.
- **Build a Car:** start from a bare ladder frame and add 14 systems one at a time (suspension, steering, brakes, powertrain, driveline, exhaust, fuel, cooling, electrical, wheels, interior, body, glass and lights).
- **Maintenance:** 14 step-by-step jobs with tools, parts, safety notes and checklists (flat tire, battery, jump start, oil change, filters, wipers, bulbs, fluids, spark plugs, brake pads, serpentine belt, fuses).
- **Electrical:** how 12 V negative-ground circuits work, a starting and charging schematic, components, multimeter tests, sensors and OBD-II codes.
- **Weld & Fab:** a 7-module course with quizzes, covering safety, metals, MIG/flux-core/stick/TIG, joints, settings and defects, fabrication basics, and car projects. Includes a MIG settings calculator.

The 3D models are simplified representations built from basic shapes, not OEM CAD data. Torque specs and service intervals are typical values. Always check your vehicle's service manual.

## Learn how it's built

[`learn/README.md`](learn/README.md) is a step-by-step guide to the code, for someone who can program a little but is new to JavaScript and 3D: 12 lessons (about 12.5 hours) from "open the file" to raycasting and animation, each with an exercise, plus a map of `index.html` and a [glossary](learn/GLOSSARY.md).

## Project layout

Everything is in `index.html`: CSS (lines 6-163), the HTML for the five tabs (165-267) and one script (269-1319). The [map in the guide](learn/README.md#map-of-indexhtml) lists every section with its line numbers.

Three.js is pinned to **r128** on purpose: `OrbitControls` is loaded as a plain script from `examples/js`, which newer Three.js versions no longer ship (they expect JavaScript modules instead).

## Saved progress

Maintenance checkmarks and quiz passes are saved in your browser's localStorage under `tg-maint`, `tg-maint-sel`, `tg-weld` and `tg-weld-sel`. To reset, press "Clear checkmarks" on a job or "Retry quiz" in a module (a passed module stays passed), or delete those keys in your browser's developer tools (F12 > Application > Local Storage).

## Customizing the teardown

`explodeProgress()` in `index.html` controls how each part moves as the slider goes from 0 to 1. Each engine part has a `stage` (removal order, 0 = first off) and a `dir` (how far and which way it pulls out). They are the 2nd and 3rd arguments of the `part(...)` calls in `buildEngine()`, e.g. `part("plugs", 1, [0, 7.8, 1.2], ...)`. `ENG.maxStage` must equal the highest stage used, or the last parts never come fully out. The slider only sets `ENG.target`; the `tick` function in `initEngine()` eases `ENG.t` toward it every frame and moves each part to `dir × explodeProgress(...)`. [Lesson 09](learn/09-teardown-slider.md) walks through it.
