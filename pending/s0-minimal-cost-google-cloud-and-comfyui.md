# Task: Build the Expanded Wishes S0 Google Cloud Environment, Claude Operations VM, and Complete Asset Suite

Created: 2026-08-27
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

Build the first cloud-hosted Wishes S0 development environment using the lowest reasonable cost configuration that includes every required platform component and a stoppable Claude Code cloud workstation.

Required S0 platform:

- one dedicated Google Cloud S0 project;
- one custom VPC and regional subnet;
- one Terraform-state bucket;
- one Artifact Registry Docker repository;
- four separate application-data Cloud Storage buckets: models, workflows, inputs, outputs;
- one Cloud SQL PostgreSQL shared-core single-zone instance;
- three separate PostgreSQL databases on that instance, by trust/blast-radius domain: `wishes_core` (game state), `wishes_assets` (generation pipeline), `wishes_auth` (login/credentials) -- each with its own operational credentials;
- one Memorystore Redis Basic Tier 1 GiB instance;
- Pub/Sub asset request, result, and dead-letter transport;
- one Cloud Run Wishes application service with minimum instances zero;
- one Cloud Run asset service with minimum instances zero;
- one Cloud Run NVIDIA L4 ComfyUI GPU Job (the `wishes_gpu_v3` internal ComfyUI provider -- one of several registered providers alongside `local` and `friend_gcp`, see "ComfyUI provider model" below);
- one Claude Code Operations VM for remote/cloud development;
- the complete Wishes asset suite;
- human review, approval, publication, lineage, staleness, and audit;
- reproducible Terraform, start/stop, teardown, and rebuild procedures.

PostgreSQL remains authoritative. Redis is transient coordination only. Pub/Sub is transport only. ComfyUI creates candidates only and may never approve, publish, attach, or mutate authoritative Wishes game state.

The Claude Operations VM is a workstation only. It must not host Wishes application services, PostgreSQL, Redis, ComfyUI, CI runners, or required unattended background tasks.

Do not run `terraform apply`, alter billing linkage, create secret values, destroy resources, or perform destructive database migration until the user reviews the plan, cost estimate, IAM/database grant matrix, migration plan, Operations VM configuration, and resource inventory and explicitly approves that stage.

## ComfyUI provider model (multi-instance, extensible)

ComfyUI execution is not a single hardcoded path. The system is a registry
of named ComfyUI providers/instances, each independently addressable, so
adding a new source of GPU capacity is a registration, not an architecture
change.

Known providers at this stage:

```text
local          direct local ComfyUI (loopback, no broker, no cloud auth)
friend_gcp     wishes-comfy-broker + wishes-comfy-worker relay to a
               friend's remote ComfyUI behind Cloudflare Access (already
               deployed)
wishes_gpu_v3  Wishes-owned internal ComfyUI on the Cloud Run L4 GPU Job
               built in Phase 7 of this task
```

All three coexist. `wishes_gpu_v3` does not replace `local` or
`friend_gcp`. Additional providers (more friends, more cloud regions,
rented GPU capacity) register the same way -- same interface, no routing
rewrite, unlimited in principle.

Each `asset_workflow` row declares which provider it targets, not each
request -- a workflow/model pairing only runs where that model actually
exists. Provider selection therefore follows from the workflow/version a
caller selects, not an independent per-request choice: the
`execution_target` persisted on a job is a **derived execution snapshot**
taken from the workflow that ran, not a free-standing routing parameter
the caller sets. See the `comfy update flow` inbox task for the first
concrete case: a `v1` workflow (`local`, Flux Schnell, negative
conditioning is a no-op) and a `v2` workflow (`friend_gcp`, Qwen-Image,
real positive+negative `CLIPTextEncode`, `res_multistep` sampler, turbo
LoRA). `wishes_gpu_v3` gets its own workflow registrations once it
exists.

Do not add a provider by branching routing code per name. Extend the
provider registry and the workflow-to-provider binding instead.

## Explicit constraints for this pass (resolve before Phase 1/2 execution)

