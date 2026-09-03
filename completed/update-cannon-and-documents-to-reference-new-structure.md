WORK REQUEST — Update Wishes Canon: Comfy Execution vs Wishes Asset Storage Boundaries

Repository:
Krowthen/wishes-canon

Objective:
Update the Wishes canon/architecture documentation to establish a hard architectural separation between the generic Comfy execution platform and the Wishes asset-management platform.

Review the existing canon repository before editing. Merge these decisions into the most appropriate existing architecture/security/storage documents rather than creating duplicate concepts. Do not overwrite newer canon blindly. If a document already defines related storage, worker, database, asset, or security boundaries, reconcile it with the rules below.

CANONICAL ARCHITECTURE DECISION

1. TWO DISTINCT PLATFORM DOMAINS

A. Comfy Execution Platform
The Comfy execution platform is a general-purpose image/workflow execution system.

It is NOT intrinsically part of the Wishes asset-management system.

Not every Comfy request originates from Wishes.

The Comfy platform must therefore maintain its own:
- Security boundary
- Request/execution identity model
- Execution/job state
- Workers
- Storage
- Input/output lifecycle
- Authorization rules

A valid authorized caller may use the Comfy execution platform independently of Wishes.

B. Wishes Asset Platform
The Wishes asset platform is the authoritative game-asset management system.

It has its own:
- High-security boundary
- wishes_assets database
- Asset request queue
- Asset roles/types/workflows
- Versioning
- Approval/rejection lifecycle
- Current-attachment state
- Asset history/state
- Wishes-specific workers/orchestration
- Canonical asset storage

The Wishes asset platform may use the Comfy platform as an execution provider, but the two systems must remain architecturally separate.


2. CORE OWNERSHIP RULE

Canonical rule:

"Comfy generates outputs. Wishes decides what becomes a Wishes asset."

A raw Comfy output MUST NOT automatically become a Wishes asset.

When Wishes requests generation:
1. Wishes creates and owns the Wishes asset request.
2. Wishes invokes an authorized Comfy execution path.
3. Comfy performs the execution.
4. Comfy stores its raw execution output in Comfy-owned storage.
5. Wishes receives or retrieves the execution result.
6. Wishes validates/imports the selected output.
7. Only the Wishes asset platform writes the canonical Wishes asset.
8. Wishes then owns approval, rejection, versioning, attachment, and lifecycle state.


3. COMFY EXECUTION STORAGE

Comfy must own independent storage for its execution environment.

Conceptually this includes:
- Models
- Workflows
- Execution inputs
- Execution outputs

The currently established S0 buckets for:
- models
- workflows
- inputs
- outputs

should be treated as Comfy execution-platform storage unless an existing canon document explicitly establishes a better naming/ownership structure.

Comfy output storage is raw execution storage, not the canonical Wishes asset repository.

Example conceptual object structure:

comfy-output-bucket/
  jobs/
    <execution-job-id>/
      output-0.png
      output-1.png

Exact object layout is implementation-specific unless already canonized elsewhere.


4. WISHES ASSET STORAGE

Wishes requires separate canonical asset storage.

This storage must NOT be the same security domain as generic Comfy output storage.

The Wishes asset store should support the authoritative Wishes lifecycle, conceptually including:

pending/
approved/
rejected/

These are Wishes lifecycle states, not Comfy execution states.

Wishes assets may originate from:
- Comfy execution
- User/custom upload
- Template cloning
- Other future asset-generation providers

All of those sources ultimately enter the same protected Wishes asset-management system.


5. SECURITY BOUNDARY

The Comfy worker identity must NOT receive direct write authority over canonical Wishes asset storage.

Expected conceptual permissions:

Comfy worker/runtime:
- Authorized access to required Comfy models
- Authorized access to required Comfy workflows
- Authorized access to Comfy execution inputs
- Create/read appropriate Comfy execution outputs
- NO authority to directly create, approve, modify, replace, or delete canonical Wishes assets

