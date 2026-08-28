# Wishes AI Asset Pipeline
## Master Reference Manual

**Document Number:** 19

**Document Title:** Wishes AI Asset Pipeline Master Reference

**Classification:** Canonical Technical Reference

**Status:** Living Reference Document

**Repository:** wishes-game

**Applies To**

- Asset Service
- API Gateway
- Character Creator
- Review Portal
- Unity Client
- PostgreSQL
- ComfyUI
- Workflow Registry
- Asset Workers
- Generated Asset Storage

---

# Revision History

| Version | Date | Description |
|----------|------|-------------|
| 1.0 | Initial Release | Initial master reference compiled from Documents 01–18. |

---

# Purpose

This document is the definitive technical reference for the Wishes AI Asset Pipeline.

Documents 01 through 18 describe how the system is designed and implemented.

This document explains how every subsystem works together after implementation.

Unlike the implementation documents, this manual is intended to remain relevant throughout the lifetime of the project.

Whenever questions arise regarding the Asset Pipeline, this document should be considered the primary engineering reference.

---

# Audience

This manual is intended for:

- Core Engine Developers
- Backend Developers
- Unity Developers
- AI Pipeline Developers
- Technical Artists
- DevOps Engineers
- QA Engineers
- Claude Code
- Future AI Engineering Assistants

---

# Relationship to Other Documents

This document does not replace Documents 01–18.

Instead, it consolidates and cross-references them.

| Document | Purpose |
|-----------|---------|
| 01 | System Architecture |
| 02 | Database & Storage |
| 03 | ComfyUI Workflows |
| 04 | Backend & API |
| 05 | Review Portal |
| 06 | Character Consistency |
| 07 | Sprite Generation |
| 08 | Card Composition |
| 09 | Deployment |
| 10 | Validation |
| 11 | Claude Execution Guide |
| 12 | Repository Structure |
| 13 | Configuration |
| 14 | Future Enhancements |
| 15 | Development Standards |
| 16 | Evolution Roadmap |
| 17 | Operations Runbook |
| 18 | Security Architecture |

---

# Reading Order

New developers should read:

1. Core Concept Documents
2. Documents 01–18
3. This reference manual

This manual assumes familiarity with the Wishes architecture.

---

# Design Philosophy

The Wishes AI Asset Pipeline is intentionally built around deterministic systems.

Artificial Intelligence is responsible only for creative asset generation.

Everything else is deterministic.

This separation is the single most important architectural principle in the project.

AI should generate:

- Portraits
- Full Body Artwork
- Icons
- Sprites
- Illustrations
- Marketing Art
- Future Media

The application is responsible for:

- Storage
- Versioning
- Review
- Publication
- Metadata
- Card Layout
- Sprite Metadata
- Asset Relationships
- Validation
- Security
- Audit History

AI never owns gameplay data.

---

# Core Architectural Principles

Every subsystem follows these principles.

## Deterministic Infrastructure

Given identical inputs:

- Same prompt
- Same workflow
- Same configuration
- Same metadata
- Same source assets

the system should produce identical deterministic outputs everywhere outside image generation.

---

## Portrait Authority

Every visual object possesses exactly one approved portrait.

That portrait represents the canonical visual identity.

Everything else derives from it.

```
Portrait

↓

Full Body

↓

Card Art

↓

Sprites

↓

Published Assets
```

Changing the portrait invalidates downstream assets.

---

## Human Approval

No AI-generated asset becomes production-ready without review.

The pipeline supports:

Generation

↓

Validation

↓

Review

↓

Approval

↓

Publication

There is intentionally no direct path from generation to publication.

---

## Immutable Publication

Published assets are permanent.

Corrections never overwrite.

Instead:

```
Published v1

↓

Published v2
```

Historical versions remain accessible.

---

## Metadata First

Every asset is defined primarily by metadata.

Images are outputs.

Metadata contains:

- lineage
- workflow
- prompt
- generation settings
- source references
- review history
- publication history
- version information

If an image exists without metadata, it is considered invalid.

---

## Workflow Driven

Every asset is produced by a workflow.

No image generation occurs outside the Workflow Registry.

Every workflow has:

- JSON definition
- Manifest
- Version
- Injection Schema
- Validation Rules

---

## Asset Relationships

Assets form a directed dependency graph.

Example:

```
Portrait

↓

Full Body

↓

Sprite Base

↓

Sprite Sheet

↓

Unity Runtime
```

or

```
Portrait

↓

Card Artwork

↓

Card Composition

↓

Published Card
```

Every relationship is explicitly stored.

---

# High-Level System Overview

The Asset Pipeline exists as an independent subsystem inside Wishes.

```
                    Wishes

                       │

───────────────────────┼────────────────────────

                       │

              Asset Service

                       │

         ┌─────────────┼─────────────┐

         │             │             │

     Database      Storage      Workflows

         │             │             │

         └──────┬──────┴──────┬──────┘

                │             │

           Asset Workers   Review Portal

                │

             ComfyUI

                │

        Generated Artwork

                │

          Publication Pipeline

                │

             Unity Client
```

No client communicates directly with ComfyUI.

---

# Major Components

The Asset Pipeline consists of twelve primary components.

## 1. Asset Service

Responsible for orchestration.

Owns:

- generation requests
- workflow selection
- review
- publication
- metadata
- storage

---

## 2. Workflow Registry

Maintains every supported workflow.

Responsible for:

- loading
- validation
- versioning
- compatibility

---

## 3. Prompt Builder

Constructs deterministic prompt packages.

Never manually concatenate prompts elsewhere.

Prompt Builder owns:

- Global Style
- Identity Block
- Role Prompt
- User Prompt
- Negative Prompt
- Revision Instructions

---

## 4. Asset Workers

Execute generation jobs.

Responsibilities:

- claim work
- prepare workflow
- dispatch to ComfyUI
- collect outputs
- normalize assets

Workers remain stateless.

---

## 5. ComfyUI

Creative engine.

Only responsible for image generation.

Never:

- stores metadata
- publishes assets
- manages reviews
- modifies database

---

## 6. Storage Layer

Owns:

- pending
- approved
- published
- rejected
- archive

Storage never determines asset state.

Database remains authoritative.

---

## 7. Review Portal

Human review interface.

Supports:

- comparison
- approval
- rejection
- revisions
- publication

Portrait-first workflow.

---

## 8. Card Composition Engine

Deterministically assembles cards.

AI never renders:

- text
- icons
- layout
- statistics

---

## 9. Sprite Pipeline

Produces Unity-ready tactical sprites.

Outputs:

- sprite sheets
- metadata
- animation definitions

---

## 10. Metadata System

Maintains every asset relationship.

Future sections define every schema in detail.

---

## 11. Publication System

Transforms approved assets into immutable runtime assets.

Unity only consumes published assets.

---

## 12. Unity Runtime

Final consumer.

Unity should never infer metadata from images.

Everything required for runtime is explicitly provided.

---

# Lifecycle Overview

Every asset follows the same lifecycle.

```
Requested

↓

Queued

↓

Generating

↓

Generated

↓

Technical Validation

↓

Pending Review

↓

Approved

↓

Published

↓

Archived
```

Failure may occur at any stage.

Recovery procedures are covered later in this manual.

---

# Asset Philosophy

Every asset inside Wishes belongs to one of four categories.

## Source Assets

Created by artists or imported.

Examples:

- icons
- fonts
- frames
- templates

---

## Generated Assets

Created by AI.

Examples:

- portrait
- full body
- sprite base

---

## Composed Assets

Created deterministically.

Examples:

- cards
- thumbnails
- sprite atlases

---

## Runtime Assets

Published assets consumed by Unity.

These are immutable.

---

# Terminology

Throughout this manual the following terminology is used consistently.

| Term | Definition |
|------|------------|
| Object | The game entity that owns assets. |
| Asset | A generated or deterministic file associated with an object. |
| Asset Role | Portrait, Full Body, Card Front, etc. |
| Candidate | Awaiting review. |
| Approved | Accepted for use as a source. |
| Published | Immutable runtime version. |
| Workflow | ComfyUI generation definition. |
| Manifest | Declarative workflow description. |
| Source Asset | Parent asset used during generation. |
| Lineage | Parent-child relationship chain. |
| Visual Identity | Canonical appearance defined by the approved portrait. |

---

# Document Structure

The remaining sections of this reference manual are organized as follows:

**Part II — Metadata Standards**

Defines every metadata object and schema used throughout the pipeline.

**Part III — Workflow Manifest Specification**

Defines the contract between the Asset Service and ComfyUI.

**Part IV — AI Prompt Style Guide**

Defines the canonical artistic direction of Wishes.

**Part V — Asset Review Standards**

Defines objective review criteria for every asset role.

**Part VI — Versioning Standards**

Defines how every versioned resource evolves over time.

**Part VII — Unity Runtime Integration**

Defines how Unity consumes the published asset pipeline.

**Part VIII — Technical Reference**

Defines services, APIs, storage, configuration, and implementation reference.

**Appendices**

Contain directory references, workflow catalog, API reference, troubleshooting, and cross-document mappings.

---

# Part II — Metadata Standards

## Purpose

Metadata is the foundation of the Wishes AI Asset Pipeline.

Images are considered **derived artifacts**.

Metadata is considered the authoritative representation of an asset.

Without metadata an asset does not officially exist within the Wishes ecosystem.

Every service interacts with metadata before interacting with image files.

---

# Metadata Design Goals

The metadata system is designed to achieve the following objectives:

- Deterministic asset reconstruction
- Complete historical traceability
- Workflow independence
- AI provider independence
- Version compatibility
- Future extensibility
- Immutable publication history
- Relationship tracking
- Efficient querying

---

# Metadata Hierarchy

Metadata exists at multiple layers.

```text
Object Metadata

↓

Asset Metadata

↓

Generation Metadata

↓

Workflow Metadata

↓

Prompt Metadata

↓

Review Metadata

↓

Publication Metadata

↓

Runtime Metadata
```

Each layer has a clearly defined responsibility.

---

# Metadata Ownership

| Metadata Type | Owner |
|--------------|------|
| Object | Game Database |
| Asset | Asset Service |
| Workflow | Workflow Registry |
| Prompt | Prompt Builder |
| Review | Review Portal |
| Publication | Publication Service |
| Runtime | Unity Export Pipeline |

No subsystem should modify metadata owned by another subsystem.

---

# Core Metadata Principles

## Principle 1 — Images Are Disposable

An image can always be regenerated.

Metadata cannot.

The pipeline assumes:

```text
Metadata

↓

Generation

↓

Image
```

Never:

```text
Image

↓

Guess Metadata
```

---

## Principle 2 — Immutable History

Historical metadata is never overwritten.

Instead:

```text
Version 1

↓

Version 2

↓

Version 3
```

Every revision remains queryable.

---

## Principle 3 — Explicit Relationships

Relationships are never inferred.

Every dependency is stored explicitly.

Example:

```text
Portrait UUID

↓

Full Body UUID

↓

Sprite UUID

↓

Card UUID
```

---

## Principle 4 — Provider Agnostic

Metadata must never depend upon:

- Flux
- SDXL
- OpenAI
- Future AI providers

Instead metadata stores:

```text
Workflow

↓

Manifest

↓

Provider

↓

Settings
```

Changing providers should not require schema changes.

---

# Canonical Metadata Categories

The following metadata categories exist throughout the pipeline.

## Object Metadata

Describes the owning entity.

Example:

```json
{
  "object_uuid": "...",
  "object_type": "card",
  "name": "Lyra Aetherwind"
}
```

Object metadata is owned by the game.

---

## Asset Metadata

Defines the asset itself.

Example:

```json
{
  "asset_uuid": "...",
  "asset_role": "portrait",
  "status": "approved",
  "version": 3
}
```

This is the primary metadata object.

---

## Workflow Metadata

Defines how the asset was created.

Example:

```json
{
  "workflow_code": "portrait_generate",
  "workflow_version": 4,
  "manifest_version": 2
}
```

---

## Prompt Metadata

Stores the complete deterministic prompt package.

Example:

```json
{
  "global_style": "...",
  "identity_block": "...",
  "role_prompt": "...",
  "negative_prompt": "...",
  "seed": 123456
}
```

Prompt metadata should always preserve the exact generation request.

---

## Generation Metadata

Stores execution information.

Example:

```json
{
  "generator": "ComfyUI",
  "provider": "Flux",
  "started_at": "...",
  "completed_at": "...",
  "duration_ms": 84512
}
```

---

## Review Metadata

Tracks every review decision.