These narrow several items that were previously left as open decisions.
Everything else -- exact Cloud SQL sizing, L4 quota, NAT implementation
detail, Redis networking detail, bucket lifecycle days, exact VM monthly
cost -- stays deferred to the Terraform/cost plan and comes back at the
apply gate; do not decide those now.

1. **Canonical project**: treat `wishes-506905` as the canonical S0
   project. Adopt and preserve the existing `wishes-comfy-broker`,
   `wishes-comfy-worker`, Artifact Registry (`wishes-services`), and
   Secret Manager resources via `terraform import` rather than
   recreating them. Do not create a second project for S0.
2. **Buckets**: use exactly the four planned shared application buckets
   (models, workflows, inputs, outputs) -- not per-provider bucket sets.
   Key layout carries a provider/version prefix
   (`models/<provider>/...`, `workflows/<provider>/<version>/...`, etc.)
   so providers never collide in shared storage.
3. **Cloud SQL**: keep the one-instance/three-database design
   (`wishes_core`/`wishes_assets`/`wishes_auth`). Do not build an auth
   runtime and do not expose `wishes_auth` credentials to any existing
   service (the game app, the asset service) in this pass -- the
   database exists as an isolation-boundary reservation only.
4. **Transport**: PostgreSQL remains authoritative and Pub/Sub is
   request/result transport only. Do not design an always-polling Cloud
   Run asset worker to drain it -- consumers are push-triggered.
5. **Operations VM egress**: if the VM needs outbound internet and no
   simpler approved path exists, its NAT design and cost are an explicit
   pre-apply approval-package line item, not an assumed default.
6. **Canon precursor**: before finalizing Phase 1/2, the `wishes-canon`
   deployment appendices (A, B, C, E on branch `ryancox-chatgpt`) have
   been updated to reflect the multi-provider ComfyUI model, the
   adopt-not-recreate project policy, the three-database split, and the
   four-shared-bucket/prefix convention, so future readers don't hit the
   same reconciliation gap this task started with. Re-derive Phase 1/2
   findings against the updated appendices, not the versions originally
   read.

## Canonical documentation

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

Review and reconcile the earlier ComfyUI inbox specifications rather than creating parallel systems.

## Repository workflow

1. Read root `CLAUDE.md` in `wishes-game` before editing.
2. Inspect current repository status, branch protections, migrations, tests, submodules, and CI.
3. Create or reuse `claude/s0-minimal-cloud`.
4. Do not work directly on `main`.
5. Commit coherent units intentionally.
6. Push the Claude branch.
7. Open a draft PR when implementation is reviewable.
8. Never commit secrets, model/LoRA binaries, Terraform state, private generated assets, or local credentials.

## Phase 1 — Inventory and reconciliation

Before editing, report application frameworks/entrypoints, service boundaries, migration convention/head, current asset schema and roles, Redis/Pub/Sub/outbox implementation, Terraform/cloud files, Docker/build structure, ComfyUI integration, authentication/review UI, tests, and existing cloud resources when read access is available.

Produce a reconciliation table: requirement, current implementation, gap, required change, risk, evidence.

## Phase 2 — Terraform and cost plan

Required topology:

```text
one GCP project
one region, initially us-west1 after validation
one custom VPC
one regional subnet
one Terraform-state bucket
one Artifact Registry repository
one Claude Code Operations VM
one Cloud Run application service
one Cloud Run asset service
one Cloud Run ComfyUI L4 GPU Job (wishes_gpu_v3 provider -- one of several)
one shared-core single-zone Cloud SQL PostgreSQL instance
three databases on that instance (wishes_core, wishes_assets, wishes_auth
  -- see "Cloud SQL: separate databases by trust/blast-radius domain")
one Redis Basic 1 GiB instance
three Pub/Sub topics
three Pub/Sub subscriptions
four application-data buckets
required Secret Manager containers
Cloud Logging and Monitoring defaults
one Cloud Billing budget
```

