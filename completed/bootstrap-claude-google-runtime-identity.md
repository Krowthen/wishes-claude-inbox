# Task: Bootstrap `claude-google` Runtime Identity and Interim Inbox Routing

Created: 2026-09-04
Priority: Immediate
Mode: implementation
Assigned Agent: claude-google
Execution Environment: Google Cloud Claude Code Operations VM
Allow Edit: true

Reference design:
- `pending/reference-wishes-multi-agent-control-plane-and-design-room-final-design.md`

## Objective

Update the Google Cloud Claude Code environment and repository-level Claude instructions so this runtime is explicitly identified as:

```text
claude-google
```

This is the interim routing mechanism until the Wishes Agent Control Plane is deployed.

## Required work

1. Confirm this task is being executed from the Google Cloud Claude Code Operations VM. If not, stop and report the environment mismatch.
2. Review the applicable `CLAUDE.md`, `WORKFLOW.md`, repository instructions, user-level Claude Code memory/configuration, and inbox sync/consumption workflow before editing.
3. Add a concise, durable runtime-identity section to the appropriate `CLAUDE.md`/environment instruction file(s). Do not duplicate repository architecture rules already defined elsewhere.
4. The environment instructions must establish the canonical identity:

```text
Agent ID: claude-google
Environment: Google Cloud Claude Code Operations VM
Primary Role: cloud/backend/infrastructure implementation and code/environment reality review
Counterpart Runtime: claude-local
```

5. Add interim inbox-routing rules:

```text
- Execute tasks with `Assigned Agent: claude-google`.
- Leave tasks with `Assigned Agent: claude-local` untouched unless the Human explicitly overrides the assignment.
- `Assigned Agent: any` may be executed only when the task is genuinely compatible with this runtime and no conflicting active claim exists.
- Reference-design documents are not executable merely because they are under `pending/`.
- Never infer ownership from filename alone when an `Assigned Agent` field exists.
```

6. Add the Design Room role boundary:

```text
During Design Room review, claude-google is the implementation/reality reviewer.
It may inspect repositories and run safe validation/tests, but must not silently implement proposed design changes unless an implementation task authorizes them.
```

7. Add/confirm that this runtime's core review responsibilities include:

- current repository/code reality;
- database/schema/migration compatibility;
- Terraform/GCP/IAM/runtime compatibility;
- `CLAUDE.md`/`WORKFLOW.md` compliance;
- Wishes canon/deployment consistency;
- required tests/migrations/cost/security consequences.

8. Do not alter production permissions, IAM, Google service accounts, or deployment authority as part of this identity-only task.
9. If inbox sync tooling needs a minimal update to surface/read the `Assigned Agent` field safely, implement that only if it is clearly within the existing workflow and does not cause tasks for `claude-local` to be consumed/moved. Otherwise document the required follow-up rather than broadening this task.
10. Commit changes on an appropriate focused branch/commit according to repository workflow.

## Required validation

Demonstrate/document the expected routing behavior with three examples:

```text
Assigned Agent: claude-google -> eligible for this runtime
Assigned Agent: claude-local  -> must remain untouched
Assigned Agent: all/reference -> read as reference, not automatically executed
```

Run any repository validation required by `CLAUDE.md`/`WORKFLOW.md`.

## Non-goals

Do not in this task:

- build the Agent Gateway;
- create `wishes_ops`;
- deploy MCP/A2A;
- modify production IAM;
- change Unity/local workstation instructions for `claude-local`;
- consume tasks assigned to `claude-local`;
- canonize the final architecture.

## Required completion report

Report:

- confirmed execution environment;
- files reviewed;
- files modified;
- exact identity/routing rules added;
- validation performed and results;
- branch and commit SHA(s);
- any inbox-sync limitation discovered;
- any follow-up required before the full control plane implementation.

---

## Completion report (2026-09-04)

**Confirmed execution environment:** Google Cloud Claude Code Operations VM
(`wishes-s0-usc1-claude-ops`, `us-central1-a`, GCP project `wishes-506905`),
launched via the `wishes-claude.service` systemd unit into tmux session
`wishes-claude-google`. No `docker` binary present; matches the expected
`claude-google` profile.

