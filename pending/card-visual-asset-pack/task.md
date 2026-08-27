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

Files currently under `assets/` are legacy repository-transfer copies from the first bundle submission:

- `leaf-emblem-medallion.webp` — primary three-leaf emblem/medallion.
- `border-buckle-crest.webp` — lower card-border buckle/crest motif.
- `card-front-border.webp` — approved front card-border direction.
- `card-back-simple.webp` — approved simple book-cover card back.

These historical WebP files are **not the preferred master format**. Do not create new WebP/JPEG/AVIF master or reference files from this task. Where original lossless sources are available, use those sources as the asset authority and retain the older compressed files only as lineage/legacy transfer artifacts.

## Canonical Visual Reference

The required reference file is:

- `references/design-notes-reference.png` — the **original 1536 × 1024 lossless PNG design-reference sheet created in the Wishes design conversation**. It defines the approved leaf emblem, individual leaf construction, tendrils, buckle variations, materials, finish, palette, and reuse guidance.

Expected SHA-256 for the original source PNG:

`b063891133788f2f9aebc42d5f261923a304b8792ac5fc0e121ca825bd454323`

The former 256px `references/design-notes-reference.webp` is explicitly superseded and must not be used. A user-interface screenshot is also not a design-reference source.

The bundle root also contains:

- `design-notes.md` — authoritative written constraints and image-master policy.
- `manifest.json` — bundle inventory, roles, dimensions, source notes, and expected source checksum.

## Image Master Policy

- Preserve source, master, approval, and visual-reference images at their **original available resolution**.
- Use **lossless PNG** for raster masters/reference sheets unless the user explicitly approves another master format.
- Never downscale an approval/reference image merely to reduce repository size.
- Do not convert reference/master images to WebP, JPEG/JPG, AVIF, or another lossy/quality-reducing format.
- Do not treat a compact transfer derivative as visual authority when an original lossless source exists.
- Preserve dimensions, checksum, source lineage, and original filename/source identity where supported.

## Scope

1. Locate the current Wishes asset-pack registry/storage conventions and existing card frame/card back asset roles.
2. Import the approved assets into the appropriate production asset pack(s), preferring original lossless masters whenever available.
3. Register reusable roles/metadata for at least:
   - card leaf emblem / medallion
   - card lower buckle / crest
   - card front border / frame
   - card back / cover
   - card visual design-reference sheet
4. Attach `references/design-notes-reference.png` as the canonical source/reference lineage for this visual language.
5. If derivative sizes/formats are required by runtime code, keep them non-authoritative and traceable to the lossless master; do not replace the master.
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
- The original 1536 × 1024 PNG design-reference sheet is available to the supported generation/reference workflow.
- Any generated derivatives are traceable back to the supplied lossless source.
- Commit and push the completed integration to the appropriate Wishes repository/branch according to repository rules.

## Safety Rules

- No destructive deletion of existing asset masters.
- Do not overwrite a newer approved asset without explicit comparison and preservation of history.
- Do not reinterpret the approved card-back design by adding more ornamentation.
- Never promote a compressed/downscaled derivative over the approved lossless source.

## Notes

- The simple centered-leaf card back supersedes the earlier, more ornate multi-crest back concept.
- The original leaf emblem & buckle design-reference sheet created in the design conversation is explicitly approved as the style anchor for future designs.
- UI screenshots are not source/reference assets for this pack.
