# Development Agent Control Platform — Revised Approved Architecture

Created: 2026-09-04
Status: Approved design / implementation reference
Mode: reference-design
Assigned Agent: all
Supersedes for implementation: `reference-wishes-multi-agent-control-plane-and-design-room-final-design.md`

## 1. Purpose

Build a separate, multi-user, multi-project Development Agent Control Platform (`agent-control`) for coordinating humans and AI development agents. Wishes is the first registered project, not the platform boundary.

The platform provides:
- private user workspaces and shared project spaces;
- user-linked and user-named AI agents;
- reusable workflow graphs and project-specific agent teams;
- an AI Project Manager per project;
- durable task, decision, feedback, approval and audit state;
- AI-to-AI Design Rooms for architecture/planning;
- near-real-time AI-to-AI execution sessions for coding/testing/review loops;
- optional Jira/Asana synchronization;
- secure capability declaration without sharing underlying external credentials.

The platform is not authoritative for application/game data. Each registered project remains authoritative for its own source/data/runtime.

## 2. Separate Domain and Infrastructure Boundary

`agent-control` is its own bounded domain and must not be implemented as another Wishes application database/schema.

Preferred production isolation:

```text
Dedicated GCP project / management boundary
  -> Agent Control API/Gateway
  -> dedicated PostgreSQL instance
       database: agent_control
  -> dedicated event transport / Redis where approved
  -> dedicated IAM/service identities
  -> dedicated Secret Manager namespace for server-side integration secrets only
```

Temporary co-location in an existing management project is allowed only with explicit Human approval and must preserve independent IAM, database and service identities.

Do not place control-plane tables in `wishes_core`, `wishes_assets`, `wishes_auth`, or a renamed Wishes database.

## 3. Human Identity and Sign-In Security

Prefer external managed identity (OIDC/OAuth with MFA/passkey-capable provider) rather than storing application passwords in the platform.

Store only minimum identity/session records needed by the platform: provider subject, approved profile metadata, memberships, session/device metadata and audit events.

Requirements:
- TLS only;
- secure HTTP-only SameSite browser cookies where applicable;
- short-lived sessions/tokens and revocation;
- MFA/step-up authentication for sensitive administration and protected approvals;
- CSRF protection on browser mutations;
- rate limiting and abuse controls;
- no passwords, MFA seeds, recovery codes or external-provider refresh tokens in ordinary application tables.

## 4. Agent Runtime Enrollment

Each concrete agent runtime is independently enrolled.

Examples:
```text
Ryan / Forge / claude-google
Ryan / Scout / claude-local
Alice / Builder / claude-local
Wishes Team / Reviewer / claude-google
```

Preferred model:
1. runtime bridge generates a local keypair/device identity;
2. Human links/enrolls the runtime through an authenticated device-code or equivalent approval flow;
3. control plane stores the public identity and enrollment metadata, not the private key;
4. runtime uses revocable short-lived platform credentials;
5. private keys and model/provider login material remain local to that runtime.

No global shared API key for all agents.

## 5. Credential Non-Transfer Rule

Agent profiles express *capabilities/access claims*, never secret values.

Example:
```text
Agent: Forge
Capabilities:
- code.backend
- code.review
- terraform.plan

Access claims:
- github:Krowthen/wishes-game:write
- gcp:wishes-506905:terraform-plan
- local:docker

Credential posture:
- github -> local_agent
- gcp -> human_interactive
- anthropic -> local_agent
```

The control plane may record:
- provider/resource;
- scope/access level;
- credential-location class;
- availability state;
- last verification time;
- whether Human intervention is required.

Allowed credential-location classes:
```text
none
local_user
local_agent
human_interactive
external_provider
secret_manager_service_integration
```

The platform must not store actual GitHub tokens, Google ADC refresh material, Claude/OpenAI credentials, passwords or equivalent secrets in agent profiles/tasks/control documents.

`secret_manager_service_integration` is allowed only when a true server-side adapter requires a secret. The database stores only an opaque Secret Manager reference plus metadata. Secrets must be isolated by integration/project/service identity and never returned via normal APIs.

An agent may advertise access while currently requiring Human sign-in:
```text
access_claim: gcp:wishes-506905
status: requires_human
reason: interactive sign-in expired
```
The PM/workflow engine treats this as blocked and requests Human intervention. Another agent must never be asked to provide the missing credential.

## 6. Multi-User / Multi-Project Ownership

Hierarchy:
```text
Organization
  -> Users
  -> Workspaces
  -> Projects
  -> Project Spaces
  -> Tasks / Design Rooms / Workflows / Agents
```

Every developer receives a personal workspace. Shared workspaces may be created for teams.

A project can appear in several project spaces:
```text
Ryan Personal / Wishes
Alice Personal / Wishes
Wishes Team / Wishes
```

Private experimentation can later be promoted/shared without losing provenance.

