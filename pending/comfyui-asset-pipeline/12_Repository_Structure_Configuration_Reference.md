# 12 — Repository Structure & Configuration Reference

## Objective

This document defines the canonical repository structure for the Wishes project after the AI Asset Pipeline has been fully implemented.

Unlike the implementation guide, this document serves as a permanent reference for developers, Claude Code, and future contributors.

It answers three questions:

1. Where does everything live?
2. Why does it live there?
3. How do all systems interact?

This document should remain synchronized with the repository.

---

# Design Goals

The repository structure should:

- Clearly separate runtime services.
- Separate generated content from source assets.
- Keep AI workflows version controlled.
- Keep implementation modular.
- Allow future microservices.
- Allow multiple asset generators.
- Allow multiple AI providers.
- Keep Unity isolated from backend implementation.

---

# Top-Level Repository Layout

```text
wishes-game/
│
├── client-unity/
├── server/
├── shared/
├── database/
├── generated-assets/
├── docs/
├── prototypes/
├── infrastructure/
├── tools/
├── .github/
├── CLAUDE.md
├── README.md
└── package.json
```

---

# client-unity

Contains the Unity game client.

```text
client-unity/

Assets/

Packages/

ProjectSettings/

UserSettings/
```

Unity should never directly communicate with ComfyUI.

Unity communicates only through the API Gateway.

Responsibilities:

- Rendering
- Gameplay
- UI
- Audio
- Input
- Networking

---

# server

Contains backend services.

```text
server/

api-gateway/

world-simulation/

battle-service/

chat-service/

rules-engine/

asset-service/
```

Each service should be independently deployable.

---

# asset-service

Responsible for the complete AI Asset Pipeline.

```text
asset-service/

src/

tests/

workflows/

workflow-manifests/

templates/

models/

scripts/

config/

package.json

tsconfig.json
```

Responsibilities:

- Prompt generation
- Queue processing
- ComfyUI integration
- Card rendering
- Sprite generation
- Asset review
- Asset publication

---

# src Layout

```text
src/

routes/

controllers/

services/

workers/

storage/

database/

types/

config/

utils/

validation/

renderers/

prompts/

metadata/
```

---

# Routes

```text
routes/

assetRoutes.ts

workflowRoutes.ts

reviewRoutes.ts

jobRoutes.ts

templateRoutes.ts

healthRoutes.ts
```

---

# Controllers

Controllers should only:

- Validate requests
- Call services
- Return responses

No business logic.

---

# Services

Suggested services:

```text
AssetRequestService

AssetReviewService

AssetPublishService

AssetDependencyService

PromptBuilderService

WorkflowRegistryService

StorageService

CardCompositionService

SpriteGenerationService

MetadataService

TemplateService
```

---

# Workers

```text
AssetGenerationWorker

CleanupWorker

ThumbnailWorker

PublicationWorker

HealthWorker
```

Workers remain stateless.

---

# Renderers

```text
CardRenderer

ThumbnailRenderer

PreviewRenderer

SpriteAtlasBuilder
```

All rendering should be deterministic.

---

# Prompts

```text
GlobalStyle.ts

PortraitPrompts.ts

FullBodyPrompts.ts

SpritePrompts.ts

CardPrompts.ts

NegativePrompts.ts

IdentityBlocks.ts
```

---

# Metadata

```text
AssetMetadataBuilder

SpriteMetadataBuilder

CompositionMetadataBuilder

WorkflowMetadataBuilder
```

---

# Workflow Directory

```text
workflows/

portrait_generate.json

portrait_refine.json

full_body_from_portrait.json

card_front_from_portrait.json

icon_from_portrait.json

thumbnail_from_portrait.json

sprite_base.json

sprite_sheet.json
```

Only approved workflow JSON files belong here.

---

# Workflow Manifests

```text
workflow-manifests/

portrait_generate.manifest.json

portrait_refine.manifest.json

...
```

Contains injection schemas.

Never modify workflow JSON directly at runtime.

---

# Templates

Templates contain deterministic layout information.

```text
templates/

card/

sprite/

icons/

frames/

fonts/
```

---

# Models

Recommended layout:

```text
models/

checkpoints/

loras/

controlnets/

vae/

embeddings/

upscalers/
```

Future:

```text
ipadapter/

redux/

flux-kontext/
```

