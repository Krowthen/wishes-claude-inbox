# Task: Integrate `claude-local` with the Wishes Agent Control Plane

Created: 2026-09-04
Priority: High after gateway deployment
Mode: implementation-and-validation
Assigned Agent: claude-local
Execution Environment: Local Windows Claude Code workstation
Allow Edit: true

Depends on:
- `pending/bootstrap-claude-local-runtime-identity.md`
- successful completion of `pending/deploy-agent-control-plane-gcp-claude-google.md`

Reference design:
- `pending/reference-wishes-multi-agent-control-plane-and-design-room-final-design.md`

## Objective

Connect the local Windows Claude Code runtime to the deployed Wishes Agent Control Plane as the canonical agent:

```text
claude-local
```

Validate task routing, checkpoint/recovery behavior, local/Unity specialist boundaries, and offline/reconnect behavior without inheriting `claude-google` cloud privileges.

## Phase 1 — Verify prerequisites

1. Confirm runtime is the local Windows Claude Code environment and identity instructions already name it `claude-local`.
2. Review current `CLAUDE.md`, `WORKFLOW.md`, local user-level Claude Code config/memory, and any inbox sync tooling.
3. Obtain the approved control-plane endpoint/configuration from the Google deployment completion report.
4. Do not copy secrets from another runtime. Use only the approved Human/local credential provisioning path.
5. Verify the local machine does not inherit or depend on the Google Operations VM service account.

## Phase 2 — Configure `wishes-control`

Configure the local Claude Code environment to use the deployed `wishes-control` MCP endpoint and any approved A2A/Design Room bridge/runtime configuration needed for local specialist participation.

Requirements:

- canonical agent identity is `claude-local`;
- credentials/config are stored according to approved local security practice;
- no secret is committed to Git;
- no permission-bypass mode is introduced;
- local environment can retrieve project state and its assigned tasks;
- local environment cannot claim a task exclusively assigned to `claude-google`.

## Phase 3 — Local task workflow validation

Create/use approved synthetic test tasks to prove:

```text
1. fresh claude-local session retrieves project state
2. lists tasks assigned to claude-local
3. claims an eligible local task
4. posts progress
5. creates a checkpoint
6. registers a branch/commit/test artifact reference
7. completes the task
8. sees dependency completion from claude-google work
9. cannot silently claim a claude-google-only task
```

## Phase 4 — Unity/Windows specialist boundary

Validate/document how the runtime is invited to Design Rooms only when client/local expertise is required.

Expected specialist areas:

- Unity Editor;
- Play Mode;
- Windows-specific behavior;
- graphical debugging;
- rendering/animation/visual QA;
- input/gameplay testing;
- client performance;
- local ComfyUI where relevant;
- local developer tooling.

Demonstrate a synthetic Design Room interaction in which `claude-local` can:

- read the room context;
- post a client/local risk or validation comment;
- issue `CLIENT_SIGNOFF`, `APPROVE_WITH_NOTES`, or `BLOCK` where the protocol supports it;
- leave unrelated backend-only Design Rooms unclaimed/unmodified.

## Phase 5 — Cloud-created branch validation

Prove the intended handoff flow:

```text
claude-google completes backend/cloud task
 -> checkpoint/commit recorded
 -> dependent task becomes ready for claude-local
 -> claude-local pulls the referenced branch/commit
 -> performs Windows/Unity validation as applicable
 -> returns structured result through the Control Plane
```

Use harmless/synthetic validation if no real Unity change is ready at the time of setup.

## Phase 6 — Offline/reconnect test

Validate:

1. leave/create a task assigned to `claude-local` while the local runtime is offline;
2. verify the task remains durable and assigned;
3. verify `claude-google` does not silently steal it;
4. reconnect/start a fresh local Claude Code session;
5. retrieve the queued task and current project state;
6. continue without needing the prior local chat/session memory.

Coordinate the offline portion with the Human; do not power off or disrupt the workstation unexpectedly.

## Non-goals

Do not in this task:

- alter GCP IAM;
- deploy/modify Cloud Run or Cloud SQL;
- assume Google VM credentials;
- implement backend gateway functionality already owned by `claude-google`;
- consume tasks assigned to `claude-google`;
- make local specialist participation mandatory for every Design Room;
- retire the Claude inbox before overall acceptance.

## Required completion report

Report:

- confirmed local runtime identity;
- files/configuration changed (without secrets);
- control-plane connectivity result;
- MCP/A2A capabilities available to local runtime;
- task routing test results;
- Design Room specialist test results;
- cloud-to-local branch handoff test results;
- offline/reconnect test results;
- any Windows/Unity-specific limitation;
- branch/commit SHA(s) for repository changes;
- any Human follow-up required.
