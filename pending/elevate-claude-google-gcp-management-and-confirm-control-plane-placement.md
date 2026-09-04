# Task: Elevate `claude-google` GCP Management Authority and Confirm Control-Plane Placement

Created: 2026-09-04
Priority: High
Mode: implementation-with-human-security-gates
Assigned Agent: claude-google
Allow Edit: true

## Human-approved decisions

The Human has explicitly approved both of the following:

1. The Wishes Agent Control Plane may remain in the existing Google Cloud project:
   - project: `wishes-506905`
   - canonical S0 region: `us-central1`
   - do not create a separate GCP project solely for the control plane at this stage.

2. `claude-google` should be elevated from its current inspection-limited posture so it can manage the Wishes Google Cloud environment as the primary cloud implementation/operator agent.

This approval is intentionally broader than the previous read-only/reconciliation posture, but it does **not** remove the existing Human authority boundary for destructive or security-critical actions.

## Current finding to reconcile

Step 02 security/domain reconciliation reported:

- the VM service account has `cloud-platform` OAuth scope;
- its effective IAM roles are still narrow;
- live inspection of Cloud SQL, Redis, Pub/Sub, Cloud Run, Secret Manager, IAM, service accounts, Compute Engine, and project IAM was denied;
- only local Terraform state and prior documentation were available as ground truth;
- no existing Agent Control Plane implementation exists in the inspected repositories;
- the current control-plane implementation is therefore greenfield.

Do not confuse OAuth scope with effective IAM authority. IAM roles remain the actual permission boundary.

## Required target authority model

Preserve the existing rule that the Operations VM attached service account remains intentionally weak.

Do **not** solve this by granting the attached VM service account primitive `Owner` or `Editor`.

Instead implement a dedicated impersonated management identity for `claude-google`.

Recommended logical identity:

```text
wishes-claude-google-admin@wishes-506905.iam.gserviceaccount.com
```

Exact name may be adjusted to existing naming conventions, but the role must remain clearly attributable to `claude-google`.

Target flow:

```text
Claude Code on Operations VM
        |
        v
weak attached VM service account
        |
        | narrowly scoped Service Account Token Creator / impersonation grant
        v
claude-google management service account
        |
        v
wishes-506905 project resources
```

No static service-account JSON keys are permitted.

## Standing management scope

The dedicated `claude-google` management identity should be able to perform normal Wishes project-management and implementation work for resources such as:

- Compute Engine resources used by Wishes development/operations;
- Cloud Run services/jobs;
- Cloud SQL instances/databases/configuration;
- Memorystore Redis;
- Pub/Sub;
- VPC/subnets/firewall/NAT/DNS resources that belong to the Wishes project;
- Artifact Registry;
- GCS application/infrastructure buckets;
- Secret Manager containers and approved secret lifecycle operations;
- Cloud Logging/Monitoring resources;
- Terraform-managed project resources;
- relevant service accounts and IAM bindings required to deploy approved Wishes services;
- required project APIs/services;
- other project-level runtime resources required by the Wishes architecture.

Prefer the narrowest practical predefined-role set or reviewed custom roles rather than primitive project-wide `Owner`/`Editor`.

Because actual required roles depend on the current Terraform and live resource inventory, first derive the exact permissions from current stacks and planned Agent Control Plane deployment, then implement the least-privilege role set that still lets `claude-google` perform normal cloud engineering without repeated permission failures.

## Human-gated actions that remain protected

Even after elevation, `claude-google` must stop and obtain explicit Human approval before performing any of the following unless a later approved policy narrows these gates:

- deleting the GCP project;
- changing billing-account ownership/attachment in a way that materially changes billing authority;
- granting primitive `Owner` or `Editor` to any identity;
- granting broad organization-level or folder-level IAM;
- creating or downloading long-lived service-account keys;
- weakening IAP, OS Login, MFA, public-access prevention, or equivalent security boundaries;
- making a private Cloud Run/service/database endpoint public unless specifically approved;
- granting `allUsers` or `allAuthenticatedUsers`;
- destructive deletion of authoritative Cloud SQL data or storage data;
- disabling backups/PITR on authoritative databases;
- destroying production or irreplaceable infrastructure/data;
- bypassing protected CI/CD production controls;
- materially expanding another agent's authority;
- reading or exporting secret payloads that are not required for the explicitly approved task.

Routine creation/update/reconciliation of normal project resources should **not** require a separate Human approval each time once this standing management identity is deployed.

## Secret Manager boundary

Design Secret Manager access deliberately.

Where practical, distinguish:

- managing secret containers, IAM, metadata, replication, and versions;
- adding a secret version when explicitly required by an approved workflow;
- reading secret payloads.

Do not grant blanket secret-payload read access merely because project-resource management is being enabled. If a workflow truly requires secret payload access, document the specific secret and reason and apply the narrowest permission possible.

## IAM implementation requirements

1. Inventory the current live identity used by the Operations VM.
2. Inventory existing relevant service accounts and IAM grants.
3. Reconcile against Terraform ownership before making changes.
4. Determine whether the dedicated management SA already exists under another approved name.
5. If absent, add it through the authoritative Terraform stack.
6. Grant the VM's attached identity only the impersonation permission required to obtain short-lived credentials for the management SA.
7. Grant the management SA the reviewed project-management roles required by current Wishes Terraform and operations.
8. Avoid duplicate IAM ownership across Terraform states.
9. Do not use manual one-off IAM commands if Terraform is authoritative for that binding, except for a clearly documented bootstrap action that is subsequently imported/reconciled.
10. Preserve audit logging and attribution.
11. Add explicit environment wrappers/configuration so cloud commands identify the target project/environment rather than depending on ambient defaults.

