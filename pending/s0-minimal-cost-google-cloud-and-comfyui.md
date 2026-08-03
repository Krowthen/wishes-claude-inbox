# Task: Build the Expanded Wishes S0 Google Cloud Environment and Complete Asset Suite

Created: 2026-08-03
Priority: High
Mode: implementation-with-approval-gates

Allow Edit: true
Allow Commit: true
Allow Push: true
Allow Delete: false
Allow Asset Import: false
Allow File Copy: true

Allow Cloud Read: true
Allow Cloud Plan: true
Allow Cloud Apply: approval-required
Allow Cloud Destroy: approval-required
Allow Billing Account Change: approval-required
Allow Secret Value Read: false
Allow Service Account Key Creation: false

## Objective

Build the first cloud-hosted Wishes S0 development environment using the lowest reasonable cost configuration that includes every required platform component:

- four separate application-data Cloud Storage buckets;
- one separate Terraform-state bucket;
- a dedicated PostgreSQL `asset` schema;
- Memorystore for Redis;
- Pub/Sub request, result, and dead-letter transport;
- Cloud Run CPU services that scale to zero;
- a one-shot Cloud Run NVIDIA L4 GPU Job for ComfyUI;
- the complete Wishes asset suite;
- human review, approval, publication, lineage, and audit;
- reproducible Terraform, teardown, and rebuild procedures.

PostgreSQL remains authoritative. Redis is transient coordination only. Pub/Sub is transport only. ComfyUI creates candidates only and may never approve, publish, attach, or mutate authoritative Wishes game state.

This task supersedes conflicting instructions in the earlier S0 and Cloud Run GPU tasks. Preserve their valid portrait-first, lineage, model, workflow, storage, and security requirements while implementing the expanded S0 baseline in the current deployment appendices.

Do not run `terraform apply`, change billing linkage, create secret values, destroy resources, or perform destructive database migration until the user reviews the plan, cost estimate, IAM/database grant matrix, migration plan, and resource inventory and explicitly approves that stage.

## Canonical sources

Read from `Krowthen/wishes-canon`, branch `ryancox-chatgpt` unless already merged:

```text
drafts/deployment/README.md
drafts/deployment/00-architecture-overview.md
drafts/deployment/02-terraform-bootstrap.md
drafts/deployment/03-networking-and-dns.md
drafts/deployment/05-cloud-sql-postgresql.md
drafts/deployment/06-memorystore-redis.md
drafts/deployment/07-storage-and-artifacts.md
drafts/deployment/08-wishes-service-deployment.md
drafts/deployment/09-comfyui-gpu-platform.md
drafts/deployment/09a-comfyui-management-layer.md
drafts/deployment/10-claude-code-operations.md
drafts/deployment/12-ci-cd-and-release.md
drafts/deployment/13-observability.md
drafts/deployment/14-security.md
drafts/deployment/15-cost-and-capacity.md
drafts/deployment/16-backup-and-disaster-recovery.md
drafts/deployment/17-operational-runbooks.md
drafts/deployment/18-validation-and-production-readiness.md
drafts/deployment/19a-scale-tiers-and-multi-region-evolution.md
drafts/deployment/appendices/README.md
drafts/deployment/appendices/a-s0-minimal-cost-environment-profile.md
drafts/deployment/appendices/b-resource-variable-and-cost-catalog.md
drafts/deployment/appendices/c-iam-secrets-and-data-boundaries.md
drafts/deployment/appendices/d-release-evidence-and-handoff-schemas.md
drafts/deployment/appendices/e-comfyui-cloud-reconciliation.md
drafts/deployment/appendices/f-documentation-completion-and-implementation-readiness.md
canon/glossary/Wishes_Version_Numbering_Standard.md
```

Review and reconcile these prior inbox inputs:

```text
pending/comfyui-asset-pipeline/01_system_architecture.md
pending/comfyui-asset-pipeline/02_database_and_storage.md
pending/comfyui-asset-pipeline/03_comfyui_workflows.md
pending/comfyui-asset-pipeline/04_backend_and_api.md
pending/deploy-comfyui-cloud-run-gpu.md
pending/s0-minimal-cost-google-cloud-and-comfyui.md
```

The current file replaces the previous version of `pending/s0-minimal-cost-google-cloud-and-comfyui.md`.

## Repository workflow

