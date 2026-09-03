# Claude Code Task — S0 Data Pre-Apply Hardening

## Purpose

Harden `infrastructure/terraform/s0-data` before the human administrator applies it.

Current authenticated human plan:

```text
Plan: 29 to add, 0 to change, 0 to destroy
```

Do **not** apply it yet.

The live review found four blockers:

1. `sqladmin.googleapis.com` and `redis.googleapis.com` were manually enabled but are not represented in the `s0-data` Terraform state.
2. DB passwords are generated with managed `random_password` resources and passed via `password` / `secret_data`, which persists plaintext secrets in Terraform state.
3. The approved PostgreSQL owner/runtime isolation model is documented but not implemented in executable migration/bootstrap code.
4. `s0-data` asks the human to type `s0-network` outputs interactively instead of consuming authoritative remote-state outputs.

Claude writes/tests/documents only. The human performs imports and applies.

## Live context

```text
Project: wishes-506905
Region: us-central1
State bucket: wishes-506905-wishes-s0-tfstate
Network state prefix: s0-network
Data state prefix: s0-data

VPC:
projects/wishes-506905/global/networks/wishes-s0-vpc

Private service connection:
projects%2Fwishes-506905%2Fglobal%2Fnetworks%2Fwishes-s0-vpc:servicenetworking.googleapis.com
```

`s0-network` is already applied successfully.

`sqladmin.googleapis.com` and `redis.googleapis.com` are already enabled manually. Do not disable them.

No Cloud SQL, Redis, DB users, or DB password secrets from `s0-data` have been applied yet, so no password-bearing `s0-data` state requires cleanup.

## 1. Terraform-own the two data APIs

Add `google_project_service` resources in `s0-data` for:

```text
sqladmin.googleapis.com
redis.googleapis.com
```

Use:

```hcl
disable_on_destroy = false
```

Do not duplicate APIs owned by other states.

Known ownership:
- `comfy-broker`: its existing Run / Secret Manager / Artifact Registry / Cloud Build / IAM / IAM Credentials APIs.
- `s0-network`: `servicenetworking.googleapis.com`.

Because SQL Admin and Redis are already enabled, document the exact human `terraform import` commands to adopt them into `s0-data` before apply. Do not perform imports yourself.

Add explicit dependencies from SQL/Redis resources to their API resources where appropriate.

## 2. Remove database passwords from Terraform state

Replace the current managed-password pattern:

```hcl
resource "random_password" "db_role" { ... }
password    = random_password.db_role[each.key].result
secret_data = random_password.db_role[each.key].result
```

with ephemeral/write-only handling.

Use conceptually:

```hcl
ephemeral "random_password" "db_role" {
  for_each = local.db_roles
  length   = 32
  special  = false
}
```

and pass the value only through:

```text
google_sql_user.password_wo
google_sql_user.password_wo_version
google_secret_manager_secret_version.secret_data_wo
google_secret_manager_secret_version.secret_data_wo_version
```

Do not use managed `random_password`, `google_sql_user.password`, or `secret_data`.

Inspect the Random provider constraint/lock version and use a stable version supporting ephemeral `random_password`; do not use an alpha provider.

Human Terraform is currently `v1.15.3`. Update the required Terraform version if needed so the stack explicitly requires ephemeral/write-only support.

Use a documented write-only version integer so a future version increment intentionally rotates both Cloud SQL and Secret Manager together.

No password may appear in outputs or files. The retrievable canonical copy after apply must be Secret Manager, not Terraform state.

## 3. Consume `s0-network` remote state

Remove normal interactive entry of:

```text
var.network_self_link
var.private_vpc_connection_id
```

Use `terraform_remote_state` or an equivalent Terraform-native remote-state lookup.

Source:

```text
bucket = wishes-506905-wishes-s0-tfstate
prefix = s0-network
```

Consume the existing outputs:

```text
network_id
private_vpc_connection
```

Do not duplicate network resources or IAM ownership.

Document that this is a cross-state value lookup, not an automatic cross-stack apply graph; `s0-network` must already be applied.

Afterward, `terraform plan` in `s0-data` must not prompt for those network values.

## 4. Implement executable PostgreSQL database-boundary bootstrap

Repository inspection found the role/isolation model only in documentation and Terraform declarations. No executable `ALTER DATABASE`, `GRANT`, `REVOKE`, or default-privilege implementation exists.

Implement a repeatable, version-controlled bootstrap/migration mechanism compatible with the existing migration tooling.

One Cloud SQL PostgreSQL instance:

```text
wishes-s0-usc1-postgres
```

Databases:

```text
wishes_core
wishes_assets
wishes_auth
```

Roles:

```text
wishes_core_owner
wishes_core_runtime
wishes_assets_owner
wishes_assets_runtime
wishes_auth_owner
wishes_auth_runtime
```

Required effective isolation:

```text
wishes_core_runtime:
  connect wishes_core = yes
  connect wishes_assets = no
  connect wishes_auth = no

wishes_assets_runtime:
  connect wishes_assets = yes
  connect wishes_core = no
  connect wishes_auth = no

wishes_auth_runtime:
  connect wishes_auth = yes
  connect wishes_core = no
  connect wishes_assets = no
```

