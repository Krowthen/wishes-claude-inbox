# Claude Local Update — Cloud VM Migration State and File Handoff

## Purpose

This update is for the existing **Claude Code Local (Windows)** instance. It records what was actually migrated to the Wishes Claude Operations VM, what was intentionally not migrated, what was converted into shared Git-tracked tooling, and how Claude Local should connect to or transfer additional files to the VM if a future need is identified.

This document also resolves the local-only audit questions raised before the migration.

---

## Current cloud Claude environment

Operations VM:

- GCP project: `wishes-506905`
- VM: `wishes-s0-usc1-claude-ops`
- zone: `us-central1-a`
- OS: Ubuntu 24.04 LTS
- VM internal IP: `10.40.0.2`
- no external IP
- inbound access: IAP + OS Login
- outbound access: Cloud NAT currently enabled
- primary user: `ryan_shanahan_cox_gmail_com`

Cloud workspace:

```text
/home/ryan_shanahan_cox_gmail_com/wishes/
├── wishes-game/
│   └── docs/canon/          # wishes-canon submodule
├── wishes-canon/            # direct canon working clone
└── wishes-claude-inbox/     # direct task inbox clone
```

Primary Claude session root:

```text
/home/ryan_shanahan_cox_gmail_com/wishes/wishes-game
```

Claude Code runs inside a persistent tmux session and now starts cleanly on Linux.

---

# Resolution of the pre-migration local-only audit

## 1. GCS / asset-storage WIP

### Audit question

The asset-storage work had previously been described as possibly uncommitted/local-only.

### Resolution

False alarm. The work was committed and pushed before the cloud handoff.

Cloud handoff branch:

```text
chatgpt/asset-storage-cloud-handoff
```

Relevant commits:

```text
5668d9e feat(asset-service): add canonical GCS storage foundation
de85dba chore(claude): make session start hook cross-platform
f4b9b29 chore(claude): track shared operator status tools
```

The VM pulled this branch and validated the asset-service GCS work under Linux.

Validation completed on the VM:

- `server.mjs` syntax: pass
- `pipeline.mjs` syntax: pass
- GCS storage tests: 17 passed, 0 failed

### Action for Claude Local

Do not manually copy these tracked files to the VM. Continue using normal Git commit/push/pull workflow.

---

## 2. Local Git branches across the three Wishes repos

### Audit question

Could there be local-only branch work not visible from the VM?

### Resolution

The audit found no local-only commits requiring manual migration at that time. The cloud environment now has direct clones of all three repositories.

### Action for Claude Local

For any future tracked source/documentation change:

1. commit locally;
2. push to the repo's own origin;
3. fetch/pull or switch to the branch on the VM.

Do not use SCP for ordinary Git-tracked project files.

---

## 3. `tools/claude/comfyui/*`

### Audit question

The following were local-only and gitignored:

```text
tools/claude/comfyui/start.mjs
tools/claude/comfyui/stop.mjs
tools/claude/comfyui/health-check.mjs
```

### Resolution

**Intentionally NOT migrated.**

These remain Windows/local-GPU operator tooling. The Operations VM is not the local ComfyUI GPU host and is not intended to replace local ComfyUI generation.

### Action for Claude Local

Leave these files local unless architecture is explicitly changed later.

Do not copy them merely to make the Windows and Linux environments identical.

---

## 4. `tools/claude/dev-status.mjs`

### Audit question

This was local-only and used by the Claude status line.

### Resolution

Promoted from local-only tooling to **shared cross-platform Git-tracked tooling**.

Tracked file:

```text
tools/claude/dev-status.mjs
```

Validated on both Windows and Linux.

Linux output currently reports `dev: monitor off` when no background watcher is running, which is expected.

### Action for Claude Local

Use Git only. Do not SCP this file.

---

## 5. `tools/claude/kill-monitors.mjs`

### Audit question

This was local-only but clears orphaned `dev-status.mjs --changes` watchers.

### Resolution

Promoted to **shared cross-platform Git-tracked tooling**.

Tracked file:

```text
tools/claude/kill-monitors.mjs
```

Implementation behavior:

- Windows: PowerShell/CIM process lookup and termination
- Linux: `pkill -f` against the dev-status watcher pattern

Validated on both Windows and Linux.

### Action for Claude Local

Use Git only. Do not SCP this file.

---

## 6. `.claude/settings.local.json`

### Audit question

The Windows local settings file contained the actual accumulated local permissions/autoMode policy and was not tracked.