1. Read the root `CLAUDE.md` in `wishes-game` before editing.
2. Inspect the current branch, repository status, submodules, CI configuration, and branch protections.
3. Create or reuse a branch named `claude/s0-minimal-cloud`.
4. Do not work directly on `main`.
5. Commit intentionally by coherent unit.
6. Push the Claude branch.
7. Open a draft PR only after the plan and implementation changes are reviewable.
8. Never commit secrets, model binaries, LoRA binaries, Terraform state, generated private assets, or local credentials.

## 1. Repository and implementation inventory

Before modifying code, inspect and report:

- application languages, frameworks, and container entrypoints;
- server and service boundaries;
- database migration convention and current migration head;
- current PostgreSQL schemas, tables, functions, views, and roles;
- current asset tables, asset roles, boundaries, versions, attachments, requests/jobs, workflow registry, lineage, review, and publication logic;
- current Redis, queue, Pub/Sub, or outbox implementations;
- existing Dockerfiles and build scripts;
- existing Terraform or Google Cloud files;
- existing ComfyUI integration, workflow JSON, manifests, models, LoRAs, and asset generator;
- current authentication, authorization, and asset-review UI;
- current test, lint, migration, and development commands;
- existing cloud resources if credentials permit read-only inspection.

Produce a reconciliation table with requirement, current implementation, gap, required change, risk, and evidence.

Do not create parallel systems merely because names differ. Where the architecture requires a dedicated `asset` schema, plan a controlled migration from the current implementation.

## 2. S0 architecture and cost plan

Prepare the exact resource inventory before cloud apply.

Required topology:

```text
one Google Cloud project
one region, initially us-central1 after availability validation
one custom VPC
one regional subnet, /26 or larger
one Terraform-state bucket
one Artifact Registry Docker repository
one Cloud Run application service
one Cloud Run asset service
one Cloud Run ComfyUI L4 GPU Job
one shared-core single-zone Cloud SQL PostgreSQL instance
one PostgreSQL asset schema
one Memorystore for Redis Basic Tier 1 GiB instance
three Pub/Sub topics
three Pub/Sub subscriptions
four application-data buckets
Secret Manager containers required by code
Cloud Logging and Monitoring defaults
one Cloud Billing budget
```

Explicitly excluded unless separately approved:

```text
GKE
external HTTP(S) load balancer
Cloud NAT
Serverless VPC Access connector when Direct VPC egress works
Cloud SQL HA or replicas
Memorystore Standard Tier or Redis Cluster
additional Redis instances
additional GPUs
GPU concurrency above one
public raw ComfyUI
service-account keys
committed-use discounts
cross-region replication
production data
```

### Four mandatory application buckets

Create exactly these logical data boundaries:

```text
<project-id>-wishes-s0-models
<project-id>-wishes-s0-workflows
<project-id>-wishes-s0-inputs
<project-id>-wishes-s0-outputs
```

The Terraform-state bucket is separate and is not one of the four. Do not merge the four application buckets or replace them with prefixes in one bucket.

### Cost report

Report:

- standing monthly Cloud SQL estimate;
- standing monthly Redis estimate;
- expected Pub/Sub cost at S0 volume;
- storage and operation assumptions for each bucket;
- Artifact Registry storage/build assumptions;
- Cloud Run CPU assumptions;
- active GPU cost per minute and per hour;
- estimated cost for one complete asset suite;
- total expected monthly cost for idle, light-use, and development-test scenarios;
- quotas and availability risks;
- budget thresholds.

Default budget:

```hcl
monthly_budget_usd = 100
```

Alerts:

```text
25% actual
50% actual
80% actual
100% actual
100% forecast
```

A budget is not a hard spending cap. Do not implement automatic billing shutdown or destructive cost actions.

## 3. Terraform structure

Use current conventions. If none exist, use a structure equivalent to:

```text
infra/terraform/
  bootstrap/
  environments/s0/
  modules/
    project_services/
    budget/
    network/
    artifact_registry/
    storage_bucket/
    cloud_sql/
    redis/
    pubsub/
    service_accounts/
    cloud_run_service/
    cloud_run_job/
    secret_container/
```

Required behavior:

- remote GCS state with locking and version protection;
- provider lock file committed;
- labels where supported;
- explicit project and region variables;
- no secret values in state or variable files;
- Cloud SQL deletion protection during normal development;
- lifecycle protections for stateful resources;
- stable names and outputs;
- no manual resources outside documented bootstrap exceptions;
- `terraform fmt`, `validate`, and plan checks;
- plan artifact and add/change/destroy summary;
- explicit stateful replacement detection.

## 4. Network and Direct VPC egress

Create `wishes-s0-vpc` and a regional S0 subnet.

