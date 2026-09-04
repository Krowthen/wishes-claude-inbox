# Task: Build Wishes Agent MCP, A2A, Redis Coordination, and Design Room Layer

Created: 2026-09-04
Priority: High
Mode: implementation-with-approval-gates
Assigned Agent: claude-google
Execution Environment: Google Cloud Claude Code Operations VM
Allow Edit: true

Depends on:
- `pending/bootstrap-claude-google-runtime-identity.md`
- `pending/build-agent-control-plane-foundation-claude-google.md`

Reference design:
- `pending/reference-wishes-multi-agent-control-plane-and-design-room-final-design.md`

## Objective

Extend the durable Agent Gateway foundation with:

1. transient Redis/event coordination;
2. the `wishes-control` MCP interface for AI -> Wishes control-plane access;
3. the A2A/Design Room interface for structured AI -> AI design collaboration;
4. role-specific authorization and sign-off semantics;
5. safe conversation limits and durable recovery.

This task must reuse the same internal service/domain layer and `wishes_ops` records created by the foundation task. Do not build a second parallel task or conversation system.

## Phase 1 — Reconcile foundation state

Before editing:

1. confirm `claude-google` runtime identity;
2. review the completion report/commits from the foundation task;
3. verify schema, state-machine, and service-layer tests pass;
4. identify any unresolved Human approval gate and do not bypass it;
5. confirm actual Redis/Memorystore deployment state before assuming a live Redis endpoint exists.

## Phase 2 — Redis/outbox coordination

Implement transient event coordination using the repository's existing Redis/event conventions where they exist.

Logical event categories must cover at least:

```text
agent task changed
agent assignment changed
agent checkpoint created
agent approval requested/resolved
Design Room message created
Design Room waiting-agent state
Design Room resolved
proposal promoted
decision created
```

Suggested logical streams/channels:

```text
wishes.agent.tasks.chatgpt
wishes.agent.tasks.claude-google
wishes.agent.tasks.claude-local
wishes.agent.events
wishes.agent.messages
wishes.agent.approvals
wishes.agent.chat
wishes.agent.chat.turn
wishes.agent.chat.waiting
wishes.agent.chat.completed
```

Requirements:

- PostgreSQL remains authoritative;
- Redis loss/flush must not lose tasks, approvals, conversations, proposals, or decisions;
- use outbox/idempotency/reconciliation appropriate to current Wishes architecture;
- every event must reference durable IDs;
- consumer retry must not duplicate durable effects;
- do not introduce an always-on polling worker if the existing S0 transport/cost design prohibits it.

## Phase 3 — MCP server: `wishes-control`

Expose the control-plane functions over MCP using the existing domain/service layer.

Initial tool surface should include, where supported by authorization:

```text
get_project_state
get_agent_status
list_tasks
get_task
create_task
assign_task
claim_task
post_message
report_progress
create_checkpoint
register_artifact
complete_task
request_review
request_approval
list_approvals
resolve_approval
list_design_rooms
get_design_room
post_design_message
create_proposal
promote_proposal
```

Requirements:

- tool authorization is role-aware;
- no MCP tool is a generic arbitrary shell executor;
- no MCP tool grants cloud/database authority beyond the caller's role;
- sensitive credentials are never returned through normal task/project-state tools;
- project-state responses should be compact enough for fresh AI sessions while retaining references to retrieve deeper detail;
- tools should be idempotent where practical and return stable durable IDs.

## Phase 4 — A2A / Design Room adapter

Implement the Design Room collaboration adapter using the approved A2A direction while mapping all durable state into the existing `wishes_ops` conversation/message/proposal/decision model.

Do not create a second independent A2A database.

Canonical Design Room participants:

```text
chatgpt-director  -> lead architect/director
claude-coop       -> co-designer/challenger
claude-google     -> implementation/reality reviewer
claude-local      -> optional Unity/Windows/client specialist
openai-director   -> optional API runtime representing the director role when unattended turns are required
```

Support:

- room creation;
- participant membership/roles;
- threaded or reply-linked messages;
- message types: proposal, response, challenge, question, evidence, alternative, risk, agreement, disagreement, summary, decision-request, human-intervention;
- waiting-agent state;
- synthesis/resolved state;
- proposal attachment;
- sign-off records/status;
- Human intervention;
- durable recovery after agent disconnect.

## Phase 5 — Canonical Design Room workflow enforcement

Represent and test the approved default workflow:

```text
ChatGPT initial scope/design
 -> Claude Google baseline reality review
 -> ChatGPT <-> Claude Coop iterative design comparison
 -> Claude Google final implementation review
 -> sign-off stage
 -> Human approval if required
 -> Decision
 -> Tasks
```

The system should not require semantic unanimity. It should track:

```text
APPROVE
APPROVE_WITH_NOTES
BLOCK
```

Specific sign-offs:

```text
DESIGN_SIGNOFF         -> claude-coop
IMPLEMENTATION_SIGNOFF -> claude-google
DIRECTOR_SIGNOFF       -> chatgpt-director/openai-director acting in director runtime role
CLIENT_SIGNOFF         -> optional claude-local specialist review
HUMAN_APPROVAL         -> when authority/risk requires it
```

A room may be promoted when there are zero unresolved blocking objections and the configured authority requirements are met.

## Phase 6 — Conversation limits and safety

Implement configurable room limits, with initial defaults equivalent to:

```text
max_rounds: 6
max_consecutive_turns_same_agent: 2
```

At the limit, the facilitator must enter a terminal/intervention state requiring one of:

- synthesize;
- request Human input;
- explicitly extend the room.

Prevent autonomous infinite agent loops.

Design-room messages must never directly grant implementation or production authority.

## Phase 7 — Runtime-facing project recovery

Implement a compact `get_project_state` response capable of reconstructing at least:

- active tasks by agent;
- blocked tasks/dependencies;
- pending approvals;
- active Design Rooms;
- latest checkpoints;
- recent decisions;
- branch/commit/artifact references;
- offline/last-seen agent status where available.

A fresh session should be able to retrieve deeper records by ID.

## Phase 8 — Tests

Test at least:

- MCP role authorization;
- cross-agent assignment protection;
- no arbitrary execution capability;
- Redis interruption and durable reconstruction;
- idempotent event handling;
- Design Room message persistence/order;
- waiting/offline agent behavior;
- max-round enforcement;
- `BLOCK` preventing promotion;
- `APPROVE_WITH_NOTES` not acting as a blocker;
- explicit Proposal -> Decision -> Task boundary;
- Human approval cannot be forged by a normal agent message;
- fresh-session project-state recovery.

## Approval gates

Do not deploy new cost-bearing GCP resources in this task without the applicable Human approval.

If a live Memorystore/Cloud Run resource is required and not already deployed, prepare the Terraform/code/plan and stop at the appropriate approval gate for the dependent deployment task.

## Non-goals

Do not in this task:

- connect real OpenAI or Claude Web secrets if Human credential work is required;
- retire the Claude inbox;
- modify Claude Local machine configuration;
- expand production deployment authority;
- make design discussions directly executable;
- canonize before acceptance.

## Required completion report

Report:

- foundation commit(s) used;
- files/modules changed;
- Redis/outbox strategy implemented;
- MCP tools implemented and authorization matrix;
- A2A/Design Room interfaces implemented;
- sign-off/blocking semantics;
- conversation-limit behavior;
- tests and results;
- branch/commit SHA(s);
- deployment prerequisites still requiring Human action;
- exact readiness for `deploy-agent-control-plane-gcp-claude-google.md`.