### Resolution

The Windows file was **not copied wholesale** to Linux.

Reason: it contained a large amount of Windows-specific historical permission state, including PowerShell, Windows paths, Docker Desktop, Windows virtualenv paths, local ComfyUI behavior, `tasklist`, `taskkill`, and other machine-specific commands.

Instead, a clean Linux-specific local file was manually created on the VM:

```text
/home/ryan_shanahan_cox_gmail_com/wishes/wishes-game/.claude/settings.local.json
```

The Linux policy preserves the important high-level rules:

- primary repo is `wishes-game`;
- `wishes-canon` and `wishes-claude-inbox` are additional trusted directories;
- normal repo-local Git work is allowed;
- project tests/builds/plan/validation are allowed;
- destructive Git requires user confirmation;
- real GCP/IAM/database/deployment mutation is human-only;
- Terraform `apply` / `destroy` are not part of autonomous Claude permissions;
- the VM is orchestration/development, not graphical Unity or local ComfyUI GPU execution.

### Action for Claude Local

Do not overwrite the Linux `settings.local.json` with the Windows file.

Future policy changes should be reconciled intentionally between environments rather than blindly copied.

---

## 7. `.claude/settings.json` / SessionStart hook

### Audit question

The tracked SessionStart hook called:

```text
powershell -File .claude/hooks/session-start.ps1
```

which failed on Linux.

### Resolution

Fixed.

The PowerShell-specific SessionStart implementation was replaced with a cross-platform Node implementation:

```text
.claude/hooks/session-start.mjs
```

and `.claude/settings.json` now invokes Node.

The hook still:

- resolves repo root;
- reads `.claude/last-session-marker`;
- summarizes commits/status/diff since the previous session;
- emits Claude `SessionStart` `additionalContext` JSON.

Windows Claude startup was tested cleanly.
Linux Claude startup was tested cleanly.

### Action for Claude Local

Use the tracked Node hook. Do not restore the PowerShell-only startup path.

---

## 8. `.claude/last-session-marker`

### Resolution

Not migrated and should remain machine-local.

Windows Claude and VM Claude should each maintain their own session marker timestamp.

### Action for Claude Local

Do not copy or synchronize this file.

---

## 9. Claude local memory outside Git

### Audit question

Claude Code's own local memory under the Windows user profile is outside the repository and keyed to the Windows project path.

### Resolution

**Not migrated.**

The VM starts with its own Claude memory namespace.

Durable project knowledge should live in:

- canon;
- `CLAUDE.md`;
- session handoff docs;
- tracked project documentation;
- explicit task/inbox records.

### Action for Claude Local

Do not copy the entire local Claude memory tree unless explicitly directed later.

If a piece of knowledge is important to both agents, move the knowledge into a durable project source instead of depending on hidden per-machine memory.

---

## 10. Local inbox processing state

### Audit question

Local `tools/claude/inbox/` processing state contained pending/active/completed/archive bookkeeping.

### Resolution

**Not manually copied.**

The authoritative external task repository is available directly on the VM at:

```text
/home/ryan_shanahan_cox_gmail_com/wishes/wishes-claude-inbox
```

Local processing state is reconstructible from the external inbox and Git history where needed.

### Action for Claude Local

Do not blindly copy local inbox bookkeeping to the VM.

Future agent-communication architecture may replace or evolve this mechanism; until then, Git/inbox remains the durable task record.

---

## 11. `.env` files and local secrets

### Audit question

Local `.env` files exist under services/prototypes and are intentionally not tracked.

### Resolution

**Not migrated.**

No secret-bearing `.env` file was manually copied as part of this migration.

### Action for Claude Local

Never commit these files.

Do not transfer them to the VM unless the human explicitly determines that a specific runtime needs a specific value.

Prefer secure re-provisioning from the appropriate secret/config source over copying a developer `.env` wholesale.

---

## 12. `.local/cloudflared/` credentials and `cloudflared.exe`

### Audit question

The Windows workstation has local Cloudflare tunnel credentials and a Windows `cloudflared.exe` binary.

### Resolution

**Not migrated.**

The VM currently does not need to become the local Cloudflare tunnel/ComfyUI host.

The Windows binary would not be usable on Linux anyway.

### Action for Claude Local

Do not copy the Windows binary or tunnel credential bundle to the VM.

If a future architecture explicitly requires cloudflared on the VM, install the Linux build and provision only the required credentials through an approved secure process.

---

