# Task: Prepare ChatGPT and Claude Coop Design Room Integration

Created: 2026-09-04
Priority: Medium after gateway deployment
Mode: implementation-and-integration-preparation
Assigned Agent: claude-google
Execution Environment: Google Cloud Claude Code Operations VM
Allow Edit: true

Depends on:
- `pending/deploy-agent-control-plane-gcp-claude-google.md`

Reference design:
- `pending/reference-wishes-multi-agent-control-plane-and-design-room-final-design.md`

## Objective

Prepare the deployed Wishes Agent Control Plane for the approved Design Room participants:

```text
chatgpt-director -> lead architect/director
claude-coop      -> co-designer/challenger
claude-google    -> implementation/reality reviewer
claude-local     -> optional local/client specialist
openai-director  -> optional API-backed director runtime
```

This task is limited to server-side integration support, participant metadata, documentation, and synthetic validation. Any account-side connection step remains a Human action.

## Required work

1. Confirm the deployed Agent Gateway and Design Room interfaces are healthy.
2. Prepare stable participant/agent metadata describing role and capabilities.
3. Ensure `claude-coop` is a Design Room participant and is not a default code-execution agent.
4. Ensure `chatgpt-director` is the default Design Room facilitator/lead role.
5. Keep `openai-director` optional and inactive until separately configured; interactive ChatGPT-led Design Rooms must not depend on it.
6. Document the account-side connection steps needed for ChatGPT and Claude Web/Coop without placing private account material in the repository.
7. If direct Claude Web/Coop participation is not supported by the available integration path, document the required bridge/manual/API adapter honestly rather than treating it as already connected.
8. Ensure Design Room messages cannot directly authorize implementation.
9. Ensure Proposal -> Decision -> Task remains the explicit promotion path.

## Canonical role workflow

```text
ChatGPT scope + initial design
 -> claude-google baseline repo/code/GCP validation
 -> ChatGPT <-> claude-coop iterative design comparison
 -> claude-google final implementation validation
 -> DESIGN_SIGNOFF / IMPLEMENTATION_SIGNOFF / DIRECTOR_SIGNOFF
 -> Human approval if required
 -> Decision -> Tasks
```

Claude Google is the reality/review anchor, not the default conversational co-designer.

## Synthetic validation

Validate, using test participants where needed:

- director creates a room;
- baseline review is requested from `claude-google`;
- `claude-coop` can post a challenge/alternative;
- final Google review can be requested;
- `BLOCK` prevents promotion;
- signoffs remain distinct by role;
- Human-required approval remains pending until Human action;
- proposal promotion uses the explicit authorized path.

## Non-goals

Do not:

- change production authority;
- force Claude Local into every room;
- turn Claude Coop into an implementation agent;
- add unrestricted shell/production execution to Design Rooms;
- retire the inbox;
- canonize before acceptance.

## Required completion report

Report:

- participant metadata/configuration implemented;
- files/docs created;
- Human-side setup steps still required;
- any current product/integration limitation affecting direct ChatGPT or Claude Web participation;
- synthetic Design Room test results;
- branch/commit SHA(s);
- readiness for full end-to-end acceptance.
