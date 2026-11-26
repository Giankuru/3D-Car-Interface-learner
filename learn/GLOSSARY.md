# Glossary

Every new term in the guide, in alphabetical order. The number in brackets is the lesson where it first
matters. [Back to the lesson list](README.md)

### aria-current
An HTML attribute that marks the current item, here the active tab (`aria-current="page"`). Screen
readers announce it and the CSS colours it. (02)

### Arrow function
Short function syntax: `x => x * 2`, or `(a, b) => { ...; return ...; }` with a body. (01)

### Callback
A function passed to another function to be called later, like `build` in `part(id, stage, dir, build)`
or the `v.onClick` slot. (01)

### CDN
Content delivery network: a public server that hosts libraries. The page loads Three.js from one with a
`<script src="...">` tag, which is why the 3D tabs need internet. (00)

### Closure
A function that keeps access to the variables around it after the outer function has returned, like
`v.pick` using `createViewer`'s `ray` and `renderer`. (01)

### CSS variable
A named value in CSS, also called a custom property: defined as `--accent: ...`, used as
`var(--accent)`, read from JavaScript with `cssVar("--accent")`. (05)

### Damping
Motion that slows down gradually after the input stops, instead of stopping dead. (07)

### Depth buffer and depthWrite
The depth buffer is the renderer's per-pixel memory of how near the closest thing drawn there is;
anything further away is skipped. `depthWrite = false` stops a mesh leaving that mark, so a faded part
doesn't hide what is behind it. (10)

### DevTools
The browser's built-in developer tools (F12): Console, Elements, Application and more. (00)

### DOM
Document Object Model: the live tree of HTML elements that JavaScript reads and changes. (03)

### dt (delta time)
Seconds since the previous frame. Moving things by `speed × dt` makes animation run at the same speed
at any frame rate. (11)

### Easing, smoothstep and ease-out
Easing reshapes a straight 0-1 progress into a curve so motion starts or stops gently. Smoothstep
`p × p × (3 − 2p)` is gentle at both ends (09); ease-out cubic `1 − (1 − a)³` starts fast and lands
gently (11).

### Emissive
Light a surface gives off by itself, visible even in shadow. Used to highlight parts and for the car's
lights. (10)

### Event and listener
An event is something that happened (a click, a key, a pointer move, a hash change). A listener is the
function registered to react, with `addEventListener("click", fn)` or `el.onclick = fn`. (02)

### Event delegation
One listener on a parent element that works out which child was clicked with
`e.target.closest(".item")`, instead of one listener per child. (03)

### Extrude
Turn a 2D outline into a 3D solid by pushing it along a depth (`ExtrudeGeometry`). (11)

### Frustum
The pyramid-with-its-tip-cut-off volume a perspective camera can see, between the near and far
planes. (06)

### Geometry
The shape of a mesh (its corner points), e.g. `BoxGeometry`, `CylinderGeometry`. (08)

### Group
An invisible node in the scene tree. Moving, rotating or hiding a group does the same to all its
children. (08)

### hidden attribute
The HTML `hidden` attribute hides an element. In JavaScript, `el.hidden = true` or `false`. (02)

### HTML, CSS, JS
The three languages of a web page: HTML is the structure, CSS the appearance, JavaScript the
behaviour. (00)

### innerHTML
An element property: assigning an HTML string to it replaces the element's contents with that HTML. (01)

### JSON
A text format for objects and arrays. `JSON.stringify(obj)` turns an object into text,
`JSON.parse(text)` turns it back. (03)

### Lazy initialisation
Building something only the first time it is needed, here each tab on its first visit. (02)

### lerp
Linear interpolation: move a fraction of the way from a to b. `a.lerp(b, 0.1)` moves `a` 10% closer to
`b`; repeated every frame it gives a smooth glide. (07)

### localStorage
A small key/value store of strings that the browser keeps per site, which survives reloads. This app's
keys: `tg-maint`, `tg-maint-sel`, `tg-weld`, `tg-weld-sel`. (03)

### Material
How a surface looks: colour, metalness, roughness, opacity, emissive. (08)

### Mesh
A visible 3D object: a geometry plus a material, with a position and rotation. (08)

### MutationObserver
A browser feature that calls your function when part of the page changes; here, when the `data-theme`
attribute changes, to repaint the 3D backgrounds. (05)

### NDC
Normalised device coordinates: a screen position as −1 to 1 on both axes, (0, 0) in the centre, y up.
(10)

### OrbitControls
A Three.js add-on that turns mouse and touch input into orbiting, zooming and panning the camera around
a target point. (07)

### PerspectiveCamera
A camera that makes far things look smaller, set by field of view, aspect ratio, and near and far
distances. (06)

### querySelector and $
`document.querySelector("#id")` finds the first element matching a CSS selector. `$` is this file's
one-line shortcut for it (not the jQuery library). (01)

### Radian
The angle unit Three.js uses: π radians = 180 degrees, so `Math.PI / 2` is 90 degrees. (06)

### Raycaster and raycasting
Shooting an invisible line from the camera through a point on the screen to find which objects it
hits, nearest first. (10)

### Render loop
The function called every frame to update things and draw the scene: here `loop()`. (06)

### Renderer
The object that draws a scene, as seen from a camera, into a `<canvas>`: here `THREE.WebGLRenderer`. (06)

### requestAnimationFrame
Asks the browser to call a function just before the next screen refresh (about 60 times a second). The
function asks again each time, which makes a loop. (02)

### ResizeObserver
A browser feature that calls your function when an element changes size. (06)

### Scene and scene graph
The scene is the root of everything drawn in 3D. The scene graph is its tree of groups and meshes. (06)

### Set
A collection without duplicates. `new Set(list)` drops repeats; `[...set]` turns it back into an array.
(03)

### setInterval
Calls a function repeatedly every so many milliseconds, until `clearInterval` stops it. (11)

### Spread and destructuring
`...arr` unpacks an array into separate values (`f(...[1, 2, 3])` is `f(1, 2, 3)`). Destructuring names
an array's items: `([h, body]) => ...`. (01)

### SVG
Scalable Vector Graphics: a 2D drawing written as tags (`rect`, `circle`, `text`, `path`). (05)

### SVG path
The `d="M50 110 V60 H600"` mini-language for lines and curves: move, line to, horizontal, vertical,
close. (05)

### Template literal
A backtick string with `${expression}` holes: `` `<h3>${name}</h3>` ``. (01)

### Ternary
`condition ? valueIfTrue : valueIfFalse`, an `if` that gives a value. (01)

### Three.js
A JavaScript 3D library built on WebGL. This app pins version r128. (06)

### traverse
`obj.traverse(fn)` calls `fn` on an object and every descendant in its tree. (08)

### URL hash and hash routing
The hash is the `#part` at the end of an address. Hash routing uses it to choose which view to show,
without reloading the page. (00, 02)

### userData
A free object on every Three.js object (meshes and materials too) for your own fields, like
`partId` and `label`. (08)

### Vector3
An (x, y, z) triple used for positions, directions and displacements, with methods like `copy`,
`multiplyScalar`, `lerp` and `distanceTo`. (07)

### viewBox
The coordinate rectangle an SVG drawing is defined in, e.g. `0 0 820 300`; it is scaled to fit the
screen. (05)

### WebGL
The browser's API for drawing with the graphics card. Three.js uses it. (06)
