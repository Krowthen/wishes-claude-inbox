# Task: Integrate `claude-local` with Agent Control Platform

Created: 2026-09-04
Priority: High after gateway deployment
Mode: implementation-and-validation
Assigned Agent: claude-local
Execution Environment: Local Windows Claude Code workstation
Allow Edit: true

Depends on:
- `pending/bootstrap-claude-local-runtime-identity.md`
- successful secure Agent Control deployment from `pending/deploy-agent-control-plane-gcp-claude-google.md`

References:
- `pending/reference-agent-control-platform-revised-approved-design.md`
- `pending/agent-control-platform-master-deployment-plan.md`

## Objective

Enroll the local Windows Claude Code runtime as an independent `agent_instance`, keep all external/provider credentials local, validate secure project/task access, and prove near-real-time push/handoff/reconnect behavior without inheriting `claude-google` privileges.

## Phase 1 — Verify prerequisites

1. Confirm this is the local Windows environment and canonical runtime role is `claude-local`.
2. Review applicable `CLAUDE.md`, `WORKFLOW.md`, local config and inbox tooling.
3. Obtain only the approved Agent Control endpoint/enrollment procedure.
4. Never copy credentials from `claude-google` or another user/agent.
5. Confirm provider/GitHub/local credentials remain local to this machine according to approved local security practice.

## Phase 2 — Secure runtime enrollment

Use the approved device/runtime enrollment flow:
- local bridge generates/uses local private device identity;
- Human approves/links runtime;
- platform stores public enrollment identity/metadata;
- runtime uses short-lived/revocable platform auth;
- no permanent shared API key;
- no private key/provider credential is committed or sent to normal Agent Control records.

Verify revocation/re-enrollment procedure without exposing secret material.

## Phase 3 — Agent profile/access posture

Validate that Agent Control can represent metadata such as:
```text
role: claude-local
capabilities: unity/windows/local-testing
access: github project repo write (if currently available)
credential location: local_agent/local_user
status: available | requires_human | unavailable
```
without revealing the actual credential.

If any interactive login is expired, report `requires_human`; do not request another agent's credential.

## Phase 4 — MCP/task authorization

Prove from fresh session:
- list only authorized projects/spaces;
- retrieve own/shared project state;
- list/claim eligible local tasks;
- cannot read private workspace/project state without membership;
- cannot claim `claude-google`/other-user-only task;
- create progress/checkpoint/artifact/feedback within authorization.

## Phase 5 — Live push runtime bridge

Configure approved outbound TLS live bridge and subscriptions.

Prove:
1. local runtime receives an assigned/claimable test event without Human saying "check for updates";
2. acknowledgement/cursor is recorded;
3. runtime receives a structured handoff from another agent;
4. task/event is project scoped;
5. no Redis/DB direct connectivity exists from the workstation.

## Phase 6 — Offline/reconnect replay

With Human coordination:
1. disconnect/stop the local bridge;
2. queue one or more authorized events/tasks;
3. verify durable state remains;
4. reconnect;
5. replay only outstanding events in correct logical order;
6. verify cancelled/reassigned/stale work is revalidated before execution;
7. continue without previous chat memory.

Do not unexpectedly power off the workstation.

## Phase 7 — Cloud/local handoff

Prove:
```text
claude-google completes/updates task
 -> emits handoff/dependency event
 -> claude-local receives push
 -> pulls referenced branch/commit where authorized
 -> performs Windows/Unity/local validation
 -> returns result/feedback
 -> PM/workflow advances automatically
```

Use harmless synthetic work if no real feature is ready.

## Phase 8 — Design Room specialist behavior

Validate optional participation for Unity/Windows/client concerns, including `CLIENT_SIGNOFF`, `APPROVE_WITH_NOTES`, or `BLOCK` if supported.

Design Room participation must not grant implementation authority by itself.

## Non-goals

Do not:
- alter GCP IAM/cloud infrastructure;
- import Google VM credentials;
- store provider/GitHub secrets centrally;
- consume unauthorized project/user work;
- expose local private keys;
- retire inbox before full acceptance.

## Completion Report

Report:
- runtime identity/profile name;
- enrollment method/config files changed without secrets;
- platform connectivity;
- project/task authorization tests;
- capability/access metadata behavior;
- `requires_human` behavior;
- live push/handoff tests;
- offline replay tests;
- cloud-to-local validation;
- Design Room specialist test;
- revocation/re-enrollment result;
- commits/SHAs;
- Human follow-up required.