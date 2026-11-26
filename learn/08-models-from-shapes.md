# 08. Models from basic shapes

**Time:** 90 min · **Difficulty:** 3 of 5 · Previous: [07](07-orbit-controls.md) · Next: [09 Teardown slider](09-teardown-slider.md)

## Goal

Build 3D parts from boxes and cylinders, group them, and attach your own data to them. By the end you
add a new part to the engine.

## Read these lines

- The geometry helpers, lines 346-353. (`tube` and `helix`, lines 354-365, are optional.)
- One `ENGINE_INFO` entry, line 371 (it's long: just see its fields).
- `buildEngine`, lines 405-417, then two parts: `block` (lines 418-423) and `pistons` (lines 437-443).
- `v.onClick`, line 521.

## The ideas

### Mesh = geometry + material

A **mesh** is something you can see. It needs a **geometry** (the shape: a box, a cylinder) and a
**material** (the surface: colour, shiny or dull). A **group** is an invisible node that holds children.

```
Scene
 +-- Group "block"          <- moving the group moves everything inside
 |     +-- Mesh (box 4.0 x 2.4 x 2.2, alu material)
 |     +-- Mesh (thin cylinder, dark)   x4   cylinder tops
 |     +-- Mesh (box, alu)                   front cover face
 |     +-- Mesh (box, alu)              x5   ribs
 +-- Group "oilpan"
 |     +-- ...
 +-- ... 21 groups in all
```

It's a tree, like nodes with child lists. A child's position is **relative to its parent**. The teardown
slider (lesson 09) moves whole groups; the meshes inside come along.

### The helpers (lines 346-353)

```js
const M = (color, o = {}) => new THREE.MeshStandardMaterial(Object.assign({ color, metalness: 0.45, roughness: 0.45 }, o));
function add(g, geo, mat, p = [0, 0, 0], r = [0, 0, 0]) {
  const m = new THREE.Mesh(geo, mat); m.position.set(...p); m.rotation.set(...r); g.add(m); return m;
}
const box = (x, y, z) => new THREE.BoxGeometry(x, y, z);
const cylY = (r, h, s = 28) => new THREE.CylinderGeometry(r, r, h, s);
const cylX = (r, h, s = 28) => new THREE.CylinderGeometry(r, r, h, s).rotateZ(Math.PI / 2);
```

- `M(color, overrides)` makes a material. `Object.assign` copies the overrides on top of the defaults:
  `M(0x737a82, { roughness: 0.55 })` keeps `metalness: 0.45` but changes the roughness.
  **metalness** 0-1 = how metallic; **roughness** 0-1 = matte (1) or glossy (0).
- `add(g, geo, mat, position, rotation)` makes a mesh, places it, puts it in group `g`, and returns it.
  Position is `[x, y, z]`; rotation is in radians around x, y and z.
- `box(x, y, z)` is a box of that width, height and depth, centred on its position.
- Three.js cylinders stand along **y**. `cylX` lies one down along x by rotating it 90 degrees
  (`Math.PI / 2`) around z. `cylZ` does the same toward z.

### The engine layout

```
 seen from above (y up, out of the page)          the crank throws, seen from the front

   z                                               y
   ^      X = -1.35  -0.45   0.45   1.35           ^   S = [ 1,  -1,  -1,   1 ]
   |         (1)     (2)     (3)     (4)            |     up   down  down   up   (x THROW = 0.35)
   |    +-----o-------o-------o-------o----+        |
   +--> x   four cylinder centres along x         ---- CY = -0.9: crankshaft height
```

Line 406 sets the numbers every part reuses:

```js
const X = [-1.35, -0.45, 0.45, 1.35], S = [1, -1, -1, 1], CY = -0.9, THROW = 0.35;
```

`X` holds the four cylinder positions, `CY` the crankshaft height, and `S` says whether each cylinder's
crank throw (and piston) is up or down. Cylinders 1 and 4 are up while 2 and 3 are down, as in a real
inline-4.

### Why materials are functions (lines 407-411)

```js
const alu = () => M(0xb7bec6, { metalness: 0.55, roughness: 0.38 });
```

`alu` is a *function* that makes a **new** material every call. When you click a part (lesson 10) the
app changes that part's material to make it glow. If several parts shared one material object, changing
it would change all of them. The exercise lets you see this happen.

### `part()`: one engine part (lines 412-417)

```js
function part(id, stage, dir, build) {
  const g = new THREE.Group(); build(g); v.scene.add(g);
  g.traverse(o => { if (o.isMesh) { o.userData.partId = id; o.userData.label = ENGINE_INFO[id].name; } });
  v.pickables.push(g);
  ENG.parts.push({ id, stage, dir: new THREE.Vector3(...dir), group: g });
}
```

1. Make an empty group and let your `build` callback fill it (lesson 01, callbacks).
2. `traverse` visits the group and every mesh under it. Each mesh gets two fields in **`userData`**, a
   free object every Three.js object has for your own data: which part it belongs to, and the name to show
   on hover. That is how a click on one small mesh later finds its part (lesson 10).
3. Register the group as clickable (`v.pickables`) and as an engine part with its `stage` and `dir`
   (lesson 09).

The name comes from `ENGINE_INFO[id]` (lines 370-392): one object per part with `name`, `sys`, `does`,
`how`, `fails` (a list) and `care`.

### Two parts, read slowly

The block (lines 418-423), with the cylinder tops placed at each `X`:

```js
part("block", 0, [0, 0, 0], g => {
  add(g, box(4.0, 2.4, 2.2), alu());
  X.forEach(x => add(g, cylY(0.42, 0.02), dark(), [x, 1.205, 0]));
```

The pistons (lines 437-443): one cylinder per `X`, high or low depending on `S`, with three thin rings
(`TorusGeometry`, a doughnut) laid flat by rotating 90 degrees around x:

```js
X.forEach((x, i) => {
  const y = S[i] > 0 ? 0.75 : 0.05;
  add(g, cylY(0.4, 0.45), alu(), [x, y, 0]);
  [0.15, 0.08, 0.0].forEach(dy => add(g, new THREE.TorusGeometry(0.4, 0.014, 6, 32), dark(), [x, y + dy, 0], [Math.PI / 2, 0, 0]));
```

The last line of `buildEngine` (line 521) fills the viewer's click slot; lesson 10 follows it.

## Exercise

**(a) Recolour.** Change `alu` (line 407) to red: `M(0xb03030, ...)`.

**(b) Add a dipstick.** Three edits:

1. In `ENGINE_INFO`, add an entry (for example before `oilpan:` on line 391):
   ```js
   dipstick: { name: "Dipstick", sys: "Lubrication", does: "Shows the oil level.", how: "A long steel blade reaching into the oil pan.", fails: ["Oil level reads low or high"], care: "Check it with the engine off and level." },
   ```
2. In `buildEngine`, just before `v.onClick` (line 521):
   ```js
   part("dipstick", 1, [0, 4, 0], g => {
     add(g, cylY(0.03, 1.6), steel(), [1.7, 1.5, 1.2]);
   });
   ```
3. Add `"dipstick"` to the end of the `order` list in `initEngine` (line 560).

**(c) Break it on purpose.** Make `alu` return one shared material:

```js
const ALU = M(0xb7bec6, { metalness: 0.55, roughness: 0.38 });
const alu = () => ALU;
```

Click the engine block, then click the alternator.

**You should now see:** (a) every aluminium part turns red: block, head, pistons, throttle body, water pump and alternator. (b) a "Dipstick" chip at the end of
the parts list, a thin rod at the right-hand end of the block, on the side facing you with a "Dipstick" hover label, the info panel
filled when you click it, and it lifts straight up when you move the slider. (c) clicking the block makes
nothing visible glow (only its four dark cylinder-top discs, which are hidden under the head); clicking the alternator makes **every** aluminium
part glow: block, head, pistons, throttle body and water pump too. Undo (c).

<details><summary>Answer (c): why the wrong parts light up</summary>

`selectEnginePart` (lines 527-531) walks over *all* parts in order and sets each mesh's material to
"glow" (selected part) or "no glow" (every other part). With one shared material, every aluminium part
writes to the same object, so the **last** aluminium part in `ENG.parts` wins. That is the alternator.
When you click the block, the later aluminium parts reset the shared material to "no glow"; when you
click the alternator, it writes "glow" last, and every part using that material shows it. That's why
`alu()` makes a new material each time.

</details>

## Traps

- **Forgot the `ENGINE_INFO` entry:** `part()` throws (`ENGINE_INFO[id].name` of undefined), and the whole
  Engine tab shows "WebGL may be disabled" (lesson 02). The real error is in the Console.
- **Forgot the `order` entry:** the part works in 3D but has no chip.
- **Positions are relative to the group,** and boxes are centred on their position.
- **Degrees typed instead of radians:** `rotation.set(90, 0, 0)` spins 90 *radians* (about 14 turns).

## New words

[Mesh](GLOSSARY.md#mesh) · [Geometry](GLOSSARY.md#geometry) · [Material](GLOSSARY.md#material) ·
[Group](GLOSSARY.md#group) · [Scene graph](GLOSSARY.md#scene-and-scene-graph) ·
[userData](GLOSSARY.md#userdata) · [traverse](GLOSSARY.md#traverse) · [Vector3](GLOSSARY.md#vector3) ·
[Radian](GLOSSARY.md#radian) · [Callback](GLOSSARY.md#callback)
