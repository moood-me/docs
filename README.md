# moood — for agents

moood (https://moood.me) is living, animated worlds. A **world** is a collection of
**scenes** — places with their mood, painted in code, animated, with state that changes as the
world is played. Games are played on worlds; this is about the worlds.

You act for a person, with their rights: what they may see, you see; what they may change, you
change.

**Read `principles.md` first**: what matters here (the viewer's quality and performance, only the right ways, no
prop-ups for one scene, all the beauty kept), how to work (by reading, not by render loops; rendering on the GPU),
what a scene and a game must be, and the pitfalls already met.

## Two ways in

- **MCP** (moood's MCP server, connected in your client): read worlds and scenes, make them, edit a scene in
  an editing session (`begin_editing`, `draft`, `commit`) — whoever watches the scene on the site
  sees every draft live. Good for a quick change of one scene.
- **The world's git repository**: everything else — many files, many scenes, anything long, shared
  code. `world_repo(address)` (an MCP tool) gives you the clone URL and a token for that repository
  alone (an hour; ask again for another); with GitHub access of your own to the org `moood-me`, use that.

Both lead to the same place: a world lives in its repository, `moood-me/<space>--<slug>` — the
repository is the world, the same one on every deployment of moood. Each deployment has a branch of its own
there: `env/dev` is moood's own (moood.compl.dev), `env/<computer>/<stack>` a stack on someone's computer;
there is no `main`. The deployment's branch is the world, for that deployment: every change of a
scene there is a new version of it; nothing is ever lost. A deployment shows a world whose repository has its
branch — a stack's branch follows another (usually `env/dev`): made from it, moved on with it while it has
nothing of its own. `world_repo` says which branch is the deployment's you work with (`branch`).

## The repository

    world.json                         title, description, themes, main (the main scene's folder; none: the first scene)
    scenes/<folder>/scene.json         a scene: its spec without its code
    scenes/<folder>/elements/<id>.js   an element's code          (objects[].code in the spec)
    scenes/<folder>/elements/<id>.script.js   an element's script (objects[].script)
    scenes/<folder>/elements/<id>.animate.js  what moves it each frame (objects[].animate — engine 2, engine 3)
    scenes/<folder>/elements/<id>.glsl  its own shader            (objects[].shader — engine 2)
    scenes/<folder>/elements/<id>.wgsl  its own shader            (objects[].shader — engine 3)
    scenes/<folder>/script.js          the scene's script          (spec.script)
    scenes/<folder>/showcase.js        its own scenario outside games (spec.showcase — engine 2, engine 3)
    scenes/<folder>/lib.js             code its elements share      (spec.lib)
    scenes/<folder>/relief.js          the land's shape            (spec.relief)
    scenes/<folder>/widgets/<id>.html  a widget's markup           (widgets[].html)
    scenes/<folder>/notes.md           the scene's notes            (spec.notes)
    modules/<name>.js                  a module: code the world's scenes share
    modules/<name>.data.json           its data, if it has any (not formatted)
    modules/<name>.md                  what it offers and how to call it

A file belongs to the element or widget of `scene.json` with that `id`; a file nothing in
`scene.json` names is an error. An element's code is the body of its draw function, as
`scene-format.md` describes (the text of the scene's engine: What a scene is, below). An element's shader is the
file its scene's engine names: `.wgsl` in a scene that says `"engine": 3`, `.glsl` in any other — the other one
is a file nothing names. A new folder is a new scene. The repository's name is
`<space>--<slug>`; the world's address on moood is `<space>/<slug>`.

## Modules

A module is code the world's scenes share — its people, its animals, its things — so they are the
same in every scene. `modules/<name>.js` is the body of `function (anim, data)` returning what it
offers (`anim`: the animator's toolkit; `data`: `modules/<name>.data.json` parsed, or null); its
`.md` says what it offers and how to call it — read it before using the module, and keep it true
when you change the module. A scene lists the modules it uses in `scene.json` (`"modules":
["people"]`) and gets them as `e.kit.people` in its elements' code (`scene.kit.people` in scripts,
`kit.people` in its lib).

Every scene shows the modules as the world has them now: a change of a module reaches every scene that
uses it, at once — it is a new version of the world, not of those scenes (a scene's versions are
the changes of its own folder; an old version opens with the modules as they were then). So before
you merge a change of a module, look at the scenes that use it (`look_at_branch`), not only at the
one you made it for. A merge is refused if a scene lists a module that isn't there, or a changed
module isn't valid JavaScript.

People in a scene are elements with `person`: drawn in 3D as the camera sees them — a figure of designed
shapes in one of its styles, or a real body in real clothes — moved by real human motion, lit by the scene, walking
where their place goes; what they do is the scene's state (`<id>_do`, `scene.person(id)`). Who they are is the
world's: a module that is its cast (its motions — `put_motion` writes one in —, its characters, its own styles,
garments, parts, bodies). All of it: `people_guide` (people.md here).

An older people module's recorded motions are entries of its data (`{n, fps, v, p, c}`: bone directions, the
pelvis, foot contacts). A motion none of its recordings gives — someone getting up and walking to the
window, a hand on another's shoulder — `make_motion` makes: a model trained on motion capture, from plain
English prompts and constraints (where the person goes, poses taken from motions made before; going on
from one). `look_at_motion` shows it on a person — anyone, in any of the styles (a real body in clay, flat or
silhouette; a figure of shapes from a sign to a toy), with a gallery of them all; it answers with such an
entry. How to ask it well, and how a person may be seen: `motion_guide` (motion.md here).

`sandbox/` here holds working examples made outside moood (a WebGL fashion show: walks put onto any
skeleton, garments over bodies, light, cameras) — ideas and pieces to take into a scene, rewritten
the way the scene format asks.

## Working in it

1. Clone — with the git arguments `world_repo(address)` gives (the token travels in a header — never put
   it into a URL or a credential store), or with your own access.
2. Work in a branch of your own, begun from the deployment's (`branch`); commit and push it as often as
   you like.
3. `look_at_branch(address, branch, folder)` shows a scene of your branch rendered, with the errors
   its code throws (moments, sheets, films: Looking at your work, below). Look at your work: does it
   show what was asked, and is it beautiful?
4. Put it into the deployment's branch: merge it there with git and push — or `merge_branch(address,
   branch, note)`, which checks it first and refuses, changing nothing, if the branch changes a scene the
   person may not change, leaves a folder that isn't a scene, or conflicts (merge the deployment's branch
   into yours, push, try again); it deletes your branch once merged. Either way each scene changed gets a
   new version, a new folder a new scene. A deployment's branch only goes forward: never rewrite it (no
   force-push, no rebase) — plays and versions hold its commits.
5. Another deployment gets the work the same way: merge into its branch. A world it doesn't have yet (its
   branch isn't in the repository) it gets once you push that branch — made from yours.

A new world: `create_world` — or make the repository yourself: a private `moood-me/<space>--<slug>`
(`<space>`: the person's handle), with `world.json` (`{"title", "description", "themes": {}}`) and its scenes'
folders, if any yet, pushed as a deployment's branch (`env/…`). Every deployment where the person is shows it
once it has its branch. A name is one world: never make a second repository for a world there is —
`create_world` refuses a name taken, here or by another deployment. A world may have no scene; its main scene
is its first, unless `world.json`'s `"main"` names another's folder.

moood formats the code it writes (Biome: 2 spaces, 120 columns); write your code readably too, one
statement a line.

## Games

A game played on worlds has a repository of its own, `<space>--game--<slug>`: `game.json` (its
type, title, description, genres, the world it is played on — that world's repository — and the
scenes it takes in, `{world, folder}`) and a folder of its type (`moood/agents.json`, what
the agents are told; `asked/story.json`, the story; `bespoke/`, the game's own code — `game.js` and
what it imports) that only that game type reads. Like a world's, it has a branch for each
deployment (`game_repo` says which); its versions are the commits of that branch. A play keeps the
commits it began on — the game's and its worlds' (an asked game's scenes: as their worlds were when
it began).

Through the MCP: `list_games`, `create_game`, `read_game` (game.json and its type's files),
`change_game` (title, description, genres, who sees it), `write_game_file` (one file of its type —
the game's next version). For more: `game_repo` (clone it, a branch of your own), then merge it into the
deployment's branch yourself or with `merge_game_branch` — which refuses if the branch changes anything but
`game.json` and its type's folder, or if `game.json` is no longer this game (its type changed, a world or a
scene it names isn't there). A new game is made the same two ways as a world: `create_game`, or the repository
`moood-me/<space>--game--<slug>` with its `game.json`, pushed as a deployment's branch. Only a game's author
changes it.

A bespoke game's mini-games (on engine-3 scenes) are made in the language of mini-games — how, and how to choose a
mechanic that is the character's action and the player's feeling: `minigames.md`.

A game's own sounds (Web Audio in `bespoke/`) obey the listener's third slider, "sounds" (beside music and voices;
`web/scene/volume.js`): the game puts `MooodVolume.output(ctx)` — a gain of its `AudioContext` that follows the
slider — between its sounds and `ctx.destination`, and never reads the slider itself. Lines of speech go through
`MooodSpeech` and obey "voices". The play's pause and the author's menu are the page's, the same for every game: a
game draws no ⏸ and no ☰ of its own — it stops what is its own (its sounds) on `bespoke.onPause(fn)`, and gives
`bespoke.menu(open)` what opens its menu (its chapters, where the play stands — drawn as it likes). `bespoke/` may hold files other than code — a recording (MP3), a picture — served as
they are beside it: the code fetches them by its own address, `new URL('sounds/sea.mp3', import.meta.url)`. Only what
is free to put in a public repository (CC0, or the author's own), and a few hundred KB each: every file is loaded by
whoever plays.

## Looking at your work

Three tools render a scene with moood's own engine — the version the site runs — and answer the
picture and the errors and warnings its code gave:
- `render_scene(address, device)` — a scene as the world has it (any version);
- `look_at_branch(address, branch, folder)` — as your pushed branch has it;
- `look_at_files(address, files)` — as files you send, not pushed anywhere: a scene's folder,
  `{path in it: text}` (the world's modules, or `modules` you send).

Each draws the scene from its starting viewpoint, or:
- `at: [0, 10, 30, 60]` — a frame at each second; `state` — values over its starting state; `set` —
  values set as it goes (`[[second, key, value, over?, ease?]]`, as a game sets them); `views` —
  other cameras on each moment; `sheet: true` — all of them in one labelled picture (up to 32
  frames). A sheet is the cheapest way to see a film move.
- `film: {"from": 0, "to": 60, "fps": 10}` — that stretch as an MP4; `pack: {"at": [...]}` — the
  frames as PNGs in a ZIP. Both are made in the background: you get a link at once that answers 202
  (and how far it is) while it is made and then the file (kept an hour). Ask after the link again within ten
  minutes, till the file is there: unasked after for that long, it is let go.
- The render service takes only so many renders at once (a few, of everyone's): full, it refuses a new one at
  once — "busy …", in the tool's answer or at a film's link. Ask later; do not ask again in a loop.

`on` says where it is drawn: `"server"` (the default — WebGL on the server's CPU: seconds a frame,
and a film there is short, 120 frames at most) or `"gpu"` (a GPU, for heavy scenes and long films —
up to 3000 frames). The GPU sleeps while unused: a call then wakes it and says so — ask
again in a minute or two (a film or a pack waits for it by itself). Nothing falls back from one to
the other. A scene that says `"engine": 3` is always drawn on the GPU, whatever `on` says — its engine draws
nowhere else; so are its tiles and every other picture of it, and a call that finds the GPU asleep says so.

`engine: "<branch>"` — drawn by the engine a pushed branch of moood's own repository has (its `moood/web`), not the
site's: for the engine's own developers — commit and push the engine change, name the branch; on either `on`.

**Measuring.** `measure_scene(address, scene?, version?)` — or `(address, branch, folder)`, a pushed branch's —
measures what the scene's frames cost: played on the GPU on the real clock as a viewer has it (opened, its film or
showcase, zoom, drag, lean, walking, cuts) and held against the thresholds of `principles.md` ("What a frame must
cost"). About two minutes a size (`sizes`: `["1080p"]` by default, `"4k"`), in the background: a link at once that
answers 202 (and how far it is) while it is measured — ask after it again within ten minutes: unasked after for
that long, the measuring is let go —, then the report as text (kept an hour) — each threshold ok or FAIL; per segment the frame
intervals, the scene thread's JS by stage, the GPU's time, repaints and uploads; the GPU's and the thread's time by
pass of the renderer's frame (shadows, opaque, skies, blends, bloom, upscale …); GC pauses; and the elements painted
again most, with the keys they read. Measure a scene once it is written and reviewed, before merging — not after every
change. `engine: "<branch>"` as above.

## Rendering on your own machine

`npm i -g @moood/render`, then `moood-render <world>/scenes/<folder> --at 0,2.5` — a scene of a
cloned world rendered with moood's own engine (the version the site runs) in a Chromium you have,
without the site: a PNG for each moment and the errors its code threw. Its README tells the rest.
It draws WebGL in software: a single frame is quick, a long film is better made with `film` above.
(`--server` draws on moood's GPU with an engine sent from a checkout of moood itself — for the engine's own
developers, as the tools above do with `engine`; a world's scenes are drawn by the site's engine.)
A scene that says `"engine": 3` is not drawn in software: `moood-render` draws it only with `--server`.

## What a scene is

`scene-format.md` — the scene's language and its craft: the schema, the elements, how they are
painted and moved, state and the director (actions, sequences, events), the showcase, the camera,
light. It is the text moood's own artist works from: where it says to answer with a JSON object, you
write the files instead. The MCP tool `scene_format` returns the same text — each engine's by its number
(below).

Three engines run scenes; a scene's `scene.json` says which, and a scene keeps the engine it says and that
engine's language.
- **Engine 3** (`"engine": 3`) — every new scene: `add_scene` and the site's artist stamp it. WebGPU: it opens where the browser has WebGPU, and its pictures come only from the GPU machine (above).
  Its language is a text of its own, complete in itself — `scene-format-3.md` (`scene_format()`): read it, not
  `scene-format.md`, for such a scene; its own rules and principles are there. An element's shader in it is WGSL
  (`elements/<id>.wgsl`).
- **Engine 2** (`"engine": 2`): painted once per state, everything that moves moved on the GPU, driven by
  events — `scene-format.md` (`scene_format(engine=2)`).
- **The old engine** (no `"engine"`): a scene made before runs on it in its own older language —
  `scene-format-1.md` (`scene_format(engine=1)`) — until it is rewritten for engine 2 (then all of it:
  its paintings static, its motion by layers, animate and the director). How to move one: `engine2-porting.md`.

A scene of engine 2 is carried over to engine 3 only on the world's owner's word, in a copy of the world:
`engine3-porting.md`.
