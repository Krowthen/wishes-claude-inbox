# Task: End-to-End Acceptance for Wishes Agent Control Plane and Design Room

Created: 2026-09-04
Priority: High after integrations complete
Mode: validation-with-human-cutover-gate
Assigned Agent: claude-google
Execution Environment: Google Cloud Claude Code Operations VM
Allow Edit: true

Depends on:
- `pending/deploy-agent-control-plane-gcp-claude-google.md`
- `pending/integrate-agent-control-plane-claude-local.md`
- `pending/prepare-design-room-participant-integration-claude-google.md`

Reference design:
- `pending/reference-wishes-multi-agent-control-plane-and-design-room-final-design.md`

## Objective

Run the technical portions of the final end-to-end acceptance plan and produce the evidence package needed for ChatGPT/Human review before the Wishes workflow is changed from inbox-first to control-plane-first.

This task does not itself authorize cutover or canonization.

## Acceptance sequence

### A. Agent identity and routing

Verify distinct identities and role boundaries for:

```text
human-owner
chatgpt-director
claude-coop
claude-google
claude-local
```

Verify no ambiguous shared Claude identity is used for task ownership.

### B. ChatGPT/director -> Claude Google task flow

Using an approved synthetic task:

1. task is created/assigned to `claude-google`;
2. Claude Google retrieves it in a fresh session;
3. claims it;
4. posts progress/checkpoint;
5. registers a harmless artifact/commit reference;
6. completes it;
7. result is visible to the directing layer.

### C. Google -> Local handoff

Using a synthetic or safe real development handoff:

1. Google task completes;
2. dependent task assigned to `claude-local` becomes ready;
3. local runtime retrieves the referenced branch/commit/context;
4. local runtime posts validation/result;
5. Google runtime does not consume the local-only task.

### D. Offline local behavior

Coordinate with the Human and verify:

- a local-only task remains durable while local runtime is unavailable;
- assignment is preserved;
- Google runtime does not silently steal it;
- fresh local session recovers the task and checkpoint state after reconnect.

### E. Design Room canonical workflow

Run the first meaningful Design Room acceptance topic using the Agent Control Plane architecture itself or another Human-approved architecture question.

Required workflow evidence:

```text
ChatGPT scope + initial design
 -> claude-google baseline reality review
 -> ChatGPT <-> claude-coop design comparison/challenge
 -> claude-google final implementation review
 -> no unresolved BLOCK
 -> DESIGN_SIGNOFF
 -> IMPLEMENTATION_SIGNOFF
 -> DIRECTOR_SIGNOFF
 -> Human approval if required
 -> explicit Decision
 -> explicit implementation Task
```

Claude Local participates only if the chosen topic includes a material local/Unity/client concern.

### F. Blocking semantics

Prove with synthetic data that:

- `BLOCK` prevents proposal promotion;
- resolving/removing the blocking objection allows progression subject to other required approvals;
- `APPROVE_WITH_NOTES` is retained but does not act as a blocker;
- signoffs remain attributable to the correct role.

### G. Human approval boundary

Prove that a Human-gated action remains pending until the approved Human action occurs.

An ordinary agent message claiming approval must not satisfy the approval state.

### H. Redis durability/recovery

Using a safe test method appropriate to the deployed environment, prove that authoritative task/conversation state can be reconstructed independently of transient Redis state.

Do not disrupt unrelated workloads.

### I. Fresh-session recovery

From fresh sessions where practical, verify `get_project_state` or equivalent returns enough context to discover:

- active tasks;
- blocked work/dependencies;
- pending approvals;
- active Design Rooms;
- recent decisions;
- latest checkpoints;
- repository/branch/commit references.

No previous model context window may be required to reconstruct current operational state.

### J. Traceability

Prove one complete trace:

```text
Requirement
 -> Design Room
 -> Proposal
 -> Decision
 -> Task
 -> Execution
 -> Commit/artifact
 -> Validation
```

## Cutover approval package

Prepare for ChatGPT/Human review:

- acceptance checklist with pass/fail evidence;
- outstanding limitations;
- current operating cost delta;
- security/IAM summary;
- recovery/fallback procedure;
- inbox fallback procedure;
- known product/integration limitations for ChatGPT or Claude Web participation;
- recommendation: READY / READY_WITH_NOTES / NOT_READY.

## Hard gate — workflow cutover

Do not change the canonical operating workflow from inbox-first to control-plane-first until the Human explicitly approves cutover after reviewing acceptance evidence.

Do not delete `wishes-claude-inbox`.

## Post-approval actions permitted by this task

Only after Human cutover approval:

1. update operational documentation to make the Agent Control Plane the normal transport;
2. document the inbox as archive/bootstrap/emergency fallback;
3. leave historical inbox content intact;
4. prepare, but do not mechanically bypass, the existing Wishes canon review/canonization workflow.

## Required completion report

Report:

- test environment/versions;
- each acceptance item pass/fail;
- evidence references;
- unresolved limitations;
- cost/security/recovery summary;
- READY / READY_WITH_NOTES / NOT_READY recommendation;
- Human cutover approval status;
- files/commits changed after any approved cutover;
- exact remaining canon/documentation step.
