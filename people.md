# People in a scene

An element with `person` is someone standing in the scene — drawn in 3D as the scene's camera sees them, moved by
real human motion (NVIDIA Kimodo's), lit by the scene's light, in front of and behind what is round them pixel by
pixel, casting shadows like any element. Who they are is a **look** (the same as `look_at_motion`'s — motion.md,
"How a person is seen": a figure of designed shapes in one of its styles, or a real body in real clothes); how they
move is the scene's **state**: where they stand (the element's `place`), what they do, where they face.

```json
{ "id": "rosa", "name": "Роза", "person": { "look": { "kind": "shapes", "style": "soft", "who": "lady" } },
  "across": -0.6, "distance": 4.6, "place": { "across": "rosa_a", "distance": "rosa_d" }, "shadow": true }
```

```js
// the scene's script: she walks to the lamp, turns to us, waves (a motion of the world's cast)
const s = scene.person('rosa').walk([[0.2, 4.2]]);
scene.after(s, () => { scene.person('rosa').turn(180, 0.8); scene.person('rosa').do('wave'); });
```

Two people by a lamp — a figure of shapes and a body, lit by it, their shadows cast:
[people/scene-lamp.webp](https://github.com/moood-me/docs/blob/main/people/scene-lamp.webp).

She stands and breathes, walks with her steps as long as the way she goes (her feet never slide), faces the way she
goes, turns, does what she is told, and goes back to standing. Her head is her point (`e.at`): her lines appear over
it, `camera_target: "rosa"` frames her, `"rosa#hand_r"` aims at her hand.

## The element

| field | what it is |
|---|---|
| `id` | needed: the keys of their state are named by it |
| `person.look` | who and how seen: `{"kind": "shapes", "style", "who", "garment", "hair", …}` or a body `{"who", "style", "outfit", …}` — motion.md, "How a person is seen" (and its gallery) |
| `person.cast` | the world's module their people are of (it must be in the scene's `modules`): its motions, characters, styles, garments, bodies… (below) |
| `person.who` | a character of the cast (`cast.characters[who]`: a look); `person.look` goes over it |
| `person.turn` | where they face standing, degrees: 0 away from the camera at the start, 90 to its right, 180 towards it (the default), 270 to its left |
| `person.does` | what they do when nothing is said (a motion's name; `stand` by default); `person.gait` — how they walk (`walk`) |
| `across`, `distance`, `elevation`, `place` | where they stand, as any element; to walk, `place` names the state keys of `across` and `distance` |
| `shadow`, `behind`, `inFront`, `inside`, `after`, … | as any element's |

Their height and box come from their look (a child is small); no `code`, no `height`, no `box`.

## Their state

| key | what it is |
|---|---|
| `<id>_do` | what they do: a motion's name (`"sit"`), or `{"motion", "loop": true \| false, "from": seconds into it}`. A new value goes over from what they did in a third of a second. Looped: on the motion's best loop (found by itself); not: played once, then held at its end |
| `<id>_turn` | where they face standing (degrees, as `person.turn`); they turn to it at a person's pace |
| `<id>_mood` | a body's face: an expression (`smile`, `sad`, `angry`, `surprised`, `thinking`… — motion.md) |
| `<id>_talk` | a body's mouth talking: 0…1 |
| `place`'s keys | where they stand; moved, they walk there |

A game sets these as it sets any key. A script has `scene.person(id)`:

| | |
|---|---|
| `.do(motion, {loop, from})` | what they do (again, if it is the same: it starts over) |
| `.walk([[across, distance], …], {speed})` | along the points (metres, at `speed` m/s — 1.3), facing the way, and stays facing so; returns its seconds |
| `.turn(degrees, over?, ease?)` | where they face standing |
| `.mood(expression)`, `.talk(on)` | a body's face |

## Motions

The engine has `stand` and `walk`. Everything else is your world's: make it, look at it, put it into the cast.

1. `make_motion(["A person waves with the right hand, smiling."], [3])` (motion.md: how to ask well) → its key.
2. `look_at_motion(key, look={…the person's look…})` — on the person who will do it.
3. `put_motion(world, key, "wave", branch)` — written into `modules/cast.data.json` of your branch (the module made
   if there is none: its data is the cast). The motion never passes through you: no tokens spent on it.
4. In the scene: `"modules": ["cast"]`, the person's `"cast": "cast"`; then `<id>_do: "wave"`.

A motion is played where they stand: what it does over the floor is left out (its turns kept). To go somewhere —
walk. A motion that moves them far (steps, a jump forward) is not for `do`.

## The cast: the world's own

The cast module returns its cast (`return data;` — its data is it; or code adding to it):

```js
// modules/cast.js
return {
  ...data,                                   // motions: put_motion's
  characters: {                              // looks by name: person.who
    rosa: { kind: 'shapes', style: 'ink', who: 'lady', colors: { garment: '#5a2e3a' }, parts: ['satchel'] },
    gleb: { who: 'oldman', outfit: 'warm', style: 'clay' },
  },
  shapes: {                                  // a figure of shapes' own, over ours
    styles: { ink: { from: 'graphic', line: 2.4, paint: 'toned', ground: 'paper' } },
    characters: { lady: { from: 'lady', body: { height: 0.9 } } },
    garments: { cloak: { name: 'плащ', hang: [{ to: 1.6, from: 'shoulders', flare: 0.35, ragged: true }], sleeves: 'garment' } },
    hair: { curls: { name: 'кудри', draw: F => { /* F.head, F.ball… (below) */ } } },
    parts: { satchel: { name: 'сумка', draw: F => {        // a bag in her right hand: where the hand is, as it turns
      const { add, mul } = MooodShapes.V, hand = F.P('RightHand'), axes = [F.side('RightHand'), F.up('RightHand'), F.fwd('RightHand')];
      F.ball(add(hand, mul(axes[1], -0.1 * F.k)), axes, [0.05, 0.09, 0.11].map(v => v * F.k), F.colors.accent, 14);
    } } },
  },
  outfits: { warm: { name: 'тепло одет', wear: ['boots', 'vikingpants', 'sweater'] } },  // a body's: our garments and the cast's
  // garments: { cape: {kind, bin, credit} }, bodies: { hero: {base, bin, height, parts, credit} } — the people tools'
};
```

This very cast in a scene — rosa in the `ink` style with her bag, gleb in the `warm` outfit:
[people/scene-cast.webp](https://github.com/moood-me/docs/blob/main/people/scene-cast.webp).

**A figure of shapes** (`cast.shapes`) — each table over ours by name; a style or a character may say `from` (one it
is made from, ours or the world's, its own fields over it):

| table | an entry |
|---|---|
| `styles` | `{name, head, egg, neck, legs, limb, sides, square, trunk: {w, d}, arms, feet, paint, line, folds, ragged, rigid, straight, drape, face, ground, …}` (shapes.js: STYLES) |
| `characters` | `{name, body, garment, hair, hat, hood, scarf, socks, face, parts, colors}` |
| `garments` | `{name, hang: [{to, from: 'shoulders' \| 'waist', flare, off, color, folds, ragged, pleats}…], sleeves, short, bare, bodice, draw?}` |
| `hair`, `hats`, `faces`, `parts` | `{name, draw(F)}` — `parts` worn by a look's `parts: [names]` |
| `arms`, `legs` | profiles `[[where, half-width]…]` (0 shoulder/hip, 1 elbow/knee, 2 wrist/ankle, 3 tips) |
| `paints`, `grounds` | names; `{name, bg, ink, grid, rim, light, haze}` |

`draw(F)` draws on the figure being made, as ours do (all of ours are made so — shapes.js shows them):
`F.look`, `F.style`, `F.colors` (each `[r, g, b, toned]`), `F.fit.h` (height), `F.k` (thickness), `F.sides`;
the joints — `F.P(name)` (where; SOMA's names: Hips, Spine1, Chest, Neck1, Head, LeftArm, LeftForeArm, LeftHand,
LeftLeg, LeftShin, LeftFoot… motion.md), `F.fwd(name)`, `F.side(name)`, `F.up(name)` (its axes); the head —
`F.head {at, axes, egg, r, out, sides, square, on(direction, out)}`; the trunk's sections `F.trunk(scale)`,
`F.ringOf(section, n, square)`; what is worn round the chest `F.worn`; `F.clear(point, ±1, r)` (kept before or
behind what is worn); `F.swing(name, target, stiffness)` (a point following, a little late: hanging things); and
the solids — `F.ball(c, axes, radii, colour, sides, square?)`, `F.cap(c, axes, radii, direction, edge, colour,
sides, square?)`, `F.tube(points, profile, colours, sides, hint, flat?, square?, cut?)`, `F.bar(a, b, ra, rb, colours,
sides, hint, flat?, square?)`, `F.shell(c, axes, radii, edge(a), flare, colour, sides, square?)`, `F.rings(rings,
colours, first?, last?)`. Vectors: `MooodShapes.V` (`add, sub, mul, dot, cross, mix, unit, level, mean`).

**A body** — the cast's `bodies`, `garments`, `outfits`; a look's `who` may name a cast body, `outfit` a cast outfit
(its `wear` from the inside out: shoes, a bottom, a top, an apron, then hair — ours and the cast's), `hair` a cast
garment of kind hair. Made with the people tools (Docker):

```
docker run --rm -v people-tools:/cache -v "$PWD:/work" ghcr.io/moood-me/people-tools neutral       # the body to model on
docker run --rm -v people-tools:/cache -v "$PWD:/work" ghcr.io/moood-me/people-tools garment cape.obj cape --kind top --hangs --middle
docker run --rm -v people-tools:/cache -v "$PWD:/work" ghcr.io/moood-me/people-tools body hero.glb hero --part Shirt=top
```

Each writes a JSON snippet for the cast's data and a picture to look at first (the image's README: every knob).

## Recipes

- **Someone walks in, stops, talks.** `place` keys; `walk(points)`; at its end `turn(180)` and, a body,
  `talk(true)` while their lines show; `talk(false)`.
- **Sitting.** A chair is an element; the person stands where its seat is, facing away from its back;
  `do({motion: 'sit', loop: false})` — a motion of yours that sits down and stays (make it end seated, still).
- **A crowd.** Several elements, each its own id and look; start them a little apart in time (`scene.after(i *
  0.3, …)`) and speed (`walk(…, {speed: 1.2 + 0.1 * i})`) — never in step.
- **Inside, by a lamp.** `"inside": true` on them as on the room; the lamp's light falls on the side facing it.
- **On a slope, on steps.** A `relief`: they stand on the land where they are, their feet on it.
- **The same person in every scene.** A character of the cast (`person.who`), not a look written out each time.
- **Your own style.** `shapes.styles: { mine: { from: 'soft', … } }`; look at it with `look_at_motion(key,
  look={kind: 'shapes', style: 'mine', cast: {shapes: {styles: {mine: …}}}})` — a look's `cast` is taken as the
  cast's tables (what is data: a `draw` is code, it comes with the module — see it in a scene: `look_at_branch`).

## Limits

- People only: Kimodo moves one human skeleton (no animals, no four legs). Children and old people move as adults do.
- A motion's way over the floor is left out (above): steps somewhere are a walk, not a motion.
- One walk (the cast's `gait`): the stride stays the motion's, its cadence follows the speed — a run is a motion of
  its own pace.
- Hands hold nothing yet, and nobody looks at anybody by themselves: a motion that reaches or looks.
- A figure of shapes' face doesn't move; a body's mouth talks without words (it follows no voice).
- Without WebGL2 (an old browser) people are drawn as flat silhouettes.

## How it is made (if you need it)

`web/scene/people.js`: a person's player mixes their motions on Kimodo's 77-joint skeleton (skeleton.js) — the act
now and the one it goes over from, a loop's seam crossfaded, the walk mixed in by the speed their place moves, its
phase by the way gone; the feet set on the relief (a two-bone reach). Each frame they are drawn by shapes.js or
figure.js into a texture of their own (ink.js: depth, lines, and how each pixel faces and how far it is), laid as
their element's card by gl.js — each pixel as deep as it is in the depth buffer, the scene's light laid on it by how
it faces. Their painting is a silhouette (Canvas 2D, picking). Motions: the motion service's `/person` (rotations,
the pelvis's place); the engine's own in `web/scene/people/`.
