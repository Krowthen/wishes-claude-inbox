# 06 — Character Consistency & Visual Identity Specification

## Objective

Define how the Wishes ComfyUI asset pipeline preserves character identity across all generated visual assets.

The central goal is simple:

```text
All derived assets must look like they belong to the same character as the approved portrait.
```

This file defines the consistency rules, reference-image strategy, prompt strategy, revision rules, model/LoRA strategy, and future automated identity scoring approach.

## Core Principle

The approved portrait is the visual identity authority.

Every downstream asset should derive from the approved portrait directly or indirectly.

Canonical visual source chain:

```text
Approved Portrait
  -> Full Body
      -> Tactical Sprite Sheet
  -> Card Front
  -> Icon
  -> Thumbnail
  -> Emoji Set
  -> Turnaround
```

If the approved portrait changes, all downstream assets become suspect and should be marked stale unless explicitly preserved by human review.

## Identity Attributes

Character visual identity is composed of persistent attributes.

The pipeline should preserve:

```text
Face structure
Eye shape
Eye color
Hair color
Hairstyle
Skin tone
Race/species traits
Age presentation
Body type
Dominant silhouette
Clothing theme
Armor theme
Primary color palette
Secondary color palette
Major accessories
Weapon identity
Art style
Lighting mood
Expression range
```

Not every asset can preserve every attribute perfectly. For example, tactical sprites cannot preserve fine facial details, but they must preserve silhouette, colors, outfit theme, and major accessories.

## Identity Tiers

Use identity tiers to evaluate assets.

### Tier 1 — Critical Identity

These must match unless explicitly changed:

```text
Race/species traits
Face identity
Hair color
Hairstyle
Skin tone
Age presentation
Primary outfit theme
Primary color palette
Art style
```

Failure of Tier 1 attributes usually means rejection.

### Tier 2 — Strong Identity

These should match closely:

```text
Eye color
Armor/clothing details
Major accessories
Weapon shape
Secondary color palette
Body type
Silhouette
Expression style
```

Tier 2 failures may be fixed through revision.

### Tier 3 — Flexible Identity

These may vary between assets:

```text
Pose
Expression
Lighting intensity
Background
Minor clothing folds
Minor accessories
Camera angle
Hand position
Action movement
```

Tier 3 variation is acceptable if the character still reads as the same person.

## Prompt Identity Block

Every downstream prompt must include an identity preservation block.

Recommended block:

```text
same character as the approved portrait, preserve the same face, same hairstyle, same hair color, same eye color, same age presentation, same race and species traits, same outfit theme, same armor style, same dominant colors, same fantasy art style, consistent identity across all assets
```

For full body:

```text
extend the approved portrait into a full body design without changing the character identity, preserve the face and hair exactly, logically continue the clothing and armor visible in the portrait, show the complete outfit head to toe
```

For sprite:

```text
simplified tactical RPG sprite version of the same approved character, preserve the dominant colors, silhouette, hairstyle, outfit theme, race traits, and weapon identity
```

For icon:

```text
small readable icon of the same approved character, preserve the face, hair, colors, and species traits
```

For card front:

```text
card artwork of the same approved character, preserve identity from the portrait, use deterministic card text and frame overlays outside the AI image generation step
```

## Negative Identity Block

Every downstream prompt should include negative identity terms.

Recommended block:

```text
different person, different face, different hairstyle, different hair color, different race, different species traits, different age, different outfit, different armor, inconsistent character design, identity drift, style drift
```

Additional full body negatives:

```text
cropped feet, missing hands, hidden face, helmet covering face, back turned, completely different costume, modern clothing
```

Additional sprite negatives:

```text
unreadable sprite, inconsistent frames, different outfit colors, changing hair between frames, malformed limbs, broken animation, noisy silhouette
```

## Reference Image Strategy

Use image-reference conditioning for all downstream assets.

Recommended reference priority:

```text
Portrait -> Icon
Portrait -> Thumbnail
Portrait -> Card Front
Portrait -> Full Body
Full Body -> Sprite Sheet
Portrait + Full Body -> Sprite Sheet when supported
```

If the full body asset is not available yet, sprite generation may use the portrait, but the result should be considered lower confidence.

## Reference Weighting

Different asset roles need different reference strength.

Suggested starting values:

```text
portrait refinement: high identity strength, medium denoise
full body: high identity strength, medium composition freedom
icon: very high identity strength, low creativity
thumbnail: deterministic crop preferred
card front: high identity strength, medium composition freedom
sprite base: medium-high identity strength, high simplification
sprite sheet: medium identity strength, high consistency between frames
emoji: high identity strength, expression freedom
```