Project/workspace isolation must be enforced in service authorization and, where practical, PostgreSQL RLS/tenant-scoped constraints. Guessing another tenant/project UUID must not reveal data.

## 7. Projects and PM AI

A user can create a project or be assigned to one. Project Owners manage membership and assign the active primary Project Manager AI.

Project members with permission may attach their own AI agents to the project.

Initial rule: one active primary PM AI per project.

PM responsibilities:
- maintain project operational state;
- decompose/create tasks;
- route by role/capability or named agent;
- accept eligible agent claims;
- track dependencies/blockers;
- request reviews/Human approvals;
- coordinate Design Rooms and live execution loops;
- maintain the shared Project Control Document;
- summarize feedback/status;
- escalate policy/security/credential blocks.

The PM never overrides Human/project security policy.

## 8. Shared Project Control Document

Every project has a shared PM-managed control document.

Canonical source = structured control-plane state. Markdown/JSON may be rendered to Git for portability but is not the task database.

It surfaces:
- purpose/scope;
- repos/environments;
- members/agent team;
- PM;
- workflows;
- priorities/tasks;
- blockers/dependencies;
- recent decisions;
- pending approvals;
- capability/access availability summaries without secrets;
- feedback;
- branch/commit/artifact references;
- project-specific rules.

## 9. Agent Model

Separate:
```text
agent_provider
agent_connection
agent_profile
agent_instance
project_agent_assignment
```

`agent_provider`: ChatGPT/OpenAI, Claude Code, Claude Coop, future providers.

`agent_connection`: how a user/runtime/provider link is enrolled; no raw provider secret in normal tables.

`agent_profile`: user-defined name, description, roles/capability declarations.

`agent_instance`: concrete runtime/device, online/offline state and enrollment identity.

`project_agent_assignment`: project role, role definition, permissions and status.

Default role/runtime names remain:
```text
chatgpt-director
claude-coop
claude-google
claude-local
openai-director (optional API runtime)
```
Users may assign friendly names.

## 10. Workflows

Users can build workflow templates of steps/transitions.

A step may target:
- named agent;
- project role;
- capability/role requirement;
- Human;
- claimable pool.

Inheritance:
```text
Organization security policy
 -> Project policy
 -> Workspace/project-space preferences
 -> Workflow template
 -> Task routing
```
Lower levels cannot weaken higher-level security/approval policy.

## 11. Task Assignment / Claiming

Modes:
```text
direct
role-routed
claimable
human-only
```

Eligibility considers:
- membership/agent attachment;
- role/capability;
- environment requirement;
- dependencies;
- security/project policy;
- declared credential/access availability;
- load/claim constraints.

Agents may claim eligible work where workflow permits; PM may assign directly.

## 12. Three Communication Modes

### 12.1 Task Control
Durable structured work: tasks, dependencies, assignments, approvals, checkpoints, artifacts and outcomes.

### 12.2 Design Rooms
Default Wishes flow:
```text
ChatGPT scope + initial design
 -> claude-google baseline implementation reality check
 -> ChatGPT <-> claude-coop iterative comparison/challenge
 -> claude-google final implementation validation
 -> 0 unresolved BLOCKs
 -> DESIGN_SIGNOFF / IMPLEMENTATION_SIGNOFF / DIRECTOR_SIGNOFF
 -> Human approval when required
 -> Decision
 -> Tasks
```
Design discussion never directly authorizes implementation.

### 12.3 Live Agent Sessions
Near-real-time execution collaboration, e.g. coding-agent <-> testing-agent.

Required:
- runtime bridge maintains outbound authenticated live connection to Gateway (WebSocket or approved equivalent);
- agents subscribe to workflow/event types;
- relevant events push immediately; no Human instruction to check updates;
- durable state/outbox commits before or transactionally with transient publication;
- offline events replay from durable cursor/checkpoint on reconnect;
- loops have iteration/time/cost limits and escalation.

Example:
```text
coding-agent
 -> code.ready_for_test
 -> testing-agent automatically validates
 -> test.failed
 -> coding-agent automatically receives failure
 -> fix
 -> code.ready_for_test
 -> testing-agent
 -> test.passed
 -> PM / next step
```
No direct remote-shell trust between agents.

## 13. Live Session Model

Add:
```text
agent_live_session
agent_subscription
agent_event
agent_handoff
agent_event_cursor
```

`agent_live_session`: availability/session metadata, not model/provider secrets.

`agent_subscription`: event types allowed to wake/invoke an agent.

`agent_handoff`: from/to, task, requested action, artifact/commit reference, return/escalation path.

## 14. Dedicated Database Model

Database: `agent_control` on its own platform database boundary.

