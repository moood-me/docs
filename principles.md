# How we make scenes and games — the principles

Read this first. It is how this project works, said by its author; the other documents (`scene-format.md`,
`people.md`, `engine2-porting.md`) say how — this says what matters and what never to do. When a rule here and a
habit of yours disagree, the rule wins.

## What matters

- **Quality for the viewer and performance** — not ease of making. Never pick the easy way or "good enough";
  pick what the viewer sees and what runs smoothly. Rewriting a scene (or the engine) whole is fine; backwards
  compatibility isn't needed.
- **Only the right ways.** No compromise, no workarounds, no shortcuts — common sense and real best practice, every
  time, not "the game is simple, we'll do it later". If something doesn't work, don't make it work at any cost:
  fix it in the right direction, or say what's missing.
- **No prop-ups for one scene.** There will be many scenes: anything bent to fit one breaks another. The engine is
  general and right; a scene that hits an engine gap reports it (what, where, how to reproduce) and doesn't fake
  around it — an invisible element standing in for a point, a clip trick, a number tuned to hide a bug, a scene
  light making up for light the engine should give.
- **Keep all the beauty.** Nothing beautiful in a scene is dropped or simplified when it is rewritten. Reach it
  another way if the old way doesn't fit (a GPU-friendly way, as good games do) — don't insist on "as it was",
  but never lose it.

## How to work

- **By reading, not by render loops.** Write the whole thing; review it by reading against the language (every
  promise of `scene-format.md` → your code; every seam → both sides); collect every defect; fix them all at once;
  only then render. Fix everything the renders show in one batch; render once more. Never "tweak a little, look
  at a hundred frames, tweak again" — renders are slow, and errors found one by one by rendering are found late.
- **Render on the GPU through moood**, never on your own machine in software: `render_scene` / `look_at_branch`
  with `on: "gpu"` (`sheet`, `at`, `state`, `views`, `showcase`). Open a picture only when you need it — it costs.
- **Long commands with long timeouts, polling loops, killing processes by name** — none of them.
- **Say what you did**: the scene's `notes.md` (what is where, why, what changed in meaning); a generator that
  makes a scene lives in the world's repository (`tools/`), never only on your machine.

## A scene

- **Painted once, moved by the engine.** A painting is drawn once per state; nothing a painting reads may move
  while a scene plays. Motion is the engine's, on the GPU: layers (`wind`, `drift`, `flicker`, `shimmer`, `pivot`)
  moved by `animate`, an element's own shader, particles, people, light. A key a painting reads that changes every
  frame repaints every frame — never. A painting that keeps repainting gets a warning: it means this rule is
  broken.
- **Light is the engine's.** Paint in daylight colours; the scene's light lays the look (time of day, lamps,
  rims, the haze after the light — linear: old look numbers come out darker, tune them on renders). Don't paint
  pools, rims, halos or the hour into pictures; a lamp's halo is its light plus the haze's glow.
- **Driven by the game's events, not by seconds.** The scene is the presentation (what is there, how it looks,
  how it can move); the game is the logic (what and when). The scene offers actions to await, tracks, sequences
  and events (`scene-format.md`, the director); its beats are ≤ 30 s, a seek point at each start. A film that is a
  table of seconds is the mistake we left behind.
- **Its own showcase** (`showcase.js`): outside games the scene plays its own little scenario showing everything it
  can do, with a timeline and seeking.
- **People at their real speed.** A motion plays at the speed its body model gives — never faster or slower,
  never stretched to fit a time or a distance; another speed is another gait, made as such (`make_motion`).
  A scene that needs someone somewhere by a moment waits for them to arrive.
- **No fades to hide a change.** Prepare the next shot ahead (`s.shot`) and cut; the change happens on the cut's
  frame. A fade only as art (a chapter's end, time passing).
- **A world where it is.** Lay things where they are and move the camera over them (kilometre worlds are fine) —
  never scroll paintings past the eye to fake travel.
- **The game's contract is kept.** Keys, element ids, named points, beat names a game or another scene uses keep
  their meaning — or say in `notes.md` what changes and why. Tell the game the moments it needs (`scene.tell`,
  a key it can watch, where someone is in the game's own units); a game must never copy a scene's numbers or
  wait milliseconds for a scene's moment.

### Pitfalls already met

- A cast's `F.fit.h` is the person's height as a **share** of a man's 1.76 m — `F.fit.h / 1.76` makes every part
  0.57 size. Parts are in metres of a man: `x * F.fit.h`.
- A layer that travels (drifts, wraps in a shader, is moved by `animate`) needs a `box` covering its whole path,
  including a shader's wrap range; a layer that paints nothing drops out and renumbers `layer(k)`.
- Everything an `animate` sets must be a number (`e.aspect` exists there too); a non-number is refused.
- Something seen only in a mirror: `"seen": "reflection"` — never a clip or an invisible plane.
- A point a game needs on a close-up or the sky: mark it (`e.at`) in the insert / sky — the page gets it as a
  screen point.
- A mirror shows only where its plane is painted.
- Several agents on one world: the shared modules have one owner; others ask for changes through them.

## A game

- **The game leads the scene by events**: it sets keys and runs actions, and waits for what the scene tells
  (an action ended, someone arrived, a line was heard, a shot cut) — not for seconds. Its own pauses are its own;
  a scene's moment is the scene's event.
- **Nothing slow for the player**: a click, a menu, a highlight, a minigame answer at once (the next frame); the
  page's main thread does almost nothing per frame.
- **The player always sees what the game waits for** (a line saying what's now; what can be pressed glows);
  something meaningful in the first 30 seconds; short chapters (5–10 min) that continue where left; fewer choices
  but sharper, their consequences seen soon; minigames that work on a phone and can be skipped after a failure;
  everything voiced; an AI character with a wish and a secret; every branch checked automatically (each beat
  played: black frames, errors, stalls).

## What a frame must cost (engine 2, the GPU stand)

60 fps (p95 ≤ 16.7 ms, worst ≤ 33 ms) — playing, zooming, dragging; the engine's and the scene's JS ≤ 3 ms a frame
(p99 ≤ 6 ms); in a settled frame zero repaints and zero uploads; GPU ≤ 8 ms; no GC pause over 2 ms; the page's main
thread ≤ 1 ms a frame, no task over 50 ms; open to a ready frame ≤ 2 s; a prepared cut ≤ 1 frame late; input seen
the next frame. A scene made by these principles passes; one that doesn't, isn't done.