Wishes asset runtime:
- Authorized to create/manage Wishes asset requests
- Authorized to invoke permitted Comfy execution workers
- Authorized to retrieve/read the corresponding Comfy execution outputs
- Authorized to manage canonical Wishes asset storage
- Authorized to use the wishes_assets runtime database role
- Responsible for importing raw execution results into Wishes

This boundary must remain true for future Comfy workers/providers, including local execution and future Wishes-owned GPU execution.


6. DATABASE SEPARATION

The existing Wishes asset structures remain Wishes-specific concepts, including structures such as:

- asset_generation_queue
- asset_attachment
- asset_workflow
- asset_role
- asset_type

Do NOT redefine the Wishes asset queue as the generic Comfy execution queue.

Generic Comfy execution state must be treated separately.

A generic Comfy execution system may contain concepts such as:
- execution/job ID
- authorized caller
- requested workflow
- status
- timestamps
- output object references
- retention/expiry

The exact Comfy persistence implementation remains open unless it has already been canonized elsewhere.

A Wishes asset request may reference a Comfy execution/job ID, but the two records have different ownership and semantics.


7. LOCAL COMFY EXECUTION

Local ComfyUI is an execution provider, not an asset-storage authority.

Even when an asset is generated locally for Wishes:
- Local Comfy performs execution.
- The resulting Wishes asset must ultimately be persisted to cloud-backed canonical Wishes asset storage.
- Local filesystem storage is temporary/staging/debug behavior only and must not be treated as the authoritative Wishes asset location.

This ensures the cloud-hosted Wishes database and application can consistently resolve assets regardless of where generation occurred.


8. CLOUD COMFY EXECUTION

The existing friend_gcp flow remains conceptually:

Authorized caller
  -> Comfy worker
  -> authenticated broker boundary as required
  -> protected ComfyUI
  -> Comfy-owned raw output storage

For a Wishes-originated request:

Wishes asset service
  -> Wishes asset request / queue
  -> Comfy worker
  -> Comfy execution
  -> Comfy raw output
  -> Wishes validates/imports output
  -> canonical Wishes asset storage
  -> Wishes attachment/version lifecycle

Do not collapse these into one storage or authorization layer.


9. FUTURE GPU PROVIDERS

Future execution targets such as wishes_gpu_v3 must follow the same separation.

Execution-provider choice must not alter asset ownership.

Whether execution occurs through:
- local ComfyUI
- friend_gcp
- wishes_gpu_v3
- another future provider

the provider produces an execution output.

If the request belongs to Wishes, the Wishes asset platform then imports and owns the canonical asset.


10. CANON TERMINOLOGY

Use these terms consistently:

"Comfy Execution Platform"
= generic execution infrastructure.

"Comfy Output"
= raw result produced by a Comfy execution job.

"Wishes Asset Platform"
= authoritative Wishes-specific request, lifecycle, versioning, attachment, and asset-management system.

"Wishes Asset"
= an asset accepted/imported into the Wishes asset-management boundary.

"Canonical Wishes Asset Storage"
= the authoritative cloud-backed storage controlled by the Wishes asset platform.

Avoid documentation that uses "Comfy output" and "Wishes asset" interchangeably.


11. CURRENT S0 IMPLICATION

Document that the existing S0:
- model bucket
- workflow bucket
- input bucket
- output bucket

belong to / should be interpreted as the Comfy execution side of the architecture.

Add a documented requirement for separate canonical Wishes asset storage.

Do not invent or deploy a concrete bucket name unless naming rules already make it deterministic. This work request is primarily a canon/architecture update, not an infrastructure deployment.


12. OPEN IMPLEMENTATION ITEMS

Record these as implementation work rather than unresolved architecture:

- Create/provision canonical Wishes asset storage.
- Establish IAM separation between Comfy workers and Wishes asset runtime.
- Implement Wishes import from Comfy raw output into canonical asset storage.
- Refactor asset-service filesystem persistence to cloud-backed Wishes asset storage.
- Preserve local generation while making cloud storage canonical.
- Determine raw Comfy output retention/expiration policy.
- Determine whether generic Comfy execution metadata needs its own persistent database.
- Determine protected asset-delivery mechanism for clients, such as signed URLs or an authenticated asset-serving endpoint.
- Backfill existing source-controlled approved assets into canonical cloud storage when migration is executed.


