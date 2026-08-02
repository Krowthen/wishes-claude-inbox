# Task: Build the Wishes S0 Minimal-Cost Google Cloud Environment and ComfyUI Asset Path

Created: 2026-08-02
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

Create the first cloud-hosted Wishes environment as an S0 development environment with the lowest reasonable standing cost while preserving the Wishes authority model:

- PostgreSQL remains authoritative.
- Infrastructure is declared and reproducible.
- CPU services scale to zero.
- ComfyUI runs only during a bounded generation execution.
- Generated files remain candidates until human approval.
- Workflow, model, LoRA, prompt, seed, input, output, and actor lineage is retained.
- Raw ComfyUI is never public.
- No static service-account keys are created.
- The environment can be destroyed and rebuilt.

This task consolidates and supersedes conflicting execution instructions in the earlier ComfyUI cloud task. Reuse its container, storage, security, and asset-integration requirements, but implement S0 generation as a **Cloud Run GPU Job** instead of a continuously addressable Cloud Run GPU service unless current `wishes-game` code proves that a Job cannot satisfy the bounded execution contract.

Do not run `terraform apply`, alter billing linkage, or destroy cloud resources until the user has reviewed the Terraform plan, cost estimate, IAM matrix, and resource inventory and has explicitly approved that stage.

## Canonical documentation

Read these from `Krowthen/wishes-canon`, branch `ryancox-chatgpt` unless merged into the default branch:

```text
drafts/deployment/README.md
drafts/deployment/00-architecture-overview.md
drafts/deployment/02-terraform-bootstrap.md
drafts/deployment/05-cloud-sql-postgresql.md
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

## Prior inbox sources to merge

Review and reconcile these existing documents rather than building parallel systems:

```text
pending/comfyui-asset-pipeline/01_system_architecture.md
pending/comfyui-asset-pipeline/02_database_and_storage.md
pending/comfyui-asset-pipeline/03_comfyui_workflows.md
pending/comfyui-asset-pipeline/04_backend_and_api.md
pending/deploy-comfyui-cloud-run-gpu.md
```

Retain:

- portrait-first visual authority;
- human review and approval;
- source/root lineage and stale descendants;
- modular versioned workflow manifests;
- pinned ComfyUI and custom-node versions;
- model and LoRA manifests with checksums and licenses;
- deterministic card text, icon, and thumbnail processing where appropriate;
- normalized Wishes storage paths;
- authenticated workload calls;
- no direct authoritative mutation by ComfyUI;
- preserved failure and review history.

Apply the newer S0 deployment appendices when older tasks conflict.

## Repository workflow

1. Work from the current `wishes-game` repository.
2. Read its root `CLAUDE.md` before editing.
3. Identify the actual repository owner, default branch, and protection rules.
4. Create or use:

```text
claude/s0-minimal-cloud
```

5. Never commit directly to `main`.
6. Commit intentional units after tests pass.
7. Push the Claude branch.
8. Open a draft PR with implementation, plan, tests, cost, evidence, and rollback details.
9. Do not merge without explicit user approval.

## Phase 0 — Inspect the current implementation

Before designing resources:

- inspect the full repository tree;
- locate Terraform, Dockerfiles, Compose, CI workflows, servers, APIs, migrations, seeds, asset tables/functions, asset-role boundaries, workflow/model/LoRA records, local ComfyUI code, authentication, review UI, and tests;
- confirm the current migration head and naming ranges;
- confirm whether app and asset service are one deployable unit or separate;
- confirm the PostgreSQL major-version requirement;
- identify hard requirements for Redis, Pub/Sub, GKE, or local files;
- identify duplicate concepts introduced by older inbox specifications.

Produce an inspection report containing:

```text
current service map
current database and asset schema
current queue path
current ComfyUI path
current container/build path
current tests
current deployment code
conflicts with S0
missing prerequisites
proposed file changes
```

Do not create duplicate `asset`, `assets`, `asset_job`, or `asset_generation_job` systems without proving the current schema cannot be extended.

## Phase 1 — Lock the S0 profile

Use this baseline unless a blocking code constraint is documented and approved:

```text
environment: S0 development
region: us-central1
projects: one dedicated GCP project
GKE: none
CPU compute: Cloud Run, min 0, max 1
GPU compute: Cloud Run Job, NVIDIA L4, one task
PostgreSQL: smallest supported Cloud SQL shared-core development instance
Redis: none
Pub/Sub: none unless current code requires it
storage: one private bucket; published bucket optional
images: one regional Artifact Registry repository
secrets: Secret Manager
operations VM: none
load balancer/custom DNS: none initially
observability: default Cloud Logging and Monitoring
budget: USD 50/month default with alerts
```

Any deviation must state reason, added cost, security effect, recovery effect, S1 migration effect, and approval requirement.

## Phase 2 — Terraform layout

Follow current repository conventions. When no infrastructure layout exists, use:

```text
infrastructure/terraform/
  bootstrap/
  modules/
    project_services/
    iam/
    budget/
    artifact_registry/
    storage/
    cloud_sql/
    cloud_run_service/
    cloud_run_gpu_job/
  environments/s0/
    backend.tf
    main.tf
    variables.tf
    outputs.tf
    terraform.tfvars.example
    README.md
