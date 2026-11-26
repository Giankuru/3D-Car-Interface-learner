# 04. Weld course quizzes and the MIG calculator

**Time:** 50 min · **Difficulty:** 2 of 5 · Previous: [03](03-content-as-data.md) · Next: [05 SVG](05-svg-schematics.md)

## Goal

Read the second shape of data in the app (modules with lessons and quizzes), find the one hidden rule the
quiz depends on, and extend a tiny calculator.

## Read these lines

- One module, `settings`, lines 1179-1198 (the calculator's HTML is lines 1181-1185).
- `initWeld`, lines 1227-1276, especially the quiz logic (lines 1244-1259) and the calculator (lines 1262-1271).

## The ideas

### The shape of a module

```js
{ id: "settings", title: "Settings & reading the bead",
  lessons: [ [heading, htmlString], [heading, htmlString], ... ],
  quiz:    [ [question, [4 options], correctIndex, why], ... ] }
```

One quiz entry, slot by slot (line 1195):

```js
["Rule-of-thumb amperage for 1/8 in mild steel?",            // [0] question
 ["About 35 A", "About 125 A", "About 300 A", "About 12 A"], // [1] exactly 4 options
 1,                                                          // [2] index of the right one (0-based)
 "Roughly 1 A per thousandth of an inch: 0.125 in → ~125 A."] // [3] explanation shown after
```

A lesson is `[heading, html]`, and line 1240 unpacks it with destructuring:

```js
m.lessons.map(([h, body]) => `<h3>${esc(h)}</h3>${body}`).join("")
```

`body` is *not* passed through `esc`: lesson text is trusted HTML written by the author, so it may
contain tags.

### How the quiz is marked

When you click an option (lines 1247-1251), every option of that question is disabled, the right one
gets the class `right`, and if you were right the question number goes into the `Set` called `right`.

Then line 1252 counts how many questions are answered:

```js
const answered = doc.querySelectorAll(".q .opt:disabled").length / 4;
```

**The hidden rule:** it counts *disabled buttons* and divides by 4. That only works if **every question
has exactly 4 options.** (All 21 questions in the file do.) When all are answered and all are right,
line 1255 saves the pass:

```js
if (right.size === m.quiz.length) { passed[id] = true; store.set("tg-weld", passed); ...
```

Passes live in localStorage under `tg-weld`; the open module under `tg-weld-sel`.

### The calculator

The calculator's controls are HTML **inside a lesson string** (lines 1182-1185):

```html
<label for="cJoint">Joint<select id="cJoint"><option value="1">Butt / lap</option><option value="1.1">T-joint (fillet)</option></select></label>
```

So `#cThk` exists only while the Settings module is on screen. That is why the code first checks
for it (line 1263, `if (thk) {`) before wiring it up.

The rule, in words (lines 1265-1267):

- **amps ≈ thickness in inches × 1000 × joint factor** (the factor is the `value` of the Joint option);
- wire size from thickness: up to 0.048 in → 0.023 in wire, up to 0.1875 in → 0.030 in, thicker → 0.035 in;
- a note for very thin or very thick metal.

```js
const t = +thk.value, j = +$("#cJoint").value, amps = Math.round(t * 1000 * j);
```

It is a shop rule of thumb, not physics. `onchange` on both dropdowns re-runs `calc` (line 1270).

## Exercise

**(a)** Add a 4th question to any module's `quiz` (4 options!). For example, at the end of the
`settings` quiz (after line 1197, and add a comma after that line's `]`):

```js
["Which gas mix does the calculator suggest?", ["100% argon", "75/25 argon/CO2", "Pure oxygen", "Compressed air"], 1, "75/25 is the usual MIG mix for mild steel."]
```

**(b)** Add a third joint type to `#cJoint` (line 1184), inside the backtick string:

```html
<option value="0.9">Outside corner</option>
```

Reload, open Weld & Fab > module 5.

**You should now see:** (a) the quiz says "Answer all 4 correctly to pass", and passing needs all 4.
(b) with 14 ga selected: Butt / lap → **75 A**, T-joint → **83 A**, Outside corner → **68 A**.

<details><summary>Answer: where the numbers come from</summary>

- 0.075 × 1000 × 1 = 75
- 0.075 × 1000 × 1.1 = 82.5 → `Math.round` → 83
- 0.075 × 1000 × 0.9 = 67.5 → 68

The new option needs no code change: `calc` reads whatever `value` the selected option has.

</details>

## Traps

- **3 or 5 options in a question:** "disabled buttons ÷ 4" is then no longer the number of answered
  questions (3 options add 0.75, 5 add 1.25), so the total usually never equals the question count and
  the quiz never finishes.
- **`correctIndex` is 0-based:** the second option is `1`.
- **Option pasted outside the backtick string:** the whole script breaks (Console: "Unexpected token").
- **Reset quiz passes:** delete `tg-weld` in DevTools > Application > Local Storage. "Retry quiz" only
  resets the current attempt; a pass stays saved.

## New words

[Spread and destructuring](GLOSSARY.md#spread-and-destructuring) · [Set](GLOSSARY.md#set) ·
[localStorage](GLOSSARY.md#localstorage) · [Ternary](GLOSSARY.md#ternary)
