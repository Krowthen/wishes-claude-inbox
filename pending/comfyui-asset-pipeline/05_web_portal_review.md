# 05 — Web Portal Review & Asset Management Specification

## Objective

Design and implement the primary web interface used to review, approve, revise, regenerate, publish, and inspect AI-generated assets for the Wishes project.

This portal is not a simple image gallery. It is the command center for the complete ComfyUI-driven asset generation pipeline.

The portal must allow artists, developers, and administrators to:

- Generate assets.
- Review candidates.
- Compare revisions.
- View lineage.
- Approve or reject assets.
- Queue regeneration.
- Track workflow status.
- Publish approved assets.
- Inspect generation metadata.
- Validate visual consistency against the approved portrait.

The portal must support every asset type defined by the Wishes asset system while remaining portrait-first.

## Core Review Rule

The approved portrait is the canonical visual source.

Every downstream review screen must show the approved portrait beside the generated candidate unless the candidate itself is the portrait.

Required downstream comparison:

```text
Approved Portrait
  -> Candidate Full Body
  -> Candidate Card Front
  -> Candidate Icon
  -> Candidate Thumbnail
  -> Candidate Tactical Sprite Sheet
```

A reviewer must always be able to answer:

```text
Does this asset still look like the approved portrait?
```

## Primary Navigation

Recommended navigation structure:

```text
Dashboard
Assets
  Portraits
  Full Body
  Card Fronts
  Sprites
  Icons
  Thumbnails
Queue
Characters
Templates
Workflows
Models
Review
Published
Settings
```

For the first milestone, implement:

```text
Dashboard
Review
Assets
Queue
Characters
Settings
```

## Dashboard

The dashboard provides immediate status of the asset pipeline.

Required widgets:

```text
Pending Review
Queued Jobs
Running Jobs
Failed Jobs
Approved Today
Rejected Today
Published Today
Portraits Awaiting Approval
Derived Assets Awaiting Review
Stale Assets
ComfyUI Status
Storage Usage
```

Recommended later widgets:

```text
Average Generation Time
GPU Utilization
Most Failed Workflow
Most Active Asset Role
Recently Published Assets
```

## Recent Activity Feed

The dashboard should include a recent activity feed.

Activity examples:

```text
Portrait generated
Portrait approved
Full body generated
Card front rejected
Sprite sheet failed
Thumbnail published
Portrait replaced
Descendants marked stale
Regeneration queued
```

Each activity item should display:

```text
Timestamp
Actor
Object type
Object name or UUID
Asset role
Previous status
New status
Short comment
```

## Asset Browser

The Asset Browser is the primary navigation tool for all generated assets.

Display modes:

```text
Grid
Large Cards
List
Timeline
Dependency Graph
```

First milestone:

```text
Grid
List
```

Each asset card should display:

```text
Preview image
Asset role
Status
Version
Object type
Object name or UUID
Created date
Workflow code
Current marker
Stale marker
Source marker
```

## Filters

Required filters:

```text
Object Type
Asset Role
Status
Version
Workflow
Created Date
Approved Date
Published Date
Is Current
Is Stale
Has Source Asset
```

Recommended filters:

```text
Character
Card Template
Card
Deck
Model
LoRA
Seed
Reviewer
Generator
```

## Search

Search should support:

```text
Asset UUID
Object UUID
Object name
Prompt text
Workflow code
Asset role
Status
Review notes
Filename
Storage URI
```

## Character Asset Workspace

Every character or character-card object should have a dedicated asset workspace.

Recommended layout:

```text
------------------------------------------------------------
Character Header
------------------------------------------------------------
Name
Object UUID
Object Type
Current Visual Status
Required Asset Completion
Primary Approved Portrait

------------------------------------------------------------
Canonical Portrait
------------------------------------------------------------
Large approved portrait preview
Portrait version
Approve/replace/revise actions

------------------------------------------------------------
Required Derived Assets
------------------------------------------------------------
Card Front
Full Body
Icon
Thumbnail
Tactical Sprite Sheet

------------------------------------------------------------
Optional Assets
------------------------------------------------------------
Emoji Set
Turnaround
Model Reference
Marketing Art

------------------------------------------------------------
History and Metadata
------------------------------------------------------------
Version History
Lineage
Generation Jobs
Review Events
Workflow Metadata
```

