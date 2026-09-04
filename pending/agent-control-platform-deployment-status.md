# Agent Control Platform — Deployment Status Ledger

Created: 2026-09-04
Owner: chatgpt-director / human-owner
Purpose: durable execution-state ledger during bootstrap before the Agent Control Platform exists.

Authoritative plan:
- `pending/agent-control-platform-master-deployment-plan.md`

Authoritative design:
- `pending/reference-agent-control-platform-revised-approved-design.md`

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
| 01A | Bootstrap `claude-google` identity/routing | claude-google | IN_PROGRESS | Execute `bootstrap-claude-google-runtime-identity.md` |
| 01B | Bootstrap `claude-local` identity/routing | claude-local | IN_PROGRESS | Execute `bootstrap-claude-local-runtime-identity.md` |
| 02 | Security/domain/live-state reconciliation | claude-google | READY | Starts immediately after 01A |
| 03 | Dedicated domain placement recommendation | claude-google | BLOCKED | Requires Step 02 completion |
| 04 | Threat model/security architecture | claude-google + chatgpt-director review | BLOCKED | Requires Step 03 recommendation |
| 05+ | Remaining master deployment plan | assigned per plan | BLOCKED | Advance sequentially from accepted prior-step output |

## Current execution pointers

- `pending/00-current-execution-claude-google.md`
- `pending/00-current-execution-claude-local.md`

## Current hard rules

1. Do not implement the obsolete `wishes_ops` single-project model.
2. The platform domain is `agent_control` and is independent from Wishes application databases.
3. No raw AI/user/GitHub/GCP/Jira/Asana credentials may be written to normal platform tables, task files, chat messages, logs, or Git.
4. Agent profiles express capabilities/access claims and credential location/state, not secret values.
5. Human-interactive authentication is a normal access state and may block work safely.
6. Live AI-to-AI execution uses push delivery with durable replay; no human should need to tell an online agent to check for updates.
7. Cross-workspace/project authorization and auditability are mandatory before multi-user cutover.
8. No cost-bearing/IAM/database apply crosses a Human gate without explicit approval.

## Advancement rule

When a step completes:

1. review its completion report against the approved design/security rules;
2. mark the step `COMPLETE` only after review;
3. record relevant branch/commit/artifact references in this ledger;
4. resolve any `BLOCK` or Human gate;
5. change the next eligible step to `READY` or `IN_PROGRESS`;
6. issue/update the corresponding agent task document;
7. keep this ledger until the deployed control plane takes over durable workflow state.