Example:

```json
{
  "reviewer_uuid": "...",
  "decision": "approved",
  "reason": "Identity preserved",
  "reviewed_at": "..."
}
```

Review history is append-only.

---

## Publication Metadata

Defines runtime publication.

Example:

```json
{
  "published": true,
  "published_version": 2,
  "published_at": "...",
  "published_by": "..."
}
```

---

## Runtime Metadata

Consumed by Unity.

Example:

```json
{
  "runtime_version": 1,
  "asset_uri": "...",
  "metadata_version": 5
}
```

Unity should never consume internal generation metadata.

---

# Canonical Asset Metadata Schema

Every asset shares the same root schema.

```json
{
  "asset_uuid": "",
  "object_uuid": "",
  "object_type": "",
  "asset_role": "",
  "status": "",
  "version": 1,
  "created_at": "",
  "updated_at": ""
}
```

This schema should remain stable across future versions.

---

# Asset Status

Valid values:

```text
pending

approved

published

rejected

archived

deleted (logical only)
```

Published assets remain immutable.

---

# Asset Role

Examples:

```text
portrait

full_body

icon

thumbnail

card_front

sprite_base

sprite_sheet

animation_preview

marketing_art
```

Roles originate from the Asset Role registry.

Never hardcode role names.

---

# Object Types

Current object types:

```text
card

template

npc

user

deck

system
```

Future object types may be added without schema changes.

---

# Source Relationships

Every generated asset should reference its immediate parents.

Example:

```json
{
  "sources": [
    {
      "asset_uuid": "...",
      "role": "portrait"
    }
  ]
}
```

Never duplicate the entire ancestry.

The dependency graph provides transitive relationships.

---

# Dependency Metadata

Dependencies define asset lineage.

Example:

```json
{
  "parent_asset_uuid": "...",
  "child_asset_uuid": "...",
  "relationship": "generated_from"
}
```

Supported relationship types:

```text
generated_from

derived_from

revision_of

published_from

references
```

---

# Version Metadata

Each asset contains version information.

```json
{
  "version": 3,
  "major_version": 1,
  "minor_version": 2
}
```

Version numbers are independent of workflow versions.

---

# Review Metadata Schema

Review records are append-only.

Example:

```json
{
  "review_uuid": "...",
  "asset_uuid": "...",
  "reviewer_uuid": "...",
  "decision": "approved",
  "notes": "...",
  "created_at": "..."
}
```

A review is a historical event.

It should never be modified after creation.

---

# Publication Metadata Schema

Published assets receive publication metadata.

```json
{
  "publication_uuid": "...",
  "asset_uuid": "...",
  "published_version": 5,
  "published_at": "...",
  "published_by": "...",
  "runtime_uri": "..."
}
```

Publication metadata links review history to runtime assets.

---

# Prompt Package Schema

Every generation stores the exact prompt package.

```json
{
  "style": "...",
  "identity": "...",
  "role": "...",
  "user": "...",
  "negative": "...",
  "revision": "...",
  "seed": 0
}
```

This package must be sufficient to reproduce the original generation request.

---

# Workflow Execution Metadata

Execution metadata captures runtime behavior.

```json
{
  "job_uuid": "...",
  "worker_uuid": "...",
  "workflow_uuid": "...",
  "started_at": "...",
  "completed_at": "...",
  "status": "completed"
}
```

Future additions may include GPU identifiers and execution nodes.

---

# Runtime Metadata Separation

Internal metadata should never be shipped to Unity.

Only export:

- Runtime URI
- Asset Version
- Runtime Metadata Version
- Sprite Metadata
- Card Composition Metadata

Internal prompts, review notes, and workflow execution details remain server-side.

---

# Metadata Validation

Every metadata object should validate against a schema before persistence.

Validation includes:

- Required fields
- Data types
- Enum values
- UUID format
- Version compatibility
- Foreign key existence

Invalid metadata must never be stored.

---

# Metadata Evolution

Metadata schemas will evolve.

Every schema should include:

```json
{
  "schema_version": 1
}
```

Future migrations should upgrade metadata without modifying historical meaning.

Backward compatibility is preferred whenever possible.

---

# Metadata Storage Strategy

Metadata should be stored in PostgreSQL as JSONB where flexibility is required, while commonly queried fields should also exist as relational columns for indexing and performance.

Examples of indexed relational fields:

- asset_uuid
- object_uuid
- asset_role
- status
- version
- created_at

Complex provider-specific metadata belongs in JSONB.

---

# Metadata Integrity

Metadata integrity requirements:

- No orphaned assets
- No circular dependencies
- Every published asset references an approved source
- Every generation references a workflow
- Every workflow references a manifest
- Every review references an existing asset

Integrity validation should execute automatically during publication.

---

# Cross References

This section is directly related to:

- Document 02 — Database & Storage
- Document 03 — Workflow Registry
- Document 04 — Backend API
- Document 06 — Character Consistency
- Document 08 — Card Composition
- Document 10 — Validation
- Document 13 — Configuration

Future sections will build upon these metadata definitions.

---

# Part III — Workflow Manifest Specification

## Purpose

The Workflow Manifest is the contract between the Asset Service and the AI generation engine.

While a ComfyUI workflow describes *how* an image is generated, the manifest describes *how the Wishes platform understands and interacts with that workflow.*

The Asset Service should never inspect or modify workflow JSON directly.

Instead, it should rely exclusively on the Workflow Manifest.

This abstraction layer allows:

- Workflow replacement
- AI provider replacement
- Version compatibility
- Automatic validation
- Deterministic parameter injection
- Future multi-provider support

---

# Manifest Philosophy

A workflow is implementation.

A manifest is interface.

The workflow may change.

The manifest should remain stable.

Think of the manifest exactly like an API contract.

```
Asset Service

↓

Workflow Manifest

↓

Workflow JSON

↓

ComfyUI

↓

Generated Image
```

---

# Why Manifests Exist

Without manifests:

Application code becomes tightly coupled to ComfyUI node IDs.

Example:

```
Node 52

↓

Prompt

Node 17

↓

Width

Node 31

↓

Height
```

If node IDs change:

Entire application breaks.

Instead:

```
Prompt

↓

Manifest

↓

Workflow Injection

↓

ComfyUI
```

Only the manifest changes.

The application remains unchanged.

---

# Manifest Responsibilities

A manifest defines:

- Workflow identity
- Compatible versions
- Supported asset role
- Supported source roles
- Required inputs
- Optional inputs
- Injection points
- Output nodes
- Validation rules
- Default parameters

It never defines generation logic.

---

# Workflow Package

Every workflow consists of:

```text
Workflow JSON

+

Workflow Manifest

+

Documentation

+

Version
```

Example:

```text
portrait_generate.json

portrait_generate.manifest.json

README.md
```

These four files represent one logical workflow.

---

# Manifest Structure

Every manifest should contain:

```text
Identity

Compatibility

Inputs

Outputs

Injection Map

Validation Rules

Defaults

Metadata
```

---

# Canonical Manifest Schema

Example:

```json
{
  "workflow_code": "portrait_generate",
  "display_name": "Portrait Generation",
  "workflow_version": 3,
  "manifest_version": 1,
  "provider": "comfyui",
  "enabled": true
}
```

These fields uniquely identify the workflow.

---

# Workflow Identity

Every workflow receives a permanent code.

Example:

```text
portrait_generate

portrait_refine

full_body_generate

sprite_base

sprite_sheet

card_art

icon_generate

thumbnail_generate
```

Workflow codes should never change.

Display names may change.

---

# Provider Definition

Current:

```text
comfyui
```

Future:

```text
flux

sdxl

openai

kontext

custom
```

The provider defines execution strategy.

The Asset Service should not assume ComfyUI forever.

---

# Compatibility

Each workflow defines compatibility.

Example:

```json
{
  "compatible_roles": [
    "portrait"
  ],
  "compatible_sources": [
    "none"
  ]
}
```

Portrait generation requires no source asset.

---

Full Body:

```json
{
  "compatible_sources": [
    "portrait"
  ]
}
```

---

Sprite Base:

```json
{
  "compatible_sources": [
    "full_body"
  ]
}
```

The Asset Service validates compatibility before queueing work.

---

# Input Definitions

Every required input is declared.

Example:

```json
{
  "inputs": [
    {
      "name": "prompt",
      "type": "string",
      "required": true
    }
  ]
}
```

Supported types:

```text
string

number

boolean

image

asset_reference

json
```

---

# Optional Inputs

Example:

```json
{
  "name": "negative_prompt",
  "required": false
}
```

Missing optional inputs should fall back to defaults.

---

# Source Asset Definitions

A workflow may require assets.

Example:

```json
{
  "required_source_roles": [
    "portrait"
  ]
}
```

Multiple:

```json
[
  "portrait",
  "equipment"
]
```

Future support:

Layer compositing.

---

# Injection Mapping

This is the most important part of the manifest.

Instead of:

```
Node 21

↓

Prompt
```

Manifest defines:

```json
{
  "prompt": {
    "node": "PositivePrompt",
    "field": "text"
  }
}
```

Node names should be stable.

Never depend on numeric IDs.

---

# Supported Injection Types

```text
Prompt

Negative Prompt

Width

Height

Seed

Steps

CFG

Sampler

Scheduler

Model

LoRA

Image Input

Mask

ControlNet
```

Future providers may add more.

---

# Default Parameters

Defaults belong in the manifest.

Example:

```json
{
  "defaults": {
    "width": 1024,
    "height": 1536,
    "cfg": 7,
    "steps": 30
  }
}
```

Avoid hardcoding defaults in application code.

---

# Validation Rules

The manifest declares validation.

Example:

```json
{
  "validation": {
    "requires_identity_block": true,
    "requires_negative_prompt": true
    }
}
```

Startup validation should reject invalid manifests.

---

# Output Definitions

Every workflow must define outputs.

Example:

```json
{
  "outputs": [
    {
      "role": "portrait",
      "node": "SaveImage"
    }
  ]
}
```

Multiple outputs:

```text
Image

Mask

Preview

Metadata
```

Future workflows may produce multiple assets.

---

# Runtime Metadata

Each workflow publishes runtime metadata.

Example:

```json
{
  "estimated_duration_ms": 60000,
  "gpu_required": true,
  "supports_batch": false
}
```

Used by scheduling.

---

# Scheduling Metadata

Future:

```json
{
  "priority": "normal",
  "memory_estimate_mb": 12000,
  "gpu_class": "high"
}
```

Allows intelligent dispatch.

---

# Manifest Versioning

Workflow Version:

Describes workflow implementation.

Manifest Version:

Describes interface.

Changing node names:

Manifest version changes.

Changing generation quality:

Workflow version changes.

---

# Workflow Validation

During startup:

Every manifest validates:

✓ JSON syntax

✓ Required fields

✓ Compatible workflow

✓ Injection map

✓ Outputs

✓ Defaults

✓ Version

Failure prevents startup.

---

# Runtime Validation

Before execution:

Validate:

Workflow enabled

↓

Manifest exists

↓

Version supported

↓

Source assets valid

↓

Injection complete

↓

Required outputs defined

---

# Registry Loading

Workflow Registry loads:

```
Manifest

↓

Workflow

↓

Validation

↓

Cache

↓

Ready
```

The registry never exposes invalid workflows.

---

# Disabled Workflows

Workflows may be disabled.

Example:

```json
{
  "enabled": false
}
```

Disabled workflows:

Remain versioned.

Cannot receive new jobs.

Historical jobs remain valid.

---

# Workflow Replacement

To replace a workflow:

1. Create new workflow.
2. Create new manifest.
3. Increment workflow version.
4. Validate.
5. Enable.
6. Disable old version if desired.

Never overwrite existing versions.

---

# Workflow Discovery

Future:

Workflow Registry scans:

```text
workflows/

↓

manifest

↓

workflow

↓

validation

↓

registration
```

Automatic discovery allows plugins.

---

# Manifest Security

Treat manifests as trusted configuration.

Requirements:

- Version controlled
- Code reviewed
- Immutable in production
- Never modified by runtime

Future:

Digital signatures.

---

# Manifest Directory Structure

```text
workflow-manifests/

portrait_generate/

manifest.json

README.md

examples/

tests/
```

Grouping manifests by workflow simplifies maintenance.

---

# Best Practices

✔ Stable workflow codes

✔ Stable injection names

✔ Version everything

✔ Validate at startup

✔ Never expose node IDs

✔ Never embed business logic

✔ Keep provider-specific data isolated

---

# Common Failure Cases

Examples:

Missing output node

