# Development Agent Control Platform — Master Deployment Plan

Created: 2026-09-04
Status: Approved for staged implementation
Mode: master-deployment-plan
Owner: Human
Director: chatgpt-director
Primary Implementer: claude-google
Local Runtime Integrator: claude-local
Design Challenger: claude-coop
Reference: `pending/reference-agent-control-platform-revised-approved-design.md`

## Objective

Deploy the independent `agent-control` domain as a secure multi-user, multi-project AI development-control platform with:

- private/shared workspaces;
- projects/project spaces;
- user-linked/named agents;
- Project Manager AI assignment;
- workflows/tasks/claims;
- Project Control Documents;
- Design Rooms;
- near-real-time AI-to-AI live sessions;
- durable recovery/audit;
- secure Human and agent identity;
- no normal storage or sharing of external provider credentials;
- optional Jira/Asana adapters.

## Global Hard Rules

1. No external provider credential values in ordinary DB tables, task payloads, control documents, Git or logs.
2. Human authentication and agent-runtime authentication are separate principal types.
3. `agent_control` is independent from Wishes databases and application runtime.
4. No live cost-bearing/IAM/database apply without Human approval package.
5. No public PostgreSQL/Redis endpoints.
6. No generic arbitrary-shell control-plane endpoint.
7. Every read/write is organization/workspace/project authorized.
8. Project/workspace policy can only become stricter at lower scopes, never weaker.
9. Design Room content is non-authoritative until explicit Decision -> Task promotion.
10. Live agent loops require retry/iteration/time/cost limits and escalation.
11. Credential capability declarations are metadata only; underlying credentials stay local/user-held unless a specific server-side connector requires a secret reference.
12. Never solve missing access by transferring another user/agent's credentials.

# Master Deployment Steps

## Stage 0 — Freeze / Reconcile

### Step 01 — Runtime naming bootstrap
Owners: `claude-google`, `claude-local`
Status: already queued

Confirm canonical runtime identities in each environment and interim inbox routing rules.

### Step 02 — Repository/live-state/security reconciliation
Owner: `claude-google`
Status: READY — execute next

Review current repos, outstanding agent-control work, GCP state, security requirements and any work already started. Produce a delta against the revised reference before building migrations.

### Step 03 — Dedicated domain placement decision
Owners: `chatgpt-director` + `claude-google`
Human gate: YES

Decide and approve:
- dedicated repository strategy;
- dedicated GCP project/management boundary vs explicitly temporary isolated placement;
- dedicated Cloud SQL instance/database placement;
- event transport placement;
- DNS/service naming;
- cost implications.

No infrastructure apply in this step.

## Stage 1 — Identity / Security Foundation

### Step 04 — Threat model and security architecture
Owner: `claude-google`
Review: `chatgpt-director`

Threat model:
- cross-tenant/IDOR;
- account takeover/session theft;
- agent impersonation;
- malicious task/event payloads;
- prompt-injection through agent messages/artifacts;
- secret leakage/logging;
- privilege escalation;
- insecure integration adapters;
- replay/duplicate events;
- infinite agent loops;
- unauthorized Human-approval forgery.

### Step 05 — Human identity/authentication design
Owner: `claude-google`
Human gate before provider setup

Implement external managed OIDC/OAuth identity boundary, session handling, MFA/step-up policy, CSRF/rate limits and admin roles. Avoid application password storage for initial release.

### Step 06 — Agent runtime enrollment design
Owner: `claude-google`

Implement local keypair/device enrollment, Human approval/linking, public identity storage, short-lived platform credentials, revocation and rotation.

### Step 07 — Credential capability/access-claim model
Owner: `claude-google`

Implement `agent_capability`, `agent_access_claim` and credential-location metadata. Prove no secret value enters profile/task/project-control APIs.

## Stage 2 — Multi-Tenant Domain / Database

### Step 08 — Dedicated PostgreSQL schema/migrations
Owner: `claude-google`
Human gate before live DB apply

Build `agent_control` database model for organizations, users, workspaces, projects, agents, workflows, tasks, design, live communication, approvals, integrations, audit/outbox/idempotency.

### Step 09 — Tenant authorization/RLS and service policy
Owner: `claude-google`

Deny-by-default policy, org/workspace/project membership checks, project-space visibility and adversarial cross-tenant tests.

### Step 10 — Project/PM/Control Document domain
Owner: `claude-google`

Implement project ownership/membership, PM AI assignment, project agents, project policy and generated/shared Project Control Document.

### Step 11 — Workflow/task routing engine
Owner: `claude-google`

Implement direct, role-routed, claimable and human-only assignment; eligibility based on role/capability/environment/dependencies/policy/access availability.

## Stage 3 — Gateway and Durable Operations

### Step 12 — Agent Gateway application/API
Owner: `claude-google`

CRUD/lifecycle for agents/projects/workspaces/tasks/checkpoints/artifacts/feedback/approvals/decisions/audit. No arbitrary shell execution.

### Step 13 — Outbox/idempotency/event layer
Owner: `claude-google`

Persist durable events then publish transiently. Replay/reconcile safely after transport failure.

### Step 14 — MCP platform interface
Owner: `claude-google`

Expose authorized project/task/agent/design/control operations without exposing credentials or external-resource authority.

## Stage 4 — Live Agent Communication