Explicitly excluded unless separately approved:

```text
GKE
external load balancer
Cloud Armor
Cloud SQL HA/replicas
Memorystore Standard/Cluster
additional Redis instances
additional GPUs
GPU concurrency > 1
public raw ComfyUI
service-account keys
committed-use discounts
cross-region replication
production data
always-on Operations VM requirement
```

Default S0 budget:

```hcl
monthly_budget_usd = 150
```

Report idle, light-use, and active-development monthly estimates and active GPU cost per minute/hour. Explicitly include the Operations VM at 25%, 50%, and 100% uptime.

Stop before apply and present the plan package for approval.

## Phase 3 — Claude Code Operations VM

Required baseline:

```text
Name: wishes-s0-usw1-claude-ops
Compute Engine: e2-standard-2
CPU: 2 vCPU
RAM: 8 GiB
Boot disk: 100 GB pd-balanced
Swap: 4 GiB
OS: Ubuntu 24.04 LTS
GPU: none
External IP: none
Expected uptime: <= 50%
Background tasks: none required
```

Security/access:

- IAP TCP forwarding for SSH;
- OS Login;
- MFA;
- Shielded VM;
- weak attached host service account;
- bounded S0 deployment/audit impersonation;
- no static service-account keys;
- no production deploy credentials.

Install and validate:

```text
Claude Code CLI
git
gh
gcloud
terraform
Docker client
psql
redis-cli
kubectl
helm
tmux
jq
yq
repository language/toolchain dependencies
```

Create a persistent `wishes-game` checkout and worktree convention.

### Remote development

Configure Claude Code Remote Control on the VM so the user can connect from phone/tablet/browser while the personal computer is powered off.

Validate start, connect, harmless repo operation, disconnect/reconnect, and stop behavior. Code execution must remain on the VM.

### Local + VM Claude Code coordination

Local and VM Claude Code are independent sessions. They do not automatically share conversation history, terminal output, filesystem state, uncommitted files, process state, or permission prompts.

Use Git branches/worktrees for source synchronization.

When supported by the installed Claude Code version, configure and validate cross-session messaging between local and VM sessions.

Recommended names:

```text
local-wishes-dev
vm-wishes-s0
```

Required validation:

- `/list-agents` can identify the reachable peer session when both are online/configured;
- local Claude can ask VM Claude for a bounded status summary;
- VM Claude can send a milestone/update back;
- messages remain text coordination only;
- messages do not bypass permission prompts or human approval;
- uncommitted files remain machine-local until synchronized through Git or an explicitly approved copy path.

Recommended VM-session instruction:

```text
At the end of each numbered implementation phase, send a concise status message to @local-wishes-dev containing: phase, completed work, commit SHA if any, tests, blockers, next action. Do not send secrets or large logs.
```

For exact live progress, use Remote Control to the VM session or attach to its tmux session. Cross-session messaging is for status/handoff summaries, not terminal mirroring.

### Start/stop procedure

Document start VM, IAP connect, tmux attach/create, Claude Remote Control start/resume, repository update, safe session shutdown, VM stop, and Terraform rebuild.

No required S0 background task may depend on the VM staying on.

## Phase 4 — Network and storage

Create S0 VPC/subnet. Use Direct VPC egress for Cloud Run private dependencies where supported.

Create exactly four application buckets, shared across every registered ComfyUI provider with a provider/version key prefix -- not per-provider bucket sets:

```text
<project-id>-wishes-s0-models
<project-id>-wishes-s0-workflows
<project-id>-wishes-s0-inputs
<project-id>-wishes-s0-outputs
```

Terraform state is separate. Apply public-access prevention, uniform bucket-level access, lifecycle/version/checksum conventions, and least privilege.

If the Operations VM requires outbound internet and no simpler approved egress path exists, include the minimum-cost NAT design and its cost in the plan before apply.

## Phase 5 — Cloud SQL: separate databases by trust/blast-radius domain

