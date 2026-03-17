# Rainbow Mode Hittable Objects — Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace normal obstacles with hittable Koparina/Goomlei objects during rainbow flash mode that fly off satisfyingly when the player collides with them.

**Architecture:** Convert existing Koparina/Goomlei Sprites to Shape3D billboards, create a new StarParticle3D Shape3D object for impact particles. Add conditional spawn logic in `spawnEntities()`, mode transition cleanup, collision handlers with Tween knockback animations, and particle/sound effects. All changes are JSON edits to Construct 3 project files.

**Tech Stack:** Construct 3 (JSON event sheets, Shape3D plugin, Tween behavior)

**Spec:** `docs/superpowers/specs/2026-03-17-rainbow-hittable-objects-design.md`

**Important — SID fields:** Every action, condition, and event block in Construct 3 JSON requires a unique `sid` (numeric identifier). All JSON snippets in this plan include SIDs where shown. For any action/condition that doesn't explicitly show a `sid`, the implementer MUST generate a unique numeric SID (e.g., `330188751978XXX` pattern) before inserting into the event sheet. Construct 3 will reject files with missing or duplicate SIDs.

---

## File Map

| File | Action | Purpose |
|------|--------|---------|
| `objectTypes/Koparina.json` | Rewrite | Convert from Sprite to Shape3D with Tween behavior |
| `objectTypes/Goomlei.json` | Rewrite | Convert from Sprite to Shape3D with Tween behavior |
| `objectTypes/StarParticle3D.json` | Create | New Shape3D billboard for impact star particles |
| `layouts/ObjectRepository.json` | Modify | Add instances of Koparina, Goomlei, StarParticle3D with Shape3D properties |
| `project.c3proj` | Modify | Register StarParticle3D object type (Koparina/Goomlei already registered) |
| `eventSheets/_Game.json` | Modify | Spawn logic, movement, destruction, collision, knockback, particles, sound |

---

### Task 1: Create Feature Branch

- [ ] **Step 1: Create and switch to new branch**

```bash
cd "F:/WORK STUFF/Proyectos de Juegos/Construct/2026/GameTestWithClaude/Rescatando a Espert"
git checkout -b feature/rainbow-hittable-objects
```

- [ ] **Step 2: Commit — branch created**

No files changed yet, branch is ready.

---

### Task 2: Convert Koparina from Sprite to Shape3D

**Files:**
- Rewrite: `objectTypes/Koparina.json`
- Modify: `layouts/ObjectRepository.json` (add Shape3D instance)

- [ ] **Step 1: Rewrite Koparina.json as Shape3D**

Replace the entire file content. Keep the same SID (`455849124037289`) and image references. Change `plugin-id` from `"Sprite"` to `"Shape3D"`, add Tween behavior.

```json
{
	"name": "Koparina",
	"plugin-id": "Shape3D",
	"sid": 455849124037289,
	"isGlobal": false,
	"editorNewInstanceIsReplica": true,
	"instanceVariables": [],
	"behaviorTypes": [
		{
			"behaviorId": "Tween",
			"name": "Tween",
			"sid": 455849124038001
		}
	],
	"effectTypes": [],
	"animations": {
		"items": [
			{
				"frames": [
					{
						"width": 68,
						"height": 64,
						"originX": 0.5,
						"originY": 0.5,
						"originalSource": "",
						"exportFormat": "lossless",
						"exportQuality": 0.8,
						"fileType": "image/png",
						"imageSpriteId": 248127,
						"useCollisionPoly": true,
						"duration": 1,
						"tag": ""
					}
				],
				"sid": 269368620469769,
				"name": "Default",
				"isLooping": false,
				"isPingPong": false,
				"repeatCount": 1,
				"repeatTo": 0,
				"speed": 0
			}
		],
		"subfolders": []
	}
}
```

- [ ] **Step 2: Add Koparina instance to ObjectRepository.json**

In `ObjectRepository.json`, find the layer named `"Layer 0"` and add a new instance entry in its `"instances"` array. Use the billboard pattern (only `face-bottom: true`, all other faces false):

