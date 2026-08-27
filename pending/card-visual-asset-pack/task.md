# Card Visual Asset Pack — Approved Leaf Emblem, Buckle, Front Border, and Card Back

Created: 2026-08-26
Priority: high
Mode: implement

Allow Edit: true
Allow Commit: true
Allow Push: true
Allow Delete: false
Allow Asset Import: true
Allow File Copy: true

## Objective

Import this approved visual bundle into the Wishes production asset-pack system and make the motifs available as reusable card/UI assets. Preserve the approved visual language exactly: clean rustic fantasy, aged bronze/brass over dark wood, restrained organic leaf ornament, and strong readable silhouettes.

## Approved Assets

Files in `assets/` are repository-transfer approval/style copies for this task:

- `leaf-emblem-medallion.webp` — primary three-leaf emblem/medallion; canonical reusable leaf motif.
- `border-buckle-crest.webp` — lower card-border buckle/crest motif; canonical lower binding element.
- `card-front-border.webp` — approved front card-border direction using the leaf emblem and lower buckle/crest.
- `card-back-simple.webp` — approved simple book-cover card back: centered leaf emblem, minimal center, retained outer bindings.

Files in `references/` and the bundle root document the approved design language:

- `references/design-notes-reference.webp` — canonical visual reference sheet originally created in the Wishes design conversation. This is the approved source reference for the leaf emblem, individual leaf construction, tendrils, buckle variations, materials, finish, palette, and reuse guidance.
- `design-notes.md` — authoritative written constraints and usage notes.
- `manifest.json` — bundle inventory, roles, hashes, dimensions, and source notes.

**Important:** Do not use a user-interface screenshot as the design reference. The original generated design-reference sheet in `references/design-notes-reference.webp` is the visual authority for this bundle. User approval is represented by this task and the bundle status, not by a screenshot artifact.

The inbox image copies are intentionally compact WebP files for reliable repository transfer. They are authoritative approval/style references, not resolution masters. If identical higher-resolution approved sources exist in the Wishes asset system, retain those as production masters and link these copies as approval/style lineage. Do not substitute a different design.

## Scope

1. Locate the current Wishes asset-pack registry/storage conventions and existing card frame/card back asset roles.
2. Import these approved assets into the appropriate production asset pack(s).
3. Register reusable roles/metadata for at least:
   - card leaf emblem / medallion
   - card lower buckle / crest
   - card front border / frame
   - card back / cover
   - card visual design-reference sheet
4. If the asset system supports source/reference lineage, attach the original design-reference sheet to the production assets.
5. If derivative sizes/formats are required by the runtime, generate non-destructive derivatives and retain traceability to the supplied approved source.
6. Update any asset-pack manifest, role mapping, or seed data needed so the assets are discoverable from the Wishes asset management layer.

## Visual Requirements

### Shared style

- Fantasy-based, clean, rustic, handcrafted.
- Dark aged wood or dark brown book-cover material.
- Aged brass/bronze metal, warm gold highlights, subtle oxidation/patina.
- Clean silhouettes that remain readable at card scale.
- Decorative detail is restrained; avoid visual noise and over-ornamentation.

### Leaf emblem

- Three-leaf composition: one tall center leaf, two outward side leaves.
- Clear engraved/raised vein structure.
- Symmetrical lower curling tendrils.
- Usually presented in an aged bronze/brass circular or oval medallion.
- Motif represents nature, growth, continuity, and unity.

### Lower buckle / crest

- Preserve the approved lower-border binding/crest language.
- Aged bronze/brass construction over a dark inset.
- Central faceted diamond/leaf-like geometric mark with restrained curling filigree.
- Should read as a structural binding or clasp, not a separate heraldic illustration.

### Card front

- Preserve the dark wood + metal binding frame.
- Leaf medallion centered at the top.
- Lower buckle/crest centered at the bottom.
- Parchment/light neutral content field remains open and highly usable.
- Side ornament can use restrained leaf/vine accents, but the frame must remain clean enough for gameplay/card information.

### Card back

- Treat as a simple fantasy book cover.
- Keep the outer metal border bindings/corners.
- Keep the center largely plain dark wood/book material.
- Center ONE leaf emblem/medallion as the focal point.
- Do not add additional large crests, secondary medallions, large filigree fields, text, runes, or extra focal symbols.
- Overall impression: premium, tactile, old-world, restrained.

## Integration Requirements

- Do not replace newer existing production assets blindly. Compare first, then import under the correct asset role/version.
- Preserve aspect ratio and source lineage.
- Store masters separately from runtime derivatives if that is the established asset-pipeline convention.
- Add human-readable names/tags including `card`, `fantasy`, `rustic`, `leaf`, `bronze`, `brass`, `wood`, `border`, `buckle`, `book-cover` as appropriate.
- Record this task bundle path in asset metadata/history when supported.
- These approved visuals are style anchors for future Wishes card asset generation; make them discoverable to ComfyUI/workflow/style-reference tooling if that system already has a supported reference-asset mechanism.

## Expected Output

- Approved assets are present in the appropriate Wishes asset pack(s).
- Asset roles/manifest entries are added and valid.
- Front border and simple card back are selectable/discoverable by the asset-management layer.
- Leaf emblem and buckle/crest are independently reusable.
- The original design-reference sheet is available to the supported generation/reference workflow.
- Any generated derivatives are traceable back to the supplied approved source.
- Commit and push the completed integration to the appropriate Wishes repository/branch according to repository rules.

## Safety Rules

- No destructive deletion of existing asset masters.
- Do not overwrite a newer approved asset without explicit comparison and preservation of history.
- Do not reinterpret the approved card-back design by adding more ornamentation.

## Notes

- The simple centered-leaf card back supersedes the earlier, more ornate multi-crest back concept.
- The original leaf emblem & buckle design-reference sheet created in the design conversation is explicitly approved as the style anchor for future designs.
- UI screenshots are not source/reference assets for this pack.
