# Step 02 — Agent Control Security / Domain Reconciliation

Created: 2026-09-04
Priority: Critical
Mode: inspection-and-design-with-approval-gates
Assigned Agent: claude-google
Execution Environment: Google Operations VM
Allow Edit: documentation / planning / non-live implementation only

References:
- `pending/reference-agent-control-platform-revised-approved-design.md`
- `pending/agent-control-platform-master-deployment-plan.md`

Depends on:
- `pending/bootstrap-claude-google-runtime-identity.md`

## Objective

Before any agent-control database migration or cloud deployment continues, reconcile all existing work against the newly approved independent multi-user/multi-project `agent-control` platform architecture and its credential/security rules.

This step is deliberately first because older tasks assumed a Wishes-specific `wishes_ops` database and single-project topology.

## 1. Confirm runtime and current work

1. Confirm identity is `claude-google` and the runtime is the Google Operations VM.
2. Review all applicable repository `CLAUDE.md` / `WORKFLOW.md` instructions.
3. Inspect current branches/worktrees/status for any agent-control implementation already started.
4. Search current inbox/repositories for:
   - `wishes_ops`;
   - `wishes-agent-gateway`;
   - agent-control schemas/migrations;
   - MCP/A2A implementation;
   - agent identity/workspace/project models;
   - live event/bridge work.
5. Do not delete or overwrite newer work. Report conflicts/deltas.

## 2. Live infrastructure inventory

From authorized environment, read-only inventory where available:
- current GCP project(s) used by Operations VM;
- Cloud SQL instances/databases;
- Redis/Memorystore/Pub/Sub resources relevant to control-plane use;
- Cloud Run services;
- Secret Manager containers relevant to existing agent/Claude services;
- IAM identities/service accounts;
- network/private access posture;
- current operations VM authentication/egress posture.

Do not create/change resources.

## 3. Security reconciliation

Evaluate current/planned work against these non-negotiable rules:

- control plane is a separate bounded domain;
- dedicated `agent_control` DB boundary; no `wishes_ops` continuation;
- no external provider credential values in normal DB/API/task/control-doc/logs;
- Human auth and agent runtime auth are distinct principal types;
- agent profiles store capability/access metadata only;
- local/user/agent-held credentials remain local;
- Human-interactive access is represented as `requires_human` when unavailable;
- server-side integration secrets, if ever required, are Secret Manager references only in DB;
- deny-by-default organization/workspace/project authorization;
- no Redis/PostgreSQL public exposure;
- no generic shell executor;
- live-agent events are push based and recoverable/offline replayable;
- no agent credential sharing as a workaround for missing access.

Produce a BLOCK list for any existing implementation violating these requirements.

## 4. Domain placement recommendation

Prepare Step 03 recommendation, without applying:

- dedicated repository vs bootstrap extraction plan;
- dedicated GCP project/management boundary name/placement;
- dedicated Cloud SQL instance/database sizing and cost implications;
- dedicated event transport strategy and cost;
- Agent Gateway/API naming;
- runtime bridge placement;
- Secret Manager/IAM boundaries;
- DNS/ingress/auth approach;
- whether any existing Wishes resources should be reused (default assumption: avoid reuse unless compelling and secure).

## 5. Data-model reconciliation

Compare any existing schema/code to required model:

```text
organization/user/workspace/project/project_space
agent_provider/connection/profile/instance/capability/access_claim
project_agent_assignment
workflow template/step/transition/run
task/dependency/assignment/claim/update/feedback
Design Room/proposal/decision/signoff
live_session/subscription/event/handoff/cursor
execution/checkpoint/artifact/approval
integration provider/connection/project integration/external work item
activity/outbox/idempotency
```

Do not create live migrations in this step.

## 6. Credential-specific review

Document, without exposing any secret values:
- what credentials currently exist on the Operations VM by category/provider;
- whether they are local files, OS keychain/helper, ADC/session, provider-native login, Secret Manager or other mechanism;
- which planned agent connections would need Human interactive login;
- which server-side adapters, if any, truly require a platform-held secret;
- how each can be revoked/rotated;
- what must never be centralized.

Do not print tokens, passwords, cookie values, private keys or refresh material into the report.

## 7. Output / Gate

Produce a completion report with:
- current implementation state;
- current live-state observations;
- deltas from revised architecture;
- security BLOCKs;
- recommended separate-domain placement;
- cost-bearing components requiring Human approval;
- exact old tasks/files needing amendment;
- whether it is safe to proceed to Step 03.

Hard stop: no Cloud SQL migration, new GCP project/resource creation, IAM mutation, Secret Manager write, credential movement or deployment apply in Step 02.