↓

Reject startup.

Missing injection

↓

Reject workflow.

Wrong asset role

↓

Reject request.

Unknown workflow version

↓

Reject execution.

---

# Cross References

Related documents:

- Document 03 — ComfyUI Workflow Specification
- Document 04 — Backend API
- Document 10 — Validation & QA
- Document 13 — Configuration
- Document 18 — Security Architecture

The next section of this reference manual defines the canonical AI Prompt Style Guide used by every Wishes asset generation workflow.

---

# Part IV — AI Prompt Style Guide

## Purpose

The AI Prompt Style Guide defines the artistic language of the Wishes universe.

Unlike technical configuration, prompts establish the visual identity of the game.

The goal of this guide is to ensure that every AI-generated asset—regardless of when it is created or which model generates it—appears to belong to the same universe.

The Prompt Builder should construct prompts using these standards rather than relying on ad-hoc prompt engineering.

---

# Artistic Philosophy

The Wishes universe should present itself as a timeless fantasy world that feels believable, lived-in, and grounded.

Characters should appear as individuals rather than generic fantasy archetypes.

Magic should feel ancient rather than futuristic.

Technology should feel handcrafted rather than industrial.

The world should prioritize:

- Identity
- Personality
- Readability
- Emotional expression
- Consistency
- World cohesion

Rather than:

- Excessive realism
- Hyper-stylization
- Trend-following aesthetics
- Overly saturated visuals
- AI-generated clichés

---

# Global Style Statement

Every generated asset should feel like it belongs to the same illustrated fantasy universe.

The visual language should communicate:

- Noble craftsmanship
- Ancient civilizations
- Natural beauty
- Mysticism
- Adventure
- Hope
- Discovery
- History

Every image should appear intentional.

---

# Artistic Pillars

## Identity First

The most important goal is preserving identity.

A player should immediately recognize:

- Lyra
- Brakk
- Aric

regardless of whether they are viewing:

- Portrait
- Full Body
- Card
- Sprite
- Marketing Art
- Story Illustration

Identity is more important than artistic variation.

---

## Readability

Images should remain readable.

Avoid:

- clutter
- excessive particles
- unnecessary background objects
- overly dramatic lighting
- confusing silhouettes

Characters should remain the primary subject.

---

## Consistency

Every asset should preserve:

Hair

↓

Eyes

↓

Body Type

↓

Race

↓

Outfit

↓

Weapon

↓

Primary Colors

↓

Silhouette

Consistency outweighs novelty.

---

# Artistic Direction

The Wishes universe should exist between realism and stylized illustration.

Reference qualities:

- Hand-painted fantasy illustration
- Modern RPG concept art
- Premium trading card artwork
- Storybook realism

Avoid:

- Cartoon exaggeration
- Anime proportions (unless intentionally specified)
- Hyperreal photography
- Oil painting texture overload
- Comic-book rendering

---

# Lighting Philosophy

Lighting should support storytelling.

Preferred lighting:

Soft natural daylight

Golden hour

Magical ambient glow

Moonlight

Interior torchlight

Temple illumination

Avoid:

Harsh flash lighting

Extreme HDR

High-contrast studio lighting

Unmotivated rim lighting

---

# Color Philosophy

Every character should possess a dominant palette.

Example:

Lyra

```text
Blue

Silver

White
```

Brakk

```text
Brown

Iron

Crimson
```

Aric

```text
Green

Leather

Steel
```

Future generated assets should preserve this palette.

---

# Palette Rules

Avoid:

Random clothing colors

Random armor colors

Unrelated magical effects

The dominant palette should remain recognizable.

Accent colors may change.

Primary colors should not.

---

# Character Composition

Portraits should generally use:

- Chest-up
- Three-quarter orientation
- Natural posture
- Neutral expression unless specified
- Eyes visible
- Face unobstructed

Avoid:

Extreme perspective

Fish-eye distortion

Aggressive cropping

Hidden facial features

---

# Facial Expression Standards

Default portrait expression:

Confident

Calm

Approachable

Future supported expressions:

Happy

Determined

Curious

Sad

Angry

Fearful

Victorious

Exhausted

Expressions should remain subtle.

Avoid exaggerated emotion.

---

# Hair Standards

Hair is a primary identity marker.

Maintain:

Length

Style

Parting

Texture

Primary Color

Secondary Highlights

Never randomly alter hairstyles between generations.

---

# Eyes

Eyes communicate personality.

Preserve:

Shape

Color

Brightness

Magical characteristics

Future metadata may store:

```text
Eye Shape

Primary Color

Secondary Color

Glow

Unique Traits
```

---

# Race Characteristics

Race-specific traits should remain consistent.

Examples:

Elf

- ears
- elegance
- refined proportions

Dwarf

- stockier frame
- facial hair
- heavier features

Human

- balanced anatomy

Race identity should never drift between assets.

---

# Clothing Standards

Outfits should communicate:

Profession

Culture

Status

Lifestyle

Experience

Equipment

Avoid decorative clutter without narrative purpose.

Every visible item should appear intentional.

---

# Armor Standards

Armor should appear functional.

Preferred:

Layered construction

Believable straps

Visible joints

Logical materials

Avoid:

Floating armor

Impossible construction

Excessive ornamentation

---

# Weapon Standards

Weapons should remain identifiable.

A signature weapon should not change shape between generations.

Weapon scale should remain believable.

Future metadata may define:

Weapon silhouette

Primary material

Grip style

Decorations

Elemental effects

---

# Magic Visualization

Magic should feel:

Ancient

Elegant

Controlled

Mystical

Avoid:

Sci-fi energy beams

Neon effects

Random particle explosions

Magic color should reinforce elemental identity.

---

# Environmental Backgrounds

Portrait backgrounds should support the subject without competing for attention.

Preferred:

Soft blur

Painterly scenery

Architectural hints

Natural environments

Avoid:

Crowded cities

Complex battle scenes

Large numbers of secondary characters

The portrait remains the focus.

---

# Background Removal

For production assets:

Portraits should support:

Transparent background

Neutral background

Simple painterly background

Complex scenic backgrounds should generally be avoided unless the asset role specifically requires them.

---

# Camera Standards

Preferred portrait angles:

Front

Three-quarter left

Three-quarter right

Eye level

Avoid:

Top-down

Extreme low angle

Dutch angles

Unmotivated cinematic shots

---

# Full Body Standards

Every full body should include:

Complete silhouette

Visible feet

Visible hands

Equipment

Accessories

Ground reference

Balanced stance

Avoid cropped limbs.

---

# Tactical Sprite Style

Sprites should emphasize:

Silhouette

Contrast

Readability

Simple color grouping

Animation clarity

Not facial detail.

---

# Card Artwork Style

Card artwork should feel cinematic while preserving identity.

The subject remains dominant.

Background supports narrative.

Effects should not obscure the character.

---

# Icon Style

Icons should:

Simplify

Clarify

Remain recognizable at small sizes

Avoid tiny decorative details.

---

# Creature Style

Creatures should appear biologically believable within the Wishes universe.

Avoid:

Random spikes

Unmotivated asymmetry

Chaotic anatomy

Creature design should suggest ecology.

---

# Architecture

Architecture should communicate culture.

Examples:

Elves

Elegant

Organic

Vertical

Dwarves

Massive

Stone

Geometric

Human Kingdoms

Practical

Layered

Historic

Architecture should reinforce worldbuilding.

---

# Visual Hierarchy

Every composition should naturally guide the viewer.

Priority:

Face

↓

Eyes

↓

Weapon

↓

Hands

↓

Costume

↓

Background

The eye should never become lost.

---

# Prompt Structure

The Prompt Builder should assemble prompts using deterministic layers.

```text
Global Style

↓

World Style

↓

Role Style

↓

Identity Block

↓

Subject

↓

Pose

↓

Environment

↓

Lighting

↓

Composition

↓

Quality

↓

Negative Prompt
```

Each layer remains independently configurable.

---

# Negative Prompt Standards

Every workflow should include a standardized negative prompt.

Avoid:

Low quality

Extra limbs

Extra fingers

Blur

Duplicate subjects

Cropped anatomy

Incorrect weapons

Incorrect clothing

Identity drift

Poor composition

Background clutter

Low contrast

The negative prompt should evolve independently of the positive prompt.

---

# Prompt Evolution

Prompt improvements should occur through versioning.

Never silently change historical prompts.

Prompt revisions should include:

Reason

↓

Reviewer

↓

Version

↓

Effective Date

---

# Model Independence

This style guide intentionally avoids provider-specific syntax.

The same artistic intent should translate across:

Flux

↓

SDXL

↓

Future Models

Only the Prompt Builder adapts wording to provider capabilities.

---

# Acceptance Criteria

The Prompt Style Guide is considered successful when:

1. Every generated asset clearly belongs to the Wishes universe.
2. Character identity remains stable across all asset roles.
3. Color palettes remain consistent.
4. Race, clothing, weapons, and silhouettes remain recognizable.
5. Prompt construction remains deterministic.
6. Artistic direction remains independent of the underlying AI model.
7. Future prompt revisions can be introduced through versioning without invalidating historical assets.

---

# Part V — Asset Review Standards

## Purpose

The Review System is the quality control mechanism of the Wishes AI Asset Pipeline.

Every AI-generated asset must pass through human review before becoming part of the canonical Wishes universe.

Review is not intended to judge artistic preference.

Its purpose is to verify that every asset satisfies the technical, visual, and canonical standards established throughout the Wishes documentation.

Reviewers act as custodians of the Wishes visual identity.

---

# Review Philosophy

The Wishes review system is built upon four principles.

## Identity Before Beauty

An image that perfectly preserves character identity but is slightly less impressive artistically is preferred over an image that is visually stunning but depicts the wrong character.

Identity is always the highest priority.

---

## Consistency Before Novelty

Generated assets should reinforce the existing visual language.

Unexpected artistic variation should be rejected unless it represents an intentional design evolution.

---

## Technical Quality Before Style

An asset cannot be approved if it contains technical defects regardless of artistic merit.

Examples:

- Incorrect anatomy
- Cropped limbs
- Missing fingers
- Broken equipment
- AI artifacts

---

## Human Authority

Artificial Intelligence proposes.

Humans approve.

No workflow should ever bypass human review.

---

# Review Lifecycle

Every generated asset follows the same review lifecycle.

```text
Queued

↓

Generating

↓

Technical Validation

↓

Pending Review

↓

Approved

↓

Published

↓

Archived
```

Rejected assets return to:

```text
Revision

↓

Generation

↓

Review
```

---

# Reviewer Responsibilities

Reviewers are responsible for verifying:

- Canonical identity
- Artistic quality
- Technical quality
- Gameplay readability
- Metadata integrity
- Publication readiness

Reviewers are not responsible for modifying prompts or workflows directly.

---

# Review Categories

Each review evaluates six independent categories.

```text
Identity

Technical

Composition

Consistency

Gameplay

Publication
```

Each category may pass or fail independently.

---

# Identity Review

Identity review verifies that the asset depicts the correct character.

Primary checklist:

Hair

Eyes

Face

Body Type

Race

Outfit

Weapon

Accessories

Color Palette

Silhouette

Personality

If multiple identity elements drift simultaneously the asset should normally be rejected.

---

# Identity Drift

Identity drift occurs whenever a generated asset deviates from the approved portrait.

Examples:

Hair color changed.

Eye color changed.

Armor replaced.

Different facial structure.

Missing race traits.

Weapon changed.

Different body proportions.

Identity drift is cumulative.

Multiple minor changes may justify rejection.

---

# Portrait Review

Portraits define canonical identity.

Portrait review is the strictest review stage.

Checklist:

✓ Eyes visible

✓ Face unobstructed

✓ Correct expression

✓ Correct hairstyle

✓ Correct colors

✓ Correct anatomy

✓ Balanced lighting

✓ Background appropriate

✓ High readability

Reject:

Hidden face

Incorrect race

Incorrect gender

Wrong age

Missing identity

AI artifacts

---

# Full Body Review

Full body assets derive from the approved portrait.

Verify:

Entire body visible

Hands visible

Feet visible

Correct proportions

Correct equipment

Correct clothing

Correct posture

Correct silhouette

Reject:

Missing limbs

Incorrect proportions

Weapon changes

Incorrect footwear

Identity drift

---

# Outfit Review

Outfits should communicate:

Profession

Culture

Status

Role

Environment

History

Review:

Materials

Construction

Color

Functionality

Wear

Damage

Reject:

Random armor

Floating objects

Impossible clothing

Visual inconsistency