### Step 15 — Runtime Bridge protocol
Owner: `claude-google`
Local validation: `claude-local`

Build outbound TLS authenticated bridge for online agent instances, heartbeat, subscriptions, durable event cursor and reconnect/replay.

### Step 16 — Live Agent Session / push delivery
Owner: `claude-google`

Implement near-real-time push events so coding/testing/review agents react automatically without manual polling.

### Step 17 — Handoff and autonomous loop engine
Owner: `claude-google`

Implement structured `agent_handoff`, workflow event transitions, loop limits, escalation and PM visibility.

### Step 18 — Coding-agent <-> testing-agent acceptance loop
Owners: `claude-google` + test agent/runtime

Prove code.ready_for_test -> automatic validation -> test.failed -> automatic coder wakeup -> fix -> retest -> test.passed, with no Human "check again" prompt.

## Stage 5 — Design Rooms / A2A

### Step 19 — Design Room persistence and A2A adapter
Owner: `claude-google`

Implement durable rooms/messages/proposals/signoffs, A2A adapter where supported, Human intervention and offline recovery.

### Step 20 — Canonical Design workflow
Owners: `chatgpt-director`, `claude-coop`, `claude-google`

Prove ChatGPT initial design -> Google reality review -> ChatGPT/Coop debate -> Google final review -> signoffs -> Decision -> Tasks.

## Stage 6 — User/Project Agent Experience

### Step 21 — Agent linking/naming UI/API
Owner: `claude-google`

Users link supported providers/runtimes, name profiles and see capability/access posture without viewing secrets.

### Step 22 — Workflow builder
Owner: `claude-google`

Create/edit reusable workflow graph, role targets and transition/event rules.

### Step 23 — Project membership/agent team experience
Owner: `claude-google`

Create/join project, attach permitted agents, owner assigns PM, PM displays team and task routing.

### Step 24 — Feedback/update surfaces
Owner: `claude-google`

Human/AI progress, test/review/user feedback, PM summaries and audit timeline.

## Stage 7 — External Integrations

### Step 25 — Integration framework
Owner: `claude-google`

Provider-neutral adapter framework and project-scoped secret-reference handling.

### Step 26 — Jira connector (optional)
Owner: later assigned

### Step 27 — Asana connector (optional)
Owner: later assigned

## Stage 8 — Cloud Deployment

### Step 28 — Terraform / deployment plan
Owner: `claude-google`
Human gate: YES

Prepare dedicated platform infrastructure plan, IAM, database, event transport, Secret Manager refs, network/private access, logging and cost estimate.

### Step 29 — Security review before apply
Owners: `chatgpt-director` + `claude-google`
Human gate: YES

Review threat model, tenant tests, IAM, secret handling, ingress/auth, DB/network isolation, rollback.

### Step 30 — Approved cloud apply
Owner: `claude-google`
Human approval required

### Step 31 — Connect `claude-google`
Owner: `claude-google`

### Step 32 — Connect `claude-local`
Owner: `claude-local`

### Step 33 — Connect ChatGPT / OpenAI Director path
Owners: Human + `chatgpt-director`

### Step 34 — Connect Claude Coop path
Owners: Human + implementation adapter owner

## Stage 9 — Full Acceptance / Cutover

### Step 35 — Multi-user isolation tests
Prove two users cannot access each other's personal spaces without explicit sharing.

### Step 36 — Multi-project isolation tests
Prove agents/tasks/context do not leak across projects.

### Step 37 — Agent credential non-disclosure tests
Prove profiles/control docs/APIs/logs expose capability metadata only.

### Step 38 — Human-interactive access test
Expire/revoke an external login; agent must report `requires_human`, not request credentials from another agent.

### Step 39 — Offline/reconnect live-agent test
Queue events while runtime offline, replay in order at reconnect.

### Step 40 — Full Project Manager workflow test
Project Owner assigns PM -> members attach agents -> PM routes/agents claim -> feedback -> control document updates.

### Step 41 — Full Design Room test

### Step 42 — Full live coding/testing loop test

### Step 43 — Audit / incident / revocation test

### Step 44 — Human cutover approval

### Step 45 — Demote Claude Inbox to archive/fallback

### Step 46 — Reconcile/canonize Wishes documentation as a consumer project

# Immediate Iteration Order

Execute now in this order:

1. Step 01 identity bootstrap tasks already queued.
2. Step 02 new security/domain reconciliation task.
3. Step 03 produce dedicated-domain placement/cost/ownership recommendation and stop for Human approval.
4. Step 04 security threat model.
5. Only after Steps 02-04 are reviewed may database migrations from older `wishes_ops` tasks proceed; they must be rewritten to `agent_control`.

# Current Superseded Assumptions

The following older implementation assumptions are no longer valid:

- `wishes_ops` as the durable control database;
- control-plane tables inside Wishes S0 Cloud SQL by default;
- `wishes-agent-gateway` as a Wishes application service name/domain;
- single-user/single-project identity tables;
- agent communication requiring explicit polling/check-update prompts.

They must be reconciled before implementation continues.

# Required Step Completion Format

Each step reports:
- step number/status;
- owner/runtime identity;
- files/resources reviewed;
- changes made;
- security findings;
- tests/validation;
- commit SHA(s);
- Human gate reached/not reached;
- next step unblocked;
- unresolved BLOCKs.