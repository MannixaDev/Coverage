# COVERAGE — Claude Code project brief

A single-file browser game: a CCTV installation simulator with a Powerwash Simulator feel. The player surveys a site, mounts and aims cameras to hit a coverage target, commissions the system through an alarm receiving centre (ARC) portal, then phones the ARC and proves every signalling camera on a walk test.

Read `README.md` for the gameplay model and controls. This file is the engineering brief.

## Hard constraints

- Everything lives in `index.html`. No build step, no bundler, no npm, no separate JS/CSS files. Keep it a single self-contained file unless I explicitly ask to split it.
- Output filename stays `index.html` (hosted on GitHub Pages).
- Three.js r128 only, loaded from `https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js`. No other libraries.
- r128 quirks to respect: no `OrbitControls`, no `CapsuleGeometry` (use cylinders/spheres), cone geometry is apex-at-origin.
- Primitive geometry only (boxes, cylinders, spheres, planes). No imported models or textures.
- Shared materials and cached geometry where possible. Target 60fps on desktop. Coverage is recomputed only when the camera set changes (place/remove), never per frame.
- No localStorage or sessionStorage.

## Verifying a change

A JavaScript error renders a blank page with no warning, so syntax-check the inline script after any edit:

```bash
sed -n '/^<script>$/,/^<\/script>$/p' index.html | sed '1d;$d' > /tmp/game.js && node --check /tmp/game.js && echo OK
```

There are two `<script>` tags. The CDN one is `<script src=...>` on a single line; the game is `<script>` on its own line, so the sed range catches the right block. WebGL can't run headless here, so reason through the Three.js math rather than relying on a runtime.

## Architecture (top to bottom in the inline script)

- `LEVEL` — all level data: building boxes, requirement zones, coverage sample regions, spawn, budget, and the NVR, router, and van positions. The engine only reads this; a new level is a new data object.
- `CAM_TYPES` — camera catalogue (fov, range, cost, mount surfaces) plus the laptop item in slot 7.
- World build reads `LEVEL.boxes` into `colliders` / `occluders` / `placeSurfaces`.
- Placement: raycast from the crosshair, surface normal decides wall vs ceiling, ghost preview, scroll = pan, shift+scroll = tilt.
- Coverage solver: a grid of sample points; a point is covered if it sits inside a camera's FOV and range with a clear occlusion raycast. `camSees()` is the shared visibility test, reused by the walk test.
- Three stages: INSTALL (coverage + zones) -> COMMISSION (laptop ARC portal schedule, router for WAN IP/port, NVR SMTP settings, provisioning timer with van/smoke/lunch time-skips) -> WALK TEST (phone the ARC, stand in each signalling camera's view). `configDirty` forces a resubmit if cameras change after submission.
- Device UIs (portal/router/NVR) are fullscreen DOM overlays. `uiOpen` gates pointer-lock and input. The van and the NVR live-feed view are separate modes.

## Working style

- State the planned change and let me approve it before editing the file. Keep it short and direct.
- No em dashes in prose or comments. Avoid AI-tell phrases ("that said", "overall", "it's worth noting").
- Decisive iteration over hedging.

## Known soft spot

Pointer-lock transitions between the DOM device panels and the game can occasionally surface a browser click-to-resume overlay (lock cooldown). A candidate for smoothing.
