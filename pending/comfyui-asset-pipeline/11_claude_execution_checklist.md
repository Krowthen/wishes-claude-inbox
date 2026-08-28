# 11 — Claude Code Implementation Execution Guide

## Purpose

This document is the **master implementation guide** for Claude Code and serves as the authoritative execution order for implementing the Wishes AI Asset Pipeline.

Unlike the previous architecture documents, this document defines **what Claude should build, in what order, how each phase should be validated, and what conditions must be met before continuing**.

---

# Global Implementation Rules

These rules apply throughout the entire implementation.

## Rule 1 — Sequential Execution

Never skip phases.

Each phase depends on successful completion of the previous phase.

Do not begin a new phase until:

- Code compiles.
- Tests pass.
- Database validates.
- Manual smoke tests pass.
- Documentation is updated.

---

## Rule 2 — Preserve Existing Architecture

Do not replace existing Wishes architecture unless explicitly instructed.

Integrate with:

- Existing PostgreSQL schema
- Existing API Gateway
- Existing Fastify services
- Existing Character Creator prototype
- Existing Unity client
- Existing Wishes card system
- Existing asset pipeline

---

## Rule 3 — Database First

The database schema is the source of truth.

Never create runtime assumptions that contradict the database.

---

## Rule 4 — Deterministic Systems

Everything outside AI image generation must be deterministic.

This includes:

- Card rendering
- Asset placement
- Metadata
- Text rendering
- Statistics
- File locations
- Versioning
- Dependencies

---

## Rule 5 — Portrait Authority

The approved portrait is always the visual authority.

Every downstream asset derives from the approved portrait either directly or indirectly.

Replacing a portrait invalidates downstream assets.

---

# Overall Execution Roadmap

```text
Repository Preparation
        ↓
Environment Validation
        ↓
Database
        ↓
Seed Data
        ↓
Storage Layer
        ↓
Workflow Registry
        ↓
Prompt Builder
        ↓
Asset Request API
        ↓
Queue Worker
        ↓
ComfyUI Dispatcher
        ↓
Asset Normalization
        ↓
Review Pipeline
        ↓
Dependency Graph
        ↓
Portrait Generation
        ↓
Full Body Generation
        ↓
Icons & Thumbnails
        ↓
Card Composition
        ↓
Sprite Generation
        ↓
Review Portal
        ↓
Validation
        ↓
Performance
        ↓
Security
        ↓
Documentation
        ↓
Production Readiness
```

---

# Phase 0 — Repository Preparation

Verify repository structure.

Expected folders:

```text
client-unity/
database/
docs/
generated-assets/
prototypes/
server/
shared/
```

Required additions:

```text
server/
    asset-service/
        src/
        workflows/
        workflow-manifests/
        templates/
        models/
        tests/

generated-assets/
    pending/
    approved/
    published/
    rejected/
    archive/
```

Verify:

- Folder permissions
- Git ignore rules
- Existing documentation

Deliverables:

- Directory structure complete
- Repository builds successfully

Validation:

- Repository loads
- No broken imports
- Existing applications still compile

---

# Phase 1 — Environment Validation

Verify local environment.

Required software:

- PostgreSQL
- Redis
- Node.js
- npm
- TypeScript
- Fastify
- Docker
- ComfyUI
- GPU drivers

Verify:

- PostgreSQL connection
- Redis connection
- ComfyUI availability
- Asset storage writable

Smoke Test:

```text
Database ✓
Redis ✓
ComfyUI ✓
Storage ✓
API ✓
```

Do not continue if any dependency fails.

---

# Phase 2 — Database

Implement all schema from:

- 02_database_and_storage.md

Required tables:

- asset_type
- asset_role
- asset
- asset_generation_job
- asset_dependency
- asset_review_event
- asset_workflow

Implement:

- Constraints
- Foreign keys
- Indexes
- Triggers
- Audit logging

Validation:

- Fresh migration succeeds
- Database rebuild succeeds
- Rollback succeeds
- Schema matches specification

---

# Phase 3 — Seed Data

Insert:

- Asset Types
- Asset Roles
- Workflow Registry
- Quality references
- Default templates

Validation:

```sql
SELECT COUNT(*) ...
```

matches expected totals.

---

# Phase 4 — Storage Layer

Implement StorageService.

Responsibilities:

- Directory creation
- Path normalization
- File movement
- Publication
- Archive
- Cleanup

Never concatenate paths manually.

Always use path utilities.

Validation:

Create test asset.

Move through:

Pending

↓

Approved

↓

Published

↓

Archive

Verify file hashes unchanged.

---

# Phase 5 — Workflow Registry

Implement WorkflowRegistryService.

Responsibilities:

- Load workflow JSON
- Load manifest
- Validate injection schema
- Detect missing nodes
- Version workflows

Validation:

Every workflow:

- Loads
- Validates
- Can be injected

---

# Phase 6 — Prompt Builder

Implement PromptBuilderService.

Prompt composition:

Global Style

↓

Role Prompt

↓

Identity Block

↓

User Prompt

↓

Negative Prompt

↓

Revision Notes

↓

Runtime Settings

Validation:

Same inputs produce identical prompt package.

---

# Phase 7 — Asset Request API

Implement:

POST /requests

Responsibilities:

- Validate object
- Validate role
- Resolve source asset
- Select workflow
- Queue generation job

Validation:

Job successfully created.

---

# Phase 8 — Queue Worker

Implement:

- Queue polling
- Job claiming
- Retry logic
- Timeout handling
- Worker heartbeat

Use:

```sql
FOR UPDATE SKIP LOCKED
```

Validation:

Multiple workers never process same job.

---

