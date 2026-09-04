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
