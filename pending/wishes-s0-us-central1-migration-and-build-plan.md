# Wishes S0 — Full Migration and Build Plan to `us-central1`

## Objective

Move the entire Wishes S0 cloud architecture to a single canonical region:

```text
us-central1
```

This plan covers migration of all currently deployed regional infrastructure from `us-west1`, preservation of global/project-level resources, revision of all not-yet-applied S0 Terraform to `us-central1`, migration of the existing ComfyUI broker/worker path, creation of all new S0 infrastructure in `us-central1`, validation, cutover, rollback, and retirement of `us-west1`.

The target is a single-region S0 architecture with no planned regional exception.

## 1. Canonical decisions

Lock these before implementation:

```text
GCP project: wishes-506905
Canonical S0 region: us-central1
Canonical S0 zone: select one us-central1 zone during live validation
GPU region: us-central1
All new regional infrastructure: us-central1
```

Do not create a second GCP project and do not introduce a permanent `us-west1` / `us-central1` split.

## 2. Final target architecture

```text
                    GCP PROJECT: wishes-506905
                             |
                         us-central1
                             |
        +--------------------+--------------------+
        |                    |                    |
     VPC/Subnet          Cloud SQL             Redis
        |                    |                    |
        +--------------------+--------------------+
                             |
                    Pub/Sub transport
                             |
             +---------------+---------------+
             |                               |
     Wishes Application               Asset Service
                                             |
                        +--------------------+--------------------+
                        |                                         |
                  friend_gcp                               wishes_gpu_v3
                        |                                         |
              wishes-comfy-worker                    Cloud Run L4 GPU Job
                        |
              wishes-comfy-broker
                        |
                 Cloudflare Access
                        |
                Friend's ComfyUI

Shared storage:
- models
- workflows
- inputs
- outputs

Operations:
- Claude Code Operations VM
- Terraform remote state
- Artifact Registry
- Secret Manager
- Logging / Monitoring
- Billing budget
```

## 3. Existing infrastructure classification

### Existing resources that do not need a regional move

Preserve these project-level/global resources:

```text
GCP project: wishes-506905

Service accounts:
wishes-comfy-broker@wishes-506905.iam.gserviceaccount.com
wishes-comfy-worker@wishes-506905.iam.gserviceaccount.com
wishes-claude-agent@wishes-506905.iam.gserviceaccount.com

Secret Manager containers:
comfy-cf-access-client-id
comfy-cf-access-client-secret

Cloudflare Access configuration: unchanged
```

Preserve the least-privilege IAM model. Do not recreate these resources unless Terraform reconciliation proves they are unmanaged.

### Existing regional resources that must move

```text
Cloud Run:
wishes-comfy-broker
wishes-comfy-worker

Artifact Registry:
current us-west1 Docker repository `wishes-services`
```

The broker and worker must be redeployed in `us-central1`. A new `us-central1` Artifact Registry repository should become canonical.

## 4. New S0 infrastructure to create in `us-central1`

### Foundation

```text
Terraform state bucket
Custom VPC
Regional subnet
Private Google Access
Optional Cloud NAT, default OFF until approved
Artifact Registry
```

### Data

```text
Cloud SQL PostgreSQL, single-zone/shared-core S0 profile

Databases:
wishes_core
wishes_assets
wishes_auth

Database roles:
wishes_core_owner
wishes_core_runtime
wishes_assets_owner
wishes_assets_runtime
wishes_auth_owner
wishes_auth_runtime
scoped auditor roles
```

### Coordination / transport

```text
Memorystore Redis Basic 1 GiB

Pub/Sub topics:
wishes-s0-asset-requests
wishes-s0-asset-results
wishes-s0-asset-dead-letter

Pub/Sub subscriptions:
wishes-s0-asset-dispatch
wishes-s0-asset-result-reconciler
wishes-s0-asset-dead-letter-monitor
```

### Storage

Exactly four canonical application buckets:

```text
wishes-506905-wishes-s0-models
wishes-506905-wishes-s0-workflows
wishes-506905-wishes-s0-inputs
wishes-506905-wishes-s0-outputs
```

