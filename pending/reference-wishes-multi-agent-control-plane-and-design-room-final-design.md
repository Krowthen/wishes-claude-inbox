# Wishes Multi-Agent Control Plane and Design Room — Final Approved Design

Created: 2026-09-04
Status: Approved design / implementation reference
Mode: reference-design
Assigned Agent: all
Execution: Do not treat this file as a standalone implementation task. Use the agent-specific implementation tasks that reference this document.

## 1. Purpose

This document is the approved target architecture for coordinating Wishes development across the Human owner, ChatGPT, Claude Web/Coop, Claude Code on the Google Operations VM, and Claude Code on the local Windows workstation.

It replaces the long-term use of `wishes-claude-inbox` as the primary AI-to-AI transport with a durable Wishes Agent Control Plane, while retaining the inbox as the interim/manual transport and future archive/fallback mechanism.

The design also establishes a parallel AI-to-AI Design Room workflow for architecture, planning, design comparison, technical challenge, and implementation review.

The architecture must preserve existing Wishes authority rules:

- Human remains the final authority.
- ChatGPT directs and leads architecture/design.
- Claude agents may inspect, design, edit, test, commit, and prepare changes within assigned authority.
- Normal protected production mutation remains human/protected-workflow controlled.
- GitHub remains the durable engineering record for source, documentation, and implementation artifacts.
- PostgreSQL is authoritative for durable agent-control state.
- Redis is transient coordination only.
- Design discussion is non-authoritative until promoted through an explicit decision/task workflow.

---

## 2. Active Parties and Runtime Identities

### 2.1 Management parties

1. `human-owner` — Human project owner and final authority.
2. `chatgpt-director` — lead architect, directing agent, task decomposition, synthesis, review, and coordination.
3. `claude-google` — Claude Code running on the Google Cloud Operations VM.
4. `claude-local` — Claude Code running on the local Windows development workstation.

### 2.2 Design-room participant

5. `claude-coop` — Claude Web/Coop used primarily for iterative architecture discussion and design challenge.

### 2.3 Optional server-side OpenAI runtime

A future/API-backed `openai-director` may represent the ChatGPT design/director role for unattended machine-to-machine turns. It does not replace the interactive ChatGPT UI and may not expand production authority.

### 2.4 Naming rules

These identifiers are canonical and lower-case:

```text
human-owner
chatgpt-director
claude-coop
claude-google
claude-local
openai-director   # optional runtime identity
```

Do not use ambiguous names such as `claude`, `cloud-claude`, `local-claude`, `claude-vm`, or `claude-2` in control-plane task ownership fields.

---

## 3. Current Environment Baseline

The existing Wishes development topology already separates:

- a Linux Claude Code Operations VM in Google Cloud for persistent cloud development/orchestration;
- a local Windows workstation for interactive Unity development and visual validation;
- separate Claude Code processes that do not automatically share terminal state, in-memory context, or running shell sessions;
- Git/branches/commits/PRs and the Claude inbox as the current durable coordination boundary.

The Operations VM is a controlled engineering workstation, not a production application server and not an unrestricted autonomous production controller.

The local Windows workstation remains the primary interactive Unity/Play Mode/visual validation environment.

The new control plane must preserve that separation rather than trying to make the two Claude Code instances share one live shell or context window.

---

## 4. Target Architecture

```text
                              HUMAN
                                |
                                v
                         ChatGPT Director
                                |
             +------------------+------------------+
             |                                     |
             v                                     v
      AUTHORITATIVE WORK                       DESIGN ROOMS
        CONTROL PLANE                       non-authoritative
             |                                collaboration
             |                                     |
      +------+------+                       +------+------+
      |             |                       |             |
      v             v                       v             v
claude-google  claude-local            claude-coop   claude-google
      |             |                       ^             |
      |             |                       |             |
      +------+------+                       +------^------+
             |                                     |
             v                                     |
           GitHub                         ChatGPT Director
   code/docs/artifacts                          synthesis
             ^                                     |
             |                                     v
             +------------------------- Proposal / Decision
                                                   |
                                                   v
                                             Control Plane
```

