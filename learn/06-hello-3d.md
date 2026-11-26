# 06. A 3D scene: renderer, camera, lights, loop

**Time:** 100 min · **Difficulty:** 4 of 5 (the hardest climb) · Previous: [05](05-svg-schematics.md) · Next: [07 Orbit](07-orbit-controls.md)

## Goal

Understand the 3D pipeline well enough to read the first half of `createViewer` (lines 288-316) and the
render loop. You will first write a tiny 3D scene yourself, so the real code looks familiar.

## Read these lines

- The library tags, lines 269-270, and `HAS3D`, line 281.
- `createViewer`, **lines 288-316 only**. (Lines 295-298 are lesson 07; lines 317-339 are lesson 10.)
- `initEngine` line 558 (the options passed in).
- `loop`, lines 1307-1312, and the theme repaint, lines 1313-1316.

## The ideas

### The pipeline

```
   Scene  (a tree of objects: meshes, lights, groups)
     +
   Camera (where you look from, and how wide)
     |
     v
   Renderer  --draws-->  <canvas> on the page
     ^
     |  again and again: requestAnimationFrame(loop), about 60 times a second
```

Three.js is a library on top of WebGL, the browser's API for drawing with the graphics card. You build
a **scene**, put a **camera** in it, and ask the **renderer** to draw what the camera sees into a
`<canvas>`. Nothing moves unless you draw again, so there is a **render loop**: like a game's
`while (true) { update(); draw(); }`, except the browser calls your function once per screen refresh.

### Axes and units

```
        y (up)
        |
        |
        +------ x (right)
       /
      /
     z (toward you)
```

Three.js is **right-handed**: x right, y **up**, z out of the screen toward you. (SVG in lesson 05 had
y *down*.) Units are whatever you decide; this file treats 1 unit as about a metre. Line 587 says so:

```js
const WB = 1.35, TR = 0.78, WY = 0.34; // half wheelbase, half track, wheel center height (meters)
```

Angles are in **radians**: `Math.PI` = 180 degrees, `Math.PI / 2` = 90 degrees.

### The camera

Line 293:

```js
const camera = new THREE.PerspectiveCamera(40, 1, 0.05, 400);
```

```
 camera                          far plane (400)
   .  \                         |
   .    \       visible          |
 eye ---[near]--- volume  --------|     40 = vertical field of view, in degrees
   .    /       (frustum)        |      1  = aspect ratio width/height (fixed in resize)
   .  /                         |      0.05 / 400 = nearest and farthest distance drawn
```

Only what is inside this pyramid with its tip cut off (the **frustum**) is drawn. Line 294 puts the
camera where the options say: `camera.position.set(...opt.cam)`, and for the engine (line 558)
`cam: [10, 7, 12]`.

### Lines 288-291: the renderer

```js
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setPixelRatio(Math.min(window.devicePixelRatio || 1, 2));
renderer.outputEncoding = THREE.sRGBEncoding;
host.prepend(renderer.domElement);
```

- `antialias` smooths jagged edges.
- `setPixelRatio`: on high-resolution screens draw more pixels so it looks sharp, but at most 2× (cost).
- `sRGBEncoding`: makes colours come out the way they were specified.
- `renderer.domElement` is the `<canvas>`; `prepend` puts it inside the stage box `host`.

### Lines 299-302: why lights

`MeshStandardMaterial`, which every part uses, is lit like a real surface. With no light, it is black.
There are four lights, named the way photographers name them:

```js
scene.add(new THREE.HemisphereLight(0xe6eef5, 0x3a2e22, 0.85));   // soft sky colour above, ground colour below
const key = new THREE.DirectionalLight(0xffffff, 1.05); key.position.set(8, 14, 10); scene.add(key);   // main light
```

…plus `rim` (bluish, from behind) and `fill` (warm, from below) to soften shadows. Colours are written
as hex `0xRRGGBB`, the last number is brightness.

### Lines 304-311: the viewer object and the theme

```js
const v = { host, renderer, scene, camera, controls, tick: null, pickables: [] };
```

`v` bundles everything about one viewer. `{ host, renderer }` is short for `{ host: host, renderer:
renderer }`. `tick` and `pickables` are empty slots that the engine and car code fill in later (lessons 08-11).

`v.applyTheme` (lines 305-310) sets the background to the CSS colour `--stage` and draws a floor grid
(`GridHelper`). Lines 1313-1316 call it again on every viewer when the light/dark theme changes.

### Lines 312-316: resize

