# 01. JavaScript for C++ programmers

**Time:** 75 min · **Difficulty:** 2 of 5 · Previous: [00](00-start-here.md) · Next: [02 Routing](02-routing.md)

## Goal

Read every JavaScript idiom `index.html` uses. Each one is shown as "what you know from C++", then a
real line from the file.

## Read these lines

Lines 272-281 (helpers), 346-353 (one-line functions), 404 (`ENG`), 412-417 (`part()`), 560-561 (the
part chips), 970 (unique categories).

Keep the Console open (F12) and try things as you go. Typing there is the fastest way to learn.

## The ideas

### 1. Variables: `const` and `let`

Both are block-scoped like C++ locals. `const` means the *name* can't be pointed at something else;
the object it names can still change (like a `T* const` pointer).

```js
const ENG = { parts: [], t: 0, ... };   // line 404: ENG can't be reassigned...
ENG.t = 0.5;                             // ...but its fields can change
let down = null;                         // line 326: let = may be reassigned
```

There are no types in declarations. A variable holds whatever you put in it.

### 2. Objects are structs (and dictionaries)

`{ key: value, ... }` builds an object on the spot, like a struct with no declared type.
Line 404:

```js
const ENG = { parts: [], t: 0, target: 0, sel: null, isolate: false, maxStage: 6, focus: null };
```

`ENG.parts` is an **array**: a growable list like `std::vector`, with `.push(x)` and `.length`.
Objects also work as dictionaries (`std::map<string, ...>`): `ENGINE_INFO["crank"]` and
`ENGINE_INFO.crank` mean the same thing.

### 3. Three ways to write a function

```js
function isShown(o) { ... }                // line 343: classic
const box = (x, y, z) => new THREE.BoxGeometry(x, y, z);   // line 350: arrow, returns the value
v.pick = ev => { ... return ...; };        // line 318: arrow with a body { }, needs return
```

`x => x * 2` is a whole function: parameter `x`, returns `x * 2`. One parameter needs no brackets.
**Default parameters** work like C++: `const M = (color, o = {}) => ...` (line 346).

### 4. Functions are values

You can store a function in a field, like a member function added at run time. `v.applyTheme`
(lines 305-310):

```js
v.applyTheme = () => { scene.background = new THREE.Color(cssVar("--stage")); ... };
```

You can also pass a function as an argument: a **callback**. `part(id, stage, dir, build)`
(lines 412-413) receives `build` and calls it:

```js
function part(id, stage, dir, build) {
  const g = new THREE.Group(); build(g); v.scene.add(g);
```

…and the caller passes a function written on the spot:

```js
part("oilpan", 2, [0, -3.2, 0], g => {
  add(g, box(3.8, 0.7, 2.0), iron(), [0, -1.55, 0]);
```

(`part` and `add` are explained in lesson 08; here, just see that the 4th argument is a function.)

### 5. Closures

An inner function remembers the variables around it, even after the outer function returned. Inside
`createViewer` (lines 317-318), `v.pick` uses `renderer`, `ray` and `ndc`, which are locals of
`createViewer`:

```js
const ray = new THREE.Raycaster(), ndc = new THREE.Vector2();
v.pick = ev => {
```

When `v.pick` runs much later (on a mouse move), those locals are still alive. Think of them as
hidden member variables of the function.

### 6. Template strings build HTML

Backtick strings can hold `${expression}` holes. This is how the app writes HTML (line 538):