### 4.1 Core principle

Claude Local and Claude Google do not need direct terminal-to-terminal communication. Both read and write the same durable control-plane state.

ChatGPT directs through that state.

A new ChatGPT or Claude session can recover current work from structured tasks, checkpoints, decisions, messages, approvals, branches, commits, and artifacts rather than depending on the previous chat/session memory.

---

## 5. Responsibility Boundaries

### 5.1 Human — `human-owner`

Human retains authority for:

- project direction;
- protected/destructive operations;
- production approval gates;
- permanent infrastructure deletion;
- sensitive IAM/security changes;
- production database destructive migration approvals;
- final architecture decisions when blocking disagreement remains;
- final canon approval/canonization where required;
- expansion of agent permissions.

### 5.2 ChatGPT — `chatgpt-director`

Primary responsibilities:

- define problem scope;
- define constraints and non-goals;
- lead architecture and system design;
- create initial design proposals;
- decompose work into tasks;
- assign work to Claude environments;
- facilitate Design Rooms;
- compare alternatives with `claude-coop`;
- synthesize final proposals;
- review implementation results;
- request Human decisions/approvals;
- maintain consistency with Wishes architecture/canon;
- determine whether local, Google, or both environments are required.

ChatGPT is the normal directing layer but not a substitute for Human protected-action authority.

### 5.3 Claude Web/Coop — `claude-coop`

Primary Design Room role: co-designer and challenger.

Responsibilities:

- challenge ChatGPT assumptions;
- propose alternative architectures;
- identify edge cases;
- compare design tradeoffs;
- identify hidden coupling;
- question incomplete requirements;
- challenge unnecessary complexity;
- review final design coherence;
- issue `DESIGN_SIGNOFF`, `APPROVE_WITH_NOTES`, or `BLOCK`.

`claude-coop` is not the default implementation agent and does not silently turn design discussion into repository changes.

### 5.4 Claude Google — `claude-google`

Primary implementation role: cloud/backend/infrastructure implementation and reality review.

Responsibilities include:

- inspect the real repository and current implementation;
- validate designs against code, schemas, Terraform, GCP, `CLAUDE.md`, `WORKFLOW.md`, and Wishes canon;
- identify migration/test/security/cost consequences;
- perform baseline and final Design Room reality checks;
- implement assigned server/database/cloud/Terraform/control-plane tasks;
- run tests and plans;
- commit/push implementation on approved branches;
- report checkpoints and artifacts;
- issue `IMPLEMENTATION_SIGNOFF`, `APPROVE_WITH_NOTES`, or `BLOCK` on final designs.

During a Design Room review, Claude Google may inspect and test but must not silently implement design changes unless an implementation task authorizes them.

### 5.5 Claude Local — `claude-local`

Primary implementation role: local Windows/Unity/client-specialist work.

Responsibilities include:

- interactive Unity development;
- Unity Editor/Play Mode work;
- Windows-specific validation;
- graphical debugging and visual QA;
- local tooling and local ComfyUI work where applicable;
- client API integration;
- pulling and validating cloud-created branches;
- reporting structured validation results.

Claude Local is not a default participant in every Design Room. Invite it when the design materially affects Unity, Windows behavior, visual workflows, client performance, interactive tooling, local ComfyUI, rendering, animation, input, or developer-facing Unity workflows.

---

## 6. Authoritative Control Plane

The Control Plane answers:

- What work exists?
- Who owns it?
- What is its status?
- What is blocked?
- What dependency must finish first?
- What branch/commit/artifact represents the work?
- What decision authorized it?
- What approval is required?
- What should happen next?

### 6.1 Durable state

Create a dedicated operations database:

```text
wishes_ops
```

This is separate from:

```text
wishes_core
wishes_assets
wishes_auth
```

Reason: agent coordination, approval, and engineering workflow state is operational state with different authority, retention, and blast-radius concerns. It must not be mixed with game/world authority, asset-generation data, or authentication data.

