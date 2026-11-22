# Teardown Garage

An interactive, single-page learning app for cars: engines, building a car, maintenance, electrical and welding/fabrication.

Open `index.html` in any modern browser. No install or build step needed (Three.js loads from a CDN).

## Sections

- **Engine Explorer:** a 3D inline-4 DOHC engine. Orbit, zoom, and drag the teardown slider to pull parts off in the order a mechanic removes them. Click any part for what it does, how it works, failure symptoms and maintenance.
- **Build a Car:** start from a bare ladder frame and add 14 systems one at a time (suspension, steering, brakes, powertrain, driveline, exhaust, fuel, cooling, electrical, wheels, interior, body, glass and lights).
- **Maintenance:** 14 step-by-step jobs with tools, parts, safety notes and checklists (flat tire, battery, jump start, oil change, filters, wipers, bulbs, fluids, spark plugs, brake pads, serpentine belt, fuses).
- **Electrical:** how 12 V negative-ground circuits work, a starting and charging schematic, components, multimeter tests, sensors and OBD-II codes.
- **Weld & Fab:** a 7-module course with quizzes, covering safety, metals, MIG/flux-core/stick/TIG, joints, settings and defects, fabrication basics, and car projects. Includes a MIG settings calculator.

The 3D models are simplified representations built from basic shapes, not OEM CAD data. Torque specs and service intervals are typical values. Always check your vehicle's service manual.

## Customizing the teardown

`explodeProgress()` in `index.html` controls how each part moves as the slider goes from 0 to 1. Each engine part has a `stage` (removal order) and a `dir` (the direction it pulls out).
