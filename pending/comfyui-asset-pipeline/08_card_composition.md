# 08 — Card Composition & Rendering Specification

## Objective

Define the complete deterministic card composition system for Wishes.

This document establishes the architecture for transforming approved artwork and game data into production-ready card images. AI is responsible only for artwork generation. Every visual interface element—including frames, typography, icons, rarity effects, statistics, and layout—is rendered deterministically by the application.

Primary pipeline:

```text
Card Template
    +
Approved Character Artwork
    +
Frame Assets
    +
UI Overlay Assets
    +
Element Icons
    +
Quality Effects
    +
Localized Text
    +
Live Card Statistics
        ↓
Card Composition Engine
        ↓
Rendered Card Front
        ↓
Review
        ↓
Published Asset
```

## Core Principles

- AI never renders final UI text.
- Card layouts are deterministic and reproducible.
- Templates define layout, not application code.
- Artwork and UI are separated.
- Every rendered card can be recreated from stored metadata.
- Runtime game logic never depends on pixels extracted from images.

## Rendering Architecture

Recommended implementation stack:

```text
TypeScript
Node.js
Canvas-based rendering library
SVG support
PNG/WebP export
```

The renderer should exist as part of the Asset Service rather than inside the browser so outputs remain identical regardless of client platform.

## Rendering Pipeline

1. Load card template.
2. Load approved source artwork.
3. Resolve artwork crop region.
4. Load frame assets.
5. Load overlay assets.
6. Load quality assets.
7. Load element icons.
8. Load fonts.
9. Resolve localized text.
10. Measure and fit text.
11. Render layers.
12. Export image.
13. Generate composition metadata.
14. Store rendered asset.

## Template System

Each card type references a template.

Example directory:

```text
card-templates/
  character/
    default/
      template.json
      frame.png
      masks/
      overlays/
  spell/
  equipment/
  race/
  profession/
```

Template responsibilities:

- Canvas dimensions
- Art bounds
- Safe areas
- Typography
- Frame assets
- Icon positions
- Stat positions
- Overlay order
- Export rules

## Template Schema

Example:

```json
{
  "template":"character_v1",
  "canvas":{"width":768,"height":1075},
  "art":{"x":56,"y":82,"width":656,"height":720,"mode":"cover"},
  "title":{"x":70,"y":28,"width":628,"height":42},
  "description":{"x":62,"y":825,"width":644,"height":150},
  "stats":{"anchor":"bottom_right"},
  "icons":{"elements":"top_left","quality":"top_right"}
}
```

## Layer Order

Render back-to-front:

```text
Background
Frame Base
Artwork Mask
Artwork
Quality Effects
Glow/Particles
Element Icons
Stat Icons
Text
Borders
Foil Overlay
Watermark (optional)
```

Layer order must be template-controlled.

## Artwork Placement

Preferred source:

```text
Approved Full Body
Approved Portrait
```

Support placement modes:

```text
cover
contain
stretch (discouraged)
manual
```

Optional focal point metadata:

```json
{"focus":{"x":0.52,"y":0.28}}
```

## Typography

Render text deterministically.

Fields:

```text
Card Name
Subtype
Rules
Flavor
Stats
```

Support:

- Word wrapping
- Auto-sizing
- Overflow detection
- Ellipsis warnings
- Localization
- Font fallback

Never rasterize AI-generated text into the card.

## Icon System

Icons originate from controlled assets.

Categories:

```text
Elements
Quality
Resources
Combat
Movement
Professions
Classes
Status
```

Icons should be versioned and replaceable without changing templates.

## Dynamic Quality Effects

Quality defines decorative overlays.

Suggested mapping:

```text
Common -> simple frame
Uncommon -> accent corners
Rare -> glow
Epic -> animated-ready highlights
Legendary -> ornate frame
Mythic -> premium embellishments
```

Effects remain deterministic.

## Responsive Layout Rules

Templates must gracefully handle:

- Long names
- Short names
- Multiple element icons
- Large stat values
- Localized text expansion

Validation should flag clipping before publication.

## Export Profiles

Support multiple outputs:

```text
game
ui_preview
thumbnail
print
nft
```

Each profile may override:

- Resolution
- Compression
- Watermark
- Metadata embedding

## Composition Metadata

Every rendered card stores metadata.

Example:

```json
{
  "template":"character_v1",
  "template_version":1,
  "art_asset_uuid":"...",
  "frame_asset":"rare_frame_v2",
  "quality":"rare",
  "export_profile":"game",
  "canvas":{"width":768,"height":1075}
}
```

## Database Integration

Store renderer metadata in the asset record.

Recommended fields:

```text
template_code
template_version
art_asset_uuid
frame_asset_uuid
export_profile
render_duration_ms
```

## Asset Caching

The renderer should cache:

- Fonts
- Frames
- Icons
- Masks
- Overlays

Invalidate cache when template versions change.

## API Endpoints

Suggested additions:

```text
POST /api/cards/render
POST /api/cards/render-preview
GET /api/cards/templates
GET /api/cards/:assetUuid/composition
```

## Review Requirements

Review screen must display:

- Final rendered card
- Source artwork
- Template used
- Statistics
- Composition metadata

Checklist:

- Artwork matches approved character
- Frame correct
- Text readable
- Icons correct
- Stats correct
- No clipping
- Safe areas respected
- Export profile correct

## Unity Integration

Unity consumes the rendered PNG only.

Gameplay data comes from APIs/database, never OCR or embedded pixels.

Card images are presentation assets only.

## Performance

Target:

- Single render < 250 ms after assets cached
- Batch rendering supported
- Deterministic output from identical inputs

## Future Enhancements

- Animated cards
- Shader-based foil
- Dynamic seasonal themes
- NFT overlays
- Multi-language batch rendering
- PDF print sheets
- Accessibility themes

## Acceptance Criteria

1. Renderer is deterministic.
2. Templates fully control layout.
3. Artwork derives from approved assets.
4. Text rendered by application.
5. Icons are deterministic assets.
6. Metadata accompanies every card.
7. Identical inputs produce identical outputs.
8. Renderer supports multiple export profiles.
9. Review portal validates layout before publish.
10. Unity can consume published card assets directly.