# Phase 9 — ComfyUI Dispatcher

Implement:

- Workflow injection
- Prompt submission
- History polling
- Output discovery
- Error handling

Validation:

Portrait generation succeeds.

---

# Phase 10 — Asset Normalization

Normalize outputs.

Store:

- Asset record
- Metadata
- Prompt package
- Workflow version
- Source references

Copy generated files into:

```text
generated-assets/pending/
```

Validation:

Database and filesystem agree.

---

# Phase 11 — Review Pipeline

Implement:

Approve

Reject

Publish

Archive

Revise

Replace Portrait

Validation:

Portrait approval updates current asset.

---

# Phase 12 — Dependency Graph

Implement:

- Parent references
- Child references
- Ancestor lookup
- Descendant lookup
- Stale propagation

Validation:

Replacing portrait marks descendants stale.

---

# Phase 13 — Portrait Pipeline

Generate:

Portrait Candidate

↓

Review

↓

Approval

↓

Current Portrait

Validation:

Portrait becomes canonical source.

---

# Phase 14 — Full Body Pipeline

Source:

Approved portrait.

Generate:

Full body candidate.

Validation:

Identity preserved.

---

# Phase 15 — Icons & Thumbnails

Generate:

- Icon
- Thumbnail

Validation:

Correct dimensions.

Readable.

Consistent identity.

---

# Phase 16 — Card Composition

Implement deterministic renderer.

AI generates:

Artwork only.

Application renders:

- Names
- Stats
- Icons
- Layout
- Text
- Frames

Validation:

Same inputs generate identical card.

---

# Phase 17 — Tactical Sprite Pipeline

Generate:

- Sprite Base
- Turnaround
- Animation Frames
- Sprite Sheet
- Metadata

Validation:

Unity imports without modification.

---

# Phase 18 — Review Portal

Implement:

Dashboard

Browser

Review

Queue

Character Assets

Published

Settings

Validation:

Complete review workflow operational.

---

# Phase 19 — API Validation

Validate every endpoint.

Required scenarios:

- Success
- Invalid payload
- Invalid UUID
- Missing asset
- Unauthorized
- Conflict
- Internal error

No unhandled exceptions.

---

# Phase 20 — Performance Validation

Measure:

Queue latency

Portrait generation

Card rendering

Sprite generation

API response

Portal loading

Targets defined in:

10_validation_testing.md

---

# Phase 21 — Security

Verify:

- ComfyUI private
- Workflow directory protected
- Model directory protected
- Secrets in environment variables
- Published assets immutable

---

# Phase 22 — Documentation

Update:

README

CLAUDE.md

API docs

Deployment docs

Workflow registry

Template documentation

Environment variables

---

# Git Strategy

Branch:

```text
feature/comfyui-asset-pipeline
```

Recommended commits:

1. Database
2. Storage
3. Workflow Registry
4. Prompt Builder
5. Queue
6. Dispatcher
7. Review
8. Dependency Graph
9. Card Renderer
10. Sprite Pipeline
11. Portal
12. Validation
13. Documentation

Each commit should:

- Compile
- Pass tests
- Contain one logical feature

---

# Rollback Strategy

If a phase fails:

Stop.

Return to previous successful commit.

Do not partially revert.

If migration fails:

Restore backup.

Repair migration.

Retry.

---

# Smoke Test

Minimum end-to-end validation:

```text
Generate Portrait
Approve Portrait
Generate Full Body
Approve Full Body
Generate Icon
Generate Thumbnail
Render Card
Generate Sprite Sheet
Approve Assets
Publish Assets
Open Unity
Verify Assets Load
```

---

# Integration Test Matrix

Validate:

- Database
- API
- Queue
- ComfyUI
- Storage
- Review
- Publication
- Dependency Graph
- Card Renderer
- Sprite Pipeline
- Unity Integration

---

# Final Acceptance Audit

Before merge:

```text
Database                  ✓
Storage                   ✓
Workflow Registry         ✓
Prompt Builder            ✓
Queue                     ✓
Comfy Dispatcher          ✓
Review Pipeline           ✓
Dependency Graph          ✓
Portrait Pipeline         ✓
Full Body Pipeline        ✓
Icons                     ✓
Card Composition          ✓
Sprite Generation         ✓
Review Portal             ✓
Validation                ✓
Performance               ✓
Security                  ✓
Documentation             ✓
Unity Integration         ✓
```

---

# Production Readiness Checklist

The implementation is production-ready when:

- Every migration succeeds from an empty database.
- Every workflow validates successfully.
- Portrait generation is deterministic.
- Full body generation preserves identity.
- Card composition is deterministic.
- Sprite sheets load directly into Unity.
- Review portal supports the complete approval workflow.
- Asset lineage is preserved.
- Replacing portraits invalidates descendants.
- Published assets remain immutable.
- Logging and monitoring are operational.
- Backup and recovery procedures have been verified.
- Documentation matches the implementation.

---

# Future Roadmap

Deferred until after the first production milestone:

- Character-specific LoRA lifecycle
- Automated identity scoring
- Multi-GPU scheduling
- Distributed asset workers
- Cloud object storage
- CDN distribution
- Animated cards
- Eight-direction sprites
- Equipment layer compositing
- Procedural animation
- Collaborative review
- AI-assisted prompt optimization
- Batch localization rendering

---

# Completion Criteria

Claude Code should consider the ComfyUI Asset Pipeline complete only when:

1. All previous specification documents have been fully implemented.
2. Every acceptance criterion from Documents 01–10 passes.
3. End-to-end generation, review, publication, and Unity integration function without manual intervention.
4. The implementation is reproducible from an empty repository using only this specification set.