Use Direct VPC egress for Cloud Run services and jobs that require Redis/private access. Default to `private-ranges-only`.

Requirements:

- validate subnet size;
- attach only required services/jobs;
- configure narrow firewall rules;
- avoid Cloud NAT unless a validated dependency cannot function without it;
- document startup retries for VPC connectivity;
- validate Redis connectivity from the asset service;
- keep raw ComfyUI without public ingress.

## 5. Four storage buckets

Create all four buckets as regional Standard Storage.

### Models bucket

```text
checkpoints/<sha256>/<filename>
diffusion-models/<sha256>/<filename>
text-encoders/<sha256>/<filename>
clip/<sha256>/<filename>
vae/<sha256>/<filename>
controlnet/<sha256>/<filename>
upscale-models/<sha256>/<filename>
loras/<sha256>/<filename>
manifests/models/<version-uuid>.json
manifests/loras/<version-uuid>.json
```

### Workflows bucket

```text
api/<workflow-version-uuid>.json
manifests/<workflow-version-uuid>.json
schemas/<schema-version>.json
packages/<package-version-uuid>/
```

### Inputs bucket

```text
objects/<object-type>/<object-uuid>/<job-uuid>/
manifests/executions/<job-uuid>.json
references/<job-uuid>/
masks/<job-uuid>/
poses/<job-uuid>/
```

### Outputs bucket

```text
pending/<object-type>/<object-uuid>/<role>/<job-uuid>/
rejected/<object-type>/<object-uuid>/<role>/<asset-version-uuid>/
approved/<object-type>/<object-uuid>/<role>/<asset-version-uuid>/
published/<object-type>/<object-uuid>/<role>/<asset-version-uuid>/
evidence/executions/<job-uuid>/
evidence/suites/<object-uuid>/
database-exports/<timestamp>/
```

Apply uniform bucket-level access, public-access prevention, immutable version/checksum references, lifecycle rules, no public listing, least-privilege permissions, and no runtime `roles/storage.admin`.

Published assets remain in the outputs bucket and are served through approved application or bounded signed-delivery controls.

## 6. Cloud SQL and dedicated asset schema

Use one Cloud SQL PostgreSQL database named `wishes` with the smallest supported shared-core development machine type.

S0 posture:

```text
single zone
no HA
no replicas
smallest practical storage
synthetic/sanitized data only
deletions protected during normal operation
Cloud SQL connector from Cloud Run
```

Create roles:

```text
wishes_asset_owner
wishes_asset_runtime
wishes_app_runtime
wishes_auditor
```

Create:

```sql
CREATE SCHEMA IF NOT EXISTS asset AUTHORIZATION wishes_asset_owner;
```

The final asset domain must contain schema-qualified compatible equivalents of:

```text
asset.asset_type
asset.asset_role
asset.asset_workflow
asset.asset_model
asset.asset_model_version
asset.asset_lora
asset.asset_lora_version
asset.asset_job
asset.asset
asset.asset_version
asset.asset_attachment
asset.asset_dependency
asset.asset_review_event
asset.asset_publication
asset.asset_outbox
asset.asset_inbox
```

Preserve `asset_role.boundaries JSONB`, UUIDs, UTC timestamps, immutable versions, source/root lineage, status guards, actor/audit history, outbox/inbox idempotency, and the `wishes_` function prefix.

Required functions or compatible application transactions:

```text
asset.wishes_asset_request_create
asset.wishes_asset_job_claim
asset.wishes_asset_result_reconcile
asset.wishes_asset_approve
asset.wishes_asset_reject
asset.wishes_asset_publish
asset.wishes_asset_mark_descendants_stale
asset.wishes_asset_suite_status
```

Inspect the current schema, map current objects, produce migration and rollback plans, migrate data safely, use temporary compatibility views only when necessary, remove or de-authorize duplicate public-schema authorities, and validate counts, checksums, references, versions, and audit history.

Do not create a second independent asset system while leaving the current one authoritative.

## 7. Memorystore Redis

Deploy:

```text
Tier: Basic
Capacity: 1 GiB
Region: same as Cloud Run
Redis version: current approved 7.x
HA: no
```

Enable Redis AUTH and in-transit encryption when supported by the client and provider. Document any compatibility exception before apply.

Approved uses:

- worker and dispatcher leases;
- transient progress;
- rate limiting;
- idempotency acceleration;
- short-lived caches;
- optional transient Redis Streams for operational visibility.

Redis is not authoritative.

Test connection through Direct VPC egress, lease expiry, duplicate-worker exclusion, rate limits, resets, flush/recreation, and recovery from PostgreSQL/outbox state without asset-state loss.