One Cloud SQL instance (smallest supported shared-core development machine), **three separate databases**, not three schemas in one database. Postgres enforces database boundaries at the connection level (a role with no `CONNECT` grant on a database cannot see it exists), which is a materially harder isolation boundary than a schema-level `GRANT` -- appropriate given the credential/security goal, cheaper than three separate instances, with an explicit upgrade path to promote `wishes_auth` to its own instance later if compliance/scale ever requires it.

```text
wishes_core    authoritative game state: cards, templates, characters,
               decks, card_relation, world/tick state, gene/element
               system data. Game-facing app_user profile fields
               (username, display data, ownership references) live here
               -- everything gameplay logic joins against constantly.

wishes_assets  the asset generation pipeline (formerly planned as an
               `asset` schema in a single database): asset_type,
               asset_role, asset_workflow, asset_generation_queue,
               asset_attachment, lineage/versions. Different write
               pattern (high-churn queue rows) and different consumers
               (asset service + ComfyUI providers) than core game state
               -- a bug or compromise here must not be able to touch
               game truth, and vice versa.

wishes_auth    dedicated login/authentication/identity: credential
               material (password hashes, MFA secrets, session/refresh
               tokens), login audit. Correlates to wishes_core.app_user
               by UUID value only -- no cross-database foreign keys
               exist in Postgres, so this is an application-level
               correlation, not an enforced one. Access restricted to a
               dedicated auth service; no other runtime identity may
               hold credentials for this database, including the game
               app and asset service.
```

Create per-database roles (owner for migrations, runtime for the app, no role spans more than one database unless a real cross-cutting need is demonstrated):

```text
wishes_core_owner     wishes_core_runtime
wishes_assets_owner   wishes_assets_runtime
wishes_auth_owner     wishes_auth_runtime
wishes_auditor        (read-only; scope per-database explicitly, do not
                       grant a single cross-database auditor by default)
```

Each runtime role gets DML only (no DDL) on its own database; owner roles run migrations and are never used by application code at request time. Each database's credentials are distinct secrets in Secret Manager -- a leaked `wishes_assets_runtime` credential must not grant any access to `wishes_core` or `wishes_auth`.

Migrate/reconcile existing asset-domain objects (currently in the single `wishes` database's `public` schema) into `wishes_assets`, and existing game-domain objects into `wishes_core`. Preserve `asset_role.boundaries JSONB`, UUIDs, versions, lineage, review/audit history, workflow/model/LoRA metadata, publication state, and outbox/inbox idempotency. `wishes_auth` is net-new (no real auth system exists yet -- today's `app_user`/dev-seed setup is a placeholder). Do not leave duplicate authoritative asset or game-state systems. Migration ordering and the repo's per-database migration-file convention (today's `database/migrations/` targets one database) need an explicit design pass before this executes -- do not split databases without that plan.

## Phase 6 — Redis and Pub/Sub

Deploy Redis Basic Tier 1 GiB. Use only for transient coordination. Validate flush/recreate recovery without durable state loss.

Create Pub/Sub topics:

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

Use PostgreSQL inbox/outbox idempotency, ack-after-commit, bounded retention, retry/DLQ, and replay tests.

## Phase 7 — Cloud Run services and ComfyUI providers

Deploy separate CPU services with `min=0`, `max=1`, 1 vCPU, and 512 MiB initially unless evidence requires more.

### wishes_gpu_v3 -- the internal ComfyUI provider

ComfyUI GPU Job profile:

```text
NVIDIA L4 x1
4 vCPU
16 GiB RAM
1 task
parallelism 1
retries 0
timeout 3600s
zonal redundancy disabled
```

Pin ComfyUI/custom nodes. Use immutable model/workflow/LoRA manifests and checksums. Raw ComfyUI is never public. This is the `wishes_gpu_v3` provider in the multi-provider model above -- it does not replace `local` or `friend_gcp`; all three coexist.

### Multi-version workflow registration (from the `comfy update flow` inbox task)

