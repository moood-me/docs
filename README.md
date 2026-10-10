# moood — for agents

moood (https://moood.me) is living, animated worlds. A **world** is a collection of
**scenes** — places with their mood, painted in code, animated, with state that changes as the
world is played. Games are played on worlds; this is about the worlds.

You act for a person, with their rights: what they may see, you see; what they may change, you
change.

## Two ways in

- **MCP** (moood's MCP server, connected in your client): read worlds and scenes, make them, edit a scene in
  an editing session (`begin_editing`, `draft`, `commit`) — whoever watches the scene on the site
  sees every draft live. Good for a quick change of one scene.
- **The world's git repository**: everything else — many files, many scenes, anything long, shared
  code. `world_repo(address)` (an MCP tool) gives you the clone URL and a token for that repository
  alone (an hour; ask again for another).

Both lead to the same place: a world lives in its repository. Each deployment of moood has a
branch of its own there — `env/dev` is moood's own, `env/<computer>/<stack>` a stack on someone's computer;
there is no `main`. The deployment's branch is the world, for that deployment: every change of a
scene there is a new version of it; nothing is ever lost. `world_repo` says which branch is the
deployment's you work with (`branch`). A deployment's branch follows another's — usually moood's
own — while it has nothing of its own: it moves on with it, and once all it had is there too
(merged, squashed — however), it goes on following. What it has of its own reaches another
deployment only when you merge it there; so agents on different deployments see each other's work
through the one they follow.

## The repository

    world.json                         title, description, themes, main (the main scene's folder)
    scenes/<folder>/scene.json         a scene: its spec without its code
    scenes/<folder>/elements/<id>.js   an element's code          (objects[].code in the spec)
    scenes/<folder>/elements/<id>.script.js   an element's script (objects[].script)
    scenes/<folder>/elements/<id>.animate.js  what moves it each frame (objects[].animate — engine 2)
    scenes/<folder>/elements/<id>.glsl  its own shader            (objects[].shader — engine 2)
    scenes/<folder>/script.js          the scene's script          (spec.script)
    scenes/<folder>/showcase.js        its own scenario outside games (spec.showcase — engine 2)
    scenes/<folder>/lib.js             code its elements share      (spec.lib)
    scenes/<folder>/relief.js          the land's shape            (spec.relief)
    scenes/<folder>/widgets/<id>.html  a widget's markup           (widgets[].html)
    scenes/<folder>/notes.md           the scene's notes            (spec.notes)
    modules/<name>.js                  a module: code the world's scenes share
    modules/<name>.data.json           its data, if it has any (not formatted)
    modules/<name>.md                  what it offers and how to call it

A file belongs to the element or widget of `scene.json` with that `id`; a file nothing in
`scene.json` names is an error. An element's code is the body of its draw function, as
`scene-format.md` describes. A new folder is a new scene. The repository's name is
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

1. `world_repo(address)` → clone with the git arguments it gives (the token travels in a header —
   never put it into a URL or a credential store).
2. Make a branch of your own, begun from the deployment's (`branch`). **Never push to a
   deployment's branch (`env/…`).**
3. Change files; commit and push your branch as often as you like.
4. `look_at_branch(address, branch, folder)` shows a scene of your branch rendered, with the errors
   its code throws (moments, sheets, films: Looking at your work, below). Look at your work: does it
   show what was asked, and is it beautiful?
5. `merge_branch(address, branch, note)` puts the branch into the world: each scene it changed gets
   a new version, a new folder a new scene. It is refused — nothing changes — if the branch changes
   a scene the person may not change, leaves a folder that isn't a scene, or conflicts with the
   deployment's branch (merge that into your branch, push, try again). The branch is deleted once
   merged. To bring work of one deployment into another, push a branch begun from the other one's
   and merge it there — through that deployment's MCP.

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
the game's next version). For more: `game_repo` (clone it, a branch of your own) and
`merge_game_branch` — refused if the branch changes anything but `game.json` and its type's folder,
or if `game.json` is no longer this game (its type changed, a world or a scene it names isn't there).
Only a game's author changes it.

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
  frames as PNGs in a ZIP. Both are made in the background: you get a link at once, kept an hour,
  that answers 202 while it is made and then the file.

`on` says where it is drawn: `"server"` (the default — WebGL on the server's CPU: seconds a frame,
and a film there is short, 120 frames at most) or `"gpu"` (a GPU, for heavy scenes and long films —
up to 3000 frames). The GPU sleeps after 15 minutes unused: a call then wakes it and says so — ask
again in a minute or two (a film or a pack waits for it by itself). Nothing falls back from one to
the other.

`engine: "<branch>"` — drawn by the engine a pushed branch of moood's own repository has (its `moood/web`), not the
site's: for the engine's own developers — commit and push the engine change, name the branch; on either `on`.

## Rendering on your own machine

`npm i -g @moood/render`, then `moood-render <world>/scenes/<folder> --at 0,2.5` — a scene of a
cloned world rendered with moood's own engine (the version the site runs) in a Chromium you have,
without the site: a PNG for each moment and the errors its code threw. Its README tells the rest.
It draws WebGL in software: a single frame is quick, a long film is better made with `film` above.
(`--server` draws on moood's GPU with an engine sent from a checkout of moood itself — for the engine's own
developers, as the tools above do with `engine`; a world's scenes are drawn by the site's engine.)

## What a scene is

`scene-format.md` — the scene's language and its craft: the schema, the elements, how they are
painted and moved, state and the director (actions, sequences, events), the showcase, the camera,
light. It is the text moood's own artist works from: where it says to answer with a JSON object, you
write the files instead. The MCP tool `scene_format` returns the same text.

Two engines run scenes. Every new scene is engine 2's (`"engine": 2` in its `scene.json`): painted
once per state, everything that moves moved on the GPU, driven by events — `scene-format.md`. A scene
made before, without `"engine": 2`, runs on the old engine in its own older language —
`scene-format-1.md` (`scene_format(engine=1)`) — until it is rewritten for engine 2 (then all of it:
its paintings static, its motion by layers, animate and the director). How to move one: `engine2-porting.md`.
