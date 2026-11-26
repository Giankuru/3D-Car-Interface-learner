# 11. Capstone: Build a Car

**Time:** 80 min · **Difficulty:** 3 of 5 · Previous: [10](10-clicking-parts.md) · [All lessons](README.md)

## Goal

Read the Build a Car tab using everything from lessons 06-10, learn one new idea (animation that runs
at the same speed on any screen, using `dt`), add a step of your own, and fix a real bug.

## Read these lines

- The size constants, line 587.
- Two steps only: "Ladder frame" (lines 589-598) and "Glass & lights" (lines 785-801). Skim the rest
  of `STEPS`.
- The body shape, lines 773-783.
- `CAR`, line 804; `initBuild`, lines 805-848; `stopPlay`, line 849; `goStep`, lines 850-864;
  `renderStep`, lines 865-879.

## The ideas

### A step = words + a build function

```js
{ title: "Ladder frame", sys: "Structure",
  what: "...", attach: "...", tools: "...", tip: "...",
  build(g) {
    const m = M(0x2b2f33, { metalness: 0.5, roughness: 0.5 });
    [-1, 1].forEach(s => add(g, box(4.0, 0.14, 0.08), m, [0, 0.48, 0.45 * s]).userData.label = "Frame rail");
```

`build(g) { ... }` is short for `build: function (g) { ... }`. Each step fills one group, like `part()`
in lesson 08. Labels are set on each mesh (`add` returns the mesh). Sharing one material `m` inside a
step is fine here, because the whole step glows together.

### All groups exist from the start (lines 807-811)

```js
STEPS.forEach((st, i) => {
  const g = new THREE.Group(); st.build(g); g.visible = false;
  g.traverse(o => { if (o.isMesh) { o.userData.step = i; if (!o.userData.label) o.userData.label = st.title; } });
  v.scene.add(g); v.pickables.push(g); CAR.groups.push(g);
});
```

All 14 groups are built once, hidden. A mesh without its own label gets the step's title. `CAR` (line 804)
holds the state: `cur` is the current step, a cursor into the list (−1 = empty garage).

### `goStep(n)`: show groups 0..n (lines 850-864)

```js
CAR.groups.forEach((g, i) => {
  const show = i <= n;
  if (show && !g.visible) { g.userData.anim = 0; g.position.y = 1.8; }
  g.visible = show;
});
```

A group that is about to appear starts 1.8 units up with `anim = 0`; the tick drops it into place.
Then `goStep` marks the step list and calls `renderStep(n)` to fill the panel (same template-string
pattern as lesson 03).

### Frame-rate-independent animation with `dt` (lines 831-841)

```js
let last = performance.now();
v.tick = now => {
  const dt = Math.min(0.05, (now - last) / 1000); last = now;
  ...
  g.userData.anim = Math.min(1, g.userData.anim + dt * 1.4);
  const e = 1 - Math.pow(1 - g.userData.anim, 3);
  g.position.y = 1.8 * (1 - e);
```

**`dt`** (delta time) is the seconds since the previous frame. Adding `dt × 1.4` per frame means `anim`
goes 0 → 1 in about 0.7 seconds **whatever the frame rate**: on a 144 Hz screen `dt` is small, on a 30 Hz
screen it is bigger. (Compare the engine's `ENG.t += … * 0.06` in lesson 09: that runs faster on a faster
screen.) `dt` is capped at 0.05 s so that coming back to a tab after a long time doesn't make things jump.

The **ease-out cubic** `1 − (1 − a)³` starts fast and lands softly, unlike smoothstep, which also starts
slowly:

```
 e
1.0 |                      ###################      # ease-out cubic  1 - (1 - a)^3
0.8 |              ########    .......              . smoothstep (lesson 09)
0.6 |         #####       .....
0.4 |     ####      ......
0.2 |  ###   .......
0.0 |##......
    +------------------------------------------> a (0 to 1)
```

### "Build all": a timer next to the loop (lines 817-822, 849)

```js
CAR.playing = setInterval(() => { if (CAR.cur >= STEPS.length - 1) stopPlay(); else goStep(CAR.cur + 1); }, 1600);
```

`setInterval(f, 1600)` calls `f` every 1.6 seconds, independent of the render loop. It returns an id;
`stopPlay` (line 849) passes it to `clearInterval` to stop. Every other button calls `stopPlay()` first.

### The glow pulse (lines 842-844)

`goStep` sets `CAR.pulse = 1`; the tick lowers it by `dt × 0.5` each frame (2 seconds to fade), and the
current step's meshes glow in the accent colour with strength `0.5 × pulse`.

### The body: a 2D outline pushed into 3D (lines 773-783)