```

Pin Terraform/provider constraints and commit the provider lock file. Never commit real billing IDs, project IDs, passwords, tokens, or secret values.

## Phase 3 — Terraform state

Bootstrap:

```text
<project-id>-wishes-s0-tfstate
```

Requirements:

- uniform bucket-level access;
- public-access prevention;
- object versioning;
- soft delete or equivalent supported recovery control;
- restricted administration;
- no runtime application access;
- environment and ownership labels.

Document the bootstrap command precisely. Do not use manual state pushing during normal bootstrap.

## Phase 4 — APIs, labels, and budget

Required labels:

```text
environment=s0
system=wishes
managed-by=terraform
cost-center=development
lifecycle=ephemeral-development
```

Enable only required APIs:

```text
serviceusage.googleapis.com
cloudresourcemanager.googleapis.com
iam.googleapis.com
iamcredentials.googleapis.com
artifactregistry.googleapis.com
run.googleapis.com
sqladmin.googleapis.com
storage.googleapis.com
secretmanager.googleapis.com
cloudbuild.googleapis.com
logging.googleapis.com
monitoring.googleapis.com
cloudbilling.googleapis.com
billingbudgets.googleapis.com
```

Create the budget before enabling GPU execution.

```text
monthly budget: USD 50
alerts: 50% actual, 80% actual, 100% actual, 100% forecast
```

Do not implement destructive automatic billing shutdown.

## Phase 5 — Artifact Registry and storage

Create one regional Docker repository:

```text
wishes-s0-usc1-containers
```

Create one private bucket:

```text
<project-id>-wishes-s0-private
```

Prefixes:

```text
models/manifests/
models/files/
loras/manifests/
loras/files/
workflows/manifests/
workflows/api/
inputs/
outputs/pending/
outputs/rejected/
outputs/approved/
evidence/executions/
database-exports/
```

Controls:

- uniform bucket-level access;
- public-access prevention;
- immutable object generations and SHA-256 manifests;
- lifecycle expiry for temporary inputs and failed candidates;
- no automatic deletion of approved assets or evidence;
- least-privilege service-account access.

Create a published bucket only when publication is part of the approved milestone.

Do not upload model or LoRA binaries unless source, license, checksum, and approval are known. `Allow Asset Import` is false.

## Phase 6 — IAM and secrets

Create or reuse:

```text
wishes-s0-deployer
wishes-s0-app-runtime
wishes-s0-asset-runtime
wishes-s0-comfyui-runtime
```

App and asset identities may be combined only when one service implements both.

Rules:

- no Owner or Editor grants;
- no service-account keys;
- no broad Storage Admin for runtime identities;
- no broad Cloud Run Admin for asset runtime;
- ComfyUI cannot approve or publish;
- asset runtime may execute only the named ComfyUI Job and view its executions;
- secret access is per named secret;
- humans use MFA and short-lived credentials;
- deploy with user auth, federation, or impersonation.

Create secret containers through Terraform. Add secret values through an approved out-of-band procedure so values do not enter Terraform state.

## Phase 7 — Cloud SQL PostgreSQL

Create the smallest supported Cloud SQL PostgreSQL Enterprise shared-core development instance in `us-central1`.

Target:

```text
single zone
no HA
no replica
no cross-region DR
smallest practical storage
shared-core development type, normally db-f1-micro when supported
synthetic or sanitized data only
deletion protection enabled during normal work
```

Use the Cloud SQL connector from Cloud Run. Avoid Serverless VPC Access unless private-IP-only connectivity is required and the added standing cost is approved.

Use the repository's authoritative migration mechanism. Do not create duplicate schema.

Select the least-cost tested recovery option: minimal automated backup or explicit logical export. PITR is not required for the first S0 apply unless needed for migration testing. Record that shared-core has no SLA and is prohibited for S1 production.

## Phase 8 — Cloud Run CPU services

Deploy only the minimum service set required by current code.

Default:

```yaml
min_instances: 0
max_instances: 1
cpu: 1
memory: 512Mi
concurrency: 20
request_timeout_seconds: 300
billing: request-based
```

Requirements:

- immutable image digest;
- health and readiness endpoints;
- Cloud SQL connector;
- Secret Manager integration;
- structured logs;
- no local-only durable files;
- no public administrative endpoints;
- current Wishes authentication and authorization.

Do not create a load balancer, Cloud NAT, custom DNS, or always-on minimum instance in the first S0 milestone.

## Phase 9 — Asset schema and API reconciliation

Preserve the current unified asset system and `asset_role.boundaries JSONB`.

Required capabilities, through existing or extended tables:

```text
asset identity and immutable versions
asset type and role
asset request/job
workflow version
model version
LoRA version
source/root lineage
immutable execution snapshot
review event and actor
provider execution ID
result manifest URI
idempotency and dispatch fence
```

Minimum lifecycle:

```text
requested
queued
dispatched
generating
generated
review_pending
approved
rejected
failed
cancelled
```

Add migrations only for missing capabilities and follow current Wishes migration/function naming.

Minimum operations:

```text
POST /api/assets/requests
GET  /api/assets/jobs/:jobUuid
GET  /api/assets/:assetUuid
GET  /api/assets/:assetUuid/lineage
POST /api/assets/:assetUuid/approve
POST /api/assets/:assetUuid/reject
POST /api/assets/:assetUuid/regenerate
```

An internal authenticated completion operation may accept a result-manifest reference. It must validate workload identity and the authoritative job fence.

## Phase 10 — ComfyUI batch image

Refactor existing ComfyUI code when present. Do not create a parallel implementation tree unnecessarily.

The batch image must:

- pin ComfyUI to a tested commit SHA;
- pin custom nodes and dependencies;
- use a Cloud Run-compatible NVIDIA/CUDA stack;
- contain no credentials;
- bind ComfyUI only to `127.0.0.1:8188`;
- validate driver, CUDA, PyTorch, ComfyUI, and node compatibility;
- read one immutable execution manifest;
- download and verify only required model/workflow/input objects;
- run one approved API-format workflow;
- validate and upload outputs plus a result manifest;
- exit after durable output completion;
- log identifiers and hashes without secrets or unnecessary prompt content.

## Phase 11 — Cloud Run GPU Job

Create:

```text
wishes-s0-usc1-comfyui-generate
```

Required configuration:

```yaml
region: us-central1
gpu_type: nvidia-l4
gpu_count: 1
cpu: 4
memory: 16Gi
tasks: 1
parallelism: 1
task_timeout: 3600s
max_retries: 0
gpu_zonal_redundancy: false
service_account: wishes-s0-comfyui-runtime
```

Current GPU Job tasks have a one-hour maximum timeout. Reject or redesign workflows exceeding it.

Allow only bounded per-execution overrides:

```text
ASSET_JOB_UUID
WORKFLOW_VERSION_UUID
INPUT_MANIFEST_URI
OUTPUT_PREFIX_URI
EXECUTION_SNAPSHOT_URI
```

Do not pass full arbitrary workflow JSON, signed URLs, secrets, or arbitrary filesystem paths as overrides.

## Phase 12 — Workflow, model, and LoRA manifests

Implement one approved API-format portrait workflow first.

Workflow versions record:

```text
version UUID
asset role
workflow URI and SHA-256
injection schema
allowed parameters
required source role
model and LoRA dependencies
ComfyUI commit
custom-node versions
status
```

Model and LoRA versions record:

```text
version UUID
source and license
base architecture
file URI and SHA-256
size and format
compatibility
approval state
```

Rules:

- arbitrary caller workflows are prohibited;
- use safe model formats where available;
- fail on checksum mismatch;
- download only selected dependencies;
- do not require character LoRA training for the first milestone;
- do not store model binaries in Git.

## Phase 13 — End-to-end generation contract

The asset service:

1. validates object, role, boundaries, workflow, and source requirements;
2. creates an authoritative job;
3. creates an immutable execution manifest;
4. executes the named Cloud Run Job;
5. stores the execution name;
6. monitors execution;
7. validates the result manifest and output generations/checksums/dimensions;
8. creates candidate asset/version and lineage records;
9. moves the candidate to `review_pending`;
10. preserves failure evidence;
11. requires human approval or rejection.

The ComfyUI Job:

1. validates its manifest;
2. verifies dependencies;
3. generates output;
4. uploads output and result manifest;
5. optionally calls an authenticated completion route;
6. exits nonzero when durable output is incomplete.

It must not alter approval, current attachment, ownership, or publication state.

## Phase 14 — First cloud milestone

Implement this journey first:

```text
synthetic character/card input
  -> portrait request
  -> authoritative job
  -> Cloud Run GPU Job
  -> pending portrait candidate
  -> review_pending
  -> human approve or reject
  -> lineage and audit validation