### 6.2 Initial database roles

```text
wishes_ops_owner
wishes_ops_runtime
wishes_ops_auditor
```

No agent-control runtime receives `wishes_auth` credentials merely because it uses the same Cloud SQL instance.

### 6.3 Initial schema

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

### 6.4 Task record minimum fields

```text
task_id
parent_task_id
created_by
assigned_to
task_type
priority
status
objective
constraints
approval_policy
related_decision_id
repository
branch
created_at
started_at
completed_at
```

### 6.5 Execution record minimum fields

```text
execution_id
task_id
agent_id
environment
status
repository
branch
commit_sha
started_at
completed_at
result
```

### 6.6 Checkpoint purpose

Checkpoints replace dependence on model conversation memory.

A checkpoint must be created before meaningful unfinished work is handed off, paused, blocked on approval, or left across a session boundary.

Example:

```json
{
  "task_id": "TASK-312",
  "agent": "claude-google",
  "status": "blocked-on-local-validation",
  "summary": "Implementation complete and automated tests pass.",
  "repository": "Krowthen/wishes-game",
  "branch": "feature/example",
  "commit": "abc123",
  "next_agent": "claude-local",
  "next_action": "Run Unity Play Mode validation."
}
```

---

## 7. Redis/Event Coordination

Redis remains transient coordination only.

Suggested logical streams/topics:

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

Every event must reference durable IDs such as `task_id` or `conversation_id`.

Loss or flush of Redis must not erase authoritative task/conversation/approval state.

Use an outbox/idempotency pattern appropriate to the existing Wishes architecture so durable state and event publication can reconcile safely.

---

## 8. Agent Gateway

Create a Wishes Agent Gateway/control service, initially deployable as one Cloud Run service if that keeps S0 simpler.

Suggested service name:

```text
wishes-agent-gateway
```

Suggested code areas:

```text
services/agent-gateway/
services/agent-control-api/
services/agent-mcp/
services/agent-a2a/
scripts/agentctl/
```

These may initially be modules of one deployable service rather than separate Cloud Run services.

### 8.1 Gateway responsibilities

- authentication;
- agent identity;
- authorization;
- task CRUD/lifecycle;
- assignment/claim rules;
- checkpoints;
- artifact registration;
- approvals;
- Design Room persistence;
- proposal/decision promotion;
- Redis publication/consumption;
- structured audit logs;
- MCP endpoint;
- A2A endpoint/adapter.

### 8.2 Explicit non-responsibilities

The gateway must not itself become an unrestricted arbitrary-shell executor.

It must not provide a generic endpoint equivalent to:

```text
run any shell command
run arbitrary Terraform apply
execute arbitrary production SQL
```

Execution remains inside authorized runtime environments or protected CI/CD.

---

## 9. MCP — Agent to Wishes

MCP is the shared tool/context interface.

Recommended MCP server identity:

```text
wishes-control
```

Initial tools should include:

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

Permissions are role-specific.

No MCP tool grants broader shell/cloud/database access than the caller already has.

---

## 10. A2A / Design Room — AI to AI

A2A is the intended AI-to-AI collaboration interface for structured architecture/design discussion.

Conceptual distinction:

```text
MCP = AI -> Wishes tools/data/control state
A2A = AI -> AI collaboration
```

Both map to the same `wishes_ops` durable records. Do not create a second independent conversation/task database for A2A.

---

## 11. Canonical Design Room Workflow

The approved default flow is:

```text
Human request / architecture question
        |
        v
ChatGPT sets scope, constraints, non-goals, and initial design
        |
        v
Claude Google baseline reality check
(repo/code/schema/Terraform/GCP/canon)
        |
        v
+-----------------------------------------+
| ChatGPT <-> Claude Coop                 |
| iterative design comparison             |
| alternatives / challenges / revisions   |
+-----------------------------------------+
        |
        v
Claude Google final implementation reality check
        |
        v
Unresolved blocker?
   | yes                 | no
   v                     v
Return to design       Sign-off stage
                         |
             +-----------+-----------+
             |           |           |
             v           v           v
       Claude Coop   Claude Google  ChatGPT
       design        implementation director
       signoff       signoff        signoff
             +-----------+-----------+
                         |
                         v
                Human approval if required
                         |
                         v
                      Decision
                         |
                         v
                       Tasks
                         |
                         v
              Claude Google / Local
```

