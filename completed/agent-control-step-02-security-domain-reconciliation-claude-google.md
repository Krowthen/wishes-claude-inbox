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

---

## Completion report (2026-09-04)

Full report: `wishes-game`'s `docs/claude/agent-control-step-02-security-
domain-reconciliation.md`, branch `claude-google/step-02-security-domain-
reconciliation`, PR https://github.com/Krowthen/wishes-game/pull/3
(not merged — left for Human review).

**Current implementation state:** none. Searched all three repos
(tracked files, all branches) for `wishes_ops`, `wishes-agent-gateway`,
agent-control schemas/migrations, MCP/A2A code, agent identity/
workspace/project models, live event/bridge work — zero hits outside
the inbox task/reference documents themselves. Genuine greenfield.

**Current live-state observations:** attempted read-only inventory of
Cloud SQL, Redis, Pub/Sub, Cloud Run, Secret Manager, IAM service
accounts, compute networks/instances, and project IAM policy via
`gcloud`, all as `wishes-s0-claude-ops-host@wishes-506905.iam.
gserviceaccount.com`. **Every call returned `PERMISSION_DENIED`**,
including reading the Terraform remote-state GCS bucket. The service
account has broad `cloud-platform` OAuth scope but an IAM role too
narrow for any of this. Only ground truth available from this runtime:
local `s0-bootstrap` Terraform state (2 resources: the resourcemanager
API enablement + the tfstate bucket itself) and the Human's own prior
documented `s0-data` pre-apply review in `docs/claude/todo.md`
(`s0-network` applied, `s0-data` plan showed 29-to-add). **A genuine
live-infrastructure inventory is not achievable from `claude-google`'s
current permissions** — this needs a Human decision: do the inventory
steps directly, or grant `claude-google` a deliberate, narrow, read-only
IAM role.

**Deltas from revised architecture:** none in code. One doc delta (see
"old tasks/files needing amendment" below).

**Security BLOCKs:** none — nothing exists yet to violate the
non-negotiable rules list.

**Recommended separate-domain placement (input to Step 03, not
applied):** the revised design's own §2/§21 already state a preference
— dedicated GCP project (distinct from `wishes-506905`) and dedicated
repository (not `wishes-game`), since "Wishes is the first registered
project, not the platform boundary." Carried forward as input; the
actual placement decision belongs to Step 03 + Human approval.

**Cost-bearing components requiring Human approval:** everything from
Step 03 onward that creates a new GCP project, Cloud SQL instance,
event transport, or Cloud Run service. Nothing cost-bearing created in
Step 02.

**Exact old tasks/files needing amendment:** `pending/canonize-agent-
control-plane-after-acceptance-claude-google.md`, lines 26 and 39 —
still literally instructs documenting a `wishes_ops` boundary as real,
unlike the rest of this batch (already updated to treat it as
superseded). Not amended in this step — out of scope for Step 02, far
downstream (gated on end-to-end acceptance), flagged for whoever next
revises that file.

**Safe to proceed to Step 03:** yes, with the live-infrastructure-
inventory caveat above carried forward as an open question rather than
resolved.

Awaiting the Human's go-ahead before starting Step 03
(`agent-control-step-03-domain-placement-claude-google.md`).