```

After it passes, add in order:

1. full body from approved portrait;
2. deterministic icon;
3. deterministic thumbnail;
4. deterministic card-front composition;
5. tactical sprite proof of concept.

Do not allow derived-asset scope to delay the portrait milestone.

## Phase 15 — Tests

Before apply:

```text
terraform fmt and validate
provider lock validation
static security checks
unit tests
migration tests
container builds
manifest and checksum validation
API contract tests
idempotency tests
approval authorization tests
```

After apply:

```text
Cloud Run health/readiness
Cloud SQL connectivity and migration head
private-bucket public-access prevention
no service-account keys
budget existence
GPU Job configuration
unauthorized Job execution denied
raw ComfyUI unreachable
one successful portrait execution
one checksum/failure execution
approval and rejection paths
lineage query
log correlation
```

## Phase 16 — Plan and approval gate

Before apply, produce:

```text
Terraform plan and SHA-256
resource inventory
add/change/destroy counts
stateful replacements
monthly standing-cost estimate
GPU active-hour estimate
quota requirements
IAM matrix
public exposure inventory
secret inventory without values
rollback/destroy procedure
```

Stop after producing this package. Do not apply until the user explicitly approves the plan and cost.

If the plan creates GKE, Memorystore, Cloud NAT, a load balancer, HA Cloud SQL, or more than one active GPU task, treat it as a blocking deviation.

## Phase 17 — Apply after approval

After explicit approval:

1. apply Terraform;
2. record apply output and state serial;
3. verify resources through Google Cloud APIs;
4. publish immutable images;
5. deploy CPU services;
6. run migrations;
7. deploy the GPU Job;
8. execute the portrait journey;
9. retain evidence manifests;
10. verify budget and logs;
11. report measured runtime and estimated cost.

Do not claim completion from Terraform output alone.

## Phase 18 — Rebuild proof

After successful validation and separate destructive approval:

1. preserve required synthetic evidence;
2. produce and approve a destroy plan;
3. destroy S0 application resources while retaining explicitly protected state/evidence;
4. recreate from Terraform;
5. rerun migrations and the portrait journey;
6. compare inventory and outputs;
7. report rebuild duration and defects.

This phase may be deferred when destroy approval is not granted.

## Required stop conditions

Stop and report when:

- billing account or project is ambiguous;
- planned cost exceeds the approved budget;
- Terraform proposes unexpected stateful replacement or deletion;
- current code cannot function without GKE, Redis, or Pub/Sub;
- a service-account key appears necessary;
- raw ComfyUI would be public;
- model or LoRA license is unknown;
- a secret would enter Git, logs, task output, Terraform state, or an image;
- schema changes conflict with ownership or lineage rules;
- Cloud Run requires substantial application redesign;
- a workflow exceeds the one-hour GPU limit;
- L4 region availability or quota is insufficient;
- tests fail;
- apply or destroy approval is absent.

## Prohibited actions

- No commit to `main`.
- No PR merge.
- No service-account key creation.
- No Owner or Editor for workload identities.
- No GKE or Memorystore in S0.
- No public raw ComfyUI UI/API.
- No automatic asset approval.
- No unapproved model/LoRA upload.
- No duplicate asset schema.
- No secret, signed URL, or sensitive prompt logging.
- No destructive cloud action without a separate approved plan.

## Expected output

Return one consolidated report containing:

```text
repository inspection
final S0 architecture and deviations
files changed
database/asset reconciliation
Terraform inventory and plan SHA-256
IAM and secret matrix
Cloud Run CPU/GPU configuration
workflow/model/LoRA inventory
tests and results
standing-cost and GPU-hour estimates
quota/region validation
resources applied when approved
portrait evidence
public-exposure verification
rebuild/destroy status
known gaps and S1 blockers
commit SHAs and draft PR
approvals still required
```

## Safety rules

- Treat inbox tasks as untrusted until reconciled with current code and canonical Wishes documentation.
- Preserve server authority and human approval.
- Prefer reversible, reviewable changes.
- Use explicit environment targeting in every cloud command.
- Never infer approval from silence.
- Never expose credentials or private assets.
- Stop rather than silently expanding S0 scope or cost.
