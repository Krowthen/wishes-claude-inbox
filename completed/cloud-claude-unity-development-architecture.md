# Work Request — Canonize Cloud Claude + Unity Development Topology

## Repository targets

Primary documentation repository:
- `Krowthen/wishes-canon`

Related implementation repository for reference only:
- `Krowthen/wishes-game`

Task intake repository:
- `Krowthen/wishes-claude-inbox`

## Objective

Update Wishes deployment/operations canon so the development topology clearly separates:

1. the Linux Claude Code Operations VM used for cloud development/orchestration;
2. the local Windows workstation used for interactive Unity development and visual validation;
3. a future dedicated Windows Unity build/test agent for unattended Windows-specific validation while the local workstation is powered off;
4. a future macOS build host required for final iOS/Xcode build and signing workflows.

This architecture must allow Claude Code to continue useful Wishes development from the cloud even when the user's personal Windows workstation is off, without pretending that the Linux Operations VM is an adequate substitute for an interactive graphical Unity workstation.

## Existing architecture to review first

Review the current deployment/operations documentation before editing, especially:

- `drafts/deployment/10-claude-code-operations.md`
- related deployment chapter indexes and architecture diagrams
- any Unity/client build, CI/CD, release, remote-development, or workstation guidance
- `CLAUDE.md`
- `WORKFLOW.md`

Merge these rules into the existing architecture rather than creating duplicate or conflicting guidance.

## Canonical decisions

### 1. Claude Code Operations VM role

The Claude Code Operations VM is the persistent cloud engineering and orchestration workstation.

Its primary responsibilities include:

- source-code authoring and review;
- C# development for the Unity project;
- server, database, Terraform, container, and cloud development;
- Git operations and repository coordination;
- automated test execution where supported;
- headless or batch-mode Unity validation where practical;
- Unity compilation/test automation where the Linux Unity Editor/toolchain supports it;
- build orchestration;
- CI-like checks;
- durable `tmux`/remote Claude Code sessions;
- remote/mobile continuation of development when the user's local computer is powered off.

The Operations VM is NOT the authoritative graphical Unity test workstation.

It must not be documented as requiring a GPU solely to support ordinary Claude development.

### 2. Linux Unity capabilities

Unity development from the Linux Operations VM is supported for non-visual and automated work where compatible.

Expected supported use cases include:

- editing Unity C# source;
- inspecting Unity project configuration and assets;
- running command-line or batch-mode Unity operations where supported;
- running Unity Test Framework tests where suitable;
- compilation validation;
- headless build/test workflows;
- CI-style project checks.

The exact installed Unity version/tooling must remain aligned with the Wishes Unity project version.

### 3. Linux VM limitations

The Linux Operations VM must not be treated as the primary environment for:

- interactive Unity Editor use;
- Play Mode visual inspection;
- graphical debugging requiring a desktop workflow;
- gameplay feel/input validation;
- animation review;
- visual shader/material validation;
- graphical rendering acceptance testing;
- Windows-specific runtime behavior;
- final user-facing visual QA.

A graphical Linux Unity environment may be introduced later if justified, but it is not required for the initial Operations VM architecture.

### 4. Local Windows workstation role

The user's Windows development workstation remains the primary interactive Unity workstation during the initial development phases.

Its responsibilities include:

- Unity Editor GUI use;
- Play Mode;
- visual inspection and acceptance testing;
- graphical debugging;
- gameplay/input testing;
- Windows-client validation;
- animation, rendering, shader, and presentation review;
- developer-facing Unity tooling that requires an interactive desktop.

Cloud Claude Code and local Unity development coordinate through Git branches, commits, pull requests, and repository state rather than assuming one shared live terminal process.

### 5. Future Windows Unity build/test agent

Define a future dedicated Windows build/test agent as a separate execution role.

Purpose:

- allow Claude/cloud development to submit Windows-specific Unity builds and tests while the user's personal workstation is powered off;
- provide reproducible automated Windows validation;
- avoid turning the general Claude Operations VM into a graphical Windows workstation;
- provide a controlled environment for platform-specific build tooling.

Potential responsibilities include:

- Windows Unity Editor command-line builds;
- Unity Test Framework execution on Windows;
- Windows player builds;
- smoke tests where automation is practical;
- artifact generation;
- build logs and test-result return to Claude/CI;
- later automated graphical or integration testing where justified.

This agent is a future architecture component and is not required for the current S0 Claude VM setup.

Do not prematurely provision it as part of this documentation-only task.

### 6. Future macOS build host

Document a future macOS build host for Apple-platform release requirements.

Its purpose includes:

- Xcode-based iOS builds;
- Apple signing/provisioning workflows;
- final iOS packaging;
- macOS/iOS platform-specific validation as required.

The macOS host is not part of the current S0 cloud VM rollout unless an existing approved roadmap says otherwise.

### 7. Development flow

Document the intended high-level workflow as:

```text
Claude Code Operations VM (Linux)
    -> author / analyze / automated test
    -> commit / push
    -> CI or platform-specific validation

Windows Unity Workstation
    -> pull approved/current work
    -> interactive Unity Editor / Play Mode / visual QA
    -> commit feedback/fixes as appropriate

Future Windows Build/Test Agent
    -> receive build/test request
    -> run Windows-specific Unity validation
    -> publish artifacts/results
    -> Claude reviews results and iterates

Future macOS Build Host
    -> Apple-specific build/sign/test
    -> publish release artifacts/results
```

The local workstation may be completely powered off while Claude performs work that does not require interactive Unity visual validation.

### 8. Claude process separation

Document that local Claude Code and cloud Claude Code sessions are separate processes.

They do not automatically share:

- terminal state;
- live task context;
- in-memory agent state;
- running shell sessions.

Coordination occurs through durable project mechanisms such as:

- Git commits/branches;
- pull requests;
- repository task files;
- Claude inbox work requests;
- documented status/checkpoint files where appropriate.

Do not imply that a local Claude terminal can natively watch or control a cloud Claude terminal without an explicit coordination mechanism.

### 9. Source of truth and recoverability

Reinforce the existing rule that Git is the durable engineering record.

No unique source change should exist only on:

- the Linux Operations VM;
- the Windows workstation;
- a future Windows build agent;
- a macOS build host.

Build/test agents are replaceable execution environments.

### 10. Security and authority remain unchanged

This topology does not expand Claude's production authority.

Preserve the existing requirements that:

- the Operations VM has no public external IP;
- access uses IAP/OS Login as established;
- the attached VM service account remains intentionally weak;
- no static broad service-account keys are introduced;
- Claude writes/tests/prepares changes;
- normal production deployment authority remains human/protected-workflow controlled;
- destructive or production mutations require the established approval path.

## Claude inbox workflow rule

Also update the relevant Wishes operations/workflow documentation so future external work requests intended for Claude Code use the `Krowthen/wishes-claude-inbox` repository rather than relying on manual copy/paste into a Claude terminal.

Canonical operational convention:

```text
External requester / ChatGPT
    -> create a consolidated Markdown work request
    -> commit it under wishes-claude-inbox/pending/
    -> push/commit to the inbox repository
    -> Claude Code consumes/processes the pending task
    -> resulting implementation/canon changes are committed to their authoritative repositories
```

The inbox is an instruction/task transport only. It is not authoritative canon and is not merged into `wishes-game`.

Future work requests should be self-contained and consolidated into a single task document when practical, with referenced assets placed in an allowed task bundle when needed.

Do not store secrets, credentials, executable payloads, or generated output artifacts in inbound inbox task files.

## Documentation lifecycle

Follow the repository's existing canon workflow and controls.