## Required Asset Completion Indicator

Each character workspace should show completion status:

```text
Portrait: Approved
Full Body: Pending Review
Card Front: Missing
Icon: Approved
Thumbnail: Approved
Sprite Sheet: Failed
```

Recommended progress display:

```text
4 / 6 Required Assets Complete
```

## Portrait Review Screen

The portrait review screen is the most important page.

Layout:

```text
------------------------------------------------------------
Candidate Portrait Preview
------------------------------------------------------------

------------------------------------------------------------
Candidate Metadata
------------------------------------------------------------
Asset UUID
Job UUID
Object Type
Object UUID
Version
Status
Workflow
Seed
Model Stack
Dimensions
Created At

------------------------------------------------------------
Prompt Data
------------------------------------------------------------
Positive Prompt
Negative Prompt
User Notes
Generated Prompt Package

------------------------------------------------------------
Review Actions
------------------------------------------------------------
Approve
Reject
Revise
Generate Another
Archive
```

Portrait image should occupy most of the available visual area.

Metadata must remain accessible without hiding the image.

## Portrait Approval UX

When approving a portrait, show a confirmation dialog:

```text
Approve this portrait as the canonical visual source?
```

If no prior portrait exists:

```text
This portrait will become the approved portrait for this object.
```

If a prior portrait exists:

```text
Approving this portrait will replace the current canonical portrait.
Existing downstream assets may become stale.
```

The dialog should list affected downstream assets:

```text
Full Body v1 — will become stale
Card Front v1 — will become stale
Icon v1 — will become stale
Thumbnail v1 — will become stale
Sprite Sheet v1 — will become stale
```

Options:

```text
Approve Only
Approve and Queue Regeneration
Cancel
```

## Portrait Rejection UX

When rejecting a portrait, require or strongly encourage a comment.

Common rejection reasons:

```text
Wrong face
Wrong race/species traits
Wrong age presentation
Wrong style
Bad anatomy
Poor image quality
Does not match prompt
Too modern
Incorrect clothing
Unusable composition
```

The UI should allow selecting one or more rejection reason tags plus free-text notes.

## Portrait Revision UX

A reviewer should be able to revise a portrait without starting over.

Revision inputs:

```text
Keep same identity
Fix eyes
Adjust expression
Change hair slightly
Improve armor
Adjust color palette
Make older
Make younger
Reduce noise
Improve face symmetry
```

The portal should generate a revision request using:

```text
source_asset_uuid = current portrait candidate or approved portrait
asset_role = portrait
workflow = portrait_refine
revision_notes = reviewer notes
```

## Side-by-Side Comparison

Every review page for a derived asset must include side-by-side comparison.

Layout:

```text
------------------------------------------------------------
Approved Portrait        Candidate Asset
------------------------------------------------------------
Source Metadata          Candidate Metadata
------------------------------------------------------------
Lineage                  Prompt / Workflow
------------------------------------------------------------
Actions
------------------------------------------------------------
Approve | Reject | Revise | Regenerate
```

For image assets, include:

```text
Synchronized zoom
Pan
Fit to screen
Actual size
Toggle background checker
Open full screen
```

## Visual Consistency Checklist

Derived asset review should display a checklist:

```text
Same face
Same hairstyle
Same hair color
Same eye color
Same race/species traits
Same age presentation
Same outfit theme
Same dominant color palette
Same art style
Readable silhouette
No major anatomy errors
```

Each item may be:

```text
Pass
Fail
Not Applicable
```

The checklist can be manual for milestone one. Automated identity scoring can come later.

## Full Body Review

Full body review must compare candidate full body against the approved portrait.

Additional checks:

```text
Head-to-toe visible
Boots/feet visible
Hands visible
Outfit consistent with portrait
Armor/clothing logically extended
No unwanted helmet if portrait has exposed face
Pose usable for sprite generation
Silhouette readable
```

Actions:

```text
Approve Full Body
Reject
Revise
Generate Sprite Base
Generate Another
```

## Icon Review

Icon review should show candidate at multiple sizes.

Preview sizes:

```text
512 x 512
256 x 256
128 x 128
64 x 64
32 x 32
```

Checklist:

```text
Readable at small size
Face clear
Matches approved portrait
No excessive background noise
No text artifacts
Correct crop
```

## Thumbnail Review

Thumbnail review should show candidate at small UI sizes.