```text
ORGANIZATION / PEOPLE
organization
user_identity
organization_member

WORKSPACES
workspace
workspace_member

PROJECTS
project
project_member
project_space
project_repository
project_environment
project_policy
project_control_document
project_control_document_version

AGENTS
agent_provider
agent_connection
agent_profile
agent_instance
agent_role_definition
agent_capability
agent_access_claim
project_agent_assignment

WORKFLOWS
workflow_template
workflow_step
workflow_transition
workflow_run

TASKS / FEEDBACK
task
task_dependency
task_assignment
task_claim
task_update
feedback

DESIGN
agent_conversation
agent_conversation_participant
agent_chat_message
agent_proposal
agent_decision
agent_signoff

LIVE COMMUNICATION
agent_live_session
agent_subscription
agent_event
agent_handoff
agent_event_cursor

EXECUTION
agent_execution
agent_checkpoint
agent_artifact

AUTHORITY
agent_approval

INTEGRATIONS
integration_provider
integration_connection
project_integration
external_work_item

AUDIT / RELIABILITY
activity_event
outbox_event
idempotency_record
```

Use typed ownership/status fields. JSON is for provider-specific/flexible detail, never primary tenant boundaries.

## 15. Near-Real-Time Transport

```text
Agent runtime bridge
  <== authenticated TLS outbound live connection ==>
Agent Gateway
  -> authz/workflow engine
  -> PostgreSQL agent_control (durable)
  -> transient event bus / Redis Streams if approved
```

Do not expose Redis or PostgreSQL directly to developer machines or AI runtimes.

Live connections authenticate to a specific `agent_instance` and permitted organization/workspace/project context. Offline replay uses durable event/outbox state.

## 16. MCP / A2A

MCP = agent-to-platform tools/context.
A2A = agent-to-agent collaboration adapter where supported.

Both terminate at the Gateway and use the same authorization/domain state. Neither reveals credentials or bypasses project policy.

## 17. Feedback and Updates

Humans and AIs can add attributed/timestamped project-scoped updates: progress, test results, review feedback, user feedback, PM transitions, design objections and Human decisions.

## 18. Jira / Asana

Optional adapters, not platform dependencies.

Model:
```text
integration_provider
integration_connection
project_integration
external_work_item
```

Tracking mode:
```text
internal
jira
asana
hybrid
```

Integration credentials follow the same rule: local/user-held where feasible, otherwise project-scoped Secret Manager reference for a server-side connector. Never put connector tokens in tasks/control docs.

## 19. Mandatory Security Controls

1. deny-by-default authz;
2. organization/workspace/project checks on every read/write;
3. separate Human and agent principals;
4. revocable runtime enrollment;
5. least-privilege service identities;
6. no generic arbitrary-shell MCP/API endpoint;
7. no credential values in normal APIs/database/logs;
8. secret redaction in logs/events/errors;
9. audit membership, permission, agent attachment, approval and integration changes;
10. adversarial IDOR/cross-tenant tests;
11. personal workflows cannot weaken project/org policy;
12. AI messages cannot forge Human approvals;
13. rate limits for live event loops;
14. max iteration/time/cost limits;
15. project/connection-scoped server integration secrets;
16. credential rotation/revocation procedures;
17. PostgreSQL/Redis private-only;
18. encryption in transit/at rest;
19. no project production credentials stored in control DB;
20. access claims are routing hints and verified at execution time where required;
21. no agent may request another user's/agent's external credential as a workaround;
22. Human intervention is explicit when interactive authentication is required.

## 20. PM Authority Boundary

PM may coordinate within project policy but cannot add itself without permission, grant new external access, obtain another user's credentials, override Human gates, weaken policy, or silently convert design discussion into protected execution.

## 21. Repository / Deployment Separation

Preferred: dedicated repository/service domain for the platform. Until provisioned, isolated bootstrap code may exist temporarily in an existing repository only with an explicit extraction/migration plan.

Suggested components:
```text
agent-control/
  gateway/
  web/
  mcp/
  a2a/
  runtime-bridge/
  workflow-engine/
  integrations/
  database/
  terraform/
  docs/
```

## 22. Acceptance Criteria

Prove:
- multiple users with isolated personal spaces;
- shared project spaces and multiple projects;
- user-linked/named agents;
- secure enrollment/revocation;
- no provider secret leakage;
- PM assignment/control document;
- role-routed/claimable tasks;
- reusable workflows;
- Design Room sign-off flow;
- near-real-time coding/testing loop without manual polling;
- offline replay/recovery;
- Human intervention for expired/missing interactive access;
- cross-tenant authorization tests;
- external tracker adapter boundary;
- durable audit/provenance;
- transient transport loss does not destroy state;
- protected production authority remains external/human-controlled.

## 23. Implementation Rule

All older Wishes-specific agent-control tasks must be reconciled to this reference before migrations/infrastructure are created. `wishes_ops` is superseded by the independent `agent_control` domain/database. No live cost-bearing, IAM-sensitive, credential-sensitive or database apply occurs without the required Human approval package.