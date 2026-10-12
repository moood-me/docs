# Carrying a scene from engine 2 to engine 3

A scene keeps the engine its `scene.json` says. One that says `"engine": 2` runs on engine 2 in its language
(`scene-format.md`) and nothing asks it to move. It is carried over to engine 3 (`"engine": 3`, `scene-format-3.md`)
only **on the world's owner's word** — never as a side effect of other work — and **in a copy of the world**: the
world as it is stays, with its games and its plays.

Engine 3's language is engine 2's with a short list of differences, so carrying over is a set of known edits, not a
rewrite. This is about scenes of engine 2 only; a scene of the old engine (no `"engine"`) is another matter —
`engine2-porting.md`.

## How to work

`scene-format-3.md` first — whole: it is complete in itself, and its rules (the engine's limits are kept, not got
around; a thing to be pressed needs a point) hold for every scene carried over.

1. **A new world** for the copy (`create_world`), of the same owner.
2. **The copy, as it is.** In a branch of the new world's repository, begun from the deployment's branch: the whole
   tree of the world carried over — `world.json` (the new title, the same main scene), `modules/`, every folder
   under `scenes/` — nothing changed, every scene still `"engine": 2`. `merge_branch`. From here the copy has the
   same scenes, drawn by engine 2: the reference every engine-3 picture is held against, kept in each scene's
   history.
3. **The conversion**, in a second branch: the edits below, every scene of it `"engine": 3`. Write all of it, then
   review it by reading against `scene-format-3.md` — every element, shader, shot, key — and fix; only then look at
   pictures; fix everything they show in one round; look once more.
4. `merge_branch` — each scene a new version. Only into a deployment whose moood has engine 3: an older one runs an
   `"engine": 3` spec as an old scene.

Every edit keeps what plays on the scene: each element id, named point (`e.at` names, `id#point`), state key, person
id and `scene.tell` a game or another scene uses keeps its name and its meaning.

## The edits

| What the scene has | What it becomes |
|---|---|
| `"engine": 2` in `scene.json` | `"engine": 3` |
| an element's shader, `elements/<id>.glsl` (GLSL) | `elements/<id>.wgsl` (WGSL), converted by the table below — mechanical, no look changes. A generator that writes shaders (the world's `tools/`) is converted too, so that it still makes the scene |
| a film module's game path that cuts to a beat with a plain `scene.cut` | it says the next beat's opening ahead — `scene.shot(values)` of the beat that follows when a beat begins (or of the beat the game names) — and cuts by that shot. Without it every cut of a game is held for a moment under the sign of waiting |
| `scene.prepare(values, seconds)` | a shot said ahead for the very place the cut will be at — `const ready = scene.shot(values)` — and the cut made by it, `ready.cut()` |
| a script that sets a camera key every frame to follow something (it cost nothing in engine 2) | it moves the key over time instead (`scene.move`, a path, beside what it follows — the same seconds and ease): a camera key set at once is a new shot, shown whole like a cut — a shot a frame stutters and slows the film |
| a scene that paints its own light — no `"ambient"`, `"light": { "people" }`, a look's `"people"` | lit by the engine: paintings in daylight colours, the hour and the lamps as the scene's light and its looks. A large edit, and the look changes — the owner sees it before it is merged |
| particles' `"veil"` | the look's `haze` and `hazeColor`, turned by the look the storm is |
| many alike things as separate paintings (a forest, a town's roofs) | `copies` of about twenty painted kinds, told apart by size, turn, flip and tint |

Not edited — these words are accepted and do nothing in engine 3, so a scene carried over keeps them (a new scene
does not write them): a caster's `soft`, a light's `size`, a mirror's and a pane's `blur`, a plane's `lit`. Nor
edited: how many lights a scene has (any number; the engine chooses which light and which cast); its plain cuts and
its close-ups (an insert opens finished by itself); its `animate`s and its `scene.every`.

Not built in engine 3 — if the scene uses one, it cannot be carried over as it is: say so, and do not imitate it
with something else: a plane's `travel`; `seen: "direct"` and `seen` as a list; particles' `onto` and `lit`; a
layer's `blend: "screen"`; a person as a body (`bodies`, `outfits`, a body's garments and face); a plane draped on
the land named in a `clip`; more than four different `clip` lists in one scene; a person in a `clip` — `clip` is for
cards and planes, a person is neither clipped nor clips; a `shader` of its own on a plane with `drape` (it is said
once and not taken).

No scene is given `copies` or a form merely because engine 3 has them; a carried scene stays as it was made unless
the owner asks.

## A shader, GLSL → WGSL

The text is still one function and what it needs above it; `shader` and `uniforms` in the spec keep their names.