```js
const resize = () => {
  const w = host.clientWidth, h = host.clientHeight; if (!w || !h) return;
  renderer.setSize(w, h, false); camera.aspect = w / h; camera.updateProjectionMatrix();
};
new ResizeObserver(resize).observe(host); resize();
```

A `ResizeObserver` calls `resize` whenever the stage box changes size (window resized, tab shown).
The camera's aspect ratio must match the canvas or the picture is stretched. After changing a camera
setting you must call `updateProjectionMatrix()`.

### The loop (lines 1307-1312)

```js
function loop(now) {
  requestAnimationFrame(loop);          // book the next frame first
  const v = active3d; if (!v) return;   // no 3D tab visible: draw nothing
  if (v.tick) v.tick(now);              // let the engine/car move things
  v.controls.update(); v.renderer.render(v.scene, v.camera);
}
```

`now` is the time in milliseconds, given by the browser.

### Why Three.js r128

Lines 269-270 load version **r128** from a CDN as two plain `<script>` files. Newer versions no longer
ship `examples/js` (where this `OrbitControls.js` lives) and expect JavaScript modules instead. Staying on
r128 keeps the two simple `<script>` tags working. `HAS3D` (line 281) is true only if both loaded:

```js
const HAS3D = typeof THREE !== "undefined" && THREE.OrbitControls;
```

## Exercise

**(a) Hello cube.** In a new file `hello.html`, *outside this repository* (or don't commit it), write a
scene with: the r128 `<script>` tag from line 269, a renderer added to the page, a box made of
`BoxGeometry` and `MeshStandardMaterial`, one `DirectionalLight`, a camera at `(3, 2, 5)` looking at the
origin, and a loop that turns the cube. About 25 lines. Try it before opening the answer.

**(b) Move the real camera.** In `index.html` line 558 change `cam: [10, 7, 12]` to `cam: [0, 3, 20]`.

**You should now see:** (a) an orange cube slowly turning; the faces turned away from the light are pure
black, and if you delete the light the whole cube goes black. (b) the engine seen square-on from its long
side: four coils on top in a row from left to right, the intake manifold facing you, the timing belt on
the left and the flywheel on the right. Undo (b).

<details><summary>Answer (a)</summary>

```html
<!doctype html>
<title>Hello cube</title>
<style>body { margin: 0; }</style>
<body>
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(40, window.innerWidth / window.innerHeight, 0.1, 100);
camera.position.set(3, 2, 5);
camera.lookAt(0, 0, 0);

const cube = new THREE.Mesh(new THREE.BoxGeometry(1, 1, 1), new THREE.MeshStandardMaterial({ color: 0xd08030 }));
scene.add(cube);

const light = new THREE.DirectionalLight(0xffffff, 1);
light.position.set(5, 8, 6);
scene.add(light);

function loop() {
  requestAnimationFrame(loop);
  cube.rotation.y += 0.01;
  renderer.render(scene, camera);
}
requestAnimationFrame(loop);
</script>
```

The `<body>` tag matters here: this script runs as soon as it is read, and without `<body>` above it
`document.body` doesn't exist yet. Add a `HemisphereLight` like line 299 and the black faces get some
soft light: that is exactly why the app uses four lights.

</details>

<details><summary>Answer (b): why that view</summary>

The camera stands on the z axis, 20 units out, 3 up, and looks at the target `[0, 0.4, 0]`. The engine's
long side runs along x, so its four cylinders (`X = [-1.35, -0.45, 0.45, 1.35]`, lesson 08) line up left
to right, and the parts at positive z (intake, alternator, oil filter) face you.

</details>

## Traps

- **Blank or black canvas:** no light; camera inside the object; or you forgot `renderer.render` in the
  loop.
- **Canvas 0 pixels tall:** the box it is in has no height. (In the app, the CSS gives `.stage` a height.)
- **`...opt.cam`** is just "pass the 3 numbers" (lesson 01).
- **Errors in the Engine tab show "WebGL may be disabled"**: read the Console (lesson 02).

## New words

[Three.js](GLOSSARY.md#threejs) · [WebGL](GLOSSARY.md#webgl) · [Renderer](GLOSSARY.md#renderer) ·
[Scene](GLOSSARY.md#scene-and-scene-graph) · [PerspectiveCamera](GLOSSARY.md#perspectivecamera) ·
[Frustum](GLOSSARY.md#frustum) · [Radian](GLOSSARY.md#radian) · [Render loop](GLOSSARY.md#render-loop) ·
[requestAnimationFrame](GLOSSARY.md#requestanimationframe) · [ResizeObserver](GLOSSARY.md#resizeobserver) ·
[Material](GLOSSARY.md#material) · [Mesh](GLOSSARY.md#mesh)