Use provider/version prefixes inside these buckets rather than provider-specific buckets.

Example:

```text
models/wishes_gpu_v3/...
workflows/local/v1/...
workflows/friend_gcp/v2/...
workflows/wishes_gpu_v3/v3/...
inputs/<asset-id>/...
outputs/<provider>/<workflow-version>/<asset-id>/...
```

### Compute

```text
Cloud Run Wishes application service
Cloud Run asset service
Cloud Run L4 GPU Job: wishes_gpu_v3
Claude Code Operations VM
```

### Operations

```text
Cloud Logging / Monitoring defaults
Billing budget
IAM matrix
Secret Manager references
Destroy/rebuild runbooks
```

## 5. Terraform strategy

### 5.1 Change canonical region everywhere

Update all S0 Terraform defaults:

```hcl
region = "us-central1"
```

Update naming where the region abbreviation is embedded. For example:

```text
wishes-s0-usw1-claude-ops
```

becomes:

```text
wishes-s0-usc1-claude-ops
```

### 5.2 Preserve existing broker/worker Terraform ownership

Do not declare duplicate broker/worker resources in new S0 stacks. Continue managing:

```text
infrastructure/terraform/comfy-broker
infrastructure/terraform/comfy-worker
```

through their existing states. Change their desired region to `us-central1` only as part of the explicit migration.

### 5.3 Avoid cross-state IAM ownership conflicts

The S0 asset-service stack may output the asset-service runtime SA email, but the worker stack remains authoritative for the worker's Cloud Run IAM policy. Add the new caller to the worker stack's existing `caller_service_account_emails` mechanism. No second Terraform state should manage the same Cloud Run IAM binding.

### 5.4 Remote state

Create the S0 Terraform-state bucket first, then move new S0 stacks to the GCS backend.

Do not move the broker/worker state into a shared backend until their current state is backed up, imports/state movement are rehearsed, and zero replacement is demonstrated. It is acceptable for broker/worker to retain their own state initially if that is safer.

## 6. Migration Phase A — live inventory and validation

Before any apply:

1. authenticate Terraform ADC
2. inventory current Cloud Run services
3. inventory Artifact Registry repositories/images
4. inventory service accounts
5. inventory Secret Manager secrets
6. inventory IAM bindings
7. confirm billing project
8. confirm required APIs
9. validate `us-central1` availability for Cloud Run CPU, Cloud Run L4 GPU, Cloud SQL PostgreSQL, Memorystore Redis, and Compute Engine e2-standard-2
10. check quotas for Cloud Run GPU, Compute Engine, Cloud SQL, Redis, VPC addresses, and Artifact Registry
11. run all Terraform plans

Hard gate:

```text
0 unintended destroys
0 unintended replacements
```

Any replacement of an existing broker/worker must be deliberate and part of the migration plan, not accidental state drift.

## 7. Migration Phase B — prepare `us-central1` foundation

Create, in this order:

1. Terraform state bucket
2. Artifact Registry repository in `us-central1`
3. VPC
4. `us-central1` subnet
5. Private Google Access
6. required APIs
7. budget/monitoring foundations

Do not enable Cloud NAT yet unless explicitly approved.

## 8. Migration Phase C — copy/rebuild container images

Canonical repository:

```text
us-central1-docker.pkg.dev/wishes-506905/wishes-services
```

Move or reproducibly rebuild:

```text
wishes-comfy-broker
wishes-comfy-worker
future Wishes application
future asset service
future wishes_gpu_v3 image
```

Record immutable digests for each image. Keep the `us-west1` images until migration acceptance.

## 9. Migration Phase D — deploy central broker

Deploy:

```text
wishes-comfy-broker
region: us-central1
runtime SA: wishes-comfy-broker@wishes-506905.iam.gserviceaccount.com
```

Reuse the existing project-level Secret Manager secrets. Do not create new Cloudflare credentials.

Validate:

```text
GET /health
GET /system-stats
```

Expected chain:

```text
human test identity
 -> central broker
 -> Secret Manager
 -> Cloudflare Access
 -> friend's ComfyUI
```

Do not remove the west broker yet.

