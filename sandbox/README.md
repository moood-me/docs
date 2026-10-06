# Sandbox: examples to take ideas and pieces from

Working things made outside moood that show how something can be done. Not part of moood and not
its format: read them for the ideas, take what fits a scene, rewrite it the way the scene format
asks (scene-format.md).

- [runway-show/](runway-show/) — a fashion show in WebGL (three.js r165): a sunset desert runway,
  models walking in a loop, automatic cinematic cameras. What to look at: a walk and an idle taken
  from one skeleton onto any other (`retarget()`), garments built over a skinned body (shrink-wrapped
  tubes, a skirt whose cloth the legs push, shoes, a hood, a ruff), seven outfits on one base body,
  a sky, lights and bloom, nine camera shots and how one is chosen. `models/` — GLB people (Mixamo,
  converted from FBX). Run: `python -m http.server` in the folder, open it; `#lineup` stands
  everyone in a row.
