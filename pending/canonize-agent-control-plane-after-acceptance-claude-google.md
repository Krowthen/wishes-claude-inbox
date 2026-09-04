# Task: Reconcile and Canonize the Wishes Agent Control Plane Architecture After Acceptance

Created: 2026-09-04
Priority: Deferred until acceptance
Mode: documentation-with-human-approval-gate
Assigned Agent: claude-google
Execution Environment: Google Cloud Claude Code Operations VM
Allow Edit: true

Depends on:
- successful completion of `pending/accept-agent-control-plane-and-design-room-claude-google.md`
- Human approval to proceed with the existing canon workflow

Reference design:
- `pending/reference-wishes-multi-agent-control-plane-and-design-room-final-design.md`

## Objective

After the Agent Control Plane and Design Room architecture has passed end-to-end acceptance, reconcile the implemented reality into Wishes deployment/operations canon without duplicating or contradicting the existing Claude Operations VM and cloud/local Unity topology documentation.

## Required work

1. Confirm acceptance status is READY or Human-approved READY_WITH_NOTES.
2. Confirm Human has approved proceeding with canon reconciliation/canonization.
3. Review the current authoritative/draft deployment documentation, especially the Claude Code Operations chapter, architecture overview, IAM/data-boundary appendices, workflow/canon rules, and the previously added cloud Claude/local Unity topology material.
4. Review the actual implemented Agent Gateway/MCP/A2A/`wishes_ops`/Redis/Design Room behavior; document implementation reality, not merely the original proposal.
5. Reconcile and update the correct existing documents rather than creating a conflicting parallel deployment chapter unless repository structure clearly requires one.
6. Document canonical identities:

```text
human-owner
chatgpt-director
claude-coop
claude-google
claude-local
openai-director (optional runtime)
```

7. Document the authority split, `wishes_ops` boundary, MCP vs A2A responsibilities, Design Room promotion boundary, inbox fallback role, and the approved ChatGPT/Claude Coop/Claude Google review flow.
8. Preserve all existing production authority/security constraints.
9. Follow `CLAUDE.md` and `WORKFLOW.md` mechanical validation and canonization steps exactly.
10. Do not silently mark a draft approved or bypass a required Human status transition.
11. Update any `wishes-game` canon submodule pointer only at the appropriate post-merge point according to the existing two-repository workflow.

## Required validation

Run all applicable canon validation/publication checks and compare failures against the baseline where the repository has known pre-existing validation issues.

## Required completion report

Report:

- acceptance evidence reviewed;
- canon files reviewed;
- files changed;
- architecture sections/diagrams updated;
- conflicts reconciled;
- validation results;
- branch and commit SHA(s);
- PR/canonization status;
- whether a submodule pointer follow-up remains.