---

# Weapon Review

Weapons are major identity markers.

Verify:

Shape

Material

Scale

Grip

Decorations

Elemental effects

Weapons should remain recognizable across:

Portrait

Full Body

Card

Sprite

Marketing Art

---

# Color Review

Review palette consistency.

Primary colors should remain stable.

Minor lighting variations are acceptable.

Reject:

Completely different palette

Unexpected neon colors

Oversaturation

Incorrect faction colors

---

# Composition Review

Composition evaluates image structure.

Review:

Subject placement

Balance

Cropping

Perspective

Visual hierarchy

Depth

Focus

Background complexity

The character should remain the dominant subject.

---

# Lighting Review

Lighting should:

Support mood

Preserve readability

Reveal facial features

Highlight equipment

Avoid obscuring identity.

Reject:

Overexposed

Underexposed

Harsh flash lighting

Unmotivated dramatic effects

---

# Background Review

Backgrounds should support the subject.

Preferred:

Simple

Painterly

Soft focus

Atmospheric

Reject:

Distracting scenery

Crowded environments

Competing characters

Busy visual noise

---

# Technical Review

Technical validation confirms image quality.

Checklist:

Correct dimensions

Correct aspect ratio

No artifacts

No duplicated limbs

No malformed anatomy

Correct resolution

Transparent background (when required)

Metadata exists

Correct file format

---

# Anatomy Review

Review:

Hands

Fingers

Eyes

Feet

Arms

Legs

Shoulders

Neck

Jaw

Hairline

Reject:

Extra limbs

Missing fingers

Broken joints

Impossible anatomy

AI distortions

---

# Card Artwork Review

Card artwork should satisfy:

Identity

↓

Composition

↓

Readability

↓

Visual Impact

↓

Gameplay Clarity

The card should remain recognizable at gameplay scale.

---

# Card Composition Review

Review deterministic rendering.

Verify:

Frame

Icons

Typography

Text alignment

Safe margins

Stat placement

Element icons

Quality frame

Reject:

Clipped text

Incorrect icons

Incorrect stats

Layout overlap

---

# Icon Review

Icons should remain readable at:

64 px

48 px

32 px

16 px

Reject:

Unreadable silhouettes

Tiny details

Color confusion

Poor contrast

---

# Thumbnail Review

Verify:

Character recognizable

Correct crop

Good contrast

Readable at small size

---

# Tactical Sprite Review

Sprites emphasize gameplay readability.

Review:

Silhouette

Animation

Color grouping

Pose clarity

Scale consistency

Reject:

Unreadable silhouettes

Frame jitter

Identity drift

Broken animations

---

# Animation Review

Review:

Idle

Walk

Attack

Cast

Hit

Down

Each animation should clearly communicate intent.

Avoid unnecessary complexity.

---

# Sprite Metadata Review

Verify:

Tile size

Frame count

Animation definitions

Directions

Metadata JSON

Runtime compatibility

Unity import validation

---

# Marketing Artwork Review

Marketing artwork may use more dramatic composition.

Still preserve:

Identity

Color

Weapons

Outfit

Race

Silhouette

Marketing art should never redefine a character.

---

# Consistency Review

Compare against:

Approved portrait

↓

Approved full body

↓

Published card

↓

Published sprite

Every approved asset should reinforce previous approvals.

---

# Review Decisions

Every review results in one of five decisions.

```text
Approve

Approve With Notes

Revision Requested

Reject

Archive
```

---

# Revision Requests

A revision should include:

Problem

↓

Suggested correction

↓

Priority

↓

Reviewer

↓

Date

Avoid vague comments.

Good:

"Hair color shifted toward gold. Preserve original silver palette."

Bad:

"Looks wrong."

---

# Scoring System (Optional)

Future implementation may score:

Identity

0–100

Technical

0–100

Composition

0–100

Readability

0–100

Overall

Weighted average

Human decision always overrides automated scoring.

---

# Automated Assistance

Future AI tools may assist reviewers by detecting:

Identity drift

Color drift

Anatomy issues

Prompt differences

Metadata inconsistencies

These systems assist.

They do not approve.

---

# Publication Readiness Checklist

Before publication verify:

✓ Approved review

✓ Metadata valid

✓ Workflow recorded

✓ Source assets approved

✓ Runtime export complete

✓ Dependency graph valid

✓ Version assigned

✓ Audit history complete

---

# Review Audit Trail

Every review records:

Reviewer

Timestamp

Decision

Notes

Version

Review duration

Previous decision

All review history is permanent.

---

# Common Rejection Reasons

Identity drift

Incorrect anatomy

Wrong equipment

Poor composition

Unreadable sprite

Metadata failure

Workflow mismatch

Background clutter

Incorrect colors

AI artifacts

These reasons should become standardized review codes.

---

# Review Metrics

Track:

Approval rate

Revision rate

Average review time

Most common rejection reasons

Workflow quality

Reviewer agreement

These metrics improve future workflows.

---

# Acceptance Criteria

The Wishes Review System is successful when:

1. Every published asset has passed human review.
2. Identity remains consistent throughout the asset lineage.
3. Technical defects are eliminated before publication.
4. Review history remains immutable.
5. Review decisions are traceable and reproducible.
6. Automated review tools assist rather than replace reviewers.
7. Published assets represent the highest canonical visual quality achievable.

---

# Part VI — Versioning Standards

## Purpose

Versioning is one of the most important systems within the Wishes AI Asset Pipeline.

The objective of versioning is to ensure that every asset, workflow, template, prompt, and metadata object can evolve without breaking historical compatibility.

Nothing should ever be silently replaced.

Every meaningful change should produce a new version.

Every version should remain reproducible.

---

# Versioning Philosophy

The Wishes Asset Pipeline treats every generated artifact as historical.

History is never destroyed.

Instead of replacing assets, the system creates new revisions while preserving lineage.

The following principles apply throughout the platform:

- Published assets are immutable.
- Historical versions remain queryable.
- Every version has a reason.
- Every version has an audit trail.
- Every version references its predecessor.

---

# Version Categories

The Wishes Asset Pipeline contains several independent version systems.

```text
Asset Versions

Workflow Versions

Manifest Versions

Prompt Versions

Template Versions

Metadata Schema Versions

Publication Versions

Configuration Versions

API Versions
```

Each evolves independently.

---

# Asset Versioning

Every asset receives a version number.

Example:

```text
Portrait

v1

↓

v2

↓

v3
```

Version numbers are unique within an asset lineage.

Replacing a portrait never modifies Version 1.

Instead:

```text
Version 1

Archived

↓

Version 2

Approved

↓

Version 3

Published
```

---

# Asset States Across Versions

Only one version may be considered the active approved version at a time.

Example:

| Version | Status |
|----------|--------|
| v1 | Archived |
| v2 | Archived |
| v3 | Approved |
| v4 | Pending Review |

The review system determines when the active version changes.

---

# Major vs Minor Versions

Recommended convention:

Major Version

Visual identity changed.

Examples:

- New portrait
- New costume
- Character redesign

Minor Version

Quality improvement.

Examples:

- Better lighting
- Improved anatomy
- Artifact removal

Example:

```text
Portrait

1.0

↓

1.1

↓

1.2

↓

2.0
```

---

# Publication Versions

Publication versions are independent from asset versions.

Example:

Asset Version

```text
3
```

Published Runtime Version

```text
8
```

Multiple publications may reference the same approved asset.

Example:

Different export profiles.

---

# Workflow Versioning

Workflow versions describe generation logic.

Examples of workflow changes:

- Different node graph
- Different sampler
- Better ControlNet
- Different LoRA
- Better composition

Workflow versions never modify historical outputs.

Old assets continue referencing the workflow version that created them.

---

# Manifest Versioning

Manifest versions describe interface compatibility.

Changing:

- Node names
- Injection schema
- Output names

requires a new manifest version.

Changing generation quality alone does not.

---

# Prompt Versioning

Prompt packages are versioned independently.

Example:

```text
Prompt Library

v1

↓

v2

↓

v3
```

Historical prompts remain stored with every asset.

The Prompt Builder always records:

- Prompt version
- Global style version
- Negative prompt version

---

# Template Versioning

Templates include:

Card Templates

Sprite Layouts

Frame Layouts

Export Profiles

Every template receives:

```text
Template Code

↓

Template Version
```

Example:

```text
character_card

v1

↓

v2
```

Existing cards remain reproducible.

---

# Metadata Schema Versioning

Every metadata object contains:

```json
{
  "schema_version": 1
}
```

Schema evolution should favor additive changes.

Avoid breaking existing metadata whenever possible.

---

# API Versioning

Recommended approach:

```text
/api/v1/

/api/v2/
```

Avoid breaking existing clients.

Deprecate gradually.

---

# Configuration Versioning

Configuration schemas evolve.

Example:

```json
{
  "configuration_version": 4
}
```

Migration scripts should automatically update configuration where practical.

---

# Database Versioning

Database schema version equals migration version.

Never manually edit production schemas.

Every schema change must:

- Create migration
- Update documentation
- Update validation
- Update reference manual

---

# Storage Versioning

Published storage should be version aware.

Example:

```text
published/

card/

uuid/

v1/

v2/

v3/
```

No version overwrites another.

---

# Workflow Compatibility Matrix

Each workflow should declare compatibility.

Example:

```text
Workflow

↓

Manifest

↓

Prompt Version

↓

Template Version
```

The Workflow Registry validates compatibility before execution.

---

# Dependency Versioning

Child assets reference exact parent versions.

Example:

```text
Portrait v3

↓

Full Body v2

↓

Sprite v5
```

Never reference "latest."

Always reference an explicit version.

---

# Runtime Version Resolution

Unity should always load:

Latest Published Runtime Version.

Not:

Latest Approved Version.

This prevents accidental runtime changes.

---

# Historical Reconstruction

The system should be capable of reconstructing any published asset.

Required inputs:

- Workflow Version
- Manifest Version
- Prompt Version
- Template Version
- Source Asset Versions
- Configuration Version

This enables deterministic historical recreation.

---

# Revision Chains

Every revision stores:

```text
Previous Version UUID

↓

Current Version UUID
```

Revision chains should never branch silently.

Future support may allow experimental branches.

---

# Version Retirement

Versions should not be deleted.

Instead:

```text
Deprecated

↓

Archived

↓

Retained
```

Only logical deletion is permitted.

---

# Version Migration

When introducing a new version:

1. Create new version.
2. Validate compatibility.
3. Migrate metadata if required.
4. Publish documentation.
5. Preserve previous versions.

Never modify historical version data.

---

# Semantic Versioning Recommendation

Recommended format:

```text
Major.Minor.Patch
```

Major

Breaking identity change.

Minor

Quality improvement.

Patch

Metadata or technical correction.

---

# Version Auditing

Every version records:

Created By

Created At

Reason

Source Version

Approval

Publication

Workflow

Prompt

Template

Reviewer

This information becomes permanent history.

---

# Future Branching

Future support may allow:

```text
Portrait

↓

Main Branch

↓

Seasonal Branch

↓

Holiday Branch

↓

Corrupted Branch

↓

Ascended Branch
```

Each branch remains independently versioned.

---

# Rollback Strategy

Rollback never deletes versions.

Instead:

```text
Current

↓

Previous Published Version

↓

Republish
```

Historical versions remain intact.

---

# Version Validation

Before publication verify:

✓ Parent versions exist

✓ Workflow compatible

✓ Prompt compatible

✓ Metadata schema compatible

✓ Runtime export complete

✓ Dependency graph valid

---

# Cross-System Version Matrix

| Component | Versioned |
|-----------|-----------|
| Assets | Yes |
| Workflows | Yes |
| Manifests | Yes |
| Prompts | Yes |
| Templates | Yes |
| Metadata | Yes |
| Configuration | Yes |
| Runtime Export | Yes |
| API | Yes |

Every major subsystem participates in versioning.

---

# Best Practices

✔ Never overwrite.

✔ Always create new versions.

✔ Preserve history.

✔ Reference explicit versions.

✔ Document every change.

✔ Validate compatibility.

✔ Keep published assets immutable.

---

# Common Versioning Mistakes

Avoid:

Using "latest" references.

Deleting old versions.

Changing workflow behavior without incrementing version.

Replacing published assets.

Mixing workflow versions with asset versions.

Reusing version numbers.

Breaking metadata compatibility.

---

# Acceptance Criteria

The versioning system is complete when:

1. Every mutable resource has an independent version.
2. Historical versions remain reproducible.
3. Published assets are immutable.
4. Every dependency references explicit versions.
5. Rollbacks preserve history.
6. Compatibility validation prevents invalid combinations.
7. Future version evolution can occur without redesign.

---

# Part VII — Unity Runtime Integration

## Purpose

This section defines how the Unity client integrates with the Wishes AI Asset Pipeline.

The Unity client is **never responsible for generating, modifying, or interpreting AI assets** beyond their intended runtime usage.

Instead, Unity consumes deterministic assets and metadata produced by the Asset Pipeline.

This separation ensures:

- Runtime stability
- Deterministic rendering
- Asset consistency
- Simplified client implementation
- AI provider independence

---

# Design Philosophy

The Unity client should behave as a rendering engine rather than an asset authoring tool.

Unity should never:

- Generate prompts
- Execute workflows
- Contact ComfyUI
- Approve assets
- Modify metadata
- Compose cards
- Assemble sprite sheets

Unity only consumes published assets.

---

# Runtime Asset Flow

```
Designer

↓

Asset Request

↓

Generation

↓

Review

↓

Approval

↓

Publication

↓

Published Assets

↓

Unity Client
```

Unity begins at the final stage only.

---

# Runtime Asset Categories

Unity consumes the following asset types.

```text
Portrait

Full Body

Card Front

Thumbnail

Icon

Sprite Sheet

Animation Metadata

UI Assets

Future Assets
```

Every runtime asset must have metadata.

---

# Runtime Asset Principles

Unity should assume:

Published assets are immutable.

Metadata is authoritative.

Images are presentation.

Gameplay comes from the database.

---

# Asset Resolution

Unity requests assets by:

```text
Object UUID

+

Asset Role
```

Never by filename.

Example:

```text
Character UUID

↓

Portrait
```

returns

```text
Published Portrait
```

The Asset Service determines the correct file.

---

# Runtime Metadata

Unity should receive a simplified metadata object.

Example:

```json
{
  "asset_uuid": "...",
  "asset_role": "portrait",
  "version": 5,
  "uri": "...",
  "metadata_version": 2
}
```

Internal generation metadata should never be exposed.

---

# Asset Manifest

Future runtime asset bundles should include:

```json
{
  "character_uuid": "...",
  "portrait": "...",
  "full_body": "...",
  "sprite_sheet": "...",
  "card_front": "...",
  "icon": "..."
}
```

Unity loads one manifest instead of querying every asset independently.

---

# Addressable Asset Strategy

Recommended Unity implementation:

```
Published Asset

↓

Addressable Registration

↓

Runtime Lookup

↓

Memory Cache
```

Addressables should reference published assets only.

---

# Asset Loading Pipeline

Recommended runtime flow:

```
Game Object

↓

Asset Manifest

↓

Metadata

↓

Download / Cache

↓

Texture Import

↓

Runtime Object
```

The runtime should never infer missing metadata.

---

# Portrait Loading

Portraits are primarily used for:

- Dialogue
- Character Sheets
- Inventory
- Party UI
- Social Systems
- NPC Windows

Recommended loading:

```
Portrait Metadata

↓

Texture

↓

Sprite

↓

UIImage
```

---

# Full Body Loading

Full body assets support:

- Character Viewer
- Equipment Preview
- Marketing
- Lore Viewer

Future:

Paper doll systems.

---

# Card Loading

Cards are already composed.

Unity loads:

```
Card Image

+

Metadata
```

Gameplay values originate from the game database.

Never extract gameplay information from the card image.

---

# Sprite Loading

Sprite metadata defines runtime behavior.

Example:

```json
{
  "tile_width":128,
  "tile_height":128,
  "animations":[]
}
```

Unity should construct animation clips using metadata.

No hardcoded frame positions.

---

# Animation Construction

Runtime sequence:

```
Sprite Sheet

↓

Metadata

↓

Animation Builder

↓

Animation Clips

↓

Animator Controller
```

Animation clips should be generated automatically.

---

# Sprite Metadata

Sprite metadata should define:

- Tile Width
- Tile Height
- Frame Rectangles
- Directions
- Animation Names
- Frame Rates
- Loop Rules
- Pivot

Everything required by Unity.

---

# Runtime Caching

Recommended cache layers:

```
Memory

↓

Disk

↓

Published Storage
```

Frequently used assets should remain in memory.

Large assets may remain disk cached.

---

# Cache Invalidation

Published assets are immutable.

Therefore cache invalidation is simple.

New versions create new URIs.

Old assets remain cacheable.

---

# Download Strategy

Assets should support:

Lazy loading

↓

Background download

↓

Preloading

↓

Streaming

Depending on gameplay requirements.

---

# Offline Support

Offline mode should use:

Previously cached published assets.

No dependency on ComfyUI.

No dependency on generation services.

---

# CDN Integration

Future architecture:

```
Published Storage

↓

CDN

↓

Unity
```

Unity should not distinguish between local storage and CDN.

Asset URI abstraction handles this.

---

# Memory Management

Large textures should unload when unused.

Portraits:

Small memory footprint.

Sprite Sheets:

Shared across multiple instances.

Marketing assets:

Load on demand only.

---

# Compression

Recommended:

Portraits

High Quality

Cards

Lossless preferred

Sprites

Depends on chosen art style.

Avoid compression artifacts affecting readability.

---

# Texture Import Settings

Portraits:

```
Texture Type:

Sprite
```

Cards:

```
Sprite

No Mipmaps
```

Sprites:

```
Sprite Multiple
```

Import settings should be deterministic.

---

# Runtime Scaling

UI scaling should never distort assets.

Maintain aspect ratio.

Avoid stretching.

Use metadata dimensions.

---

# Localization

Future localization should affect:

Card text

Dialogue

UI

Never regenerate artwork solely for localization.

Artwork remains language independent whenever possible.

---

# Asset Bundles

Future export profiles:

```
Gameplay

UI

Marketing

Mobile

Console

Editor
```

Each profile references the same published assets.

---

# Asset Dependencies

Unity should understand dependencies.

Example:

```
Character

↓

Portrait

↓

Card

↓

Sprite
```

Future systems may preload related assets.

---

# Runtime Validation

Before loading an asset verify:

Metadata exists.

↓

URI valid.

↓

Version supported.

↓

Role correct.

↓

File available.

Gracefully handle failures.

---

# Failure Recovery

If asset unavailable:

Fallback priority:

```
Published Version

↓

Previous Published Version

↓

Placeholder Asset

↓

Error
```

Never attempt generation during gameplay.

---

# Placeholder Assets

Maintain placeholders for:

Portrait

Card

Icon

Sprite

Thumbnail

These should be visually obvious while avoiding runtime failures.

---

# Future Runtime Features

Future additions:

Dynamic skins

Equipment overlays

Runtime palette swaps

Procedural animation

Streaming assets

Cloud asset bundles

Live seasonal events

All should remain compatible with the metadata architecture.

---

# Runtime Performance Goals

Portrait load:

<100 ms cached

Card load:

<100 ms cached

Sprite import:

One-time on first load

Animation construction:

Automatic

Background downloads:

Non-blocking

---

# Runtime Security

Unity should never receive:

Prompt packages

Workflow JSON

Review history

Generation metadata

Internal configuration

Only published runtime metadata.

---

# Best Practices

✔ Metadata driven.

✔ Published assets only.

✔ Immutable assets.

✔ Addressables preferred.

✔ No hardcoded frame data.

✔ Separate gameplay from presentation.

✔ Cache aggressively.

✔ Never communicate with ComfyUI.

---

# Cross References

Related documents:

- Document 05 — Review Portal
- Document 07 — Tactical Sprite Generation
- Document 08 — Card Composition
- Document 09 — Deployment
- Document 13 — Configuration
- Document 17 — Operations Runbook

The next section transitions from runtime integration into the complete technical reference for every backend service, worker, API, and subsystem in the Wishes Asset Pipeline.

---

# Part VIII — Technical Reference

## Section 8.1 — Service Architecture

## Purpose

This section serves as the authoritative reference for every backend service that participates in the Wishes AI Asset Pipeline.

Unlike previous documents which describe implementation, this section documents the operational responsibilities of every service.

Each service should have:

- Clearly defined ownership
- Single responsibility
- Well-defined APIs
- Explicit dependencies
- Independent deployment capability
- Independent versioning
- Independent scaling

No service should own another service's business logic.

---

# Service Topology

The production topology should resemble the following architecture.

```text
                     Unity Client

                           │

                    API Gateway

                           │

        ┌──────────────────┼──────────────────┐

        │                  │                  │

World Simulation      Asset Service      Chat Service

        │                  │

        │          ┌───────┼────────┐

        │          │       │        │

        │     Review   Workers  Storage

        │                  │

        │              Workflow

        │                  │

        │              ComfyUI

        │

   PostgreSQL
```

Services communicate through authenticated APIs.

No service directly manipulates another service's internal state.

---

# Asset Service

## Purpose

The Asset Service is the orchestrator of the entire AI Asset Pipeline.

It owns:

- Asset requests
- Generation queue
- Review lifecycle
- Publication
- Metadata
- Asset lineage
- Workflow selection

It does **not** generate images itself.

---

## Responsibilities

Primary responsibilities include:

- Validate requests
- Select workflows
- Build prompt packages
- Queue jobs
- Record metadata
- Coordinate review
- Publish approved assets

---

## Explicit Non-Responsibilities

The Asset Service must never:

- Perform gameplay calculations
- Execute AI inference
- Render Unity assets
- Modify world simulation
- Interpret card rules
- Execute combat logic

---

## Public API

Suggested endpoints:

```text
POST   /api/assets/request

GET    /api/assets/{uuid}

GET    /api/assets

POST   /api/assets/{uuid}/approve

POST   /api/assets/{uuid}/reject

POST   /api/assets/{uuid}/publish

POST   /api/assets/{uuid}/archive
```

Future endpoints should remain resource-oriented.

---

# Asset Request Lifecycle

Every request follows the same sequence.

```text
Request

↓

Validation

↓

Workflow Resolution

↓

Prompt Construction

↓

Queue Creation

↓

Worker Assignment

↓

Generation

↓

Metadata Creation

↓

Review

↓

Publication
```

Every transition should be auditable.

---

# Prompt Builder Service

## Purpose

Construct deterministic prompt packages.

Prompt Builder owns prompt assembly.

No other subsystem should manually concatenate prompt strings.

---

## Responsibilities

Generate:

Global Style

↓

World Style

↓

Identity Block

↓

Role Prompt

↓

User Prompt

↓

Negative Prompt

↓

Revision Notes

↓

Provider Formatting

↓

Prompt Package

---

## Output

Produces:

```json
{
  "prompt": "...",
  "negative_prompt": "...",
  "seed": 12345
}
```

The package becomes immutable generation metadata.

---

# Workflow Registry Service

## Purpose

Maintain every supported workflow.

The registry abstracts ComfyUI from the rest of the application.

---

## Responsibilities

Load workflows.

Load manifests.

Validate compatibility.

Cache workflow definitions.

Provide lookup APIs.

Reject invalid workflows.

---

## Startup Sequence

```
Workflow Folder

↓

Manifest Discovery

↓

Validation

↓

Registration

↓

Runtime Cache
```

Startup should fail if mandatory workflows cannot be registered.

---

# Storage Service

## Purpose

Provide deterministic storage operations.

The Storage Service owns every filesystem interaction.

---

## Responsibilities

Create directories.

Normalize paths.

Move assets.

Publish assets.

Archive assets.

Delete temporary files.

Verify storage integrity.

---

## Storage Rule

Every filesystem operation must occur through Storage Service.

Never manipulate files directly elsewhere.

---

# Metadata Service

## Purpose

Generate and maintain metadata.

Metadata Service owns:

Asset Metadata

↓

Review Metadata

↓

Publication Metadata

↓

Runtime Metadata

It does not own business rules.

---

# Dependency Service

## Purpose

Maintain the asset graph.

Responsibilities:

Parent lookup.

Child lookup.

Ancestor lookup.

Descendant lookup.

Stale propagation.

Version traversal.

---

## Dependency Graph Example

```text
Portrait

↓

Full Body

↓

Sprite Base

↓

Sprite Sheet

↓

Published Runtime
```

The graph is directional.

Circular references are prohibited.

---

# Publication Service

## Purpose

Convert approved assets into runtime assets.

Publication Service owns:

Version assignment.

↓

Runtime export.

↓

Published storage.

↓

Runtime metadata.

↓

Publication audit.

---

## Publication Flow

```
Approved Asset

↓

Validation

↓

Runtime Export

↓

Published Storage

↓

Runtime Metadata

↓

Audit

↓

Complete
```