## Required `claude-google` operating convention

Update the Google Operations VM / repository instructions so routine Google Cloud work uses explicit impersonation of the management identity.

The operational convention should make it obvious when commands are running as:

```text
claude-google / management identity
```

rather than the weak host identity.

Where practical, provide wrappers such as:

```text
wishes-gcp <command>
wishes-terraform <environment> <command>
```

or the repository's existing equivalent.

Do not silently change projects based on ambient `gcloud` configuration.

## Control-plane placement update

Record the Human decision that Agent Control Plane S0 resources remain inside `wishes-506905`.

Reconcile any open design notes that still treat a dedicated GCP operations/control-plane project as an undecided requirement.

The S0 target is:

```text
project: wishes-506905
region: us-central1

Agent Gateway / MCP / A2A / Design Room runtime
wishes_ops database
Redis coordination
logging/monitoring
supporting IAM/service accounts
```

A future project split remains possible if scale, compliance, blast-radius, or organizational requirements justify it, but it is **not** part of the current deployment.

## Stale-file cleanup

The Step 02 report flagged a stale reference in:

```text
canonize-agent-control-plane-after-acceptance-claude-google.md
```

that still references the superseded `wishes_ops` model.

Do not fix unrelated downstream implementation prematurely, but update or supersede this stale task/document when the corrected implemented model is known so the later canonization step does not reintroduce obsolete architecture.

## Implementation sequence

### Phase 1 — live inventory

Using the Human-approved elevation bootstrap path available to you, inventory:

- project IAM;
- Operations VM identity;
- service accounts;
- enabled APIs;
- Cloud Run;
- Compute Engine;
- Cloud SQL;
- Redis;
- Pub/Sub;
- Secret Manager metadata/IAM without exposing payloads;
- Artifact Registry;
- storage buckets;
- networking;
- relevant Terraform state ownership.

If the current identity still cannot inspect enough to construct the role set, produce the exact minimal Human bootstrap command(s) required to grant the one-time impersonation/bootstrap permission. Stop only where interactive Human execution is genuinely required.

### Phase 2 — Terraform IAM design

Create/reconcile the dedicated `claude-google` management identity and role grants in authoritative Terraform.

Produce a plan showing:

- identities created/changed;
- roles added/removed;
- impersonation chain;
- any sensitive permissions;
- no primitive Owner/Editor grants unless explicitly requested later;
- no service-account keys.

### Phase 3 — Human security checkpoint

Before applying the IAM elevation itself, present the exact proposed IAM matrix and Terraform plan to the Human.

The Human has approved the **intent to elevate** `claude-google`, but must still be shown the concrete role set before first apply because this is a durable security-boundary change.

Do not require another architecture decision about whether elevation is desired; that decision is already approved.

### Phase 4 — apply elevation

After Human confirms the concrete role matrix:

- apply the Terraform/IAM change;
- authenticate/impersonate from the Operations VM;
- verify short-lived credentials;
- verify no static keys exist;
- verify the weak host identity alone still cannot perform broad management;
- verify the impersonated management identity can perform required read/manage actions.

### Phase 5 — validation

At minimum validate the ability to inspect and, through Terraform plan or a harmless non-destructive operation, manage:

- Cloud Run;
- Compute Engine;
- Cloud SQL;
- Redis;
- Pub/Sub;
- networking;
- Artifact Registry;
- GCS;
- Secret Manager metadata;
- IAM required by approved Terraform.

Do not use destructive validation.

### Phase 6 — resume Agent Control Plane work

Once management access is functional:

- continue the approved Agent Control Plane implementation from the next open dependency step;
- use live GCP inventory rather than stale local-only assumptions;
- keep all existing Human gates for destructive/security-critical operations.

## Required documentation changes

Update the appropriate authoritative/draft operations documentation and repository instructions to record:

- runtime name `claude-google`;
- dedicated management identity;
- impersonation model;
- standing normal-resource-management authority;
- retained Human security/destructive gates;
- Agent Control Plane placement in `wishes-506905`;
- no service-account keys;
- Git/Terraform as authoritative engineering state.

Do not silently canonize a draft if repository workflow requires Human approval.

## Required final report

Report:

- live inventory obtained;
- current host identity;
- dedicated management identity name;
- exact IAM roles granted;
- exact impersonation grant;
- Terraform files/stacks changed;
- plan/apply result;
- validation results by GCP service;
- any remaining permission gaps;
- any permissions deliberately withheld;
- security gates retained;
- documentation changed;
- branches/commit SHAs;
- next Agent Control Plane task now unblocked.

## Completion criteria

This task is complete when:

- control-plane S0 placement is explicitly recorded as `wishes-506905`;
- `claude-google` has a dedicated short-lived impersonated management identity;
- routine Wishes GCP management no longer fails due to the current narrow host IAM role;
- the host service account itself remains weak;
- no static service-account key exists;
- the concrete IAM elevation has been Human-reviewed before first apply;
- destructive/security-boundary actions remain Human-gated;
- the Agent Control Plane implementation can resume using live GCP state.