```json
{
	"type": "Koparina",
	"properties": {
		"shape": "box",
		"z-origin": "back",
		"initially-visible": true,
		"face-back": false,
		"face-front": false,
		"face-left": false,
		"face-right": false,
		"face-top": false,
		"face-bottom": true,
		"back-face-culling": false,
		"z-tiling-factor": 8,
		"object-back": -1,
		"object-front": -1,
		"object-left": -1,
		"object-right": -1,
		"object-top": -1,
		"object-bottom": -1
	},
	"uid": 115,
	"sid": 455849124039001,
	"tags": "",
	"instanceVariables": {},
	"behaviors": {
		"Tween": {
			"properties": {
				"enabled": true
			}
		}
	},
	"showing": true,
	"locked": false,
	"world": {
		"x": 0,
		"y": 0,
		"width": 68,
		"height": 4,
		"originX": 0.5,
		"originY": 0.5,
		"color": [1, 1, 1, 1],
		"z": 0,
		"depth": 64,
		"angle": 0
	}
}
```

**Note:** The `uid` must be unique across all layout instances. The highest existing UID in ObjectRepository is 114, so start from 115. Same for `sid` — must be globally unique. The `height` is small (4) with `depth` matching the sprite height (64) — this is the billboard pattern used by TreeBillboard/CoinBillboard where the "height" in world space is thin and the 3D depth shows the texture.

- [ ] **Step 3: Commit**

```bash
git add objectTypes/Koparina.json layouts/ObjectRepository.json
git commit -m "feat: convert Koparina from Sprite to Shape3D with Tween"
```

---

### Task 3: Convert Goomlei from Sprite to Shape3D

**Files:**
- Rewrite: `objectTypes/Goomlei.json`
- Modify: `layouts/ObjectRepository.json` (add Shape3D instance)

- [ ] **Step 1: Rewrite Goomlei.json as Shape3D**

Same pattern as Koparina. Keep SID (`607391674835390`) and image references.

```json
{
	"name": "Goomlei",
	"plugin-id": "Shape3D",
	"sid": 607391674835390,
	"isGlobal": false,
	"editorNewInstanceIsReplica": true,
	"instanceVariables": [],
	"behaviorTypes": [
		{
			"behaviorId": "Tween",
			"name": "Tween",
			"sid": 607391674836001
		}
	],
	"effectTypes": [],
	"animations": {
		"items": [
			{
				"frames": [
					{
						"width": 46,
						"height": 44,
						"originX": 0.5,
						"originY": 0.5,
						"originalSource": "",
						"exportFormat": "lossless",
						"exportQuality": 0.8,
						"fileType": "image/png",
						"imageSpriteId": 6330687,
						"useCollisionPoly": true,
						"duration": 1,
						"tag": ""
					}
				],
				"sid": 816025923863203,
				"name": "Default",
				"isLooping": false,
				"isPingPong": false,
				"repeatCount": 1,
				"repeatTo": 0,
				"speed": 0
			}
		],
		"subfolders": []
	}
}
```

- [ ] **Step 2: Add Goomlei instance to ObjectRepository.json**

Same billboard pattern as Koparina in `"Layer 0"`, with Goomlei dimensions (46x44):

```json
{
	"type": "Goomlei",
	"properties": {
		"shape": "box",
		"z-origin": "back",
		"initially-visible": true,
		"face-back": false,
		"face-front": false,
		"face-left": false,
		"face-right": false,
		"face-top": false,
		"face-bottom": true,
		"back-face-culling": false,
		"z-tiling-factor": 8,
		"object-back": -1,
		"object-front": -1,
		"object-left": -1,
		"object-right": -1,
		"object-top": -1,
		"object-bottom": -1
	},
	"uid": 116,
	"sid": 607391674837001,
	"tags": "",
	"instanceVariables": {},
	"behaviors": {
		"Tween": {
			"properties": {
				"enabled": true
			}
		}
	},
	"showing": true,
	"locked": false,
	"world": {
		"x": 0,
		"y": 70,
		"width": 46,
		"height": 4,
		"originX": 0.5,
		"originY": 0.5,
		"color": [1, 1, 1, 1],
		"z": 0,
		"depth": 44,
		"angle": 0
	}
}
```

- [ ] **Step 3: Commit**

```bash
git add objectTypes/Goomlei.json layouts/ObjectRepository.json
git commit -m "feat: convert Goomlei from Sprite to Shape3D with Tween"
```

---

### Task 4: Create StarParticle3D Object

