# Step 03 — Agent Control Domain Placement Within `wishes-506905`

Created: 2026-09-04
Priority: Critical
Mode: design-and-cost-approval-package
Assigned Agent: claude-google
Allow Edit: planning/docs only

Depends on:
- `completed/agent-control-step-02-security-domain-reconciliation-claude-google.md`
- successful `claude-google` GCP management bootstrap or sufficient live read access to complete inventory

References:
- `pending/reference-agent-control-platform-revised-approved-design.md`
- `pending/agent-control-platform-master-deployment-plan.md`
- `pending/decision-s0-agent-control-placement-and-claude-google-management-authority.md`

## Human decision already made

The Human has explicitly approved S0 co-location in:

```text
GCP project: wishes-506905
Region: us-central1
```

Do **not** spend this step re-deciding whether to create a dedicated GCP project for S0, and do not create one solely for Agent Control.

The independent `agent_control` platform domain must still preserve separate database, service identity, authorization, secret-access and deployment boundaries so later extraction to a dedicated project remains possible.

This approval supersedes the earlier Step 03 requirement to compare a dedicated project against co-location as an unresolved Human decision.

## Objective

Using Step 02 findings plus the live inventory obtained after the `claude-google` management bootstrap, produce the concrete repository/database/network/runtime placement for the independent `agent_control` platform **inside `wishes-506905`**. No live resource creation occurs in this step unless a separate approved implementation task explicitly authorizes it.

## Required recommendation

### Repository

Compare and recommend:
- dedicated `agent-control` repository (preferred long-term);
- temporary isolated bootstrap path only if repository provisioning is not immediately available;
- migration/extraction plan for any bootstrap code.

### GCP project boundary — locked

Use:

```text
project: wishes-506905
region: us-central1
```

Within that project, define strict independent boundaries for:
- Agent Control service accounts;
- Agent Gateway/runtime services;
- `agent_control` database;
- event transport;
- Secret Manager access;
- logging/monitoring labels and ownership;
- Terraform state/stack ownership.

Document a future project-extraction path, but do not make it a prerequisite for S0.

### Database

Recommend a dedicated `agent_control` Cloud SQL PostgreSQL database boundary and initial sizing/topology. Compare:
- separate Cloud SQL instance inside `wishes-506905`;
- lower-cost shared-instance bootstrap only if it can maintain acceptable trust/blast-radius separation.

Do not place Agent Control tables in `wishes_core`, `wishes_assets`, or `wishes_auth`.

### Event transport

Recommend secure internal event transport for outbox dispatch and near-real-time runtime push. Compare Memorystore Redis, Pub/Sub, or approved existing platform capabilities as appropriate. Agent runtimes never connect directly to the event backend.

### Gateway / runtime bridge

Define:
- service names;
- region;
- ingress/auth model;
- outbound agent bridge connectivity;
- private DB/event networking;
- availability/scale-to-zero implications;
- local/Google runtime reachability.

### Identity / secrets

Define:
- Human OIDC provider boundary (no password DB initial release);
- agent device enrollment/short-lived platform token boundary;
- service identities;
- `claude-google` management impersonation boundary versus Agent Control runtime identities;
- Secret Manager namespace/labels/access for only server-side integration secrets;
- no external provider credential centralization.

## Cost package

Provide estimated monthly standing/light-use cost ranges for the selected in-project S0 architecture, with separate line items for:
- Cloud SQL;
- event transport/Redis;
- Cloud Run/API;
- NAT/egress if needed;
- Secret Manager/logging/storage;
- identity provider charges where known/configurable.

Use live pricing/quotas from the authorized environment where available; otherwise clearly mark estimates requiring verification.

## Security analysis

Assess the selected same-project placement on:
- tenant isolation;
- blast radius;
- IAM complexity;
- credential exposure risk;
- network isolation;
- recovery/rebuild;
- future project extraction;
- cost;
- operational complexity.

Explicitly identify compensating controls required because the platform and Wishes currently share a GCP project.

## Human Gate

The GCP project-placement decision itself is already approved and is not a remaining gate.

End with:
- one recommended concrete S0 topology inside `wishes-506905`;
- cost-bearing components that still require approval before creation;
- IAM/database/network changes requiring approval before apply;
- any material security blocker that would make the approved same-project placement unsafe.

Do not create databases, IAM bindings, secrets, network resources, repositories or service deployments in this design step unless separately authorized.
