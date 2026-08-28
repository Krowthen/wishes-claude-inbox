# S0 GCP Environment — Status Summary for Cross-Verification

Prepared 2026-08-28 by Claude Code (`wishes-game` session) for the user to
share with ChatGPT ahead of continuing `s0-minimal-cost-google-cloud-and-
comfyui.md`. Covers current state, decisions made this session, and open
gaps. Nothing in this session ran `terraform apply`, changed billing/IAM,
or created cloud resources — this is planning/reconciliation only.

## 1. Current state (verified by reading the repo, not live `gcloud`)

- **GCP project**: `wishes-506905`, region `us-west1`.
- **Deployed**: `wishes-comfy-broker` (Cloud Run, holds the Cloudflare
  Access service-token credential in Secret Manager, relays to a friend's
  remote ComfyUI at `https://comfyui.triumphcoding.net`) and
  `wishes-comfy-worker` (Cloud Run, the only thing allowed to call the
  broker, holds zero Cloudflare/Secret Manager credentials itself).
- **Not deployed**: the Wishes game application and the Node.js asset
  service have no Dockerfile, no Terraform, no Cloud Run presence — local
  dev processes only.
- **Database**: one local Postgres (`wishes`) via docker-compose, 11
  migrations, everything in the default `public` schema. No `CREATE ROLE`
  anywhere — a single shared `wishes` role for all access. No real
  authentication system exists (`app_user` + a hardcoded dev-user UUID is
  the current placeholder).
- **Redis**: provisioned in docker-compose but zero application code
  connects to it — unused today.
- **Pub/Sub**: no reference anywhere in the codebase.
- **Storage**: asset pipeline writes to local filesystem
  (`generated-assets/{pending,approved,rejected}`), no GCS buckets exist.
- **Terraform**: two flat, standalone stacks
  (`infrastructure/terraform/{comfy-broker,comfy-worker}/`), local state
  only (no GCS backend, no locking).
- **CI/CD**: none — no `.github/` directory.
- **Git**: working directly on `main`; the task's required
  `claude/s0-minimal-cloud` branch does not exist yet.
- **Asset routing**: as of this session, the Node asset service dispatches
  generation jobs to either `local` (direct local ComfyUI) or `friend_gcp`
  (via the worker) based on a per-job `execution_target` column, with no
  silent fallback between them. Built and tested (77 assertions) but not
  deployed anywhere.

## 2. Decisions made this session (need your read before Terraform is written)

### 2.1 Region
Task doc's default changed from `us-central1` to `us-west1` — this
**already matches** what's deployed (broker/worker Terraform already
defaulted to `us-west1`). No conflict; just a doc correction.

### 2.2 ComfyUI: multi-provider model (resolves the topology conflict)
The canon deployment docs (architecture overview, ComfyUI GPU platform
chapters, and the dedicated "ComfyUI Cloud Reconciliation" appendix) only
ever describe Wishes **owning** its ComfyUI runtime (a Cloud Run L4 GPU Job
now, GKE long-term) with Wishes-populated model/workflow buckets. None of
them anticipate the friend-broker-worker arrangement that's actually
deployed. Rather than pick one, the decision made this session is: **all
three coexist as named, registered providers**, and the system must
support registering more without a routing rewrite:

```text
local          direct local ComfyUI, no broker, no cloud auth (existing)
friend_gcp     wishes-comfy-broker + wishes-comfy-worker -> friend's
               remote ComfyUI behind Cloudflare Access (existing, deployed)
wishes_gpu_v3  Wishes-owned internal ComfyUI on a Cloud Run L4 GPU Job
               (net-new, canon's original self-hosted design, to be built)
```

Each `asset_workflow` row will declare which provider it targets (workflow/
model availability drives the provider, not an independent per-request
choice). The concrete first case, from the `comfy update flow.md` inbox
task, is folded in as the model for how future providers/versions register:
a `v1` workflow (`local`, Flux Schnell, negative conditioning is a no-op)
and a `v2` workflow (`friend_gcp`, Qwen-Image, real positive+negative
`CLIPTextEncode`, `res_multistep` sampler, turbo LoRA).

**Open task from `comfy update flow.md`, now tracked inside S0 Phase 7,
not yet executed:** register `v1`/`v2` in `asset_workflow` + seeds, add
`v2` generate/revise workflow JSON files, add a version selector + a
conditional negative-prompt field to the Character Creator portal, and
resolve whether a `v1`-generated asset can be revised with `v2` (if not,
lock revise to the generation version).

### 2.3 Queue management service — deferred placeholder only
Added as **Phase 7a**, explicitly scoped as "to be defined" and **not
required for initial S0 completion**. Rationale given: once the provider
registry grows past two or three ComfyUI instances, per-request routing
inside the asset service has no visibility into provider load/availability/
fair distribution. Stated responsibilities only (no design yet):
distribution (assign jobs to an available/capable provider), tracking
(job lifecycle independent of which provider runs it), completion
(finalize results into the existing approve/reject/attachment flow
unchanged). Explicitly: do not build ahead of a dedicated planning pass for
this.