```js
const s = new THREE.Shape();
s.moveTo(-2.2, 0.3); s.lineTo(-WB - 0.42, 0.3);
s.absarc(-WB, 0.34, 0.42, Math.PI, 0, true);   // wheel arch: half circle around the rear wheel
...
const geo = new THREE.ExtrudeGeometry(s, { depth: 1.7, bevelEnabled: true, ... });
geo.translate(0, 0, -0.85);
```

A `Shape` is a 2D outline drawn with pen commands, like an SVG path (lesson 05), but in the x-y plane
with **y up**. Draw it on graph paper: it's the car's side view with two wheel arches. **Extrude** pushes
it 1.7 units along z to make a solid; `translate` centres it. The material is see-through
(`opacity: 0.28`, `depthWrite: false`), the same trick as isolation in lesson 10; "Body: solid" flips it
(lines 823-828).

### Clicking in this tab (line 830)

Same picking code as the engine (lesson 10). Here `v.onClick` shows the clicked mesh's step with its
label. The step list uses event delegation with `closest(".step")` (line 813).

## Exercise

**(a) A 15th step.** After the "Glass & lights" step (its last line is `} }` on line 801: add a comma),
add:

```js
{ title: "Roof rack", sys: "Body",
  what: "Two bars on the roof for carrying loads.", attach: "Clamps to the roof rails.",
  tools: "Hex key", tip: "Check the roof's load rating before loading it.",
  build(g) {
    const m = M(0x222222, { metalness: 0.4 });
    [-1, 1].forEach(s => add(g, box(1.4, 0.04, 0.04), m, [-0.2, 1.47, 0.6 * s]));
  } }
```

**(b) Debugging: the lights never glow.** Press "Add next" until "Glass & lights" (step 14) and
watch the headlights. The step's code gives them a warm-white glow (`emissive: 0xfff2c0`, line 797) and
the taillights a red one (line 798). Do they keep it? Find out why not, and fix it. Hint: read line 844,
then search the whole file for the word it checks.

**You should now see:** (a) 15 entries in the Assembly order list, "Step 15 of 15 · Body" in the panel,
two bars dropping onto the roof, and hovering them shows "Roof rack" (they have no label of their own, so
they get the step title). (b) before the fix, the lights flash the accent colour with the rest of the
step and then go dark. After your fix, the headlights stay warm white and the taillights stay red while
the rest of the step pulses.

<details><summary>Answer (b): the bug and the fix</summary>

Line 844 makes the current step's meshes pulse, **except** meshes whose material has
`userData.glow` set:

```js
if (g) g.traverse(o => { if (o.isMesh && !o.material.userData.glow) { o.material.emissive.copy(accent); o.material.emissiveIntensity = 0.5 * CAR.pulse; } });
```

Nothing in the file ever sets `userData.glow`, so the lights' own colour is overwritten with the accent,
and their strength fades to 0 with the pulse. (`goStep` on line 854 skips anything labelled `/light/i`
when clearing the glow, so the author clearly meant lights to keep theirs.)

Fix: in "Glass & lights" (lines 797-798), make the two materials first and mark them:

```js
const head = M(0xfff6d8, { emissive: 0xfff2c0, emissiveIntensity: 0.8 });
const tail = M(0xb01818, { emissive: 0xff2010, emissiveIntensity: 0.6 });
head.userData.glow = true; tail.userData.glow = true;
add(g, box(0.05, 0.08, 0.32), head, [2.24, 0.7, 0.58 * s]).userData.label = "Headlight";
add(g, box(0.05, 0.08, 0.3), tail, [-2.27, 0.76, 0.6 * s]).userData.label = "Taillight";
```

Materials have their own `userData` object, separate from the mesh's. These lines go inside the
`[-1, 1].forEach(s => { ... })`, replacing the two original lines.

</details>

## Traps

- **A missing field** (say `tip`) shows "undefined" in the panel.
- **The rack hides inside the see-through body** if you place it too low: toggle "Body: solid" to check.
- **`build(g)` must use `add(g, ...)`,** not `v.scene.add`. Otherwise the part is not in the step's group:
  it is visible from the start and doesn't drop in.
- **Mesh `userData` vs material `userData`:** `o.userData.label` is on the mesh,
  `o.material.userData.glow` is on the material.

## Where next

You have now read every part of the app that has logic in it. Ideas to keep going: give the engine an
"exploded view" camera that zooms out as the slider moves, add a quiz to Build a Car, or turn the
engine's `ENG.t` chase into a `dt`-based one like this tab's.

## New words

[dt](GLOSSARY.md#dt-delta-time) · [Easing and ease-out](GLOSSARY.md#easing-smoothstep-and-ease-out) ·
[setInterval](GLOSSARY.md#setinterval) · [Extrude](GLOSSARY.md#extrude) ·
[Event delegation](GLOSSARY.md#event-delegation) · [Emissive](GLOSSARY.md#emissive) ·
[userData](GLOSSARY.md#userdata)