Preview sizes:

```text
256 x 256
128 x 128
64 x 64
```

Thumbnail approval should usually be fast because thumbnails are deterministic derivatives of approved portrait or full body assets.

## Card Front Review

Card front review should support both image review and card data review.

Display:

```text
Card front preview
Approved portrait or full body source
Card metadata
Frame metadata
Rendered text
Stats
Element icons
Quality
```

Critical rule:

```text
Do not approve card fronts where important text is AI-generated or unreadable.
```

Text should be rendered deterministically by the app, not generated by the image model.

Card front checklist:

```text
Character art matches portrait
Frame matches quality
Element icons are correct
Card name is readable
Stats are readable
No AI text artifacts
No watermark/signature
Composition fits card frame
```

## Tactical Sprite Sheet Review

Sprite review must support animation playback.

Minimum viewer features:

```text
Show full sprite sheet
Detect frame grid
Preview animation by row
Preview direction
Change playback speed
Toggle background checker
Zoom
Step frame forward/backward
```

Required animation previews:

```text
Idle
Walk
Attack
Cast
Hit
Down
```

Required direction previews:

```text
Front
Back
Left
Right
```

If the initial implementation does not support automatic slicing, provide manual metadata entry:

```text
tile_width
tile_height
row
frame_count
direction
animation
```

## Sprite Approval Checklist

```text
Character resembles approved portrait
Outfit colors preserved
Silhouette readable at tactical scale
Frames align correctly
No major flicker
No missing frames
Animations are understandable
Grid metadata is correct
Transparent or valid background
Usable in Unity tactical prototype
```

## Asset Lineage View

Every asset detail page must include lineage.

Simple first milestone lineage:

```text
Root Portrait
  -> Full Body
      -> Sprite Sheet
  -> Icon
  -> Thumbnail
  -> Card Front
```

Each node should show:

```text
Asset role
Version
Status
Current marker
Stale marker
Created date
Approved date
```

Clicking a node opens that asset detail page.

## Version History

Each object/role should show version history.

Example:

```text
Portrait
  v1 Approved Archived
  v2 Approved Current

Full Body
  v1 Stale
  v2 Pending Review
```

Version list should include:

```text
Preview
Version
Status
Created at
Approved at
Reviewer
Workflow
Seed
Source version
Review notes
```

## Queue Monitor

The Queue page should show all generation jobs.

Columns:

```text
Job UUID
Object Type
Object UUID
Asset Role
Workflow
Status
Priority
Attempt Count
Created At
Started At
Finished At
Worker ID
ComfyUI Prompt ID
```

Actions:

```text
Retry Failed
Cancel Queued
Open Asset
Open Object
View Error
```

Statuses should be color-coded.

## Failed Job Detail

Failed jobs should show:

```text
Error code
Error message
Workflow
Prompt
Source asset
Attempt count
Raw error metadata
Retry button
```

Common error help text:

```text
ComfyUI unavailable
Workflow node missing
Source image missing
Output not found
Image validation failed
Database error
```

## Prompt Editor

The review portal should include a structured prompt editor.

Fields:

```text
Positive Prompt
Negative Prompt
Style Prompt
Role Instruction
Identity Constraints
User Notes
Revision Notes
Seed
Width
Height
Workflow
Source Asset
```

Editing a prompt should not mutate the existing asset. It should queue a new generation job.

Buttons:

```text
Regenerate With Same Seed
Regenerate With New Seed
Create Revision
Reset to Generated Prompt
```

## Regeneration Flow

Regeneration should preserve lineage.

Flow:

```text
Open asset
Click Regenerate
Choose source asset
Edit revision notes
Choose workflow/settings
Submit
New job queued
New candidate appears in review
```

For derived assets, default source should be the current approved portrait or preferred approved parent.

## Batch Review

Batch review is useful after generating multiple candidates.

Batch actions:

```text
Approve Selected
Reject Selected
Archive Selected
Queue Regeneration
Publish Selected
```

Batch approval should be restricted when current-asset conflicts exist.

Example warning:

```text
You selected 3 portrait candidates for the same character.
Only one portrait can be current.
Choose one canonical portrait.
```

## Publication View

Published page should show only assets ready for game/client use.

Display:

```text
Published URI
Object
Asset Role
Version
Published Date
Publisher
Current/Stale status
```

