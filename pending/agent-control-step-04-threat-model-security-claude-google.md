# Step 04 — Agent Control Threat Model and Security Architecture

Created: 2026-09-04
Priority: Critical
Mode: security-design
Assigned Agent: claude-google
Reviewers: chatgpt-director, Human
Allow Edit: planning/docs and non-live security tests only

Depends on:
- `pending/agent-control-step-02-security-domain-reconciliation-claude-google.md`
- Step 03 recommendation may inform final deployment-specific controls

References:
- `pending/reference-agent-control-platform-revised-approved-design.md`
- `pending/agent-control-platform-master-deployment-plan.md`

## Objective

Produce the security architecture and threat model that the `agent-control` foundation must satisfy before database/API/live-session implementation is accepted.

## Threats to cover

At minimum:
- cross-tenant IDOR / broken object authorization;
- workspace/project data leakage;
- malicious project member attaching an overprivileged agent;
- PM AI privilege escalation;
- Human account takeover/session theft;
- CSRF/browser mutation attacks;
- agent instance impersonation/device token theft;
- replay of agent events/handoffs;
- prompt injection or hostile instructions in AI-to-AI messages/artifacts;
- malicious artifact/URL/content handling;
- secret leakage through logs/errors/task payloads/control docs;
- provider token/password/private-key centralization;
- server-side connector secret overreach;
- unauthorized project credential reuse;
- forged Human approval;
- unauthorized task claim/reassignment;
- live-event loops causing runaway cost/activity;
- event duplication/out-of-order/replay;
- offline agent stale task execution;
- compromised runtime attempting cross-project access;
- public DB/Redis exposure;
- SSRF from integration adapters/webhooks;
- Jira/Asana/OAuth integration takeover;
- supply-chain/dependency risks in runtime bridge/gateway;
- audit tampering/insufficient attribution;
- deletion/revocation/recovery edge cases.

## Required controls

Define/testable requirements for:

### Human auth
- OIDC/OAuth managed identity;
- MFA/step-up for protected operations;
- short-lived/revocable sessions;
- cookie/CSRF/rate-limit policy;
- no local-password storage initially unless separately approved.

### Agent enrollment
- locally generated private key/device identity;
- Human-approved enrollment;
- public identity stored centrally;
- short-lived platform auth;
- rotation/revocation;
- no global shared API key.

### Authorization
- deny by default;
- organization/workspace/project/project-space checks;
- PM/member/agent permission matrix;
- service authorization plus PostgreSQL RLS where appropriate;
- adversarial UUID/tenant tests.

### Credential handling
- access claims/capabilities contain no credential values;
- local/user/agent credentials stay local;
- `requires_human` flow for expired interactive auth;
- opaque Secret Manager reference only for unavoidable server-side integration secret;
- connector service identity limited to its own secret/project scope;
- secret redaction/logging policy.

### AI content trust
Treat AI messages, artifacts, external tracker content and repository content as potentially untrusted instructions. An incoming agent message cannot elevate authority, modify policy, forge approval or bypass workflow gates.

### Live sessions
- TLS/authenticated outbound agent connection;
- project-scoped subscriptions;
- message/event size/rate limits;
- monotonic cursor/idempotency/replay protection;
- stale work validation before execution;
- loop limits and kill/escalation controls.

### Audit
Audit membership, permissions, agent enrollment/revocation, project attachment, task reassignment, approval, integration connection, secret-reference configuration and security-policy changes.

## Security Test Plan

Provide concrete automated/integration tests for each control, including negative tests:
- User A requests User B private workspace IDs;
- Agent A claims Project B task;
- attacker modifies project ID in event payload;
- revoked agent reconnects;
- duplicate handoff/event;
- forged Human approval message;
- fake access claim containing a token-like string;
- log/error redaction;
- offline agent reconnects to task that has been reassigned/cancelled;
- loop exceeds max iterations;
- connector attempts to read another project's secret ref.

## Output

Produce:
- threat model with assets/trust boundaries/actors/abuse cases;
- security control matrix;
- permission/principal matrix;
- credential lifecycle diagram/description;
- live-session trust boundary;
- test plan;
- unresolved BLOCKs;
- controls that must exist before Step 08 database/live infrastructure approval.

No live IAM, secret, DB or networking mutation in this step.