Register the existing `local` (`v1`, Flux Schnell) and `friend_gcp` (`v2`, Qwen-Image) generate workflows explicitly, and extend the same pattern to `wishes_gpu_v3` once it exists:

- add the target provider to `asset_workflow` and seed it for `v1`/`v2`;
- add `v2` generate/revise workflow JSON files alongside the existing `v1` pair;
- add a version selector to the Character Creator web portal;
- when `v2` is selected for generate or revise, expose a negative-prompt field (`v1`'s Flux Schnell negative input is a no-op; `v2` actually uses it);
- resolve whether a `v1`-generated asset can be revised with `v2` -- if not, lock revise to the asset's generation version and only allow a version change on a fresh generate.

Plan this before executing -- do not implement until triaged, per the inbox task's own instruction.

## Phase 7a — Queue management service (scope to be defined)

As the provider registry grows past two or three ComfyUI instances, per-request routing inside the asset service (`executorRouting.mjs` today) stops being sufficient -- it has no visibility into provider load, availability, or fair distribution across instances.

Introduce a dedicated queue management service responsible for:

- distribution: assigning each generation job to an available, capable provider from the registry;
- tracking: job status/lifecycle across submission, in-flight, and terminal states, independent of which provider is executing it;
- completion: collecting/finalizing results and handing them back to the existing approve/reject/attachment flow unchanged.

Full design (queueing model, provider health/capacity signals, retry and failover policy, relationship to the existing `asset_generation_queue` table and the Phase 6 Pub/Sub transport) is intentionally not specified here -- scope this out in its own planning pass before implementation. Do not build ahead of that plan. Not required for initial S0 completion.

## Phase 8 — Complete asset suite

Required primary assets:

```text
portrait
full_body
icon
thumbnail
card_front
tactical_sprite_sheet
```

Tactical animations: `idle`, `walk`, `run`, `attack`, `cast`, `hit`, `down`, `guard`, `interact` in `front`, `back`, `left`, `right` directions.

Emojis: happy, angry, sad, surprised, confused, determined, injured, laughing.

Only portrait may originate from text alone. Downstream identity assets use the approved portrait or approved derivative as visual authority. Card text/stats are deterministic application rendering. Icons/thumbnails prefer deterministic image processing where appropriate.

Generate, review, approve, and publish one complete suite. Replace the approved portrait and verify descendants become stale and can be regenerated.

## Phase 9 — Validation and evidence

Retain evidence for:

- Terraform plan/apply/re-plan;
- resource inventory;
- IAM/database-grant matrix;
- Operations VM access/no-external-IP;
- VM start/stop and cost behavior;
- Remote Control mobile/browser test;
- local/VM cross-session status-message test;
- four bucket boundaries;
- Cloud SQL migration/rollback;
- Redis flush recovery;
- Pub/Sub duplicate/retry/DLQ/replay;
- Cloud Run scale-to-zero;
- GPU Job execution;
- full asset-suite lineage/review/publication;
- staleness/regeneration;
- budget alerts;
- no static service-account keys;
- no public raw ComfyUI;
- destroy/rebuild procedure.

## Apply gate

Before first `terraform apply`, provide:

```text
Terraform plan
add/change/destroy summary
stateful replacement analysis
resource inventory
IAM matrix
database role/grant matrix
asset migration plan
Operations VM access/security plan
Operations VM current-price estimate at 25/50/100% uptime
Redis/Cloud SQL/storage/GPU cost estimate
quota/regional availability report
rollback/destroy plan
```

Then stop for explicit user approval.

## Definition of done

S0 is complete only when the VM, Remote Control, local/VM messaging, four buckets, `asset` schema, Redis, Pub/Sub, Cloud Run services, private ComfyUI GPU execution (`wishes_gpu_v3`, coexisting with the already-deployed `local`/`friend_gcp` providers), complete asset suite, staleness behavior, budget controls, and destroy/rebuild procedures are implemented and validated, and no required background task depends on the Operations VM remaining powered on. The Phase 7a queue management service is scoped separately and is not required for initial S0 completion.
