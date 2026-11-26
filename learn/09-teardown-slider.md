# 09. The teardown slider

**Time:** 60 min · **Difficulty:** 3 of 5 · Previous: [08](08-models-from-shapes.md) · Next: [10 Clicking a part](10-clicking-parts.md)

## Goal

Follow the chain from the slider to each part's position:
slider → `ENG.target` → eased `ENG.t` → `explodeProgress` → the part's group moves.

## Read these lines

- `explodeProgress` and its comment, lines 394-402.
- `ENG`, line 404 (note `maxStage: 6`).
- The `stage` and `dir` arguments of the `part(...)` calls, lines 418-520 (just the first line of each).
- The slider and its buttons, lines 563-566.
- The engine's `v.tick`, lines 570-580.

## The ideas

### The chain

```
 you drag the slider (0-100)
      |  slider.oninput: ENG.target = value / 100              (line 564)
      v
 ENG.target  (0..1, where you WANT to be)
      |  every frame: ENG.t += (ENG.target - ENG.t) * 0.06     (line 571)   smoothing #1: in time
      v
 ENG.t       (0..1, where the animation IS)
      |  for each part: explodeProgress(part, ENG.t, 6)        (line 575)   smoothing #2: per part
      v
 0..1 for this part  x  part.dir  =  group.position
```

There are **two different smoothings**, and they are easy to mix up:

1. **The chase (in time).** Each frame `ENG.t` moves 6% of the remaining way to `ENG.target`. Big gap =
   big step, small gap = small step, so the motion slows as it arrives. The "Assemble" and "Tear down"
   buttons (lines 565-566) only set `ENG.target` to 0 or 1; the chase does the animation.
2. **Smoothstep (in shape).** For a given `ENG.t`, `explodeProgress` decides how far out *this* part is,
   with its own soft start and stop.

### Stages and directions

Each `part(id, stage, dir, build)` call gives:

- **`stage`**: removal order, 0 = removed first. Coils are stage 0; the cover, plugs, intake and other
  bolt-ons are 1; the crankshaft is 6, last out.
- **`dir`**: a full displacement in units, not just a direction. Fully pulled out, the part sits at
  `dir`. The plugs (line 471) end up 7.8 units up and 1.2 toward you:

```js
part("plugs", 1, [0, 7.8, 1.2], g => X.forEach(x => {
```

The block has `dir` `[0, 0, 0]` (line 418), so it never moves.

### `explodeProgress` (lines 397-402)

```js
function explodeProgress(part, t, maxStage) {
  const share = 0.45;                                   // slice of the slider each stage uses
  const start = (part.stage / maxStage) * (1 - share);  // later stages start later
  const p = Math.min(1, Math.max(0, (t - start) / share));
  return p * p * (3 - 2 * p);                           // smoothstep easing
}
```

Each stage gets a window 0.45 wide on the 0-1 slider, starting at `stage / 6 × 0.55`. Windows overlap, so
the next stage starts before the previous one is done:

```
slider t:  0         0.25      0.5       0.75      1
           |---------|---------|---------|---------|
stage 0    ##################                        [0.000, 0.450]
stage 1        ##################                    [0.092, 0.542]
stage 2           ##################                 [0.183, 0.633]
stage 3               ##################             [0.275, 0.725]
stage 4                   ##################         [0.367, 0.817]
stage 5                      ##################      [0.458, 0.908]
stage 6                          ##################  [0.550, 1.000]
```

Inside its window, `p` goes straight from 0 to 1 (`Math.min`/`Math.max` clamp it outside). Then
**smoothstep** `p × p × (3 − 2p)` bends that straight line into an S: the part starts gently and lands
gently (the slope is 0 at both ends).

```
out
1.0 |                                   ******
0.8 |                           ****
0.6 |                      ***
0.4 |                ***
0.2 |          ****
0.0 |******
    +-----------------------------------------> p
    0                  0.5                    1
```

### Worked example: the plugs at t = 0.3

- start = 1 / 6 × 0.55 = 0.0917
- p = (0.3 − 0.0917) / 0.45 = 0.463
- smoothstep = 0.463² × (3 − 2 × 0.463) = 0.445
- position = `[0, 7.8, 1.2]` × 0.445 ≈ `[0, 3.47, 0.53]`

That last step is line 575:

```js
ENG.parts.forEach(p => p.group.position.copy(p.dir).multiplyScalar(explodeProgress(p, ENG.t, ENG.maxStage)));
```

### Don't fight the user's finger (line 573)

```js
if (document.activeElement !== slider) slider.value = Math.round(ENG.t * 100);
```

The tick moves the slider knob to follow the animation (for the buttons), except while you are holding
the slider. `document.activeElement` is the element that has focus.

## Exercise

**(a)** In the Console (on the Engine tab):

```js
explodeProgress({stage: 1}, 0.3, 6)
explodeProgress({stage: 6}, 0.5, 6)
explodeProgress({stage: 0}, 0.5, 6)
```

**(b)** Edit `share` (line 398): first `1.0`, reload, press Tear down; then `0.1`, reload, press Tear down.

**(c)** Put `share` back to `0.45`, and change the chase factor `0.06` (line 571) to `0.5`.

**You should now see:** (a) `0.4445…` (the plugs example), `0` (the crank hasn't started at half way)
and `1` (the coils are already fully out). (b) with `1.0` every part moves together; with `0.1` the parts
come off strictly one stage after another. (c) "Tear down" becomes almost instant. Undo (c).

<details><summary>Answer (b): why</summary>

With `share = 1`, `start = stage / 6 × 0 = 0` for every stage, so all windows are the full slider and
every part gets the same progress. With `share = 0.1`, the windows are 0.1 wide and start at
`stage / 6 × 0.9` (0, 0.15, 0.3, …, 0.9): they no longer overlap, so each stage finishes before the next
begins.

</details>

## Traps

- **A part with a stage above 6:** raise `ENG.maxStage` (line 404) too. A stage-7 part starts at 0.64,
  so at t = 1 its `p` is only 0.80 and it never comes fully out.
- **`dir` is not a unit vector:** `[0, 1, 0]` moves the part only 1 unit.
- **Removing the `activeElement` check** makes the knob jump back under your finger while the chase is
  still running.

## New words

[Easing and smoothstep](GLOSSARY.md#easing-smoothstep-and-ease-out) · [lerp](GLOSSARY.md#lerp) ·
[Vector3](GLOSSARY.md#vector3) · [Render loop](GLOSSARY.md#render-loop)