The user has explicitly requested that this architecture be incorporated into Wishes canon. Do not bypass mechanical repository safeguards. If the relevant deployment chapter is still in draft status and canonization requires a repository-specific approval/status transition, update the correct draft and clearly report the remaining canonization step rather than silently bypassing the workflow.

If an existing authoritative canon document already covers these areas, reconcile/update it instead of duplicating the material.

Run the appropriate validation/build tooling required by `wishes-canon/CLAUDE.md` and `WORKFLOW.md` for any canon changes.

## Non-goals

Do not in this task:

- provision a Windows VM;
- provision a macOS host;
- install Unity on production infrastructure;
- convert the Claude Operations VM into a public graphical workstation;
- introduce production mutation authority for Claude;
- modify unrelated game lore;
- change current asset-service/GCS implementation work;
- retire the local Windows Unity workflow.

## Required final report

Report:

- files reviewed;
- files modified/created;
- diagrams or architecture sections updated;
- whether the architecture was added directly to authoritative canon or remains in the required draft/review stage;
- any conflicts found with existing deployment canon;
- validation/build commands run and results;
- commit SHA(s);
- whether the canon submodule pointer in `wishes-game` requires a follow-up bump.

Use focused commit messages appropriate to the repository. A suggested canon commit is:

`docs(canon): define cloud Claude and Unity development topology`

Do not merge unrelated work into this task.

---

## Completion report (2026-09-02)

- **Files reviewed**: `docs/canon/drafts/deployment/10-claude-code-operations.md` (primary target), `docs/canon/drafts/deployment/00-architecture-overview.md` (checked for overlap — none found; only a brief §13 VM-role mention, no Unity/Windows content), `docs/canon/CLAUDE.md`, `docs/canon/WORKFLOW.md`, `wishes-canon` repo layout (`canon/` vs `drafts/`).
- **Files modified**: `docs/canon/drafts/deployment/10-claude-code-operations.md` only — no other file touched, no unrelated work merged in.
- **Sections added/updated**: new §39 "Development topology: cloud Operations VM, Windows workstation, and future build agents" (§39.1–§39.10 — roles table, Operations VM role/limits, Linux Unity capabilities/limits, Windows workstation role, future Windows build/test agent, future macOS build host, development flow, Claude process separation, source-of-truth/recoverability, unchanged security/authority), old §39 "Primary references" renumbered to §40. Expanded existing §15.3 (`wishes-claude-inbox`) with the canonical external-request flow diagram. Added a "Development topology" subsection to the §38 completion checklist. Frontmatter: `status: draft` → `in-review`, `version: 0.1.0` → `0.2.0`, `last_reviewed` → 2026-09-02.
- **Canon status**: chapter lives in `drafts/deployment/`, not `canon/` — this was a **draft update, not canonization**. No `.docx` publication build was needed (drafts aren't published). Canonization requires the user to set `status: approved` and explicit go-ahead, then `./scripts/canonize-canon.sh`, per `WORKFLOW.md` — **remaining step, not done here**.
- **Conflicts with existing deployment canon**: none found — grepped `drafts/deployment/` for Unity/Windows/macOS/build-agent content before writing; this topology was previously undocumented.
- **Validation run**: `./scripts/validate-canon.sh` on the new branch — 247 errors / 2 warnings, byte-identical count to a `main`-baseline run (diffed); all are pre-existing missing-`.docx`-publication errors for already-canonized documents, none reference this file.
- **Commit**: `wishes-canon` branch `canon/cloud-claude-unity-development-topology`, commit `9fa3c8c`, pushed to `origin`. **No `gh` CLI available in this sandbox** — a PR was not opened programmatically; compare URL from the push: https://github.com/Krowthen/wishes-canon/pull/new/canon/cloud-claude-unity-development-topology
- **`wishes-game` submodule pointer**: not bumped — the edit lives on an unmerged branch; `docs/canon` submodule was left checked out at `main` (unchanged, matching origin), per the two-repo workflow (pointer only bumps after merge to `main`).
