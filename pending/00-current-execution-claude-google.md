# Current Execution Pointer — `claude-google`

Created: 2026-09-04
Priority: Immediate / execute now
Assigned Agent: claude-google
Execution Environment: Google Cloud Claude Code Operations VM
Mode: execution-controller

## Purpose

This file is the current manual execution pointer for `claude-google` until the Agent Control Platform is live.

## Current state

Completed:

1. `completed/bootstrap-claude-google-runtime-identity.md`
2. `completed/agent-control-step-02-security-domain-reconciliation-claude-google.md`

Step 02 proved that the current Operations VM host identity is too narrow for live GCP inventory or management.

The Human has now resolved both open decisions:

- S0 Agent Control placement is approved inside `wishes-506905`, region `us-central1`, while preserving an independent `agent_control` domain/database/IAM boundary.
- `claude-google` is approved for standing normal Wishes GCP management through a dedicated short-lived impersonated management identity.

Authoritative decision:

- `pending/decision-s0-agent-control-placement-and-claude-google-management-authority.md`

## Execute next — do not skip

1. Execute `pending/elevate-claude-google-gcp-management-and-confirm-control-plane-placement.md` immediately.
2. Derive the concrete least-privilege role matrix and authoritative Terraform for the dedicated `claude-google` management identity.
3. If the current host identity cannot perform the bootstrap, return the exact minimal one-time Human command(s) needed. Do not request credential transfer.
4. Stop before the first durable IAM-elevation apply and present the exact role matrix/Terraform plan for Human confirmation.
5. After that confirmation, apply and validate short-lived impersonation.
6. Re-run the live GCP inventory that Step 02 could not perform.
7. Continue to the revised Step 03 placement task using the approved same-project S0 decision.

Queued after management bootstrap:

- `pending/agent-control-step-03-domain-placement-claude-google.md`
- `pending/agent-control-step-04-threat-model-security-claude-google.md`

## Current architecture authority

Use together:

- `pending/reference-agent-control-platform-revised-approved-design.md`
- `pending/agent-control-platform-master-deployment-plan.md`
- `pending/decision-s0-agent-control-placement-and-claude-google-management-authority.md`

Interpret the revised platform design as follows for S0:

- `agent_control` remains an independent multi-user/multi-project platform domain.
- S0 hosting is co-located in GCP project `wishes-506905` by explicit Human approval.
- Do not resurrect the obsolete Wishes-specific `wishes_ops` schema/database model.
- Do not create a separate GCP project solely for S0 Agent Control.
- Preserve independent database/service/IAM boundaries so later extraction to a dedicated project remains possible.

## Credential rule

Do not copy or centralize provider/user credentials. Record capabilities/access claims and credential location only. Actual provider credentials remain local to the runtime/user unless a separately approved server-side integration explicitly requires a managed secret reference.

The GCP management model uses short-lived service-account impersonation; do not create static service-account JSON keys.

## Completion handling

For each completed task:

- append the required completion report;
- move/promote the task according to the existing inbox workflow;
- record branch/commit references;
- do not consume `claude-local` tasks;
- do not cross retained Human destructive/security gates.