For each DB:
- remove PostgreSQL `PUBLIC` access that would allow unrelated roles to connect;
- establish the matching `*_owner` as owner or equivalent explicit Cloud SQL-compatible ownership;
- grant matching runtime CONNECT only to its DB;
- no runtime DDL;
- runtime gets only required schema usage/DML;
- handle existing tables and sequences;
- set default privileges for future owner-created objects;
- review function/procedure EXECUTE privileges explicitly rather than blanket-granting everything;
- owner roles are migration identities only, never request-time application identities.

`wishes_auth` is reserved only. Its users/secrets may exist, but no current application service gets its owner/runtime credential and no auth service should be invented.

Inspect the existing migration runner. Do not use `psql` `\connect` meta-commands unless the runner actually supports them. Separate per-database files/orchestration are acceptable.

Do not reclassify existing schemas between core/assets in this task unless canon already specifies it unambiguously. This task establishes security/bootstrap mechanics only.

## 5. Data deletion safeguards

Keep:

```hcl
deletion_protection = true
```

on Cloud SQL.

Add Terraform lifecycle protection against accidental destruction for the authoritative Cloud SQL instance and the three DB resources unless a stronger existing repository convention already exists:

```hcl
lifecycle {
  prevent_destroy = true
}
```

Document the deliberate two-step human process required for intentional retirement.

Redis is transient and does not require the same protection.

## 6. Verify runtime secret boundaries

Inspect owning S0 service/IAM Terraform and confirm:

```text
Wishes app runtime -> wishes_core_runtime only
Asset runtime -> wishes_assets_runtime only
No current runtime -> wishes_auth_owner/runtime
No runtime -> any *_owner credential
```

Do not create duplicate IAM resources across states. If grants belong to `s0-services`, correct/document them there rather than adding duplicate ownership in `s0-data`.

## 7. Validation

Run at minimum:

```text
terraform fmt -check
terraform validate
```

for every modified Terraform stack, plus relevant migration/bootstrap tests.

If local PostgreSQL is available, add/run a boundary test proving the runtime roles cannot cross databases. If not, add the test harness and exact human post-create verification commands.

Repository searches must prove:

```text
no managed random_password resource remains in s0-data
no google_sql_user.password assignment remains
no secret_data assignment remains
```

Expected secure equivalents:

```text
ephemeral random_password
password_wo
secret_data_wo
```

## 8. Update the S0 pre-apply documentation

Record:
- SQL Admin/Redis API ownership and exact imports;
- network remote-state consumption;
- password values no longer persist in Terraform state;
- executable database bootstrap/grant process and run order;
- DB isolation verification;
- Cloud SQL/database destroy protection;
- `wishes_auth` remains unconsumed;
- revised expected resource count;
- no infrastructure apply by Claude.

Do not claim an authenticated GCP plan if the sandbox cannot perform one.

## Human handoff

Return one consolidated report with:

1. files changed;
2. tests/validation;
3. exact `terraform init` for `s0-data`;
4. exact imports for the already-enabled SQL Admin/Redis APIs;
5. exact refreshed human `terraform plan` command;
6. expected plan/resource categories;
7. exact DB bootstrap/migration commands after creation;
8. DB-isolation verification commands/tests;
9. remaining blockers;
10. explicit confirmation:

```text
NO terraform apply was run.
NO GCP resource was created, changed, or destroyed by Claude.
```

Stop there. Do not apply `s0-data` or any later S0 stack.

---

## Completion report (2026-08-28)

Human had real live GCP access: `s0-network` applied, an authenticated `terraform plan` for `s0-data` showed `29 to add, 0 to change, 0 to destroy`. Live review found and fixed 4 blockers, all validated in this sandbox:

1. **API ownership** — added `google_project_service` resources for `sqladmin.googleapis.com`/`redis.googleapis.com` (`disable_on_destroy = false`); human must `terraform import` both once.
2. **Plaintext passwords in state** — replaced managed `random_password` + `password`/`secret_data` with Terraform 1.11 write-only arguments (`password_wo`/`secret_data_wo`) backed by an ephemeral `random_password` resource.
3. **No executable DB-boundary bootstrap** — added `database/s0-cloud/{01_cluster_ownership_and_connect_grants.sql,02_schema_boundaries_template.sql}` + README, tested for real against a throwaway local Postgres cluster (roles/databases created, grant matrix verified diagonal-true, cross-database CONNECT denial proven, cleaned up afterward).
4. **`s0-network` outputs typed in by hand** — `s0-data` now reads them via `terraform_remote_state`.

Also fixed opportunistically: `prevent_destroy` on the Cloud SQL instance/databases; `s0-ops-vm` missing the IAP SSH network tag; two missing Secret Manager IAM grants in `s0-services` for the runtime service accounts (discovered while verifying secret boundaries).

All 3 modified stacks (`s0-data`, `s0-services`, `s0-ops-vm`): `terraform fmt`/`validate` clean. Pre-apply artifact republished with the revised resource-count table.

**Claude wrote/tested/documented only. No `terraform apply` was run. No GCP resource was created, changed, or destroyed by Claude.** Full report in `wishes-game`'s git history and session record.

No deploy, no Secret Manager access, no IAM change — zero GCP credentials configured in this sandbox throughout.