**Files:**
- Create: `objectTypes/StarParticle3D.json`
- Modify: `layouts/ObjectRepository.json` (add instance)
- Modify: `project.c3proj` (register new object type)

- [ ] **Step 1: Create StarParticle3D.json**

New Shape3D billboard using the ParticleStarTex "Star" animation frame image (imageSpriteId `7019782`, 11x12px). We need to copy the image file for this new object type — or reference the same imageSpriteId. Since Construct 3 allows different objects to reference the same sprite sheet images, we use a new imageSpriteId and copy the image.

**Simpler approach:** Create the object type JSON with a new unique imageSpriteId, then copy the star image file with the new naming convention.

```json
{
	"name": "StarParticle3D",
	"plugin-id": "Shape3D",
	"sid": 330188751969001,
	"isGlobal": false,
	"editorNewInstanceIsReplica": true,
	"instanceVariables": [],
	"behaviorTypes": [
		{
			"behaviorId": "Tween",
			"name": "Tween",
			"sid": 330188751969002
		}
	],
	"effectTypes": [],
	"animations": {
		"items": [
			{
				"frames": [
					{
						"width": 16,
						"height": 16,
						"originX": 0.5,
						"originY": 0.5,
						"originalSource": "",
						"exportFormat": "lossless",
						"exportQuality": 0.8,
						"fileType": "image/png",
						"imageSpriteId": 3301887,
						"useCollisionPoly": true,
						"duration": 1,
						"tag": ""
					}
				],
				"sid": 330188751969003,
				"name": "Star",
				"isLooping": false,
				"isPingPong": false,
				"repeatCount": 1,
				"repeatTo": 0,
				"speed": 0
			}
		],
		"subfolders": []
	}
}
```

- [ ] **Step 2: Copy star texture image**

```bash
cp "F:/WORK STUFF/Proyectos de Juegos/Construct/2026/GameTestWithClaude/Rescatando a Espert/images/particlestartex-star-000.png" "F:/WORK STUFF/Proyectos de Juegos/Construct/2026/GameTestWithClaude/Rescatando a Espert/images/starparticle3d-star-000.png"
```

The image filename must follow the Construct 3 convention: `<objectname-lowercase>-<animationname-lowercase>-<framenumber>.png`.

- [ ] **Step 3: Add StarParticle3D instance to ObjectRepository.json**

```json
{
	"type": "StarParticle3D",
	"properties": {
		"shape": "box",
		"z-origin": "back",
		"initially-visible": true,
		"face-back": false,
		"face-front": false,
		"face-left": false,
		"face-right": false,
		"face-top": false,
		"face-bottom": true,
		"back-face-culling": false,
		"z-tiling-factor": 8,
		"object-back": -1,
		"object-front": -1,
		"object-left": -1,
		"object-right": -1,
		"object-top": -1,
		"object-bottom": -1
	},
	"uid": 117,
	"sid": 330188751969004,
	"tags": "",
	"instanceVariables": {},
	"behaviors": {
		"Tween": {
			"properties": {
				"enabled": true
			}
		}
	},
	"showing": true,
	"locked": false,
	"world": {
		"x": 0,
		"y": 120,
		"width": 16,
		"height": 4,
		"originX": 0.5,
		"originY": 0.5,
		"color": [1, 1, 1, 1],
		"z": 0,
		"depth": 16,
		"angle": 0
	}
}
```

- [ ] **Step 4: Register StarParticle3D in project.c3proj**

Add `"StarParticle3D"` to the `objectTypes.items` array in `project.c3proj`, alongside the other object types.

- [ ] **Step 5: Commit**

```bash
git add objectTypes/StarParticle3D.json images/starparticle3d-star-000.png layouts/ObjectRepository.json project.c3proj
git commit -m "feat: create StarParticle3D Shape3D object for impact particles"
```

---

### Task 5: Add Movement and Destruction Logic for Koparina/Goomlei

**Files:**
- Modify: `eventSheets/_Game.json`

- [ ] **Step 1: Add movement actions for Koparina and Goomlei**

In `_Game.json`, find the "Move obstacles" comment block (around line 970) where Rock, RockBig, and Wood have their Y positions updated. Add two new actions in the same event block:

```json
{
	"id": "set-y",
	"objectClass": "Koparina",
	"sid": 330188751974001,
	"parameters": {
		"y": "Self.Y + gameSpeed * dt"
	}
},
{
	"id": "set-y",
	"objectClass": "Goomlei",
	"sid": 330188751974002,
	"parameters": {
		"y": "Self.Y + gameSpeed * dt"
	}
}
```

- [ ] **Step 2: Add destruction events for Koparina and Goomlei**

Find the "Destroy entities" section (around line 1135) and add two new event blocks after the existing Rock/RockBig/Wood/CoinBillboard destroy blocks:

```json
{
	"eventType": "block",
	"conditions": [
		{
			"id": "compare-y",
			"objectClass": "Koparina",
			"parameters": {
				"comparison": 4,
				"y-co-ordinate": "384"
			}
		}
	],
	"actions": [
		{
			"id": "destroy",
			"objectClass": "Koparina",
			"sid": 330188751974003
		}
	],
	"sid": 330188751970001
},
{
	"eventType": "block",
	"conditions": [
		{
			"id": "compare-y",
			"objectClass": "Goomlei",
			"sid": 330188751974005,
			"parameters": {
				"comparison": 4,
				"y-co-ordinate": "384"
			}
		}
	],
	"actions": [
		{
			"id": "destroy",
			"objectClass": "Goomlei",
			"sid": 330188751974004
		}
	],
	"sid": 330188751970002
}
```

- [ ] **Step 3: Commit**

```bash
git add eventSheets/_Game.json
git commit -m "feat: add movement and destruction logic for Koparina and Goomlei"
```

---

### Task 6: Modify spawnEntities() for Rainbow Mode

**Files:**
- Modify: `eventSheets/_Game.json`

- [ ] **Step 1: Add RainbowMode conditional branch in spawnEntities()**

In the `spawnEntities()` function (around line 2139), add a new conditional block **before** the existing obstacle spawn logic. When `RainbowMode == true`, spawn Koparina/Goomlei instead.

Add a new local variable to the function for rainbow spawn type:

```json
{
	"eventType": "variable",
	"name": "rainbowSpawnRow",
	"type": "number",
	"initialValue": "0",
	"comment": "Whether to spawn a row of rainbow objects (0=single, 1=row)"
}
```

Then add the rainbow spawn block. This should be a top-level condition that, when true, skips the normal obstacle spawning via `else`:

```json
{
	"eventType": "comment",
	"text": "Spawn rainbow hittable objects when in rainbow mode."
},
{
	"eventType": "block",
	"conditions": [
		{
			"id": "compare-boolean-eventvar",
			"objectClass": "System",
			"sid": 330188751971009,
			"parameters": {
				"variable": "RainbowMode"
			}
		}
	],
	"actions": [
		{
			"id": "set-eventvar-value",
			"objectClass": "System",
			"sid": 330188751977001,
			"parameters": {
				"variable": "rainbowSpawnRow",
				"value": "floor(random(5)) = 0"
			}
		}
	],
	"children": [
		{
			"eventType": "comment",
			"text": "Single spawn: one random Koparina or Goomlei."
		},
		{
			"eventType": "block",
			"conditions": [
				{
					"id": "compare-eventvar",
					"objectClass": "System",
					"parameters": {
						"variable": "rainbowSpawnRow",
						"comparison": 0,
						"value": "0"
					}
				}
			],
			"children": [
				{
					"eventType": "block",
					"conditions": [
						{
							"id": "evaluate-expression",
							"objectClass": "System",
							"parameters": {
								"value": "floor(random(2)) = 0"
							}
						}
					],
					"actions": [
						{
							"id": "create-object",
							"objectClass": "System",
							"sid": 330188751977002,
							"parameters": {
								"object-to-create": "Koparina",
								"layer": "\"Game\"",
								"x": "choose(26, 58, 90, 122, 154)",
								"y": "-WPNDIST",
								"create-hierarchy": false,
								"template-name": "\"\""
							}
						}
					],
					"sid": 330188751971001
				},
				{
					"eventType": "block",
					"conditions": [
						{
							"id": "else",
							"objectClass": "System"
						}
					],
					"actions": [
						{
							"id": "create-object",
							"objectClass": "System",
							"parameters": {
								"object-to-create": "Goomlei",
								"layer": "\"Game\"",
								"x": "choose(26, 58, 90, 122, 154)",
								"y": "-WPNDIST",
								"create-hierarchy": false,
								"template-name": "\"\""
							}
						}
					],
					"sid": 330188751971002
				}
			],
			"sid": 330188751971003
		},
		{
			"eventType": "comment",
			"text": "Row spawn: 3-4 objects spread across the road (~20% chance)."
		},
		{
			"eventType": "block",
			"conditions": [
				{
					"id": "else",
					"objectClass": "System"
				}
			],
			"children": [
{
	"eventType": "block",
	"conditions": [
		{
			"id": "for",
			"objectClass": "System",
			"parameters": {
				"name": "\"row\"",
				"start-index": "0",
				"end-index": "choose(3, 4) - 1"
			}
		}
	],
	"children": [
		{
			"eventType": "block",
			"conditions": [
				{
					"id": "evaluate-expression",
					"objectClass": "System",
					"parameters": {
						"value": "floor(random(2)) = 0"
					}
				}
			],
			"actions": [
				{
					"id": "create-object",
					"objectClass": "System",
					"parameters": {
						"object-to-create": "Koparina",
						"layer": "\"Game\"",
						"x": "26 + loopindex(\"row\") * 45",
						"y": "-WPNDIST",
						"create-hierarchy": false,
						"template-name": "\"\""
					}
				}
			],
			"sid": 330188751971006
		},
		{
			"eventType": "block",
			"conditions": [
				{
					"id": "else",
					"objectClass": "System"
				}
			],
			"actions": [
				{
					"id": "create-object",
					"objectClass": "System",
					"parameters": {
						"object-to-create": "Goomlei",
						"layer": "\"Game\"",
						"x": "26 + loopindex(\"row\") * 45",
						"y": "-WPNDIST",
						"create-hierarchy": false,
						"template-name": "\"\""
					}
				}
			],
			"sid": 330188751971007
		}
	],
	"sid": 330188751971008
}
```