| GLSL | WGSL |
|---|---|
| `vec4 shade(vec4 painted, vec2 at, vec2 uv) { … }` | `fn shade(painted: vec4f, at: vec2f, uv: vec2f) -> vec4f { … }` |
| `float`, `int`, `uint`, `bool` | `f32`, `i32`, `u32`, `bool` |
| `vec2`, `vec3`, `vec4` | `vec2f`, `vec3f`, `vec4f` |
| `float x = …;` / `const float K = …;` | `let x = …;` (`var x = …;` if it is assigned later) / `const K = …;` |
| `float a = 1.0, b = 2.0;` | one declaration a statement |
| `c ? a : b` | `select(b, a, c)` — both sides are worked out |
| `mod(x, y)` | `wrap(x, y)` |
| `fwidth(at)`; `fwidth(at.x)` | `abs(at_dx) + abs(at_dy)`; `abs(at_dx.x) + abs(at_dy.x)` |
| `hash1(vec2)`, `hash1(vec3)`, `hash1(uint)` | `hash2`, `hash3`, `hashU` |
| a uniform `name` (from `"uniforms"`) | `u_name` in the text; the spec's name and `e.uniform(name, v)` stay |
| `float[5](…)` | `array<f32, 5>(…)` |
| a parameter that is assigned | a `var` copy of it |
| `v.rgb += x;` (a part of a vector assigned) | the whole vector assigned: `v = vec4f(v.rgb + x, v.a);` |
| `if (c) x = 1.0;` | braces round every body: `if (c) { x = 1.0; }` |
| `for (int i = 0; i < 12; i++) { … }` | `for (var i = 0; i < 12; i++) { … }` — constant bounds |
| `float f(float x) { … }` | `fn f(x: f32) -> f32 { … }` |

Keep their names: `layer(k, at)` (k 0 … 9; outside the painting it gives nothing, a layer that wraps is read
round), `emission`, `u_time`, `cycle`, `noise1`, `noise2`, `noise3`, `wobble`,
`wobble2`, `luma`, `srgbToLinear`, `PI`, `uv`. Not there: textures of its own, `discard`, derivatives. A name of the
scene's may not begin with `e_` or `u_`, and WGSL's reserved words are not the scene's to use (a uniform is safe —
it is `u_<name>` in the text; its name is letters, digits and `_`, beginning with a letter). Numbers that are not whole are written with their point. Convert by reading the
types of every expression — WGSL does not turn an integer into a float by itself.

## What will look different

Carried over well, a scene shows the same place; these are the engine's own differences, not faults of the edit:

- It opens under a sign of waiting and then shows a finished picture — never a blurred one sharpening. A cut said
  ahead comes at once, finished; one nobody said holds the last picture for a moment.
- The picture is no larger than 1920 × 1080 (1600 × 900 on older built-in graphics, 960 × 540 or 1280 × 720 on a
  phone), stretched to the screen; its size is not lowered as it plays.
- A picture its shader reads at another point than its own (moved, tiled), and a layer that drifts, is never
  sharper than about a screenful of pixels; a shader that reads its paintings where it stands is as sharp as any
  painting.
- A view with more painting than the memory holds is evenly a little softer, and stays so.
- A shot made ready beside a heavy view, or while its camera rode something fast, is held briefly at its cut for
  its fine detail.
- A camera moving faster than the painters shows the new strip a little softer for a moment; floors and water at a
  low angle are a little softer along the gaze.
- The scene stops, its voices and music too, while its tab is hidden, it is scrolled off the screen or a viewer is
  open over it.
- Shadows: one softness; at most four lights cast at once, chosen by the engine; what only sways casts as it stands
  at rest; no coloured shadows.
- Lamp pools are sharp on floors and walls from the two lights that matter most, soft from the others; a far small
  light starts to light things only once its pool is a few pixels wide — its glow is always there. Of the far lights
  (a sun, a moon) eight light at once at the most, of the lights `through` windows four — the strongest.
- Mirrors: a small image (the rougher the mirror, the coarser), smeared along the mirror's way as `rough` says,
  with the path of light under what shines; a mirrored thing fades out with its height as `fade` says. No particles
  in them, no blur by a mirrored thing's height, and everything in a mirror is lit at its vertices (no sharp lamp
  pools in a reflection).
- No weather curtain (a blizzard is thicker air); nothing is lit by the land from below.
- A scene that painted its own light is lit by the engine: the hour's colours come from light, not paint.
- People's loose cloth and hair are simpler, and at rest after a cut or a jump — as the pose has it: over the
  thighs of someone seated, as wide as a stride needs.
- What a game offers is pressed at the same round marks; the offered thing itself breathes brighter, where engine 2
  glows along its outline.
- It opens only where the browser has WebGPU.

What must not change: people, faces and the action read clearly in every frame. If a carried scene is darker or
harder to read than its engine-2 picture, that is a fault to mend (the scene's light, its looks) or to report.

## How it is checked

- **Pictures, on the GPU machine only** — an engine-3 scene is drawn nowhere else (`look_at_branch` of the
  conversion's branch: the starting view, a `sheet` of its showcase, its looks through `state`; asleep, the machine
  wakes in a minute or two — ask again). Each is held beside the engine-2 picture of the same moment: the copy's
  first version (`render_scene` with that version), or the world it was copied from. On the site, the comparing
  page shows a scene's two engines side by side. Compare the same moments on both engines, and read engine 2's own
  log of that picture: its warnings ("the virtual texture's page table is full", "more than 32 lights reach one
  place") mark the places where engine 2's picture is the broken one, not the reference — there engine 3's
  picture stands.
- **The stand** — `measure_scene` on each scene once its pictures are right: what its frames cost and where.
- **By reading** — the game's contract: every id, point, key and tell a game reads is still there, under its name.