If a published asset becomes stale, show a warning:

```text
Published but stale because source portrait was replaced.
```

## Settings Page

Settings should expose local development configuration:

```text
ComfyUI Base URL
Asset Storage Root
Asset Relative Root
Worker Poll Interval
Max Concurrent Jobs
Default Portrait Workflow
Default Full Body Workflow
Default Sprite Workflow
```

Settings can be read-only in first milestone if configured through `.env`.

## Access Control

For first prototype, authentication may be stubbed.

Future permission model:

```text
Viewer
Reviewer
Artist
Publisher
Admin
```

Suggested permissions:

```text
Viewer: view assets and metadata
Artist: queue generation and revisions
Reviewer: approve/reject
Publisher: publish approved assets
Admin: manage workflows and settings
```

## Keyboard Shortcuts

Optional but useful:

```text
A = approve
R = reject
G = regenerate
P = publish
Left/Right = previous/next asset
Space = play/pause sprite animation
F = fullscreen preview
```

## Accessibility

Minimum accessibility requirements:

```text
All buttons have labels
Images have alt text or generated labels
Status is not color-only
Keyboard navigation works
Dialog confirmations are focus-trapped
Text contrast is readable
```

## Frontend Implementation Recommendation

If the prototype web portal already uses React/Vite, implement this as a React module.

Recommended structure:

```text
prototypes/character-creator/src/
  screens/
    AssetDashboard.tsx
    AssetReview.tsx
    AssetBrowser.tsx
    AssetQueue.tsx
    CharacterAssets.tsx
  components/assets/
    AssetCard.tsx
    AssetGrid.tsx
    AssetStatusBadge.tsx
    AssetPreview.tsx
    AssetComparison.tsx
    AssetLineageGraph.tsx
    AssetVersionHistory.tsx
    PromptEditor.tsx
    SpriteSheetPreview.tsx
    ReviewActions.tsx
  services/
    assetApi.ts
  types/
    asset.ts
```

If a new web portal is created later, this module can be moved.

## API Integration

Frontend should use the asset API from `04_backend_and_api.md`.

Required API calls:

```text
GET /api/assets/health
GET /api/assets/roles
GET /api/assets/workflows
POST /api/assets/requests
GET /api/assets/jobs/:jobUuid
GET /api/assets/objects/:objectType/:objectUuid
GET /api/assets/:assetUuid
GET /api/assets/:assetUuid/lineage
POST /api/assets/:assetUuid/approve
POST /api/assets/:assetUuid/reject
POST /api/assets/:assetUuid/publish
POST /api/assets/:assetUuid/regenerate
POST /api/assets/:assetUuid/regenerate-descendants
```

## First Milestone UI Scope

Implement only:

```text
Asset Dashboard
Asset Browser
Asset Detail
Portrait Review
Derived Asset Review
Queue Monitor
Approve/Reject buttons
Basic Prompt Regeneration
Basic Lineage View
```

Defer:

```text
Advanced graph visualization
Automated identity scoring
Batch operations
Keyboard shortcuts
Role-based permissions
Advanced sprite slicing
Collaborative comments
```

## Manual Test Flow

1. Open Asset Dashboard.
2. Confirm ComfyUI status.
3. Queue portrait generation for a test object.
4. Wait for job to complete.
5. Open review pending portrait.
6. Approve portrait.
7. Queue full body generation from approved portrait.
8. Open full body review.
9. Compare full body to portrait.
10. Approve full body.
11. Queue icon, thumbnail, card front, sprite sheet.
12. Approve or reject each.
13. Replace portrait.
14. Confirm derived assets show stale state.
15. Queue regeneration of stale assets.

## Acceptance Criteria

The web portal review implementation is complete when:

1. Dashboard shows asset system status.
2. Asset browser lists generated assets.
3. Review screen displays candidate image and metadata.
4. Portraits can be approved or rejected.
5. Derived assets can be compared to approved portrait.
6. Full body, icon, thumbnail, card front, and sprite sheet roles are visible.
7. Queue page displays generation job status.
8. Failed jobs show useful error information.
9. Asset lineage can be viewed.
10. Version history can be viewed.
11. Prompt regeneration can queue a new job.
12. Stale assets are clearly marked.
13. Published assets are distinguishable from approved-only assets.
14. UI supports the full portrait-first review workflow.
