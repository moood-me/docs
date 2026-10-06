# Runway Show — WebGL fashion show (three.js r165)

Sunset desert runway, 14 models walking in a loop with automatic cinematic cameras (drag to orbit; returns to auto after 12s idle).

## Run
Browsers block ES modules and model loading from file://, so serve the folder:
  npx serve .        (or)   python3 -m http.server
then open http://localhost:3000 (or :8000).
Add #lineup to the URL to see every model standing in a row (handy for checking outfits).

## Structure
- index.html — everything: scene, lighting, garments, walk logic, cameras (one <script type="module">)
- models/ — Xbot.glb (source of the walk/idle animations), Michelle.glb (base body for the 7 procedural outfits), 7 Mixamo characters
- vendor/ — three.js r165 + addons, imports rewritten to "three" / "three/addons/..." (resolved by the importmap in index.html)

## Code map (index.html)
- Sky / lights / venue / ground / post-processing — top of the script
- Garment toolkit: Rig, bodyCloud, fitRing, chainTube (skinned shrink-wrap tubes), skirt (upright cloth with leg-push shader), shoe, hood, ruff
- LOOKS — the 7 outfits built on Michelle (gown, suit, streetwear, coat, avant-garde, minimal, sport)
- retarget() — copies Xbot's walk/idle onto any Mixamo skeleton
- PEOPLE — list of uploaded characters + target heights (in the load block near the bottom)
- Actor — per-model state machine (out → pose → back), catwalk leg tweak, skirt physics
- shots / shotEval — the 9 camera shots
