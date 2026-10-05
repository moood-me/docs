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

Both lead to the same place: a world lives in its repository. Its `main` is the world; every
change of a scene there is a new version of it; nothing is ever lost.

## The repository

    world.json                         title, description, themes, main (the main scene's folder)
    scenes/<folder>/scene.json         a scene: its spec without its code
    scenes/<folder>/elements/<id>.js   an element's code          (objects[].code in the spec)
    scenes/<folder>/elements/<id>.script.js   an element's script (objects[].script)
    scenes/<folder>/script.js          the scene's script          (spec.script)
    scenes/<folder>/lib.js             code its elements share      (spec.lib)
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

Every scene shows the modules as `main` has them now: a change of a module reaches every scene that
uses it, at once — it is a new version of the world, not of those scenes (a scene's versions are
the changes of its own folder; an old version opens with the modules as they were then). So before
you merge a change of a module, look at the scenes that use it (`look_at_branch`), not only at the
one you made it for. A merge is refused if a scene lists a module that isn't there, or a changed
module isn't valid JavaScript.

## Working in it

1. `world_repo(address)` → clone with the git arguments it gives (the token travels in a header —
   never put it into a URL or a credential store).
2. Make a branch of your own. **Never push to `main`.**
3. Change files; commit and push your branch as often as you like.
4. `look_at_branch(address, branch, folder)` shows a scene of your branch rendered, with the errors
   its code throws. Look at your work: does it show what was asked, and is it beautiful?
5. `merge_branch(address, branch, note)` puts the branch into the world: each scene it changed gets
   a new version, a new folder a new scene. It is refused — nothing changes — if the branch changes
   a scene the person may not change, leaves a folder that isn't a scene, or conflicts with `main`
   (merge `main` into your branch, push, try again). The branch is deleted once merged.

moood formats the code it writes (Biome: 2 spaces, 120 columns); write your code readably too, one
statement a line.

## Games

A game played on worlds has a repository of its own, `<space>--game--<slug>`: `game.json` (its
type, title, description, genres, the world it is played on — that world's repository — and the
scenes it takes in, `{world, folder, version}`) and a folder of its type (`moood/agents.json`, what
the agents are told; `asked/story.json`, the story) that only that game type reads. Its versions are
the commits of its `main`; a play keeps the game's version and the world's it began on. Games are
changed through moood (the site, the game types) — not through the MCP, which is for worlds.

## What a scene is

`scene-format.md` — the scene's language and its craft: the schema, the elements, how they are
drawn and animated, state, the camera, light. It is the text moood's own artist works from: where
it says to answer with a JSON object, you write the files instead. The MCP tool `scene_format`
returns the same text.
