# Task: Prepare and Deploy Agent Control Platform to Google Cloud

Created: 2026-09-04
Priority: High
Mode: implementation-with-human-approval-gates
Assigned Agent: claude-google
Execution Environment: Google Cloud Claude Code Operations VM
Allow Edit: true

Depends on:
- `pending/agent-control-step-02-security-domain-reconciliation-claude-google.md`
- Human Step 03 placement approval
- Step 04 security/threat-model review
- `pending/build-agent-control-plane-foundation-claude-google.md`
- `pending/build-agent-control-plane-mcp-a2a-claude-google.md`

References:
- `pending/reference-agent-control-platform-revised-approved-design.md`
- `pending/agent-control-platform-master-deployment-plan.md`

## Objective

Prepare the independent `agent-control` platform for secure cloud deployment, present a complete Human approval package, deploy only after approval, and connect enrolled agent runtimes without centralizing their external provider credentials.

Do not assume `wishes-506905` or Wishes Cloud SQL is the deployment target. Use the approved Step 03 dedicated-domain placement.

## Phase 1 — Reconcile candidate

1. Confirm `claude-google` identity.
2. Review Step 02/03/04 outputs and Human approvals.
3. Review foundation/MCP/A2A/live-session commits.
4. Run all security/tenant/event tests.
5. Confirm no unresolved BLOCK.
6. Confirm exact state ownership and avoid cross-state conflicts.

## Phase 2 — Dedicated infrastructure design

Prepare approved minimum platform resources, expected to include where authorized:
```text
Dedicated/approved GCP management project boundary
Agent Gateway / API service
Dedicated Cloud SQL PostgreSQL instance
  database: agent_control
Dedicated private event transport / Redis if approved
Secret Manager for server-side integration secrets only
Dedicated least-privilege runtime service identities
Private networking/connectivity
Logging/Monitoring/Audit
DNS/ingress/auth boundary
```

Do not expose PostgreSQL/Redis publicly.

No normal task/agent API may retrieve Secret Manager values.

## Phase 3 — Human approval package

Before live apply provide:
- live inventory;
- exact Terraform plan add/change/destroy counts;
- dedicated domain/project placement;
- monthly incremental cost (Cloud SQL, Redis/event transport, Cloud Run, storage/network/logging, NAT if any);
- IAM/service accounts;
- DB roles/grants/RLS model;
- Human sign-in model;
- agent enrollment/token model;
- server-side integration secret references and why each is necessary;
- proof normal agent profiles store capability/access metadata only;
- ingress/network/private-service posture;
- threat-model/security test summary;
- rollback/rebuild/revocation procedures.

Hard stop on unexpected destroy/replacement or any credential-centralization not explicitly approved.

## Phase 4 — Approved apply

Only after Human approval:
1. create/apply approved dedicated infrastructure;
2. migrate/apply `agent_control` schema;
3. deploy Agent Gateway;
4. verify TLS and authenticated ingress;
5. verify no unauthenticated writes;
6. verify DB/event endpoints are private;
7. verify runtime identity least privilege;
8. verify cross-tenant access denial;
9. verify logs redact secret-like values;
10. verify audit trail.

## Phase 5 — Connect `claude-google`

Enroll the Google runtime as its own `agent_instance`.

Requirements:
- private runtime key/provider credentials remain on Google runtime;
- platform receives public enrollment identity and short-lived/revocable platform auth only;
- `claude-google` can retrieve allowed project state/tasks;
- it cannot claim another user's/private/project task without authorization;
- capability/access claims are visible as metadata, not secrets;
- `requires_human` access state can be raised when interactive sign-in is needed;
- live push subscriptions work without manual check/poll prompts.

## Phase 6 — Acceptance

From a fresh Google runtime prove:
```text
identify/enroll
retrieve allowed projects/project state
receive pushed test task/event
claim eligible task
post checkpoint/update/artifact reference
participate in Design Room review
receive handoff
handle requires_human test without credential transfer
complete test
leave unauthorized project/task untouched
revoke and re-enroll test identity according to policy
```

## Non-goals

Do not:
- configure another user's machine/provider credentials;
- store OpenAI/Claude/GitHub/GCP user credential values centrally;
- retire Claude inbox before full acceptance;
- expand production authority;
- deploy unrelated Wishes resources.

## Completion Report

Report:
- deployment candidate commits;
- approved project/domain placement;
- live inventory and plan/apply state;
- cost delta;
- IAM/DB/network/secret boundaries;
- security test results;
- endpoint/auth posture without secrets;
- runtime enrollment configuration changed;
- live push/replay tests;
- Human approval gate result;
- commits/SHAs;
- readiness for `claude-local` integration.