## 10. Migration Phase E — deploy central worker

Deploy:

```text
wishes-comfy-worker
region: us-central1
runtime SA: wishes-comfy-worker@wishes-506905.iam.gserviceaccount.com
```

Change `COMFY_BROKER_URL` and `COMFY_BROKER_AUDIENCE` to the new `us-central1` broker URL.

The worker must retain `roles/run.invoker` on the central broker only.

Validate:

```text
GET worker /health
GET worker /system-stats
```

Expected chain:

```text
caller
 -> central worker
 -> central broker
 -> Cloudflare
 -> friend ComfyUI
```

Verify again that the worker cannot access Cloudflare Secret Manager secrets, cannot impersonate the broker SA, and cannot update the broker.

## 11. Migration Phase F — application data platform

After central broker/worker are proven:

### Cloud SQL

Create the single S0 Cloud SQL instance in `us-central1` and create:

```text
wishes_core
wishes_assets
wishes_auth
```

Do not migrate data until the multi-database migration convention is finalized.

At S0 there is currently local-development data only; treat migration as schema/data bootstrap unless a specific authoritative local dataset is approved for import.

### Redis

Create Basic 1 GiB Redis in `us-central1`. Use Direct VPC egress from Cloud Run where supported. Redis remains transient only.

### Pub/Sub

Create S0 asset request/result/dead-letter topics and subscriptions. PostgreSQL remains authoritative. Pub/Sub is transport only. Use inbox/outbox and idempotency.

## 12. Migration Phase G — shared storage

Create the four shared buckets aligned to the S0 canonical region.

Apply:

```text
uniform bucket-level access
public access prevention
versioning/lifecycle policy as approved
provider/version prefixes
least-privilege IAM
```

No fifth provider-specific bucket without explicit approval.

## 13. Migration Phase H — deploy Wishes application and asset service

Create dedicated identities:

```text
wishes-s0-app-runtime
wishes-s0-asset-runtime
```

or the exact canonical names already defined in Terraform.

Wishes app should receive only the access it needs for `wishes_core`, approved application secrets, required Pub/Sub access, and required storage access. No auth DB credential unless an auth-service design explicitly permits it.

Asset service should receive only the access it needs for `wishes_assets`, Pub/Sub, asset storage, and `roles/run.invoker` on `wishes-comfy-worker`.

Explicitly omit direct broker invocation, Cloudflare secrets, broker impersonation, and worker impersonation.

Deploy private Cloud Run services with `min instances = 0` and `max instances = 1` unless load testing proves a different S0 value is required.

## 14. Migration Phase I — deploy `wishes_gpu_v3`

Create the Wishes-owned ComfyUI provider in `us-central1`.

Initial profile:

```text
NVIDIA L4 x1
4 vCPU
16 GiB RAM
1 task
parallelism 1
retries 0
timeout 3600 seconds
zonal redundancy disabled
```

Use the models, workflows, inputs, and outputs buckets. Register `provider = wishes_gpu_v3` in the provider registry.

Do not expose raw ComfyUI publicly. Validate one complete generation.

## 15. Migration Phase J — Operations VM

Create:

```text
wishes-s0-usc1-claude-ops
e2-standard-2
2 vCPU
8 GiB RAM
100 GB pd-balanced
Ubuntu 24.04 LTS
no external IP
```

Access controls:

```text
IAP TCP forwarding
OS Login
MFA
Shielded VM
```

Cloud NAT defaults to OFF. Enable only if Phase 2 proves required and the standing monthly cost is explicitly approved. Use Private Google Access for Google APIs. If general outbound internet is required, compare Cloud NAT with a temporary controlled-egress approach or another lower-cost approved alternative before enabling permanent NAT.

## 16. Migration Phase K — application routing cutover

Update all code/config references from the west worker URL to the central worker URL.

Canonical friend path becomes:

```text
Asset Service
 -> central wishes-comfy-worker
 -> central wishes-comfy-broker
 -> friend ComfyUI
```

Workflow/provider model remains:

```text
local
friend_gcp
wishes_gpu_v3
```

`execution_target` remains a derived/snapshot execution field, not an independent arbitrary routing choice.

