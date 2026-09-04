# Task: Prepare and Deploy Wishes Agent Control Plane to Google Cloud

Created: 2026-09-04
Priority: High
Mode: implementation-with-human-approval-gates
Assigned Agent: claude-google
Execution Environment: Google Cloud Claude Code Operations VM
Allow Edit: true

Depends on:
- `pending/bootstrap-claude-google-runtime-identity.md`
- `pending/build-agent-control-plane-foundation-claude-google.md`
- `pending/build-agent-control-plane-mcp-a2a-claude-google.md`

Reference design:
- `pending/reference-wishes-multi-agent-control-plane-and-design-room-final-design.md`

## Objective

Prepare the approved Agent Control Plane for deployment to the Wishes S0 Google Cloud environment, present a complete approval package, deploy only after the required Human approval, and connect the `claude-google` runtime to the deployed control plane.

Canonical project/region from current S0 design:

```text
GCP project: wishes-506905
region: us-central1
```

Verify these against the current live/canon state before apply.

## Phase 1 — Reconcile completed implementation

1. Confirm runtime identity `claude-google`.
2. Review completion reports/commits from the foundation and MCP/A2A tasks.
3. Run all tests again from the deploy candidate commit.
4. Confirm no unresolved `BLOCK`/security issue remains.
5. Confirm current S0 Terraform state ownership and avoid cross-state ownership conflicts.

## Phase 2 — Terraform/deployment design

Prepare the minimum required infrastructure for the deployed control plane, reusing existing resources when appropriate rather than duplicating them.

Expected areas may include:

```text
Cloud Run: wishes-agent-gateway
Cloud SQL: wishes_ops database + roles on the approved S0 instance
Redis/Memorystore: reuse approved existing S0 Redis if appropriate
Secret Manager: runtime secrets only
IAM: dedicated least-privilege runtime identity
Logging/Monitoring: structured service/audit logs
```

Do not assume all listed resources already exist. Inventory first.

The Agent Gateway runtime identity must not receive broad Owner/Editor/production-deployer privileges.

No unauthenticated write path is allowed.

## Phase 3 — Human approval package

Before any live apply that creates/modifies cost-bearing, IAM-sensitive, or database resources, provide the Human with:

- live inventory summary;
- exact Terraform plan/add/change/destroy counts;
- expected monthly incremental cost and important standing cost lines;
- service account/IAM grants;
- database/role changes;
- secrets required and how they are provisioned without committing credentials;
- ingress/authentication model;
- rollback procedure;
- validation plan;
- confirmation that normal production deployment authority remains unchanged.

Hard stop on any unexpected destroy/replacement.

## Phase 4 — Approved apply

Only after Human approval required by Wishes deployment policy:

1. apply the approved infrastructure/database changes;
2. deploy `wishes-agent-gateway` to the approved `us-central1` runtime;
3. verify TLS/authentication boundary;
4. verify no unauthenticated write access;
5. verify runtime identity permissions;
6. verify database connectivity only to intended operational data;
7. verify Redis/event path if enabled;
8. verify logging/audit records.

## Phase 5 — Connect `claude-google`

Configure Claude Code on the Operations VM to use the deployed `wishes-control` MCP endpoint and any approved A2A/bridge configuration required for its reviewer/agent role.

Requirements:

- identity resolves as `claude-google`;
- fresh session can call `get_project_state`;
- assigned tasks can be listed/read;
- task assigned to `claude-local` cannot be claimed by `claude-google`;
- checkpoint/artifact/result APIs work;
- Design Room baseline/final review interactions work;
- no permission bypass mode is introduced;
- credentials are stored using approved secure mechanisms, not repository files or shell history.

## Phase 6 — Google runtime acceptance test

From a fresh Claude Code session on the Operations VM, prove:

```text
1. identify as claude-google
2. retrieve project state
3. retrieve a test task assigned to claude-google
4. claim it
5. post progress/checkpoint
6. register a harmless test artifact/reference
7. complete it
8. read a Design Room
9. post an implementation/reality-review message
10. leave claude-local task untouched
```

Clean up synthetic test data according to the service's test-data policy without deleting audit evidence required for acceptance.

## Non-goals

Do not in this task:

- configure the local Windows Claude runtime;
- store OpenAI/Claude Web credentials without Human-approved provisioning;
- retire `wishes-claude-inbox`;
- expand Claude's protected production apply authority;
- deploy unrelated S0 resources;
- bypass required Human approvals.

## Required completion report

Report:

- deploy candidate commits;
- live inventory;
- Terraform plan/apply status;
- Human approval gate result;
- resources created/changed;
- cost delta;
- IAM/secret/database boundaries;
- endpoint/authentication details without secrets;
- `claude-google` configuration files changed;
- acceptance-test results;
- branch/commit SHA(s);
- whether the system is ready for `integrate-agent-control-plane-claude-local.md`.
