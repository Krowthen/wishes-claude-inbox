# Current Execution Pointer — `claude-local`

Created: 2026-09-04
Priority: Immediate / execute now
Assigned Agent: claude-local
Execution Environment: Local Windows Claude Code workstation
Mode: execution-controller

## Purpose

This file is the current manual execution pointer for `claude-local` until the Agent Control Platform is live.

Execute only:

1. `pending/bootstrap-claude-local-runtime-identity.md`

After completing the identity/bootstrap task, stop. Do not begin the full control-plane integration until the shared Agent Gateway has been deployed and the dependent task becomes valid:

- `pending/integrate-agent-control-plane-claude-local.md`

## Current architecture authority

Use the superseding reference:

- `pending/reference-agent-control-platform-revised-approved-design.md`
- `pending/agent-control-platform-master-deployment-plan.md`

## Credential rule

Do not copy credentials from `claude-google` or any other runtime. Local provider/repository/cloud credentials remain local to this runtime/user unless a later approved integration defines a different secure provisioning mechanism.

## Routing rule

- Execute `Assigned Agent: claude-local` work only.
- Leave `Assigned Agent: claude-google` work untouched unless the Human explicitly overrides it.
- Do not infer ownership from filenames when an explicit assignment exists.

## Completion handling

Append the required completion report, follow the existing inbox completion workflow, and then wait for the shared gateway deployment before further platform work.