Actual settings depend on the installed ComfyUI nodes.

Document the chosen value in `generation_metadata`.

## Denoise Guidance

For img2img-style workflows:

```text
0.15 - 0.30 = minor refinement
0.30 - 0.45 = controlled variation
0.45 - 0.60 = significant rework
0.60+ = high risk of identity drift
```

Recommended:

```text
portrait_refine: 0.25 - 0.40
full_body_from_portrait: 0.45 - 0.60
icon_from_portrait: 0.15 - 0.30
emoji_from_portrait: 0.35 - 0.50
```

For Flux workflows that do not expose denoise the same way, store equivalent strength or guidance settings.

## Seed Strategy

For identity consistency:

1. Preserve the seed when making small revisions.
2. Change the seed when the composition is fundamentally wrong.
3. Store every seed in generation metadata.
4. Allow reviewer to regenerate with same seed or new seed.
5. Do not overwrite prior candidates.

UI actions:

```text
Regenerate Same Seed
Regenerate New Seed
Revise Same Seed
Revise New Seed
```

## Style Locking

The global Wishes style should be stable across assets.

Baseline style:

```text
high quality fantasy RPG character art, painterly digital illustration, clean readable silhouette, detailed but not noisy, expressive face, elegant fantasy design, cohesive color palette, game-ready concept art, consistent lighting, polished card-game illustration
```

The style should be stored in prompt metadata as:

```json
{
  "style": "high quality fantasy RPG character art..."
}
```

Future style profiles:

```text
wishes_default
wishes_dark_fantasy
wishes_cel_shaded
wishes_tactical_sprite
wishes_card_art
wishes_icon
```

## Outfit Consistency

The portrait may not show the entire outfit. Full body generation must infer missing details carefully.

Rules:

1. Preserve visible clothing and armor.
2. Extend visible clothing logically.
3. Preserve color palette.
4. Preserve cultural/racial cues.
5. Avoid adding unrelated modern items.
6. Avoid changing class/profession visual language unless requested.
7. Preserve weapon identity if visible or specified.

Example full body instruction:

```text
The portrait shows a blue cloak, silver shoulder armor, and a dark leather tunic. Continue those elements into a complete full-body outfit with matching boots, gloves, belt, and travel gear.
```

## Race and Species Consistency

Race/species traits are Tier 1 identity.

Examples:

```text
Elf: pointed ears, elegant features, slender silhouette
Dwarf: sturdy build, broad frame, strong facial structure
Human: no non-human traits unless specified
Beastkin: animal traits must remain consistent
Frogfolk: amphibian traits must remain consistent
Harpies: wing and avian traits must remain consistent
```

If a race has canonical anatomy, future prompts should derive from the card template/body part system.

## Age Presentation

Age drift is a common AI failure.

Store intended age presentation in prompt metadata when known:

```json
{
  "age_presentation": "young adult"
}
```

Use clear prompt terms:

```text
young adult
middle-aged
elder
child
ancient immortal
```

Avoid ambiguous terms like "mature" unless context is clear.

## Expression Consistency

The approved portrait may define a neutral expression, but expression may vary for derived assets.

Acceptable variation:

```text
neutral
determined
serious
confident
focused
smiling
angry
injured
surprised
```

Unacceptable unless requested:

```text
wildly different personality
comic exaggeration
horror distortion
uncharacteristic caricature
```

## Character-Specific LoRA Strategy

Do not require character-specific LoRA for the first milestone.

However, prepare the data model and folder structure for future LoRA training.

Recommended lifecycle:

```text
Approved Portrait
  -> Generate additional controlled references
  -> Human approve reference set
  -> Train character LoRA
  -> Register LoRA as asset/workflow metadata
  -> Use LoRA for downstream assets
```

Minimum reference set:

```text
Approved portrait
Approved full body
Approved icon
Approved expression variants
Approved turnaround if available
```

Recommended number of images:

```text
5 - 20
```

LoRA storage:

```text
server/asset-service/models/loras/
  character/
    {object_uuid}/
      visual_identity_v1.safetensors
      training_metadata.json
```

LoRA metadata:

```json
{
  "lora_code": "character_visual_identity_v1",
  "object_type": "card",
  "object_uuid": "...",
  "source_asset_uuids": [],
  "base_model": "flux1-schnell",
  "training_config": {},
  "model_uri": "server/asset-service/models/loras/character/.../visual_identity_v1.safetensors",
  "status": "trained"
}
```

## LoRA Approval