Publication is deterministic.

---

# Review Service

## Purpose

Coordinate human review.

Responsibilities:

Approve.

Reject.

Request Revision.

Archive.

Replace Portrait.

Maintain review history.

---

## Review Rules

Review Service never modifies images.

It only modifies asset state.

---

# Worker Service

Workers execute queued jobs.

Workers remain stateless.

---

## Worker Lifecycle

```
Idle

↓

Claim Job

↓

Validate

↓

Execute

↓

Collect Outputs

↓

Normalize

↓

Metadata

↓

Complete

↓

Idle
```

Workers should recover automatically after failures.

---

# Worker Responsibilities

Workers:

Inject workflows.

Submit generation.

Collect outputs.

Generate metadata.

Upload assets.

Update job status.

Workers should never:

Approve assets.

Publish assets.

Modify prompts.

---

# Queue Service

## Purpose

Coordinate asynchronous generation.

The queue should remain database-backed during Generation One.

Future:

Kafka.

RabbitMQ.

SQS.

---

## Queue States

```text
Queued

Claimed

Running

Completed

Failed

Retry

Cancelled
```

Each transition should generate an audit event.

---

# Queue Recovery

If a worker disappears:

Heartbeat expires.

↓

Job unlocked.

↓

Returned to queue.

↓

Another worker claims.

No job should remain permanently orphaned.

---

# Comfy Dispatcher

## Purpose

Translate platform requests into provider requests.

The Dispatcher isolates provider-specific logic.

---

## Responsibilities

Inject parameters.

Submit workflow.

Poll execution.

Download outputs.

Handle failures.

Capture execution metadata.

---

# Provider Independence

Current provider:

ComfyUI.

Future:

Flux API.

↓

OpenAI Images.

↓

Cloud Rendering.

↓

Internal GPU Farm.

Only Dispatcher changes.

Everything else remains identical.

---

# Card Composition Service

## Purpose

Render deterministic card images.

Inputs:

Approved artwork.

↓

Template.

↓

Frame.

↓

Icons.

↓

Statistics.

↓

Typography.

↓

Card Front.

No AI involvement.

---

# Sprite Generation Service

## Purpose

Coordinate tactical sprite production.

Responsibilities:

Sprite base.

Turnaround.

Animation generation.

Atlas construction.

Metadata generation.

Unity validation.

---

# Notification Service (Future)

Future responsibilities:

Generation complete.

↓

Review requested.

↓

Publication complete.

↓

Revision requested.

↓

Workflow failure.

Supports:

Email.

Discord.

Slack.

Internal notifications.

---

# Health Service

Every backend service should expose:

```text
/health

/ready

/live

/version
```

Health endpoints should never require authentication inside trusted infrastructure.

---

# Metrics Service

Expose:

Generation duration.

Queue depth.

Review duration.

Publication duration.

Worker utilization.

GPU utilization.

Storage growth.

Failure rate.

Metrics should be Prometheus-compatible where possible.

---

# Configuration Service

Every service consumes configuration through a shared Configuration Service.

Never:

Read environment variables throughout the codebase.

Configuration should be parsed exactly once.

---

# Shared Contracts

Every service should consume:

Shared DTOs.

Shared Schemas.

Shared Enums.

Shared Constants.

These belong under:

```text
shared/

contracts/

schemas/

types/
```

Avoid duplicated models.

---

# Service Communication

Preferred communication:

```
HTTP

↓

JSON

↓

Authenticated

↓

Versioned
```

Future:

Internal gRPC.

Event-driven messaging.

Service mesh.

---

# Error Handling

Every service returns standardized errors.

Example:

```json
{
  "code": "ASSET_NOT_APPROVED",
  "message": "...",
  "retryable": false
}
```

Error structures should remain consistent across services.

---

# Logging

Every service logs:

Startup.

Shutdown.

Requests.

Errors.

Warnings.

Performance.

Configuration validation.

Logs should include correlation IDs for distributed tracing.

---

# Service Lifecycle

Every service should support:

Startup.

↓

Validation.

↓

Ready.

↓

Running.

↓

Graceful Shutdown.

↓

Stopped.

No service should terminate active work unexpectedly.

---

# Future Services

Expected future additions:

```text
LoRA Training Service

Identity Analysis Service

Video Generation Service

Voice Generation Service

Marketplace Service

Analytics Service

GPU Scheduler

Cloud Storage Service

CDN Service

Review Collaboration Service
```

The architecture should support these without modification to existing services.

---

# Acceptance Criteria

The service architecture is considered complete when:

1. Every service has a single responsibility.
2. Every service exposes explicit APIs.
3. Services communicate through contracts rather than implementation details.
4. Workers remain stateless.
5. Providers remain abstracted behind the Dispatcher.
6. Shared models exist only once.
7. Services can be deployed and scaled independently.
8. Future services can be introduced without restructuring the platform.

---

# Part IX — Technical Reference

## Section 9.1 — API Reference

## Purpose

This section defines the canonical API contract for the Wishes AI Asset Pipeline.

The API exists to expose deterministic asset management capabilities to trusted clients while abstracting the internal implementation of generation, review, publication, and storage.

The API should be resource-oriented, versioned, and backward compatible.

All endpoints described here represent the logical API. Internal implementation details may evolve provided the public contract remains stable.

---

# API Design Principles

The Asset API follows these principles:

- RESTful resource design
- Stateless requests
- Deterministic responses
- Explicit versioning
- Standard HTTP semantics
- Consistent error handling
- Strong validation
- Complete auditability

---

# Base URL

Recommended structure:

```text
/api/v1
```

Future versions:

```text
/api/v2
/api/v3
```

Older versions should remain operational during migration periods.

---

# Authentication

Every authenticated request should provide:

```text
Authorization: Bearer <JWT>
```

Future authentication providers:

- OpenID Connect
- OAuth2
- Enterprise SSO

Public asset endpoints may support anonymous access if explicitly enabled.

---

# Standard Response Format

Successful responses should follow:

```json
{
  "success": true,
  "data": {}
}
```

Errors:

```json
{
  "success": false,
  "error": {
    "code": "ASSET_NOT_FOUND",
    "message": "...",
    "retryable": false
  }
}
```

Avoid inconsistent response structures.

---

# Pagination

Collection endpoints should support:

```text
page

pageSize

sort

order

filter
```

Future:

Cursor pagination.

---

# Filtering

Assets should support filtering by:

```text
Object UUID

Asset Role

Status

Version

Reviewer

Workflow

Created Date

Updated Date
```

Filters should remain composable.

---

# Asset Endpoints

## Create Asset Request

```text
POST /api/v1/assets/request
```

Purpose:

Queue generation.

---

### Request

```json
{
  "object_uuid": "...",
  "asset_role": "portrait",
  "template": "default"
}
```

---

### Response

```json
{
  "job_uuid": "...",
  "status": "queued"
}
```

---

# Get Asset

```text
GET /api/v1/assets/{asset_uuid}
```

Returns:

- Metadata
- Status
- Versions
- Runtime URI (if published)

---

# List Assets

```text
GET /api/v1/assets
```

Supports:

Filtering.

Sorting.

Pagination.

Role filtering.

Status filtering.

---

# Asset Versions

```text
GET /api/v1/assets/{asset_uuid}/versions
```

Returns complete version history.

Historical versions remain immutable.

---

# Asset Lineage

```text
GET /api/v1/assets/{asset_uuid}/lineage
```

Returns:

Parents.

Children.

Ancestors.

Descendants.

---

# Asset Dependencies

```text
GET /api/v1/assets/{asset_uuid}/dependencies
```

Useful for impact analysis.

Example:

Replacing portrait.

↓

List affected assets.

---

# Asset Metadata

```text
GET /api/v1/assets/{asset_uuid}/metadata
```

Returns internal metadata.

Requires elevated permissions.

Unity should never call this endpoint.

---

# Asset Runtime Metadata

```text
GET /api/v1/assets/{asset_uuid}/runtime
```

Returns simplified runtime metadata.

Designed for Unity.

---

# Review Endpoints

## Approve

```text
POST /api/v1/review/{asset_uuid}/approve
```

Request:

```json
{
  "notes":"Identity preserved."
}
```

---

## Reject

```text
POST /api/v1/review/{asset_uuid}/reject
```

Request:

```json
{
  "reason":"Incorrect anatomy."
}
```

---

## Revision

```text
POST /api/v1/review/{asset_uuid}/revision
```

Purpose:

Request regeneration.

---

## Archive

```text
POST /api/v1/review/{asset_uuid}/archive
```

Archives without deleting history.

---

# Publication Endpoints

## Publish

```text
POST /api/v1/publication/{asset_uuid}
```

Requirements:

Approved.

Metadata valid.

Dependencies valid.

---

## Republish

```text
POST /api/v1/publication/{asset_uuid}/republish
```

Creates new runtime package.

Does not regenerate artwork.

---

## Publication History

```text
GET /api/v1/publication/{asset_uuid}
```

Returns every publication event.

---

# Workflow Endpoints

## List Workflows

```text
GET /api/v1/workflows
```

Returns:

Workflow code.

Version.

Provider.

Status.

---

## Workflow Details

```text
GET /api/v1/workflows/{workflow_code}
```

Returns:

Manifest.

Compatibility.

Versions.

Supported roles.

---

## Validate Workflow

```text
POST /api/v1/workflows/{workflow_code}/validate
```

Runs startup validation.

Administrator only.

---

# Job Endpoints

## Queue Status

```text
GET /api/v1/jobs
```

Returns:

Queued.

Running.

Completed.

Failed.

---

## Job Details

```text
GET /api/v1/jobs/{job_uuid}
```

Returns:

Execution metadata.

Worker.

Duration.

Outputs.

---

## Retry Job

```text
POST /api/v1/jobs/{job_uuid}/retry
```

Creates new execution.

Does not modify original history.

---

# Worker Endpoints

```text
GET /api/v1/workers
```

Returns:

Worker UUID.

Heartbeat.

Current Job.

Status.

GPU.

Memory.

---

# Storage Endpoints

Administrator only.

Examples:

```text
GET /api/v1/storage/statistics

POST /api/v1/storage/reindex

POST /api/v1/storage/verify
```

---

# Health Endpoints

Every service should expose:

```text
GET /health

GET /ready

GET /live

GET /version
```

Health endpoints should remain lightweight.

---

# Metrics Endpoints

Future:

```text
GET /metrics
```

Prometheus-compatible.

---

# Administrative Endpoints

Examples:

```text
POST /api/v1/admin/cache/rebuild

POST /api/v1/admin/workflows/reload

POST /api/v1/admin/storage/cleanup

POST /api/v1/admin/config/reload
```

Require Administrator role.

---

# Standard HTTP Status Codes

| Status | Meaning |
|----------|---------|
| 200 | Success |
| 201 | Resource Created |
| 202 | Accepted / Queued |
| 204 | No Content |
| 400 | Validation Failure |
| 401 | Authentication Required |
| 403 | Forbidden |
| 404 | Resource Not Found |
| 409 | State Conflict |
| 422 | Business Rule Failure |
| 429 | Rate Limited |
| 500 | Internal Error |
| 503 | Service Unavailable |

Avoid using 500 for validation failures.

---

# Error Codes

Recommended standard codes:

```text
ASSET_NOT_FOUND

INVALID_WORKFLOW

WORKFLOW_DISABLED

INVALID_SOURCE

REVIEW_REQUIRED

PUBLICATION_FAILED

INVALID_METADATA

DEPENDENCY_FAILURE

QUEUE_FULL

WORKER_OFFLINE

CONFIGURATION_ERROR
```

These should remain stable.

---

# Idempotency

Safe operations:

GET

HEAD

OPTIONS

Idempotent POSTs should support client idempotency keys where duplicate submissions are possible.

---

# Correlation IDs

Every request should generate or propagate:

```text
Correlation-ID
```

Used for:

Logs.

Tracing.

Metrics.

Audit.

---

# API Documentation

Generate OpenAPI documentation from source.

Every endpoint should include:

Description.

Permissions.

Request schema.

Response schema.

Examples.

Error codes.

Version history.

---

# API Deprecation

Deprecation policy:

1. Mark deprecated.
2. Document replacement.
3. Support transition period.
4. Remove only in major version.

Never silently remove endpoints.

---

# Acceptance Criteria

The API is considered complete when:

1. Every asset lifecycle operation is exposed through versioned endpoints.
2. Responses follow a consistent structure.
3. Errors are standardized.
4. Authorization is enforced.
5. APIs remain backward compatible.
6. Runtime consumers receive only runtime metadata.
7. Administrative functionality remains isolated behind elevated permissions.

---

# Part X — Technical Reference

## Section 10.1 — Database & Storage Reference

## Purpose

This section defines the canonical database architecture and storage model used by the Wishes AI Asset Pipeline.

The database is the authoritative source of truth.

The filesystem is a deterministic storage medium.

The database determines what an asset is.

The filesystem stores the files representing that asset.

These two systems should never disagree.

---

# Design Principles

The persistence layer follows five core principles.

1. Database First
2. Metadata Driven
3. Immutable Publication
4. Explicit Relationships
5. Deterministic Storage

Every persistence decision should reinforce these principles.

---

# Persistence Architecture

```
Application

↓

Asset Service

↓

PostgreSQL

↓

Storage Service

↓

Filesystem

↓

Published Runtime
```

No service should bypass PostgreSQL.

---

# Database Responsibilities

The database owns:

- Asset identities
- Metadata
- Relationships
- Review history
- Publication history
- Workflow references
- Prompt references
- Queue state
- Audit history

The database never stores image binaries.

---

# Filesystem Responsibilities

The filesystem stores:

- Images
- Sprite sheets
- Card renders
- Icons
- Thumbnails
- Temporary renders
- Export packages

The filesystem should never determine application state.

---

# Primary Tables

Generation One defines the following primary tables.

```text
asset

asset_type

asset_role

asset_generation_job

asset_dependency

asset_review_event

asset_workflow

asset_template

asset_publication

audit_log
```

Future tables may extend this list without restructuring.

---

# Asset Table

Purpose:

Defines every generated asset.

Responsibilities:

Identity

Version

Status

Role

Ownership

Relationships

Storage reference

Never duplicate metadata stored elsewhere.

---

# Asset Type

Defines broad asset categories.

Examples:

```text
Image

Sprite

Card

Animation

Audio

Video

3D Model
```

Asset types evolve independently of roles.

---

# Asset Role

Roles describe purpose.

Examples:

```text
Portrait

Full Body

Card Front

Thumbnail

Icon

Sprite Base

Sprite Sheet

Marketing Art
```

Roles should originate from data.

Never hardcode role names.

---

# Asset Generation Job

Tracks generation lifecycle.

States:

```text
Queued

Claimed

Running

Completed

Failed

Cancelled
```

Workers update only their assigned jobs.

---

# Asset Dependency

Stores explicit parent-child relationships.

Example:

```text
Portrait

↓

Full Body

↓

Sprite Sheet
```

Every dependency is directional.

Circular references prohibited.

---

# Asset Review Event

Stores review history.

Review events are immutable.

Examples:

Approve

Reject

Revision Requested

Publish

Archive

Historical review records are never edited.

---

# Asset Publication

Tracks publication events.

Stores:

Published version

Runtime package

Publication timestamp

Publisher

Export profile

Publication history remains permanent.

---

# Asset Workflow

Associates assets with workflow versions.

Example:

```text
Asset

↓

Workflow Version

↓

Manifest Version

↓

Prompt Version
```

Enables deterministic reconstruction.

---

# Audit Log

Every significant operation records:

Actor

↓

Action

↓

Object

↓

Timestamp

↓

Result

↓

Correlation ID

Audit history should never be modified.

---

# Queue Tables

Generation One uses PostgreSQL-backed queues.

Future support:

Kafka

RabbitMQ

AWS SQS

The database abstraction should permit future migration.

---

# Database Constraints

Required constraints:

Primary Keys

Foreign Keys

Unique Constraints

Check Constraints

Not Null

Validation Constraints

Database integrity is preferred over application assumptions.

---

# Referential Integrity

Every relationship should validate.

Examples:

Asset references existing object.

Workflow exists.

Review references asset.

Publication references approved asset.

No orphaned records.

---

# Transactions

The following operations should always execute within transactions:

Generation completion.

Review approval.

Publication.

Asset replacement.

Metadata migration.

Rollback.

Never partially complete these operations.

---

# Index Strategy

Recommended indexes:

Asset UUID

Object UUID

Asset Role

Status

Version

Workflow

Created Date

Updated Date

Publication Status

Frequently queried JSONB fields should receive expression indexes where appropriate.

---

# JSONB Usage

Store flexible metadata as JSONB.

Examples:

Generation settings.

Prompt packages.

Provider metadata.

Runtime extensions.

Avoid storing commonly filtered fields exclusively inside JSONB.

---

# Database Functions

Preferred database functions:

Create Asset

Approve Asset

Publish Asset

Replace Portrait

Archive Asset

Validate Dependencies

Rebuild Lineage

Database functions should encapsulate multi-table operations.

---

# Storage Layout

Canonical storage:

```text
generated-assets/

pending/

approved/

published/

rejected/

archive/

temp/
```

Each directory has one responsibility.

---

# Pending Storage

Contains:

Generated assets awaiting review.

Files remain mutable.

Never referenced by Unity.

---

# Approved Storage

Contains:

Canonical review-approved assets.

Serves as source material for derived generation.

Only one approved version should be active for a given lineage.

---

# Published Storage

Contains immutable runtime assets.

Unity consumes only published storage.

Published assets are never overwritten.

---

# Rejected Storage

Contains rejected generations.

Reasons to retain:

Prompt tuning.

Future comparison.

Regression analysis.

Reviewer training.

Rejected assets should not be deleted automatically.

---

# Archive Storage

Historical storage.

Contains:

Superseded versions.

Legacy exports.

Deprecated runtime packages.

Archive remains append-only.

---

# Temporary Storage

Temporary files.

Examples:

Intermediate renders.

Upscaling outputs.

Masks.

Preview images.

Safe to clean automatically.

---

# Object Directory Structure

Recommended layout:

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

Every asset version receives its own directory.

---

# Naming Standards

Avoid human-readable filenames.

Preferred:

```text
asset_uuid.ext
```

Metadata provides friendly names.

---

# File Integrity

Every stored file should record:

SHA-256

File Size

Dimensions

MIME Type

Created Timestamp

Integrity should be verified during publication.

---

# Storage Validation

Validation includes:

Directory exists.

↓

Writable.

↓

Checksum valid.

↓

Metadata present.

↓

Database matches filesystem.

Validation failures prevent publication.

---

# Metadata Synchronization

Database remains authoritative.

Filesystem discrepancies should trigger repair.

Never infer missing database records from existing files.

---

# Backup Strategy

Back up:

Database.

↓

Approved Assets.

↓

Published Assets.

↓

Workflow Registry.

↓

Templates.

↓

Configuration.

Temporary directories excluded.

---

# Restore Strategy

Recovery order:

Database.

↓

Storage.

↓

Metadata Validation.

↓

Dependency Validation.

↓

Publication Validation.

↓

Resume Services.

---

# Database Maintenance

Routine maintenance:

VACUUM

ANALYZE

Index Rebuild

Statistics Update

Constraint Validation

Future partitioning should preserve logical schema.

---

# Growth Strategy

Expected future expansion:

Millions of assets.

Hundreds of millions of metadata records.

Multiple storage volumes.

Cloud object storage.

CDN replication.

Current schema should support future scaling.

---

# Future Storage Providers

Current:

Local filesystem.

Future:

Amazon S3.

Azure Blob.

Google Cloud Storage.

On-premise object storage.

Provider changes should require Storage Service changes only.

---

# Storage Abstraction

All storage operations pass through:

Storage Service.

Never:

Direct filesystem access.

This allows provider replacement.

---

# Data Retention

Recommended policy:

Pending

90 Days

Rejected

Permanent

Approved

Permanent

Published

Permanent

Archive

Permanent

Temporary

Automatic cleanup

Retention policies should remain configurable.

---

# Acceptance Criteria

The persistence layer is considered complete when:

1. PostgreSQL remains the authoritative source of truth.
2. Filesystem state matches database state.
3. Published assets are immutable.
4. Every asset has complete metadata.
5. Referential integrity is enforced.
6. Storage is deterministic.
7. Backups and restores preserve complete history.
8. Future storage providers can be introduced without redesign.

---

# Section 10.2 — Directory Reference

## Canonical Repository Layout

```text
wishes-game/
│
├── client-unity/
├── server/
├── shared/
├── database/
├── infrastructure/
├── generated-assets/
├── docs/
├── prototypes/
├── tools/
└── .github/
```

This directory layout should remain stable across future releases.

Each top-level directory has a single clearly defined responsibility.

Detailed directory ownership, maintenance responsibilities, and lifecycle rules are defined in Document 12 and should be considered canonical.

---

# Part XI — Operations, Configuration & Troubleshooting

## Section 11.1 — Configuration Matrix

## Purpose

Configuration is the mechanism that allows the Wishes Asset Pipeline to adapt to different environments without requiring source code changes.

Every configurable value should originate from one of four sources:

1. Environment Variables
2. Configuration Files
3. Database Configuration
4. Runtime Defaults

Configuration precedence is always:

```text
Environment Variable

↓

Configuration File

↓

Database Configuration

↓

Application Default
```

No application component should directly read environment variables after startup.

All configuration should be exposed through the Configuration Service.

---

# Configuration Categories

The platform configuration is divided into logical domains.

```text
Application

Database

Redis

Storage

Workflow Registry

ComfyUI

Prompt Builder

Workers

Logging

Security

Monitoring

Performance

Unity Export

Feature Flags
```

Each category should be independently documented and validated.

---

# Required Startup Validation

During startup, every service should validate:

✓ Configuration syntax

✓ Required values

✓ Directory existence

✓ Database connectivity

✓ Redis connectivity

✓ Workflow availability

✓ Manifest validity

✓ Storage accessibility

✓ Required templates

✓ Required fonts

✓ Required export profiles

Startup should fail immediately if validation fails.

---

# Runtime Configuration Rules

Configuration should be:

Immutable after startup unless specifically designed to support live reload.

Live reload should be limited to:

- Logging level
- Feature flags
- Queue polling interval
- Monitoring endpoints

Core infrastructure settings should require service restart.

---

# Feature Flags

Every experimental capability should be controlled by a feature flag.

Examples:

```text
ENABLE_BATCH_GENERATION

ENABLE_CHARACTER_LORA

ENABLE_IDENTITY_SCORING

ENABLE_VIDEO_EXPORT

ENABLE_3D_EXPORT

ENABLE_DISTRIBUTED_RENDERING

ENABLE_PLUGIN_SYSTEM
```

Feature flags should never permanently replace versioned functionality.

---

# Environment Profiles

Recommended environments:

```text
Development

Testing

Staging

Production
```

Every environment should maintain its own:

- Configuration
- Secrets
- Storage
- Database
- Monitoring

Cross-environment sharing should be avoided.

---

# Configuration Documentation

Every configuration entry should document:

- Name
- Description
- Data Type
- Default Value
- Required
- Valid Range
- Restart Required
- Security Classification

This documentation should remain synchronized with implementation.

---

# Section 11.2 — Monitoring

## Monitoring Philosophy

Monitoring exists to detect operational issues before users experience failures.

The monitoring platform should answer:

Is the system healthy?

Is the system fast?

Is the system reliable?

Is the system scaling?

---

# Health Indicators

Recommended health indicators:

```text
API Gateway

Asset Service

Workflow Registry

Workers

ComfyUI

PostgreSQL

Redis

Storage

GPU

Queue
```

Each should expose:

- Health
- Readiness
- Version
- Metrics

---

# Key Performance Indicators

Track:

Generation latency

↓

Queue latency

↓

Review duration

↓

Publication duration

↓

Worker utilization

↓

GPU utilization

↓

Storage growth

↓

Failure rate

These KPIs should be visualized through dashboards.

---

# Alert Thresholds

Suggested defaults:

Queue Length

>500

Worker Offline

>60 seconds

Storage Usage

>80%

GPU Memory

>90%

Workflow Validation Failure

Immediate

Publication Failure

Immediate

Repeated Generation Failure

>10%

Thresholds should remain configurable.

---

# Logging Standards

Every service should log:

Startup

Shutdown

Configuration

Errors

Warnings

Performance

Security Events

Audit Events

Structured JSON logging is recommended.

---

# Correlation IDs

Every request should generate or propagate a Correlation ID.

The same identifier should appear in:

API logs

Worker logs

Database audit

Review history

Publication history

Metrics

This enables end-to-end tracing.

---

# Section 11.3 — Troubleshooting Guide

## Philosophy

Troubleshooting should prioritize diagnosis before intervention.

