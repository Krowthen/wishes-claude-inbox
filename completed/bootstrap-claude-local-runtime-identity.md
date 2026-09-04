# Task: Bootstrap `claude-local` Runtime Identity and Interim Inbox Routing

Created: 2026-09-04
Priority: Immediate
Mode: implementation
Assigned Agent: claude-local
Execution Environment: Local Windows Claude Code workstation
Allow Edit: true

Reference design:
- `pending/reference-wishes-multi-agent-control-plane-and-design-room-final-design.md`

## Objective

Update the local Windows Claude Code environment and repository-level Claude instructions so this runtime is explicitly identified as:

```text
claude-local
```

This is the interim routing mechanism until the Wishes Agent Control Plane is deployed.

## Required work

1. Confirm this task is being executed from the local Windows Claude Code environment. If not, stop and report the environment mismatch.
2. Review the applicable `CLAUDE.md`, `WORKFLOW.md`, repository instructions, local/user-level Claude Code memory/configuration, and inbox sync/consumption workflow before editing.
3. Add a concise, durable runtime-identity section to the appropriate `CLAUDE.md`/environment instruction file(s). Do not duplicate repository architecture rules already defined elsewhere.
4. The environment instructions must establish the canonical identity:

```text
Agent ID: claude-local
Environment: Local Windows development workstation
Primary Role: Unity/Windows/local implementation and validation
Counterpart Runtime: claude-google
```

5. Add interim inbox-routing rules:

```text
- Execute tasks with `Assigned Agent: claude-local`.
- Leave tasks with `Assigned Agent: claude-google` untouched unless the Human explicitly overrides the assignment.
- `Assigned Agent: any` may be executed only when the task is genuinely compatible with this runtime and no conflicting active claim exists.
- Reference-design documents are not executable merely because they are under `pending/`.
- Never infer ownership from filename alone when an `Assigned Agent` field exists.
```

6. Add the Design Room specialist boundary:

```text
claude-local is not a required participant in every Design Room.
Join as the local/client specialist when the design materially affects Unity, Windows behavior, Play Mode, visual/rendering/animation workflows, local ComfyUI, client performance, input, or local developer tooling.
```

7. Add/confirm that this runtime's primary responsibilities include:

- Unity Editor and Play Mode work;
- Windows-specific validation;
- graphical debugging and visual QA;
- local developer tooling;
- client API integration;
- local ComfyUI work where applicable;
- pulling and validating branches produced by `claude-google`;
- returning structured local validation results.

8. Explicitly state that this identity does not inherit the Google Operations VM's cloud permissions and must not assume it can perform GCP/infrastructure mutations assigned to `claude-google`.
9. Do not alter cloud IAM, production permissions, Google service accounts, or deployment authority as part of this identity-only task.
10. If inbox sync tooling needs a minimal update to surface/read the `Assigned Agent` field safely, implement that only if it is clearly within the existing workflow and does not cause tasks for `claude-google` to be consumed/moved. Otherwise document the required follow-up rather than broadening this task.
11. Commit changes on an appropriate focused branch/commit according to repository workflow.

## Required validation

Demonstrate/document the expected routing behavior with three examples:

```text
Assigned Agent: claude-local  -> eligible for this runtime
Assigned Agent: claude-google -> must remain untouched
Assigned Agent: all/reference -> read as reference, not automatically executed
```

Run any repository validation required by `CLAUDE.md`/`WORKFLOW.md`.

## Non-goals

Do not in this task:

- build the Agent Gateway;
- create `wishes_ops`;
- deploy MCP/A2A;
- modify Google Operations VM configuration;
- consume tasks assigned to `claude-google`;
- change production authority;
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
- any follow-up required before full local control-plane integration.

---

## Completion report (2026-09-04)

**Confirmed execution environment:** Local Windows Claude Code workstation
(`C:\dev\wishes\repos\wishes-game`, PowerShell + Git Bash tools, Docker
Desktop/local ComfyUI/Cloudflare tunnel all present) — matches the expected
`claude-local` profile.