## 8. Pub/Sub transport

Create topics:

```text
wishes-s0-asset-requests
wishes-s0-asset-results
wishes-s0-asset-dead-letter
```

Create subscriptions:

```text
wishes-s0-asset-dispatch
wishes-s0-asset-result-reconciler
wishes-s0-asset-dead-letter-monitor
```

Use standard Pub/Sub, not Pub/Sub Lite.

Rules:

- identifiers, versions, attempts, timestamps, and manifest URIs only;
- no secrets, large prompts, or arbitrary workflow JSON;
- at-least-once delivery;
- idempotency through `asset.asset_inbox` and job fences;
- acknowledgement only after durable database commit;
- dead-letter routing after five attempts by default;
- bounded retention;
- replay from transactional outbox history;
- monitor backlog age, publish errors, delivery attempts, and DLQ count.

Request event:

```json
{
  "schemaVersion": 1,
  "eventId": "uuid",
  "assetJobUuid": "uuid",
  "workflowVersionUuid": "uuid",
  "executionManifestUri": "gs://inputs-bucket/manifests/executions/job.json",
  "attempt": 1,
  "createdAt": "timestamp"
}
```

Result event:

```json
{
  "schemaVersion": 1,
  "eventId": "uuid",
  "assetJobUuid": "uuid",
  "cloudRunExecution": "name",
  "resultManifestUri": "gs://outputs-bucket/evidence/executions/job/result.json",
  "status": "generated",
  "createdAt": "timestamp"
}
```

Validate outbox-before-publish, duplicate request/result delivery, retry, DLQ, ack-after-commit, replay, and poison-message handling.

## 9. Cloud Run services

Deploy at least two CPU services:

```text
Wishes application
Wishes asset service
```

Initial settings:

```text
min instances: 0
max instances: 1
cpu: 1
memory: 512 MiB unless evidence requires more
request-based billing
private-ranges-only Direct VPC egress where required
```

The application handles player/operator authentication, game APIs, review UI integration, approved/published asset reads, and calls the asset service rather than directly mutating asset tables.

The asset service handles requests, prompts, workflow/model/LoRA selection, asset-schema transactions, outbox publication, result reconciliation, Redis leases/progress, GPU Job execution, candidate lifecycle, lineage, staleness, and suite status.

Expose `GET /health` and `GET /ready`. Asset readiness checks PostgreSQL, Redis, Pub/Sub configuration, and all four buckets.

## 10. ComfyUI GPU Job

Implement one bounded Cloud Run Job:

```text
GPU: NVIDIA L4 x1
CPU: 4
Memory: 16 GiB
Tasks: 1
Parallelism: 1
Retries: 0
Timeout: 3600 seconds
Zonal redundancy: disabled
```

The image must pin ComfyUI, custom nodes, and packages; contain no credentials; bind raw ComfyUI to loopback; read immutable manifests; validate checksums; download only required artifacts; submit API workflows; upload outputs and results; publish the result event; and exit nonzero on invalid input or failed durable publication.

Per-execution identifiers:

```text
ASSET_JOB_UUID
WORKFLOW_VERSION_UUID
INPUT_MANIFEST_URI
OUTPUT_PREFIX_URI
EXECUTION_SNAPSHOT_URI
PUBSUB_RESULT_TOPIC
```

ComfyUI never connects to PostgreSQL and never approves or publishes assets.

## 11. Complete asset suite

S0 is not complete after portrait generation.

Required primary roles:

```text
portrait
full_body
icon
thumbnail
card_front
tactical_sprite_sheet
```

Required tactical animations:

```text
idle
walk
run
attack
cast
hit
down
guard
interact
```

Required directions:

```text
front
back
left
right
```

Required common emoji roles:

```text
emoji_happy
emoji_angry
emoji_sad
emoji_surprised
emoji_confused
emoji_determined
emoji_injured
emoji_laughing
```

Required manifests/processors:

```text
portrait_generate
portrait_refine
full_body_from_portrait
icon_from_portrait_or_crop
thumbnail_from_portrait
card_front_compose
sprite_base_from_full_body
sprite_sheet_from_sprite_base
emoji_from_portrait
```

Rules:

- portrait is generated from structured character/card data;
- identity-bearing descendants use an approved portrait or full body;
- full body normally precedes sprites;
- icon and thumbnail prefer deterministic crop/resize;
- card text, stats, symbols, frames, and labels are deterministic rendering;
- every role is versioned, reviewable, rejectable, regenerable, and publishable;
- every derived asset records source/root lineage and source version;
- portrait replacement marks dependent suite assets stale;
- stale assets cannot remain current published authority;
- suite status is queryable by object.

