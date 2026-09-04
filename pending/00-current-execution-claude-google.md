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

The Human has now completed the GCP bootstrap needed to resolve the Step 02 live-inventory blocker.

Approved S0 placement:

```text
project: wishes-506905
region: us-central1
domain: agent_control
```

`agent_control` remains an independent bounded platform domain/database/service/IAM boundary even though it is co-located in the Wishes GCP project for S0.

Approved management identity:

```text
wishes-claude-google-admin@wishes-506905.iam.gserviceaccount.com
```

Weak Operations VM host identity:

```text
wishes-s0-claude-ops-host@wishes-506905.iam.gserviceaccount.com
```

The host identity may impersonate the management identity with short-lived credentials. No static service-account keys.

## Execute next — do not skip

Execute this task immediately:

```text
pending/claude-google-resume-live-gcp-and-agent-control.md
```

That task supersedes the manual copy/paste handoff for the current GCP bootstrap stage.

It requires you to:

- validate VM-side management-identity impersonation;
- re-run complete live GCP inventory;
- verify bucket and runtime-service-account resource-scoped access;
- reconcile Terraform and live state before apply;
- evaluate Cloud Build only if actually required;
- document any exact remaining permission gaps;
- update `claude-google` operating instructions/wrappers for explicit impersonation;
- resume Agent Control Platform deployment after reconciliation.

Do not repeat the already-completed Human bootstrap unless verification proves a specific binding is missing.

## Queued after live GCP reconciliation

Continue the approved Agent Control sequence using:

```text
pending/agent-control-step-03-domain-placement-claude-google.md
pending/agent-control-step-04-threat-model-security-claude-google.md
pending/agent-control-platform-master-deployment-plan.md
```

Step 03 must use the already-approved same-project S0 decision rather than reopening whether a separate GCP project should be created.

## Current architecture authority

Use together:

```text
pending/reference-agent-control-platform-revised-approved-design.md
pending/agent-control-platform-master-deployment-plan.md
pending/decision-s0-agent-control-placement-and-claude-google-management-authority.md
pending/claude-google-resume-live-gcp-and-agent-control.md
```

Interpretation for S0:

- `agent_control` remains independent from Wishes application databases;
- host in `wishes-506905` by Human approval;
- do not revive `wishes_ops`;
- do not create a separate GCP project solely for S0 Agent Control;
- preserve independent database/service/IAM boundaries and future extractability.

## Credential / security rule

Do not copy or centralize provider/user credentials. GCP privileged operations use short-lived service-account impersonation. Do not create static service-account JSON keys.

Do not independently grant Owner/Editor, Project IAM Admin, Service Account Key Admin, blanket Secret Accessor, or broader organization/folder authority.

Do not cross retained Human destructive/security gates.

## Completion handling

For completed work:

- append the required completion report;
- move/promote completed tasks according to inbox workflow;
- record branch/commit references;
- do not consume `claude-local` tasks;
- advance the Agent Control deployment only after live GCP/Terraform reconciliation is complete.