13. NON-GOALS / DO NOT CHANGE

Do not:
- Merge the Comfy execution queue with wishes_assets.
- Give Comfy workers direct authoritative control of Wishes assets.
- Make the generic Comfy platform Wishes-only.
- Treat local generated-assets folders as future canonical storage.
- Deploy infrastructure as part of this canon-only request.
- Modify runtime code in wishes-game as part of this request.
- Rewrite unrelated lore/gameplay canon.


14. EXECUTION

Review the existing wishes-canon repository structure first.

Update the appropriate existing documents and indexes/references so this architecture is discoverable from the canon structure.

Where current canon conflicts with this decision, update it to the new canonical architecture while preserving still-valid material.

If a new architecture document is genuinely required, place it within the repository's existing architecture/technical-canon structure and link it from the appropriate parent/index.

Validate internal references.

Provide a final report containing:
- Files modified/created
- Existing statements superseded or clarified
- Canon decisions recorded
- Remaining implementation gaps
- Any conflict discovered with previously locked canon

Commit the canon changes as one focused commit with:

docs(canon): separate Comfy execution and Wishes asset domains

Push the branch according to the repository's existing ChatGPT/Claude contribution workflow.

Do not merge a pull request unless the repository workflow explicitly requires it and prior instructions authorize that merge.
---

## Completion report (2026-09-02)

Executed in `wishes-canon` on new branch `canon/comfy-execution-wishes-asset-separation` (commit `08186e2`), pushed — not merged, per instructions. `docs/canon` returned to `main` afterward (unchanged, `c3ee228`); `wishes-game`'s submodule pointer untouched.

**Files modified** (all under `drafts/deployment/`, since this is architecture/deployment material, not locked lore canon):
- `appendices/e-comfyui-cloud-reconciliation.md` — new terminology/ownership-rule section (§3b), corrected §4, new open-gaps sections (§12/§13)
- `appendices/c-iam-secrets-and-data-boundaries.md` — new §11a (confirms IAM already meets the security half of the boundary; flags the storage half as still open)
- `appendices/a-s0-minimal-cost-environment-profile.md` — new decision 39; corrected Outputs-bucket table row (was using Wishes lifecycle language for what is Comfy execution storage)
- `appendices/b-resource-variable-and-cost-catalog.md` — note on `ASSET_OUTPUT_BUCKET`'s current domain
- `00-architecture-overview.md` — split §11.4 object-storage list by domain, new §11.5, fixed the ComfyUI Worker IAM-table row that improperly granted asset-record write access
- `07-storage-and-artifacts.md` — new §4.9a, Domain column on the bucket-inventory table, IAM footnote
- `09-comfyui-gpu-platform.md`, `09a-comfyui-management-layer.md`, `14-security.md` — domain-boundary notes
- `README.md`, `appendices/README.md` — version bumps + cross-references

**Canon decisions recorded**: two-domain split (Comfy Execution Platform vs. Wishes Asset Platform); ownership rule ("Comfy generates outputs. Wishes decides what becomes a Wishes asset."); the S0 four buckets (models/workflows/inputs/outputs) documented as Comfy execution storage, not canonical Wishes asset storage; security boundary — the Comfy worker identity has no authority to create/approve/modify/delete canonical Wishes assets; database separation preserved (`asset_generation_queue`/`asset_attachment`/etc. stay Wishes-specific, not redefined as generic Comfy execution state).

**Remaining implementation gaps** (recorded as backlog, not built): canonical Wishes asset storage doesn't exist yet; no IAM separation for it; no actual Comfy-Output → Wishes-asset import step; asset-service still persists to local `generated-assets/` (staging-only) pending the cloud-backed store; retention policy, a possible separate Comfy execution-metadata database, and client asset-delivery mechanism are all open.

**No conflict with locked canon** — deployment/architecture drafts only; nothing touches Core Concept 1/2 (elements/realms/genes/cosmology).

`validate-canon.sh`: 247 pre-existing errors/2 warnings, all in unrelated `canon/nations/*` publication files — none reference any file this task touched.
