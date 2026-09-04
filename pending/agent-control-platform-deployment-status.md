# Agent Control Platform — Deployment Status Ledger

Created: 2026-09-04
Owner: chatgpt-director / human-owner
Purpose: durable execution-state ledger during bootstrap before the Agent Control Platform exists.

Authoritative plan:
- `pending/agent-control-platform-master-deployment-plan.md`

Authoritative design:
- `pending/reference-agent-control-platform-revised-approved-design.md`

Current Human decision override:
- `pending/decision-s0-agent-control-placement-and-claude-google-management-authority.md`

## Status legend

- `READY` — may begin when assigned runtime is available.
- `IN_PROGRESS` — currently assigned/started.
- `BLOCKED` — waiting on dependency or approval.
- `HUMAN_GATE` — cannot advance without Human approval.
- `COMPLETE` — completion report accepted.
- `FUTURE` — intentionally not started yet.

## Current execution

| Step | Work | Owner | Status | Dependency / next action |
|---|---|---|---|---|
| 01A | Bootstrap `claude-google` identity/routing | claude-google | COMPLETE | Completed and promoted to `completed/` |
| 01B | Bootstrap `claude-local` identity/routing | claude-local | COMPLETE | Completed and promoted to `completed/` |
| 02 | Security/domain/live-state reconciliation | claude-google | COMPLETE | Report confirmed live GCP access is blocked by narrow host IAM |
| 02B | Elevate `claude-google` through dedicated impersonated GCP management identity | claude-google | IN_PROGRESS | Execute `elevate-claude-google-gcp-management-and-confirm-control-plane-placement.md`; derive role matrix; if bootstrap is blocked, return exact minimal Human command(s); stop before first durable IAM apply for concrete matrix confirmation |
| 03 | Agent Control domain placement inside `wishes-506905` | claude-google | BLOCKED | Project placement already approved; begin after 02B provides live inventory/management access |
| 04 | Threat model/security architecture | claude-google + chatgpt-director review | BLOCKED | Requires Step 03 recommendation |
| 05+ | Remaining master deployment plan | assigned per plan | BLOCKED | Advance sequentially from accepted prior-step output |

## Human decisions now locked

### S0 GCP placement

```text
project: wishes-506905
region: us-central1
```

The independent `agent_control` platform remains a separate bounded domain/database/IAM/service identity even though it is co-located in the Wishes project for S0.

Do not create a separate GCP project solely for Agent Control at this stage.

### `claude-google` authority

The Human has approved standing normal Wishes GCP management for `claude-google` through a dedicated short-lived impersonated management service account.

The Operations VM host service account must remain weak. No static service-account keys.

Routine resource management may proceed after bootstrap within the approved role set. Destructive/security-critical actions remain Human-gated.

## Current execution pointers

- `pending/00-current-execution-claude-google.md`
- `pending/00-current-execution-claude-local.md`

## Current hard rules

1. Do not implement the obsolete Wishes-specific `wishes_ops` model.
2. The platform domain/database is `agent_control` and is independent from Wishes application databases.
3. For S0, `agent_control` is hosted in `wishes-506905` by explicit Human approval while maintaining independent boundaries and future extractability.
4. No raw AI/user/GitHub/GCP/Jira/Asana credentials may be written to normal platform tables, task files, chat messages, logs, or Git.
5. Agent profiles express capabilities/access claims and credential location/state, not secret values.
6. Human-interactive authentication is a normal access state and may block work safely.
7. Live AI-to-AI execution uses push delivery with durable replay; no human should need to tell an online agent to check for updates.
8. Cross-workspace/project authorization and auditability are mandatory before multi-user cutover.
9. The `claude-google` host identity remains weak; broad cloud management uses reviewed short-lived impersonation.
10. No destructive/security-critical/IAM-boundary action crosses a retained Human gate without explicit approval.

## Advancement rule

When a step completes:

1. review its completion report against the approved design/security rules;
2. mark the step `COMPLETE` only after review;
3. record relevant branch/commit/artifact references in this ledger;
4. resolve any `BLOCK` or Human gate;
5. change the next eligible step to `READY` or `IN_PROGRESS`;
6. issue/update the corresponding agent task document;
7. keep this ledger until the deployed control plane takes over durable workflow state.