### 11.1 Why Claude Google reviews twice

The first review grounds the initial proposal in actual implementation reality.

The second review validates the final evolved design after ChatGPT/Claude Coop debate. A final architecture may differ materially from the initial design, so the first review cannot be assumed to remain valid.

### 11.2 Sign-off semantics

Do not require fake unanimous wording.

Each reviewer returns one of:

```text
APPROVE
APPROVE_WITH_NOTES
BLOCK
```

A `BLOCK` must include a concrete unresolved reason.

Completion requires:

```text
0 unresolved blocking objections
```

Specific sign-offs:

```text
DESIGN_SIGNOFF         -> claude-coop
IMPLEMENTATION_SIGNOFF -> claude-google
DIRECTOR_SIGNOFF       -> chatgpt-director
HUMAN_APPROVAL         -> when required by authority/risk
```

### 11.3 Claude Local specialist participation

Claude Local is added when the room materially affects:

- Unity;
- Windows-specific execution;
- Play Mode;
- visual/rendering/animation workflows;
- local ComfyUI;
- client performance;
- local developer tooling;
- user interaction/input behavior.

Claude Local may provide `CLIENT_SIGNOFF` or `BLOCK` for those concerns.

---

## 12. Design Room Data Model

### `agent_conversation`

Minimum fields:

```text
conversation_id
title
objective
conversation_type
facilitator
status
related_task_id
created_by
created_at
closed_at
max_rounds
current_round
requires_human_review
```

Conversation types:

```text
architecture
planning
design
debugging
review
brainstorm
risk-review
```

Statuses:

```text
open
waiting_agent
waiting_human
synthesizing
resolved
cancelled
archived
```

### `agent_conversation_participant`

```text
conversation_id
agent_id
role
joined_at
left_at
```

### `agent_chat_message`

```text
message_id
conversation_id
round
sender
recipient
reply_to
message_type
content
structured_data
created_at
```

Message types:

```text
proposal
response
challenge
question
evidence
alternative
risk
agreement
disagreement
summary
decision-request
human-intervention
```

### `agent_proposal`

Should capture:

```text
problem
constraints
recommended_design
alternatives
tradeoffs
risks
open_questions
affected_components
affected_repositories
migration_requirements
security_implications
cost_implications
required_tasks
```

### `agent_decision`

Minimum fields:

```text
decision_id
proposal_id
title
decision
rationale
alternatives_considered
consequences
approved_by
created_at
```

---

## 13. Non-Authoritative Discussion Rule

A Design Room cannot directly mutate Wishes merely because agents agree.

Required lifecycle:

```text
Design Room
   -> Proposal
   -> Decision
   -> Task
   -> Assigned Agent
   -> Implementation
```

Forbidden lifecycle:

```text
Design discussion
   -> agent notices an improvement
   -> silently edits implementation
```

During design review, agents may inspect repositories, run safe read-only checks, and execute non-mutating validation/tests as allowed by existing permissions.

---

## 14. Human Intervention and Disagreement

Human may enter a Design Room at any time and introduce a new authoritative constraint or decision.

Agents are not required to reach consensus.

If blocking disagreement remains after the configured discussion budget, the Director must summarize:

- Option A;
- Option B;
- evidence/tradeoffs;
- Claude Coop position;
- Claude Google implementation concern;
- ChatGPT recommendation;
- exact Human decision required.

Do not force artificial consensus.

---

## 15. Conversation Limits / Cost Control

Every Design Room must include:

```text
max_rounds
time/token/cost budget where measurable
facilitator
exit criteria
```

Initial recommended default:

```text
max_rounds: 6
max_consecutive_turns_same_agent: 2
```

