# Task: Resume Live GCP Management and Agent Control Deployment

Created: 2026-09-04
Priority: Immediate
Assigned Agent: claude-google
Execution Environment: Google Cloud Claude Code Operations VM
Mode: live-reconciliation-and-implementation
Allow Edit: true

## Human-completed GCP bootstrap

The Human has completed the GCP permission bootstrap for project:

- project: `wishes-506905`
- region: `us-central1`

Management identity:

```text
wishes-claude-google-admin@wishes-506905.iam.gserviceaccount.com
```

Weak host identity:

```text
wishes-s0-claude-ops-host@wishes-506905.iam.gserviceaccount.com
```

The weak host identity may impersonate the management identity through short-lived service-account credentials. No static service-account key was created.

From this point forward, routine privileged GCP operations must use explicit impersonation of:

```text
wishes-claude-google-admin@wishes-506905.iam.gserviceaccount.com
```

Do not rely on the weak host identity for privileged cloud management.

## Current standing project roles

The management identity currently has:

```text
roles/artifactregistry.admin
roles/artifactregistry.reader
roles/cloudsql.admin
roles/compute.admin
roles/iam.securityReviewer
roles/logging.admin
roles/monitoring.admin
roles/pubsub.admin
roles/redis.admin
roles/run.admin
roles/secretmanager.editor
roles/secretmanager.viewer
roles/serviceusage.serviceUsageAdmin
roles/viewer
```

It does not have, and must not independently obtain:

```text
roles/owner
roles/editor
roles/resourcemanager.projectIamAdmin
roles/iam.serviceAccountKeyAdmin
blanket roles/secretmanager.secretAccessor
organization/folder-wide IAM authority
```

## Known GCS resources

Human inventory confirmed these buckets:

Terraform state:

```text
gs://wishes-506905-wishes-s0-tfstate
```

Wishes S0 application/asset buckets:

```text
gs://wishes-506905-wishes-s0-assets
gs://wishes-506905-wishes-s0-inputs
gs://wishes-506905-wishes-s0-models
gs://wishes-506905-wishes-s0-outputs
gs://wishes-506905-wishes-s0-workflows
```

Cloud Build bucket:

```text
gs://wishes-506905_cloudbuild
```

Do not assume the Cloud Build bucket should be modified or granted new access merely because it exists.

The Human has been provisioning resource-scoped object access to the Terraform and Wishes S0 buckets. Verify actual bindings from the VM before relying on them.

## Known service accounts

Human inventory confirmed:

```text
wishes-comfy-worker@wishes-506905.iam.gserviceaccount.com
910633976836-compute@developer.gserviceaccount.com
wishes-claude-agent@wishes-506905.iam.gserviceaccount.com
wishes-s0-asset-runtime@wishes-506905.iam.gserviceaccount.com
wishes-s0-app-runtime@wishes-506905.iam.gserviceaccount.com
wishes-s0-comfyui-runtime@wishes-506905.iam.gserviceaccount.com
wishes-claude-google-admin@wishes-506905.iam.gserviceaccount.com
wishes-comfy-broker@wishes-506905.iam.gserviceaccount.com
wishes-s0-claude-ops-host@wishes-506905.iam.gserviceaccount.com
```

The Human has been provisioning `roles/iam.serviceAccountUser` (`iam.serviceAccounts.actAs`) resource-scoped access for approved Wishes runtime service accounts. Verify actual bindings. Do not request project-wide `Service Account User` unless resource-scoped bindings are demonstrably insufficient.

Do not add `actAs` on the weak host account, the management account itself, or the default compute service account unless an approved Terraform requirement proves it is necessary.

## Immediate execution requirements

### 1. Verify VM-side impersonation

From the Operations VM, confirm short-lived impersonation works using the management identity.

All privileged `gcloud` operations should be explicit about project and impersonation identity.

Preferred pattern:

```text
gcloud ... \
  --project=wishes-506905 \
  --impersonate-service-account=wishes-claude-google-admin@wishes-506905.iam.gserviceaccount.com
```

Do not create or download service-account JSON keys.

### 2. Re-run complete live GCP inventory

Inventory at minimum:

- project IAM;
- enabled APIs;
- Compute Engine;
- the Operations VM;
- VPCs;
- subnets;
- firewall rules;
- reserved/global addresses;
- private service access/service networking;
- Cloud NAT if present;
- Cloud Run;
- Cloud SQL;
- Redis / Memorystore;
- Pub/Sub;
- Artifact Registry;
- Secret Manager metadata only; do not read payloads;
- GCS buckets and relevant IAM;
- Terraform remote state access;
- relevant service accounts and IAM bindings;
- Cloud Build usage;
- Logging / Monitoring;
- any existing Agent Control resources.

Do not stop because the earlier Step 02 report said live inventory was impossible. That blocker has been resolved by the Human.

### 3. Verify Terraform state access

Confirm the management identity can read and write the Terraform state objects required by the existing Wishes Terraform stacks.

Do not manually edit Terraform state objects.

Do not apply Terraform until live state and Terraform ownership have been reconciled.

### 4. Reconcile live GCP state against Terraform

Identify and document:

- resources managed by Terraform;
- resources existing outside Terraform;
- duplicate ownership risk;
- drift;
- stale local state;
- missing imports;
- resources that must not be recreated;
- dependencies between stacks.

Before any apply, establish which Terraform stack owns each relevant IAM binding, bucket, service account, network object, Cloud SQL/Redis resource, and service deployment.

### 5. Verify resource-scoped runtime `actAs`

Verify `roles/iam.serviceAccountUser` for the management identity on approved Wishes runtime service accounts, including as applicable:

```text
wishes-comfy-broker@wishes-506905.iam.gserviceaccount.com
wishes-comfy-worker@wishes-506905.iam.gserviceaccount.com
wishes-s0-asset-runtime@wishes-506905.iam.gserviceaccount.com
wishes-s0-app-runtime@wishes-506905.iam.gserviceaccount.com
wishes-s0-comfyui-runtime@wishes-506905.iam.gserviceaccount.com
wishes-claude-agent@wishes-506905.iam.gserviceaccount.com
```

If one is not actually required for deployment, report that and avoid unnecessary expansion.

### 6. Verify resource-scoped GCS authority

Confirm actual access to:

```text
gs://wishes-506905-wishes-s0-tfstate
gs://wishes-506905-wishes-s0-assets
gs://wishes-506905-wishes-s0-inputs
gs://wishes-506905-wishes-s0-models
gs://wishes-506905-wishes-s0-outputs
gs://wishes-506905-wishes-s0-workflows
```

Prefer bucket-scoped roles. Do not request blanket project-wide Storage Admin unless there is a concrete approved requirement.

Treat `gs://wishes-506905_cloudbuild` separately. It uses different/legacy bucket access behavior and must not be modified just to make permissions uniform.

### 7. Determine whether Cloud Build is actually required

Inspect current Wishes deployment/build workflows and recent Cloud Build usage.

If Wishes does not currently require Cloud Build, do not request Cloud Build Admin solely because the `_cloudbuild` bucket exists.

If Cloud Build is required, identify the narrowest required role(s), exact build identity, and any service-account `actAs` requirements before requesting additional Human IAM work.

### 8. Handle any remaining `PERMISSION_DENIED` precisely

For every remaining permission failure:

- record the exact command;
- record the missing permission if Google reports it;
- identify the narrowest predefined role supplying it;
- determine whether the role can be resource-scoped;
- distinguish routine project-management authority from a retained Human security gate;
- never request Owner/Editor as a workaround.

Return to the Human only if a genuinely new security-boundary grant is required.

### 9. Update `claude-google` operating instructions

Update the appropriate `CLAUDE.md`, operations instructions, and/or wrappers so routine cloud work is visibly executed as:

```text
claude-google
  -> wishes-claude-google-admin
  -> wishes-506905
```

Do not depend silently on ambient `gcloud` account/project defaults.

Where practical, provide/reconcile wrappers such as:

```text
wishes-gcp
wishes-terraform
```

that make project, region, and impersonation identity explicit.

### 10. Resume Agent Control Platform deployment

Once live-state/Terraform reconciliation is complete, resume the approved Agent Control Platform deployment sequence.

Current S0 placement is final:

```text
GCP project: wishes-506905
Region: us-central1
Domain: agent_control
```

The Agent Control Platform remains an independent bounded domain/database/service/IAM boundary even though it is co-located in the Wishes project for S0.

Do not revive the obsolete `wishes_ops` application-database model.

Do not create a separate GCP project solely for S0 Agent Control.

Use these authoritative references together:

```text
pending/reference-agent-control-platform-revised-approved-design.md
pending/agent-control-platform-master-deployment-plan.md
pending/decision-s0-agent-control-placement-and-claude-google-management-authority.md
pending/agent-control-step-03-domain-placement-claude-google.md
```

### 11. Preserve Human-only gates

Stop for explicit Human approval before:

- deleting the GCP project;
- materially changing billing-account authority;
- granting Owner or Editor;
- expanding project/folder/org IAM trust boundaries;
- creating long-lived service-account keys;
- making private services public;
- weakening IAP, OS Login, MFA, public-access prevention, or equivalent controls;
- destructive deletion of authoritative Cloud SQL/GCS data;
- disabling backups/PITR;
- reading/exporting secret payloads unless a specific approved task requires it;
- materially expanding another agent's authority;
- destructive production/irreplaceable infrastructure changes.

Routine create/update/reconcile operations inside `wishes-506905` that are already covered by the approved standing role set may proceed without repeated Human approval.

## Required completion report

Report all of the following:

- VM-side impersonation result;
- management identity used;
- complete live resource inventory;
- Terraform/state ownership map;
- drift/conflicts/imports required;
- Terraform-state access verification;
- bucket IAM/access verification;
- runtime service-account `actAs` verification;
- Cloud Build requirement determination;
- remaining permission gaps, if any;
- any additional role requested and exact reason;
- permissions deliberately withheld;
- `CLAUDE.md` / wrapper changes;
- branches and commit SHAs;
- Agent Control deployment step now active;
- any Human approval currently required.

Begin immediately. Do not wait for a separate copy/paste instruction from the Human; this inbox task is the execution instruction.