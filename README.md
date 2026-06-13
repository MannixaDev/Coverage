# COVERAGE

A single-file browser game: a CCTV installation simulator with a Powerwash Simulator feel. You survey a site, mount and aim cameras to hit a coverage target, commission the system through an alarm receiving centre (ARC) portal, then phone the ARC and prove every signalling camera on a walk test.

Built with Three.js r128 from cdnjs. No build step, no bundler, no dependencies. Open `index.html` in a browser or host it on GitHub Pages.

## Running it

Open `index.html` directly, or serve the folder and visit it:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

Desktop only for now. Click to start, which locks the mouse pointer. Esc releases it and brings the menu back.

## The job, in three stages

A tracker at the top of the screen shows which stage you're on.

### 1. Install

Reach the coverage target (80%) inside and out, within the budget. Cameras mount on walls (2m or higher) or ceilings depending on type. Three zones have specific requirements:

- **Gate** needs an ANPR camera.
- **Refuse store** and the **flammable goods area** need thermal cameras.

Coverage dots show the picture: green is seen, red is a blind spot, amber means a required zone is covered but by the wrong camera type.

### 2. Commission

- Take the **laptop** from slot 7 (free) and place it on any flat surface. Press Q on it to open the **ARC portal**. The portal refuses a schedule until the survey passes.
- Lodge the signalling schedule: site name, external IP, open port, and a per-camera table where you name each channel and set its alarm action. ANPR channels must be view only; thermal channels must signal thermal. The external IP and port live on the **router** by the NVR desk (press Q to read them).
- A successful schedule issues your SMTP connection pack and the ARC phone number. Enter the SMTP details into the **NVR** network settings (press Q at the desk, then use the network form) and run the connection test.
- Submitting the schedule starts a provisioning timer. This is the ARC setting things up on their side, so there is a wait. Pass the time at the van: sit in it and scroll the feed (time runs faster while you do), take a smoke break round the back, or eat your lunch.

### 3. Walk test

Once the site is provisioned and the NVR is connected, press C to phone the ARC. The operator works through your signalling cameras in a random order, calling each one by the name you gave it. Stand in that camera's view, with clear line of sight, for two seconds to register the activation. Prove them all to finish the job.

Changing the camera set after you have submitted the schedule flags the configuration as changed, drops any active walk test, and forces a resubmit on the portal.

## Controls

| Key | Action |
| --- | --- |
| WASD | Move |
| Mouse | Look |
| 1 - 7 | Select camera type or the laptop |
| Scroll | Pan the camera's aim before placing |
| Shift + Scroll | Tilt the camera's aim |
| Left click | Place on the highlighted surface |
| E | Remove the camera you're looking at, or pick the laptop back up |
| Q | Interact (NVR, router, laptop portal, van, smoke break) |
| C | Call the ARC for the walk test, when the site is ready |
| F | Toggle field-of-view cones |
| G | Toggle coverage dots |
| R | Toggle the roof |
| Esc | Menu, or close an open device panel |

In the NVR live view: A / D cycle cameras, Q exits. In the van: scroll the feed, L eats lunch once, Q gets out.

## Project layout

Everything is in `index.html`. The inline script is organised top to bottom as:

- `LEVEL` — all level data (building boxes, requirement zones, coverage sample regions, spawn, budget, and the NVR, router, and van positions). The engine only reads this; a new level is a new data object.
- `CAM_TYPES` — the camera catalogue and the laptop item.
- World build, player controller, and pointer lock.
- Placement: ghost preview, surface-normal mount detection, pan and tilt aim.
- Coverage solver, with `camSees()` as the shared visibility test reused by the walk test.
- The three-stage flow: stage tracker, the portal / router / NVR device UIs, the provisioning timer and van activities, and the walk test call.

## Editing notes

A JavaScript error renders a blank page with no warning, so syntax-check the inline script after any change:

```bash
sed -n '/^<script>$/,/^<\/script>$/p' index.html | sed '1d;$d' > /tmp/game.js && node --check /tmp/game.js && echo OK
```

Constraints worth keeping in mind: Three.js r128 only (no `OrbitControls`, no `CapsuleGeometry`), primitive geometry only, shared materials and cached geometry, and coverage recomputed only when the camera set changes rather than every frame.
