# Rainbow Mode Hittable Objects

**Date:** 2026-03-17
**Status:** Approved
**Branch:** feature/rainbow-hittable-objects (to be created from feature/rainbow-flash-system)

## Overview

During rainbow flash mode, normal obstacles (Rock, Wood, RockBig) are replaced by hittable objects (Kooparina and Goomlei) that the player can collide with for a satisfying knockback effect. The objects fly off diagonally when hit, accompanied by star particles and impact sound.

## Goals

- Make rainbow mode feel rewarding and fun by replacing hazards with satisfying hittable objects
- Maintain visual consistency with the existing 3D projection system
- Reuse existing spawn/destroy patterns to keep implementation simple

## New Objects

| Object | Plugin | Size | Behaviors | Purpose |
|--------|--------|------|-----------|---------|
| Kooparina | Shape3D | ~68x64 | Tween | Large hittable rainbow object |
| Goomlei | Shape3D | ~46x44 | Tween | Small hittable rainbow object |
| StarParticle3D | Shape3D | ~16x16 | Tween | Impact star particle (3D billboard) |

### Shape3D Configuration (all three objects)

- Shape: box
- Only `face-bottom: true` (all other faces false)
- Back-face culling: false
- This matches the billboard pattern used by TreeBillboard, CoinBillboard, PlayerBillboard

### Existing Assets

- `koparina-default-000.png` (1.7 KB) — already in project as Sprite, needs conversion to Shape3D
- `goomlei-default-000.png` (798 bytes) — already in project as Sprite, needs conversion to Shape3D
- ParticleStarTex texture — already exists, used as face texture for StarParticle3D

## Spawn Logic

### When Rainbow Mode Activates (rainbowCharge >= 160)

1. Destroy all existing Rock, Wood, RockBig instances on screen
2. Existing rainbow activation logic runs (RainbowMode = true, visual/audio transitions)
3. From this point, `spawnEntities()` spawns Kooparina/Goomlei instead of obstacles

### During Rainbow Mode

- **Standard spawn:** 1 object per wave, randomly chosen between Kooparina and Goomlei
- **Special spawn (occasional):** Row of 3-4 objects crossing the path horizontally
- Objects move downward at `gameSpeed * dt` (same as all other entities)
- Objects destroyed if they pass Y > 384 without being hit

### When Rainbow Mode Deactivates (rainbowCharge <= 0)

1. Destroy all existing Kooparina and Goomlei instances on screen
2. Existing rainbow deactivation logic runs (RainbowMode = false, visual/audio transitions)
3. Normal obstacle spawning resumes

## Collision & Knockback Effect

### Detection

- `PlayerCollision` on collision with Kooparina → trigger knockback
- `PlayerCollision` on collision with Goomlei → trigger knockback
- No damage to player (rainbow mode = invincible to these objects)

### Knockback Animation (per object hit)

1. **Direction:** Random 50/50 left or right
2. **Position Tween:**
   - X: current X ± ~300px (based on chosen direction)
   - Y: current Y - ~400px (upward, off screen)
   - Duration: ~0.5s
   - Ease: ease-out (starts fast, decelerates)
3. **Rotation Tween:**
   - Z angle: ~720 degrees (2 full spins)
   - Duration: ~0.5s
   - Ease: ease-out
4. **On Tween complete:** destroy the object

### Impact Particles (per collision)

1. Spawn 3-5 StarParticle3D instances at the collision point
2. Each star is a Shape3D billboard (face-bottom only) at the player's Z elevation
3. Each star gets:
   - **Position Tween:** random direction outward from impact point (spread pattern)
   - **Scale Tween:** scale from 1 to 0 (shrink to nothing)
   - Duration: ~0.3-0.5s
4. On Tween complete: destroy each star

### Impact Sound

- Play a satisfying impact sound effect on each collision (e.g., a "bonk" or knockback sound)
- One sound per hit, not per particle

## Integration Points

### spawnEntities() Function

The existing spawn function needs a conditional branch:
- If `RainbowMode == true`: spawn Kooparina or Goomlei (random choice)
- If `RainbowMode == false`: spawn Rock/Wood/RockBig (existing behavior)

The special row spawn (3-4 objects) can use a random chance check (e.g., 20-25% of spawns during rainbow mode).

### Rainbow Mode Transitions (existing event groups)

Add to the existing activation block:
- Destroy all Rock, Wood, RockBig instances

Add to the existing deactivation block:
- Destroy all Kooparina, Goomlei instances

### Object Lifecycle

Same pattern as existing obstacles:
- Created at Y = -2560 (above viewport)
- Move down with gameSpeed * dt
- Destroyed at Y > 384 (below viewport) OR on collision knockback complete

## Out of Scope

- Scoring/points for hitting objects (purely satisfying, no gameplay reward)
- Different knockback behaviors per object type (both behave the same)
- Combo systems or chain reactions
- Changes to rainbow charge/drain rates