Never modify production data without understanding the root cause.

Every incident should leave the system in a better documented state.

---

# Common Issue — Workflow Validation Failure

Symptoms:

- Startup failure
- Workflow unavailable
- Generation requests rejected

Checklist:

✓ Manifest present

✓ Workflow JSON present

✓ Versions compatible

✓ Injection nodes valid

✓ Output nodes defined

Resolution:

Repair the manifest or workflow.

Restart validation.

---

# Common Issue — Worker Stuck

Symptoms:

Queue grows.

Worker heartbeat missing.

No completed jobs.

Checklist:

✓ Worker process alive

✓ Database reachable

✓ ComfyUI reachable

✓ Storage writable

✓ Heartbeat updating

Resolution:

Terminate worker.

Return claimed jobs to queue.

Launch replacement worker.

---

# Common Issue — Generation Failure

Symptoms:

Job enters Failed state.

No output images.

Checklist:

✓ Workflow valid

✓ Prompt valid

✓ ComfyUI operational

✓ GPU memory available

✓ Output directory writable

Resolution:

Correct root cause.

Retry job.

Do not modify historical metadata.

---

# Common Issue — Publication Failure

Symptoms:

Asset approved.

Not available in runtime.

Checklist:

✓ Runtime export completed

✓ Storage writable

✓ Metadata valid

✓ Dependency graph valid

✓ Export profile available

Resolution:

Retry publication.

Do not regenerate artwork.

---

# Common Issue — Metadata Mismatch

Symptoms:

Image exists.

Database missing.

Checklist:

✓ Audit history

✓ Storage integrity

✓ Backup availability

✓ Asset lineage

Resolution:

Database remains authoritative.

Never recreate metadata by inspecting images.

---

# Common Issue — Identity Drift

Symptoms:

Generated asset resembles wrong character.

Checklist:

✓ Portrait source correct

✓ Identity block correct

✓ Prompt version correct

✓ Workflow version correct

✓ Review history

Resolution:

Request revision.

Do not publish.

---

# Common Issue — Storage Corruption

Symptoms:

Missing files.

Checksum mismatch.

Unreadable assets.

Resolution:

Restore storage.

Validate checksums.

Rebuild runtime exports if necessary.

Published versions should remain unchanged.

---

# Common Issue — Queue Deadlock

Symptoms:

Jobs never execute.

Workers idle.

Resolution:

Validate locks.

Release orphaned claims.

Restart queue processor.

Monitor recovery.

---

# Section 11.4 — Maintenance Procedures

## Daily

Verify:

Queue healthy.

Workers healthy.

Storage healthy.

GPU healthy.

Database healthy.

Review backlog reasonable.

---

## Weekly

Validate:

Workflow registry.

Manifest registry.

Storage integrity.

Rejected asset growth.

Backup completion.

Review metrics.

---

## Monthly

Perform:

Restore test.

Security audit.

Performance review.

Dependency updates.

Capacity review.

Documentation review.

---

## Quarterly

Review:

Architecture.

Prompt library.

Workflow quality.

Storage growth.

Identity consistency.

Future roadmap.

---

# Section 11.5 — Operational Best Practices

✔ Validate before deploying.

✔ Backup before migrating.

✔ Never overwrite published assets.

✔ Preserve audit history.

✔ Preserve review history.

✔ Validate workflows after updates.

✔ Monitor queue continuously.

✔ Treat metadata as authoritative.

✔ Test recovery procedures regularly.

✔ Document every production incident.

---

# Cross References

This operational reference complements:

- Document 09 — Deployment Guide
- Document 13 — Configuration
- Document 17 — Operations Runbook
- Document 18 — Security Architecture

---

# Transition to Final Reference

The next and final section of this manual provides:

- Workflow Catalog
- Repository Cross Reference
- Glossary
- Acronym Index
- Canonical Terminology
- Implementation Matrix
- Document Cross Reference
- Final Acceptance Checklist

These appendices serve as the permanent index for the entire Wishes AI Asset Pipeline documentation set.

---

# Part XII — Appendices & Reference Index

## Appendix A — Workflow Catalog

This appendix serves as the canonical registry of every supported workflow within the Wishes AI Asset Pipeline.

Every production workflow must be:

- Version controlled
- Manifest backed
- Independently testable
- Independently replaceable
- Independently documented

No workflow should exist without a corresponding manifest.

---

# Generation One Workflow Catalog

| Workflow | Purpose | Source Asset |
|-----------|---------|--------------|
| portrait_generate | Initial portrait generation | None |
| portrait_refine | Portrait refinement | Portrait |
| full_body_generate | Full body generation | Portrait |
| icon_generate | Icon generation | Portrait |
| thumbnail_generate | Thumbnail generation | Portrait |
| card_art_generate | Card illustration | Portrait |
| sprite_base_generate | Tactical sprite base | Full Body |
| sprite_sheet_generate | Tactical sprite sheet | Sprite Base |

Future workflows may be added without modifying existing workflow definitions.

---

# Workflow Dependency Tree

```text
Portrait

↓

Full Body

↓

Sprite Base

↓

Sprite Sheet

↓

Published Runtime
```

Alternative branch:

```text
Portrait

↓

Card Illustration

↓

Card Composition

↓

Published Card
```

Each workflow should define its dependencies explicitly through its manifest.

---

# Workflow Validation Checklist

Every workflow should successfully validate:

✓ JSON syntax

✓ Manifest compatibility

✓ Injection points

✓ Output definitions

✓ Version

✓ Required templates

✓ Required models

✓ Required LoRAs

✓ Required ControlNets

✓ Provider compatibility

Validation failures should prevent production startup.

---

# Appendix B — Repository Cross Reference

## Primary Directories

| Directory | Purpose |
|-----------|---------|
| client-unity | Unity client |
| server | Backend services |
| shared | Shared contracts and schemas |
| database | Migrations, functions, seed data |
| generated-assets | Runtime asset storage |
| docs | Documentation |
| prototypes | Prototype applications |
| infrastructure | Deployment and infrastructure |
| tools | Development tooling |
| .github | CI/CD workflows |

---

## Asset Service

```text
server/

asset-service/

src/

controllers/

services/

workers/

storage/

renderers/

prompts/

metadata/

config/

tests/
```

---

## Generated Asset Storage

```text
generated-assets/

pending/

approved/

published/

rejected/

archive/

temp/
```

Every directory has one clearly defined responsibility.

---

# Appendix C — Canonical Asset Roles

Current production asset roles:

```text
Portrait

Full Body

Icon

Thumbnail

Card Front

Sprite Base

Sprite Sheet
```

Planned future roles:

```text
Animation

Voice

Video

3D Mesh

Marketing Banner

Loading Screen

Quest Illustration

Dialogue Portrait

Creature Illustration

Building Illustration

Environment Illustration
```

The Asset Role registry should remain data-driven.

---

# Appendix D — Canonical Status Values

## Asset Status

```text
Pending

Approved

Published

Rejected

Archived
```

---

## Job Status

```text
Queued

Claimed

Running

Completed

Failed

Retry

Cancelled
```

---

## Review Decisions

```text
Approve

Approve With Notes

Revision Requested

Reject

Archive
```

---

# Appendix E — Canonical Service Responsibilities

| Service | Responsibility |
|----------|----------------|
| Asset Service | Orchestration |
| Workflow Registry | Workflow management |
| Prompt Builder | Prompt construction |
| Workers | AI execution |
| Storage Service | File management |
| Metadata Service | Metadata generation |
| Dependency Service | Asset graph |
| Publication Service | Runtime publication |
| Review Service | Human review |
| API Gateway | External access |

No responsibility should exist in multiple services.

---

# Appendix F — Canonical Terminology

## Asset

A generated or deterministic artifact associated with a game object.

---

## Object

A game entity capable of owning assets.

Examples:

Character

NPC

Card

Template

Deck

---

## Asset Role

The purpose of an asset.

Examples:

Portrait

Full Body

Icon

---

## Workflow

The implementation used to generate an asset.

---

## Manifest

The interface definition describing how the Asset Service interacts with a workflow.

---

## Identity

The canonical visual representation of a character established by the approved portrait.

---

## Publication

The process of converting an approved asset into an immutable runtime asset.

---

## Runtime Asset

A published asset consumed by Unity.

---

## Source Asset

An approved asset used to generate another asset.

---

## Derived Asset

Any asset created from another approved asset.

---

## Lineage

The complete ancestry of an asset.

---

## Dependency Graph

The directed graph describing parent-child asset relationships.

---

## Review Event

An immutable record of a review decision.

---

## Prompt Package

The deterministic collection of prompt components used during generation.

---

# Appendix G — Acronyms

| Acronym | Meaning |
|----------|---------|
| API | Application Programming Interface |
| CDN | Content Delivery Network |
| DTO | Data Transfer Object |
| GPU | Graphics Processing Unit |
| JSON | JavaScript Object Notation |
| JSONB | Binary JSON (PostgreSQL) |
| JWT | JSON Web Token |
| LoRA | Low-Rank Adaptation |
| NPC | Non-Player Character |
| REST | Representational State Transfer |
| SSO | Single Sign-On |
| UUID | Universally Unique Identifier |

---

# Appendix H — Implementation Matrix

| System | Implemented In |
|----------|----------------|
| Architecture | Document 01 |
| Database | Document 02 |
| Workflow Registry | Document 03 |
| Backend APIs | Document 04 |
| Review Portal | Document 05 |
| Character Consistency | Document 06 |
| Tactical Sprites | Document 07 |
| Card Composition | Document 08 |
| Deployment | Document 09 |
| Validation | Document 10 |
| Claude Execution Guide | Document 11 |
| Repository Layout | Document 12 |
| Configuration | Document 13 |
| Future Roadmap | Document 14 |
| Development Standards | Document 15 |
| Evolution Roadmap | Document 16 |
| Operations | Document 17 |
| Security | Document 18 |
| Master Reference | Document 19 |

---

# Appendix I — Master Acceptance Checklist

The Wishes AI Asset Pipeline should satisfy all of the following before production release.

## Architecture

✓ Modular

✓ Deterministic

✓ Versioned

✓ Extensible

---

## Database

✓ Normalized

✓ Indexed

✓ Versioned

✓ Audited

---

## Storage

✓ Deterministic

✓ Immutable publication

✓ Metadata synchronized

---

## Workflow Registry

✓ Manifest validated

✓ Version controlled

✓ Provider independent

---

## Prompt Builder

✓ Deterministic

✓ Identity aware

✓ Version controlled

---

## Workers

✓ Stateless

✓ Recoverable

✓ Scalable

---

## Review

✓ Human approval

✓ Complete audit history

✓ Immutable review events

---

## Publication

✓ Immutable runtime assets

✓ Runtime metadata

✓ Unity compatible

---

## Unity

✓ Metadata driven

✓ Published assets only

✓ Addressable ready

---

## Operations

✓ Monitoring

✓ Alerting

✓ Recovery

✓ Backup

---

## Security

✓ Least privilege

✓ Secrets protected

✓ Immutable publication

✓ Audit logging

---

## Documentation

✓ Architecture

✓ Implementation

✓ Operations

✓ Security

✓ Reference

---

# Final Architectural Principles

Every future feature added to the Wishes AI Asset Pipeline should preserve the following principles.

1. The database is authoritative.

2. Metadata is more important than generated images.

3. Every generated asset has a deterministic lineage.

4. Human review is required before publication.

5. Published assets are immutable.

6. Unity consumes only published assets.

7. AI generates creative content only.

8. Deterministic systems own gameplay, metadata, storage, and publication.

9. Every workflow is versioned and manifest-backed.

10. Every decision is auditable.

---

# Closing Statement

The Wishes AI Asset Pipeline has been designed as a long-lived platform rather than a single feature.

Its architecture separates creativity from determinism, allowing artificial intelligence to generate visual content while ensuring that every asset remains traceable, reproducible, reviewable, and compatible with the evolving Wishes universe.

The system is intentionally modular so that future AI providers, rendering technologies, media formats, and deployment environments can be integrated without redesigning the underlying architecture.

By adhering to the principles established throughout Documents **01–19**, the Wishes project gains a scalable, maintainable, and production-ready asset pipeline capable of supporting the game's growth for many years while preserving the artistic integrity and canonical consistency of its world.

---

# End of Document

**Document:** 19 — Wishes AI Asset Pipeline Master Reference

**Status:** Complete

**Supersedes:** None

**References:** Documents 01–18

**Next Recommended Documentation:** Wishes AI Gameplay & World Simulation Master Reference (Series 20–39)

---