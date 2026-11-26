# 07. Orbit and zoom

**Time:** 35 min · **Difficulty:** 2 of 5 · Previous: [06](06-hello-3d.md) · Next: [08 Models](08-models-from-shapes.md)

## Goal

Know what OrbitControls does for you, and the three places it touches the code: set-up, the loop, and
the buttons that steer it.

## Read these lines

- The script tag, line 270.
- Set-up in `createViewer`, lines 295-298.
- `v.controls.update()` in `loop`, line 1311.
- The Auto-rotate button `#btnSpin`, line 567.
- "Center camera on it": the button handler, lines 544-547, and the glide in the tick, lines 576-579.

## The ideas

### The camera rides on a sphere

```
                 . - ~ ~ - .
             .'      cam     '.         left-drag:  move the camera around the sphere
           /        o          \                    (change the two angles)
          |          \          |       scroll:     change the radius (zoom),
          |           \ radius  |                   kept between minDistance and maxDistance
          |            * target |       right-drag: move the target itself (pan)
           \                   /
             '.             .'
                 ' - . . - '
```

OrbitControls keeps the camera on a sphere around a **target** point and always looks at it. In
physics terms the camera position is in spherical coordinates: a radius and two angles. Your mouse
changes those numbers; OrbitControls turns them back into `camera.position`.

### Set-up (lines 295-298)

```js
const controls = new THREE.OrbitControls(camera, renderer.domElement);
controls.target.set(...opt.target);
controls.enableDamping = true; controls.dampingFactor = 0.08; controls.autoRotateSpeed = 1.2;
controls.minDistance = opt.minD; controls.maxDistance = opt.maxD;
```

It needs the camera to move and the canvas to listen to. For the engine (line 558) the target is
`[0, 0.4, 0]`, and zoom is kept between 4 and 45 units.

### Damping needs `update()` every frame

With **damping**, the view keeps gliding after you let go and slows down a bit each frame (here it keeps
92% of its speed per frame: `1 - 0.08`). That slowing-down happens inside `controls.update()`, so it
only works if `update()` is called every frame. That is line 1311:

```js
v.controls.update(); v.renderer.render(v.scene, v.camera);
```

Auto-rotate also lives in `update()`. The button just flips a flag (line 567):

```js
$("#btnSpin").onclick = e => { v.controls.autoRotate = !v.controls.autoRotate; e.target.classList.toggle("on", v.controls.autoRotate); };
```

### `lerp`: a smooth glide to a new target

**lerp** (linear interpolation) means "go a fraction of the way from a to b". `a.lerp(b, 0.1)` moves `a`
10% of the remaining distance toward `b`. Called every frame, the steps shrink as you get closer: a
smooth glide that slows down at the end.

"Center camera on it" stores the centre of the chosen part (lines 544-547), and the engine's tick glides
the target there (lines 576-579):

```js
if (ENG.focus) {
  v.controls.target.lerp(ENG.focus, 0.1);
  if (tmp.copy(v.controls.target).distanceTo(ENG.focus) < 0.01) ENG.focus = null;   // close enough: stop
}
```

## Exercise: change three numbers

One at a time, reload after each, and try dragging and zooming in the Engine tab:

- **(a)** in `createViewer` (line 297) change `controls.dampingFactor = 0.08` to `0.01`;
- **(b)** in `initEngine` (line 558) change `minD: 4` to `minD: 1`;
- **(c)** in `loop` (line 1311) temporarily comment out the update: `// v.controls.update();`
  (keep the `v.renderer.render(...)` part).

**You should now see:** (a) long, floaty glides after you let go; (b) you can zoom right into the engine
block (and inside it); (c) dragging still turns the view, but sluggishly and the glide stops dead when
you stop moving the mouse; Auto-rotate does nothing, and "Center camera on it" no longer turns the view.
Undo all three.

<details><summary>Answer: why (c) behaves like that</summary>

OrbitControls itself calls `update()` once on each mouse move, so dragging still does something. With
damping on, each `update()` applies only 8% of the pending movement; without the per-frame calls the
rest never arrives once you stop moving. Auto-rotate and the camera turning toward a new target only
happen inside `update()`, so they stop completely.

</details>

## Traps

- **Load order:** `OrbitControls.js` (line 270) must come after `three.min.js` (line 269), because it
  attaches itself to `THREE`. If either fails, `HAS3D` (line 281) is false and the 3D tabs show an error.
- **Too small `minD`** lets the camera inside parts, where you see nothing useful (faces are drawn from
  the outside only).

## New words

[OrbitControls](GLOSSARY.md#orbitcontrols) · [Damping](GLOSSARY.md#damping) · [lerp](GLOSSARY.md#lerp) ·
[Vector3](GLOSSARY.md#vector3)
