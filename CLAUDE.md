# Rescatando a Espert - Construct 3 Game

## Project Structure
- All game logic lives in JSON event sheets under `eventSheets/`
- `_Game.json` is the main event sheet (~4500 lines)
- Object types defined in `objectTypes/*.json`
- Layout instances in `layouts/*.json`
- `layouts/ObjectRepository.json` holds default instances for template spawning

## Construct 3 JSON Conventions
- Event sheets use tabs for indentation — edits MUST match exact tab indentation
- `create-object` with `template-name: ""` creates from first project instance (ObjectRepository)
- `compare-boolean-eventvar` is a valid condition ID for boolean event variables
- SIDs must be unique across the project — never reuse an existing SID

## Shape3D Critical Rule
- Shape3D objects MUST have exactly 6 animation frames (one per box face: back, front, left, right, top, bottom)
- `_DrawFace` uses face index as frame index — fewer than 6 frames causes `RangeError: invalid frame`
- For billboard-style Shape3D: set `face-bottom: true`, all others `false`
- Image files: duplicate frame 0 for frames 1-5 if using same texture on all faces

## Image Files
- Naming: `{objectname_lowercase}-{animationname_lowercase}-{NNN}.png` (zero-padded 3 digits)
- `imageSpriteId` must be unique per frame within an object's animations

## Game Mechanics
- Single-character runner: jump, dodge left/right, collect bolsitas for rainbow meter
- Rainbow mode: activated when meter is full, spawns hittable objects (Koparina/Goomlei)
- No 3-form system (Wolf/Bird/Lizard portals removed)

## Audio
- Audio play action uses `audio-file.path`, `loop`, `volume`, `stereo-pan`, `tag-optional`
- Sound files in `sounds/`, music in `music/` — all `.webm` format