## 13. Local ComfyUI / GPU generation

### Resolution

Intentionally remains on the Windows workstation.

The Operations VM has no GPU and is not the local ComfyUI execution host.

### Action for Claude Local

No migration required.

Do not treat absence of local ComfyUI on the VM as an error.

---

# Files manually transferred to the VM

## Terraform bootstrap state — manually copied

The one critical local-only state identified in the audit was:

```text
infrastructure/terraform/s0-bootstrap/terraform.tfstate
infrastructure/terraform/s0-bootstrap/terraform.tfstate.backup
```

These files were intentionally local by design and are ignored by Git.

They were manually transferred from Windows to the VM using IAP SCP.

Destination:

```text
/home/ryan_shanahan_cox_gmail_com/wishes/wishes-game/infrastructure/terraform/s0-bootstrap/
```

VM permissions were set to:

```text
-rw-------
```

via:

```bash
chmod 600 terraform.tfstate terraform.tfstate.backup
```

The transferred files were verified byte-identical using SHA-256.

Expected hashes:

```text
terraform.tfstate
CE4D1BFD35EFCC83A0F96C0532130BB24358F4874AF6F21EFB70B51CAB449AAC

terraform.tfstate.backup
2E4215B49C61832EAE860FB0FCF253B65A9EE32B4DF43EE428085D79630BA19F
```

### Important

Do not commit either Terraform state file.

Do not casually overwrite the VM copy with a Windows copy or vice versa. If the bootstrap stack is operated from one machine, reconcile state ownership before operating it from the other.

---

# Files/configuration created manually on the VM rather than copied

The Linux `.claude/settings.local.json` was created directly on the VM and is intentionally machine-local.

It should not be overwritten by the Windows version.

The VM also has machine-local authentication/configuration that was created through installation/login flows rather than copied from Windows, including:

- Claude Code authentication;
- GitHub CLI authentication;
- Git global author identity;
- GCP attached service-account context;
- tmux runtime/session state.

These are not Git artifacts and should not be copied back and forth between machines.

---

# How Claude Local connects to the cloud VM

Run from Windows PowerShell.

## Interactive SSH through IAP

```powershell
gcloud.cmd compute ssh wishes-s0-usc1-claude-ops `
  --zone=us-central1-a `
  --project=wishes-506905 `
  --tunnel-through-iap
```

The VM has no public IP; do not attempt direct public SSH.

After connecting, the primary workspace is:

```bash
cd /home/ryan_shanahan_cox_gmail_com/wishes/wishes-game
```

Persistent Claude tmux session:

```bash
tmux attach -t wishes-claude
```

If the session does not exist, create it from the repo root:

```bash
tmux new-session -s wishes-claude -c /home/ryan_shanahan_cox_gmail_com/wishes/wishes-game
```

---

# Preferred method for moving additional files

## Rule 1 — if the file belongs in Git, use Git

This is the normal path for source, docs, shared tooling, Terraform configuration, tests, scripts, and other durable repository content.

### Windows / Claude Local

```powershell
git status
git add <paths>
git commit -m "<message>"
git push
```

### VM

```bash
cd /home/ryan_shanahan_cox_gmail_com/wishes/wishes-game
git fetch origin
git pull --ff-only
```

If the work is on a new branch:

```bash
git fetch origin
git switch --track origin/<branch-name>
```

Use the corresponding repo directory for `wishes-canon` or `wishes-claude-inbox`.

This is preferred because Git provides history, reviewability, synchronization, and recovery.

---

## Rule 2 — use SCP only for intentionally untracked machine-local files

Examples may include:

- local Terraform state when explicitly approved;
- a non-secret local artifact that truly must exist on both machines;
- temporary diagnostics specifically requested by the human.

Do **not** use SCP as a replacement for Git.

Do **not** use it to casually move `.env` files, Cloudflare credentials, private keys, access tokens, Claude authentication, GitHub authentication, or other secrets.

---

# How to copy an individual local-only file to the VM

Windows `gcloud compute scp` uses PuTTY `pscp` in this environment, and `~` did not expand reliably for the remote path.

Always use the full Linux destination path.

Example:

```powershell
gcloud.cmd compute scp `
  .\path\to\local-file `
  wishes-s0-usc1-claude-ops:/home/ryan_shanahan_cox_gmail_com/wishes/wishes-game/path/to/destination/ `
  --zone=us-central1-a `
  --project=wishes-506905 `
  --tunnel-through-iap
```

