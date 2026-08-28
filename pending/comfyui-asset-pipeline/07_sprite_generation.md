# 07 — Tactical Sprite Generation Specification

## Objective

Define the full tactical sprite generation pipeline for Wishes.

This document specifies how approved character artwork becomes a game-ready tactical sprite sheet suitable for Unity tactical battles.

The tactical sprite pipeline must preserve character identity while simplifying the character enough to be readable on a tactical grid.

Primary goal:

```text
Approved Portrait
  -> Approved Full Body
  -> Tactical Sprite Base
  -> Tactical Animation Frames
  -> Tactical Sprite Sheet
  -> Metadata
  -> Human Review
  -> Published Unity-Ready Asset
```

## Core Rule

The tactical sprite sheet is not an independent character design.

It is a simplified tactical representation of the approved character.

The sprite must preserve:

- Hair color.
- Hair silhouette.
- Race/species traits where visible.
- Outfit theme.
- Dominant color palette.
- Major accessories.
- Weapon identity.
- General body type.
- Character silhouette.

Fine facial details are allowed to simplify, but the sprite must still be recognizable as the same character.

## Source Asset Priority

Sprite generation should use approved assets in this order:

```text
1. Approved full_body
2. Approved full_body + approved portrait
3. Approved portrait only
```

Preferred:

```text
full_body -> sprite_base -> sprite_sheet
```

Fallback:

```text
portrait -> sprite_base -> sprite_sheet
```

The fallback path should be marked lower confidence in metadata.

## Required Asset Role

Sprite sheet output role:

```text
tactical_sprite_sheet
```

Recommended intermediate roles:

```text
sprite_base
sprite_turnaround
sprite_animation_test
sprite_atlas
```

These may be added later as official `asset_role` rows.

## Visual Style

The sprite style should be tactical RPG readable, similar in function to Final Fantasy Tactics style sprites, but without copying any protected art style.

Recommended prompt direction:

```text
small tactical RPG character sprite, readable silhouette, simplified fantasy outfit, clean pixel-friendly details, hand-painted game sprite look, consistent colors, transparent background, tactical battle scale
```

Avoid:

```text
high-detail portrait rendering, photorealism, noisy painterly detail, unreadable miniature detail, excessive gradients, overly complex background
```

## Initial Technical Targets

Use these defaults for the first milestone:

```text
Tile Width: 128
Tile Height: 128
Directions: 4
Animations: 6
Sheet Format: PNG
Background: transparent
Metadata Format: JSON
```

Required directions:

```text
front
back
left
right
```

Required animations:

```text
idle
walk
attack
cast
hit
down
```

Recommended future directions:

```text
front_left
front_right
back_left
back_right
```

Recommended future animations:

```text
guard
interact
run
use_item
celebrate
death
dash
jump
ranged_attack
special
```

## Generation Pipeline

### Stage 1 — Validate Source

Before queuing a sprite job:

1. Confirm object exists.
2. Confirm approved portrait exists.
3. Prefer approved full body.
4. Confirm source asset is not stale.
5. Confirm source asset file exists.
6. Confirm source asset is readable.
7. Confirm source asset has role metadata.
8. Confirm target asset role `tactical_sprite_sheet` exists.

If no approved portrait exists, block sprite generation.

### Stage 2 — Generate Sprite Base

The sprite base is a simplified neutral character reference.

Purpose:

```text
Create a tactical-scale version of the approved character in a neutral pose.
```

Recommended source:

```text
approved full_body
```

Prompt instruction:

```text
Create a small tactical RPG character sprite base from the approved full body character art. Preserve the same hairstyle, hair color, outfit colors, race traits, weapon, and overall silhouette. Simplify details for readability at tactical battle scale. Use a neutral standing pose. Transparent background.
```

Negative prompt additions:

```text
different character, different hair, different outfit, different colors, missing weapon, unreadable silhouette, noisy details, photorealistic, background scene, cropped body
```

Output:

```text
sprite_base candidate
```

Recommended canvas:

```text
512 x 512 working image
```

The sprite base should later be downscaled or sliced into 128 x 128 tactical frames.

### Stage 3 — Generate Directional Turnaround