The existing obstacle spawn blocks should be wrapped in an `else` condition so they only run when `RainbowMode == false`.

- [ ] **Step 2: Wrap existing obstacle spawns with else block**

The existing obstacle spawn logic (everything after the rainbow spawn block — the `obstacleType` random assignment, and the three `compare-eventvar obstacleType` blocks that create Rock/Wood/RockBig) must be wrapped in an `else` block that pairs with the `compare-boolean-eventvar RainbowMode` condition from Step 1. Structure:

```json
{
	"eventType": "block",
	"conditions": [
		{
			"id": "else",
			"objectClass": "System",
			"sid": 330188751977010
		}
	],
	"children": [
		// ... move ALL existing obstacle spawn events here:
		// - set obstacleType = floor(random(3))
		// - compare obstacleType == 0 → create Rock
		// - compare obstacleType == 1 → create Wood
		// - else → create RockBig
		// - coin spawn logic
		// - portal spawn logic
	],
	"sid": 330188751977011
}
```

This ensures normal obstacles only spawn when `RainbowMode` is false (the `else` of the `RainbowMode == true` check).

- [ ] **Step 3: Commit**

```bash
git add eventSheets/_Game.json
git commit -m "feat: add rainbow mode conditional spawn for Koparina/Goomlei"
```

---

### Task 7: Add Rainbow Mode Transition Cleanup

**Files:**
- Modify: `eventSheets/_Game.json`

- [ ] **Step 1: Destroy obstacles on rainbow activation**

Find the rainbow activation block (where `rainbowCharge >= 160` and `rainbowState == 0`). Add destroy actions for all normal obstacles at the **beginning** of the actions list (before the visual transition):

```json
{
	"id": "destroy",
	"objectClass": "Rock",
	"sid": 330188751975001
},
{
	"id": "destroy",
	"objectClass": "RockBig",
	"sid": 330188751975002
},
{
	"id": "destroy",
	"objectClass": "Wood",
	"sid": 330188751975003
}
```

- [ ] **Step 2: Destroy hittables on rainbow deactivation**

Find the rainbow deactivation block (where `rainbowCharge <= 0` and `rainbowState == 1`). Add destroy actions for rainbow objects at the **beginning** of the actions list:

```json
{
	"id": "destroy",
	"objectClass": "Koparina",
	"sid": 330188751975004
},
{
	"id": "destroy",
	"objectClass": "Goomlei",
	"sid": 330188751975005
},
{
	"id": "destroy",
	"objectClass": "StarParticle3D",
	"sid": 330188751975006
}
```

