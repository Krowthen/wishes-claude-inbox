# Task: Build Agent Control Platform Foundation

Created: 2026-09-04
Priority: High
Mode: implementation-with-approval-gates
Assigned Agent: claude-google
Execution Environment: Google Cloud Claude Code Operations VM
Allow Edit: true

Depends on:
- `pending/bootstrap-claude-google-runtime-identity.md`
- `pending/agent-control-step-02-security-domain-reconciliation-claude-google.md`
- Human approval of Step 03 dedicated-domain placement where required
- Step 04 threat-model/security architecture review

References:
- `pending/reference-agent-control-platform-revised-approved-design.md`
- `pending/agent-control-platform-master-deployment-plan.md`

## Objective

Build the durable foundation of the independent multi-user/multi-project `agent-control` domain. Do not continue the superseded `wishes_ops` design.

The foundation must support organizations/users/workspaces/projects/project spaces, securely linked/named agents, project AI teams/PM assignment, workflows/tasks/claims, Design Rooms, live-agent session metadata, approvals/audit and credential capability metadata without storing external credential values.

## Phase 1 — Reconcile Step 02-04 outputs

1. Confirm `claude-google` runtime identity.
2. Review Step 02 completion report and any Human Step 03 placement decision.
3. Review threat model/security architecture from Step 04.
4. Inspect any implementation already started and merge/rework rather than blindly overwriting.
5. Stop and report if any unresolved security BLOCK remains.

## Phase 2 — Independent repository/domain structure

Use the approved dedicated repository/domain placement. If the dedicated repository cannot yet be provisioned, use only the explicitly approved isolated bootstrap location and include an extraction plan.

Logical modules may include:
```text
agent-control/
  gateway/
  workflow-engine/
  runtime-bridge/
  mcp/
  a2a/
  integrations/
  database/
  terraform/
  docs/
```

## Phase 3 — Dedicated `agent_control` database model

Do not create `wishes_ops`.

Implement migrations/schema for at least:
```text
organization
user_identity
organization_member
workspace
workspace_member
project
project_member
project_space
project_repository
project_environment
project_policy
project_control_document
project_control_document_version
agent_provider
agent_connection
agent_profile
agent_instance
agent_role_definition
agent_capability
agent_access_claim
project_agent_assignment
workflow_template
workflow_step
workflow_transition
workflow_run
task
task_dependency
task_assignment
task_claim
task_update
feedback
agent_conversation
agent_conversation_participant
agent_chat_message
agent_proposal
agent_decision
agent_signoff
agent_live_session
agent_subscription
agent_event
agent_handoff
agent_event_cursor
agent_execution
agent_checkpoint
agent_artifact
agent_approval
integration_provider
integration_connection
project_integration
external_work_item
activity_event
outbox_event
idempotency_record
```

Requirements:
- UUID/durable IDs per repository convention;
- typed organization/workspace/project ownership columns;
- deny cross-tenant access by design;
- use RLS where practical in addition to service authorization;
- no cross-database FK to Wishes/project databases;
- provider-specific metadata may use JSON, primary authz boundaries may not;
- design messages are distinct from Decisions/Tasks;
- Human approvals are explicit durable records;
- claims/event processing are concurrency/idempotency safe.

## Phase 4 — Credential-safe agent model

Implement metadata only:
```text
credential_location_class:
  none
  local_user
  local_agent
  human_interactive
  external_provider
  secret_manager_service_integration
```

`agent_access_claim` may record resource/scope/access level/availability/last verification/Human-intervention requirement.

Never create columns/API payloads intended to store raw GitHub tokens, Google ADC material, Claude/OpenAI credentials, passwords, private keys or refresh tokens.

When a server connector requires a secret, model an opaque secret reference only.

## Phase 5 — Human/agent principal separation

Build distinct principal concepts for:
- Human authenticated users;
- enrolled agent runtime/device instances;
- server service identities.

Agent runtime enrollment uses public/device identity + short-lived platform credentials; private key/provider credentials remain local.

## Phase 6 — Gateway internal domain layer

Implement/test internal services for:
- organizations/workspaces/projects/membership;
- project spaces and policies;
- project agent attachment and PM assignment;
- agent profiles/capabilities/access claims;
- workflows;
- task create/read/assign/claim/state/dependencies;
- checkpoints/artifacts/feedback;
- approvals;
- Project Control Document generation state;
- Design Room/proposal/decision/signoff state;
- live session/subscription/handoff metadata;
- audit/outbox/idempotency.

MCP/A2A/live-transport protocol exposure remains in dependent tasks.

## Phase 7 — Core security/state rules

Test at least:
1. User A cannot read/write User B personal workspace without membership/share.
2. Project A context cannot leak into Project B.
3. Agent attached to one project cannot claim another project's task without permission.
4. Personal workflow cannot weaken project policy.
5. PM AI cannot grant itself external access or bypass Human approval.
6. Agent access claims contain no credential values.
7. `requires_human` blocks execution when interactive access is unavailable.
8. Another agent cannot satisfy missing access by supplying credentials.
9. Design Room message -> Decision -> Task requires explicit promotion.
10. Human approval cannot be forged by AI message.
11. Durable state reconstructs without Redis/live transport.

## Approval Gates

### Database/live infrastructure gate
Before any live DB/IAM/cost-bearing apply, provide Human:
- exact schema/migration list;
- tenant/RLS/authz design;
- DB roles/grants;
- dedicated-domain placement;
- cost plan;
- rollback;
- confirmation of credential non-storage.

No live apply without required approval.

## Non-goals

Do not:
- deploy live MCP/A2A/live bridge;
- store or move external credentials;
- expose arbitrary shell execution;
- reuse Wishes application DB by default;
- retire Claude inbox;
- expand production authority.

## Completion Report

Report:
- Step 02-04 inputs used;
- repository/domain placement;
- schema and security design;
- credential metadata model;
- gateway modules;
- RLS/authz tests;
- unit/integration test results;
- commits/SHAs;
- Human gate status;
- readiness for MCP/A2A/live communication task.