### 2.4 Cloud SQL: split into three databases by trust/blast-radius domain
Original Phase 5 was "one `asset` schema in one `wishes` database, four
roles." Revised to **three separate databases on one Cloud SQL instance**
(not three instances — cost-driven choice for S0, with an explicit note
that `wishes_auth` could be promoted to its own instance later if
compliance/scale ever requires it):

```text
wishes_core    authoritative game state (cards, templates, characters,
               decks, world/tick state, gene/element data); game-facing
               app_user profile fields live here
wishes_assets  the generation pipeline (asset_type/role/workflow/queue/
               attachment) -- different write pattern and consumers than
               game state, formerly planned as a schema, now a database
wishes_auth    net-new: login/credentials (password hashes, MFA secrets,
               session/refresh tokens, login audit). No real auth system
               exists yet. Correlates to wishes_core.app_user by UUID
               value only (Postgres has no cross-database FKs). Only a
               dedicated auth service may hold its credentials -- not the
               game app, not the asset service.
```

Per-database owner + runtime roles (`wishes_core_owner/runtime`,
`wishes_assets_owner/runtime`, `wishes_auth_owner/runtime`), each with its
own Secret Manager credential; no role spans more than one database by
default. Rationale: Postgres enforces database boundaries at the
connection level (no `CONNECT` grant = the database is invisible), a
materially harder isolation boundary than schema-level `GRANT`s, at the
cost of one instance rather than three.

**Explicitly not designed yet:** migration ordering, and how the repo's
single-database migration-file convention
(`database/migrations/001...058_*.sql`) becomes per-database. Flagged in
the task doc as needing its own pass before this executes.

## 3. Update (same day): six explicit constraints resolved most of §2's open items

After this summary was first drafted, the user issued six explicit
constraints and asked for Phase 1/2 to be finalized. Applied, and **the
canon itself was updated to match** (`wishes-canon` Appendices A, B, C, E,
branch `ryancox-chatgpt`, commit `621f7e9`) so a future reader doesn't hit
the same reconciliation gap this task started with:

1. **Canonical project**: `wishes-506905` adopted via `terraform import`
   for the existing broker/worker/Artifact Registry/Secret Manager
   resources — no second project. *(Resolves the "one project" ambiguity.)*
2. **Buckets**: exactly the four planned shared buckets
   (models/workflows/inputs/outputs), provider/version key-prefixed, not
   per-provider bucket sets. *(Resolves gap 2 below — no separate
   `wishes_gpu_v3` bucket set needed.)*
3. **Cloud SQL**: one instance, three databases confirmed
   (`wishes_core`/`wishes_assets`/`wishes_auth`); no auth runtime built,
   no `wishes_auth` credential exposed to any existing service in this
   pass — explicit reservation, not a consumed service yet.
4. **Transport**: Pub/Sub is push-delivered to Cloud Run consumers;
   explicitly no always-on polling asset worker.
5. **Operations VM egress**: if NAT is needed, its design and cost are
   an **explicit pre-apply approval-package line item** — still
   unresolved by design, not decided here.
6. **Canon precursor**: appendices rewritten before finalizing, per
   above.

The IAM matrix (old gap 3) is now drafted — see the artifact — since it
was blocked only on the topology/bucket decisions above, both now
resolved.

## 4. Still-open gaps / questions (updated list)

1. **Cloud SQL instance topology**: one instance confirmed acceptable for
   S0 (constraint 3 above); `wishes_auth` promotion to its own instance
   remains a documented future option, not a current requirement.
2. ~~Bucket count/shape~~ — resolved (constraint 2 above).
3. ~~IAM matrix~~ — resolved, see the artifact.
4. **Migration-file convention** for a multi-database repo is still
   undecided (per-database subdirectories? separate migration runners?
   one runner with per-file target-database metadata?) — needs its own
   design pass before the `wishes_core`/`wishes_assets`/`wishes_auth`
   split is actually implemented.
5. **No live `gcloud` access** in this session — the quota/regional-
   availability report the task requires before apply has not been
   produced; needs to happen from an authorized environment.
6. **Operations VM NAT/egress**: explicitly deferred to the apply-gate
   package per constraint 5 above, not resolved here.
7. **`comfy update flow.md`'s concrete tasks** (asset_workflow columns,
   `v2` workflow files, portal UI) are captured in the plan but not
   started — still needs the "triage before execution" pass the original
   inbox task asked for.
8. **Queue management service (Phase 7a)** has zero design beyond stated
   responsibilities — genuinely open, not just unimplemented.
9. **Sync-script gap** (unrelated to architecture, flagged for repo
   hygiene): the external-inbox sync script never overwrites an existing
   local file, so a source-side update can silently strand a stale local
   copy — already happened once with this exact task file. Needs a
   content-hash/mtime-aware overwrite rule that still protects locally-
   edited files.
10. **Git workflow discipline**: today's broker/worker/asset-service work
    was committed straight to `main`; the task calls for a
    `claude/s0-minimal-cloud` branch + draft PR going forward — not yet
    started.
