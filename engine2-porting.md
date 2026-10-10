# Moving a scene to engine 2

A scene made before engine 2 (no `"engine": 2` in its `scene.json`) runs on the old engine in its old language
(`scene-format-1.md`). Moving it means rewriting it in the new language (`scene-format.md`) — all of it, in place (the
same folder: its old version stays in the scene's history), keeping everything beautiful it shows; the beauty may be
reached another way, never simplified away.

## How to work

1. Read the old scene whole (scene.json, elements, lib, script, relief, notes) and what plays on it: every key, element
   id, named point (`e.at` names, `id#point`) and person id a game or another scene uses must keep its meaning — or say
   what changes and why.
2. Write the whole new scene, then review it by reading against `scene-format.md` — every element, key, motion — fix,
   review again. Only then render; fix everything the renders show in one round; render once more.
3. Render on the GPU render service, not on your machine (`render_scene` / `look_at_branch` with `on: "gpu"`, a
   `showcase` sheet and the looks via `state`).

## What becomes what

| Old | Engine 2 |
|---|---|
| a painting reading `e.t` / `e.phase` (twinkle, sway, flicker, drift, smoke, snow) | a static painting + what moves it: a layer's `wind`, `drift`, `flicker`, `shimmer`, `pivot` moved by `animate`, the element's own GLSL (`shade`), particles |
| `beforeFrame` memory in a lib/module | the element's `animate` (and `e.mem`) — no painting reads time |
| `sway`, `bob` | a layer's `wind` or `animate` |
| landscape layers, `spec.sky`, `stars`, old `particles`, built-in kinds | sky elements (layers + shader), particles (`particles`, presets), cards |
| `scene.play([[at, key, value], …])` second rows | the director: actions on tracks, `scene.sequence(async s => …)`, events; a beat ≤ 30 s (seek points are beat starts) |
| a film playing when the scene opens | `showcase` — the scene's own little film outside games, with its timeline |
| a fade to hide a change | none: prepare the next shot (`s.shot(…)`), cut, the change on the cut's frame |
| `e.lights()` for rims | the scene's light lays rims, fronts and backs itself — paint in daylight colours |
| the painted haze of a far thing | the engine's haze (paintings are painted clear) |

## What to retune

- **Light is linear now** and the haze comes after the light: the old look numbers come out darker — tune `ambient`,
  `looks`, the lights' strengths, and `shafts` (the air's light) on the renders.
- **Weather and smoke**: start from the engine's presets (`blizzard`, `snow`, `rain`, `smoke`, …, with `veil` for the
  far whitening) and override little; a preset is tuned to look right.
- **People's parts**: a cast's `F.fit.h` is the person's height as a share of a man's 1.76 m, not metres; a point on a
  held thing is best marked where it is drawn (`F.point(name, at)`).
- **Kilometre scenes** are fine (a flight over 100 km of drape, cities of a thousand lights): lay the world where it is
  and move the camera over it, rather than scrolling paintings.