A trained LoRA should not automatically become active.

Review process:

1. Generate test images with the LoRA.
2. Compare against approved portrait.
3. Approve or reject LoRA.
4. Mark LoRA active only after approval.

Test prompts:

```text
portrait
full body
action pose
icon
expression
```

## Automated Identity Scoring

Automated identity scoring is a future enhancement.

Possible methods:

```text
Face embedding similarity
CLIP image similarity
Color palette similarity
Silhouette comparison
Manual checklist scoring
Reviewer score aggregation
```

First milestone should use manual review only.

Future scoring output:

```json
{
  "identity_score": 0.87,
  "face_similarity": 0.91,
  "palette_similarity": 0.82,
  "style_similarity": 0.88,
  "silhouette_similarity": 0.74,
  "recommendation": "review"
}
```

Suggested thresholds:

```text
0.90+ likely consistent
0.75 - 0.89 human review
below 0.75 likely identity drift
```

Never auto-approve solely from score. Human approval remains required.

## Manual Consistency Checklist

Every derived asset review should include this checklist:

```text
Same face
Same hairstyle
Same hair color
Same eye color
Same skin tone
Same race/species traits
Same age presentation
Same outfit theme
Same armor/clothing colors
Same art style
Same personality impression
Readable silhouette
No major anatomy issues
No watermark/signature/text artifacts
```

Checklist values:

```text
Pass
Fail
Not Applicable
```

A failed Tier 1 item should block approval unless reviewer provides an override.

## Override Rules

Sometimes visual changes are intentional.

Override examples:

```text
New armor after progression
Disguise
Class change
Age progression
Transformation
Corruption
Divine awakening
Battle damage
Seasonal outfit
```

Overrides must be stored in review metadata:

```json
{
  "manual_overrides": [
    {
      "field": "outfit_theme",
      "reason": "Approved class-change armor upgrade",
      "actor_uuid": "...",
      "created_at": "..."
    }
  ]
}
```

## Visual Identity Profile

Create a derived profile after portrait approval.

Example:

```json
{
  "hair_color": "silver",
  "hairstyle": "long braided hair",
  "eye_color": "blue",
  "skin_tone": "fair",
  "race_traits": ["pointed ears"],
  "age_presentation": "young adult",
  "primary_colors": ["blue", "silver", "white"],
  "outfit_theme": "arcane travel cloak and light armor",
  "major_accessories": ["staff", "silver circlet"],
  "art_style": "painterly fantasy RPG"
}
```

This profile may be manually edited and used by PromptBuilderService.

## Revision Taxonomy

Use consistent revision categories.

```text
identity_fix
style_fix
anatomy_fix
outfit_fix
color_fix
composition_fix
quality_fix
role_requirement_fix
sprite_grid_fix
card_layout_fix
```

Revision request shape:

```json
{
  "revision_type": "identity_fix",
  "revision_notes": "Make the face match the approved portrait more closely and restore the silver hair color.",
  "preserve_seed": true,
  "source_asset_uuid": "..."
}
```

## Drift Detection by Asset Type

### Full Body

Common drift:

```text
Different face
Different hair
Different outfit
Wrong body type
Race traits missing
Feet cropped
Hands malformed
```

### Icon

Common drift:

```text
Over-stylized
Face too different
Unreadable at small size
Wrong crop
Too much background
```

### Card Front

Common drift:

```text
AI changed face
Frame overwhelms character
Text artifacts
Wrong quality visual language
Wrong element symbol
```

### Sprite Sheet

Common drift:

```text
Colors change per frame
Character changes direction-to-direction
Hair disappears
Weapon changes shape
Animation flicker
Sprite too detailed to read
```

## Stale Identity Handling

When portrait changes:

1. All descendants should be marked stale.
2. UI should show old portrait beside derived assets.
3. Reviewer may manually preserve a descendant only if still visually valid.
4. Preserved descendant must update source reference or store override metadata.
5. Default behavior should be regeneration.

## Acceptance Criteria

Character consistency implementation is complete when:

1. Approved portrait is treated as visual authority.
2. Downstream prompts include identity preservation blocks.
3. Downstream prompts include identity drift negatives.
4. Generated assets store source references and source versions.
5. Review UI exposes manual consistency checklist.
6. Full body, icon, thumbnail, card front, and sprite sheet review all compare against the approved portrait.
7. Portrait replacement marks descendants stale.
8. Manual overrides are captured in metadata.
9. Visual identity profile can be stored or generated from approved portrait metadata.
10. Future LoRA training has a defined storage and metadata path.
