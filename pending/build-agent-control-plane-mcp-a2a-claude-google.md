# Task: Build Agent Control MCP, A2A, Live Sessions, and Event Layer

Created: 2026-09-04
Priority: High
Mode: implementation-with-approval-gates
Assigned Agent: claude-google
Execution Environment: Google Cloud Claude Code Operations VM
Allow Edit: true

Depends on:
- `pending/build-agent-control-plane-foundation-claude-google.md`

References:
- `pending/reference-agent-control-platform-revised-approved-design.md`
- `pending/agent-control-platform-master-deployment-plan.md`

## Objective

Extend the independent `agent-control` foundation with:
1. durable outbox/idempotency + transient event dispatch;
2. MCP agent-to-platform interface;
3. A2A/Design Room adapter;
4. authenticated push-based Live Agent Sessions;
5. runtime subscriptions/handoffs/offline replay;
6. security boundaries that expose capabilities/access posture but never external credential values.

Do not build a second task/conversation database and do not expose project credentials through protocol payloads.

## Phase 1 — Foundation reconciliation

Confirm:
- `claude-google` identity;
- current foundation commit/tests;
- dedicated `agent_control` database/domain model;
- no unresolved security BLOCK;
- no live resource assumption without inventory/approval.

## Phase 2 — Durable outbox and transient event dispatch

Implement durable `outbox_event` + `idempotency_record` behavior and transient dispatch through the approved event transport.

Events cover:
```text
task.created/changed/assigned/claimable
checkpoint.created
approval.requested/resolved
feedback.created
agent.online/offline
access.requires_human
design.message/proposal/signoff/resolved
code.ready_for_test
test.failed/test.passed
handoff.created/accepted/completed
workflow.transition
```

Requirements:
- PostgreSQL is authoritative;
- transient transport loss cannot lose committed work;
- duplicate delivery cannot duplicate durable effects;
- every event is org/workspace/project scoped;
- event payloads contain no provider credentials;
- offline consumers can resume by durable cursor/checkpoint.

## Phase 3 — MCP interface

Expose authorized tools such as:
```text
get_project_state
get_agent_status
list_projects
list_tasks/get_task/create_task/assign_task/claim_task
post_update/post_feedback
create_checkpoint/register_artifact/complete_task
request_review/request_approval/list_approvals/resolve_approval
list_design_rooms/get_design_room/post_design_message
create_proposal/promote_proposal
get_agent_capabilities/get_agent_access_posture
request_human_intervention
```

Requirements:
- deny by default;
- tenant/project authorization on every tool;
- no generic arbitrary shell execution;
- no returned credential/token/private-key values;
- access posture returns metadata only (`available`, `requires_human`, `unavailable`, local/server-reference class);
- stable IDs/idempotency where practical.

## Phase 4 — A2A / Design Room

Implement durable Design Room collaboration using the same domain state.

Canonical roles remain configurable; Wishes default:
```text
chatgpt-director -> lead design/director
claude-coop      -> co-designer/challenger
claude-google    -> implementation/reality reviewer
claude-local     -> optional client/Windows specialist
openai-director  -> optional unattended API runtime
```

Support room messages, reply/thread references, proposals, BLOCK/APPROVE_WITH_NOTES/APPROVE, signoffs, Human intervention and offline recovery.

Design messages cannot directly authorize implementation.

## Phase 5 — Live Agent Runtime Bridge

Implement a secure runtime bridge protocol.

Required design:
```text
agent runtime bridge
  -> outbound TLS authenticated connection
  -> Agent Gateway
  -> authorized subscription/event delivery
```

Never expose Redis/PostgreSQL directly to runtime devices.

Runtime identity must bind to a specific enrolled `agent_instance` using the approved device/public-key enrollment model and revocable short-lived platform credentials.

Support:
- connect/disconnect/heartbeat;
- online/offline state;
- subscriptions;
- pushed events/messages;
- acknowledgements/cursors;
- reconnect/replay;
- revocation;
- rate limiting;
- per-project authorization.

Do not use one shared permanent agent API key.

## Phase 6 — Agent subscriptions and handoffs

Implement:
```text
agent_subscription
agent_handoff
agent_event_cursor
```

A handoff includes:
- from principal;
- target agent/role;
- project/task;
- requested action;
- artifact/branch/commit reference;
- return/escalation path;
- deadline/iteration metadata where relevant.

## Phase 7 — Autonomous loop controls

Support event-triggered workflow loops such as coding <-> testing without Human polling.

Example:
```text
coder -> code.ready_for_test -> tester
tester -> test.failed -> coder
coder -> code.ready_for_test -> tester
tester -> test.passed -> PM / next step
```

Every loop must support:
- max iterations;
- time limit;
- optional cost/token budget;
- repeated-failure escalation;
- Human/PM stop control;
- no cross-project routing.

## Phase 8 — Credential-safe runtime behavior

At execution time an agent may have an access claim such as `gcp:project-x:plan` but status `requires_human`.

The workflow must:
1. stop that step;
2. emit/request Human intervention;
3. resume after the owning runtime verifies access;
4. never request the credential value from another agent or persist it centrally.

For server-side connectors, only approved opaque Secret Manager refs are visible to the connector service identity; never to task/design APIs.

## Phase 9 — Fresh-session recovery

`get_project_state` must reconstruct:
- project/space;
- PM/team;
- active/blocked/claimable tasks;
- latest checkpoints;
- pending approvals/Human-intervention requests;
- active Design Rooms;
- online/offline agents;
- access availability metadata without secrets;
- recent decisions/artifact refs.

## Phase 10 — Tests

Test at minimum:
- cross-user and cross-project access denial;
- MCP authorization;
- secret/credential redaction;
- agent revocation;
- event idempotency;
- event replay after transient-transport loss;
- live push without polling;
- offline reconnect/replay;
- handoff routing;
- max-loop enforcement;
- requires-human credential flow;
- Design Room promotion boundaries;
- Human approval forgery resistance;
- fresh-session recovery.

## Approval Gates

No new live Cloud Run/Cloud SQL/Redis/PubSub/IAM/Secret Manager resource or external provider credential setup without the required Human approval.

## Non-goals

Do not:
- move user/agent provider credentials into the platform;
- retire the inbox;
- expand protected production authority;
- make design messages executable;
- create public DB/event endpoints.

## Completion Report

Report:
- foundation commit;
- event/outbox strategy;
- MCP authz matrix;
- A2A/Design Room implementation;
- runtime bridge/auth model;
- subscriptions/handoff/live-loop behavior;
- credential-safety tests;
- tenant security tests;
- commits/SHAs;
- deployment prerequisites/Human gates.