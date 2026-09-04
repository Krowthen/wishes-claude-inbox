# Claude Google — GCP Read Bootstrap Complete / Final Management IAM Plan

Created: 2026-09-04
Priority: Immediate
Assigned Agent: claude-google
Mode: live-inventory-reconciliation-and-management-iam-plan

## Human-completed bootstrap

The Human has completed the short-lived impersonation bootstrap for:

- project: `wishes-506905`
- region: `us-central1`
- weak host identity: `wishes-s0-claude-ops-host@wishes-506905.iam.gserviceaccount.com`
- management identity: `wishes-claude-google-admin@wishes-506905.iam.gserviceaccount.com`

The host identity and Human account can impersonate the management identity through `roles/iam.serviceAccountTokenCreator`. No static service-account key was created.

Initial inventory roles granted to the management identity:

- `roles/viewer`
- `roles/iam.securityReviewer`
- `roles/secretmanager.viewer`
- `roles/artifactregistry.reader`

Human-side impersonation test succeeded.

## Live inventory confirmed by the Human

The management identity successfully read all of the following through service-account impersonation:

- project metadata for `wishes-506905`;
- Compute Engine instance `wishes-s0-usc1-claude-ops` in `us-central1-a`, running with no external IP shown;
- Cloud Run services `wishes-comfy-broker` and `wishes-comfy-worker` in `us-central1`;
- Cloud SQL instance `wishes-s0-usc1-postgres`, PostgreSQL 16, `db-f1-micro`, private address `10.238.16.3`, RUNNABLE;
- Memorystore Redis `wishes-s0-usc1-redis`, Redis 7.2 BASIC 1 GiB on `wishes-s0-vpc`, READY;
- Pub/Sub topics `wishes-s0-asset-dead-letter`, `wishes-s0-asset-results`, and `wishes-s0-asset-requests`;
- Secret Manager metadata for the existing Cloudflare Access and Wishes database password secrets, without reading secret payloads;
- Artifact Registry repository `wishes-services` in `us-central1`;
- full project IAM policy.

This resolves the Step 02 live-inventory visibility blocker.

## Architecture correction / placement

The Human approved co-location in the same GCP project for S0, but the Agent Control Platform remains an independent bounded domain.

Correct target:

```text
GCP project: wishes-506905
Region: us-central1
Domain: agent_control
Database boundary: independent agent_control database/domain
```

Do not revive the obsolete `wishes_ops` application-database model. Co-location in the same GCP project does not mean mixing Agent Control tables into Wishes application databases.

## Immediate required work

1. Re-run the complete live inventory from the Google Operations VM using explicit impersonation of `wishes-claude-google-admin@wishes-506905.iam.gserviceaccount.com`.
2. Inventory storage buckets, Terraform remote-state ownership, networking/private-service-access, enabled APIs, relevant runtime service accounts, Cloud Build requirements, and any remaining GCP services used by the current Terraform stacks.
3. Reconcile live state against Terraform before proposing writes.
4. Produce the exact final standing management IAM matrix needed for routine `claude-google` cloud engineering.
5. Prefer predefined least-privilege roles and resource-scoped bindings where practical.
6. Explicitly identify permissions needed only for occasional Human-approved IAM/security work rather than including them in standing authority.
7. Preserve these Human-only/security gates unless separately approved:
   - project deletion;
   - primitive Owner/Editor grants;
   - project/folder/org IAM authority expansion;
   - service-account key creation;
   - public-access/security-boundary weakening;
   - destructive authoritative data deletion;
   - blanket secret-payload read access.
8. For deploying workloads with runtime service accounts, identify the exact runtime service accounts requiring `roles/iam.serviceAccountUser` (`iam.serviceAccounts.actAs`) and request/grant that role only on those service-account resources.
9. For GCS, inventory buckets first and distinguish Terraform-state access from application-data access. Avoid project-wide object access unless justified.
10. Return the exact PowerShell/gcloud or Terraform changes required for the final Human security checkpoint.

## Candidate routine management roles to evaluate

Evaluate, do not blindly assume, at least:

- `roles/run.admin`
- `roles/compute.admin`
- `roles/cloudsql.admin`
- `roles/redis.admin`
- `roles/pubsub.admin`
- `roles/artifactregistry.admin`
- `roles/secretmanager.editor` for Secret Manager create/update/version management without payload access; keep `roles/secretmanager.viewer` as needed for metadata inspection
- `roles/serviceusage.serviceUsageAdmin`
- `roles/logging.admin`
- `roles/monitoring.admin`
- networking-specific roles required by the actual Terraform stacks
- Cloud Build role(s) only if builds are executed through Cloud Build
- storage roles scoped to the actual Terraform-state/application buckets where feasible

Do not include `roles/secretmanager.admin` in standing authority because that predefined role includes `secretmanager.versions.access` and therefore can read secret payloads. Do not include `roles/owner`, `roles/editor`, `roles/resourcemanager.projectIamAdmin`, `roles/iam.serviceAccountKeyAdmin`, or blanket `roles/secretmanager.secretAccessor` in standing authority without a new explicit Human approval.

## Completion output

Report:

- successful VM-side impersonation;
- complete live resource inventory;
- Terraform/state ownership map;
- exact proposed standing management roles;
- exact resource-scoped `Service Account User` bindings;
- exact GCS/storage bindings;
- any permissions intentionally withheld behind Human gates;
- exact Human bootstrap commands still required, if any;
- next Agent Control deployment step unblocked.