The sprite needs at least four directional views.

First milestone options:

```text
Option A: Generate four separate directional sprite bases.
Option B: Generate a four-view turnaround sheet.
Option C: Generate front only first and defer direction expansion.
```

Preferred first production target:

```text
front, back, left, right
```

Turnaround prompt:

```text
four direction tactical RPG sprite turnaround of the same character, front view, back view, left view, right view, consistent outfit, consistent hair, consistent weapon, transparent background, aligned height, same proportions in every view
```

Review criteria:

- Character height consistent in all views.
- Hair silhouette consistent.
- Outfit colors consistent.
- Weapon appears consistently.
- No direction has a different character identity.
- Feet align to common baseline.

### Stage 4 — Generate Animation Frames

For each direction and animation, generate frame sequences.

Minimum frame counts:

```text
idle: 4
walk: 8
attack: 6
cast: 6
hit: 3
down: 1
```

These are suggested defaults. Metadata must store actual frame counts.

### Stage 5 — Assemble Sprite Sheet

The final sheet should be a deterministic composition of frames.

Preferred layout for first milestone:

```text
Direction groups stacked vertically.
Within each direction, animation rows are stacked.
Frames progress left to right.
```

Example:

```text
front_idle row
front_walk row
front_attack row
front_cast row
front_hit row
front_down row

back_idle row
back_walk row
back_attack row
back_cast row
back_hit row
back_down row

left_idle row
left_walk row
left_attack row
left_cast row
left_hit row
left_down row

right_idle row
right_walk row
right_attack row
right_cast row
right_hit row
right_down row
```

This is verbose but simple for Unity to consume.

Alternative compact layout may be implemented later.

## Animation Definitions

### Idle

Purpose:

Character stands ready.

Requirements:

```text
4 frames
looping
minimal movement
no major pose shift
```

Motion:

- Subtle breathing.
- Slight cloth movement.
- Minor hair movement.

Reject if:

- Character changes identity.
- Weapon changes shape.
- Feet slide excessively.
- Frames flicker.

### Walk

Purpose:

Character moves one tactical unit.

Requirements:

```text
8 frames preferred
looping
clear stepping motion
```

Motion:

- Alternating legs.
- Slight arm movement.
- Weapon remains consistent.
- Feet contact ground convincingly.

Reject if:

- Character appears to float.
- Feet slide badly.
- Outfit changes frame to frame.
- Direction is ambiguous.

### Attack

Purpose:

Basic physical attack.

Requirements:

```text
6 frames preferred
non-looping
anticipation -> strike -> recovery
```

Motion:

- Clear wind-up.
- Clear strike frame.
- Return to neutral.

For weapon users:

- Weapon must be visible.
- Weapon must remain same type.
- Attack direction must be readable.

Reject if:

- Weapon disappears.
- Character changes weapon.
- Impact pose unreadable.
- Limbs malformed.

### Cast

Purpose:

Spell or ability casting.

Requirements:

```text
6 frames preferred
non-looping or short loop
```

Motion:

- Hands, staff, or focus object emphasized.
- Magical energy may appear but should not obscure character.
- Pose distinct from attack.

Reject if:

- Spell effect hides sprite.
- Character identity changes.
- Colors become inconsistent.

### Hit

Purpose:

Damage reaction.

Requirements:

```text
3 frames preferred
non-looping
```

Motion:

- Brief recoil.
- Preserve silhouette.
- Avoid excessive deformation.

Reject if:

- Character collapses unless intended.
- Character identity lost.
- Frame too blurry.

### Down

Purpose:

Defeated or incapacitated state.

Requirements:

```text
1 frame minimum
non-looping
```

Motion:

- Character down on ground.
- Readable as defeated.
- Should fit tile bounds.

Reject if:

- Too large for tile.
- Unrecognizable.
- Severe artifacting.

## Metadata Format

Every published sprite sheet must have metadata.

Recommended file:

```text
metadata.json
```

Recommended shape:

```json
{
  "asset_role": "tactical_sprite_sheet",
  "tile_width": 128,
  "tile_height": 128,
  "sheet_width": 1024,
  "sheet_height": 3072,
  "directions": ["front", "back", "left", "right"],
  "animations": {
    "front": {
      "idle": {
        "row": 0,
        "frame_count": 4,
        "frame_rate": 6,
        "loop": true
      },
      "walk": {
        "row": 1,
        "frame_count": 8,
        "frame_rate": 10,
        "loop": true
      }
    }
  }
}
```

Expanded frame metadata:

```json
{
  "frames": [
    {
      "direction": "front",
      "animation": "idle",
      "frame": 0,
      "x": 0,
      "y": 0,
      "w": 128,
      "h": 128,
      "duration_ms": 120
    }
  ]
}
```

Use explicit frame rectangles so Unity import does not require guessing.

## Unity Runtime Requirements

Unity should consume metadata rather than hard-coded layout assumptions.

Required runtime data:

```text
Sprite Sheet Texture
Metadata JSON
Tile Width
Tile Height
Animation Name
Direction
Frame Rectangles
Frame Duration
Loop Flag
Pivot
Pixels Per Unit
```

Recommended default pivot:

```text
bottom_center
```

This supports tactical tile placement.

## Unity Import Recommendations

When importing into Unity:

```text
Texture Type: Sprite
Sprite Mode: Multiple
Filter Mode: Point or Bilinear depending art style
Compression: None or high quality
Pivot: Bottom Center
Pixels Per Unit: consistent tactical scale
```

If sprites are painterly rather than pixel art, Bilinear may look better. If pixel-art style is selected later, use Point.

## Frame Alignment

All frames in an animation must align to a common baseline.

Rules:

- Feet should remain near the same baseline.
- Character should not jump unless animation requires it.
- Head height should remain consistent.
- Weapon should not shift wildly between frames.
- Directional sets should share scale.

Metadata may include alignment hints:

```json
{
  "pivot": "bottom_center",
  "baseline_y": 118,
  "center_x": 64
}
```

## Transparency Requirements

Preferred output:

```text
transparent PNG
```

If ComfyUI cannot produce transparency directly:

1. Generate with plain background.
2. Remove background through deterministic process or matting workflow.
3. Review transparent output.
4. Store transparency method in metadata.

Reject sprites with complex backgrounds unless they are temporary candidates.

## Sprite Review UI Requirements

The web portal must provide sprite-specific review features.

Required:

```text
Show full sheet
Show individual frames
Preview animation loop
Choose direction
Choose animation
Change playback speed
Toggle checkerboard background
Zoom
Step frame forward/backward
Show metadata
```

Recommended:

```text
Compare sprite colors to approved portrait
Show onion-skin frame overlay
Show tile boundary grid
Show pivot/baseline overlay
Export test GIF
```

## Sprite Review Checklist

Manual checklist:

```text
Matches approved portrait
Matches approved full body
Hair color consistent
Outfit color consistent
Race/species traits preserved
Weapon preserved
Readable silhouette
No severe frame jitter
No missing frames
No background artifacts
Correct tile size
Correct metadata
Animations understandable
Unity-ready
```

Approval should require passing most identity and technical checks.

## Common Failure Modes

### Identity Drift

```text
Different hair
Different colors
Different race traits
Different outfit
Different weapon
Different body type
```

Action:

```text
Reject or revise with identity_fix.
```

### Frame Drift

```text
Character changes between frames
Weapon changes shape
Hair length changes
Armor appears/disappears
```

Action:

```text
Reject animation set.
```

### Grid Failure

```text
Frames not aligned
Wrong tile size
Characters cropped
Rows uneven
```

Action:

```text
Reject or send to deterministic sheet assembly fix.
```

### Background Failure

```text
Background not transparent
Checker pattern baked into image
Scene background included
```

Action:

```text
Reject or process background removal.
```

### Overdetail

```text
Sprite too detailed to read at tactical scale
Noisy armor
Tiny unreadable face
```

Action:

```text
Revise with simplification prompt.
```

## Deterministic Assembly

Do not rely on AI to produce the final atlas layout if avoidable.

Preferred approach:

1. Generate individual frame images.
2. Validate each frame size.
3. Normalize each frame to tile canvas.
4. Assemble final sheet using code.
5. Generate metadata from assembly process.

