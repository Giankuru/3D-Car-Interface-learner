# 03. Content as data: maintenance guides

**Time:** 60 min · **Difficulty:** 2 of 5 · Previous: [02](02-routing.md) · Next: [04 Quizzes and calculator](04-quiz-and-calculator.md)

## Goal

See the main pattern of this app: **the words live in lists, and one function turns a list item into a
page.** Then add your own maintenance job.

## Read these lines

- One guide object, `GUIDES[0]`, lines 885-889.
- `initMaintain`, lines 967-1014.
- Skim `initElectrical`, lines 1019-1077: same idea, different shape of data.

## The ideas

### Data in, page out

```
GUIDES[]  (lines 884-965: 14 plain objects, just words)
   |
   +--> list(q)  -> sidebar buttons, grouped by category   -> $("#mList").innerHTML
   |
   +--> show(id) -> one job's page: tools, safety, steps   -> doc.innerHTML
```

Adding a job means adding an object to `GUIDES`. No HTML to write, no function to change.

### The shape of one guide

Lines 885-889, shortened:

```js
{ id: "flat-tire", cat: "Wheels & tires", title: "Change a flat tire", diff: 1, time: "20–30 min",
  tools: ["Spare tire", "Jack (in the trunk kit)", ...],
  safety: ["Get well off the road onto flat, firm ground. Hazards on.", ...],
  steps: ["Park, set the parking brake, and put it in Park (or 1st gear on a manual).", ...],
  tips: ["Compact 'donut' spares are usually limited to about 50 mph (80 km/h) and roughly 70 miles. ...", ...] },
```

| Field | Meaning |
|---|---|
| `id` | unique name, used in saved progress |
| `cat` | sidebar group; groups are made automatically from these (line 970) |
| `title`, `time` | shown as text |
| `diff` | difficulty 1-3, drawn as bars |
| `tools`, `safety`, `steps`, `tips` | arrays of strings, each becomes a list |
| `parts` | optional array of strings |

### `list(q)`: the sidebar and search

For each category, keep the guides in that category whose title or steps contain the search text
(line 977):

```js
const items = GUIDES.filter(g => g.cat === c && (g.title + " " + g.steps.join(" ")).toLowerCase().includes(q));
```

### `show(id)`: one page from one object

Line 984 finds the object, and lines 987-999 write the whole page as one template string. An optional
field gets an optional box (line 992):

```js
${g.parts ? `<div class="box"><h4>Parts</h4><ul>${g.parts.map(t => `<li>${esc(t)}</li>`).join("")}</ul></div>` : ""}
```

### Saving checkmarks: localStorage

**localStorage** is a small key/value store the browser keeps for each site. It survives reloads, but
it can hold only strings, so `store` (lesson 01) converts with **JSON** (`JSON.stringify` /
`JSON.parse`).

The ticked steps are one object, `done`: guide id -> list of ticked step numbers (line 969):

```js
const done = store.get("tg-maint", {});        // e.g. { "flat-tire": [0, 1, 4] }
```

When you tick a box (lines 1000-1003), the step number is added and the object saved:

```js
const i = +cb.dataset.i, arr = new Set(done[id] || []);
cb.checked ? arr.add(i) : arr.delete(i);
done[id] = [...arr]; store.set("tg-maint", done);
```

(`data-i="2"` in HTML becomes `cb.dataset.i` in JS, as the string `"2"`; the `+` makes it a number.)
The app also remembers the last job you opened, under `tg-maint-sel`.

### One listener for many buttons: event delegation

Instead of a click handler on every sidebar button, there is one on the whole sidebar (line 1011):

```js
side.addEventListener("click", e => { const b = e.target.closest(".item"); if (b) { show(b.dataset.id); ...
```

`e.target` is what was actually clicked (maybe a `<span>` inside the button). `closest(".item")` walks
up to the button. This keeps working when `list()` replaces the buttons with new ones.

### The electrical tab: same idea, arrays instead of objects

Skim `initElectrical` (lines 1019-1077). Its data are arrays of arrays, read by position. Line 1021:

```js
["Battery", "Stores energy as chemical potential. ...", "Slow cranking, clicking, or a reading under 12.4 V at rest"],
//  [0] name   [1] what it does                          [2] symptoms when failing
```

`comps` (line 1020) and `sensors` (line 1030) rows are `[name, what it does or measures, symptoms]`;
`tests` (line 1043) rows are `[test, how, good reading]`. Shorter than objects, but you must remember
which slot is which. Objects (like `GUIDES`) name their fields; that is the trade-off.

## Exercise: add a 15th job

Add this object at the end of `GUIDES`, after the `fuses` guide (line 964). Mind the comma after
the previous `}`.

```js
{ id: "horn", cat: "Electrical", title: "Test the horn", diff: 1, time: "5 min",
  tools: ["Test light"],
  safety: ["Keep fingers clear of the radiator fan."],
  steps: ["Press the horn button.", "Find the horn fuse and check it.", "Probe the horn connector with the test light."],
  tips: ["A clicking relay but no sound usually means a dead horn."] }
```

Reload, open Maintenance, then tick the first two steps of your job and reload again.

**You should now see:** "Test the horn" in the Electrical group; typing "horn" in the search finds it;
ticking two of three steps shows **Progress 67%**; after reload the ticks are still there; and DevTools >
Application > Local Storage shows `tg-maint` containing `"horn":[0,1]`.

<details><summary>Answer: what the end of GUIDES looks like</summary>

```js
    tips: ["If the new fuse blows immediately, there's a short circuit. ...", "Many cars include spare fuses in the box lid."] },
  { id: "horn", cat: "Electrical", title: "Test the horn", diff: 1, time: "5 min",
    tools: ["Test light"],
    safety: ["Keep fingers clear of the radiator fan."],
    steps: ["Press the horn button.", "Find the horn fuse and check it.", "Probe the horn connector with the test light."],
    tips: ["A clicking relay but no sound usually means a dead horn."] }
];
```

67% is `Math.round(2 / 3 * 100)` (line 1005).

</details>

## Traps

- **Missing comma between objects:** the Console shows "Unexpected token" and nothing on the page works.
- **Duplicate `id`:** two jobs share the same checkmarks.
- **`diff: 4`:** all three bars fill and the label reads "undefined" (only 1-3 have names, line 988).
- **Leaving out `tips`** (or `steps`, `tools`, `safety`) crashes the page (`g.tips.map` of undefined).
  Leaving out `parts` is fine, because line 992 checks for it.
- **Reset your progress:** delete the `tg-maint` key in DevTools > Application > Local Storage, or press
  "Clear checkmarks".

## New words

[localStorage](GLOSSARY.md#localstorage) · [JSON](GLOSSARY.md#json) ·
[Event delegation](GLOSSARY.md#event-delegation) · [innerHTML](GLOSSARY.md#innerhtml) ·
[DOM](GLOSSARY.md#dom) · [Set](GLOSSARY.md#set)