Minimum API operations:

```text
POST /api/assets/requests
GET  /api/assets/jobs/:jobUuid
GET  /api/assets/objects/:objectType/:objectUuid/suite
GET  /api/assets/:assetUuid
GET  /api/assets/:assetUuid/lineage
POST /api/assets/:assetUuid/approve
POST /api/assets/:assetUuid/reject
POST /api/assets/:assetUuid/regenerate
POST /api/assets/:assetUuid/publish
POST /api/assets/objects/:objectType/:objectUuid/regenerate-stale
```

Review evidence must display source, candidate, role/version, workflow/runtime, model/LoRA stack, prompts, seed/settings, dimensions/checksum, lineage, actor/comments, and tactical frame metadata.

## 12. Tests

Add tests for:

### Database

- clean and upgrade migrations;
- schema grants and denied actions;
- outbox atomicity and inbox idempotency;
- status guards and current-version uniqueness;
- source/root lineage and staleness;
- suite-completeness query.

### Redis

- lease expiry;
- concurrent exclusion;
- flush recovery;
- rate limit;
- connection reset.

### Pub/Sub

- publish/consume;
- duplicate request/result;
- retry and DLQ;
- replay;
- ack-after-commit.

### Storage

- all four buckets exist;
- public-access prevention;
- unauthorized cross-bucket access denied;
- ComfyUI reads only required buckets and creates only assigned outputs;
- approval/publication is controlled by asset service;
- checksum mismatch rejected.

### ComfyUI

- missing artifacts fail safely;
- invalid checksum fails before generation;
- timeout/cancellation classified;
- result manifest validated;
- no public raw endpoint;
- no database access.

### Full suite

- generate and approve portrait and full body;
- derive and approve icon and thumbnail;
- compose and approve card front;
- generate tactical animations/directions and validate metadata;
- generate and approve all emojis;
- publish the suite;
- replace portrait and verify staleness;
- regenerate a stale descendant;
- verify audit and lineage.

## 13. Plan and approval checkpoint

Before cloud apply, provide:

1. repository inspection summary;
2. files changed;
3. schema migration and rollback design;
4. Terraform plan file/hash and summary;
5. complete resource inventory;
6. IAM matrix;
7. database role/grant matrix;
8. bucket access matrix;
9. Pub/Sub resource/message table;
10. Redis purpose/recovery statement;
11. quota and regional availability report;
12. standing and usage cost estimate;
13. per-GPU-hour and complete-suite cost estimate;
14. risks and unresolved prerequisites;
15. apply and validation plan;
16. destroy and recovery procedures.

Stop and wait for explicit approval before apply.

## 14. Approved apply and validation

After explicit approval only:

1. apply Terraform;
2. query every resource;
3. run migrations;
4. deploy immutable image digests;
5. validate identities and denied permissions;
6. validate Redis and Pub/Sub;
7. execute the complete asset suite;
8. review and publish the suite;
9. test portrait replacement and staleness;
10. record costs, logs, IDs, checksums, and evidence;
11. run a stable post-apply plan;
12. prove destroy/rebuild behavior in an approved window.

## Stop conditions

Stop and report when:

- project or billing target is ambiguous;
- the region lacks L4, Cloud SQL, Redis, or quota;
- estimated cost exceeds the approved budget;
- Terraform proposes unexpected destruction or replacement;
- any application bucket would be omitted or combined;
- the `asset` schema cannot be migrated safely;
- duplicate authoritative asset tables would remain;
- Redis or Pub/Sub would be omitted;
- a static service-account key appears necessary;
- raw ComfyUI would become public;
- model or LoRA licensing is unclear;
- the complete asset suite cannot meet the bounded Job contract;
- production data would enter S0;
- secret values would enter Git, logs, Terraform state, or task output.

## Required final report

Return one consolidated report containing:

```text
Summary
Repository and branch
Commits and draft PR
Files changed
Database schema and migration results
Terraform resources planned/applied
Four-bucket inventory and IAM
Redis configuration and validation
Pub/Sub configuration and validation
Cloud Run services and GPU Job
Full asset-suite status
Tests and evidence
Costs and quotas
Security exceptions
Known gaps
Rollback/destroy/rebuild procedure
Approvals still required
```

Do not claim S0 completion unless all four buckets, the `asset` schema, Redis, Pub/Sub, and the complete reviewed asset suite exist and have been validated.
