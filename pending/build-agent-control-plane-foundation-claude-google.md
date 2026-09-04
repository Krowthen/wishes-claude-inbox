# Task: Build Wishes Agent Control Plane Foundation

Created: 2026-09-04
Priority: High
Mode: implementation-with-approval-gates
Assigned Agent: claude-google
Execution Environment: Google Cloud Claude Code Operations VM
Allow Edit: true

Depends on:
- `pending/bootstrap-claude-google-runtime-identity.md`

Reference design:
- `pending/reference-wishes-multi-agent-control-plane-and-design-room-final-design.md`

## Objective

Begin implementation of the approved Wishes Multi-Agent Control Plane by grounding the design in the live repository/GCP state, creating the durable `wishes_ops` data model and role boundary, and building the initial Agent Gateway application foundation.

This task intentionally stops before live cost-bearing/destructive GCP apply unless an existing approved workflow explicitly authorizes the specific apply and the Human has approved the gate.

## Phase 1 — Verify current reality

Before editing:

1. Confirm runtime identity is `claude-google` and this is the Google Operations VM.
2. Review current repository instructions including all applicable `CLAUDE.md` and `WORKFLOW.md` files.
3. Review the current Wishes deployment/canon material relevant to:
   - Claude Code Operations VM;
   - Cloud SQL/data boundaries;
   - Redis/Memorystore;
   - Cloud Run;
   - IAM/service-account boundaries;
   - Terraform state/layout;
   - production apply authority;
   - Claude inbox workflow.
4. Inventory the actual implementation repository structure and identify where server services, database migrations, Terraform, and tests currently live.
5. From the authorized Google environment, inventory relevant live S0 resources where credentials permit, without changing them.
6. Confirm whether the planned S0 Cloud SQL instance/Redis resources are actually deployed yet. Do not assume planning documents equal live state.
7. Produce a short implementation-delta note in the task completion report identifying any material difference between the approved design and current repository/cloud state.

## Phase 2 — Branch and implementation structure

Create a focused implementation branch according to repository workflow. Preferred logical branch name if not conflicting with repository rules:

```text
feature/agent-control-plane
```

Do not implement directly on `main` unless the repository explicitly requires that workflow and report why.

Identify/establish the minimum implementation areas. The approved design suggests the following logical structure, but adapt to the actual repository rather than duplicating existing service frameworks:

```text
services/agent-gateway/
database/migrations/.../wishes_ops/
infrastructure/terraform/.../agent-control-plane/
scripts/agentctl/
```

Do not create duplicate application frameworks if an existing service shell is the correct home.

## Phase 3 — `wishes_ops` durable schema design and migrations

Implement the approved operational domain with a dedicated database boundary:

```text
wishes_ops
```

Target roles:

```text
wishes_ops_owner
wishes_ops_runtime
wishes_ops_auditor
```

The schema must support at least:

```text
agent_identity
agent_session
agent_task
agent_execution
agent_checkpoint
agent_artifact
agent_approval
agent_conversation
agent_conversation_participant
agent_chat_message
agent_proposal
agent_decision
```

Requirements:

- use UUIDs or the repository's established durable identifier convention;
- include timestamps/status fields and foreign-key constraints where valid within `wishes_ops`;
- avoid cross-database foreign keys to `wishes_core`, `wishes_assets`, or `wishes_auth`;
- repository/commit/artifact references are references/snapshots, not foreign keys into GitHub;
- task assignment must support canonical agent IDs;
- durable state must survive Redis loss;
- design-room messages/proposals/decisions must be distinguishable from authoritative implementation tasks;
- Human approvals must be explicit durable records;
- include idempotency/concurrency considerations for task claims and event processing;
- use JSON only where flexibility is justified; retain query-critical status/ownership fields as normal typed columns.

Canonical identities to seed/register at the logical layer:

```text
human-owner
chatgpt-director
claude-coop
claude-google
claude-local
openai-director  # optional/inactive until configured
```

Do not create credentials for `openai-director` merely by creating its logical identity.

## Phase 4 — Migration convention reconciliation

The existing S0 work previously identified multi-database migration conventions as a design gap. Before adding a new database migration path:

1. inspect what has changed since that gap was recorded;
2. use the current repository's established convention if one now exists;
3. if still unresolved, implement the smallest coherent convention that can support `wishes_ops` without breaking the existing databases;
4. document the convention and its ordering/runner behavior;
5. do not perform a live database migration until the required Human approval/data gate is satisfied.

## Phase 5 — Agent Gateway application foundation

Build the internal application/service layer for:

- agent identity lookup;
- task create/read/list;
- assign/claim with concurrency protection;
- status transitions;
- checkpoints;
- artifact references;
- approval request/read/resolve state;
- Design Room create/read/list;
- participant registration;
- durable chat messages;
- proposal persistence;
- decision persistence/promotion boundary.

At this stage, prioritize a clean internal service/API layer and tests. MCP/A2A protocol exposure is handled in the dependent protocol task.

## Phase 6 — Authority and state-machine rules

Implement and test at least these rules:

1. A task assigned to `claude-local` cannot be silently claimed by `claude-google`.
2. A task assigned to `claude-google` cannot be silently claimed by `claude-local`.
3. Reassignment requires authorized control-plane action.
4. Design Room messages do not automatically create executable work.
5. Proposal -> Decision is explicit.
6. Decision -> Task is explicit.
7. Human-gated approval cannot be satisfied by an arbitrary agent message.
8. Completed/failed execution records retain their task relationship.
9. A fresh session can recover unfinished work from durable state.
10. Redis is not required to reconstruct authoritative task/conversation state.

## Phase 7 — Tests

Add tests for:

- schema constraints;
- task state transitions;
- concurrent claim handling;
- identity/assignment boundaries;
- proposal/decision/task separation;
- approval boundary;
- checkpoint recovery;
- artifact registration;
- Design Room message ordering/threading basics;
- durable reconstruction without Redis.

Run all repository-required formatting/lint/test validation.

## Approval gates

### Gate A — before any live Cloud SQL/database mutation

Provide:

- proposed migration list;
- database/role grants;
- plan/output showing intended changes;
- rollback/recovery approach;
- confirmation of no cross-database credential broadening.

Do not cross this gate without Human approval where required by existing Wishes deployment policy.

### Gate B — before any new cost-bearing GCP resource

Do not create Cloud Run/Memorystore/Cloud SQL/NAT or other cost-bearing resources in this foundation task without the explicit required approval.

## Non-goals

Do not in this task:

- implement unrestricted shell execution;
- deploy A2A/MCP endpoints to production;
- configure ChatGPT credentials;
- configure Claude Local;
- retire the Claude inbox;
- grant broad production roles to the Operations VM;
- treat Design Room consensus as implementation authorization;
- canonize architecture before implementation acceptance.

## Required completion report

Report:

- runtime/environment confirmation;
- repositories/files reviewed;
- live-state observations available from the authorized VM;
- implementation delta from the approved reference design;
- branch name;
- files created/modified;
- `wishes_ops` migration/schema design;
- role/grant design;
- gateway modules implemented;
- tests/validation and results;
- commits pushed and SHA(s);
- any Human approval gate reached;
- exact next work unblocked for `build-agent-control-plane-mcp-a2a-claude-google.md`.