```js
el.innerHTML = `<span class="sys">${esc(info.sys)}</span><h3 style="margin-top:6px">${esc(info.name)}</h3>
```

A hole can hold a choice: `${cond ? "some html" : ""}` adds a piece of HTML only when `cond` is true
(line 992: ``${g.parts ? `<div class="box">...` : ""}``).

### 7. Array methods: `map(...).join("")` makes HTML

| Method | Does | C++ idea |
|---|---|---|
| `forEach(f)` | call `f` on each item, return nothing | range-for loop |
| `map(f)` | new array of `f(item)` | `std::transform` |
| `filter(f)` | new array of items where `f` is true | `std::copy_if` |
| `find(f)` / `findIndex(f)` | first item (or index) where `f` is true | `std::find_if` |
| `some(f)` | is `f` true for any item? | `std::any_of` |
| `join(s)` | glue an array of strings with `s` between | - |

The pattern you will see everywhere: turn each item into an HTML string, then glue them. Line 561:

```js
$("#partChips").innerHTML = order.map(id => `<button class="chip" data-id="${id}">${esc(ENGINE_INFO[id].name)}</button>`).join("");
```

The callback can also take the index: `X.forEach((x, i) => ...)` (line 430).

### 8. Spread and destructuring

`...` unpacks an array into separate arguments. `opt.cam` is `[10, 7, 12]`, so line 294:

```js
camera.position.set(...opt.cam);   // same as camera.position.set(10, 7, 12)
```

Destructuring names the parts of an array: `([h, body]) => ...` (line 1240) takes a 2-item array and
calls its items `h` and `body`. And `[...new Set(list)]` (line 970) is the "unique values" trick: a
`Set` drops duplicates, `[... ]` turns it back into an array.

```js
const cats = [...new Set(GUIDES.map(g => g.cat))];
```

### 9. Small surprises

- **Ternaries chain** (line 1266): `a ? b : c ? d : e` means "if a then b, else if c then d, else e".
- **Unary plus** turns a string into a number: `+thk.value` (line 1265). Form values are always
  strings.
- **`"use strict"`** (line 272) turns some silent mistakes, like assigning to a misspelt variable, into
  errors.

### 10. The helpers at the top (lines 273-281)

```js
const $ = (s, el = document) => el.querySelector(s);
```

`$` (line 273) is a one-line shortcut: `$("#tip")` finds the first element matching the CSS selector
`#tip`. It is **not** the jQuery library, which uses the same symbol.

`store` (lines 275-278) wraps localStorage (saving in the browser, lesson 03) in `try`/`catch`,
because storage can be blocked, for example in some private windows:

```js
get(k, d) { try { const v = localStorage.getItem(k); return v ? JSON.parse(v) : d; } catch (e) { return d; } },
```

`esc` (line 279) replaces `<`, `>`, `&` and `"` with safe codes, so text from the data can never be
read as HTML. Every `${...}` that puts text into HTML goes through `esc`.

## Exercise: one-liners in the Console

Open the app, open the Console, and run:

```js
GUIDES.map(g => g.title).join(" | ")
GUIDES.filter(g => g.diff === 3).map(g => g.title)
[...new Set(GUIDES.map(g => g.cat))]
esc("<b>hi</b>")
+"0.075" * 1000
```

**You should now see:**

1. one long string with all 14 job titles separated by `|`;
2. `["Replace front brake pads"]` (the only difficulty-3 job);
3. 6 categories, starting `"Wheels & tires", "Electrical", "Engine"`;
4. `"&lt;b&gt;hi&lt;/b&gt;"`;
5. `75`.

Now write your own one-liner: list the `id` of every guide that has a `parts` field. You should get
10 ids.

<details><summary>Answer</summary>

```js
GUIDES.filter(g => g.parts).map(g => g.id)
// ["oil", "air-filter", "cabin-filter", "wipers", "bulbs", "fluids", "plugs", "brake-pads", "belt", "fuses"]
```

A missing field reads as `undefined`, which counts as false in a test, so `filter(g => g.parts)` keeps
only the guides that have one.

</details>

## Traps

- `=>` (arrow) is not `>=` (greater or equal).
- After `=>`, a `{` starts a function body, not an object. To return an object, wrap it:
  `x => ({ id: x })`.
- `map` returns a new array; `forEach` returns nothing. The file uses `forEach` when it only wants side
  effects (most of the 3D code).
- `===` is "equal", the one to use. (`==` also exists and converts types first; avoid it.)

## New words

[Arrow function](GLOSSARY.md#arrow-function) · [Callback](GLOSSARY.md#callback) ·
[Closure](GLOSSARY.md#closure) · [Template literal](GLOSSARY.md#template-literal) ·
[Spread and destructuring](GLOSSARY.md#spread-and-destructuring) · [Ternary](GLOSSARY.md#ternary) ·
[querySelector and $](GLOSSARY.md#queryselector-and-) · [innerHTML](GLOSSARY.md#innerhtml) ·
[JSON](GLOSSARY.md#json) · [localStorage](GLOSSARY.md#localstorage)