## 17. Migration Phase L — validation

Must pass before removing any west resource.

### Broker/worker

```text
central broker /health = success
central broker /system-stats = success
central worker /health = success
central worker /system-stats = success
```

### Security

```text
worker cannot access Cloudflare secrets
asset runtime cannot access Cloudflare secrets
asset runtime cannot invoke broker
asset runtime can invoke worker
no allUsers
no allAuthenticatedUsers
no service-account keys
```

### Data

```text
Cloud SQL connections verified
three DB boundaries verified
runtime roles cannot cross DBs
Redis flush/recreate test passes
Pub/Sub retry/DLQ/idempotency test passes
```

### Storage

```text
four buckets exist
public access prevention active
provider/version prefixes work
generated output survives service restart
```

### GPU

```text
wishes_gpu_v3 starts
model loads
workflow runs
output persists to GCS
GPU job terminates
no raw public ComfyUI
```

### Application

```text
local generation remains local
friend_gcp uses central worker
wishes_gpu_v3 uses internal GPU provider
no silent fallback
```

## 18. Migration Phase M — retire `us-west1`

Only after all central acceptance tests pass.

Retire in reverse dependency order:

1. stop sending traffic to west worker
2. verify no west references remain
3. remove west worker
4. remove west broker
5. retain west Artifact Registry temporarily
6. verify no rollback needed
7. delete west Artifact Registry after approved retention period

Do not delete service accounts, Secret Manager secrets, project-level IAM, or Cloudflare Access configuration because they are reused centrally.

## 19. Rollback plan

Until final retirement:

```text
central failure
 -> point callers back to west worker
 -> west worker remains connected to west broker
 -> friend path remains functional
```

Therefore do not delete west broker/worker during initial central deployment, do not overwrite their image tags, keep their last known-good immutable digests, and keep current IAM intact until central validation passes.

For new S0 resources, rollback is Terraform destroy/revert only after confirming no authoritative data needs preservation. Cloud SQL and storage require separate data-protection approval before destructive rollback.

## 20. Cost controls

Keep the S0 target budget:

```text
$150/month
```

Live pricing must separately show:

```text
Cloud SQL idle/month
Redis idle/month
Cloud Run CPU services idle/light-use
Cloud Run L4 GPU cost per minute/hour
Operations VM at 25%, 50%, 100% uptime
Cloud NAT fixed standing cost
Storage
Artifact Registry
network egress
logging
```

NAT remains a separately approved line item. GPU remains scale-to-zero/job-based.

## 21. Required approval gates

### Gate 1 — before any central apply

Review:

```text
live GCP inventory
live quota/availability
Terraform plan
add/change/destroy counts
cost estimate
IAM matrix
DB role matrix
state migration plan
rollback plan
```

### Gate 2 — before database migration

Review:

```text
three-database migration convention
migration ordering
role grants
data backup
rollback
```

### Gate 3 — before west retirement

Review:

```text
central acceptance evidence
all URL/config cutovers
no remaining west dependencies
rollback retention period
```

## 22. Definition of done

The migration is complete when:

```text
Canonical region = us-central1

Current infrastructure:
- broker central
- worker central
- central Artifact Registry

New infrastructure:
- VPC/subnet central
- Terraform state
- Cloud SQL central
- wishes_core
- wishes_assets
- wishes_auth
- Redis central
- Pub/Sub
- four shared buckets
- Wishes app
- asset service
- wishes_gpu_v3 L4 job
- Claude Operations VM
- budget/monitoring

Security:
- no public raw ComfyUI
- no static SA keys
- worker alone can call broker
- asset runtime can call worker, not broker
- Cloudflare secrets remain broker-only

Operations:
- all tests pass
- cost within approved S0 envelope
- rollback documented
- west resources retired only after approval
```

## 23. Immediate next action

Do not run the existing `us-west1` S0 Terraform plan as final.

First update the S0 plan and Terraform defaults to `us-central1`, then run a fresh authenticated live inventory and Terraform plan. Stop at the pre-apply gate.

No infrastructure apply should occur until the updated central-region plan is reviewed and approved.
