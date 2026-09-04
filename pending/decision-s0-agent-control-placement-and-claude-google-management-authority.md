# Human Decision — S0 Agent-Control Placement and `claude-google` GCP Management Authority

Created: 2026-09-04
Status: Approved / effective immediately
Owner: Human
Applies to: `claude-google`, `chatgpt-director`, Agent Control Platform deployment

## Decision 1 — S0 placement

For S0, deploy the independent `agent_control` platform domain inside the existing GCP project:

```text
project: wishes-506905
canonical region: us-central1
```

This is an explicit Human approval of same-project co-location for S0.

The platform remains an independent bounded domain. Same-project placement does **not** mean reusing Wishes application databases or collapsing IAM/service identities.

Required separation remains:

- dedicated `agent_control` database boundary;
- dedicated Agent Control service identities;
- dedicated Gateway/runtime identities and authorization;
- separate Secret Manager access policy for any server-side integration secrets;
- no control-plane tables in `wishes_core`, `wishes_assets`, or `wishes_auth`;
- no reuse of production application credentials merely because resources share a GCP project.

A future dedicated GCP project remains an optional hardening/scaling migration, not an S0 requirement.

## Decision 2 — `claude-google` management authority

The Human authorizes elevation of `claude-google` so it can act as the primary Google Cloud implementation/operator agent for normal Wishes project resource management in `wishes-506905`.

Use a dedicated impersonated management identity rather than making the Operations VM host identity broadly privileged.

Target model:

```text
Claude Code on Operations VM
  -> weak attached host service account
  -> short-lived service-account impersonation
  -> dedicated claude-google management service account
  -> approved project-scoped GCP management permissions
```

No static service-account JSON keys.

## Standing routine authority after bootstrap

Once the management identity is applied, `claude-google` may perform ordinary creation/update/reconciliation/inspection required by approved Wishes work for project resources including Compute Engine, Cloud Run, Cloud SQL, Memorystore/Redis, Pub/Sub, project VPC/networking, Artifact Registry, GCS, Secret Manager metadata/container management, Logging/Monitoring, enabled project services/APIs, Terraform-managed IAM for approved service deployment, and related project runtime resources.

Use least-privilege predefined/custom roles sufficient for normal operations; do not use primitive `Owner` or `Editor` as the routine solution.

## Human-gated authority retained

Explicit Human approval remains required for:

- deleting the GCP project;
- primitive Owner/Editor grants;
- organization/folder-level IAM expansion;
- material billing-account authority changes;
- creation/download of long-lived service-account keys;
- public-access/security-boundary weakening (`allUsers`, `allAuthenticatedUsers`, public private-service endpoints, disabling IAP/OS Login/MFA/public-access prevention);
- destructive deletion of authoritative database/storage data;
- disabling backups/PITR on authoritative data;
- destructive production/irreplaceable infrastructure operations;
- bypassing protected CI/CD production controls;
- materially expanding another agent's authority;
- broad secret-payload export/read access not required by an approved task.

## Immediate execution authorization

`claude-google` is authorized to begin the management bootstrap now.

Sequence:

1. Determine the Operations VM attached service account and current IAM from what is available.
2. Derive the exact least-privilege management role matrix from current Terraform and required S0/Agent Control operations.
3. Prepare authoritative Terraform for the dedicated management service account, project roles, and host-to-management impersonation binding.
4. If the current host identity cannot create/apply the bootstrap, emit the exact minimal one-time Human command(s) required. Do not ask for a credential transfer.
5. Present the concrete role matrix and Terraform plan before the first durable IAM elevation apply.
6. After Human confirmation of that concrete matrix, apply and validate impersonation.
7. Resume live GCP inventory and Agent Control implementation using the elevated identity.

The architecture decision to elevate is already approved; the remaining checkpoint is review of the concrete permissions before first IAM apply.
