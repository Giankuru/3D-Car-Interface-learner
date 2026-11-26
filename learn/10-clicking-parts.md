# 10. Clicking a part: raycasting and highlight

**Time:** 85 min · **Difficulty:** 4 of 5 · Previous: [09](09-teardown-slider.md) · Next: [11 Build a Car](11-build-a-car.md)

## Goal

Trace, step by step, what happens between "I click on the engine" and "the info panel fills".

```
pointerdown / pointerup on the canvas      (lines 327-331)  moved < 5 px? then it's a click
   -> v.pick(e)                            (lines 318-324)  pixel -> NDC -> ray -> first visible hit
   -> v.onClick(hit)                       (line 521)       hit mesh -> userData.partId
   -> selectEnginePart(id)                 (lines 524-548)  glow, chips, isolation, info panel
```

## Read these lines

- `createViewer`, lines 317-339 (picking and pointer events), and `isShown`, line 343.
- `v.onClick`, line 521.
- `selectEnginePart`, lines 524-548, and `applyIsolation`, lines 549-555.
- The chips' click handler, line 562.

Take the four listeners one at a time.

## The ideas

### Step 1: is it a click or a drag? (lines 326-331)

Dragging orbits the camera (lesson 07), so a drag must not select anything.

```js
let down = null;
renderer.domElement.addEventListener("pointerdown", e => { down = [e.clientX, e.clientY]; });
renderer.domElement.addEventListener("pointerup", e => {
  if (down && Math.hypot(e.clientX - down[0], e.clientY - down[1]) < 5 && v.onClick) v.onClick(v.pick(e));
  down = null;
});
```

`addEventListener(type, fn)` registers `fn` to be called on that event. "Pointer" events cover mouse,
pen and touch. Remember where the button went down; on release, if it moved less than 5 pixels
(`Math.hypot` = length of the movement), treat it as a click.

### Step 2: pixels to NDC (lines 319-320)

The mouse position is in pixels with (0, 0) at the top left and y going **down**. Three.js wants
**NDC** (normalised device coordinates): −1 to 1 on both axes, (0, 0) in the centre, y going **up**.

```
 canvas pixels                           NDC
 (0,0) ------------> x  (width)          (-1, 1) ------------ (1, 1)
   |                                        |         ^ y        |
   |       * mouse                          |         |          |
   v                                        |  -------0------> x |
   y (height)                               |         |          |
                                         (-1,-1) ------------ (1,-1)
```

```js
const r = renderer.domElement.getBoundingClientRect();
ndc.set(((ev.clientX - r.left) / r.width) * 2 - 1, -((ev.clientY - r.top) / r.height) * 2 + 1);
```

Read the x part from the inside out: `ev.clientX - r.left` = pixels from the canvas's left edge;
`/ r.width` = 0..1; `* 2 - 1` = −1..1. The y part is the same with a minus in front, which flips it so
up is positive.

### Step 3: shoot a ray (lines 321-323)

```
 camera eye
     o ----------- ray through the mouse point ---------->
      \        |                   |
       \  near plane          [ part A ]   [ part B ]
                            first hit        second hit
```

```js
ray.setFromCamera(ndc, camera);
const hits = ray.intersectObjects(v.pickables, true);
return hits.find(h => h.object.visible && isShown(h.object) && h.object.material.opacity > 0.3);
```

A **raycaster** shoots a line from the camera through that point on the screen and lists every mesh it
passes through, **nearest first**. `true` means "also search the children" (the pickables are groups).
Then it keeps the first hit that is:

- visible, and not inside a hidden parent (`isShown`, line 343, walks up the tree checking `visible`);
- not faded out (opacity above 0.3; isolation fades parts to 0.1, see below).

`find` returns `undefined` if nothing qualifies: you clicked empty space.

### Step 4: hit → part (line 521)

```js
v.onClick = h => selectEnginePart(h ? h.object.userData.partId : null);
```

`h.object` is the exact small mesh that was hit. Its `userData.partId` was stamped on in `part()`
(lesson 08). `v.onClick` is a callback slot: `createViewer` calls it without knowing what the engine
will do with it. Clicking empty space passes `null`, which deselects.

### Step 5: highlight with emissive (lines 526-531)

**Emissive** is light a surface gives off by itself, so it shows even in shadow. The selected part's
meshes get the accent colour at strength 0.45, all others get 0:

```js
o.material.emissive.copy(p.id === id ? accent : new THREE.Color(0));
o.material.emissiveIntensity = p.id === id ? 0.45 : 0;
```

(This is why each part needs its own materials, lesson 08.) Then lines 532-535 mark the chip, enable the
Isolate button, and apply isolation.

### Step 6: isolate (lines 549-555)

```js
const faded = ENG.isolate && ENG.sel && p.id !== ENG.sel;
o.material.transparent = faded; o.material.opacity = faded ? 0.1 : 1; o.material.depthWrite = !faded;
```

Every other part becomes 10% opaque. Why `depthWrite = false`? The renderer keeps a **depth buffer**:
for every pixel it remembers how near the closest thing drawn there so far is, and skips anything
further away.

```
 camera ->   [faded cover, 10%]   [selected camshaft]
             depth written here   -> camshaft is "behind" it: skipped, invisible!
 with depthWrite = false the cover leaves no depth mark, so the camshaft is drawn.
```

### Step 7: the panel (lines 536-543)

`ENGINE_INFO[id]` gives the words, and a template string with `esc()` writes them into `#partInfo`
(lesson 01). The chips row uses event delegation, like lesson 03 (line 562).

### Hover labels (lines 332-338)

Same `v.pick`, on every mouse move. `e.buttons` is non-zero while a button is held (dragging), so the
tooltip hides. Otherwise the hit mesh's `userData.label` goes into the `#tip` box at the mouse position.

## Exercise

**(a)** Change line 521 to log every click:

```js
v.onClick = h => { console.log(h && h.object.userData); selectEnginePart(h ? h.object.userData.partId : null); };
```

**(b)** On line 329 change `< 5` to `< 80`, then try short drags across the engine.

**(c)** Undo (b). Select the camshafts, press "Isolate part", then click a faded part in front of them.

**You should now see:** (a) objects like `{partId: "head", label: "Cylinder head"}` in the Console;
clicking empty space logs `undefined` and clears the selection. (b) short drags now also select parts.
(c) the click goes straight through the faded part: you get the selected part behind it, or nothing
(which deselects and ends isolation).

<details><summary>Answer (c): why</summary>

The faded part is still hit by the ray, but the `find` on line 323 skips it because its opacity is 0.1,
not above 0.3. The next hit is used instead. Only the selected part is not faded, so it's either that
part or `undefined`. With `undefined`, `selectEnginePart(null)` runs, and line 534 turns isolation off.

</details>

## Traps

- **The NDC line:** go back to the diagram and do one number by hand (e.g. a click in the exact middle
  should give 0, 0).
- **`v.onClick` is defined far from where it is called** (line 521 vs line 329). That's a callback slot,
  as in lesson 01.
- **Click detection lives in `createViewer`**, shared with Build a Car; changing it changes both tabs.

## New words

[Raycasting](GLOSSARY.md#raycaster-and-raycasting) · [NDC](GLOSSARY.md#ndc) ·
[Emissive](GLOSSARY.md#emissive) · [Depth buffer](GLOSSARY.md#depth-buffer-and-depthwrite) ·
[Event and listener](GLOSSARY.md#event-and-listener) · [userData](GLOSSARY.md#userdata) ·
[Callback](GLOSSARY.md#callback)