- [ ] **Step 3: Commit**

```bash
git add eventSheets/_Game.json
git commit -m "feat: destroy obstacles/hittables on rainbow mode transitions"
```

---

### Task 8: Add Collision Handlers with Knockback Animation

**Files:**
- Modify: `eventSheets/_Game.json`

- [ ] **Step 1: Add collision event group**

Add a new event group in `_Game.json` after the existing collision handlers (around the "Rainbow Flash Logic" group). Create a comment and event blocks for both Koparina and Goomlei collisions.

**Koparina collision:**

```json
{
	"eventType": "comment",
	"text": "Rainbow mode: Koparina knockback on player collision."
},
{
	"eventType": "block",
	"conditions": [
		{
			"id": "on-collision-with-another-object",
			"objectClass": "PlayerCollision",
			"parameters": {
				"object": "Koparina"
			}
		}
	],
	"actions": [
		{
			"id": "play",
			"objectClass": "Audio",
			"sid": 330188751976001,
			"parameters": {
				"file": "CrashA",
				"looping": "not-looping",
				"vol": "-5",
				"stereo-pan": "0",
				"tag": "\"knockback\""
			}
		},
		{
			"id": "tween-two-properties",
			"objectClass": "Koparina",
			"behaviorType": "Tween",
			"sid": 330188751976002,
			"parameters": {
				"tags": "\"KnockbackPos\"",
				"property-x": "x",
				"x-end-value": "Self.X + choose(-1, 1) * 300",
				"property-y": "y",
				"y-end-value": "Self.Y - 400",
				"time": "0.5",
				"ease": "easeoutsine",
				"destroy-on-complete": "after-last-instance",
				"loop": "no",
				"ping-pong": "no",
				"repeat-count": "1"
			}
		},
		{
			"id": "tween-one-property",
			"objectClass": "Koparina",
			"behaviorType": "Tween",
			"sid": 330188751976003,
			"parameters": {
				"tags": "\"KnockbackRot\"",
				"property": "angle",
				"end-value": "Self.Angle + 720",
				"time": "0.5",
				"ease": "easeoutsine",
				"destroy-on-complete": "no",
				"loop": "no",
				"ping-pong": "no",
				"repeat-count": "1"
			}
		}
	],
	"sid": 330188751972001
}
```

**Goomlei collision (same pattern):**

```json
{
	"eventType": "comment",
	"text": "Rainbow mode: Goomlei knockback on player collision."
},
{
	"eventType": "block",
	"conditions": [
		{
			"id": "on-collision-with-another-object",
			"objectClass": "PlayerCollision",
			"parameters": {
				"object": "Goomlei"
			}
		}
	],
	"actions": [
		{
			"id": "play",
			"objectClass": "Audio",
			"sid": 330188751976004,
			"parameters": {
				"file": "CrashA",
				"looping": "not-looping",
				"vol": "-5",
				"stereo-pan": "0",
				"tag": "\"knockback\""
			}
		},
		{
			"id": "tween-two-properties",
			"objectClass": "Goomlei",
			"behaviorType": "Tween",
			"sid": 330188751976005,
			"parameters": {
				"tags": "\"KnockbackPos\"",
				"property-x": "x",
				"x-end-value": "Self.X + choose(-1, 1) * 300",
				"property-y": "y",
				"y-end-value": "Self.Y - 400",
				"time": "0.5",
				"ease": "easeoutsine",
				"destroy-on-complete": "after-last-instance",
				"loop": "no",
				"ping-pong": "no",
				"repeat-count": "1"
			}
		},
		{
			"id": "tween-one-property",
			"objectClass": "Goomlei",
			"behaviorType": "Tween",
			"sid": 330188751976006,
			"parameters": {
				"tags": "\"KnockbackRot\"",
				"property": "angle",
				"end-value": "Self.Angle + 720",
				"time": "0.5",
				"ease": "easeoutsine",
				"destroy-on-complete": "no",
				"loop": "no",
				"ping-pong": "no",
				"repeat-count": "1"
			}
		}
	],
	"sid": 330188751972002
}
```

- [ ] **Step 2: Commit**

```bash
git add eventSheets/_Game.json
git commit -m "feat: add collision handlers with knockback tweens for Koparina/Goomlei"
```