When the limit is reached, `chatgpt-director`/`openai-director` must:

1. synthesize;
2. request Human input; or
3. explicitly extend the room.

Agents may not trigger unbounded autonomous debate.

---

## 16. Interim Manual Routing Before Control Plane Cutover

Until the Control Plane is operational, `wishes-claude-inbox/pending/` remains the task transport.

Every new executable task must contain an explicit field:

```text
Assigned Agent: claude-google
```

or:

```text
Assigned Agent: claude-local
```

Optionally:

```text
Assigned Agent: any
```

only where either environment is genuinely permitted to execute the task.

Each Claude Code environment must update its `CLAUDE.md`/environment instructions to identify its canonical runtime ID and route tasks accordingly.

Interim rule:

- `claude-google` executes tasks explicitly assigned to `claude-google`.
- `claude-local` executes tasks explicitly assigned to `claude-local`.
- A runtime must leave a task assigned to the other runtime untouched unless Human explicitly overrides the assignment.
- Shared/reference design documents are not automatically executable tasks.

---

## 17. GitHub and Engineering Traceability

GitHub remains authoritative for durable engineering changes.

Desired trace:

```text
Requirement
 -> Design Room
 -> Proposal
 -> Decision
 -> Task
 -> Branch/Commit/PR
 -> Validation
```

Implementation commits should include task/decision references where practical, for example:

```text
feat(agent): persist agent task lifecycle

Task: AGENT-012
Decision: AGENT-DEC-003
```

No unique source change should exist only on the Operations VM or local workstation.

---

## 18. Security and Authority Rules

The new coordination architecture must not expand existing production authority.

Preserve:

- Operations VM no-public-IP model;
- IAP/OS Login access controls;
- intentionally weak attached VM identity;
- service-account impersonation boundaries;
- no routine static broad service-account keys;
- no unrestricted MCP/A2A execution tool;
- no agent self-approval of Human-gated actions;
- no production/destructive mutation based solely on another agent message;
- auditability of approvals and agent identity.

A Human approval is valid only when recorded through an approved Human control path. An agent statement claiming Human approval is not itself authorization.

---

## 19. Offline and Recovery Behavior

The system must support one runtime being offline.

Example:

```text
claude-local offline
 -> local task remains queued/durable
 -> claude-google does not silently steal it
 -> ChatGPT sees assignment/status
 -> claude-local later reconnects and retrieves the task
```

Fresh agent sessions must recover state using `get_project_state`, assigned tasks, open Design Rooms, checkpoints, and repository state.

The control plane must not depend on the previous model context window.

---

## 20. Inbox Lifecycle

During bootstrap:

```text
ChatGPT/Human
 -> wishes-claude-inbox/pending
 -> explicitly assigned Claude environment
```

After successful cutover:

```text
ChatGPT/Human
 -> Wishes Agent Control Plane
 -> assigned agent
```

The inbox remains:

- human-readable archive;
- emergency fallback;
- bootstrap transport;
- historical task record during migration.

It should no longer be required for routine agent-to-agent transport after the Control Plane is accepted.

---

## 21. Implementation Phases

### Phase A — Interim identity/routing bootstrap

1. Update Claude Google environment instructions with canonical identity `claude-google`.
2. Update Claude Local environment instructions with canonical identity `claude-local`.
3. Add interim task-routing rules based on the `Assigned Agent` field.
4. Verify each runtime ignores tasks assigned to the other runtime.

### Phase B — Environment inventory and implementation planning

5. Claude Google inventories the live Operations VM and relevant S0 GCP resources.
6. Review current `wishes-game`, `wishes-canon`, `CLAUDE.md`, `WORKFLOW.md`, database migration conventions, Redis, Cloud SQL, and Cloud Run topology.
7. Produce implementation delta/risk report before any apply.

### Phase C — Durable control-plane foundation

8. Add `wishes_ops` design/migrations/roles.
9. Add agent identity/task/execution/checkpoint/artifact/approval schema.
10. Add Design Room/proposal/decision schema.
11. Add tests for database/role boundaries.