**Files reviewed before editing:** `CLAUDE.md` (full, including the
`## Runtime Identity` section `claude-google` had already added on branch
`claude-google/bootstrap-runtime-identity`, PR #1, not yet merged),
`docs/claude/workflows.md`, `docs/claude/external-inbox-repo.md`,
`docs/claude/todo.md`, `docs/claude/session-state.md`, this task's own
reference design (`reference-wishes-multi-agent-control-plane-and-design-
room-final-design.md`) and its supersession
(`reference-agent-control-platform-revised-approved-design.md`), and PR #1's
full diff (`gh pr diff 1`).

**Coordination note:** `claude-google` had already completed its own
counterpart task and, as part of it, wrote a *shared* `## Runtime Identity`
section covering both identities' basics plus the interim routing rules —
but it did not include several things this task specifically requires for
`claude-local` (an explicit "Primary role" line, the non-inheritance-of-
cloud-permissions statement, a `claude-local` Design Room specialist
boundary, a responsibilities list, or worked examples from `claude-local`'s
own perspective). Rather than open a second PR duplicating/conflicting with
PR #1's edits to the same file section, this task built directly on PR #1's
branch tip (`f724c58`) and opened PR #2 targeting PR #1's branch rather
than `main`, so the two PRs combine into one coherent Runtime Identity
section instead of racing each other.

**Files modified:** `CLAUDE.md` only (32 insertions, 1 deletion) — no other
file.

**Exact identity/routing rules added** (extending the existing `## Runtime
Identity` section):
- `claude-local` block: added an explicit "Primary role: Unity/Windows/
  local implementation and validation" line, and a new paragraph stating
  this identity does not inherit the Google Operations VM's cloud
  permissions and must not assume it can perform GCP/Terraform/IAM
  mutations or any action assigned to `claude-google`.
- A `claude-local` Design Room specialist-boundary paragraph, symmetric to
  the existing `claude-google` one: not a required participant in every
  Design Room; joins as local/client specialist when the design materially
  affects Unity, Windows-specific behavior, Play Mode, visual/rendering/
  animation, local ComfyUI, client performance, input/gameplay testing, or
  local developer tooling; may issue `CLIENT_SIGNOFF`, `APPROVE_WITH_NOTES`,
  or `BLOCK` for those concerns; leaves unrelated backend-only rooms
  unclaimed.
- Core responsibilities list for `claude-local`: Unity Editor/Play Mode
  work, Windows-specific validation and graphical debugging/visual QA,
  local developer tooling, client API integration, local ComfyUI work,
  pulling/validating branches produced by `claude-google`, returning
  structured local validation results.
- A second "worked examples" block, evaluated from `claude-local`'s own
  perspective (the existing one in PR #1 was evaluated from
  `claude-google`'s).

The interim inbox-routing rules themselves, the environment-detection
heuristic, and the `claude-google` Design Room boundary were already
present from PR #1 and were not duplicated.

**Validation performed:** doc-only change, no code/build/migration
affected — matches the precedent set by `claude-google`'s equivalent task.
Worked the three required routing examples directly against the rules from
`claude-local`'s perspective and confirmed each resolves as expected:
`Assigned Agent: claude-local` → eligible; `Assigned Agent: claude-google`
→ untouched; `Assigned Agent: all` on a reference/design file → read as
reference, not auto-executed (recorded inline in `CLAUDE.md`).

**Branch and commit:** `local/bootstrap-claude-local-runtime-identity`,
commit `6d9a532` (based directly on PR #1's tip `f724c58`, not on `main`,
to avoid a merge conflict between the two PRs). PR opened:
https://github.com/Krowthen/wishes-game/pull/2, targeting PR #1's branch
(`claude-google/bootstrap-runtime-identity`) rather than `main` — not
merged, left for Human review per repo norms for this shared file.

**Inbox-sync limitation discovered:** none new — `claude-google`'s own
completion report already identified that neither the tracked PowerShell
sync script nor its VM-side bash port parse the `Assigned Agent` field (both
copy every valid `pending/` item regardless of assignment; routing is
enforced only when a runtime reads the field per-task after sync). Separately
in this session (before this task started), the sync script's *other*
long-standing gap — the old "never overwrite existing local items" rule
silently stranding a stale copy when the source changed — was fixed
(content-hash comparison + lifecycle-aware guard against touching anything
that moved on to `active/`/`completed/`/`rejected/`/`archive/`); see
`docs/claude/external-inbox-repo.md` and `docs/claude/todo.md` for that
separate fix. The `Assigned Agent`-parsing gap itself is left as a
documented follow-up per this task's own scope instruction (#10) rather
than broadened into this task.

**Follow-up required before full local control-plane integration:**
1. PR #1 and PR #2 both need Human review/merge (in that order, since #2
   targets #1's branch) before the `## Runtime Identity` section is on
   `main`.
2. Decide whether the sync script should filter/tag by `Assigned Agent`
   (still open, per both bootstrap tasks' completion reports).
3. `integrate-agent-control-plane-claude-local.md` remains blocked — depends
   on this task (now done, pending merge) plus `claude-google`'s GCP
   deployment task, which has not started.
4. Per `00-current-execution-claude-local.md`'s own instruction: "After
   completing the identity/bootstrap task, stop." Not proceeding to
   `integrate-agent-control-plane-claude-local.md` or any other platform
   work without further Human direction.

**Merge update (2026-09-04):** PR #1 merged to `main` (`5a7a2b0`). PR #2 was
auto-closed by GitHub when PR #1's head branch was deleted (a closed PR
can't be retargeted) — recovered by merging `main` into the same commit and
opening PR #4 targeting `main` directly, merged (`22f32d6`). Both identity
blocks, both Design Room boundaries, both responsibilities lists now live
on `main`'s `CLAUDE.md`.