---

### Task 9: Add Star Particle Spawn on Impact

**Files:**
- Modify: `eventSheets/_Game.json`

- [ ] **Step 1: Add star particle spawning to collision handlers**

In both the Koparina and Goomlei collision blocks (from Task 8), add a **child event** (sub-event) with a repeat condition. This creates 4 StarParticle3D instances per collision. Add this as a child block inside each collision event:

```json
{
	"eventType": "block",
	"conditions": [
		{
			"id": "repeat",
			"objectClass": "System",
			"parameters": {
				"count": "4"
			}
		}
	],
	"actions": [
		{
			"id": "create-object",
			"objectClass": "System",
			"sid": 330188751978001,
			"parameters": {
				"object-to-create": "StarParticle3D",
				"layer": "\"Game\"",
				"x": "PlayerCollision.X",
				"y": "PlayerCollision.Y",
				"create-hierarchy": false,
				"template-name": "\"\""
			}
		},
		{
			"id": "tween-two-properties",
			"objectClass": "StarParticle3D",
			"behaviorType": "Tween",
			"sid": 330188751978002,
			"parameters": {
				"tags": "\"StarFly\"",
				"property-x": "x",
				"x-end-value": "Self.X + random(-80, 80)",
				"property-y": "y",
				"y-end-value": "Self.Y + random(-120, -20)",
				"time": "0.4",
				"ease": "easeoutsine",
				"destroy-on-complete": "after-last-instance",
				"loop": "no",
				"ping-pong": "no",
				"repeat-count": "1"
			}
		},
		{
			"id": "tween-one-property",
			"objectClass": "StarParticle3D",
			"behaviorType": "Tween",
			"sid": 330188751978003,
			"parameters": {
				"tags": "\"StarShrink\"",
				"property": "width",
				"end-value": "0",
				"time": "0.4",
				"ease": "easeinsine",
				"destroy-on-complete": "no",
				"loop": "no",
				"ping-pong": "no",
				"repeat-count": "1"
			}
		},
		{
			"id": "tween-one-property",
			"objectClass": "StarParticle3D",
			"behaviorType": "Tween",
			"sid": 330188751978004,
			"parameters": {
				"tags": "\"StarShrinkH\"",
				"property": "height",
				"end-value": "0",
				"time": "0.4",
				"ease": "easeinsine",
				"destroy-on-complete": "no",
				"loop": "no",
				"ping-pong": "no",
				"repeat-count": "1"
			}
		}
	],
	"sid": 330188751973001
}
```

**Implementation note:** The star particle sub-events need to be children of each collision block. The structure is:

```
On PlayerCollision collides with Koparina
  → knockback tweens + sound (existing from Task 8)
  → Repeat 4 times
    → Create StarParticle3D at player position
    → Tween position outward (random spread)
    → Tween width to 0
    → Tween height to 0
```

Duplicate the same repeat block for both Koparina and Goomlei collision events.

- [ ] **Step 2: Commit**

```bash
git add eventSheets/_Game.json
git commit -m "feat: spawn star particles on rainbow object impact"
```

---

### Task 10: Manual Testing in Construct 3

- [ ] **Step 1: Open project in Construct 3 and verify**

Open the `.c3p` project in Construct 3 editor. Check:

1. **ObjectRepository layout:** Koparina, Goomlei, and StarParticle3D appear as Shape3D objects (not Sprites). They should render with their textures on the bottom face.

2. **Preview the Game layout:** Play through until rainbow mode activates:
   - Normal obstacles (Rock/Wood/RockBig) should be destroyed when rainbow activates
   - Koparina/Goomlei should start spawning
   - Row spawns should occasionally appear (3-4 across the road)

3. **Collision test:** Move into a Koparina or Goomlei:
   - Object should fly off diagonally (left or right, upward)
   - Object should spin 2 full rotations
   - 4 star particles should appear and spread outward while shrinking
   - CrashA sound should play
   - Player should NOT take damage

4. **Deactivation test:** Let rainbow charge drain to 0:
   - All Koparina/Goomlei/StarParticle3D should be destroyed
   - Normal obstacles should resume spawning

- [ ] **Step 2: Commit any fixes**

```bash
git add -A
git commit -m "fix: adjust rainbow hittable objects after testing"
```