### Phase D — Gateway/event layer

12. Implement Agent Gateway application layer.
13. Implement authorization/agent identities.
14. Implement Redis/outbox/idempotency coordination.
15. Implement task/approval/checkpoint/artifact APIs.
16. Implement Design Room conversation APIs.

### Phase E — MCP and A2A

17. Implement `wishes-control` MCP interface.
18. Implement A2A/Design Room adapter mapped to the same durable records.
19. Add role-specific permissions.
20. Add conversation limits and sign-off semantics.

### Phase F — Google deployment

21. Add Terraform/Cloud Run/Secret Manager/IAM configuration.
22. Run plans and security checks.
23. Human approval before apply/cost-bearing changes.
24. Deploy approved control plane to `us-central1`.
25. Configure `claude-google` against it.

### Phase G — Local integration

26. Configure `claude-local` against the shared control plane.
27. Validate Windows credential handling and task retrieval.
28. Validate local specialist Design Room participation.
29. Validate offline/reconnect behavior.

### Phase H — ChatGPT/Claude Coop integration

30. Connect ChatGPT/Wishes control interface according to available account/tool capabilities.
31. Add `claude-coop` Design Room participation path.
32. Add API-backed `openai-director` only where unattended ChatGPT-role turns are required.

### Phase I — Acceptance

33. ChatGPT -> Claude Google task test.
34. Claude Google -> Claude Local handoff test.
35. Design Room test using the control-plane architecture itself as the first real review topic.
36. Baseline Google reality check.
37. ChatGPT <-> Claude Coop iterative design discussion.
38. Final Google implementation review.
39. Sign-off state test.
40. Human approval test.
41. Fresh ChatGPT/context recovery test.
42. Offline local runtime test.
43. Full Requirement -> Design -> Decision -> Task -> Commit trace test.

### Phase J — Cutover/canon

44. Human approves control-plane-first cutover.
45. Inbox becomes archive/fallback rather than routine transport.
46. Reconcile Wishes canon/deployment docs with the implemented architecture.
47. Follow existing canon review/canonization workflow.

---

## 22. Acceptance Criteria

Implementation is not complete until all of the following are proven:

```text
[ ] canonical identities exist for human-owner/chatgpt-director/claude-coop/claude-google/claude-local
[ ] interim inbox routing distinguishes claude-google from claude-local
[ ] wishes_ops exists with isolated roles
[ ] task/execution/checkpoint/artifact/approval state is durable
[ ] Design Room/proposal/decision state is durable
[ ] Redis loss does not lose authoritative work state
[ ] Agent Gateway enforces role boundaries
[ ] MCP works without exposing unrestricted execution
[ ] A2A/Design Room messages map to the same durable state
[ ] claude-google can recover work in a fresh session
[ ] claude-local can recover work in a fresh session
[ ] local offline tasks remain assigned/durable
[ ] Claude Google performs both baseline and final Design Room reality reviews
[ ] Claude Coop can challenge and sign off design
[ ] ChatGPT can synthesize/promote a proposal
[ ] Design Room discussion cannot directly authorize implementation
[ ] protected Human approval cannot be forged by an agent message
[ ] commits/artifacts are traceable to tasks/decisions
[ ] no production authority is accidentally expanded
[ ] end-to-end four-party workflow passes
[ ] inbox can be retired to archive/fallback role
```

---

## 23. Approved Operating Summary

```text
Human = final authority
ChatGPT = lead design + directing agent
Claude Coop = architecture co-designer/challenger
Claude Google = code/cloud reality reviewer + cloud/backend implementer
Claude Local = local Windows/Unity implementer + optional client specialist

MCP = AI -> Wishes tools/control state
A2A = AI -> AI design collaboration
wishes_ops = durable project/agent memory
Redis = transient coordination/events
GitHub = durable engineering truth
Design Room = non-authoritative discussion
Decision/Task = authorization boundary
```

This document is approved as the implementation reference. Agent-specific task files in `pending/` define the actual executable work and dependencies.