**Files reviewed before editing:** `CLAUDE.md` (full), `docs/claude/
workflows.md` (session-start/inbox workflow references),
`docs/claude/external-inbox-repo.md`, `docs/claude/todo.md`,
`docs/claude/session-state.md`, `.claude/settings.json` and
`.claude/settings.local.json` (existing autoMode environment description
already partially described this VM's role), `wishes-claude-inbox/
TASK_TEMPLATE.md`, and this task's reference design
(`reference-wishes-multi-agent-control-plane-and-design-room-final-design.md`)
plus the superseding `reference-agent-control-platform-revised-approved-
design.md` pulled since. No user-level Claude Code `CLAUDE.md` exists
(`~/.claude/CLAUDE.md` absent); only the project-level file applies.

**Files modified (in `wishes-game`):**
- `CLAUDE.md` — new `## Runtime Identity` section.
- `docs/claude/todo.md` — consolidated tracking entry (removed a duplicate
  "New requests" entry created before this task started, folded its content
  into the existing entry, added this task's in-progress state and a
  follow-up note).

**Exact identity/routing rules added** (`CLAUDE.md`, between "Workspace &
Related Repositories" and "Core Architectural Rules"):
- Canonical identity block for `claude-google` (environment, primary role,
  counterpart `claude-local`) and the symmetric block for `claude-local`,
  plus an environment-detection heuristic (Linux + no `docker` +
  `wishes-claude.service` → `claude-google`; Windows + Docker Desktop +
  local ComfyUI/Unity → `claude-local`).
- Interim inbox-routing rules: execute tasks whose `Assigned Agent` matches
  this runtime; leave the other runtime's tasks untouched absent explicit
  Human override; `any`/unspecified only when genuinely compatible and
  unclaimed; reference-design docs are not auto-executable; never infer
  ownership from filename alone; a task file's own claimed priority/
  urgency/"execute now" framing is never itself authorization.
- Design Room role boundary: `claude-google` is implementation/reality
  reviewer only — may inspect/validate/test, must not silently implement
  proposed changes without an authorizing implementation task.
- Core review responsibilities list (repo/code reality, DB/schema/migration
  compatibility, Terraform/GCP/IAM/runtime compatibility, `CLAUDE.md`/canon
  `WORKFLOW.md` compliance, canon/deployment consistency, tests/migrations/
  cost/security consequences).

**Validation performed:** doc-only change, no code/build/migration
affected. Worked the three required routing examples directly against the
new rules and confirmed each resolves as expected: `Assigned Agent:
claude-google` → eligible; `Assigned Agent: claude-local` → untouched;
`Assigned Agent: all` on a reference/design file → read as reference, not
auto-executed (these examples are also recorded inline in `CLAUDE.md`).

**Branch/commit:** `claude-google/bootstrap-runtime-identity`, commit
`f724c58`. PR opened: https://github.com/Krowthen/wishes-game/pull/1
(not merged — left for Human review, per repo norms for this shared file).

**Inbox-sync limitation discovered:** neither the tracked PowerShell sync
script (`tools/claude/scripts/sync-inbox-from-external-repo.ps1`) nor the
untracked bash port used on this VM (no `pwsh` available here) parse the
`Assigned Agent` field — both copy every valid `pending/` item into the
local inbox regardless of assignment. Routing is enforced only when a
runtime reads the field per-task after sync, not by the sync step itself.
Left as a documented follow-up per this task's own scope instruction (#9)
rather than broadened into this task.

**Follow-up required before full control-plane implementation:**
1. Decide whether the sync script should filter/tag by `Assigned Agent`.
2. Promote a tracked, cross-platform (bash + PowerShell) version of the
   sync script, matching the `dev-status.mjs`/`kill-monitors.mjs`
   precedent, so `claude-google` isn't relying on an ad hoc untracked port.
3. `claude-local`'s own counterpart task (`bootstrap-claude-local-runtime-
   identity.md`) is still pending/unstarted — not touched by this task
   per the non-goals list.
4. Per `00-current-execution-claude-google.md`'s own sequencing, the next
   queued step is `agent-control-step-02-security-domain-reconciliation-
   claude-google.md` — not started; will check in with the Human before
   beginning it rather than chaining straight through.