Project implementation language should follow the Wishes stack. Use TypeScript/Node for asset-service processing unless there is a strong reason to use another service.

Do not introduce Python into the Wishes runtime pipeline unless explicitly approved.

## File Outputs

For each approved sprite sheet, store:

```text
sheet.png
metadata.json
preview.gif optional
source_frames/ optional
```

Recommended approved folder:

```text
generated-assets/approved/{object_type}/{object_uuid}/tactical_sprite_sheet/v{version}/
  sheet.png
  metadata.json
```

Recommended candidate folder:

```text
generated-assets/pending/{object_type}/{object_uuid}/tactical_sprite_sheet/v{version}/{job_uuid}/
  sheet.png
  metadata.json
  frames/
```

## Database Metadata

Store sprite metadata in `asset.generation_metadata` or as an adjacent metadata asset.

Recommended `generation_metadata.sprite`:

```json
{
  "tile_width": 128,
  "tile_height": 128,
  "directions": ["front", "back", "left", "right"],
  "animations": ["idle", "walk", "attack", "cast", "hit", "down"],
  "metadata_uri": "generated-assets/approved/card/.../metadata.json",
  "sheet_uri": "generated-assets/approved/card/.../sheet.png",
  "source_role": "full_body"
}
```

## API Requirements

Sprite sheet review uses the asset API defined earlier.

Recommended sprite-specific endpoints may be added later:

```text
GET /api/assets/:assetUuid/sprite-metadata
POST /api/assets/:assetUuid/sprite-metadata
POST /api/assets/:assetUuid/generate-preview-gif
```

First milestone can serve metadata through the normal asset detail endpoint.

## Prompt Templates

### Sprite Base Prompt

```text
small tactical RPG character sprite base, same character as approved full body and portrait, preserve hairstyle, hair color, outfit colors, race traits, weapon, and silhouette, simplified readable fantasy game sprite, neutral standing pose, transparent background
```

### Turnaround Prompt

```text
four direction tactical RPG sprite turnaround, front back left right, same character in every view, consistent height, consistent outfit, consistent hair, consistent weapon, aligned feet, transparent background
```

### Idle Prompt

```text
idle animation frames for the same tactical RPG sprite character, subtle breathing motion, consistent outfit, consistent colors, no identity drift, transparent background
```

### Walk Prompt

```text
walk cycle frames for the same tactical RPG sprite character, clear stepping motion, consistent weapon and outfit, aligned feet, no identity drift, transparent background
```

### Attack Prompt

```text
basic attack animation frames for the same tactical RPG sprite character, anticipation strike recovery, readable weapon motion, consistent outfit and identity, transparent background
```

### Cast Prompt

```text
spell casting animation frames for the same tactical RPG sprite character, magical casting pose, hands or staff emphasized, consistent outfit and identity, transparent background
```

## Implementation Phases

### Phase 1 — Minimal Sprite Candidate

- Generate one front-facing sprite base.
- Store as candidate.
- Review in portal.
- Approve/reject.

### Phase 2 — Four-Direction Turnaround

- Generate front/back/left/right views.
- Store metadata.
- Review directional consistency.

### Phase 3 — Animation Rows

- Generate required animation rows.
- Assemble sheet.
- Preview animations in portal.

### Phase 4 — Unity Import

- Generate metadata consumed by Unity.
- Create test Unity import.
- Verify tactical map rendering.

### Phase 5 — Advanced Generation

- Add eight directions.
- Add optional animations.
- Add equipment overlays.
- Add character LoRA assistance.

## Acceptance Criteria

Sprite generation implementation is complete when:

1. Sprite jobs require an approved portrait.
2. Sprite jobs prefer approved full body source.
3. Sprite candidates preserve visual identity.
4. Required animations are generated or represented.
5. Required directions are generated or represented.
6. Sprite sheet is saved as PNG.
7. Metadata JSON is generated.
8. Review UI can preview animation loops.
9. Unity can consume the sheet and metadata.
10. Approved sprite sheet links back to source portrait/full body.
11. Portrait replacement marks sprite sheets stale.
12. Failed sprite generation jobs are retryable.