No generated outputs belong here.

---

# Scripts

```text
scripts/

validate-workflows.ts

rebuild-cache.ts

seed-workflows.ts

verify-storage.ts

cleanup-temp.ts
```

---

# Tests

```text
tests/

unit/

integration/

workflow/

performance/

manual/
```

---

# Shared Library

```text
shared/

contracts/

schemas/

types/

constants/

game-rules/
```

Shared code should never depend on backend implementation.

---

# Database

```text
database/

migrations/

functions/

views/

seed/

```

Recommended additions:

```text
seed/

asset/

workflow/

templates/
```

---

# Generated Assets

Generated assets are runtime artifacts.

```text
generated-assets/

pending/

approved/

published/

rejected/

archive/

temp/
```

---

# Pending

Contains review candidates.

Never used by runtime gameplay.

---

# Approved

Contains canonical reviewed assets.

Used to generate derived assets.

---

# Published

Contains immutable runtime assets.

Unity should only consume published assets.

---

# Rejected

Stores rejected candidates.

Never deleted automatically.

Useful for:

- Prompt tuning
- AI comparison
- Training

---

# Archive

Stores historical versions.

Never overwrite.

---

# Temp

Stores temporary render files.

Safe to clean automatically.

---

# Object Layout

Recommended:

```text
generated-assets/

approved/

card/

{object_uuid}/

portrait/

v1/

portrait.png

metadata.json
```

Equivalent layout for:

- full_body
- icon
- thumbnail
- sprite_sheet
- card_front

---

# Documentation

```text
docs/

architecture/

database/

api/

asset-pipeline/

deployment/

templates/
```

Keep implementation documents version controlled.

---

# Infrastructure

```text
infrastructure/

docker/

terraform/

kubernetes/

monitoring/
```

Future additions:

```text
grafana/

prometheus/

loki/
```

---

# Tools

```text
tools/

claude/

comfy/

asset-import/

migration/

```

Your existing Claude Inbox belongs here.

---

# Prototype Applications

```text
prototypes/

character-creator/

asset-review/

admin/

```

Prototype code should not become production code without review.

---

# Configuration

Prefer:

```text
.env

.env.local

.env.development

.env.production
```

Never commit secrets.

---

# Runtime Configuration

Configuration hierarchy:

```text
Environment Variables

↓

Configuration Service

↓

Runtime Objects

↓

Consumers
```

Never hardcode:

- URLs
- File paths
- Credentials
- Ports

---

# Naming Standards

Folders:

lowercase-kebab-case

Files:

camelCase.ts

Interfaces:

PascalCase

Enums:

PascalCase

Database:

snake_case

API:

kebab-case

---

# Import Rules

Allowed:

```text
Route

↓

Controller

↓

Service

↓

Storage

↓

Database
```

Not allowed:

```text
Database

↓

Route
```

Avoid circular dependencies.

---

# Build Pipeline

Suggested build order:

```text
Shared

↓

Database

↓

Asset Service

↓

API Gateway

↓

Other Services

↓

Unity
```

---

# CI Pipeline

```text
Install

↓

Lint

↓

Type Check

↓

Unit Tests

↓

Integration Tests

↓

Workflow Validation

↓

Build

↓

Package

↓

Deploy
```

---

# Versioning

Version independently:

- Workflows
- Templates
- Asset Roles
- Card Templates
- Metadata Schema

Never overwrite old versions.

---

# Repository Rules

The repository should satisfy:

- Modular architecture
- Clear ownership
- No duplicated responsibilities
- Deterministic rendering
- Runtime reproducibility
- Version-controlled workflows
- Version-controlled templates
- Immutable published assets

---

# Future Repository Growth

Expected future additions:

```text
voice-service/

npc-memory/

ai-director/

analytics/

telemetry/

cloud-storage/

cdn/
```

Repository organization should allow these without restructuring.

---

# Acceptance Criteria

Repository organization is complete when:

1. Every subsystem has a clearly defined home.
2. Runtime-generated assets are isolated from source assets.
3. Workflows are version controlled.
4. Templates are version controlled.
5. Generated assets follow normalized storage rules.
6. Services remain modular.
7. Unity consumes only published assets.
8. Configuration is environment driven.
9. Build order is deterministic.
10. Future expansion can occur without major structural changes.