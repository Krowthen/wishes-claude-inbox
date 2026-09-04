# Current Execution Pointer — `claude-google`

Created: 2026-09-04
Priority: Immediate / execute now
Assigned Agent: claude-google
Execution Environment: Google Cloud Claude Code Operations VM
Mode: execution-controller

## Purpose

This file is the current manual execution pointer for `claude-google` until the Agent Control Platform is live.

Execute the following sequence in order and do not skip ahead:

1. `pending/bootstrap-claude-google-runtime-identity.md`
2. `pending/agent-control-step-02-security-domain-reconciliation-claude-google.md`
3. Stop and publish the Step 02 completion report before beginning Step 03 unless Step 02 explicitly concludes there is no Human/design decision required.

After Step 02 is complete, the next queued task is:

- `pending/agent-control-step-03-domain-placement-claude-google.md`

Step 04 is already queued behind Step 03:

- `pending/agent-control-step-04-threat-model-security-claude-google.md`

## Current architecture authority

Use the superseding reference:

- `pending/reference-agent-control-platform-revised-approved-design.md`
- `pending/agent-control-platform-master-deployment-plan.md`

Do not implement the obsolete `wishes_ops` single-project model. The target is the separate, multi-user, multi-project `agent_control` platform domain.

## Credential rule

Do not copy or centralize provider/user credentials during these steps. Record capabilities/access claims and credential location only. Actual credentials remain local to the runtime/user unless a separately approved server-side integration explicitly requires a managed secret reference.

## Completion handling

For each completed task:

- append the required completion report;
- move/promote the task according to the existing inbox workflow;
- record branch/commit references;
- do not consume `claude-local` tasks;
- do not cross Human approval gates.