For a file intended for `wishes-canon`, use:

```text
/home/ryan_shanahan_cox_gmail_com/wishes/wishes-canon/
```

For a file intended for `wishes-claude-inbox`, use:

```text
/home/ryan_shanahan_cox_gmail_com/wishes/wishes-claude-inbox/
```

Before copying, ensure the destination directory exists.

---

# How to copy a directory if explicitly required

Use recursive SCP only when the human has determined the entire directory should be transferred:

```powershell
gcloud.cmd compute scp `
  --recurse `
  .\path\to\directory `
  wishes-s0-usc1-claude-ops:/home/ryan_shanahan_cox_gmail_com/wishes/wishes-game/path/to/parent/ `
  --zone=us-central1-a `
  --project=wishes-506905 `
  --tunnel-through-iap
```

Do not recursively copy broad local configuration directories such as `.claude`, `.local`, `.env` trees, credential stores, or user-profile directories.

---

# Verification after any manual file transfer

For important non-secret files, verify size/hash on both machines.

## Windows

```powershell
Get-FileHash .\path\to\file -Algorithm SHA256
```

## Linux VM

```bash
sha256sum /path/to/file
```

For sensitive local state, also restrict Linux permissions where appropriate:

```bash
chmod 600 /path/to/file
```

Then confirm the file is still ignored if it belongs to an ignored category:

```bash
git check-ignore -v /path/inside/repo
```

---

# Safety rules for Claude Local when transferring files

1. Prefer Git for anything that is supposed to be durable/shared project content.
2. Never commit Terraform state.
3. Never commit `.env`, Cloudflare credentials, private keys, GitHub/Claude/GCP tokens, or other credentials.
4. Never copy the entire Windows `.claude/settings.local.json` to Linux.
5. Never copy Windows binaries such as `cloudflared.exe` to Linux.
6. Do not migrate local ComfyUI GPU tooling merely because it is missing on the VM.
7. Do not overwrite a VM-local file without first checking whether the VM has diverged.
8. For critical manual copies, verify SHA-256 after transfer.
9. Real cloud/IAM/deployment mutation remains human-executed. Claude may prepare plans/commands but must not autonomously apply them.
10. Do not change the current cloud infrastructure architecture or milestone scope as a side effect of this handoff.

---

# Current interpretation for each audited item

| Audited item | Current decision |
|---|---|
| GCS/asset-storage WIP | Already committed/pushed; use Git |
| Local Git branches | No manual copy needed; use Git |
| `tools/claude/comfyui/*` | Stay Windows-local |
| `tools/claude/dev-status.mjs` | Promoted to tracked shared tool |
| `tools/claude/kill-monitors.mjs` | Promoted to tracked shared tool |
| Windows `.claude/settings.local.json` | Do not copy; Linux-specific version created separately |
| SessionStart PowerShell hook | Replaced by tracked cross-platform Node hook |
| `.claude/last-session-marker` | Machine-local; do not synchronize |
| `tools/claude/inbox/` local processing state | Do not blindly copy; reconstruct/sync from external inbox |
| `.env` files | Not migrated; secure re-provision only if required |
| `.local/cloudflared/` credentials | Not migrated |
| `cloudflared.exe` | Windows-only; not migrated |
| Local ComfyUI/GPU runtime | Remains on Windows by architecture |
| Claude per-machine memory | Not migrated; durable knowledge belongs in tracked project sources |
| `s0-bootstrap/terraform.tfstate` | Manually copied to VM, chmod 600, hashes verified |
| `s0-bootstrap/terraform.tfstate.backup` | Manually copied to VM, chmod 600, hashes verified |

---

# Final state

The Claude Operations VM migration is operationally complete for Claude development/orchestration.

Claude Local should treat the VM as a second trusted Wishes development environment, **not** as a byte-for-byte mirror of Windows.

The intended split is:

```text
Shared durable project state
  -> Git / canon / inbox

Linux VM-specific state
  -> Linux settings, auth, tmux, VM-local state

Windows-specific state
  -> local ComfyUI/GPU, Windows operator helpers, Windows local credentials

Sensitive state
  -> never Git; provision intentionally and minimally
```

If Claude Local discovers another local-only file that may matter to the VM, first classify it as:

1. shared/durable -> promote or commit through Git;
2. machine-local but required -> ask the human before SCP;
3. secret/credential -> do not copy automatically; request secure provisioning decision;
4. Windows-only runtime/tool -> leave local unless architecture explicitly changes.
