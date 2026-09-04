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
